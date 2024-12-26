# README: YOU SHALL NOT PASS

## **Project Description**
This project sets up a network infrastructure with multiple virtual machines (VMs) configured for specific roles. The main objective is to deploy a functional environment with web services, segmented networks, a database server, and a gateway to connect the subnets while adhering to security requirements.

---

## **Project Structure**

### **VM1: Gateway (OpenBSD)**
- **Role**: Gateway between subnets and the Internet.
- **Features**:
  - DHCP server to assign fixed IPs to other VMs.
  - NAT to enable communication between internal networks and the Internet.
  - Firewall (PF) configured to control access.

### **VM2: Web Server (FreeBSD)**
- **Role**: Web and application server.
- **Features**:
  - NGINX configured to serve HTML pages and execute PHP scripts.
  - PHP 7.4 with the `mysqli` and `fpm` module.
  - MySQL 8.0 for database management.
  - Fixed IP address (192.168.42.70) obtained via DHCP.

### **VM3: Employees (Debian)**
- **Role**: User workstation with limited access to the web server.
- **Features**:
  - HTTP/HTTPS access only to VM2.

### **VM4: Administrators (Debian)**
- **Role**: User workstation with administrative access.
- **Features**:
  - Full access to VM2 and other subnets for administration.

---

## **Detailed Configuration**

### **VM1: DHCP Configuration**
In `/etc/dhcpd.conf`:
```dhcp
#LAN-1: Administration

subnet 192.168.42.0 netmask 255.255.255.192 {
    range 192.168.42.40 192.168.42.60;
    option routers 192.168.42.1;
    option broadcast-address 192.168.42.63;
    option domain-name-server 8.8.8.8, 8.8.4.4;
}

#LAN-2: Server

subnet 192.168.42.64 netmask 255.255.255.192 {
    range 192.168.42.70 192.168.42.110;
    option routers 192.168.42.65;
    option broadcast-address 192.168.42.127;
    option domain-name-server 8.8.8.8, 8.8.4.4;
}

#LAN-3: Employee

subnet 192.168.42.128 netmask 255.255.255.192 {
    range 192.168.42.140 192.168.42.180;
    option routers 192.168.42.129;
    option broadcast-address 192.168.42.191;
    option domain-name-server 8.8.8.8, 8.8.4.4;
}
```
Restart the DHCP service:
```bash
sudo /etc/rc.d/dhcpd restart
```

### **VM2: NGINX and PHP Configuration**
- **NGINX**:
In `/usr/local/etc/nginx/nginx.conf`:
```nginx
server {
    listen 80;
    server_name localhost;

    root /usr/local/www/nginx;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include        fastcgi_params;
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
```
- **Test the configuration**:
```bash
nginx -t && service nginx restart
```
- **PHP-FPM**: Enable and start the service:
```bash
sysrc php_fpm_enable="YES"
service php-fpm start
```

### **VM3 and VM4: Network Configuration**
- Configure the network in DHCP mode.
- Check the assigned IP address with `ip a`.

---

## **Testing and Validation**

### **1. Connectivity**
- Ping between VMs to validate network connections.
- Access VM2's web page from VM3 and VM4:
  ```bash
  wget 192.168.42.70:80
  ```

### **2. Web Server Functionality**
- Test a PHP file on VM2:
  Create `/usr/local/www/nginx/info.php`:
  ```php
  <?php phpinfo(); ?>
  ```
  Access it at `http://192.168.42.70/info.php`.

### **3. MySQL Access**
- Test the connection from VM4 using `mysql-client`:
  ```bash
  mysql -u backend -h 192.168.42.70 -p Bit8Q6a6G
  ```

---


## **Author**
This project was completed by Gibril Kharfallah & Rayan Habes as part of the T-NSA-501 course. For any questions, feel free to contact us!

