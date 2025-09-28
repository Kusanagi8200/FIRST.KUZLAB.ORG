
# **`FIRST.KUZLAB.ORG`**  
### **`HISTORICAL FIRST WORLD WIDE WEB INTERNET SITE REVISITED`**

### **`Line Mode Browser Simulator - Local Deployment`**

**This project makes it possible to clone and run the world’s first website locally, using the original CERN architecture together with the Line Mode Browser simulator. Anyone can reproduce the historical experience on their own server.**

**The very first website, served from a NeXT computer at CERN in 1991, introduced the core technologies that still power the modern web -->**

HTTP (HyperText Transfer Protocol) — a lightweight request/response protocol designed by Tim Berners-Lee.

HTML (HyperText Markup Language) — the first simple markup language for linking and structuring documents.

The WorldWideWeb application (later renamed Nexus) — the first browser/editor, running exclusively on NeXTSTEP.

The Line Mode Browser (1992) — the first cross-platform browser, accessible from any terminal, which truly democratized access to the web.  


**Today, The Kuz Network is building a revisited edition of this historic site - preserving its structure while presenting it under The Kuz Network identity, as both a tribute and a bridge between the early web and today’s open knowledge initiatives.**

**`Example deployment:` https://first.kuzlab.org/**  

---
### **`1. CLONE THE REPOSITORY`**

       cd /var/www/html 
       git clone https://github.com/cern-vet/line-mode-browser.git first 
       cd first 

---
### **`2. DIRECTORY LAYOUT`**

**/var/www/html/first/   
├── public/   
│   ├── config.js   
│   ├── autolaunch.js   
│   └── css/…   
├── site/                # local mirror of the original CERN website   
│   └── hypertext/WWW/…  
├── index.js             # Node.js server for the simulator   
├── package.json   
└── …**  

---
### **`3. PUBLIC/CONFIG.JS`**

**Configure the simulator to use the local mirror**

    window.LMB_CONF = {
    BASE_URL: "/site",       // local mirror root
    AUTOLAUNCH: true         // auto-start CLI
    };  
    
---
### **`4. PUBLIC/AUTOLAUNCH.JS`** 

**Force the simulator to launch automatically**

    document.addEventListener("DOMContentLoaded", () => {
    const link = document.querySelector(".lmblaunch");
    if (link) link.click();
    });  
    
---
### **`5. RUN THE SIMULATOR (NODE.JS)`**

**Start the Node.js backend on port 8000:**

    cd /var/www/html/first
    nohup node index.js &

---
### **`6. APACHE VIRTUAL HOSTS`**

**`CREATE A DNS RECORD`**

**Add an A record pointing $YOURDOMAIN to your VPS public IP.**  

**`INSTALL CERTBOT AND APACHE PLUGIN`**  

    apt update
    apt install certbot python3-certbot-apache -y  

**`ISSUE A CERTIFICATE FOR YOUR DOMAIN`**  

    certbot --apache -d first.kuzlab.org  
    apachectl -t
    systemctl reload apache2  

**`CONFIGURE VHOSTS FILES`**  

**/etc/apache2/sites-available/first-le-ssl.conf**
        
        <IfModule mod_ssl.c>
        <VirtualHost *:443>
        ServerName first.kuzlab.org

        # --- Local static mirror (/site/) ---
        Alias /site/ "/var/www/html/first/site/"
        <Directory "/var/www/html/first/site/">
          Options Indexes FollowSymLinks
          AllowOverride None
        Require all granted
        </Directory>

        # --- Local JS (bypass proxy) ---
        ProxyPass        /config.js !
        ProxyPass        /autolaunch.js !
        ProxyPass        /site/ !

        Alias /config.js "/var/www/html/first/public/config.js"
        Alias /autolaunch.js "/var/www/html/first/public/autolaunch.js"
        <Location "/config.js">Require all granted</Location>
        <Location "/autolaunch.js">Require all granted</Location>

        # --- Root redirect (directly to CLI) ---
        ProxyPassMatch ^/$ !
        RedirectMatch   301 ^/$ /www/hypertext/WWW/TheProject.html

        # --- Proxy everything else to Node.js ---
        ProxyPreserveHost On
        ProxyPass        / http://127.0.0.1:8000/
        ProxyPassReverse / http://127.0.0.1:8000/

        # --- Security headers ---
        <IfModule mod_headers.c>
        Header always set Content-Security-Policy "default-src 'self'; img-src 'self' data:; style-src 'self' 'unsafe-inline'; font-src 'self' data:; script-src 'self' 'unsafe-inline'; connect-src 'self'; frame-ancestors 'self'; base-uri 'self'; form-action 'self'"
        Header always set Referrer-Policy "strict-origin-when-cross-origin"
        Header always set X-Content-Type-Options "nosniff"
        Header always set X-Frame-Options "SAMEORIGIN"
        </IfModule>

        SSLCertificateFile /etc/letsencrypt/live/first.kuzlab.org/fullchain.pem
        SSLCertificateKeyFile /etc/letsencrypt/live/first.kuzlab.org/privkey.pem
        Include /etc/letsencrypt/options-ssl-apache.conf
        </VirtualHost>
        </IfModule>


 **/etc/apache2/sites-available/first.conf**

        <VirtualHost *:80>
        ServerName first.kuzlab.org

        # Redirect all HTTP to HTTPS
        RewriteEngine On
        RewriteCond %{SERVER_NAME} =first.kuzlab.org
        RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]
        </VirtualHost>

---
### **`7. ENABLE AND RELOAD APACHE`**

        a2enmod proxy proxy_http headers rewrite
        apachectl -t
        systemctl reload apache2

---
### **`8. FINAL CHECKS`**

        # HTTP → HTTPS redirect
        curl -I http://first.kuzlab.org/

        # Root → auto redirect to CLI simulator
        curl -I https://first.kuzlab.org/

        # Simulator target (should be 200)
        curl -I https://first.kuzlab.org/www/hypertext/WWW/TheProject.html

        # Local JS files (200)
        curl -I https://first.kuzlab.org/config.js
        curl -I https://first.kuzlab.org/autolaunch.js

        # Local mirror (200)
        curl -I https://first.kuzlab.org/site/hypertext/WWW/TheProject.html  
         
---
**`Visiting https://first.kuzlab.org/ launches the Line Mode Browser simulator with a fully local copy of the original CERN website.`**
<br>
**`Make your own code on it. Enjoy`**  

