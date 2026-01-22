# Category CRUD API

A simple REST API for managing categories with in-memory storage built with Go.

![CI Tests](https://github.com/YOUR_USERNAME/YOUR_REPO/workflows/CI%20Tests/badge.svg)

## Features

- ✅ Full CRUD operations for categories
- ✅ In-memory storage (no database required)
- ✅ Thread-safe with RWMutex
- ✅ RESTful API design
- ✅ JSON request/response
- ✅ Automated CI tests

## Getting Started

### Prerequisites

- Go 1.22 or higher

### Installation

1. Clone the repository
```bash
git clone https://github.com/YOUR_USERNAME/YOUR_REPO.git
cd YOUR_REPO
```

2. Run the server
```bash
go run main.go
```

The server will start on port 8080 (or the port specified in `PORT` environment variable).

### Build

```bash
go build -o api-server main.go
./api-server
```

## API Endpoints

### Base URL
```
http://localhost:8080
```

### Health Check
```bash
curl http://localhost:8080/health
```

**Response:**
```
OK
```

### Version
```bash
curl http://localhost:8080/version
```

**Response:**
```
Commit: unknown
```

---

## Category Endpoints

### 1. Get All Categories

**Endpoint:** `GET /categories`

**Request:**
```bash
curl http://localhost:8080/categories
```

**Response:**
```json
[
  {
    "id": 1,
    "name": "Electronics",
    "description": "Electronic devices and gadgets"
  },
  {
    "id": 2,
    "name": "Books",
    "description": "Physical and digital books"
  }
]
```

---

### 2. Get Category by ID

**Endpoint:** `GET /categories/{id}`

**Request:**
```bash
curl http://localhost:8080/categories/1
```

**Response (Success - 200):**
```json
{
  "id": 1,
  "name": "Electronics",
  "description": "Electronic devices and gadgets"
}
```

**Response (Not Found - 404):**
```
Category not found
```

---

### 3. Create Category

**Endpoint:** `POST /categories`

**Request:**
```bash
curl -X POST http://localhost:8080/categories \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Electronics",
    "description": "Electronic devices and gadgets"
  }'
```

**Response (Success - 201):**
```json
{
  "id": 1,
  "name": "Electronics",
  "description": "Electronic devices and gadgets"
}
```

**Response (Bad Request - 400):**
```
Name is required
```

---

### 4. Update Category

**Endpoint:** `PUT /categories/{id}`

**Request:**
```bash
curl -X PUT http://localhost:8080/categories/1 \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Electronics and Tech",
    "description": "Electronic devices, gadgets, and technology products"
  }'
```

**Response (Success - 200):**
```json
{
  "id": 1,
  "name": "Electronics and Tech",
  "description": "Electronic devices, gadgets, and technology products"
}
```

**Response (Not Found - 404):**
```
Category not found
```

---

### 5. Delete Category

**Endpoint:** `DELETE /categories/{id}`

**Request:**
```bash
curl -X DELETE http://localhost:8080/categories/1
```

**Response (Success - 204):**
```
(No content)
```

**Response (Not Found - 404):**
```
Category not found
```

---

## Quick Testing Examples

### Complete Workflow

```bash
# 1. Create a category
curl -X POST http://localhost:8080/categories \
  -H "Content-Type: application/json" \
  -d '{"name":"Electronics","description":"Electronic devices"}'

# 2. Create another category
curl -X POST http://localhost:8080/categories \
  -H "Content-Type: application/json" \
  -d '{"name":"Books","description":"Physical and digital books"}'

# 3. List all categories
curl http://localhost:8080/categories

# 4. Get specific category
curl http://localhost:8080/categories/1

# 5. Update category
curl -X PUT http://localhost:8080/categories/1 \
  -H "Content-Type: application/json" \
  -d '{"name":"Electronics and Tech","description":"Updated description"}'

# 6. Delete category
curl -X DELETE http://localhost:8080/categories/2

# 7. Verify deletion
curl http://localhost:8080/categories
```

### Using jq for Pretty Output

If you have `jq` installed, you can format the JSON output:

```bash
curl http://localhost:8080/categories | jq
```

---

## Error Handling

The API returns appropriate HTTP status codes:

- `200 OK` - Request successful
- `201 Created` - Resource created successfully
- `204 No Content` - Resource deleted successfully
- `400 Bad Request` - Invalid request (missing required fields, invalid ID format)
- `404 Not Found` - Resource not found
- `500 Internal Server Error` - Server error

---

## Data Model

### Category

| Field       | Type   | Required | Description           |
|-------------|--------|----------|-----------------------|
| id          | int    | Auto     | Unique identifier     |
| name        | string | Yes      | Category name         |
| description | string | No       | Category description  |

---

## Configuration

### Environment Variables

- `PORT` - Server port (default: 8080)
- `GitCommit` - Build-time git commit hash

Example:
```bash
PORT=3000 go run main.go
```

---

## Development

### Running Tests

The project includes automated tests via GitHub Actions. To run manually:

```bash
# Start the server
go run main.go

# In another terminal, run the test script
chmod +x test_api.sh
./test_api.sh
```

### Project Structure

```
.
├── .github/
│   └── workflows/
│       └── ci.yml          # GitHub Actions CI configuration
├── main.go                 # Main application code
└── README.md              # This file
```

---

## Technical Details

- **Storage:** In-memory HashMap (`map[int]Category`)
- **Concurrency:** Thread-safe using `sync.RWMutex`
- **ID Generation:** Auto-incrementing integer
- **Time Complexity:** O(1) for all operations

---

## License

This project is open source and available under the MIT License.

---

## Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

## Contact

For questions or issues, please open an issue on GitHub.

---

**Happy Coding! 🚀**
