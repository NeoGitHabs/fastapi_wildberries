# 🛒 E-Commerce Marketplace API

> Production-ready REST API for a multi-vendor marketplace —
> membership tiers, cart management, favorites, reviews, and social auth.

[![Python](https://img.shields.io/badge/Python-3.11-blue)]()
[![FastAPI](https://img.shields.io/badge/FastAPI-async-teal)]()
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15-blue)]()
[![SQLAlchemy](https://img.shields.io/badge/SQLAlchemy-2.x-red)]()
[![License: MIT](https://img.shields.io/badge/License-MIT-green)]()

---

## Problem

Marketplaces need a structured product and order API to handle seller
listings, buyer carts, and review signals at scale. Without membership
tiers and persistent cart logic, platforms lose retention revenue and
repeat-purchase incentives.

---

## What's Built

- **JWT auth** — register / login / logout / refresh;
  refresh tokens stored in DB, deleted on logout
- **Auto cart on register** — `Cart` created atomically at registration,
  no extra client call needed
- **Cart** — add / remove items with duplicate guard per user
- **Favorites** — add / remove products with duplicate guard per user
- **Product catalog** — full CRUD; article number, video, type flag,
  price, category FK, owner FK
- **Reviews** — star rating + comment per product, full CRUD
- **Membership tiers** — gold / silver / bronze / simple on user profile
- **OAuth2** — GitHub and Google via authlib
- **Admin panel** — sqladmin with search + sort on all 4 entities

---

## Tech Stack

| Category   | Technology                              |
|------------|-----------------------------------------|
| Language   | Python 3.11                             |
| Framework  | FastAPI, Uvicorn (ASGI)                 |
| ORM        | SQLAlchemy 2.x (Mapped / mapped_column) |
| Validation | Pydantic v2                             |
| Auth       | python-jose (JWT), passlib (bcrypt)     |
| OAuth2     | authlib (GitHub, Google)                |
| Database   | PostgreSQL                              |
| Admin      | sqladmin (search, sort, filter)         |
| Config     | python-dotenv                           |

---

## Architecture
Client → FastAPI (ASGI / Uvicorn)
↕
APIRouter modules:
auth · product · category · cart
favorite · review · social_auth
↕
SQLAlchemy 2.x ORM → PostgreSQL
↕
sqladmin (admin panel)

Each domain is a separate `APIRouter` module.
Models use SQLAlchemy 2.x `Mapped` typed columns.
Pydantic Create / Get schema split keeps input and output contracts explicit.
Business guards (duplicate cart items, missing products) enforced
inside each router.

---

## Key Decisions

**Auto cart creation at registration**
`Cart` is inserted in the same `/auth/register` handler right after
user commit — every user has a cart from day one, client setup
goes from 2 calls to 1.

**DB-persisted refresh tokens**
Refresh tokens stored in `RefreshToken` table — logout deletes the
record, refresh validates against DB. Immediate revocation, no Redis needed.

**Duplicate guards at API layer**
Cart and favorites check for existing items before insert and return
`400` with a clear message — prevents silent duplicates without
unique constraints at the DB level.

---

## Quick Start

```bash
git clone https://github.com/your-username/fastapi-wildberries
cd fastapi-wildberries
cp .env.example .env        # SECRET_KEY, DB_URL, OAuth keys
pip install -r requirements.txt
```

```bash
# Create tables
python -c "from fastapi_wildberries.db.database import Base, engine; \
           Base.metadata.create_all(engine)"
```

```bash
uvicorn fastapi_wildberries.main:online_store --reload
# Swagger UI → http://localhost:8000/docs
# Admin UI   → http://localhost:8000/admin
```

---

## Demo

**Add item to cart:**
```bash
curl -X POST "http://localhost:8000/cart/?user_id=1" \
  -H "Content-Type: application/json" \
  -d '{"product_id": 3, "quantity": 2}'
```
```json
{"product_id": 3, "quantity": 2}
```

**Get cart:**
```bash
curl "http://localhost:8000/cart/?user_id=1"
```
```json
{
  "id": 1,
  "user_id": 1,
  "cart_item": [
    {"id": 5, "product_id": 3, "quantity": 2}
  ]
}
```

---

## Endpoints Overview

| Method | Endpoint              | Description           |
|--------|-----------------------|-----------------------|
| POST   | /auth/register        | Register + create cart|
| POST   | /auth/login           | Login, get tokens     |
| POST   | /auth/logout          | Logout, delete token  |
| POST   | /auth/refresh         | Refresh access token  |
| GET    | /oauth/github         | GitHub OAuth2         |
| GET    | /oauth/google         | Google OAuth2         |
| CRUD   | /product/             | Product management    |
| CRUD   | /category/            | Categories            |
| GET    | /cart/                | Get user cart         |
| POST   | /cart/                | Add item to cart      |
| DELETE | /cart/{product_id}    | Remove from cart      |
| GET    | /favorite/            | Get favorites         |
| POST   | /favorite/            | Add to favorites      |
| DELETE | /favorite/{product_id}| Remove from favorites |
| CRUD   | /review/              | Reviews               |

---

## Project Structure
```
fastapi_wildberries/
├── .gitignore
├── readme.md
└── fastapi_wildberries/
    ├── admin/
    │   ├── __init__.py
    │   ├── setup.py
    │   └── views.py
    ├── api/
    │   ├── __init__.py
    │   ├── auth.py
    │   ├── cart.py
    │   ├── category.py
    │   ├── favorite.py
    │   ├── product.py
    │   ├── review.py
    │   └── social_auth.py
    ├── create_secret_key.py
    ├── db/
    │   ├── __init__.py
    │   ├── config.py
    │   ├── database.py
    │   ├── models.py
    │   └── schema.py
    ├── main.py
    └── requirements.txt
```
---
