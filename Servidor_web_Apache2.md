# Servidor Web Apache 2.4

#### Tarea 1: Crea un escenario Vagrant con una máquina con una red pública o utiliza una máquina del cloud. Instala el servidor web Apache2 en la máquina. Modifica la página index.html que viene por defecto y accede a ella desde un navegador. Entrega una captura de pantalla accediendo a ella.

Creación de la máquina con Vagrant:
[Máquina_servidor_apache](https://github.com/PalomaR88/Practica_Apache2/blob/master/Vagrantfile "Máquina Debian Buster con Vagrant")

Se isntala el paquete de Apache2:
~~~
vagrant@servidor:~$ sudo apt install apache2
~~~

Se modifica **/var/www/html/index.html**

![Imagen1](images/Img_tarea1.png)



## Virtual Hosting

**Queremos que nuestro servidor web ofrezca dos sitios web, teniendo en cuenta lo siguiente:**

- Cada sitio web tendrá nombres distintos.
- Cada sitio web compartirán la misma dirección IP y el mismo puerto (80).

**Los dos sitios web tendrán las siguientes características:**

- El nombre de dominio del primero será www.iesgn.org, su directorio base será /srv/www/iesgn y contendrá una página llamada index.html, donde sólo se verá una bienvenida a la página del Instituto Gonzalo Nazareno.
- En el segundo sitio vamos a crear una página donde se pondrán noticias por parte de los departamento, el nombre de este sitio será departamentos.iesgn.org, y su directorio base será /srv/www/departamentos. En este sitio sólo tendremos una página inicial index.html, dando la bienvenida a la página de los departamentos del instituto.

### Tarea 2: Configura la resolución estática en los clientes y muestra el acceso a cada una de las páginas.

> Primer sitio: www.iesgn.org:
Configuración **/etc/apache2/sites-available/iesgn.conf**:
~~~
<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        DocumentRoot /srv/www/iesgn
        ServerName www.iesgn.org
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
~~~

Crear el DocumentRoot:
~~~
vagrant@servidor:/srv/www$ sudo mkdir iesgn
vagrant@servidor:/srv/www/iesgn$ sudo nano index.html
vagrant@servidor:/srv/www/iesgn$ sudo chown -R www-data:www-data /srv/www/*
~~~

Y se configura **/etc/apache2/apache2.conf** para que la nueva ruta sea accesible:
~~~
<Directory /srv/>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
</Directory>
~~~

Se crea el enlace y se reinicia Apache2:
~~~
vagrant@servidor:/srv/www/iesgn$ sudo a2ensite iesgn
Enabling site iesgn.
To activate the new configuration, you need to run:
  systemctl reload apache2
~~~


> Segundo sitio: departamentos.iesgn.org

Configuración **/etc/apache2/sites-available/departamentos.conf**:
~~~
vagrant@servidor:/srv/www/iesgn$ sudo cp /etc/apache2/sites-available/iesgn.conf /etc/apache2/sites-available/departamentos.conf
~~~
~~~
<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        DocumentRoot /srv/www/departamentos
        ServerName departamentos.iesgn.org
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
~~~

Crear el DocumentRoot:
~~~
vagrant@servidor:/srv/www$ sudo mkdir departamentos
vagrant@servidor:/srv/www/iesgn$ sudo nano index.html
vagrant@servidor:/srv/www/iesgn$ sudo chown -R www-data:www-data /srv/www/*
~~~


Se crea el enlace y se reinicia Apache2:
~~~
vagrant@servidor:/srv/www/iesgn$ sudo a2ensite departamentos
Enabling site iesgn.
To activate the new configuration, you need to run:
  systemctl reload apache2
vagrant@servidor:/srv/www$ sudo systemctl reload apache2
~~~

> Configuración de **/etc/hosts** en la máquina donde se va a probar:
~~~ 
172.22.8.56 www.iesgn.org departamentos.iesgn.org
~~~

![Imagen2](images/Img_tarea2.A.png)
![Imagen2B](images/Img_tarea2.B.png)



## Mapeo de URL

> Cambia la configuración del sitio web www.iesgn.org para que se comporte de la siguiente forma:

#### Tarea 3: Cuando se entre a la dirección www.iesgn.org se redireccionará automáticamente a www.iesgn.org/principal, donde se mostrará el mensaje de bienvenida. En el directorio principal no se permite ver la lista de los ficheros, no se permite que se siga los enlaces simbólicos y no se permite negociación de contenido. Muestra al profesor el funcionamiento.

Hay que modificar el fichero **/etc/apache2/sites-available/iesgn.conf** de la siguiente manera:
~~~
        RedirectMatch 301 ^/$ http://www.iesgn.org/principal
~~~


#### Tarea 4: Si accedes a la página www.iesgn.org/principal/documentos se visualizarán los documentos que hay en /srv/doc. Por lo tanto se permitirá el listado de fichero y el seguimiento de enlaces simbólicos siempre que sean a ficheros o directorios cuyo dueño sea el usuario. Muestra al profesor el funcionamiento.

~~~
vagrant@servidor:/srv/www/iesgn$ sudo mkdir /srv/doc
~~~
En /etc/apache2/sites-available/iesgn.conf se añade:
~~~
        Alias "/principal/documentos" "/srv/doc"
~~~

Se configura /etc/apache2/apache2.conf:
~~~
<Directory /srv/www/>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
</Directory>
<Directory /srv/doc/>
        Options Indexes SymLinksIfOwnerMatch
        AllowOverride All
        Require all granted
</Directory>

~~~
Se crea los enlaces simbólicos:
- fich2.txt que será con el mismo usuario:
~~~
vagrant@servidor:/srv/doc$ nano /home/vagrant/fich2.txt
vagrant@servidor:/srv/doc$ sudo ln -s /home/vagrant/fich2.txt .
vagrant@servidor:/srv/doc$ sudo chown -R www-data:www-data fich2.txt 
~~~

- fich3.txt diferentes usuarios:
~~~
vagrant@servidor:/srv/doc$ nano /home/vagrant/fich3.txt
vagrant@servidor:/srv/doc$ sudo ln -s /home/vagrant/fich3.txt 
vagrant@servidor:/srv/doc$ sudo chown www-data:www-data fich3.txt 
~~~

Comprobamos que los usuarios se han modificado correctamente:
~~~
vagrant@servidor:/srv/doc$ ls -l
total 0
-rw-r--r-- 1 root     root      0 Oct 22 17:49 fic1.txt
lrwxrwxrwx 1 www-data www-data 23 Oct 23 07:30 fich2.txt -> /home/vagrant/fich2.txt
lrwxrwxrwx 1 root     root     23 Oct 23 07:35 fich3.txt -> /home/vagrant/fich3.txt
vagrant@servidor:/srv/doc$ ls -l /home/vagrant/
total 8
-rw-r--r-- 1 www-data www-data  9 Oct 23 07:30 fich2.txt
-rw-r--r-- 1 www-data www-data 12 Oct 23 07:35 fich3.txt
~~~

![Imagen4](images/Img_tarea4.png)



#### Tarea 5: En todo el host virtual se debe redefinir los mensajes de error de objeto no encontrado y no permitido. Para el ello se crearan dos ficheros html dentro del directorio error. Entrega las modificaciones necesarias en la configuración y una comprobación del buen funcionamiento.

Autentificación, Autorización, y Control de Acceso
~~~
vagrant@servidor:/srv/www/iesgn$ sudo mkdir error
vagrant@servidor:/srv/www/iesgn/error$ sudo nano error404.html
vagrant@servidor:/srv/www/iesgn/error$ sudo nano error403.html
~~~

Configuración iesgn.org:
~~~
        ErrorDocument 404 /error/error404.html
        ErrorDocument 403 /error/error403.html
        <Directory /srv/www/iesgn/nopasar>
                Options All
                AllowOverride All
                Require all denied
        </Directory>
~~~

![Imagen5A](images/Img_tarea5.png)
![Imagen5B](images/Img_tarea5B.png)


#### Tarea 6: Añade al escenario Vagrant otra máquina conectada por una red interna al servidor. A la URL departamentos.iesgn.org/intranet sólo se debe tener acceso desde el cliente de la red local, y no se pueda acceder desde la anfitriona por la red pública. A la URL departamentos.iesgn.org/internet, sin embargo, sólo se debe tener acceso desde la anfitriona por la red pública, y no desde la red local.

Se modifica /etc/apache2/sites-available/departamentos.conf:
~~~
	<Directory "/srv/www/departamentos/intranet/">
		Require ip 192.168.100
	</Directory>      
	<Directory "/srv/www/departamentos/internet/">
                <RequireAll>
                        Require all granted
                        Require not ip 192.168.100
                </RequireAll>
        </Directory>
~~~

En la máquina cliente obtengo los siguientes resultados entrando en departamentos.iesgn.org/intranet:
~~~
   Intranet


Commands: Use arrow keys to move, '?' for help, 'q' to quit, '<-' to go b
  Arrow keys: Up and Down to move.  Right to follow a link; Left to go bac
 H)elp O)ptions P)rint G)o M)ain screen Q)uit /=search [delete]=history l
~~~

Y entrando en departamentos.iesgn.org/internet:
~~~
                                Forbidden

   You don't have permission to access this resource.
     ____________________________________________________________


    Apache/2.4.38 (Debian) Server at departamentos.iesgn.org Port
    80
~~~

![Imagen6](images/Img_tarea6A.png)
![Imagen6B](images/Img_tarea6B.png) 



#### Tarea 7: Autentificación básica. Limita el acceso a la URL departamentos.iesgn.org/secreto. Comprueba las cabeceras de los mensajes HTTP que se intercambian entre el servidor y el cliente. ¿Cómo se manda la contraseña entre el cliente y el servidor?. Entrega una breve explicación del ejercicio.

Se crea un directorio llamado secreto con un fichero html:
~~~
vagrant@servidor:~$ sudo mkdir /srv/www/departamentos/secreto
vagrant@servidor:~$ sudo touch /srv/www/departamentos/secreto/index.html
~~~

Se crea un direcotrio claves en /etc/apache2/:
~~~
vagrant@servidor:~$ sudo mkdir /etc/apache2/claves
~~~

Y un usuario con su respectiva contraseña:
~~~
vagrant@servidor:~$ sudo htpasswd -c /etc/apache2/claves/passwd.txt paloma
New password: 
Re-type new password: 
Adding password for user paloma
~~~

Se edita departamentos.conf:
~~~
        <Directory "/srv/www/departamentos/secreto/">
                AuthUserFile "/etc/apache2/claves/passwd.txt"
                AuthName "Contraseña"
                AuthType Basic
                Require valid-user
        </Directory>
~~~

Y se hace el tcpdump:
~~~
vagrant@servidor:~$ sudo tcpdump -vi eth1
tcpdump: listening on eth1, link-type EN10MB (Ethernet), capture size 262144 bytes
07:13:59.610124 IP (tos 0x0, ttl 64, id 60203, offset 0, flags [DF], proto TCP (6), length 60)
    172.22.8.100.54108 > 172.22.8.56.http: Flags [S], cksum 0xe287 (correct), seq 924958074, win 29200, options [mss 1460,sackOK,TS val 3457676350 ecr 0,nop,wscale 7], length 0
07:13:59.610207 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto TCP (6), length 60)
    172.22.8.56.http > 172.22.8.100.54108: Flags [S.], cksum 0x68f7 (incorrect -> 0x49ca), seq 1654091429, ack 924958075, win 28960, options [mss 1460,sackOK,TS val 1185972656 ecr 3457676350,nop,wscale 6], length 0
07:13:59.634311 IP (tos 0x0, ttl 64, id 60205, offset 0, flags [DF], proto TCP (6), length 411)
    172.22.8.100.54108 > 172.22.8.56.http: Flags [P.], cksum 0x43de (correct), seq 1:360, ack 1, win 229, options [nop,nop,TS val 3457676558 ecr 1185972656], length 359: HTTP, length: 359
	GET /secreto HTTP/1.1
	Host: departamentos.iesgn.org
	User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:60.0) Gecko/20100101 Firefox/60.0
	Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
	Accept-Language: es-ES,es;q=0.8,en-US;q=0.5,en;q=0.3
	Accept-Encoding: gzip, deflate
	DNT: 1
	Connection: keep-alive
	Upgrade-Insecure-Requests: 1
	
07:13:59.634448 IP (tos 0x0, ttl 64, id 24263, offset 0, flags [DF], proto TCP (6), length 52)
    172.22.8.56.http > 172.22.8.100.54108: Flags [.], cksum 0x68ef (incorrect -> 0xe590), ack 360, win 470, options [nop,nop,TS val 1185972680 ecr 3457676558], length 0
07:13:59.636986 IP (tos 0x0, ttl 64, id 24264, offset 0, flags [DF], proto TCP (6), length 787)
    172.22.8.56.http > 172.22.8.100.54108: Flags [P.], cksum 0x6bce (incorrect -> 0xda1e), seq 1:736, ack 360, win 470, options [nop,nop,TS val 1185972681 ecr 3457676558], length 735: HTTP, length: 735
	HTTP/1.1 401 Unauthorized
	Date: Wed, 23 Oct 2019 07:13:59 GMT
	Server: Apache/2.4.38 (Debian)
07:13:59.651580 IP (tos 0x0, ttl 64, id 60206, offset 0, flags [DF], proto TCP (6), length 52)
    172.22.8.100.54108 > 172.22.8.56.http: Flags [.], cksum 0xe379 (correct), ack 736, win 240, options [nop,nop,TS val 3457676587 ecr 1185972681], length 0
07:14:04.642232 IP (tos 0x0, ttl 64, id 24265, offset 0, flags [DF], proto TCP (6), length 52)
    172.22.8.56.http > 172.22.8.100.54108: Flags [F.], cksum 0x68ef (incorrect -> 0xcf03), seq 736, ack 360, win 470, options [nop,nop,TS val 1185977688 ecr 3457676587], length 0
07:14:04.648214 IP (tos 0x0, ttl 64, id 60207, offset 0, flags [DF], proto TCP (6), length 52)
    172.22.8.100.54108 > 172.22.8.56.http: Flags [F.], cksum 0xcfef (correct), seq 360, ack 736, win 240, options [nop,nop,TS val 3457681588 ecr 1185972681], length 0
07:14:04.648263 IP (tos 0x0, ttl 64, id 24266, offset 0, flags [DF], proto TCP (6), length 52)
    172.22.8.56.http > 172.22.8.100.54108: Flags [.], cksum 0x68ef (incorrect -> 0xbb73), ack 361, win 470, options [nop,nop,TS val 1185977694 ecr 3457681588], length 0
07:14:04.650122 IP (tos 0x0, ttl 64, id 60208, offset 0, flags [DF], proto TCP (6), length 52)
    172.22.8.100.54108 > 172.22.8.56.http: Flags [.], cksum 0xbc5c (correct), ack 737, win 240, options [nop,nop,TS val 3457681591 ecr 1185977688], length 0
07:14:04.927040 IP (tos 0x0, ttl 64, id 18677, offset 0, flags [DF], proto TCP (6), length 60)
    172.22.8.100.54110 > 172.22.8.56.http: Flags [S], cksum 0xde7d (correct), seq 3490648937, win 29200, options [mss 1460,sackOK,TS val 3457681769 ecr 0,nop,wscale 7], length 0
07:14:04.927103 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto TCP (6), length 60)
    172.22.8.56.http > 172.22.8.100.54110: Flags [S.], cksum 0x68f7 (incorrect -> 0x547e), seq 608864624, ack 3490648938, win 28960, options [mss 1460,sackOK,TS val 1185977972 ecr 3457681769,nop,wscale 6], length 0
07:14:04.929177 IP (tos 0x0, ttl 64, id 18678, offset 0, flags [DF], proto TCP (6), length 52)
    172.22.8.100.54110 > 172.22.8.56.http: Flags [.], cksum 0xf31c (correct), ack 1, win 229, options [nop,nop,TS val 3457681873 ecr 1185977972], length 0
07:14:04.931126 IP (tos 0x0, ttl 64, id 18679, offset 0, flags [DF], proto TCP (6), length 454)
    172.22.8.100.54110 > 172.22.8.56.http: Flags [P.], cksum 0x3b7f (correct), seq 1:403, ack 1, win 229, options [nop,nop,TS val 3457681873 ecr 1185977972], length 402: HTTP, length: 402
	GET /secreto HTTP/1.1
	Host: departamentos.iesgn.org
	User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:60.0) Gecko/20100101 Firefox/60.0
	Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
	Accept-Language: es-ES,es;q=0.8,en-US;q=0.5,en;q=0.3
	Accept-Encoding: gzip, deflate
	DNT: 1
	Connection: keep-alive
	Upgrade-Insecure-Requests: 1
	Authorization: Basic cGFsb21hOnBhbG9tYQ==
	
07:14:04.931241 IP (tos 0x0, ttl 64, id 48383, offset 0, flags [DF], proto TCP (6), length 52)
    172.22.8.56.http > 172.22.8.100.54110: Flags [.], cksum 0x68ef (incorrect -> 0xf094), ack 403, win 470, options [nop,nop,TS val 1185977977 ecr 3457681873], length 0
07:14:04.933294 IP (tos 0x0, ttl 64, id 48384, offset 0, flags [DF], proto TCP (6), length 664)
    172.22.8.56.http > 172.22.8.100.54110: Flags [P.], cksum 0x6b53 (incorrect -> 0x5db8), seq 1:613, ack 403, win 470, options [nop,nop,TS val 1185977979 ecr 3457681873], length 612: HTTP, length: 612
	HTTP/1.1 301 Moved Permanently
	Date: Wed, 23 Oct 2019 07:14:04 GMT
	Server: Apache/2.4.38 (Debian)
	Location: http://departamentos.iesgn.org/secreto/
	Content-Length: 336
	Keep-Alive: timeout=5, max=100
	Connection: Keep-Alive
	Content-Type: text/html; charset=iso-8859-1
	
	<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
	<html><head>
	<title>301 Moved Permanently</title>
	</head><body>
	<h1>Moved Permanently</h1>
	<p>The document has moved <a href="http://departamentos.iesgn.org/secreto/">here</a>.</p>
	<hr>
	<address>Apache/2.4.38 (Debian) Server at departamentos.iesgn.org Port 80</address>
	</body></html>
07:14:04.935481 IP (tos 0x0, ttl 64, id 18680, offset 0, flags [DF], proto TCP (6), length 52)
    172.22.8.100.54110 > 172.22.8.56.http: Flags [.], cksum 0xef10 (correct), ack 613, win 238, options [nop,nop,TS val 3457681879 ecr 1185977979], length 0
07:14:04.939225 IP (tos 0x0, ttl 64, id 9168, offset 0, flags [DF], proto TCP (6), length 60)
    172.22.8.100.54112 > 172.22.8.56.http: Flags [S], cksum 0xb808 (correct), seq 2433841256, win 29200, options [mss 1460,sackOK,TS val 3457681883 ecr 0,nop,wscale 7], length 0
07:14:04.939279 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto TCP (6), length 60)
    172.22.8.56.http > 172.22.8.100.54112: Flags [S.], cksum 0x68f7 (incorrect -> 0x5361), seq 2732582261, ack 2433841257, win 28960, options [mss 1460,sackOK,TS val 1185977985 ecr 3457681883,nop,wscale 6], length 0
07:14:04.942174 IP (tos 0x0, ttl 64, id 9169, offset 0, flags [DF], proto TCP (6), length 52)
    172.22.8.100.54112 > 172.22.8.56.http: Flags [.], cksum 0xf265 (correct), ack 1, win 229, options [nop,nop,TS val 3457681885 ecr 1185977985], length 0
07:14:04.942913 IP (tos 0x0, ttl 64, id 9170, offset 0, flags [DF], proto TCP (6), length 455)
    172.22.8.100.54112 > 172.22.8.56.http: Flags [P.], cksum 0x8e44 (correct), seq 1:404, ack 1, win 229, options [nop,nop,TS val 3457681885 ecr 1185977985], length 403: HTTP, length: 403
	GET /secreto/ HTTP/1.1
	Host: departamentos.iesgn.org
	User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:60.0) Gecko/20100101 Firefox/60.0
	Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
	Accept-Language: es-ES,es;q=0.8,en-US;q=0.5,en;q=0.3
	Accept-Encoding: gzip, deflate
	DNT: 1
	Authorization: Basic cGFsb21hOnBhbG9tYQ==
	Connection: keep-alive
	Upgrade-Insecure-Requests: 1
	
07:14:04.943008 IP (tos 0x0, ttl 64, id 53479, offset 0, flags [DF], proto TCP (6), length 52)
    172.22.8.56.http > 172.22.8.100.54112: Flags [.], cksum 0x68ef (incorrect -> 0xefde), ack 404, win 470, options [nop,nop,TS val 1185977988 ecr 3457681885], length 0
07:14:04.946170 IP (tos 0x0, ttl 64, id 53480, offset 0, flags [DF], proto TCP (6), length 333)
    172.22.8.56.http > 172.22.8.100.54112: Flags [P.], cksum 0x6a08 (incorrect -> 0xb247), seq 1:282, ack 404, win 470, options [nop,nop,TS val 1185977992 ecr 3457681885], length 281: HTTP, length: 281
	HTTP/1.1 200 OK
	Date: Wed, 23 Oct 2019 07:14:04 GMT
	Server: Apache/2.4.38 (Debian)
	Last-Modified: Wed, 23 Oct 2019 06:41:01 GMT
	ETag: "0-5958e33466792"
	Accept-Ranges: bytes
	Content-Length: 0
	Keep-Alive: timeout=5, max=100
	Connection: Keep-Alive
	Content-Type: text/html
	
07:14:04.950742 IP (tos 0x0, ttl 64, id 9171, offset 0, flags [DF], proto TCP (6), length 52)
    172.22.8.100.54112 > 172.22.8.56.http: Flags [.], cksum 0xefa3 (correct), ack 282, win 237, options [nop,nop,TS val 3457681892 ecr 1185977992], length 0
07:14:04.952385 IP (tos 0x0, ttl 64, id 18681, offset 0, flags [DF], proto TCP (6), length 52)
    172.22.8.100.54110 > 172.22.8.56.http: Flags [F.], cksum 0xeefe (correct), seq 403, ack 613, win 238, options [nop,nop,TS val 3457681896 ecr 1185977979], length 0
07:14:04.959201 IP (tos 0x0, ttl 64, id 48385, offset 0, flags [DF], proto TCP (6), length 52)
    172.22.8.56.http > 172.22.8.100.54110: Flags [F.], cksum 0x68ef (incorrect -> 0xedfb), seq 613, ack 404, win 470, options [nop,nop,TS val 1185978005 ecr 3457681896], length 0
07:14:04.961213 IP (tos 0x0, ttl 64, id 18682, offset 0, flags [DF], proto TCP (6), length 52)
    172.22.8.100.54110 > 172.22.8.56.http: Flags [.], cksum 0xeeda (correct), ack 614, win 238, options [nop,nop,TS val 3457681905 ecr 1185978005], length 0
07:14:09.935457 IP (tos 0x0, ttl 64, id 53481, offset 0, flags [DF], proto TCP (6), length 52)
    172.22.8.56.http > 172.22.8.100.54112: Flags [F.], cksum 0x68ef (incorrect -> 0xdb3c), seq 282, ack 404, win 470, options [nop,nop,TS val 1185982981 ecr 3457681892], length 0
07:14:09.940786 IP (tos 0x0, ttl 64, id 9172, offset 0, flags [DF], proto TCP (6), length 52)
    172.22.8.100.54112 > 172.22.8.56.http: Flags [F.], cksum 0xc8a3 (correct), seq 404, ack 283, win 237, options [nop,nop,TS val 3457686885 ecr 1185982981], length 0
07:14:09.940924 IP (tos 0x0, ttl 64, id 53482, offset 0, flags [DF], proto TCP (6), length 52)
    172.22.8.56.http > 172.22.8.100.54112: Flags [.], cksum 0x68ef (incorrect -> 0xc7b5), ack 405, win 470, options [nop,nop,TS val 1185982986 ecr 3457686885], length 0
~~~


#### Tarea 8: Cómo hemos visto la autentificación básica no es segura, modifica la autentificación para que sea del tipo digest, y sólo sea accesible a los usuarios pertenecientes al grupo directivos. Comprueba las cabeceras de los mensajes HTTP que se intercambian entre el servidor y el cliente. ¿Cómo funciona esta autentificación?

Se activa el módulo auth_digest:
~~~
vagrant@servidor:~$ sudo a2enmod auth_digest 
Considering dependency authn_core for auth_digest:
Module authn_core already enabled
Enabling module auth_digest.
To activate the new configuration, you need to run:
  systemctl restart apache2
vagrant@servidor:~$ sudo systemctl restart apache2
~~~

En la configuración de departamento se cambia el tipo de autentificación:
~~~
        <Directory "/srv/www/departamentos/secreto/">
                AuthUserFile "/etc/apache2/claves/passwdDigest.txt"
                AuthName "Contraseña"
                AuthType Digest
                Require valid-user
        </Directory>
~~~

Se añade un usuario con su respectiva contraseña:
~~~
vagrant@servidor:~$ sudo htdigest -c /etc/apache2/claves/passwdDigest.txt Contraseña paloma
Adding password for usuario1 in realm Contraseña.
New password: 
Re-type new password: 
~~~

Y se restaura apache2.

La cabecera del mensaje HTTP es la siguiente:
~~~
    servidor-19ac.http > coatlicue-73df.45012: Flags [P.], cksum 0xda3c (incorrect -> 0x660f), seq 613:1008, ack 1347, win 495, options [nop,nop,TS val 4117658060 ecr 356853645], length 395: HTTP, length: 395
	HTTP/1.1 200 OK
	Date: Thu, 24 Oct 2019 18:00:26 GMT
	Server: Apache/2.4.38 (Debian)
	Authentication-Info: rspauth="667a9d11218432b725019b228d6cbe8f", cnonce="02092357cd4b6860", nc=00000002, qop=auth
	Last-Modified: Wed, 23 Oct 2019 06:41:01 GMT
	ETag: "0-5958e33466792"
	Accept-Ranges: bytes
	Content-Length: 0
	Keep-Alive: timeout=5, max=99
	Connection: Keep-Alive
	Content-Type: text/html
~~~

![Imagen8](images/Img_tarea8.png)


#### Tarea 9: Vamos a combinar el control de acceso (tarea 6) y la autentificación (tareas 7 y 8), y vamos a configurar el virtual host para que se comporte de la siguiente manera: el acceso a la URL departamentos.iesgn.org/secreto se hace forma directa desde la intranet, desde la red pública te pide la autentificación. Muestra el resultado al profesor.

>Configuración con .htaccess

Configuración de departamentos.conf:
~~~
<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        DocumentRoot /srv/www/departamentos
        ServerName departamentos.iesgn.org
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
	<Directory "/srv/www/departamentos/intranet/">
		Require ip 192.168.100
	</Directory>      
	<Directory "/srv/www/departamentos/internet/">
                <RequireAll>
                        Require all granted
                        Require not ip 192.168.100
                </RequireAll>
        </Directory>
        <Directory "/srv/www/departamentos/secreto/">
                AuthUserFile "/etc/apache2/claves/passwdDigest.txt"
                AuthName "Contraseña"
                AuthType Digest
                Require valid-user
                <RequireAll>
                        Require all granted
                        Require ip 192.168.100
                </RequireAll>
        </Directory>
</VirtualHost>
~~~

Desde la intranet entra sin pedir nada y desde internet si:
~~~




Commands: Use arrow keys to move, '?' for help, 'q' to quit, '<-' to go b
  Arrow keys: Up and Down to move.  Right to follow a link; Left to go bac
 H)elp O)ptions P)rint G)o M)ain screen Q)uit /=search [delete]=history l
~~~

![Imagen9](images/Img_tarea9.png)


## Proveedor de hosting
**Date de alta en un proveedor de hosting. Si necesitamos configurar el servidor web que han configurado los administradores del proveedor, ¿qué podemos hacer? Explica la directiva AllowOverride de apache2. Utilizando archivos .htaccess realiza las siguientes configuraciones:**

El hosting que hoy a utilizar es 10hosting. Y se crea en los siguientes pasos:
1. Nombre: palomapache2.x10host.com
2. Correo
3. Contraseña
4. Ver los terminos y condiciones
5. Añadir nuestro nombre
6. Correo de confirmación

#### Tarea 10: Habilita el listado de ficheros en la URL http://host.dominio/nas.

En el fichero .htaccess se añade para que salga el listado de ficheros:
~~~
Options +Indexes
~~~

![Imagen10](images/Img_tarea10.png)
![Imagen10B](images/Img_tarea10B.png)

#### Tarea 11: Crea una redirección permanente: cuando entremos en http://host.dominio/google salte a www.google.es.

En .htaccess se añade:
~~~
RedirectMatch permanent /google http://www.google.es
~~~


#### Tarea 12: Pedir autentificación para entrar en la URL http://host.dominio/prohibido. (No la hagas si has elegido como proveedor CDMON, en la plataforma de prueba no funciona.)

Crea un nuevo .htaccess en el directorio /prohibido con:
~~~
AuthType Basic
AuthName "contraseña"
AuthUserFile "/home/palomap2/.htpasswds/public_html/prohibido/passwd"
require valid-user
~~~

Y en /home/palomap2/.htpasswds/public_html/prohibido/passwd se crean las contraseñas con los usuarios correspondientes.



## Módulos

#### Tarea 13: Módulo userdir: Activa y configura el módulo userdir, que permite que cada usuario del sistema tenga la posibilidad de tener un directorio (por defecto se llama public_html) donde alojar su página web. Publica una página de un usuario, y accede a la misma. Esta tarea la tienes que hacer en tu servidor.

Se crea el directorio **public_html**:
~~~
vagrant@servidor:/srv/www/departamentos/secreto$ mkdir /home/vagrant/public_html
vagrant@servidor:/srv/www/departamentos/secreto$ chmod 0755 /home/vagrant/public_html/
~~~

Se configura **/etc/apache2/mods-enabled/userdir.conf**:
~~~
<IfModule mod_userdir.c>
    UserDir public_html
    UserDir disabled root

    <Directory /home/*/public_html>
        AllowOverride FileInfo AuthConfig Limit Indexes
        Options MultiViews Indexes SymLinksIfOwnerMatch IncludesNoExec
        <Limit GET POST OPTIONS>
            Require all granted
        </Limit>
        <LimitExcept GET POST OPTIONS>
            Require all denied
        </LimitExcept>
    </Directory>
</IfModule>
~~~


Se ejecuta el módulo y Apache2:
~~~
vagrant@servidor:~$ sudo a2enmod userdir
Enabling module userdir.
To activate the new configuration, you need to run:
  systemctl restart apache2
vagrant@servidor:~$ sudo systemctl restart apache2
~~~

![Imagen13](images/Img_tarea13.png)



#### Tarea 14: En tu servidor crea una carpeta php donde vamos a tener un fichero index.php con el siguiente contenido:

~~~
      <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
      <html xmlns="http://www.w3.org/1999/xhtml">
      <head>
      <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
      <title>Conversor de Monedas</title>
      </head>

      <body>
      <form action="index.php" method="get">
         	<input type="text" size="30" name="monto" /><br/>
          <select name="pais">
              <option name="Dolar">Dolar</option>
              <option name="Libra">Libra</option>
              <option name="Yen">Yen</option>
          </select>
          <input type="submit" value="convertir" />
         </form>
      <?php
          // averiguamos si se ha introducido un dinero
          if (isset($_GET['monto'])) {
            define ("cantidad", $_GET['monto']);
          } else {
       	  define ("cantidad", 0);
          }
          if($_GET){
          // definimos los países
          $tasacambios = array ("Libra"=>0.86,"Dolar"=>1.34,"Yen"=>103.56);
          // imprimimos el monto ingresado
          echo "<b>".cantidad." euros</b><br/> ".$_GET["pais"]." = ".cantidad*$tasacambios[$_GET["pais"]]." <br><br>";
          // por cada país imprimimos el cambio
          }
         ?>

      </body>
      </html>
~~~

> Prueba la página utilizando parámetros en la URL (parámetros GET), por ejemplo: http://nombre_página/php/index.php?monto=100&pais=Libra

> Configura mediante un fichero .htaccess, la posibilidad de acceder a la URL http://nombre_página/php/moneda/cantidad, donde moneda indica el nombre de la moneda a la que queremos convertir (Dolar,Libra,Yen) y cantidad indica los euros que queremos convertir.

Se instala el módulo php de Apache2:
~~~
vagrant@servidor:~$ sudo apt install libapache2-mod-php
~~~

Configuración de la nueva dirección:
~~~
vagrant@servidor:/var/www/html$ sudo mkdir php
vagrant@servidor:/var/www/html/php$ sudo touch index.php
vagrant@servidor:/var/www/html$ sudo chown -R www-data:www-data php/
vagrant@servidor:/var/www/html$ sudo cp /etc/apache2/sites-available/iesgn.conf /etc/apache2/sites-available/php.conf
~~~

Configuración php.conf:
~~~
<VirtualHost *:80>

        ServerName www.nombre_pagina.org
        DocumentRoot /var/www/html/
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
~~~

~~~
vagrant@servidor:/var/www/html$ sudo a2ensite php
Enabling site php.
To activate the new configuration, you need to run:
  systemctl reload apache2
vagrant@servidor:/var/www/html$ sudo systemctl reload apache2.service
~~~

Se activa el modo rewerite:
~~~
vagrant@servidor:~$ sudo a2enmod rewrite
~~~

Y se configura el fichero /var/www/html/php/.htaccess 
~~~
Options FollowSymLinks
RewriteEngine On
RewriteBase /
RewriteRule ^([0-9]+)/([a-zA-Z]+)$ index.php?monto=$1&pais=$2
~~~

![Imagen14](images/Img_tarea14.png)



## **IPv6**

    Tarea 15 (1 punto): Comprueba que el servidor web con la configuración por defecto está escuchando por el puerto 80 en ipv6.
    Tarea 16 (1 punto): Configura la máquina para que tenga una ipv6 global. Activa el virtualhost por defecto y accede a la página principal utilizando la ipv6 global que tiene asignada.
    Tarea 17 (1 punto): Configura la resolución estática para acceder a los virtualhost utilizando ipv6.


