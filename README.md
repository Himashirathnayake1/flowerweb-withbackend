# Flower Website

A full-stack flower shop application with a Next.js storefront and a Laravel REST API. The application supports flower browsing, category filtering, account registration and login, authenticated carts, and checkout orders.


## Technologies

### Frontend: `flower-shop`

- Next.js `16.3.0` with the App Router
- React `19.2.8`
- TypeScript `5`
- Tailwind CSS `4`
- `@tailwindcss/postcss` for Tailwind/PostCSS integration
- ESLint `9` with `eslint-config-next`
- Next/Image for optimized flower and brand images
- Browser `localStorage` for the bearer token and bookmark state
- REST/JSON API calls using the browser `fetch` API

Frontend configuration is in `next.config.ts`, `tsconfig.json`, `postcss.config.mjs`, and `app/globals.css`.

### Backend: `flower-shop-api`

- PHP `8.3`
- Laravel `13`
- Composer
- MySQL through PDO MySQL
- Eloquent ORM and Laravel migrations
- Laravel request validation
- Custom bearer-token authentication middleware
- JSON REST API
- CORS configuration for the Next.js development server

Backend development and testing tools include:

- PHPUnit `12`
- Laravel Pint
- Laravel Pail
- Laravel Tinker
- Faker
- Mockery
- Collision

The API does not use Laravel Sanctum or Passport. It stores a SHA-256 hash of a generated bearer token in the `users.api_token` column.

## Requirements

Install the following before developing:

- Node.js and npm
- PHP `8.3` or newer
- Composer
- MySQL Server
- Git

On Windows, make sure PHP includes the `pdo_mysql` extension. Verify it with:

```powershell
php -m | Select-String pdo_mysql
```

## Initial Setup

### 1. Configure MySQL

Create a database named `flower_shop` in MySQL:

```sql
CREATE DATABASE flower_shop;
```

The working local backend configuration uses:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=flower_shop
DB_USERNAME=root
DB_PASSWORD=
```

Copy the example environment file if needed, then update the database values in `flower-shop-api/.env`:

```powershell
cd D:\flower_website\flower-shop-api
Copy-Item .env.example .env
php artisan key:generate
```

`flower-shop-api/.env.example` contains Laravel's SQLite defaults. This project currently uses MySQL because the local PHP installation provides `pdo_mysql` but not `pdo_sqlite`.

### 2. Install backend dependencies and seed data

```powershell
cd D:\flower_website\flower-shop-api
composer install
php artisan migrate:fresh --seed
```

The seeder creates:

- A test user: `test@example.com` with password `password`
- Five flower categories
- Six flower products

Use `migrate:fresh --seed` only when resetting local development data. It deletes existing tables first.

### 3. Install frontend dependencies

```powershell
cd D:\flower_website\flower-shop
npm install
```

The frontend defaults to this API URL:

```text
http://127.0.0.1:8000/api
```

To override it, create `flower-shop/.env.local`:

```env
NEXT_PUBLIC_API_URL=http://127.0.0.1:8000/api
```

## Running the Application

Run the backend and frontend in separate terminals.

### Laravel API

```powershell
php D:\flower_website\flower-shop-api\artisan serve --host=127.0.0.1 --port=8000
```

The API is available at:

```text
http://127.0.0.1:8000
```

### Next.js frontend

```powershell
npm --prefix D:\flower_website\flower-shop run dev
```

The storefront is available at:

```text
http://localhost:3000
```

Keep both terminals open while developing. The Laravel command must be run with the `flower-shop-api` `artisan` path; running `php artisan serve` from `D:\flower_website` fails because there is no root-level `artisan` file.

## Application Features

### Flower catalog

Public endpoints provide categories and active flowers. The frontend normalizes Laravel's nested category response and decimal values before rendering cards.

### Authentication

1. Open `/auth`.
2. Register with a name, email, and password of at least eight characters, or log in.
3. The API returns a bearer token.
4. The frontend stores the token as `flower_token` in browser `localStorage`.
5. The Account page validates the token with `/api/auth/me`.
6. Logout revokes the token and removes it from local storage.

If a stale token is present during local development, clear it in the browser console:

```js
localStorage.removeItem("flower_token")
```

### Cart

Cart data belongs to the authenticated user. Users can add flowers, change quantities, remove items, and view the current total.

### Checkout

Checkout validates delivery details, calculates the total from the server-side flower prices, creates an order and order items in a database transaction, and clears the user's cart.

## API Endpoints

All routes are prefixed with `/api`.

### Public endpoints

| Method | Endpoint | Purpose |
| --- | --- | --- |
| `GET` | `/categories` | List categories with flower counts |
| `GET` | `/flowers` | List active flowers |
| `GET` | `/flowers/{flower}` | Show one flower |
| `POST` | `/auth/register` | Create an account and return a token |
| `POST` | `/auth/login` | Authenticate and return a token |

### Authenticated endpoints

Send the token as:

```http
Authorization: Bearer <token>
```

| Method | Endpoint | Purpose |
| --- | --- | --- |
| `GET` | `/auth/me` | Return the current user |
| `POST` | `/auth/logout` | Revoke the current token |
| `GET` | `/cart` | Return the current user's cart |
| `POST` | `/cart/items` | Add a flower; body includes `flower_id` and optional `quantity` |
| `PATCH` | `/cart/items/{item}` | Update an item quantity |
| `DELETE` | `/cart/items/{item}` | Remove a cart item |
| `POST` | `/checkout` | Create an order from the current cart |

Checkout request example:

```json
{
  "shipping_name": "Alex Flower",
  "shipping_phone": "555-0100",
  "shipping_address": "1 Flower Street"
}
```

## Development Workflow

1. Start MySQL.
2. Start Laravel on `127.0.0.1:8000`.
3. Start Next.js on port `3000`.
4. Change backend behavior in `flower-shop-api/app`, `routes`, or `database`.
5. Change storefront behavior in `flower-shop/app`, `components`, `lib`, `models`, or `data`.
6. Run the focused validation commands before committing changes.

### Frontend commands

```powershell
cd D:\flower_website\flower-shop
npm run dev
npm run lint
npx tsc --noEmit
npm run build
npm start
```

`npm start` serves the latest production build, so run `npm run build` first.

### Backend commands

```powershell
cd D:\flower_website\flower-shop-api
php artisan route:list --path=api
php artisan migrate:status
php artisan migrate:fresh --seed
php artisan test
```

Check PHP syntax for a changed file:

```powershell
php -l app/Http/Controllers/Api/AuthController.php
```

## Troubleshooting

### `Could not open input file: artisan`

You are in `D:\flower_website` instead of the Laravel project. Use:

```powershell
php D:\flower_website\flower-shop-api\artisan serve --host=127.0.0.1 --port=8000
```

### `Cannot connect to the API`

Confirm Laravel is running and that this URL responds:

```powershell
(Invoke-WebRequest -UseBasicParsing http://127.0.0.1:8000/api/flowers).StatusCode
```

Expected status: `200`.

### `could not find driver`

The configured database driver is not installed in PHP. This project uses MySQL and requires `pdo_mysql`:

```powershell
php -m | Select-String pdo_mysql
```

### Login appears to reset

Clear an old browser token, restart both development servers, and log in again:

```js
localStorage.removeItem("flower_token")
```

The Account page checks `/api/auth/me`; a token that is missing, revoked, or associated with a different database is treated as logged out.

## Production Notes

Before deploying:

- Set `APP_ENV=production` and `APP_DEBUG=false`.
- Use a strong `APP_KEY`.
- Configure production MySQL credentials through environment variables.
- Set `FRONTEND_URL` to the deployed frontend origin.
- Use HTTPS so bearer tokens are not sent over plain HTTP.
- Build the frontend with `npm run build` and serve it with `npm start`.
- Run Laravel migrations with `php artisan migrate --force`.
- Do not use `migrate:fresh` in production.
"# flowerweb-withbackend" 
