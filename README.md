# PSU Attendance & Presence Verification System

A Laravel application for campus attendance, event management, student records, geofence presence checks, reporting, activity logs, and event e-certificates. It also exposes REST and legacy PHP-compatible endpoints for the mobile app.

## Requirements

- PHP 8.2 or newer with the PHP extensions required by Laravel and GD enabled for e-certificate rendering
- Composer 2
- MySQL/MariaDB or SQLite
- Node.js and npm to build frontend assets
- A web server configured to serve the Laravel `public` directory

## Local setup

```bash
composer install
copy .env.example .env
php artisan key:generate
```

Configure `.env` for the local database, then set a private `INITIAL_SUPERADMIN_PASSWORD` (at least 12 characters) before seeding:

```bash
php artisan migrate --seed
npm install
npm run build
php artisan serve
```

The initial superadmin username and email can be set with `INITIAL_SUPERADMIN_USERNAME` and `INITIAL_SUPERADMIN_EMAIL`. Keep the initial password private; do not commit `.env` or share deployment credentials.

## Production deployment checklist

1. **Prepare the host.** Use a supported PHP version, install Composer dependencies, enable GD and the required PHP extensions, and configure MySQL/MariaDB or SQLite. Set the site's document root to this project's `public` directory. Never expose the project root, `.env`, `vendor`, or `storage` directly to the web.
2. **Configure the production environment.** Create a production `.env` with:
   - `APP_ENV=production`
   - `APP_DEBUG=false`
   - `APP_URL=https://your-domain`
   - a unique `APP_KEY`; preserve the existing key when moving an already-running installation, and only generate one for a fresh installation
   - production database credentials and `LOG_LEVEL=warning`
   - `SESSION_SECURE_COOKIE=true` when HTTPS is enabled
   - `FILESYSTEM_DISK=public` for public-disk uploads
3. **Install and build.** Run `composer install --no-dev --optimize-autoloader` and `npm ci && npm run build`. Do not upload a development `.env` or reuse the example key.
4. **Migrate carefully.** Back up the database first, check `php artisan migrate:status`, then run `php artisan migrate --force` only if migrations are pending.
5. **Create the first administrator only for a fresh database.** Set `INITIAL_SUPERADMIN_PASSWORD` to a unique password of at least 12 characters in the server environment, then run `php artisan db:seed --force`. The production seeder refuses to create a fresh superadmin without this password. Clear the bootstrap password from the deployment environment after seeding. Never seed a populated database as part of routine deployment. Existing accounts are not reset by the seeder; change any default or shared passwords before opening the site.
6. **Set writable directories.** Ensure the web-server user can write to `storage` and `bootstrap/cache`. Create the public storage link with `php artisan storage:link` if the host supports symlinks.
7. **Cache production configuration.** After environment values are final, run `php artisan optimize:clear` followed by `php artisan config:cache`, `php artisan route:cache`, and `php artisan view:cache`.
8. **Verify before launch.** Check `/up`, sign in with the new administrator, open the dashboard and scanner, test a check-in/check-out, assign an event designation, verify student CSV downloads with year/block filters, confirm attendance reports, upload and preview an e-certificate, and verify a certificate's public page/download. Confirm backups and HTTPS are working.

### Windows / XAMPP with an existing database

- Before changing an existing database, back it up and check `php artisan migrate:status`. Do not run `migrate:fresh`, `migrate:refresh`, `db:wipe`, or `db:seed` against a populated database.
- The project is currently under `C:\xampp\htdocs`. Do not expose that project root as the website: configure an Apache virtual host whose `DocumentRoot` points only to the project's `public` directory, or move the application outside `htdocs` and point the virtual host to its `public` directory. Enable Apache `mod_rewrite`, and confirm GD is enabled for the PHP version used by Apache (not just CLI PHP).
- Before public access, set the server `.env` to production mode, disable debug output, set the real HTTPS `APP_URL`, preserve the current `APP_KEY` if this is the same installation, and review database and file permissions. Do not expose phpMyAdmin or the MySQL port publicly; use HTTPS and a firewall.
- Do not use `php artisan serve` as an internet-facing production server. Use Apache with the configured virtual host.

For other shared hosting, keep the Laravel application outside the public web directory and point the domain document root to its `public` directory. If the provider cannot change the document root, ask the provider for its supported Laravel deployment layout rather than exposing the project root.

## Tests

```bash
php artisan test
```

The test database is configured in `phpunit.xml`; configure an isolated test database before running the suite in any environment containing live records.
