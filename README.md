# Instalación de Oracle Database 23c en Rocky Linux 8
Fecha: 14 de septiembre de 2025
Autor: Sebastián Cabezas Ríos

# Requisitos
## Servidor UTM con Rocky Linux 8
```bash
systemctl start sshd
```
```bash
nmcli device status
```
Si está desconectado el **ethernet** entonces hay que levantarlo.
```bash
nmcli connection up enp0s1
```


## Servidor VPS con Rocky Linux 8


## Instalación de Oracle Database 23c XE
### Paso 1 de 7 - Conexión SSH
Conéctate por ssh con tu servidor
```bash
ssh root@<dirección ip>
```

### Paso 2 de 7 - Actualización del sistema
* Como **root**
```bash
sudo dnf -y update
```

### Paso 3 de 7 - Instalar paquetes de prerrequisitos
* Como **root**
```bash
sudo dnf -y install wget curl unzip nano iptables-services
```

### Paso 4 de 7 - Descargar con WGET el paquete preinstall de Oracle en carpeta temporal
* Como **root**
```bash
cd /tmp
```
```bash
wget https://yum.oracle.com/repo/OracleLinux/OL8/appstream/x86_64/getPackage/oracle-database-preinstall-21c-1.0-1.el8.x86_64.rpm
```

### Paso 5 de 7 - Instalar el preinstall de Oracle RPM
Al instalar, crea el usuario oracle, los grupos y los paquetes necesarios para que Oracle Database 21c XE funcione en el sistema.

* Como **root**
```bash
sudo dnf -y localinstall oracle-database-preinstall-21c-1.0-1.el8.x86_64.rpm
```

### Paso 6 de 7 - Descargar Oracle XE 21c desde la web de Oracle
* En tu máquina, abre el navegador web y sigue los siguientes pasos 6.1 al 6.4:

6.1. Ir a la página web de Oracle
```url
https://www.oracle.com/database/technologies/xe-downloads.html
```
6.2. Descargar: **Oracle Database 21c Express Edition for Linux x64 ( OL8 )**

6.3. Cuando comience a descargar, click en todas las descargas y copiar link.

6.4. Puedes detener la descarga

* Como **root**

Cambia el nombre del archivo descargado con **mv** 
```bash
mv 'oracle-database-x[TAB]' oracle-database-xe-21c.rpm
```

### Paso 7 de 7 - Instalar Oracle XE 21c RPM
* Como **root**
```bash
sudo dnf -y localinstall oracle-database-xe-21c.rpm
```
Oracle Database habrá quedado instalado en el sistema

## Configuración de Oracle Dabase 23c XE


### Paso 1 de xx - Activar el servicio, habilitar arranque automático

* Como **root**
```bash
sudo systemctl enable oracle-xe-21c
```
```bash
sudo systemctl start oracle-xe-21c
```
* Como **oracle**
```bash
sudo su - oracle
```
```bash
nano ~/.bash_profile
```
En el archivo bash_profile pegar al final:
```nano
# Variables de Oracle
export ORACLE_BASE=/opt/oracle
export ORACLE_HOME=/opt/oracle/product/21c/dbhomeXE
export ORACLE_SID=XE
export PATH=$ORACLE_HOME/bin:$PATH
```
Aplicar los cambios inmediatamente:
```bash
source ~/.bash_profile
```
Para verificar que está todo en orden:
```bash
echo $ORACLE_HOME
echo $ORACLE_SID
echo $ORACLE_BASE
which dbca
```
La salida esperada es:
```bash
echo $ORACLE_HOME
/opt/oracle/product/21c/dbhomeXE

echo $ORACLE_SID
XE

echo $ORACLE_BASE
/opt/oracle

which dbca
/opt/oracle/product/21c/dbhomeXE/bin/dbca
```

### Paso 2 de xx - Configurar Listener
* Como **root**
```bash
sudo nano /opt/oracle/homes/OraDBHome21cXE/network/admin/listener.ora
````
Tiene que estar el host como 0.0.0.0 para que sea accesible desde cualquier IP y no solo el mismo servidor. Si se desea que sea el mismo servidor: 127.0.0.1 en *HOST*:
```bash
DEFAULT_SERVICE_LISTENER = XE

LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = 0.0.0.0)(PORT = 1521))
      (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))
    )
  )
```
Hay que bajar el **listener**
```bash
lsnrctl stop
```
Hay que subir el **listener**
```bash
lsnrctl start
```


### Paso 2 de xx - Verificar Listener
* Como **oracle**
```bash
sudo su - oracle
```
```bash
lsnrctl status
```
La salida esperada es:
```bash
LSNRCTL for Linux: Version 21.0.0.0.0 - Production on 14-SEP-2025 14:13:43

Copyright (c) 1991, 2021, Oracle.  All rights reserved.

Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=0.0.0.0)(PORT=1521)))
STATUS of the LISTENER
------------------------
Alias                     LISTENER
Version                   TNSLSNR for Linux: Version 21.0.0.0.0 - Production
Start Date                14-SEP-2025 14:25:04
Uptime                    497 days 2 hr. 16 min. 31 sec
Trace Level               off
Security                  ON: Local OS Authentication
SNMP                      OFF
Default Service           XE
Listener Parameter File   /opt/oracle/homes/OraDBHome21cXE/network/admin/listener.ora
Listener Log File         /opt/oracle/diag/tnslsnr/oracle-21c-xe-rocky-8/listener/alert/log.xml
Listening Endpoints Summary...
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=0.0.0.0)(PORT=1521)))
  (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(KEY=EXTPROC1521)))
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcps)(HOST=oracle-21c-xe-rocky-8)(PORT=5500))(Security=(my_wallet_directory=/opt/oracle/admin/XE/xdb_wallet))(Presentation=HTTP)(Session=RAW))
Services Summary...
Service "3ebcf8c66b9746b1e0650225cb1adc3e" has 1 instance(s).
  Instance "XE", status READY, has 1 handler(s) for this service...
Service "XE" has 1 instance(s).
  Instance "XE", status READY, has 1 handler(s) for this service...
Service "XEXDB" has 1 instance(s).
  Instance "XE", status READY, has 1 handler(s) for this service...
Service "xepdb1" has 1 instance(s).
  Instance "XE", status READY, has 1 handler(s) for this service...
The command completed successfully
```
### Paso 3 de xx - Abrir el puerto

```bash
sudo iptables -L -n
```
```bash
sudo iptables -A INPUT -p tcp --dport 1521 -j ACCEPT
```
Guardar las reglas del Firewall

```bash
sudo mkdir -p /etc/iptables
```
```bash
sudo iptables-save > /etc/iptables/rules.v4
```

## Posibles errores:

### Listener no inicia

```bash
lsnrctl start 

LSNRCTL for Linux: Version 21.0.0.0.0 - Production on 14-SEP-2025 11:07:46 Copyright (c) 1991, 2021, Oracle. All rights reserved. Starting /opt/oracle/product/21c/dbhomeXE/bin/tnslsnr: please wait... TNSLSNR for Linux: Version 21.0.0.0.0 - Production NL-00280: error creating log stream /opt/oracle/product/21c/dbhomeXE/network/log/listener.log NL-00278: cannot open log file SNL-00016: snlfohd: error opening file Linux Error: 13: Permission denied
```
