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


![](assets\posts\nessus_img1.png)

## [](#header-2)Previos

Luego de instalar y configurar nuestro **Tenable Nessus**, parte de las configuraciones de seguridad que debemos realizar es configurar de manera el acceso vía la consola web, configurando nuestros certificados confiables y **Tenable Nessus** no muestre el error de Certificados ya conocido.
Estamos asumiendo que Nessus Professional está instalado sobre un Sistema Operativo Linux, sin embargo sería una lógica similar para un Sistema Operativo basado en Windows. Tambien se asume que ya tienes configurado el dominio o subdominio para que resuelva a la dirección IP donde se tiene instalado Tenable Nessus.

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

![](assets\posts\nessus_img2.png)

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
![](assets\posts\nessus_img3.png)


## [](#header-2)Renovando Certificados en Tenable Nessus
Hasta el paso previo, ya tendriamos todo ok y funcionando, pero no olvidemos que los certificados emitidos por Let's Encrypt tienen una caducidad bastante corta. Con lo cual tenemos dos opciones:
*   ~~Hacer este procedimiento cada 3 meses de manera manual~~.
*   Activar el autorenew . 

Vamos por esa última opción :
```bash
certbot renew --dry-run
```

Luego de eso :

```bash

/etc/letsencrypt/renewal-hooks/pre
sudo sh -c 'printf "#!/bin/sh\nservice nessusd stop\n" > /etc/letsencrypt/renewal-hooks/pre/nessus.sh'
sudo chmod 755 /etc/letsencrypt/renewal-hooks/pre/nessus.sh
```
Y claro tambien 
```bash
/etc/letsencrypt/renewal-hooks/post

sudo sh -c 'printf "#!/bin/sh" >> /etc/letsencrypt/renewal-hooks/post/nessus.sh'
sudo sh -c 'printf "cp -i /etc/letsencrypt/live/demo.foo.bar/fullchain.pem /opt/nessus/com/nessus/CA/servercert.pem\n" >> /etc/letsencrypt/renewal-hooks/post/nessus.sh'
sudo sh -c 'printf "cp -i /etc/letsencrypt/live/demo.foo.bar/privkey.pem /opt/nessus/var/nessus/CA/serverkey.pem\n" >> /etc/letsencrypt/renewal-hooks/post/nessus.sh'
sudo sh -c 'printf "cp -i /etc/letsencrypt/live/demo.foo.bar/cert.pem /opt/nessus/com/nessus/CA/cacert.pem\n" >> /etc/letsencrypt/renewal-hooks/post/nessus.sh'
sudo sh -c 'printf "service nessusd start\n" >> /etc/letsencrypt/renewal-hooks/post/nessus.sh'
sudo chmod 755 /etc/letsencrypt/renewal-hooks/post/nessus.sh
```

 This is an unordered list following a header.
 can be **bold**, _italic_, ~~strikethrough~~ or `keyword`.

[Para un mayor Link to another page](another-page).

There should be whitespace between paragraphs.

There should be whitespace between paragraphs. We recommend including a README, or a file with information about your project.

# [](#header-1)Header 1

This is a normal paragraph following a header. GitHub is a code hosting platform for version control and collaboration. It lets you and others work together on projects from anywhere.

## [](#header-2)Header 2

> This is a blockquote following a header.
>
> When something is important enough, you do it even if the odds are not in your favor.

### [](#header-3)Header 3



This is a normal paragraph following a header. GitHub is a code hosting platform for version control and collaboration. It lets you and others work together on projects from anywhere.

```ruby
# Ruby code with syntax highlighting
GitHubPages::Dependencies.gems.each do |gem, version|
  s.add_dependency(gem, "= #{version}")
end
```

#### [](#header-4)Header 4

*   This is an unordered list following a header.
*   This is an unordered list following a header.
*   This is an unordered list following a header.

##### [](#header-5)Header 5

1.  This is an ordered list following a header.
2.  This is an ordered list following a header.
3.  This is an ordered list following a header.

###### [](#header-6)Header 6

| head1        | head two          | three |
|:-------------|:------------------|:------|
| ok           | good swedish fish | nice  |
| out of stock | good and plenty   | nice  |
| ok           | good `oreos`      | hmm   |
| ok           | good `zoute` drop | yumm  |

### There's a horizontal rule below this.

* * *

### Here is an unordered list:

*   Item foo
*   Item bar
*   Item baz
*   Item zip

### And an ordered list:

1.  Item one
1.  Item two
1.  Item three
1.  Item four

### And a nested list:

- level 1 item
  - level 2 item
  - level 2 item
    - level 3 item
    - level 3 item
- level 1 item
  - level 2 item
  - level 2 item
  - level 2 item
- level 1 item
  - level 2 item
  - level 2 item
- level 1 item

### Small image

![](https://assets-cdn.github.com/images/icons/emoji/octocat.png)

### Large image

![](https://guides.github.com/activities/hello-world/branching.png)


### Definition lists can be used with HTML syntax.

<dl>
<dt>Name</dt>
<dd>Godzilla</dd>
<dt>Born</dt>
<dd>1952</dd>
<dt>Birthplace</dt>
<dd>Japan</dd>
<dt>Color</dt>
<dd>Green</dd>
</dl>

```
Long, single-line code blocks should not wrap. They should horizontally scroll if they are too long. This line should be long enough to demonstrate this.
```

```
The final element.
```
