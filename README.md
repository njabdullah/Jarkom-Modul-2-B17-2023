# Jarkom-Modul-2-B17-2023

Nama Anggota
1. Abdullah Nasih Jasir (5025211111)
2. Yohanes Teguh Ukur Ginting (5025211179)

## Soal 1
Yudhistira akan digunakan sebagai DNS Master, Werkudara sebagai DNS Slave, Arjuna merupakan Load Balancer yang terdiri dari beberapa Web Server yaitu Prabakusuma, Abimanyu, dan Wisanggeni. Buatlah topologi dengan pembagian sebagai berikut. Folder topologi dapat diakses pada drive berikut

#### Konfigurasi
*Di Router*

	auto eth0
	iface eth0 inet dhcp
	
	auto eth1
	iface eth1 inet static
		address 10.17.1.1
		netmask 255.255.255.0
	
	auto eth2
	iface eth2 inet static
		address 10.17.2.1
		netmask 255.255.255.0
	
	auto eth3
	iface eth3 inet static
		address 10.17.3.1
		netmask 255.255.255.0

#### Jawaban

## Soal 2
Buatlah website utama pada node arjuna dengan akses ke arjuna.yyy.com dengan alias www.arjuna.yyy.com dengan yyy merupakan kode kelompok.

#### Konfigurasi
*Di Node Yudhistira*

    apt-get update
    apt-get install bind9 -y
    cp -r -f /root/prak1/bind /etc/
    service bind9 restart

    nano /etc/bind/named.conf.local

    zone "arjuna.b17.com" {
	    type master;
	    file "/etc/bind/jarkom/arjuna.b17.com";
    };

    mkdir /etc/bind/jarkom

    cp /etc/bind/db.local /etc/bind/jarkom/arjuna.b17.com
    nano /etc/bind/jarkom/arjuna.b17.com  

    ((ganti menjadi arjuna.b17.com)
    ((ganti ip (10.17.1.4)))
    ((Tambahkan www	IN	CNAME	arjuna.b17.com.))

    service bind9 restart

*Di Node Nakula*

    echo nameserver 10.17.2.2 > /etc/resolv.conf
    ping www.arjuna.b17.com

#### Jawaban

## Soal 3
Dengan cara yang sama seperti soal nomor 2, buatlah website utama dengan akses ke abimanyu.yyy.com dan alias www.abimanyu.yyy.com.

#### Konfigurasi
*Di Node Yudhistira*

    nano /etc/bind/named.conf.local

    zone "abimanyu.b17.com" {
	    type master;
	    file "/etc/bind/jarkom/abimanyu.b17.com";
    };

    cp /etc/bind/db.local /etc/bind/jarkom/abimanyu.b17.com

    nano /etc/bind/jarkom/abimanyu.b17.com

    ((ganti nama menjadi abimanyu.b17.com))
    ((ganti IP (10.17.3.3)))
    ((Tambahkan www	IN	CNAME	abimanyu.b17.com.))
    
    service bind9 restart

*Di Node Nakula*

    echo nameserver 10.17.2.2 > /etc/resolv.conf (DIULANG) 
    ping www.abimanyu.b17.com

#### Jawaban

## Soal 4
Kemudian, karena terdapat beberapa web yang harus di-deploy, buatlah subdomain parikesit.abimanyu.yyy.com yang diatur DNS-nya di Yudhistira dan mengarah ke Abimanyu.

#### Konfigurasi
*Di Node Yudhistira*

    nano /etc/bind/jarkom/abimanyu.b17.com
    
    ((Tambahkan))
    parikesit	IN	A	10.17.3.3
    service bind9 restart

*Di Node Nakula*
    
    ping parikesit.abimanyu.b17.com

#### Jawaban

## Soal 5
Buat juga reverse domain untuk domain utama. (Abimanyu saja yang direverse)

#### Konfigurasi
*Di Node Yudhistira*
  
    nano /etc/bind/named.conf.local

    zone "3.17.10.in-addr.arpa" {
      type master;
      file "/etc/bind/jarkom/3.17.10.in-addr.arpa";
    };

    cp /etc/bind/db.local /etc/bind/jarkom/3.17.10.in-addr.arpa

    nano /etc/bind/jarkom/3.17.10.in-addr.arpa

    ((Tambahkan))
    3.17.10.in-addr.arpa.	IN	NS	abimanyu.b17.com.
    3			IN	PTR	abimanyu.b17.com.

    service bind9 restart

*Di Node Nakula*

    echo nameserver 192.168.122.1 > /etc/resolv.conf
    apt-get update
    apt-get install dnsutils
    echo nameserver 10.17.2.2 > /etc/resolv.conf
    host -t PTR 10.17.3.3 (punya abimanyu)

#### Jawaban

## Soal 6
Agar dapat tetap dihubungi ketika DNS Server Yudhistira bermasalah, buat juga Werkudara sebagai DNS Slave untuk domain utama.

#### Konfigurasi
*Di Node Yudhistira*

    nano /etc/bind/named.conf.local

    ((Pada setiap zone (kecuali reverse), tambahkan))
    also-notify { 10.17.2.3; }; // IP Werkudara
    allow-transfer { 10.17.2.3; }; // IP Werkudara

    service bind9 restart

*Di Node Werkudara*

    apt-get update
    apt-get install bind9 -y
    nano /etc/bind/named.conf.local

    zone "arjuna.b17.com" {
      type slave;
      masters { 10.17.2.2; };
      file "/var/lib/bind/arjuna.b17.com";
    };

    zone "abimanyu.b17.com" {
      type slave;
      masters { 10.17.2.2; }; 
      file "/var/lib/bind/abimanyu.b17.com";
    };

    service bind9 restart

*Di Node Yudhistira*

    service bind9 stop

*Di Node Nakula*

    nano /etc/resolv.conf
    nameserver 10.17.2.2
    nameserver 10.17.2.3

    ping www.arjuna.b17.com
    ping www.abimanyu.b17.com

#### Jawaban

## Soal 7
Seperti yang kita tahu karena banyak sekali informasi yang harus diterima, buatlah subdomain khusus untuk perang yaitu baratayuda.abimanyu.yyy.com dengan alias www.baratayuda.abimanyu.yyy.com yang didelegasikan dari Yudhistira ke Werkudara dengan IP menuju ke Abimanyu dalam folder Baratayuda.

#### Konfigurasi
*Di Yudhistira*

	  nano /etc/bind/jarkom/abimanyu.b17.com

	  ((Tambahkan))
	  ns1	IN	A	10.17.3.3
	  baratayuda	IN	NS	ns1

	  nano /etc/bind/named.conf.options

	  comment //dnsec
	  allow-query{any;};

	  service bind9 restart

*Di Node Werkudara*

	nano /etc/bind/named.conf.options
 
    comment //dnsec
    allow-query{any;};

	  nano /etc/bind/named.conf.local
      
    zone "baratayuda.abimanyu.b17.com" {
      type master;
      file "/etc/bind/Baratayuda/baratayuda.abimanyu.b17.com";
    };

	  mkdir /etc/bind/Baratayuda

    cp /etc/bind/db.local /etc/bind/Baratayuda/baratayuda.abimanyu.b17.com
	
	  nano  /etc/bind/Baratayuda/baratayuda.abimanyu.b17.com

	  ((ganti nama menjadi baratayuda.abimanyu.b17.com))
	  ((ganti IP abimanyu))
	  ((ganti AAAA jadi www	IN	A	10.17.3.3))

	  service bind9 restart

*Di Node Nakula*

  	nano /etc/resolv.conf

    nameserver 10.17.2.2
    nameserver 10.17.2.3	

    ping www.baratayuda.abimanyu.b17.com

#### Jawaban

## Soal 8
Untuk informasi yang lebih spesifik mengenai Ranjapan Baratayuda, buatlah subdomain melalui Werkudara dengan akses rjp.baratayuda.abimanyu.yyy.com dengan alias www.rjp.baratayuda.abimanyu.yyy.com yang mengarah ke Abimanyu.

#### Konfigurasi
*Di Node Werkudara*

    nano /etc/bind/Baratayuda/baratayuda.abimanyu.b17.com

    ((Tambahkan))
    rjp	IN	A	10.17.3.3
    www.rjp	IN	CNAME	rjp

    service bind9 restart

*Di Node Nakula*

    ping www.rjp.baratayuda.abimanyu.b17.com

#### Jawaban

## Soal 9
Arjuna merupakan suatu Load Balancer Nginx dengan tiga worker (yang juga menggunakan nginx sebagai webserver) yaitu Prabakusuma, Abimanyu, dan Wisanggeni. Lakukan deployment pada masing-masing worker.

#### Konfigurasi
*Di Router*
    
    apt-get update
    apt-get install nginx -y
    service nginx start

*Di Node Yudhistira*
    
    apt-get install bind9 nginx -y
    service nginx start

*Di Node Prabukusuma/Wisangeni/Abimanyu*
  
    apt-get update && apt install nginx php php-fpm -y
    mkdir /var/www/jarkom

	  nano /var/www/jarkom/index.php

—---------------------------------------------(MILIH)------------------------------------------------------
    	
     <?php
    echo "Halo, Kamu berada di Prabukusuma";
     	?>
    
    	<?php
    echo "Halo, Kamu berada di Abimanyu";
     	?>
    
    	<?php
    echo "Halo, Kamu berada di Wisanggeni";
     	?>
—---------------------------------------------(MILIH)------------------------------------------------------
	 
     nano /etc/nginx/sites-available/jarkom

—---------------------------------------------(MILIH)------------------------------------------------------
	
     server {
     	listen 8001;
     	root /var/www/jarkom;
     	index index.php index.html index.htm;
     	server_name _;
     	location / {
     			try_files $uri $uri/ /index.php?$query_string;
     	}
     	# pass PHP scripts to FastCGI server
     	location ~ \.php$ {
     	include snippets/fastcgi-php.conf;
     	fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
     	}
    location ~ /\.ht {
     			deny all;
     	}
     	error_log /var/log/nginx/jarkom_error.log;
     	access_log /var/log/nginx/jarkom_access.log;
     	}
    
    	server {
     	listen 8002;
     	root /var/www/jarkom;
     	index index.php index.html index.htm;
     	server_name _;
     	location / {
     			try_files $uri $uri/ /index.php?$query_string;
     	}
     	# pass PHP scripts to FastCGI server
     	location ~ \.php$ {
     	include snippets/fastcgi-php.conf;
     	fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
     	}
    location ~ /\.ht {
     			deny all;
     	}
     	error_log /var/log/nginx/jarkom_error.log;
     	access_log /var/log/nginx/jarkom_access.log;
     	}
    
    	server {
     	listen 8003;
     	root /var/www/jarkom;
     	index index.php index.html index.htm;
     	server_name _;
     	location / {
     			try_files $uri $uri/ /index.php?$query_string;
     	}
     	# pass PHP scripts to FastCGI server
     	location ~ \.php$ {
     	include snippets/fastcgi-php.conf;
     	fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
     	}
    location ~ /\.ht {
     			deny all;
     	}
     	error_log /var/log/nginx/jarkom_error.log;
     	access_log /var/log/nginx/jarkom_access.log;
     	}

—---------------------------------------------(MILIH)------------------------------------------------------

    ln -s /etc/nginx/sites-available/jarkom /etc/nginx/sites-enabled

    service nginx start
	  service php7.0-fpm start

    nginx -t

#### Jawaban

## Soal 10
Kemudian gunakan algoritma Round Robin untuk Load Balancer pada Arjuna. Gunakan server_name pada soal nomor 1. Untuk melakukan pengecekan akses alamat web tersebut kemudian pastikan worker yang digunakan untuk menangani permintaan akan berganti ganti secara acak. Untuk webserver di masing-masing worker wajib berjalan di port 8001-8003. Contoh
    - Prabakusuma:8001
    - Abimanyu:8002
    - Wisanggeni:8003

#### Konfigurasi
*Di Node Arjuna*

    cp -r -f /etc/nginx /root/prak1/ (PENTING)
	  cp -r -f /root/prak1/nginx /etc/

    apt-get update
    apt-get install bind9 nginx -y

    nano /etc/nginx/sites-available/lb-jarkom
  
    upstream myweb {
      server 10.17.3.2:8001;
      server 10.17.3.3:8002;
      server 10.17.3.4:8003;
    }
    
    server {
      listen 80;
      server_name arjuna.b17.com;
    
      location / {
        proxy_pass http://myweb;
      }
    }

    ln -s /etc/nginx/sites-available/lb-jarkom /etc/nginx/sites-enabled

    service nginx start
    service nginx restart

*Di Node Nakula*
    
    apt-get install lynx -y
    nano /etc/resolv.conf

    nameserver 10.17.2.2
    nameserver 10.17.2.3	

    lynx http://arjuna.b17.com

#### Jawaban

## Soal 11
Selain menggunakan Nginx, lakukan konfigurasi Apache Web Server pada worker Abimanyu dengan web server www.abimanyu.yyy.com. Pertama dibutuhkan web server dengan DocumentRoot pada /var/www/abimanyu.yyy

#### Konfigurasi
*Di Node Abimanyu*

	apt-get install apache2 -y
	service apache2 start

	cp /etc/apache2/sites-available/000-default.conf  /etc/apache2/sites-available/abimanyu.b17.com.conf

	nano /etc/apache2/sites-available/abimanyu.b17.com.conf

	((Ubah ke))
	/var/www/abimanyu.b17
	
	((Dibawahnya tambahkan))
	ServerName abimanyu.b17.com
	ServerAlias www.abimanyu.b17.com
	
	a2ensite abimanyu.b17.com
	service apache2 restart
	mkdir /var/www/abimanyu.b17
	nano /var/www/abimanyu.b17/index.php
	
	<?php
	    echo "Abimanyu adalah kota di One Piece...";
	?>

### Jawaban

## Soal 12
Setelah itu ubahlah agar url www.abimanyu.yyy.com/index.php/home menjadi www.abimanyu.yyy.com/home.

#### Konfigurasi
	<VirtualHost *:80>
	        ServerAdmin webmaster@localhost
	        DocumentRoot /var/www/abimanyu.b19
	        ServerName abimanyu.b19.com
	        ServerAlias www.abimanyu.b19.com
	
	        Alias "/home" "/var/www/abimanyu.b19/index.php"
	
	        ErrorLog ${APACHE_LOG_DIR}/error.log
	        CustomLog ${APACHE_LOG_DIR}/access.log combined
	</VirtualHost>

#### Jawaban

## Soal 13
Selain itu, pada subdomain www.parikesit.abimanyu.yyy.com, DocumentRoot disimpan pada /var/www/parikesit.abimanyu.yyy

#### Konfigurasi
	<VirtualHost *:80>
	   DocumentRoot /var/www/parikesit.abimanyu.b19
	   ServerName parikesit.abimanyu.b19.com
	   ServerAlias www.parikesit.abimanyu.b19.com
	</VirtualHost>

#### Jawaban

## Soal 14
Pada subdomain tersebut folder /public hanya dapat melakukan directory listing sedangkan pada folder /secret tidak dapat diakses (403 Forbidden).
### Jawaban

## Soal 15
Buatlah kustomisasi halaman error pada folder /error untuk mengganti error kode pada Apache. Error kode yang perlu diganti adalah 404 Not Found dan 403 Forbidden.
### Jawaban

## Soal 16
Buatlah suatu konfigurasi virtual host agar file asset www.parikesit.abimanyu.yyy.com/public/js menjadi 
www.parikesit.abimanyu.yyy.com/js 
### Jawaban

## Soal 17
Agar aman, buatlah konfigurasi agar www.rjp.baratayuda.abimanyu.yyy.com hanya dapat diakses melalui port 14000 dan 14400.
### Jawaban

## Soal 18
Untuk mengaksesnya buatlah autentikasi username berupa “Wayang” dan password “baratayudayyy” dengan yyy merupakan kode kelompok. Letakkan DocumentRoot pada /var/www/rjp.baratayuda.abimanyu.yyy.
### Jawaban

## Soal 19
Buatlah agar setiap kali mengakses IP dari Abimanyu akan secara otomatis dialihkan ke www.abimanyu.yyy.com (alias)
### Jawaban

## Soal 20
Karena website www.parikesit.abimanyu.yyy.com semakin banyak pengunjung dan banyak gambar gambar random, maka ubahlah request gambar yang memiliki substring “abimanyu” akan diarahkan menuju abimanyu.png.

#### Konfigurasi
	<VirtualHost *:80>
	   DocumentRoot /var/www/parikesit.abimanyu.b19
	   ServerName parikesit.abimanyu.b19.com
	   ServerAlias www.parikesit.abimanyu.b19.com
	
	        <Directory /var/www/parikesit.abimanyu.b19/public>
	                Options +Indexes
	         </Directory>
	         <Directory /var/www/parikesit.abimanyu.b19/secret>
	                Require all denied
	         </Directory>
	
	    Alias "/js" "/var/www/parikesit.abimanyu.b19/public/js"
	    ErrorDocument 404 /error/404.html
	    ErrorDocument 403 /error/403.html
	
	        RewriteEngine On
	        RewriteCond %{REQUEST_URI} abimanyu
	        RewriteRule ^.abimanyu.\.(jpg|jpeg|png|gif)$ /public/images/abimanyu.$
	</VirtualHost>
#### Jawaban
