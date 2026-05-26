Tecnológico de Estudios Superiores del Oriente del Estado de México
Reporte Técnico
Configuración de NGINX y PHP-FPM en AlmaLinux 9

Carrera: Ingeniería en Sistemas Computacionales
Materia: Taller de Sistemas Operativos

Integrantes:

** Dominguez Jasso Julieta
** Juarez Morales Miguel Angel
** Reyna Bello Eduardo Rodrigo

Profesor:
Gustavo Moises Romero Gonzalez

Fecha:
Mayo 2026

1. Introducción

Actualmente los servidores web requieren configuraciones más eficientes para mejorar el rendimiento y el control de los servicios utilizados. Por ello, en esta práctica se realizó la compilación manual de NGINX y PHP 8.4 en AlmaLinux 9.

La implementación se enfocó en utilizar PHP-FPM junto con comunicación mediante Sockets UNIX, permitiendo una conexión más rápida entre procesos y evitando conexiones TCP innecesarias.

Además, se configuró el inicio automático de servicios con SystemD y se resolvieron problemas relacionados con SELinux durante la ejecución del entorno.

2. Objetivo General

Implementar un entorno web funcional y optimizado utilizando NGINX y PHP-FPM en AlmaLinux 9.

3. Objetivos Específicos
Instalar dependencias necesarias para la compilación.
Configurar NGINX como servidor web.
Compilar PHP 8.4 con soporte PHP-FPM.
Implementar comunicación FastCGI mediante Sockets UNIX.
Automatizar servicios utilizando SystemD.
Resolver problemas relacionados con SELinux.
4. Desarrollo
4.1 Instalación de Dependencias

Primero se actualizó el sistema y se instalaron librerías necesarias para evitar errores durante la compilación.

sudo dnf update -y

sudo dnf groupinstall "Development Tools" -y

sudo dnf install -y wget pcre-devel zlib-devel openssl-devel \
libxml2-devel libpng-devel libjpeg-turbo-devel \
freetype-devel libwebp-devel libcurl-devel \
libicu-devel oniguruma-devel
4.2 Creación de Usuarios

Para mejorar la seguridad se crearon usuarios sin acceso al sistema.

sudo groupadd nginx

sudo useradd -g nginx -s /sbin/nologin -r nginx

sudo useradd -g nginx -s /sbin/nologin -r php
4.3 Compilación de NGINX

Se descargó y compiló el código fuente oficial de NGINX.

cd /usr/src

sudo wget http://nginx.org/download/nginx-1.26.0.tar.gz

sudo tar -xzvf nginx-1.26.0.tar.gz

cd nginx-1.26.0
Configuración
sudo ./configure \
--prefix=/srv/nginx \
--user=nginx \
--group=nginx \
--with-http_ssl_module \
--with-http_v2_module
Instalación
sudo make

sudo make install
4.4 Compilación de PHP 8.4

Posteriormente se descargó PHP con soporte para PHP-FPM.

cd /usr/src

sudo wget https://www.php.net/distributions/php-8.4.1.tar.gz

sudo tar -xzvf php-8.4.1.tar.gz

cd php-8.4.1
Configuración
sudo ./configure \
--prefix=/srv/nginx \
--enable-fpm \
--with-fpm-user=php \
--with-fpm-group=nginx \
--enable-gd \
--enable-mbstring
Compilación
sudo make -j4

sudo make install
4.5 Configuración de PHP-FPM

Archivo utilizado:

/srv/nginx/etc/php-fpm.d/www.conf

Configuración aplicada:

listen = /tmp/php84.sock
listen.owner = php
listen.group = nginx
listen.mode = 0660
4.6 Configuración de NGINX

Se agregó la comunicación FastCGI en el archivo principal de configuración.

location ~ \.php$ {

    fastcgi_pass unix:/tmp/php84.sock;

    fastcgi_index index.php;

    include fastcgi_params;

    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
}
4.7 Configuración de Servicios

Se habilitaron los servicios para iniciar automáticamente.

sudo systemctl daemon-reload

sudo systemctl enable nginx.service

sudo systemctl enable php-fpm8.4.service

sudo systemctl start nginx.service

sudo systemctl start php-fpm8.4.service
4.8 Problemas Encontrados

Durante las pruebas apareció el siguiente error:

status=203/EXEC

El problema fue causado por restricciones de SELinux.

Solución

Archivo modificado:

/etc/selinux/config

Configuración aplicada:

SELINUX=permissive
5. Resultados

Después de realizar la configuración:

NGINX respondió correctamente a peticiones HTTP.
PHP-FPM procesó archivos PHP sin errores.
La comunicación mediante Sockets UNIX funcionó correctamente.
Los servicios iniciaron automáticamente después del reinicio del sistema.
6. Conclusiones

La compilación manual de NGINX y PHP-FPM permitió crear un entorno web más optimizado y personalizado.

El uso de Sockets UNIX ayudó a mejorar la comunicación entre procesos y reducir la sobrecarga interna del sistema. Además, esta práctica permitió comprender mejor la administración de servicios Linux y la importancia de SELinux en sistemas empresariales.

7. Bibliografía

AlmaLinux Documentation
https://docs.almalinux.org/

NGINX Documentation
https://nginx.org/en/docs/

PHP Manual
https://www.php.net/manual/en/install.fpm.php
