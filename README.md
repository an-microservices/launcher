# Microservices Architecture with NestJS

A microservices-based application built with NestJS, featuring a client gateway and multiple microservices communicating through NATS messaging system.

## Architecture

This project implements a microservices architecture with the following components:

- `client-gateway` — HTTP API gateway (for clients / UI)
- `products-ms` — Products microservice (SQLite / Prisma)
- `orders-ms` — Orders microservice (Postgres / Prisma)
- `auth-ms` — Authentication microservice (MongoDB / Prisma)
- `payments-ms` — Payments microservice (Stripe integration)

Infrastructure services (via `docker-compose.yml` in repo root):

- `nats-server` — NATS message broker (ports: 4222 client, 8222 monitoring)
- `orders-db` / `postgres` — Postgres DB used by `orders-ms`
- `auth-db` / `mongo` — MongoDB for `auth-ms` (can be configured as replica set)

## Quick start (recommended)

This starts all services using the root `docker-compose.yml`.

```powershell
# start everything (builds images if necessary)
docker-compose up --build

# or run in background
docker-compose up -d --build

# follow logs
docker-compose logs -f
```

After startup:
- Client Gateway: http://localhost:3000 (unless overridden in `.env`)
- NATS monitoring: http://localhost:8222

If you need to reset volumes and start clean:

```powershell
docker-compose down -v
docker-compose up --build
```

This will start:
- NATS Server on port `8222`
- Client Gateway on port `3000`
- Products Microservice on port `3001`
- Orders Microservice on port `3002`
- PostgreSQL Database on port `5432`

### 4. Running Services Individually (Development)

If you want to run services individually without Docker:

#### Start NATS Server
```bash
docker run -d -p 4222:4222 -p 8222:8222 nats:latest
```

#### Client Gateway
```bash
cd client-gateway
npm install
npm run start:dev
```

#### Products Microservice
```bash
cd products-ms
npm install
npm run start:dev
```

#### Orders Microservice
```bash
cd orders-ms
npm install
npm run start:dev
```

## Microservices Details

### Client Gateway

- **Port**: 3000
- **Type**: HTTP REST API
- **Responsibilities**:
  - Receives HTTP requests from clients
  - Routes requests to appropriate microservices via NATS
  - Aggregates responses and returns to client

**Endpoints**:
- `/products` - Product management endpoints
- `/orders` - Order management endpoints

### Products Microservice

- **Port**: 3001
- **Database**: SQLite (file-based)
- **Communication**: NATS
- **Responsibilities**:
  - CRUD operations for products
  - Product catalog management
  - Inventory tracking

### Orders Microservice

- **Port**: 3002
- **Database**: PostgreSQL
- **Communication**: NATS
- **Responsibilities**:
  - Order creation and management
  - Order status tracking
  - Integration with Products MS

## Database

### PostgreSQL (Orders)

- **Host**: `localhost` (or `orders-db` in Docker)
- **Port**: 5432
- **Database**: ordersdb
- **User**: postgres
- **Password**: 12345

### SQLite (Products)

- **Location**: `./products-ms/dev.db`

## Development

### Project Structure

```
microservices/
├── client-gateway/       # API Gateway
│   ├── src/
│   │   ├── products/    # Products module
│   │   ├── orders/      # Orders module
│   │   └── transports/  # NATS configuration
│   └── dockerfile
├── products-ms/         # Products Microservice
│   ├── src/
│   │   └── products/
│   ├── prisma/
│   └── dockerfile
├── orders-ms/           # Orders Microservice
│   ├── src/
│   │   └── orders/
│   ├── prisma/
│   └── dockerfile
└── docker-compose.yml
```

## API Documentation

### Products

- `GET /products` - List all products
- `GET /products/:id` - Get product by ID
- `POST /products` - Create new product
- `PATCH /products/:id` - Update product
- `DELETE /products/:id` - Delete product

### Orders

- `GET /orders` - List all orders
- `GET /orders/:id` - Get order by ID
- `POST /orders` - Create new order
- `PATCH /orders/:id` - Update order status
- `DELETE /orders/:id` - Delete order

## Docker Commands

```bash
# Build and start all services
docker-compose up --build

# Start services in detached mode
docker-compose up -d

# Stop all services
docker-compose down

# View logs
docker-compose logs -f

# Rebuild specific service
docker-compose up --build <service-name>
```

## Technologies Used

- **NestJS** - Progressive Node.js framework
- **NATS** - Message broker for microservices communication
- **Prisma** - Next-generation ORM
- **PostgreSQL** - Relational database for Orders
- **SQLite** - Lightweight database for Products
- **Docker** - Containerization
- **TypeScript** - Type-safe JavaScript
