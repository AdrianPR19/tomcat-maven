# Tomcat y Maven - Versión 1.0.0

## 1. Instalación de OpenJDK

Instalamos el kit de desarrollo de Java:
```bash
sudo apt install -y openjdk-11-jdk
```

## 2. Instalación y Configuración de Tomcat

### 2.2 Instalación del paquete

Instalamos el servidor de aplicaciones Tomcat:
```bash
sudo apt install -y tomcat9
```

### 2.3 Creación del grupo y usuario para Tomcat

#### Creación de grupo:
```bash
sudo groupadd tomcat9
```
![Captura del funcionamiento](/1.png)
#### Creación del usuario:
```bash
sudo useradd -s /bin/false -g tomcat9 -d /etc/tomcat9 tomcat9
```
Este usuario será utilizado exclusivamente para gestionar el servicio de Tomcat.

### 2.4 Arranque y comprobación del servicio

#### Arranque del servicio:
```bash
sudo systemctl start tomcat9
```

#### Comprobación del estado del servicio:
```bash
sudo systemctl status tomcat9
```
Deberías ver que el servicio está activo y corriendo.

### 2.5 Configuración de administración

Editamos el archivo `/etc/tomcat9/tomcat-users.xml` para configurar los usuarios y roles necesarios:
```xml
<tomcat-users xmlns="http://tomcat.apache.org/xml"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://tomcat.apache.org/xml tomcat-users.xsd"
              version="1.0">
    <role rolename="admin"/>
    <role rolename="admin-gui"/>
    <role rolename="manager"/>
    <role rolename="manager-gui"/>
    <user username="alumno" password="1234" roles="admin,admin-gui,manager,manager-gui"/>
</tomcat-users>
```

### 2.6 Instalación del administrador web

Instalamos las herramientas adicionales:
```bash
sudo apt install -y tomcat9-admin
```
Esto proporciona acceso a las interfaces de administración y gestión del servidor.

#### Acceso a los paneles de administración:
- **Admin GUI:** [http://localhost:8080/manager/html](http://localhost:8080/manager/html)
- **Host Manager GUI:** [http://localhost:8080/host-manager/html](http://localhost:8080/host-manager/html)

En caso de requerir acceso remoto, sustituimos el archivo `/usr/share/tomcat9-admin/host-manager/META-INF/context.xml` por el siguiente contenido:
```xml
<Context antiResourceLocking="false" privileged="true">
    <Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="\d+\.\d+\.\d+\.\d+"/>
</Context>
```

Reiniciamos el servidor:
```bash
sudo systemctl restart tomcat9
```

---

## 3. Despliegue manual mediante GUI

1. Accedemos a la GUI de Tomcat con el usuario configurado previamente.
2. Buscamos la sección para desplegar archivos WAR.
3. Seleccionamos el archivo `tomcat1.war`.
4. Pulsamos **Desplegar** y verificamos que la aplicación esté disponible en el directorio `/tomcat1`.

---

## 4. Configuración y Despliegue con Maven

### 4.1 Instalación de Maven

Instalamos Maven:
```bash
sudo apt-get update && sudo apt-get -y install maven
```
Verificamos la instalación:
```bash
mvn -v
```

### 4.2 Configuración de roles y usuarios

Editamos el archivo `/etc/tomcat9/tomcat-users.xml` para incluir un usuario específico para despliegues con Maven:
```xml
<user username="deploy" password="1234" roles="manager-script"/>
```

### 4.3 Configuración de Maven

Editamos el archivo `/etc/maven/settings.xml` para agregar las credenciales del servidor:
```xml
<servers>
    <server>
        <id>Tomcat</id>
        <username>deploy</username>
        <password>1234</password>
    </server>
</servers>
```

### 4.4 Despliegue con Maven

Generamos una aplicación de ejemplo:
```bash
mvn archetype:generate -DgroupId=org.example -DartifactId=tomcat-war -DarchetypeArtifactId=maven-archetype-webapp -DinteractiveMode=false
cd tomcat-war
```

Editamos el archivo `pom.xml` para incluir el plugin de despliegue:
```xml
<build>
    <finalName>tomcat-war</finalName>
    <plugins>
        <plugin>
            <groupId>org.apache.tomcat.maven</groupId>
            <artifactId>tomcat7-maven-plugin</artifactId>
            <version>2.2</version>
            <configuration>
                <url>http://localhost:8080/manager/text</url>
                <server>Tomcat</server>
                <path>/despliegue</path>
            </configuration>
        </plugin>
    </plugins>
</build>
```

Comandos para desplegar la aplicación:
```bash
mvn tomcat7:deploy
mvn tomcat7:redeploy
mvn tomcat7:undeploy
```

---

## 5. Tareas

1. Realizar el despliegue de la aplicación de prueba generada.
2. Repetir el despliegue con otra aplicación. Por ejemplo:
```bash
git clone https://github.com/cameronmcnz/rock-paper-scissors.git
cd rock-paper-scissors
mvn tomcat7:deploy
