# **FIRST.KUZLAB.ORG**

### **Line Mode Browser Simulator - Local Deployment**

### **This guide explains how to host a fully local clone of the first website and run the Line Mode Browser simulator under your own domain.**
**Example deployment: https://first.kuzlab.org/** 


### **1. Clone the Repository**

       cd /var/www/html 
       git clone https://github.com/cern-vet/line-mode-browser.git first 
       cd first 
       npm install  


### **2. Directory Layout**

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


### **3. public/config.js**

**Configure the simulator to use the local mirror**

    window.LMB_CONF = {
    BASE_URL: "/site",       // local mirror root
    AUTOLAUNCH: true         // auto-start CLI
    }; 


### **4. public/autolaunch.js** 

**Force the simulator to launch automatically**

    document.addEventListener("DOMContentLoaded", () => {
    const link = document.querySelector(".lmblaunch");
    if (link) link.click();
    }); 


### **5. Run the Simulator (Node.js)**

**Start the Node.js backend on port 8000:**

    cd /var/www/html/first
    nohup node index.js &


### **6. Apache Virtual Hosts**

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


### **7. Enable and Reload Apache**

        a2enmod proxy proxy_http headers rewrite
        apachectl -t
        systemctl reload apache2


### **8. Final Checks**

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


**Result**

**Visiting https://first.kuzlab.org/
directly launches the Line Mode Browser simulator with a fully local copy of the original CERN website.**







