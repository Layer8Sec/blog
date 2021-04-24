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


![](assets\posts\nessus_img.jpg)

## [](#header-2)Previos

Luego de instalar y configurar nuestro **Tenable Nessus**, parte de las configuraciones de seguridad que debemos realizar es configurar de manera correcta los certificados y dejar correctamente configurados los certificados HTTPS.
Estamos asumiendo que Nessus Professional está instalado sobre un Sistema Operativo Linux, sin embargo sería la misma lógica para un Sistema Operativo basado en Windows. Tambien se asume que ya tienes configurado el dominoi o subdominio para que resuelva a la dirección IP donde se tiene instalado Tenable Nessus.

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

![](assets\posts\nessus_img.jpg)

Si todo nos fue bien con el comando ya tenemos casi todo listo.

## [](#header-2)Configurando Certificados en Tenable Nessus
Ahora que ya tenemos los certificados generados 


```bash
sudo service nessusd stop
sudo cp -i /etc/letsencrypt/live/demo.foo.bar/fullchain.pem /opt/nessus/com/nessus/CA/servercert.pem
sudo cp -i /etc/letsencrypt/live/demo.foo.bar/privkey.pem /opt/nessus/var/nessus/CA/serverkey.pem
sudo cp -i /etc/letsencrypt/live/demo.foo.bar/cert.pem /opt/nessus/com/nessus/CA/cacert.pem
sudo service nessusd start
```


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
