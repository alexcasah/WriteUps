<h1 align="center">
<br>
  <img src="https://anaparrillablog.files.wordpress.com/2016/04/cayo-julio-cc3a9sar-imperio-romano.png?w=656" alt="Front-End Checklist" width="200">
  <br>
  HACKSUDO: 1.0.1
  <br>
</h1>

<h4 align="center">Maquina creada por <img src="https://img.icons8.com/fluent/344/email-open.png" alt="Front-End Checklist" width="20">vishal@hacksudo.comr</h4>
<h4 align="center">Aquí puedes descargarte la <a href="https://www.vulnhub.com/entry/hacksudo-101,650/">página web</a> y ver una descripción de ella</h4>

---

# Índice

- [Recopilación de información](#recopilación-de-información)
  - [Nmap](#nmap)
  - [Fuzz / Whatweb](#)
- [Búsqueda de vulnerabilidades](#búsqueda-de-vulnerabilidades)
  - [Tomcat](#)
- [Explotación de vulnerabilidades](#explotación-de-vulnerabilidades)
  - [Metasploit](#)
- [Post-explotación](#post-explotación)
  - [Movimiento Lateral I](#)
  - [Movimiento Lateral II](#)
       - [linPEAS](#)
       - [Reverse Shell]()
  - [Búsqueda de vulnerabilidades](#búsqueda-de-vulnerabilidades-1)
       - [SUDO -L](#)
  - [Explotación de vulnerabilidades](#explotación-de-vulnerabilidades-1)
       - [SCP](#)
- [Información Adicional](#información-adicional)


---

## Recopilación de información

Aquí veremos que esta conectado a la subred, los puerto abiertos, las paginas web, etc.

### Nmap
Veremos que redes estan conectadas a esta subred con este comando:

```nmap -sP 192.168.221.0/24```

Para ver los puertos abiertos que tiene la red utilizaremos este comando:

```nmap -sV -sC 192.168.221.9```

### Fuzz / Whatweb
Para fuzzear la web usaremos este comando con una wordlist que tengamos:

```ffuf -c -w /opt/wordlists/SecLists/Discovery/Web-Content/common.txt -u http://192.168.221.9/FUZZ```

Para ver solo archivos con extensión .txt,.php,.html

```ffuf -c -w /opt/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://192.168.221.9/FUZZ -e .txt,.html,.php -ic```

Con este comando podemos recolectar información de un servidor web:
```whatweb http://192.168.221.9```

## Búsqueda de vulnerabilidades

### Tomcat

El login del tomcat lo hemos encontrado en esta url: ```http://192.168.221.9:8080/manager```

En esta [página web ](https://github.com/netbiosX/Default-Credentials/blob/master/Apache-Tomcat-Default-Passwords.mdown) encontramos las credenciales por defecto.

Conseguimos conectarnos con las credenciales por defecto: 
```
User: tomcat
Password: tomcat
```
