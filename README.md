# ServerUbuntuAdmin

### 1.Initial server configuration
1.1 Add new user
    ls /home
    adduser andrei

1.2 Give root privileges
    visudo
    add the following: andrei ALL=(ALL:ALL) ALL
  
1.3 Disable root access
    1.3.1 backup the configuration file
          
          cd /etc/ssh
          
          cp sshd_config sshd_config.bak
          
          nano sshd_config
          
          Modify: PermitRootLogin no
          
          systemctl restart ssh

1.4 Server package updates
    
    clear
    
    sudo apt update (updates the latest packages)
    
    sudo apt upgrade (run the package upgrade)
    
    sudo apt autoremove (removed unneded packages)
    
1.5 Firewall
    Allow only ports 80,443,22
    
    1.5.1 Check if firewall is installed
    
    sudo ufw status verbose
    
    1.5.2 Install ufw and configure
    
    sudo apt install ufw
    
    sudo ufw default deny incoming
    
    sudo ufw default allow outgoing
    
    sudo ufw allow ssh
    
    sudo ufw allow http
    
    sudo ufw allow https
    
    sudo ufw enable
    
    sudo ufw status verbose
    
    sudo reboot
    
1.6 Fail 2 Ban
    sudo apt install fail2ban
    
    cd /etc/fail2ban
    
    ls -l
    
    sudo cp jail.conf jail.local
    
    sudo nano jail.local
    
    jail.local files changes:
    ------------------------------------------------------------------------------------
    # "bantime" is the number of seconds that a host is banned.
    bantime  = 604800s

    # A host is banned if it has generated "maxretry" during the last "findtime"
    # seconds.
    findtime  = 10800s

    # "maxretry" is the number of failures before a host get banned.
    maxretry = 5

    # "maxmatches" is the number of matches stored in ticket (resolvable via tag <matches>>
    maxmatches = %(maxretry)s
    --------------------------------------------------------------------------------------
    
    Find (ctrl + w) [sshd] section
    --------------------------------------------------------------------------------------
    [sshd]

    # To use more aggressive sshd modes set filter parameter "mode" in jail.local:
    # normal (default), ddos, extra or aggressive (combines all).
    # See "tests/files/logs/sshd" or "filter.d/sshd.conf" for usage example and details.
    mode   = aggressive
    port    = ssh
    logpath = %(sshd_log)s
    backend = %(sshd_backend)s
    enabled = true
    ---------------------------------------------------------------------------------------

    Restart the service:
    sudo systemctl restart fail2ban

### 2. Installing & Optimizing Nginx, MariaDB and PHP 7.4

        sudo apt install nginx libnginx-mod-http-headers-more-filter
        sudo systemctl status nginx
        
        sudo apt install mariadb-server

### 3. NGINX configuration

    /etc/nginx$ sudo cp nginx.conf nginx.conf.bak
    
    sudo nano nginx.conf
    
nginx.conf edited file:

    ##
    # Basic Settings
    ##

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    server_tokens off;
    more_clear_headers 'Server';
    
Setup virtual host:

    sudo nano /etc/nginx/sites-available/domain-name.com
    
    Insert configuration:
    server {
        listen 80; # Specify the listening port
        listen [::]:80; # The same thing for IPv6
        root /var/www/domain-name.com/html; # The path to the website files
        index index.html index.htm; # Files to display if only the domain name is specified in the address
        server_name domain-name.com; # Domain name of this site
        location / {
        try_files $uri $uri/ =404;
        }
    }
    
    mkdir -p /var/www/domain-name.com/html
    chmod -R 755 /var/www
    ln -s /etc/nginx/sites-available/domain-name.com /etc/nginx/sites-enabled/
    nginx -t
    systemctl restart nginx

### 4. SSL configuration

### 5. Fail2Ban Config

### 6. Install .Net Core / .Net Core SDK

### 7. Lets encrypt SSL with CloudFlare DNS
based on: https://www.bjornjohansen.com/wildcard-certificate-letsencrypt-cloudflare

    a. Get CloudFlare API credentials
    sudo chmod 0700 /root/.secrets/
    sudo chmod 0400 /root/.secrets/cloudflare.ini
    dns_cloudflare_email = "youremail@example.com"
    dns_cloudflare_api_key = <ApiToken>
    
    b.Install Certbot
    
    sudo apt-get update
    sudo apt-get install software-properties-common
    sudo add-apt-repository universe
    sudo add-apt-repository ppa:certbot/certbot
    sudo apt-get update
    sudo apt-get install certbot python-certbot-nginx python3-certbot-dns-cloudflare
    
    c. Run Certbot with CloudFlare authenticator
    sudo certbot certonly --dns-cloudflare --dns-cloudflare-credentials /root/.secrets/cloudflare.ini -d example.com,*.example.com --preferred-challenges dns-01
    
    d. Automatic renew
    certbot renew --dry-run
    
    f. Cron autorenew
    14 5 * * * /usr/bin/certbot renew --quiet --post-hook "/usr/sbin/service nginx reload" > /dev/null 2>&
    
based on: https://hijackson.com/how-to-generate-lets-encrypt-ssl-wildcard-with-cloudflare-using-certbot/
