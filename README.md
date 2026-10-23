# Jengo Hardware

PHP 8.0+ and MySQL storefront backend with catalogue, customer accounts, session cart, order checkout, and staff order management.

## Installation

1. Start Apache and MySQL in XAMPP.
2. Copy `.env.example` to `.env` and adjust the DB settings. The included `.env` is configured for a default local XAMPP MySQL (`root`, empty password); never deploy it as production configuration.
3. Import [`database/schema.sql`](./database/schema.sql) with phpMyAdmin or the MySQL command line, then import [`database/seed_products.sql`](./database/seed_products.sql) to add the starter catalogue. The seed script skips product names already in the catalogue, so it is safe to run again.
4. Browse to `http://localhost/jengo_hardware/`.
5. Before customer registration, visit `admin.php` and create the first administrator with a unique password of at least 10 characters. The one-time setup is disabled as soon as an account exists.
6. Sign in as administrator, review and update the starter product prices and quantities to match your real inventory, then create attendant accounts in the administrator page. Attendants can then sign in and open the order desk to record registered-customer and walk-in sales.

To open the local XAMPP app from another device on the same trusted Wi-Fi/LAN, use `http://<this-PC-LAN-IP>/jengo_hardware/`. Apache must be allowed through Windows Firewall on the **Private** network profile for inbound TCP port 80. Do not expose this development XAMPP server directly to the public internet; use HTTPS hosting for a public site. The LAN IP can change when the PC reconnects.

For an existing database, apply [`database/migrations/001_attendant_sales.sql`](./database/migrations/001_attendant_sales.sql) once before using the attendant sale form. New installations get the required guest customer and phone columns from the schema.

The PHP process needs PDO MySQL enabled and write access to `storage/logs/`. Runtime settings are loaded from environment variables first, then `.env`; environment variables take precedence. Set `APP_DEBUG=false` in shared or production environments. `.env` and application logs are excluded from version control and blocked from direct HTTP access.

## Production deployment

1. Choose PHP 8.0+ hosting with Apache 2.4, HTTPS, PDO MySQL, and writable `storage/logs/`. Enable Apache `.htaccess` overrides for access control and response headers.
2. Upload the application over SFTP or deploy from a private Git repository. Do not publish the local `.env`; create a new production `.env` on the server with the host-provided database name/user/password, `APP_ENV=production`, `APP_DEBUG=false`, and the canonical HTTPS `APP_URL`.
3. Create the production database and restricted database user in the hosting control panel. Import `database/schema.sql`; for an older database, take a backup first and apply each unapplied migration exactly once.
4. Ensure the Apache/PHP process can write to `storage/logs/`. Test that `.env`, `/database/schema.sql`, `/includes/functions.php`, and `/config/config.php` are denied over HTTP before public launch.
5. Create the administrator, add real products and inventory, create staff accounts, and test registration, login, order placement, stock deduction, and staff sales on the production URL.
6. Configure automatic database backups and monitor application/Apache logs. Do not enable online payment flags until verified provider initiation, signed callback/webhook handling, and reconciliation have been implemented.

Tailwind utility classes are generated in the browser by the Tailwind CDN, so pages using those classes need internet access. The bundled CSS keeps the main visual design usable if the CDN is unavailable.

## Application structure

The five customer-facing frontend pages are separated from their backend route/controller files:

| Frontend view | Backend route/controller | Purpose |
| --- | --- | --- |
| `frontend/views/home.php` | `index.php` | Storefront landing page and featured products |
| `frontend/views/shop.php` | `shop.php` | Product search and catalogue |
| `frontend/views/product.php` | `product.php` | Product details and add-to-cart form |
| `frontend/views/cart.php` | `cart.php` | Session cart and order summary |
| `frontend/views/checkout.php` | `checkout.php` | Delivery form and checkout summary |

`frontend/views/partials/` contains the shared HTML header and footer; `frontend/views/not-found.php` is the shared product-not-found state. Backend route files validate access, load database records, and pass data into views through `render_view()`. State-changing forms submit to `actions.php`, which uses the backend services and database helpers. The view directory is blocked from direct HTTP access and is rendered only through its route/controller.

### Backend services and static frontend assets

- `config/config.php`: environment loading and application/database/payment settings.
- `includes/init.php`: common bootstrap, session cookie settings, and logging.
- `includes/functions.php`: PDO connection, escaping, authorization, CSRF, cart, view rendering, and payment configuration helpers.
- `includes/layout.php`: backend-to-frontend shared layout adapter.
- `actions.php`: POST-only application action endpoint, including attendant sales; `action.php` remains as a compatibility endpoint.
- `attendant.php`: records registered-customer or walk-in sales, multiple product lines, phone and sale location, and verified payment status.
- `auth.php`, `customer.php`, `dashboard.php`: sign-in/registration, order history, and role routing.
- `payments/mpesa.php`, `payments/paystack.php`: payment availability pages.
- `database/schema.sql`: MySQL schema.
- `assets/app.css`, `assets/app.js`, `assets/images/products/`: static assets and product image location. Selected product categories use locally stored Unsplash stock photos; unmatched items retain the original SVG illustrations.
- `storage/logs/`: application logs.

The frontend views use semantic HTML and Tailwind CSS via CDN, with custom responsive styles in `assets/app.css` and small vanilla-JavaScript interactions in `assets/app.js`. Product names are matched to suitable local stock photos where available, with the original SVG illustrations as category-specific or generic fallbacks. Product images are representative and may not show the exact brand, colour, or model sold.

### Product photo sources

The locally stored photographs below were sourced from Unsplash and may be used under the [Unsplash License](https://unsplash.com/license). The site does not fetch these images from Unsplash when it loads; deploy the files in `assets/images/products/` with the application.

| Local image | Unsplash photo |
| --- | --- |
| `assorted-tools.jpg` | [1530124566582-a618bc2615dc](https://images.unsplash.com/photo-1530124566582-a618bc2615dc) |
| `building-site.jpg` | [1504307651254-35680f356dfd](https://images.unsplash.com/photo-1504307651254-35680f356dfd) |
| `fasteners-tools.jpg` | [1581244277943-fe4a9c777189](https://images.unsplash.com/photo-1581244277943-fe4a9c777189) |
| `hammer.jpg` | [1586864387967-d02ef85d93e8](https://images.unsplash.com/photo-1586864387967-d02ef85d93e8) |
| `hero-workshop.jpg` | [1530124566582-a618bc2615dc](https://images.unsplash.com/photo-1530124566582-a618bc2615dc) |
| `nails-tools.jpg` | [1581783898377-1c85bf937427](https://images.unsplash.com/photo-1581783898377-1c85bf937427) |
| `paint-roller.jpg` | [1562259949-e8e7689d7828](https://images.unsplash.com/photo-1562259949-e8e7689d7828) |
| `power-drill.jpg` | [1504148455328-c376907d081c](https://images.unsplash.com/photo-1504148455328-c376907d081c) |
| `safety-worker.jpg` | [1589939705384-5185137a7f0f](https://images.unsplash.com/photo-1589939705384-5185137a7f0f) |

## Payments

Cash on delivery is the only active checkout method. M-Pesa and Paystack are deliberately disabled by default. Setting credentials or enabling a flag does not initiate transactions: provider request signing, order-to-transaction persistence, idempotent webhook verification, and reconciliation must be implemented and tested before enabling real payments. No credentials should be committed.
