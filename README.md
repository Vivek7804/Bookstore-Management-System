# Bookstore Management API

A Django REST Framework API for managing a bookstore catalogue and customer orders. It supports JWT authentication, role-based access control, searchable book listings, inventory management, and stock-safe order placement.

## Features

- Customer and administrator registration with email/password login
- JWT access and refresh tokens
- Public, paginated book catalogue with search, filtering, and ordering
- Administrator-only book creation, updates, and deletion
- Customer order placement with automatic stock deduction
- Order history scoped to the signed-in customer; administrators can view all orders
- Administrator-only order status updates
- OpenAPI schema and Swagger UI
- Django admin integration for users, books, and orders

## Tech stack

- Python and Django 4.2
- Django REST Framework
- Simple JWT for token authentication
- django-filter for catalogue and order filtering
- drf-spectacular for API documentation
- SQLite (default development database)

## Project structure

```text
accounts/     User model, registration, and JWT login
books/        Book catalogue and administrator inventory operations
orders/       Orders, line items, stock handling, and order-status updates
config/       Django settings and root URL configuration
postman/      Importable Postman collection
```

## Setup

1. Create and activate a virtual environment.

   ```bash
   python -m venv .venv
   source .venv/bin/activate
   ```

   On Windows, activate it with `.venv\Scripts\activate`.

2. Install the required packages.

   ```bash
   pip install "Django>=4.2,<5.0" djangorestframework djangorestframework-simplejwt django-filter drf-spectacular
   ```

3. Apply migrations.

   ```bash
   python manage.py migrate
   ```

4. Optionally create a Django superuser for `/admin/`.

   ```bash
   python manage.py createsuperuser
   ```

5. Start the development server.

   ```bash
   python manage.py runserver
   ```

The API is then available at `http://127.0.0.1:8000/`.

## Authentication and roles

The API uses `Authorization: Bearer <access-token>` for protected endpoints.

| Role | Permissions |
| --- | --- |
| Customer | Browse books, place orders, and view their own orders |
| Admin | Manage books, view all orders, and update order status |
| Django superuser | Has the same API administrative access and can access `/admin/` |

Register an account with `POST /api/register`, then use `POST /api/login` to receive an `access` token and a `refresh` token. The registration endpoint accepts both `customer` and `admin` roles; restrict administrator account creation appropriately before deploying to production.

## API reference

Interactive documentation is available while the server is running:

- Swagger UI: [`/api/docs/`](http://127.0.0.1:8000/api/docs/)
- OpenAPI schema: [`/api/schema/`](http://127.0.0.1:8000/api/schema/)

All API routes below omit a trailing slash.

### Accounts

| Method | Endpoint | Access | Description |
| --- | --- | --- | --- |
| POST | `/api/register` | Public | Create a user account |
| POST | `/api/login` | Public | Obtain JWT access and refresh tokens using email and password |

Registration example:

```json
{
  "name": "Ada Lovelace",
  "email": "ada@example.com",
  "password": "strong-password-123",
  "role": "customer"
}
```

Login example:

```json
{
  "email": "ada@example.com",
  "password": "strong-password-123"
}
```

### Books

| Method | Endpoint | Access | Description |
| --- | --- | --- | --- |
| GET | `/api/books` | Public | List books (paginated) |
| GET | `/api/books/{id}` | Public | Retrieve one book |
| POST | `/api/books` | Admin | Create a book |
| PUT | `/api/books/{id}` | Admin | Replace a book |
| PATCH | `/api/books/{id}` | Admin | Partially update a book |
| DELETE | `/api/books/{id}` | Admin | Delete a book |

Catalogue query parameters:

- `search` searches book titles and authors.
- `genre` filters by exact genre.
- `ordering` accepts `title`, `author`, `price`, or `created_at`; prefix with `-` for descending order.
- `page` selects a paginated result page (10 books per page by default).

Example: `/api/books?search=django&genre=Technology&ordering=price`

### Orders

| Method | Endpoint | Access | Description |
| --- | --- | --- | --- |
| GET | `/api/orders` | Authenticated | List orders (customers see only their own; admins see all) |
| GET | `/api/orders/{id}` | Owner or admin | Retrieve an order |
| POST | `/api/orders` | Customer | Place an order |
| PUT | `/api/orders/{id}/status` | Admin | Change order status |

Order list query parameters: `status`, `payment_status`, and, for administrators, `user` (a user ID).

Place an order:

```json
{
  "items": [
    { "book": 1, "quantity": 2 }
  ],
  "payment_status": "pending"
}
```

Each book can appear once per order. The API validates availability inside a database transaction, stores the book's current price in the order item, and deducts the requested quantity from stock. An order is rejected when stock is insufficient.

Valid order statuses are `pending`, `shipped`, and `delivered`. Payment status is `pending` or `paid`.

## Postman collection

Import [`postman/Bookstore_Management_API.postman_collection.json`](postman/Bookstore_Management_API.postman_collection.json) into Postman. Set its `accessToken`, `bookId`, and `orderId` variables after registering/logging in and creating a book.

## Tests

Run the automated API tests with:

```bash
python manage.py test
```

The test suite covers password hashing and JWT login, public catalogue search, admin-only book creation, stock deduction, order-status updates, and insufficient-stock rejection.

## Development notes

- The default database is `db.sqlite3`; do not use Django's development settings or bundled secret key in production.
- The root route redirects to Swagger UI.
- API pagination defaults to 10 results per page.
# Bookstore-Management-System
