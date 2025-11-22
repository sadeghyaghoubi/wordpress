# ------------------------------------------------------------
# فعال کردن Rewrite برای وردپرس
# ------------------------------------------------------------
RewriteEngine On

# ------------------------------------------------------------
# ریدایرکت اجباری HTTPS برای بهبود سئو
# ------------------------------------------------------------
RewriteCond %{HTTPS} !=on
RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]

# ------------------------------------------------------------
# ریدایرکت به نسخه بدون www یا با www (نسخه پیشنهادی: بدون www)
# اگر می‌خواهی با www باشد بگو تا نسخه دیگر را بدهم
# ------------------------------------------------------------
RewriteCond %{HTTP_HOST} ^www\.atourcenter\.ir [NC]
RewriteRule ^(.*)$ https://atourcenter.ir/$1 [L,R=301]

# ------------------------------------------------------------
# جلوگیری از نمایش لیست فایل‌ها
# ------------------------------------------------------------
Options -Indexes

# ------------------------------------------------------------
# جلوگیری از دسترسی به فایل‌های حساس
# ------------------------------------------------------------
<FilesMatch "\.(htaccess|htpasswd|ini|log|conf|env)$">
Order Allow,Deny
Deny from all
</FilesMatch>

# ------------------------------------------------------------
# فعال‌سازی کش مرورگر برای افزایش سرعت
# ------------------------------------------------------------
<IfModule mod_expires.c>
    ExpiresActive On
    ExpiresByType image/jpg "access plus 1 year"
    ExpiresByType image/jpeg "access plus 1 year"
    ExpiresByType image/png "access plus 1 year"
    ExpiresByType image/gif "access plus 1 year"
    ExpiresByType image/webp "access plus 1 year"
    ExpiresByType image/svg+xml "access plus 1 year"
    ExpiresByType text/css "access plus 1 month"
    ExpiresByType application/javascript "access plus 1 month"
    ExpiresByType text/html "access plus 1 hour"
</IfModule>

# ------------------------------------------------------------
# فعال‌سازی فشرده‌سازی Gzip
# ------------------------------------------------------------
<IfModule mod_deflate.c>
    AddOutputFilterByType DEFLATE text/plain
    AddOutputFilterByType DEFLATE text/html
    AddOutputFilterByType DEFLATE text/xml
    AddOutputFilterByType DEFLATE text/css
    AddOutputFilterByType DEFLATE application/xml
    AddOutputFilterByType DEFLATE application/xhtml+xml
    AddOutputFilterByType DEFLATE application/javascript
    AddOutputFilterByType DEFLATE application/x-javascript
</IfModule>

# ------------------------------------------------------------
# جلوگیری از هات‌لینک تصاویر (دزدیدن ترافیک)
# ------------------------------------------------------------
RewriteCond %{HTTP_REFERER} !^$
RewriteCond %{HTTP_REFERER} !^https?://(www\.)?atourcenter\.ir/ [NC]
RewriteRule \.(jpg|jpeg|png|gif|webp)$ - [F]

# ------------------------------------------------------------
# بلاک حملات معمول (Query های مشکوک)
# ------------------------------------------------------------
<IfModule mod_security.c>
    SecFilterEngine On
    SecFilterScanPOST On
</IfModule>

# ------------------------------------------------------------
# بخش اصلی وردپرس — تغییر نده
# ------------------------------------------------------------
# BEGIN WordPress
RewriteRule ^index\.php$ - [L]

RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . /index.php [L]
# END WordPress
