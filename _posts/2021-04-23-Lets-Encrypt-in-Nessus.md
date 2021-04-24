---
title: ¿Cómo instalar Let's Encrypt en Tenable Nessus?
published: true
date: 2021-04-23
img: cabin.png
alt: image-alt
project-date: Apr 2021
client: Start Bootstrap
category: Tools
---
## [](#header-2)Previos

Luego de instalar y configurar nuestro **Tenable Nessus**, parte de las configuraciones de seguridad que debemos realizar es configurar de manera el acceso vía la consola web, configurando nuestros certificados confiables y **Tenable Nessus** no muestre el error de Certificados ya conocido.
Estamos asumiendo que Nessus Professional está instalado sobre un Sistema Operativo Linux, sin embargo sería una lógica similar para un Sistema Operativo basado en Windows. Tambien se asume que ya tienes configurado el dominio o subdominio para que resuelva a la dirección IP donde se tiene instalado Tenable Nessus.

![](assets\posts\nessus_img1.png)

## [](#header-2)Instalar Certbot

En este caso, como va a depender de la distribución sobre la que se quiera instalar el `Certbot` por lo cual les recomendamos revisar la web oficial para tener las especificaciones por cada caso: 

[Link a la Web de Certbot](https://certbot.eff.org/).

## [](#header-2)Generar nuestro certificado
La forma que vamos a utilizar es generarlos en modo standalone y luego moverlos a la carpeta donde Tenable Nessus busca los certificados.

Lo primero en hacer es lo siguiente: 
```bash
certbot certonly --standalone --preferred-challenges http -d demo.foo.bar
```
En este comando le indicamos a certbot que solo nos cree los certificados y que haga las validaciones levantando un servidor http para realizar las validaciones sobre el dominio /subdominoi al que le queremos generar los certificados.
Como salida a dicho comando tendremos algo como esto : 
* * *
![](assets\posts\nessus_img2.png)
* * *

Si todo nos fue bien con el comando ya tenemos casi todo listo.


## [](#header-2)Configurando Certificados en Tenable Nessus
Ahora que ya tenemos los certificados generados vamos a ejecutar lo siguiente:

```bash
sudo service nessusd stop
sudo cp -i /etc/letsencrypt/live/demo.foo.bar/fullchain.pem /opt/nessus/com/nessus/CA/servercert.pem
sudo cp -i /etc/letsencrypt/live/demo.foo.bar/privkey.pem /opt/nessus/var/nessus/CA/serverkey.pem
sudo cp -i /etc/letsencrypt/live/demo.foo.bar/cert.pem /opt/nessus/com/nessus/CA/cacert.pem
sudo service nessusd start
```
Básicamente detenemos el servicio de  Nessus y reemplazamos los certificados predefinidos por los certificado generados con `Certbot`
* * *
![](assets\posts\nessus_img3.png)
* * *


## [](#header-2)Renovando Certificados en Tenable Nessus
Hasta el paso previo, ya tendriamos todo ok y funcionando, pero no olvidemos que los certificados emitidos por Let's Encrypt tienen una caducidad bastante corta. Con lo cual tenemos dos opciones:
*   ~~Hacer este procedimiento cada 3 meses de manera manual~~.
*   Activar el autorenew . 

Vamos por esa última opción :
```bash
certbot renew --dry-run
```

Luego de eso veran que en `/etc/letsencrypt/renewal/` se ha creado un archivo con detalles sobre la generación del vertificado

Nosotros debemos generar 2 archivos en las siguientes carpetas : `/etc/letsencrypt/renewal-hooks/pre` y `/etc/letsencrypt/renewal-hooks/post`, como los nombres nos deben dar a entender, ahí se pueden agregar scripts para que sean ejecutados "pre" y "post" la generación de los certificados. 

En PRE pondremos: 
```bash
sudo sh -c 'printf "#!/bin/sh\nservice nessusd stop\n" > /etc/letsencrypt/renewal-hooks/pre/nessus.sh'
sudo chmod 755 /etc/letsencrypt/renewal-hooks/pre/nessus.sh
```
En POST pondremos:

```bash
sudo sh -c 'printf "#!/bin/sh" >> /etc/letsencrypt/renewal-hooks/post/nessus.sh'
sudo sh -c 'printf "cp -i /etc/letsencrypt/live/demo.foo.bar/fullchain.pem /opt/nessus/com/nessus/CA/servercert.pem\n" >> /etc/letsencrypt/renewal-hooks/post/nessus.sh'
sudo sh -c 'printf "cp -i /etc/letsencrypt/live/demo.foo.bar/privkey.pem /opt/nessus/var/nessus/CA/serverkey.pem\n" >> /etc/letsencrypt/renewal-hooks/post/nessus.sh'
sudo sh -c 'printf "cp -i /etc/letsencrypt/live/demo.foo.bar/cert.pem /opt/nessus/com/nessus/CA/cacert.pem\n" >> /etc/letsencrypt/renewal-hooks/post/nessus.sh'
sudo sh -c 'printf "service nessusd start\n" >> /etc/letsencrypt/renewal-hooks/post/nessus.sh'
sudo chmod 755 /etc/letsencrypt/renewal-hooks/post/nessus.sh
```

Con esto, los certificados emitidos por Let`s Encrypt deberían quedar configurados en Tenable Nessus. 

```
think outside the box
```
