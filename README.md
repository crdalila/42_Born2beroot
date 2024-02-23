# 42_Born2beroot
Configurar una máquina virtual siguiendo una serie de normas concretas

A continuación, detallo paso a paso el esquema que he seguido para la ejecución de este proyecto.

## 1) Instalar máquina virtual
Descargamos la máquina virtual Virtualbox y la instalamos en nuestra máquina real

## 2) Instalar Debian
Descargamos el sistema operativo Debian (recomendado para personas que nunca hemos instalado máquinas virtuales y somos principiantes en la programación)

## 3) Configurar el proyecto
Con las especificaciones que nos pide el ejercicio, pasamos a configurar la máquina virtual.
  + Instalar sudo y configurar usuarios y grupos
  + Instalar y configurar SSH
  + Instalar y configurar UFW
  + Configurar contraseña fuerte sudo
  + Configurar política de contraseñas fuerte
  + Conectarse vía SSH

## 4) Script
Script = Secuencia de comandos guardada en un fichero, cuando se ejecuta el fichero, hace lo que pone dentro.
  + Arquitecture
  + Núcleos físicos
  + Núcleos virtuales
  + Memoria RAM
  + Memoria de disco
  + Porcentaje de uso de CPU
  + Uso LVM
  + Número de usuarios
  + Dirección IP y Mac
  + Número de comandos ejecutados con sudo

## 5) Crontab
Crontab = Administrador de procesos en segundo plano, los procesos son ejecutados en el momento que se especifique en el fichero.

## 6) Signature.txt
Apagar la máquina virtual, acceder mediante Terminal de nuestra máquina real donde tengamos nuestro archivo .vdi (en mi caso en sgoinfre/goinfre/Perso/dalcabre/Born2beroot) y ejecutar el comando **shasum nombredemaquina.vdi**
El código que salga de aquí, será para pegar en el archivo signature.txt que subiremos al repositorio.
