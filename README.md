# Usando Cron  en centos 7

## Enviar correos por línea de comandos
### Instalar mailx

 1. #yum -y update 
 2. #yum install -y mailx
 3. #ln -s /bin/mailx /bin/email
 
 ### Configuramos mailx
 #vi /etc/mail.rc
 
 Agregamos las siguientes líneas:
 
set smtp=smtps://smtp.gmail.com:465
set smtp-auth=login
set smtp-auth-user=maigangelus@gmail.com
set smtp-auth-password=password
set ssl-verify=ignore
set nss-config-dir=/etc/pki/nssdb/

### Enviamos correo 
echo "Your message" | mail -v -s "Message Subject" maig19@hotmail.com

https://www.binarytides.com/linux-mailx-command/

### Notas
En caso de usar el smtp de gmail, necesitamos dar permisos a apps de terceros :
google acount -> acceso y seguridad ->  Permitir el acceso de aplicaciones menos seguras: SÍ

## Crear respaldos de db mysql con cron

### Instalando Mysql
https://www.linode.com/docs/databases/mysql/how-to-install-mysql-on-centos-7/

### Creando base de datos y tablas de prueba
#mysql -u root
mysql> create database gastos;
mysql> use gastos;
mysql> create table Egresos (id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY, monto DOUBLE NOT NULL, fecha DATE NOT NULL);
mysql> create table Ingresos (id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY, monto DOUBLE NOT NULL, fecha DATE NOT NULL);

### Crear shell para insertar datos a las tablas de prueba
#vi inserta_gastos.sh

#!/bin/bash
echo "Insert into Egresos values(null, 100, '$(date +"%Y-%m-%d")')" | mysql -u root gastos
echo "Insert into Ingresos values(null, 500, '$(date +"%Y-%m-%d")')" | mysql -u root gastos

Damos permisos de ejecución al archivo 
#chmod +x inserta_gastos.sh
### Agregamos una tarea al cron

Listar tareas: 
#cat /etc/crontab

Agregar tareas 
#*/3 * * * * root /home/vagrant/inserta_gastos.sh

Para acualizar el servicio de cron 
#systemctl restart crond.service

### Crear dump de base de datos y enviar correo para notificar la ejecución de la tarea

#vi make_dump.sh

#!/bin/bash

anio=$(date +'%Y')

if [ ! -d "$anio" ]; then
  mkdir $anio
fi

ruta=`$`anio/`$`(echo $(date +'%b') | tr 'a-z' 'A-Z')

[ ! -d "$ruta" ] && mkdir $ruta || >/dev/null

fecha=$(date +'%d%m%y_%H%M%S')

result=`$`(mysqldump -u root gastos > gastos$fecha.sql)

if [ ! `$`? = 0 ]; then
 rm gastos`$`fecha.sql
 echo $result | mail -v -s "Error en respaldo" maig19@hotmail.com
 exit 1
fi

tar -zcvf "gastos$fecha.sql.tar.gz" gastos$fecha.sql
rm -f gastos$fecha.sql
mv "gastos$fecha.sql.tar.gz" "$ruta/"

echo "Se creo el respaldo gastos$fecha.sql.tar.gz " | mail -v -s "Notificación de respaldo" maig19@hotmail.com
