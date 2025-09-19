# Guía de Instalación: Oracle Database 21c Express Edition (XE) en Rocky Linux 8 para macOS

**Autor:** Sebastián Cabezas Ríos

**Fecha:** 14 de septiembre de 2025

-----

## 1\. Requisitos y Opciones de Servidor

Esta guía asume que tienes un equipo Mac (MacBook, iMac, etc.) y deseas instalar Oracle Database 21c XE en un entorno Linux. Puedes usar una de las siguientes opciones para configurar el servidor:

  * **Opción Gratuita:** Usa **UTM** para crear una máquina virtual local con Rocky Linux 8.
  * **Opción Pagada:** Usa un **servidor VPS** con Rocky Linux 8. Puedes ver en [PremiumHosting](https://premiumhosting.cl/soporte/?affid=622) Planes VPS desde los CLP $9.900 + IVA:
    - 1 vCPU AMD Ryzen
    - 4 GB RAM DDR4
    - 60 GB Espacio NVMe SSD
    - 1 TB Transferencia
    - 1 Dirección IP


-----

## 2\. Preparación del Sistema

Después de instalar Rocky Linux 8, sigue estos pasos para preparar el sistema.

### 2.1. Configuración de la Red y Acceso SSH

***2.1.a.***  **Conexión de red:** Si la red `ethernet` no está conectada, levántala con el siguiente comando:

  ```bash
  nmcli connection up enp0s1
  ```

  Puedes verificar el estado de los dispositivos de red con:

  ```bash
  nmcli device status
  ```

  Para que la conexión de red se levante automáticamente al iniciar el sistema, debes configurar su archivo de conexión.

  ```bash
  nmcli connection show
  ```

  Esto te mostrará una lista de las conexiones disponibles, y deberías ver el nombre de tu conexión enp0s1 o algo similar.

  Edita el archivo de configuración de la conexión con nano o tu editor de texto preferido. El archivo se encuentra en /etc/sysconfig/network-scripts/. Reemplaza ifcfg-enp0s1 con el nombre correcto de tu interfaz.


  ```bash
  sudo vi /etc/sysconfig/network-scripts/ifcfg-enp0s1
  ```

  Dentro del archivo, busca la línea que dice ONBOOT=no y cámbiala a ONBOOT=yes.

  ```vi
  ONBOOT=yes
  ```

***2.1.b***  **Habilitar SSH:** Para conectarte remotamente, inicia el servicio SSH.

  ```bash
  systemctl start sshd
  ```

  **💡 Ojo!:** Si usas un VPS, ya debería estar activo.

***2.1.c***  **Conexión SSH:** Conéctate a tu servidor desde la terminal de tu Mac.

   ```bash
   ssh root@<dirección_ip_del_servidor>
   ```

### 2.2. Actualización e Instalación de Paquetes

1.  **Actualizar el sistema:**
    ```bash
    sudo dnf -y update
    ```
2.  **Instalar prerrequisitos:** Instala los paquetes esenciales que necesitarás.
    ```bash
    sudo dnf -y install wget curl unzip nano iptables-services
    ```

-----

## 3\. Instalación de Oracle Database 21c XE

Sigue estos pasos para instalar la base de datos.

### 3.1. Descargar el Paquete de Preinstalación

1.  **Navega a la carpeta temporal:**
    ```bash
    cd /tmp
    ```
2.  **Descarga el paquete preinstall:** Este paquete configurará automáticamente el usuario `oracle` y los grupos necesarios.
    ```bash
    wget https://yum.oracle.com/repo/OracleLinux/OL8/appstream/x86_64/getPackage/oracle-database-preinstall-21c-1.0-1.el8.x86_64.rpm
    ```

### 3.2. Instalar el Preinstall de Oracle

```bash
sudo dnf -y localinstall oracle-database-preinstall-21c-1.0-1.el8.x86_64.rpm
```

**Importante:** Después de este paso, se creará un usuario `oracle` en el sistema.

### 3.3. Descargar el Paquete de Oracle XE 21c

1.  **Desde tu Mac:** Abre tu navegador y ve a la página de descargas de Oracle XE.

    ```url
    https://www.oracle.com/database/technologies/xe-downloads.html
    ```

2.  **Copia el enlace de descarga:** Haz clic en **"Descargar"** para **"Oracle Database 21c Express Edition for Linux x64 (OL8)"**. Cuando la descarga inicie, cópialo y puedes cancelarla.

3.  **Desde el servidor:** En la terminal del servidor, usa `wget` para descargar el archivo usando el enlace que copiaste.

    ```bash
    wget <pega_el_enlace_aqui>
    ```

### 3.4. Instalar Oracle XE 21c

1.  **Renombrar el archivo:** Si el nombre del archivo descargado es muy largo, puedes renombrarlo para facilitar la instalación.
    ```bash
    mv 'oracle-database-x[TAB]' oracle-database-xe-21c.rpm
    ```
    **💡 Consejo:** Usa `ls` para ver el nombre completo del archivo.
2.  **Instalar el paquete RPM:**
    ```bash
    sudo dnf -y localinstall oracle-database-xe-21c.rpm
    ```
    El proceso puede tardar unos minutos. Una vez completado, Oracle Database estará instalado.

-----

## 4\. Configuración Post-Instalación de Oracle

### 4.1. Habilitar y Configurar el Servicio

1.  **Activar y habilitar el servicio:**
    ```bash
    sudo systemctl enable oracle-xe-21c
    sudo systemctl start oracle-xe-21c
    sudo /etc/init.d/oracle-xe-21c configure
    ```
2.  **Configurar variables de entorno:** Cambia al usuario `oracle` y configura su perfil.
    ```bash
    sudo su - oracle
    nano ~/.bash_profile
    ```
    Pega las siguientes líneas al final del archivo:
    ```nano
    # Variables de Oracle
    export ORACLE_BASE=/opt/oracle
    export ORACLE_HOME=/opt/oracle/product/21c/dbhomeXE
    export ORACLE_SID=XE
    export PATH=$ORACLE_HOME/bin:$PATH
    ```
3.  **Aplicar los cambios:**
    ```bash
    source ~/.bash_profile
    ```
4.  **Verificar la configuración:** Confirma que las variables se hayan cargado correctamente.
    ```bash
    echo $ORACLE_HOME
    echo $ORACLE_SID
    echo $ORACLE_BASE
    which dbca
    ```
    Las rutas mostradas deben coincidir con las que configuraste.

### 4.2. Configurar el Listener

1.  **Editar `listener.ora`:** Abre el archivo de configuración del listener.
    ```bash
    sudo nano /opt/oracle/homes/OraDBHome21cXE/network/admin/listener.ora
    ```
2.  **Modificar `HOST`:** Cambia el valor de `HOST` a `0.0.0.0` para que el listener acepte conexiones desde cualquier dirección IP.
    ```bash
    (ADDRESS = (PROTOCOL = TCP)(HOST = 0.0.0.0)(PORT = 1521))
    ```
    **⚠️ Importante:** Si solo quieres que sea accesible localmente, déjalo como `127.0.0.1`.
3.  **Reiniciar el listener:**
    ```bash
    lsnrctl stop
    lsnrctl start
    ```
4.  **Verificar el estado del listener:**
    ```bash
    lsnrctl status
    ```
    Asegúrate de que el estado indique `READY` para los servicios y que el host sea `0.0.0.0` (o el que hayas configurado).

### 4.3. Abrir el Puerto del Firewall

0. **Revisar que el puerto esté abierto**.

   ```bash
   sudo firewall-cmd --list-ports
   ```

1.  **Abrir el puerto 1521:** Permite el tráfico entrante para el puerto de Oracle.
    ```bash
    sudo firewall-cmd --add-port=1521/tcp --permanent
    ```
2.  **Habilitar las políticas del Firewall:**
    ```bash
    sudo firewall-cmd --reload
    ```
-----

## 5\. Solución de Problemas Comunes

### Listener no se inicia (Error `Permission denied`)

**Error:**

```bash
NL-00280: error creating log stream /opt/oracle/product/21c/dbhomeXE/network/log/listener.log
Linux Error: 13: Permission denied
```

**Causa:** El usuario `oracle` no tiene los permisos para escribir en el directorio de logs del listener.
**Solución:** Cambia la propiedad del directorio de logs al usuario `oracle`.

```bash
sudo chown -R oracle:oinstall /opt/oracle/product/21c/dbhomeXE/network/log
```
