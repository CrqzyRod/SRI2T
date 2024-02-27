# Juan Manuel González Márquez - SRI - 2ASIR
## Servidor de Alojamiento Web
### Introducción.
El proyecto del segundo trimestre en el módulo de redes del ciclo formativo de grado superior de ASIR se enfoca en proporcionar un servicio integral de alojamiento web. Se dará soporte a páginas estáticas y dinámicas con PHP, asignando a cada cliente un directorio de usuario que incluirá una página web por defecto. Además, se facilitará el manejo de bases de datos SQL a través de phpMyAdmin. Para la administración de archivos, se implementará el acceso mediante FTP con TLS y SSH/SFTP. Asimismo, se configurarán de manera adecuada Postfix y Dovecot para el envío y gestión de correos electrónicos, asegurando un entorno completo y seguro para los usuarios.
### Requisitos previos.
Antes de comenzar con la explicación del script, indicaremos los requisitos necesarios para poder lanzar dicho script.
#### Instalaciones:
- Apache: sudo apt-get install apache2

![imagen](https://github.com/CrqzyRod/SRI2T/assets/122454007/dc8dbe2b-f739-4cdc-8e0d-003def38c2a5)

- Dns (bind9):

![imagen](https://github.com/CrqzyRod/SRI2T/assets/122454007/d81ebfcc-db5d-4cd1-b8ef-fe8ce89f9234)

- Php: librerías de php para apache: sudo apt-get install php libapache2-mod-php

![imagen](https://github.com/CrqzyRod/SRI2T/assets/122454007/027e9cd5-7c46-4146-a875-3e75a8065bc5)

- MySqlServer: sudo apt install mysql-server

![imagen](https://github.com/CrqzyRod/SRI2T/assets/122454007/1cd272a6-afe4-4c69-bace-f29339412583)

- PhPMyAdmin: sudo apt install phpmyadmin

![imagen](https://github.com/CrqzyRod/SRI2T/assets/122454007/f680971d-153e-4e35-af3e-1df74cb13233)

- Python: módulo de python3 para apache

![imagen](https://github.com/CrqzyRod/SRI2T/assets/122454007/bddd8ece-f359-4b46-afd2-7404f99b55cf)

- ProFtpd:

![imagen](https://github.com/CrqzyRod/SRI2T/assets/122454007/a4249d4f-e46e-450c-88d3-b9eaa1b9abcd)

- OpenSsl:

![imagen](https://github.com/CrqzyRod/SRI2T/assets/122454007/66894870-bc5f-4995-b5c8-c00594347848)

- Filezilla:

![imagen](https://github.com/CrqzyRod/SRI2T/assets/122454007/0c205d6a-5e05-454a-a2eb-9520cd61945e)

- Postfix:

![imagen](https://github.com/CrqzyRod/SRI2T/assets/122454007/f5d47abd-cd8b-40f6-ba6e-54a9b9fa26f8)

- Dovecot:
  
![imagen](https://github.com/CrqzyRod/SRI2T/assets/122454007/2e7a2eb4-ee4e-4c9a-9d2a-cc8fa6e0d5aa)

#### Configuraciones.

Generar certificado ssl: 

![imagen](https://github.com/CrqzyRod/SRI2T/assets/122454007/b82f0fe8-e171-4129-bbbb-772e637270fa)

Configuración de filezilla para entrar usando el certificado ssl:

![imagen](https://github.com/CrqzyRod/SRI2T/assets/122454007/c2419953-927e-4615-b3c1-87da7cd525be)

![imagen](https://github.com/CrqzyRod/SRI2T/assets/122454007/712d6aab-2f62-49df-8faf-e97b55dcf9d5)

Configuración postfix

![imagen](https://github.com/CrqzyRod/SRI2T/assets/122454007/bc4b6c23-cef5-4ece-8b30-f9c74622fd83)

![imagen](https://github.com/CrqzyRod/SRI2T/assets/122454007/8655aae5-392a-4e27-9588-f2f1b8ca00ef)

![imagen](https://github.com/CrqzyRod/SRI2T/assets/122454007/40a67db0-a7bf-4694-8957-bc3d317cc30f)

![imagen](https://github.com/CrqzyRod/SRI2T/assets/122454007/3b39a732-06af-492e-a767-dc40a6bc85aa)

![imagen](https://github.com/CrqzyRod/SRI2T/assets/122454007/298b259c-17b7-4182-a8ff-c7a237c3d24e)

Configuración para permitir los protocolos pop3 e imap en dovecot.

![imagen](https://github.com/CrqzyRod/SRI2T/assets/122454007/82752456-b3dc-4d89-bc15-a7c2b56e14a0)

Nos aseguramos que dovecot tenga activado ssl.

![imagen](https://github.com/CrqzyRod/SRI2T/assets/122454007/7b7ef802-576c-42c4-856c-77cd1e710d60)

### Descripción del script.
```bash
#!/bin/bash

usuario=$1
contra=$2
ip=$3
dominio="${usuario}.marisma.local"

if [ $# -ne 3 ]; then
    echo "Error: Debes proporcionar exactamente 3 argumentos - nombre, contraseña e IP."
    exit 1
fi


# Variables para las rutas de zonas
zona_directa="/etc/bind/zones/db.marisma.local"
zona_inversa="/etc/bind/zones/db.192.168.195"

# Creación del directorio de trabajo
sudo useradd -m -d /home/$usuario -s /bin/bash $usuario

# Configuración de la contraseña del usuario
echo "$usuario:$contra" | sudo chpasswd

# Configuración subdominio DNS
#Zona directa
echo "\$ORIGIN ${dominio}." >> $zona_directa
echo "@ IN A ${ip}" >> $zona_directa
echo "www IN A ${ip}" >> $zona_directa
#Zona inversa
echo "${ip} IN PTR ns.${dominio}." >> $zona_inversa

service apache2 reload > /dev/null
service bind9 reload > /dev/null
service proftpd reload > /dev/null

# Configuración del host virtual en Apache
ruta_a="/etc/apache2/sites-available/${dominio}.conf"
ruta_e="/etc/apache2/sites-enabled/${dominio}"
document_root="/var/www/html/$usuario"
dir_python="${document_root}/python-web"
app_python="${dir_python}/mypythonapp"
public_python="${dir_python}/public_html"
controller_py="${app_python}/controller.py"

#Añadir el dominio al fichero hosts

echo "${ip} ${dominio}" >> /etc/hosts
echo "127.0.0.1 ${dominio}" >> /etc/hosts

mkdir $document_root
mkdir $dir_python
mkdir $app_python
mkdir $public_python

#Crear fichero python

touch $controller_py
echo "# -*- coding: utf-8 -*-
def application(environ,start_response):
status= '200 OK'
html = 'Página del usuario ${usuario} python'
html = bytes(html,encoding='utf-8')
response_header = [('Content-type','text/html')]
start_response(status,response_header)
yield html" > ${controller_py}

# Crear el archivo de configuración del host virtual
echo "<VirtualHost *:80>
ServerAdmin admin@$dominio
ServerName $dominio
WSGIScriptAlias / $controller_py
DocumentRoot $public_python
<Directory />
Options FollowSymLinks
AllowOverride all
</Directory>
ErrorLog /var/log/apache2/$dominio.errorLog.log
CustomLog /var/log/apache2/$dominio.customLog.log combined
</VirtualHost>" | sudo tee $ruta_a > /dev/null

# Verificar la configuración de Apache
apache2ctl configtest

# Habilitar el sitio y reiniciar Apache si la verificación es exitosa
if [ $? -eq 0 ]; then
    a2ensite $dominio > /dev/null
    systemctl restart apache2
else
    echo "Error en la configuración de Apache. No se ha habilitado el sitio."
fi

# Configuración base de datos
mysql -u root -e "CREATE DATABASE $usuario;"
mysql -u root -e "CREATE USER '$usuario'@'localhost' IDENTIFIED BY  '$contra';"
mysql -u root -e "GRANT ALL PRIVILEGES ON $usuario.* TO '$usuario'@'localhost';"
mysql -u root -e "FLUSH PRIVILEGES;"

echo "El usuario ${usuario} ha sido creado con exito"
```
