## 1. Instalación

```
sudo apt install apache2
sudo service apache2 start
```

## 2. Configuración

```
sudo cp /etc/apache2/apache2.conf /etc/apache2/apache2.conf.backup 
sudo gedit /etc/apache2/apache2.conf
```

### 2.1 Desactivar despliegue de versión de Apache

**Agregar al final del archivo de configuración**

`
ServerSignature Off
ServerTokens Prod
`

### 2.2 Desactivar visualización de directorios (localhost/test)

`
<Directory /var/www/html>
	Options -Indexes
</Directory>
`

### 2.3 Crear nuevo usuario sin privilegios

```
sudo groupadd <grupo>
sudo useradd -d /var/www/ -g <grupo> -s /bin/no-login <user>
```

**En apache2.conf comentar grupo y usuario por defecto y añadir**

`
User <usuario>
Group <grupo>
`

### 2.4 Configurar https con SSL

**Instalamos openssl y activamos modulo ssl**

```
sudo apt install openssl
sudo a2enmod ssl
```

**Generamos una llave, una solicitud de firma de certificado y autofirmamos el certificado**

```
openssl genrsa -des3 -out ca.key 1024
openssl req -new -key ca.key -out ca.csr
openssl x509 -req -days 365 -in ca.csr -signkey ca.key -out ca.crt
```

**Movemos los certificados a la carpeta indicada**

```
sudo mv ca.crt /etc/pki/tls/certs/ca.crt
sudo mv ca.key /etc/pki/tls/certs/ca.key
sudo mv ca.csr /etc/pki/tls/certs/ca.csr
```

**Escuchamos peticiones https en el puerto 443**

```
NameVirtualHost *:443
<VirtualHost *:443>
        SSLEngine on
        SSLCertificateFile /etc/pki/tls/certs/ca.crt
        SSLCertificateKeyFile /etc/pki/tls/certs/ca.key
        # SSLCertificateChainFile /etc/pki/tls/certs/certificadoExterno.crt
        DocumentRoot /var/www/html
        # ServerName redes3.com
</VirtualHost>
```

**Reiniciamos el servidor**

```
sudo service apache2 restart
```


## Fuentes
- [Como configurar HTTPS con openSSL](http://www.linuxhispano.net/2011/02/21/configurar-soporte-https-en-apache/)
- [Tips para mejorar la seguridad en apache](https://www.tecmint.com/apache-security-tips/)
