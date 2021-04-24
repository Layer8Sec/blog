---
title: ¿Cómo exportar los escaneos de Nessus en una Base de Datos?
published: true
date: 2021-04-23
img: bullet.png
project-date: Apr 2021
excerpt: Pequeña guía para configurar exportar los resultados del escaneo de Nessus en una Base de Datos con Python3 y MySQL
category: Tools
---

## [](#header-2)Previos

`Esto no se trata de una guía oficial, es algo que nos ha funcionado y esperamos que sea de utilidad.
Run it at your own risk`

Luego de instalar y configurar nuestro **Tenable Nessus**, parte de las configuraciones de seguridad que debemos realizar es habilitar de manera segura el acceso vía la plataforma web, esto lo conseguimos configurando nuestros certificados confiables y **Tenable Nessus** no muestre el error de Certificados ya conocido.

Estamos asumiendo que Nessus Professional está instalado sobre un Sistema Operativo Linux, sin embargo sería una lógica similar para un Sistema Operativo basado en Windows. Tambien se asume que ya tienes configurado el dominio o subdominio para que resuelva a la dirección IP donde se tiene instalado Tenable Nessus.

![](assets\posts\nessus_img1.png)

## [](#header-2)Instalar Script de GitHub

En este caso, descargaremos el siguiente repositorio de GitHub [Link del Repositorio](https://github.com/eddiez9/nessus-database-export) y seguimos las instruccion de la instalacion.


## [](#header-2)Generamos las API Key de Nessus

En caso no contemos con la API Key en Nessus, las volvemos a generar y las guardamos para utilizarlas luego. `Recordar que si ya se encuentra usando esta API Key en otra aplicacion y volvemos a generarla, deberas importar esta nueva API Key en tu aplicacion`


* * *
![](assets\posts\db_nessus9.png)
* * *

## [](#header-2)Configurando nuestro MySQL

Primero debemos levantar nuestro servicio de MySQL

```bash
sudo service mysql start
```

Ingresaremos a nuestro MySQL y luego configuraremos la contraseña a nuestro usuario, en este caso usaremos el mismo root.


```bash
mysql -u root -p
ALTER USER 'root'@'localhost' IDENTIFIED BY 'NuevaPassword';
FLUSH PRIVILEGES;
```

## [](#header-2)Configuracion del Script de Github

Cambiaremos la configuracion del config.ini.example a config.ini y lo editaremos hasta que tenga el siguiente formato:

* * *
![](assets\posts\db_nessus1.jpeg)
* * *

Colocamos nuestra **Access Key** y nuestra **Secret Key** generados en Nessus, tambien colocamos muestro usuario de MySQL y la contraseña configurada previamente.

Ademas ingresamos al MySQL y generamos las base de datos necesaria. El archivo **schema.sql** se encuentra en la carpeta del repositorio, en caso MySQL no lo encuentre es porque no se encuentra en la misma carpeta del archivo, en ese caso colocar la ruta absoluta del archivo.

```bash
mysql -u root -p
source schema.sql;
```

## [](#header-2)Ejecutamos el Script

Ejecutaremos el script, lo que hara sera ejecutar cada uno de nuestros scans que se encuentran en nuestros folders.

* * *
![](assets\posts\db_nessus5.png) ![](assets\posts\db_nessus4.png)
* * *


El script **EJECUTARÁ TODOS** los escaneos que encuentre en las carpetas, tomar las previciones del caso antes de ejecutarlo.

Luego de finalizado el script, se podra revisar los resultados de todos los escaneos en nuestro MySQL

## [](#header-2)Verificamos la base de datos

Ingresaremos a MySQL con nuestro usuario y haremos unas consultas para verificar que esten nuestros escaneos.

Verificamos que esten nuestros reportes.

```sql
SELECT * FROM nessusdb.scan;
```
* * *
![](assets\posts\db_nessus7.png)
* * *

Hacemos un conteo rapido para saber que si se logro la ejecucion del script correctamente y se obtuvieron varias vulnerabilidades


```sql
SELECT COUNT(host_vuln_id) FROM nessusdb.vuln_output;
* * *
```
![](assets\posts\db_nessus8.png)
* * *

Si se desea se podria exportar la base de datos con el siguiente comando:

```bash
mysqldump -u [username] -p [database name] > [database name].sql
```

Con la base de datos exportada ya se puede trabajar en nuestra maquina personal si asi se desea, o compartirla entre los miembros de la empresa


```
think outside the box
```

