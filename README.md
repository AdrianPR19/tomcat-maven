# Tomcat y Maven - Versión 1.0.0

## 2. Instalación de OpenJDK

Instalamos el kit de desarrollo de Java:
```bash
sudo apt install -y openjdk-11-jdk
```

## 2.1 Instalación y Configuración de Tomcat

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

1. Accedemos a la GUI de Tomcat con el usuario configurado previamente.alumno / 1234
![Captura del funcionamiento](/2.png)
2. Buscamos la sección para desplegar archivos WAR.
3. Seleccionamos el archivo `tomcat1.war`.
4. Pulsamos **Desplegar** y verificamos que la aplicación esté disponible en el directorio `/tomcat1`.

![Captura del funcionamiento](/3.png)
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
```
3. Para comprobar su funcionamiento entramos en http://localhost:8080/manager/text



# Despliegue con Maven en Vagrant

Este archivo README.md describe cómo realizar el despliegue de una aplicación en Tomcat utilizando Maven en una máquina Vagrant nueva con Debian. A continuación, se detallan los pasos a seguir para la instalación de Maven, configuración de Tomcat y despliegue de una aplicación, tanto de prueba como clonada desde un repositorio.

## Requisitos Previos

- Máquina virtual Vagrant con Debian.
- Tomcat instalado y configurado.
- Acceso al terminal con privilegios de sudo.

## 1. Instalación de Maven

### Instalación mediante gestor de paquetes APT

Primero, actualiza los repositorios:

```bash
sudo apt-get update
```

Luego, instala Maven:

```bash
sudo apt-get -y install maven
```

Para comprobar que Maven se ha instalado correctamente, ejecuta:

```bash
mvn --version
```

Deberías ver la versión de Maven instalada.

### Instalación manual (opcional)

Si prefieres instalar una versión más reciente de Maven manualmente, puedes seguir la guía oficial de instalación de Maven.

## 2. Configuración de Maven para Despliegue en Tomcat

### Modificación de usuarios y roles en Tomcat

Edita el archivo de usuarios y roles de Tomcat (`/etc/tomcat9/tomcat-users.xml`) para agregar los roles necesarios:

```xml
<role rolename="manager-gui"/>
<role rolename="manager-status"/>
<role rolename="manager-script"/>
<role rolename="manager-jmx"/>
<user username="alumno" password="1234" roles="admin, admin-gui, manager, manager-gui"/>
<user username="deploy" password="1234" roles="manager-script"/>
```

Asegúrate de tener un usuario con el rol `manager-script` para el despliegue.

### Configuración de Maven

Edita el archivo de configuración de Maven (`/etc/maven/settings.xml`) para incluir el servidor Tomcat:

```xml
<servers>
    <server>
        <id>Tomcat</id>
        <username>deploy</username>
        <password>1234</password>
    </server>
</servers>
```

Cambia el nombre de usuario y la contraseña según lo hayas configurado en el archivo `tomcat-users.xml`.

## 3. Generación de una Aplicación de Prueba

Desde tu directorio personal, genera una aplicación de prueba:

```bash
mvn archetype:generate -DgroupId=org.zaidinvergeles \
                       -DartifactId=tomcat-war \
                       -deployment \
                       -DarchetypeArtifactId=maven-archetype-webapp \
                       -DinteractiveMode=false
```

Esto creará un proyecto básico de aplicación web.

Accede al directorio creado:

```bash
cd tomcat-war-deployment
```

### Modificación del POM para Despliegue en Tomcat

Abre el archivo `pom.xml` y agrega el siguiente bloque dentro de la sección `<build>`:

```xml
<build>
    <finalName>tomcat-war-deployment</finalName>
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

Este bloque configura el plugin de Maven para desplegar la aplicación en el servidor Tomcat.

## 4. Despliegue de la Aplicación de Prueba

Para realizar el despliegue de la aplicación, ejecuta el siguiente comando en el directorio del proyecto:

```bash
mvn tomcat7:deploy
```

Si el despliegue es exitoso, verás un mensaje similar a:

```bash
[INFO] Deploying war to http://localhost:8080/despliegue
```

Accede a la aplicación a través de la URL:

```
http://localhost:8080/despliegue
```

## Despliegue de la Aplicación de Prueba y Nueva Aplicación (Rock Paper Scissors) usando Maven en Tomcat

Este documento explica de forma detallada cómo realizar el despliegue de una aplicación web en Tomcat utilizando Maven, con dos aplicaciones distintas: una aplicación de prueba y la aplicación "Rock Paper Scissors" clonada desde un repositorio. A continuación, se detallan los pasos necesarios para completar la tarea.

### 1. Despliegue de la Aplicación de Prueba

#### Paso 1: Generar la aplicación de prueba con Maven

Abre la terminal en tu máquina Vagrant.

Dirígete al directorio de tu usuario:

```bash
cd
```

Ejecuta el siguiente comando para generar una aplicación de prueba utilizando Maven:

```bash
mvn archetype:generate -DgroupId=org.zaidinvergeles \
                       -DartifactId=tomcat-war \
                       -deployment \
                       -DarchetypeArtifactId=maven-archetype-webapp \
                       -DinteractiveMode=false
```

Este comando crea un proyecto básico con la estructura de un archivo WAR (web application archive) que puedes desplegar en Tomcat. Maven descargará automáticamente los arquetipos necesarios para crear la estructura de la aplicación.

Una vez que el comando termine, tendrás un nuevo directorio llamado `tomcat-war-deployment`. Accede a este directorio:

```bash
cd tomcat-war-deployment
```

#### Paso 2: Modificar el archivo pom.xml para habilitar el despliegue en Tomcat

Abre el archivo `pom.xml` en un editor de texto (puedes usar nano, vim, o cualquier otro editor):

```bash
nano pom.xml
```

Dentro del archivo `pom.xml`, busca la sección `<build></build>` y añade el siguiente bloque de configuración para el plugin de Tomcat:

```xml
<build>
    <finalName>tomcat-war-deployment</finalName>
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

Aquí están las configuraciones clave:

- `url`: Especifica la URL de Tomcat para la interfaz de gestión (`/manager/text`).
- `server`: Define el nombre del servidor que configuramos previamente en el archivo `settings.xml` de Maven.
- `path`: Especifica el contexto o la URL a la que se desplegará la aplicación (en este caso, `/despliegue`).

Guarda y cierra el archivo (CTRL+X, luego Y para guardar en nano).

#### Paso 3: Desplegar la aplicación en Tomcat

Para desplegar la aplicación en Tomcat, ejecuta el siguiente comando en la terminal:

```bash
mvn tomcat7:deploy
```

Maven se encargará de empaquetar la aplicación y cargarla en el servidor Tomcat usando las configuraciones definidas.

Si el despliegue es exitoso, deberías ver un mensaje similar a:

```bash
[INFO] Deploying war to http://localhost:8080/despliegue
```

Ahora, abre tu navegador y accede a la URL:

```
http://localhost:8080/despliegue
```

Verás la aplicación desplegada correctamente.

### 2. Despliegue de la Nueva Aplicación (Rock Paper Scissors)

#### Paso 1: Clonar el repositorio de la nueva aplicación

Abre la terminal y clona el repositorio de la aplicación "Rock Paper Scissors" desde GitHub:

```bash
git clone https://github.com/cameronmcnz/rock-paper-scissors.git
```

Dirígete al directorio del repositorio clonado:

```bash
cd rock-paper-scissors
```

#### Paso 2: Cambiar a la rama correcta

Cambia a la rama `patch-1`, que contiene los cambios necesarios para el despliegue:

```bash
git checkout patch-1
```

#### Paso 3: Modificar el archivo pom.xml para habilitar el despliegue en Tomcat

Abre el archivo `pom.xml` del repositorio clonado en un editor de texto:

```bash
nano pom.xml
```

Al igual que en el caso de la aplicación de prueba, añade el siguiente bloque de configuración dentro de la sección `<build></build>`:

```xml
<build>
    <finalName>rock-paper-scissors</finalName>
    <plugins>
        <plugin>
            <groupId>org.apache.tomcat.maven</groupId>
            <artifactId>tomcat7-maven-plugin</artifactId>
            <version>2.2</version>
            <configuration>
                <url>http://localhost:8080/manager/text</url>
                <server>Tomcat</server>
                <path>/rockpaperscissors</path>
            </configuration>
        </plugin>
    </plugins>
</build>
```

Las configuraciones son las mismas que en el paso anterior:

- `url`: Especifica la URL de la interfaz de gestión de Tomcat.
- `server`: Define el nombre del servidor configurado en Maven.
- `path`: Define el contexto de la aplicación, que será `/rockpaperscissors` en este caso.

Guarda y cierra el archivo.

#### Paso 4: Desplegar la aplicación "Rock Paper Scissors"

Al igual que en el caso anterior, ejecuta el siguiente comando para desplegar la nueva aplicación en Tomcat:

```bash
mvn tomcat7:deploy
```

Maven empaquetará la aplicación y la desplegará en el servidor Tomcat.

Si todo sale bien, deberías ver un mensaje como:

```bash
[INFO] Deploying war to http://localhost:8080/rockpaperscissors
```

#### Paso 5: Verificar el despliegue en el navegador

Abre tu navegador y accede a la siguiente URL para comprobar que la aplicación está desplegada correctamente:

```
http://localhost:8080/rockpaperscissors
```

Deberías ver la aplicación "Rock Paper Scissors" funcionando.