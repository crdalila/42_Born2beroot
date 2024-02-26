 # 42_Born2beroot
Configurar una máquina virtual siguiendo una serie de normas concretas

A continuación, detallo paso a paso el esquema que he seguido para la ejecución de este proyecto.
## ÍNDICE

  - [1) Instalar máquina virtual](#instalar-máquina-virtual)
  - [2) Instalar Debian](#instalar-debian)
  - [3) Configurar el proyecto](#configurar-el-proyecto)
  - [4) Script](#script)
  - [5) Crontab](#crontab)
  - [6) Signature.txt](#signature-.-txt)

## 1) Instalar máquina virtual
Descargamos la máquina virtual Virtualbox y la instalamos en nuestra máquina real

## 2) Instalar Debian
Descargamos el sistema operativo Debian (recomendado para personas que nunca hemos instalado máquinas virtuales y somos principiantes en la programación)

## 3) Configurar el proyecto
Con las especificaciones que nos pide el ejercicio, pasamos a configurar la máquina virtual.

  + **Instalar sudo y configurar usuarios y grupos**

    *Sudo = Super usuario, administrador, root. Usuario que puede ejecutar y administrar tareas para las que otros usuarios necesitan permisos específicos.*
      Comandos utilizados para esta parte de la configuración:
      - **su** -> acceder al super usuario (sudo, root)
      - **sudo reboot** -> reiniciar la máquina
      - **sudo -v** -> muestra la versión de la máquina y todos sus datos
      - **sudo adduser nombreusuario** -> añadir un usuario
      - **sudo addgroup nombregrupo** -> añadir un grupo
      - **sudo addusser usuario1 grupo1** -> añadir el usuario1 al grupo1
      - **getent group nombregrupo** -> comprobación de si se ha creado el grupo
    
  + **Instalar y configurar SSH**

    *SSH = Nombre de protocolo. Permite el acceso remoto a un servidor por medio de un canal seguro con información cifrada.*
      Comandos utilizados para esta parte de la configuración:
      - **sudo apt update** -> actualizar repositorios
      - **sudo apt install openssh-server** -> instalar openssh (lo necesitamos para utilizar ssh)
      - **sudo service ssh status** -> comprobar el estado del ssh
      Además de esto, editaremos los puertos en el fichero *sshd_config* (en este caso, pondremos los puertos 4242 como indica el subject)

  + **Instalar y configurar UFW**

    *UFW = Firewall para configurar Iptables.
    Iptables = Programa que permite a un administrador de sistema configurar tablas, y las acdenas y reglas que almacenan.*
      Comandos utilizados para esta parte de la configuración:
      - **sudo apt install ufw** -> instalar UFW
      - **sudo ufw enable** -> habilitar el firewall
      - **sudo ufw allow 4242** -> para permitir las conexiones del puerto 4242
      - **sudo ufw status** -> comprobar el estado del UFW

  + **Configurar contraseña fuerte sudo**

    Para ello creamos una carpeta sudo en la ruta **/var/log** y un fichero en **/etc/sudoers.d** con el contenido:
    ```
    Defaults  passwd_tries=3
    Defaults  badpass_message= "Wrong password"
    Defaults  logfile="var/log/sudo/nombrefichero"
    Defaults  log_input, log output
    Defaults  iolog_dir="/var/log/sudo"
    Defaults  requiretty
    Defaults  secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"
      ```
    Explicación:
      - Tres intentos para introducir la contraseña
      - El texto que aparecerá si lo introduces mal
      - Archivo donde quedan registrados los comandos sudo
      - Que cada comando ejecutado con sudo (tanto input como output), quede registrado en esa carpeta
      - Activar el modo TTY
      - Restringir los directorios utilizables por sudo
  
  + **Configurar política de contraseñas fuerte**

    Para ello editamos el fichero */etc/login.defs* y modificamos estos datos:
      - PASS_MAX_DAYS -> tiempo de expiración de la contraseña en días
      - PASS_MIN_DAYS -> número mínimo de días permitido antes de cambiar la contraseña
      - PASS_WARN_AGE -> para que el usuario reciba un mensaje de aviso indicando que faltan X días para que expire su contraseña

    Instalamos el paquete libpam_pwquality
    
    Editamos el fichero **/etc/pam.d/common_password** y le añadimos algunas líneas después de *retry=3*
      - **minlen = 10** -> caracteres mínimos de la contraseña
      - **ucredit = -1** -> mínimo una letra mayúscula
      - **dcredit = -1** -> mínimo un dígito
      - **lcredit = -1** -> mínimo una letra minúscula
      - **maxrepeat = 3** -> no puede tener más de tres letras seguidas
      - **reject_username** -> no puede contener el nombre del usuario
      - **difok = 7** -> siete caracteres que no sean parte de la antigua contraseña
      - **enforce_for_root** -> política para usuario root
  
  + **Conectarse vía SSH**

    Cerramos la máquina virtual.
    
    En el VirtualBox, vamos a **Configuración > Red > Avanzadas > Reenvío de Puertos**

    Agregamos un reenvío con puertos 4242

    Abrimos un terminal en la máquina real

    - **ssh dalcabre@localhost -p 4242** -> para entrar en la máquina virtual desde la terminal de otra máquina.


## 4) Script

  *Script = Secuencia de comandos guardada en un fichero, cuando se ejecuta el fichero, hace lo que pone dentro.*

  Creamos un archivo .sh que contenga los siguientes datos:
  
  + **#ARCHITECTURE**
    - **uname -a** -> ver la arquitectura del Sistema Operativo
      
  + **#NÚCLEOS FÍSICOS**
    - **grep "physical id" /proc/cpuinfo | wc -l** -> mostrar el número de núcleos virtuales. Buscamos el fichero que contenga "physical id" en esa ruta  y contamos las líneas
      
  + **#NÚCLEOS VIRTUALES**
    - **grep processor /proc/cpuinfo | wc -l** -> mostrar el número de núcleos virtuales.

  + **#MEMORIA RAM**
    - **free** -> información completa sobre la memoria RAM
    - **free --help** -> información sobre el comando
    - **free --mega** -> mostrar la información en MegaBytes (como especifica el subject)
    - **free --mega | awk '$1 == "Mem:" {print $3}** -> de toda la información, nos quedamos con la tercera palabra de la fila $1, memoria total
    - **free --mega | awk '$1 == "Mem:" {print $2}** -> de toda la información, nos quedamos con la segunda palabra de la fila $1, memoria usada
    - **free --mega | awk '$1 == "Mem:" {printf("%.df%%)\n", $3/$2*100)}** -> calcular (dividiendo el tercer y el segundo dato y multiplicándolo por 100) e imprimir el % de la memoria usada
      
  + **#MEMORIA DE DISCO**
    - **df -m** -> disk filesystem (para mostrar información sobre la memoria de disco) en MegaBytes
    - **df -m | grep "/dev/" | grep -v "/boot/" | awk '{memory_use += $3} END {print memory_use}'** -> *Espacio ocupado* de la memoria de disco, lo que hace es buscar archivos que contengan /dev/ pero que obvien /boot/, sumar el dato $3 de cada línea y cuando termina, lo imprime
    - **df -m | grep "/dev/" | grep -v "/boot/" | awk '{memory_result += $2} END {printf ("%0fGb\n"), memory_result/1024}'** -> *Espacio total* de la memoria de disco, lo que hace es sumar el dato $2 de cada línea y cuando termina, lo imprime sin decimales y en Gb (para eso hay que dividir entre 1024)
    - **df -m | grep "/dev/" | grep -v "/boot/" | awk '{use += $3} {total += $2} END {printf("("%d%%)\n"), use/total*100}'** -> *% de la memoria usada*, lo que hace es dividir el uso entre el total y multiplicarlo por 100 para sacar el %
    
  + **#PORCENTAJE DE USO DE CPU##**
    - **vmstat** -> muestra las estadísticas del sistema
    - **vmstat 1 4 | tail -1 | awk '{print $15}'** -> *Uso de memoria disponible*, de un intervalo de 1 a 4 segundos, se queda con la última (tail -1) e imprime el $15 dato

  + **#ÚLTIMO REINICIO**
    - **who -b | awk '{$1 == "system" {print $3 " " $4}'** -> tiempo del último reinicio del sistema, si la primera palabra es "system", imprime el $3 dato, un espacio y el 4$
        
  + **#USO LVM**
    - **lsblk** -> muestra toda la información de dispositivos de bloque (discos duros, SSD, memorias, etc)
    - **if [ $(lsblk) grep "lvm" | wc -l) -gt 0 ], then echo yes; else echo no; fi** -> Chequear si el LVM está activo o no. Si hay un lvm, imprime yes, si no, no

  + **#CONEXIONES TCP**
    - **ss -ta | grep ESTAB | wc -l** -> número de conexiones TCP establecidas, el -ta es para especificar TCP 
  
  + **#NÚMERO DE USUARIOS**
    - **users | wc -l** -> contar el número de usuarios que hay loggeados
    
  + **#DIRECCIÓN IP Y MAC**
    - **ip link | grep "link/ether" | awk '{print $2}'** -> para sacar la dirección MAC
    - **hostname -I** -> dirección del host
  
  + **#NÚMERO DE COMANDOS EJECUTADOS CON SUDO**
    - **journalctl** -> recopilación y administraión de registros del sistema
    - **journalctl -COMM=sudo | grep COMMAND | wc -l** -> especificamos las entradas por ruta y solo si aparece en la línea de comandos

## 5) Crontab
  *Crontab = Administrador de procesos en segundo plano, los procesos son ejecutados en el momento que se especifique en el fichero.*
  - **sudo crontab -u root -e** -> para editar el fichero crontab
  Dentro del fichero, añadimos esta línea:
```
  */10 * * * * sh /rutadelscript (en mi caso: /home/dalcabre/monitoring.sh)
```
  Esto hará que el archivo script creado en el paso anterior, se ejecute cada 10 minutos.
  
## 6) Signature.txt
  - Apagamos la máquina virtual
  - Acceder mediante Terminal de nuestra máquina real donde tengamos nuestro archivo .vdi (en mi caso en sgoinfre/goinfre/Perso/dalcabre/Born2beroot)
  - Ejecutar el comando **shasum nombredemaquina.vdi** en la terminal de la máquina real
  Esto nos proporcionará un código que será nuestra firma, este código lo añadimos a un archivo **signature.txt** que es el que tenemos que entregar y subir al repositorio.

  Esta firma cambia cada vez que abrimos o interactuamos con la máquina virtual, por lo que una vez hecha, es recomendable NO TOCAR NADA. Si aun así queremos hacer pruebas, podemos o bien clonar la máquina virtual o crear un **snapshot** (una especie de copia de seguridad de lo que hemos hecho hasta ahora), y al cerrar este snapshot especificar que NO GUARDE CAMBIOS.
