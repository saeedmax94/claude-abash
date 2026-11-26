# Nubia app: Laravel + React + MySQL

```
backend-overlay/   files to copy over a fresh Laravel project
frontend/          React (Vite) app, RTL Arabic
```

## 1. Database (MySQL)

```sql
CREATE DATABASE nubia CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

## 2. Backend (Laravel 11 or newer, PHP 8.2+, Composer)

```bash
composer create-project laravel/laravel backend
cd backend
php artisan install:api          # installs Sanctum and routes/api.php
cp -r ../backend-overlay/* .     # copies models, controllers, migrations, routes, config
```

Edit `backend/.env`:

```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=nubia
DB_USERNAME=root
DB_PASSWORD=

FRONTEND_URL=http://localhost:5173

# Development: reset emails are written to storage/logs/laravel.log
MAIL_MAILER=log
# Production: use a real SMTP service
# MAIL_MAILER=smtp
# MAIL_HOST=...
# MAIL_PORT=587
# MAIL_USERNAME=...
# MAIL_PASSWORD=...
# MAIL_FROM_ADDRESS=no-reply@yourdomain.com
```

Then:

```bash
php artisan migrate --seed
php artisan serve                # http://localhost:8000
```

First account: username `admin`, password `ChangeMe-123`. Change it, and change the email in `database/seeders/DatabaseSeeder.php` to your real one before seeding, because the reset link is sent to that address.

## 3. Frontend

```bash
cd frontend
cp .env.example .env             # VITE_API_URL=http://localhost:8000/api
npm install
npm run dev                      # http://localhost:5173
```

## How the screens work

- `/login`: username and password, a login button, and a link "تغيير كلمة السر".
- `/forgot-password`: asks for the email and Laravel sends a link.
- `/reset-password?token=...&email=...`: the page the email link opens, where the new password is set.
- After login: news (home), history, pyramids, songs, languages, dictionary, membership.
- Dictionary: add an Arabic word with its Nubian equivalent and the language, search, and delete your own entries.

## API

| Method | Path | Auth |
|---|---|---|
| POST | /api/login, /api/forgot-password, /api/reset-password | no |
| POST | /api/logout, GET /api/me | yes |
| GET | /api/articles/{news,history,pyramids}, /api/songs, /api/languages | yes |
| GET/POST/DELETE | /api/words | yes |
| GET/POST | /api/membership | yes |

## Adding content

There is no admin screen yet. Add articles, songs and languages with `php artisan tinker` or by editing the seeder, for example:

```php
App\Models\Song::create(['title' => '...', 'artist' => '...', 'language' => 'Nobiin', 'url' => 'https://...']);
```

Add users the same way: `App\Models\User::create(['name'=>'...','username'=>'...','email'=>'...','password'=>'...']);`

## Before going live

- Serve over HTTPS, set `APP_ENV=production` and `APP_DEBUG=false`.
- Use real SMTP, and set `FRONTEND_URL` to the real site address.
- The seeded article text is placeholder. Replace it with verified content.

## Mobile app later

The API is plain JSON with Bearer tokens, so a React Native (Expo) app can use the same backend. Move the `api.js` and `auth.jsx` logic over and replace `localStorage` with secure storage.
