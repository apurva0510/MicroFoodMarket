# Micro Foods Market

A Docker Compose based microservice backend for a mini grocery store system built with Flask and SQLite.  
Services: **users**, **products**, **search**, **orders**, **logs**.

## Table of Contents
- Overview
- Services and Ports
- Tech Stack
- How It Works
- Setup and Run
- API Summary
- Example Requests
- Project Structure
- Notes

## Overview
This project implements a microservice architecture where each service owns its own SQLite database and exposes a small HTTP API. Services communicate over HTTP on a shared Docker network.

## Services and Ports
| Service | Container Name | Port | Database |
|---|---|---:|---|
| User Management | `user` | 9000 | `user.db` |
| Product Management | `products` | 9001 | `products.db` |
| Product Search | `search` | 9002 | `search.db` (optional) |
| Ordering | `orders` | 9003 | `ordering.db` (optional) |
| Logging | `logs` | 9004 | `logs.db` |

## Tech Stack
- Python + Flask
- SQLite (`sqlite3`)
- Docker + Docker Compose
- `requests` for service to service HTTP calls
- JWT authentication

## How It Works
- Users register and log in through the **user** service
- Login returns a JWT used to authorize requests to other services
- Employees can create and edit products via the **products** service
- The **search** service supports searching by product name or category and includes `last_mod` sourced from the **logs** service
- The **orders** service accepts an order list, validates products and quantities, and returns a total cost
- The **logs** service records successful actions and allows authorized viewing

## Setup and Run

### Prerequisites
- Docker
- Docker Compose

### Build and start
From the repository root (where `compose.yaml` is located):

```bash
docker compose build
docker compose up
```

### Stop
```bash
docker compose down
```

## API Summary

### User Service (9000)
- `POST /create_user`
  - Form params: `first_name`, `last_name`, `username`, `email_address`, `employee` (bool), `password`, `salt`
- `POST /login`
  - Form params: `username`, `password`
  - Returns: `{ "status": <code>, "jwt": "<token>" }`

### Products Service (9001)
- `POST /create_product` (employee only)
  - Form params: `name`, `price`, `category`
- `POST /edit_product` (employee only)
  - Form params: `name` plus one of `new_price` or `new_category`

### Search Service (9002)
- `GET /search`
  - Query param: exactly one of `product_name` or `category`
  - Returns: `{ "status": <code>, "data": [ ... ] }`

### Orders Service (9003)
- `POST /order`
  - JSON body: `{ "order": [ { "product": "<name>", "quantity": <int> }, ... ] }`
  - Returns: `{ "status": <code>, "cost": <total_or_NULL> }`

### Logs Service (9004)
- `GET /view_log` (authorized)
  - Query param: exactly one of `username` or `product`
  - Returns: `{ "status": <code>, "data": { "1": {...}, "2": {...} } }` or `"NULL"`

### Clear Endpoint (all services)
- `GET /clear`
  - Clears that service database and resets state

## Example Requests (cURL)

> Replace `$JWT` with the token returned from `/login`.

### Create a user
```bash
curl -X POST http://localhost:9000/create_user   -d "first_name=Jane"   -d "last_name=Doe"   -d "username=jdoe"   -d "email_address=jdoe@example.com"   -d "employee=true"   -d "password=pass123"   -d "salt=somesalt"
```

### Login
```bash
curl -X POST http://localhost:9000/login   -d "username=jdoe"   -d "password=pass123"
```

### Create a product (employee only)
```bash
curl -X POST http://localhost:9001/create_product   -H "Authorization: Bearer $JWT"   -d "name=milk"   -d "price=3.99"   -d "category=dairy"
```

### Edit a product (employee only)
```bash
curl -X POST http://localhost:9001/edit_product   -H "Authorization: Bearer $JWT"   -d "name=milk"   -d "new_price=4.25"
```

### Search by category
```bash
curl "http://localhost:9002/search?category=dairy"   -H "Authorization: Bearer $JWT"
```

### Place an order
```bash
curl -X POST http://localhost:9003/order   -H "Authorization: Bearer $JWT"   -H "Content-Type: application/json"   -d '{"order":[{"product":"milk","quantity":2}]}'
```

### View logs for a product (authorized)
```bash
curl "http://localhost:9004/view_log?product=milk"   -H "Authorization: Bearer $JWT"
```

### Clear all services
```bash
curl http://localhost:9000/clear
curl http://localhost:9001/clear
curl http://localhost:9002/clear
curl http://localhost:9003/clear
curl http://localhost:9004/clear
```

## Project Structure
```text
.
├── compose.yaml
├── Dockerfile.users
├── Dockerfile.products
├── Dockerfile.search
├── Dockerfile.order
├── Dockerfile.logs
├── users/
│   ├── app.py
│   └── schema.sql
├── products/
│   ├── app.py
│   └── schema.sql
├── search/
│   ├── app.py
│   └── schema.sql
├── orders/
│   ├── app.py
│   └── schema.sql
└── logs/
    ├── app.py
    └── schema.sql
```

## Notes
- Use parameterized SQL queries to avoid SQL injection
- JWT payload should only include the username
- Each service owns its database and initializes tables on startup (and via `/clear`)

## License
Copyright (c) 2026 . All rights reserved.
No permission is granted to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of this software without explicit written permission.
