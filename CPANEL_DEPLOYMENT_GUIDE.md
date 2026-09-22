# cPanel Deployment Guide for Crew LMS / Loops Work

Target URL: `https://hrdemo.loopsintegrated.com/hrlms`  
Repository: `https://github.com/dilmith-loops/crew-lms.git`

---

## 1. Verify Domain & Document Root in cPanel
1. In cPanel, navigate to **Domains** (or **Subdomains**).
2. Check the document root for `hrdemo.loopsintegrated.com`.
   - Typically, it is either `public_html/hrdemo` or `public_html`.
   - The destination directory for the LMS subfolder will be:
     `<document_root>/hrlms` (e.g. `public_html/hrdemo/hrlms` or `public_html/hrlms`).

---

## 2. Set PHP Version
1. In cPanel, go to **MultiPHP Manager** (or **Select PHP Version**).
2. Select domain `hrdemo.loopsintegrated.com`.
3. Set the PHP version to **PHP 8.2** or **PHP 8.3**.
4. In **PHP Extensions**, ensure the following are enabled:
   - `pdo_mysql`, `mbstring`, `openssl`, `bcmath`, `curl`, `xml`, `fileinfo`, `tokenizer`, `zip`, `gd`.

---

## 3. Create MySQL Database and User
1. In cPanel, open **MySQL® Databases**.
2. **Create New Database**: e.g., `loopsint_hrlms`.
3. **Create New User**: e.g., `loopsint_hruser` with a secure password.
4. **Add User To Database**: Select the user and database, click **Add**, and grant **ALL PRIVILEGES**.

---

## 4. Clone Repository via cPanel Git™ Version Control
1. In cPanel, open **Git™ Version Control** (under the **Files** section).
2. Click **Create** (top right):
   - **Clone a repository**: Turn this ON.
   - **Clone URL**: `https://github.com/dilmith-loops/crew-lms.git`
   - **Repository Path**: Path to your subfolder (e.g., `public_html/hrdemo/hrlms` or `public_html/hrlms`).
   - **Repository Name**: `hrlms`
3. Click **Create**.
   - cPanel will clone the repository directly into the `hrlms` directory.

---

## 5. Configure `.env` on the Server
1. In cPanel **File Manager** (ensure "Show Hidden Files" is enabled in Settings), navigate into the `hrlms` folder.
2. Locate [.env.cpanel.example](file:///.env.cpanel.example) and make a copy named `.env`.
3. Edit `.env` and fill in:
   ```env
   APP_NAME="Loops Work"
   APP_ENV=production
   APP_KEY=base64:... (generate via artisan or copy your existing key)
   APP_DEBUG=false
   APP_URL=https://hrdemo.loopsintegrated.com/hrlms
   ASSET_URL=https://hrdemo.loopsintegrated.com/hrlms

   DB_CONNECTION=mysql
   DB_HOST=127.0.0.1
   DB_PORT=3306
   DB_DATABASE=loopsint_hrlms
   DB_USERNAME=loopsint_hruser
   DB_PASSWORD=YourSecurePassword
   ```

---

## 6. Install Dependencies & Initialize Database

### Option A: Using cPanel Terminal (Recommended)
In cPanel, open **Terminal** and run:
```bash
# Navigate to the project directory
cd ~/public_html/hrdemo/hrlms   # (or your exact repo path)

# 1. Install production PHP dependencies
composer install --no-dev --optimize-autoloader

# 2. Generate application key (if not already set in .env)
php artisan key:generate

# 3. Run core migrations & module migrations
php artisan migrate --force
php artisan module:migrate

# 4. (Optional) Seed initial data & roles
php artisan db:seed --force

# 5. Create storage symlink
php artisan storage:link

# 6. Set directory permissions
chmod -R 775 storage bootstrap/cache

# 7. Optimize caches for production
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

### Option B: If cPanel Terminal is Disabled
1. **Vendor folder**:
   - Run `composer install --no-dev --optimize-autoloader` locally or upload `vendor.zip` via File Manager into `hrlms/` and extract it.
2. **Migrations & Artisan**:
   - In cPanel **Cron Jobs**, you can set a one-time cron command:
     `/usr/local/bin/php /home/USER/public_html/hrdemo/hrlms/artisan migrate --force && /usr/local/bin/php /home/USER/public_html/hrdemo/hrlms/artisan module:migrate`
   - Or import an exported `.sql` database dump via **phpMyAdmin**.

---

## 7. How Root `.htaccess` Routes Subfolder Requests
The repository includes a root [.htaccess](file:///.htaccess):
- Automatically routes all incoming requests from `https://hrdemo.loopsintegrated.com/hrlms/...` into the `public/` directory without exposing `public/` in the URL.
- Blocks direct HTTP access to `.env`, `.git`, `app/`, `bootstrap/`, `config/`, `database/`, `Modules/`, `resources/`, `routes/`, `storage/`, `tests/`, and `vendor/`.

---

## 8. Verification
Open your browser and visit:
👉 **`https://hrdemo.loopsintegrated.com/hrlms`**
