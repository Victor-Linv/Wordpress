# Wordpress

## 1. Requisitos necesarios
Una instancia de Ubuntu Server en AWS EC2.
Conexión SSH a dicha instancia.
Apache instalado.
PHP instalado.
Un servidor MySQL o MariaDB accesible.
Un nombre de dominio que apunte a la IP pública de la instancia.
Los puertos 22, 80 y 443 abiertos en el grupo de seguridad de AWS.

## 2. Organización de archivos
text
.
├── README.md
├── conf/
│   └── 000-default.conf
├── htaccess/
│   └── .htaccess
└── scripts/
    ├── .env
    ├── install_lamp.sh
    ├── setup_letsencrypt_https.sh
    ├── deploy_wordpress.sh
3. Variables de entorno

Los scripts leen sus parámetros de un archivo .env.

Ejemplo de contenido:

bash
WORDPRESS_DB_NAME=wordpress
WORDPRESS_DB_USER=wordpress_user
WORDPRESS_DB_PASSWORD=wp_wordpress
WORDPRESS_DB_HOST=localhost

IP_CLIENTE_MYSQL=localhost

CERTIFICATE_DOMAIN=midominio.com
CERTIFICATE_EMAIL=correo@example.com

Significado de cada variable:

WORDPRESS_DB_NAME: nombre que tendrá la base de datos de WordPress.
WORDPRESS_DB_USER: usuario con acceso a esa base de datos.
WORDPRESS_DB_PASSWORD: contraseña de dicho usuario.
WORDPRESS_DB_HOST: dirección del servidor donde está la base de datos.
IP_CLIENTE_MYSQL: dirección desde la que el usuario de MySQL puede conectarse.
CERTIFICATE_DOMAIN: dominio para el que se pide el certificado.
CERTIFICATE_EMAIL: correo vinculado al certificado.

Conviene limitar los permisos del archivo .env:

bash
chmod 600 .env
4. Descripción de los scripts
-install_lamp.sh

Se encarga de instalar y preparar todo lo necesario para ejecutar WordPress:

Apache.
PHP.
MySQL o MariaDB.
Las extensiones de PHP que WordPress requiere.
-setup_letsencrypt_https.sh

Instala Certbot junto con su plugin para Apache.

Pide un certificado SSL/TLS para el dominio indicado en .env, activa los módulos SSL y Rewrite de Apache y deja configurada la redirección automática de HTTP a HTTPS.

Además, verifica que la renovación automática del certificado funcione.

-deploy_wordpress.sh

Instala WordPress directamente en la carpeta raíz de Apache:

text
/var/www/html

Los pasos que realiza son:

Descargar WordPress desde el sitio oficial.
Descomprimir el paquete.
Copiar los archivos a la carpeta raíz de Apache.
Crear la base de datos y el usuario de WordPress.
Generar y ajustar el archivo wp-config.php.
Activar los enlaces permanentes a través de .htaccess.
Aplicar los permisos adecuados.
Reiniciar Apache.
5. Cómo ejecutar los scripts

Lo primero es entrar en la carpeta de scripts:

bash
cd scripts

Después se dan permisos de ejecución:

bash
chmod +x setup_letsencrypt_https.sh
chmod +x deploy_wordpress.sh

Se activa HTTPS:

bash
./setup_letsencrypt_https.sh

Y se instala WordPress:

bash
./deploy_wordpress.sh
6. Ajustes de Apache

Apache tiene que admitir archivos .htaccess, algo que se consigue con esta directiva:

apache
<Directory /var/www/html>
    AllowOverride All
    Require all granted
</Directory>

Asimismo, el módulo Rewrite debe estar activo:

bash
sudo a2enmod rewrite
sudo systemctl restart apache2
7. Ajustes en AWS

El grupo de seguridad de la instancia EC2 debe admitir el siguiente tráfico de entrada:

Puerto	Protocolo	Finalidad
22	TCP	Conexión SSH
80	TCP	Tráfico HTTP
43	TCP	Tráfico HTTPS

El dominio que se use tiene que resolver a la IP pública de la instancia EC2.

8. Verificación del despliegue

Para ver si Apache está funcionando:

bash
sudo systemctl status apache2

Para consultar los certificados instalados:

bash
sudo certbot certificates

Para simular la renovación del certificado:

bash
sudo certbot renew --dry-run

Para confirmar que WordPress funciona, basta con abrir en el navegador:

text
https://midominio.com

Si WordPress se instala en un subdirectorio propio, sus archivos quedan en /var/www/html/wordpress, aunque se puede acceder a él desde el dominio raíz siempre que WP_HOME esté configurado en ese sentido.