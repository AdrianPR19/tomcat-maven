# Instalación de Tomcat 9 y OpenJDK

Este documento describe los pasos para instalar **OpenJDK** y **Tomcat 9**, y configurar los servicios necesarios en un sistema basado en Debian.

## **Requisitos previos**
- Un sistema operativo basado en Linux (por ejemplo, Debian 11).
- Acceso a un usuario con privilegios de administrador.

---

## **Paso 1: Instalación de OpenJDK**

1. Instalar OpenJDK 11 utilizando el gestor de paquetes:
   ```bash
   sudo apt install -y openjdk-11-jdk
   ```

2. Verificar que la instalación fue exitosa ejecutando:
   ```bash
   java -version
   ```
   Deberías ver un resultado similar a:
   ```
   openjdk version "11.x.x"
   ```

---

## **Paso 2: Instalación de Tomcat 9**

### **2.1 Instalación del paquete Tomcat 9**

1. Instalar Tomcat 9 utilizando el gestor de paquetes:
   ```bash
   sudo apt install -y tomcat9
   ```

### **2.2 Creación de un grupo y usuario para Tomcat**

1. Crear un grupo de usuarios específico para Tomcat:
   ```bash
   sudo groupadd tomcat9
   ```

2. Crear un usuario dedicado para el servicio:
   ```bash
   sudo useradd -s /bin/false -g tomcat9 -d /etc/tomcat9 tomcat9
   ```
   - `-s /bin/false`: Bloquea el acceso a la shell.
   - `-g tomcat9`: Asigna al grupo `tomcat9`.
   - `-d /etc/tomcat9`: Asigna el directorio de inicio del usuario.

---

## **Paso 3: Configuración y verificación del servicio**

1. Iniciar el servicio Tomcat 9:
   ```bash
   sudo systemctl start tomcat9
   ```

2. Verificar que el servicio esté activo:
   ```bash
   sudo systemctl status tomcat9
   ```
   Deberías ver algo como:
   ```
   Active: active (running) ...
   ```

3. Acceder a la página de bienvenida de Tomcat:
   - Abre tu navegador y visita: `http://localhost:8080`
   - Deberías ver la página de inicio de Apache Tomcat.

---

## **Notas finales**

- Si encuentras algún problema durante la instalación o el servicio no inicia correctamente, revisa los logs en `/var/log/tomcat9/` para obtener más información.
- Para detener o reiniciar el servicio, puedes utilizar:
  ```bash
  sudo systemctl stop tomcat9
  sudo systemctl restart tomcat9
  ```

