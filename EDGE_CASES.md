# Edge Cases Analysis

## Currently Tested ✅

1. **Health Check** - Server availability
2. **Version Check** - API versioning
3. **Empty Collections** - GET when no data exists
4. **Full CRUD Operations** - Create, Read, Update, Delete for both resources
5. **Foreign Key Constraint** - Cannot delete category with products (409)
6. **Empty Name Validation** - Rejects empty strings (400)
7. **Invalid ID Format** - Non-numeric IDs (400)
8. **Non-existent Resources** - GET/PUT/DELETE on missing IDs (404)
9. **Category-Product Relationship** - JOIN queries return category info
10. **Concurrent Operations** - Multiple creates, updates, deletes

## Missing Edge Cases ❌

### 1. **Negative or Zero Values**
- **What:** Product price/stock with negative or zero values
- **Current:** No validation, accepts any integer
- **Impact:** Could create products with -$100 or 0 stock
- **Fix:** Add validation: `price > 0` and `stock >= 0`
```bash
# Test case
curl -X POST http://localhost:8080/products \
  -H "Content-Type: application/json" \
  -d '{"name":"Bad Product","price":-50,"stock":-10,"category_id":1}'
# Expected: 400 Bad Request
# Actual: 201 Created (BUG)
```

### 2. **Non-existent Category ID in Product Creation**
- **What:** Creating product with `category_id` that doesn't exist
- **Current:** Creates product successfully (silently fails to JOIN)
- **Impact:** Orphaned products with no category info
- **Fix:** Add foreign key constraint validation
```bash
# Test case
curl -X POST http://localhost:8080/products \
  -H "Content-Type: application/json" \
  -d '{"name":"Orphan","price":100,"stock":5,"category_id":9999}'
# Expected: 400 Bad Request (invalid category)
# Actual: 201 Created (BUG)
```

### 3. **Whitespace-Only Names**
- **What:** Names with only spaces: `"   "`
- **Current:** Accepts them (validation only checks `== ""`)
- **Impact:** Creates confusing resources
- **Fix:** Trim and validate: `strings.TrimSpace(name) == ""`
```bash
# Test case
curl -X POST http://localhost:8080/categories \
  -H "Content-Type: application/json" \
  -d '{"name":"   ","description":"spaces only"}'
# Expected: 400 Bad Request
# Actual: 201 Created (BUG)
```

### 4. **Very Long Names/Descriptions**
- **What:** Extremely long strings (e.g., 10MB text)
- **Current:** No length limits
- **Impact:** Database bloat, performance issues
- **Fix:** Add max length validation (e.g., 255 for name, 5000 for description)
```bash
# Test case with 1000+ character name
curl -X POST http://localhost:8080/categories \
  -H "Content-Type: application/json" \
  -d '{"name":"'$(printf 'a%.0s' {1..1000})'","description":"test"}'
# Expected: 400 Bad Request (name too long)
# Actual: 201 Created (BUG)
```

### 5. **Special Characters in Names**
- **What:** SQL injection attempts or control characters
- **Current:** Parameterized queries prevent injection, but no validation
- **Impact:** Weird data in database
- **Fix:** Add character validation or sanitization
```bash
# Test case
curl -X POST http://localhost:8080/categories \
  -H "Content-Type: application/json" \
  -d '{"name":"Test<script>alert(1)</script>","description":"XSS attempt"}'
# Expected: 400 Bad Request or escaped
# Actual: 201 Created with HTML tags
```

### 6. **Invalid JSON Structure**
- **What:** Malformed JSON in request body
- **Current:** Returns "Invalid request body" (generic)
- **Impact:** No detailed error info
- **Fix:** Provide more specific JSON parsing errors
```bash
# Test case - missing closing brace
curl -X POST http://localhost:8080/categories \
  -H "Content-Type: application/json" \
  -d '{"name":"Test","description":"missing brace"'
# Returns: 400 Bad Request (too generic)
```

### 7. **Missing Required Fields in JSON**
- **What:** POST/PUT without required fields
- **Current:** Partially validated, but `category_id: 0` might be accepted
- **Impact:** Incomplete data
- **Fix:** Validate all required fields explicitly
```bash
# Test case - missing category_id
curl -X POST http://localhost:8080/products \
  -H "Content-Type: application/json" \
  -d '{"name":"Incomplete","price":100,"stock":5}'
# Expected: 400 Bad Request (missing category_id)
# Actual: 201 Created with category_id: 0
```

### 8. **Duplicate Names**
- **What:** Creating categories/products with same name
- **Current:** Allowed (no unique constraint)
- **Impact:** Confusing for users
- **Fix:** Consider unique constraint or documented behavior
```bash
curl -X POST http://localhost:8080/categories \
  -H "Content-Type: application/json" \
  -d '{"name":"Electronics","description":"First"}'

curl -X POST http://localhost:8080/categories \
  -H "Content-Type: application/json" \
  -d '{"name":"Electronics","description":"Second"}'
# Both succeed - is this expected?
```

### 9. **Large ID Values**
- **What:** IDs beyond typical ranges (e.g., 2^31-1)
- **Current:** Uses `int` (32-bit or 64-bit depending on platform)
- **Impact:** Potential overflow on some platforms
- **Fix:** Document ID range limits or use `int64`

### 10. **NULL Values in Optional Fields**
- **What:** Explicitly sending `null` for description
- **Current:** Might fail or behave unexpectedly
- **Impact:** Inconsistent behavior
- **Fix:** Validate and convert nulls to empty strings
```bash
curl -X POST http://localhost:8080/categories \
  -H "Content-Type: application/json" \
  -d '{"name":"Test","description":null}'
# Should convert to empty string
```

### 11. **Concurrent Update Conflicts**
- **What:** Two updates to same resource simultaneously
- **Current:** Last-write-wins (no conflict detection)
- **Impact:** Data loss if updates conflict
- **Fix:** Add version/timestamp-based optimistic locking
```bash
# Simulate concurrent updates
curl -X PUT http://localhost:8080/categories/1 ... &
curl -X PUT http://localhost:8080/categories/1 ... &
# Second update might overwrite first without warning
```

### 12. **Floating Point Prices**
- **What:** Price with decimals: `"price": 19.99`
- **Current:** Field is `int` (cents only)
- **Impact:** Precision loss or parsing errors
- **Fix:** Clarify API (cents only, no decimals) or change to `float64`
```bash
curl -X POST http://localhost:8080/products \
  -H "Content-Type: application/json" \
  -d '{"name":"Item","price":19.99,"stock":5,"category_id":1}'
# Expected: 400 Bad Request (invalid type) or 201 with price: 19
```

### 13. **Race Condition on Delete**
- **What:** Delete followed immediately by GET
- **Current:** Proper 404 (should be fine)
- **Impact:** None currently
- **Note:** Already working correctly

### 14. **Empty Product List with Valid Category**
- **What:** GET /products when category exists but no products
- **Current:** Returns `[]` (correct)
- **Impact:** None
- **Note:** Already working correctly

### 15. **Update Product with Non-existent Category**
- **What:** Updating product to reference deleted category
- **Current:** Probably succeeds (no FK check on update)
- **Impact:** Orphaned products
- **Fix:** Add FK validation on update
```bash
curl -X PUT http://localhost:8080/products/1 \
  -H "Content-Type: application/json" \
  -d '{"name":"Updated","price":100,"stock":5,"category_id":9999}'
# Expected: 400 Bad Request (invalid category)
# Actual: 200 OK (BUG)
```

---

## Priority Fixes

| Priority | Issue | Impact | Effort |
|----------|-------|--------|--------|
| **HIGH** | Negative prices/stock | Data integrity | Low |
| **HIGH** | Non-existent category_id in product | Data integrity | Low |
| **MEDIUM** | Whitespace-only names | UX/Data quality | Low |
| **MEDIUM** | Missing field validation | Data integrity | Low |
| **MEDIUM** | Very long strings | Performance | Low |
| **LOW** | Duplicate names | Documentation | None |
| **LOW** | Special characters | Security (already safe) | Low |
| **LOW** | Concurrent updates | Consistency | High |

---

## Recommendations

1. ✅ **Add input validation** for negative/zero prices and whitespace
2. ✅ **Add FK validation** when creating/updating products
3. ✅ **Add length limits** to name/description fields
4. ⚠️ **Document** expected behavior for duplicates and NULL values
5. 🚀 **Consider** optimistic locking for concurrent updates (future enhancement)
