---
title: Configurando los certificados emitidos por Let's Encrypt en Burp Suite Enterprise
published: true
date: 2021-04-24
img: cabin.png
alt: image-alt
project-date: Apr 2021
client: Start Bootstrap
category: Tools
---

## [](#header-2)Previos

`Esto no se trata de una guía oficial, es algo que nos ha funcionado y esperamos que sea de utilidad.
Run it at your own risk`

Luego de instalar y configurar nuestro **Burp Suite Enterprise**, es recomendable configurar el acceso vía la consola web con certificados de una CA válida y **Burp Suite Enterprise** no muestre el error de Certificados ya conocido.

## [](#header-2)Instalar Certbot

En este caso, dependerá de la distribución sobre la que se quiera instalar el `Certbot`, por lo cual les recomendamos revisar la web oficial para tener las especificaciones por cada caso: 

[Link a la Web de Certbot](https://certbot.eff.org/).

## [](#header-2)Generar nuestro certificado
Vamos a generar los certificados de la misma forma que hicimos en el [Post de Tenable Nessus](Lets-Encrypt-in-Nessus). (En modo standalone).

Lo primero en hacer es lo siguiente: 
```bash
certbot certonly --standalone --preferred-challenges http -d demo.foo.bar
```
En este comando le indicamos a certbot que solo nos genere los certificados necesarios y que haga las validaciones levantando un servidor http sobre el dominio/subdominio al cual le queremos generar los certificados.

Como salida a dicho comando tendremos algo como esto : 
* * *
![](assets\posts\nessus_img2.png)
* * *

Si todo nos fue bien con el comando ya tenemos casi todo listo.


## [](#header-2)Generando el PKCS#12
Ahora que ya tenemos los certificados generados vamos a ejecutar el siguiente comando para convertir los .PEM a un PKCS#12 :

```bash
openssl pkcs12 -export -out certificate.p12 -inkey /etc/letsencrypt/live/demo.foo.bar/privkey.pem -in /etc/letsencrypt/live/demo.foo.bar/cert.pem -certfile /etc/letsencrypt/live/demo.foo.bar/chain.pem
```
Luego de ejecutarlo nos pedirá que ingresemos una contraseña, acto seguido nos pide volver a ingresarla para verificar que todo funciona correctamente.

## [](#header-2)Configurando Burp Suite Enterprise
Luego de esto seguimos el proceso usual para subir nuestro PKCS#12: `**RECORDAR QUE ESTO REINICIARA EL SERVICIO WEB**`


![](assets\posts\burp_img00.png)

![](assets\posts\burp_img3.png)

![](assets\posts\burp_img1.png)

## [](#header-3)`Listo!`

![](assets\posts\burp_img5.png)


## [](#header-2)Renovando Certificados 
Con eso ya tenemos todo ok con el certificado. No olvidemos que los certificados emitidos por Let's Encrypt tienen una caducidad bastante corta. Con lo cual tenemos dos opciones:
*   ~~Hacer este procedimiento cada 30 días de manera manual~~.
*   Activar el autorenew. 

Vamos por esa última opción :
```bash
certbot renew --dry-run
```
Luego de eso veran que en `/etc/letsencrypt/renewal/` se ha creado un archivo con detalles sobre la generación del certificado

Nosotros debemos generar 1 archivos en las siguiente carpeta : `/etc/letsencrypt/renewal-hooks/post`  como el nombre nos da a entender, ahí se pueden agregar scripts para que sean ejecutados  "post" la generación de los certificados. 

En POST pondremos:

`CUIDADO!! Estamos hardcodeando la clave para "automatizarlo (No recomendado)"`
```bash
sudo sh -c 'printf "#!/bin/sh" >> /etc/letsencrypt/renewal-hooks/post/burp_enterprise.sh'
sudo sh -c 'printf "openssl pkcs12 -export -out certificate.p12 -inkey /etc/letsencrypt/live/demo.foo.bar/privkey.pem -in /etc/letsencrypt/live/demo.foo.bar/cert.pem -certfile /etc/letsencrypt/live/demo.foo.bar/chain.pem -password pass:TuClaveSuperSegura\n" >> /etc/letsencrypt/renewal-hooks/post/burp_enterprise.sh'
sudo sh -c 'printf "mail -s \"Certificado PKCS#12 Generado\" -t john@layer8.pe \n" >> /etc/letsencrypt/renewal-hooks/post/burp_enterprise.sh'
```
Basicamente creamos un PKCS12 y luego enviamos un correo para que nos avise y podamos hacer la parte manual.

Luego le damos permisos a los archivos: 
```bash
sudo chmod 755 /etc/letsencrypt/renewal-hooks/post/burp_enterprise.sh
```

## [](#header-2) Otras opciones!

Actualmente el archivo p12 se aloja aquí: `/var/lib/BurpSuiteEnterpriseEdition/sslCertificates/certificate.p12`.
En teoria sería generar directamente el pkcs12 cada 30 dias en esta carpeta, hemos realizado pruebas generando archivos p12 y todo funciona bien, sin embargo hasta que no renovemos el cert no podemos asegurarlo, en todo caso el script quedaría así: 

```bash
sudo sh -c 'printf "#!/bin/sh" >> /etc/letsencrypt/renewal-hooks/post/burp_enterprise.sh'
sudo sh -c 'printf "openssl pkcs12 -export -out /var/lib/BurpSuiteEnterpriseEdition/sslCertificates/certificate.p12 -inkey /etc/letsencrypt/live/demo.foo.bar/privkey.pem -in /etc/letsencrypt/live/demo.foo.bar/cert.pem -certfile /etc/letsencrypt/live/demo.foo.bar/chain.pem -password pass:TuClaveSuperSegura\n" >> /etc/letsencrypt/renewal-hooks/post/burp_enterprise.sh'
```
Luego le damos permisos a los archivos: 
```bash
sudo chmod 755 /etc/letsencrypt/renewal-hooks/post/burp_enterprise.sh
```

Nuevamente: `CUIDADO!! Estamos hardcodeando la clave para "automatizarlo (No recomendado)"`


Keep Hacking!
