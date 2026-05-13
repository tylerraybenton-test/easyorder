# Relay

A TypeScript/Node.js service for sending and processing orders.

---

## Prerequisites

- Node.js v24.15.0
- PostgreSQL

---

## Setup

### 1. Install Dependencies

```bash
npm install
```

### 2. Configure Database

Open `config/config.json` and fill in your PostgreSQL credentials:

```json
{
  "development": {
    "username": "your_db_user",
    "password": "your_db_password",
    "database": "your_db_name",
    "host": "127.0.0.1",
    "dialect": "postgres"
  }
}
```

### 3. Configure Environment Variables

Rename `.env.example` to `.env`:

```bash
mv .env.example .env
```

Then open `.env` and set the required environment variable(s).

### 4. Run Migrations

```bash
npm run db:migrate
```

### 5. Seed the Database

```bash
npm run db:seed
```

---

## Running the App

```bash
npm run dev
```

The server will start at `http://localhost:3000`.

---

## API

### Send an Order

**`POST /order`**

**Request Body:**

```json
{
  "customer": {
    "email": "customer@example.com",
    "name": "Jane Doe"
  },
  "items": [
    { "productId": 1, "unitPrice": 29999, "quantity": 10 },
    { "productId": 3, "unitPrice": 44999, "quantity": 20 }
  ],
  "shipping": {
    "address": "123 Main St, Apt 4B, Austin, TX 78701"
  },
  "payment": {
    "card": {
      "number": "4111111111111111"
    }
  }
}
```

| Field | Type | Description |
|---|---|---|
| `customer.email` | `string` | Customer's email address |
| `customer.name` | `string` | Customer's full name |
| `items` | `array` | List of order line items |
| `items[].productId` | `number` | ID of the product |
| `items[].unitPrice` | `number` | Price per unit in cents |
| `items[].quantity` | `number` | Number of units |
| `shipping.address` | `string` | Full shipping address. If the address contains a recognized `City, ST` (e.g. `Austin, TX`), the order will be enriched with coordinates via `geocodeAddress.ts` (see supported cities below) |
| `payment.card.number` | `string` | Payment card number. Use `6666666666666666` or `9999999999999999` to simulate an invalid card |

#### Supported Cities for Geocoding

If `shipping.address` includes a city and state in the format `City, ST`, coordinates will be resolved automatically. The following locations are supported:

| City, State | Latitude | Longitude |
|---|---|---|
| New York, NY | 40.7128 | -74.0060 |
| Boston, MA | 42.3601 | -71.0589 |
| Philadelphia, PA | 39.9526 | -75.1652 |
| Baltimore, MD | 39.2904 | -76.6122 |
| Washington, DC | 38.9072 | -77.0369 |
| Miami, FL | 25.7617 | -80.1918 |
| Orlando, FL | 28.5383 | -81.3792 |
| Atlanta, GA | 33.7490 | -84.3880 |
| Charlotte, NC | 35.2271 | -80.8431 |
| Chicago, IL | 41.8781 | -87.6298 |
| Detroit, MI | 42.3314 | -83.0458 |
| Minneapolis, MN | 44.9778 | -93.2650 |
| St. Louis, MO | 38.6270 | -90.1994 |
| Cleveland, OH | 41.4993 | -81.6944 |
| Dallas, TX | 32.7767 | -96.7970 |
| Houston, TX | 29.7604 | -95.3698 |
| Austin, TX | 30.2672 | -97.7431 |
| Phoenix, AZ | 33.4484 | -112.0740 |
| Las Vegas, NV | 36.1699 | -115.1398 |
| Los Angeles, CA | 34.0522 | -118.2437 |
| San Francisco, CA | 37.7749 | -122.4194 |
| San Diego, CA | 32.7157 | -117.1611 |
| Seattle, WA | 47.6062 | -122.3321 |
| Portland, OR | 45.5051 | -122.6750 |
| Denver, CO | 39.7392 | -104.9903 |
| Salt Lake City, UT | 40.7608 | -111.8910 |

---

## Production Readiness (`POST /order`)

- **Request validation** — Zod schema rejects bad input before touching the DB
- **DB-level validation** — `enrichItems` re-verifies product prices inside the transaction to catch stale data
- **Atomic transaction** — entire order flow is one DB transaction; any failure = full rollback
- **Idempotent customer upsert** — `ON CONFLICT (email) DO UPDATE` prevents duplicate constraint errors
- **Typed error classes** — custom errors map to correct HTTP status codes (400, 409, 500)
- **Domain-organized services** — order services grouped under `src/services/orders/`