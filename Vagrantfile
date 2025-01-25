Vagrant.configure("2") do |config|
  config.vm.box = "debian/bullseye64"
  
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"
    vb.cpus = 2
  end
  
  config.vm.network "forwarded_port", guest: 8080, host: 8080
  
  config.vm.provision "shell", inline: <<-SHELL
    sudo apt-get update

    # Aquí instalaremos el opendjk
    sudo apt-get install -y openjdk-11-jdk

    # Aquí el tomcat9
    sudo apt-get install -y tomcat9

    # creamos el grupo tomcat9
    sudo groupadd tomcat9

    # y su usuario, para luego posteriormente poder acceder
    sudo useradd -s /bin/false -g tomcat9 -d /etc/tomcat9 tomcat9

    sudo systemctl start tomcat9
    sudo systemctl enable tomcat9

    # Instalamos administrador web de tomcat 
    sudo apt-get install -y tomcat9-admin

    # Configuramos los usuarios del archivo "tomcat-users.xml"
    sudo tee /etc/tomcat9/tomcat-users.xml > /dev/null << EOF
<?xml version="1.0" encoding="UTF-8"?>
<tomcat-users xmlns="http://tomcat.apache.org/xml"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://tomcat.apache.org/xml tomcat-users.xsd"
version="1.0">
<role rolename="admin"/>
<role rolename="admin-gui"/>
<role rolename="manager"/>
<role rolename="manager-gui"/>
<role rolename="manager-status"/>
<role rolename="manager-script"/>
<role rolename="manager-jmx"/>
<user username="alumno"
password="1234"
roles="admin,admin-gui,manager,manager-gui"/>
<user username="deploy" 
password="1234" 
roles="manager-script"/>
</tomcat-users>
EOF

    # Modificamos el archivo de context.xml para el acceso remoto
    sudo tee /usr/share/tomcat9-admin/host-manager/META-INF/context.xml > /dev/null << EOF
<?xml version="1.0" encoding="UTF-8"?>
<Context antiResourceLocking="false" privileged="true" >
  <CookieProcessor className="org.apache.tomcat.util.http.Rfc6265CookieProcessor"
                   sameSiteCookies="strict" />
  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="\d+\.\d+\.\d+\.\d+" />
  <Manager sessionAttributeValueClassNameFilter="java\.lang\.(?:Boolean|Integer|Long|Number|String)|org\.apache\.catalina\.filters\.CsrfPreventionFilter\$LruCache(?:\$1)?|java\.util\.(?:Linked)?HashMap"/>
</Context>
EOF

    # Instalamos el maven
    sudo apt-get install -y maven

    # y lo configuramos
    sudo tee /etc/maven/settings.xml > /dev/null << EOF
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 
                      https://maven.apache.org/xsd/settings-1.0.0.xsd">
  <servers>
    <server>
      <id>Tomcat</id>
      <username>deploy</username>
      <password>1234</password>
    </server>
  </servers>
</settings>
EOF

    # Clonamos el repositorio y cambiamos a la rama patch-1
git clone https://github.com/cameronmcnz/rock-paper-scissors.git
cd rock-paper-scissors
git checkout patch-1

# Verificar si el archivo pom.xml existe
if [ ! -f pom.xml ]; then
  echo "El archivo pom.xml no existe, creándolo..."
  # Crear un archivo pom.xml básico si no existe
  cat << EOF > pom.xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example</groupId>
  <artifactId>rock-paper-scissors</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>war</packaging>
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.tomcat.maven</groupId>
        <artifactId>tomcat7-maven-plugin</artifactId>
        <version>2.2</version>
        <configuration>
          <url>http://localhost:8080/manager/text</url>
          <server>Tomcat</server>
          <path>/rock-paper-scissors</path>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>
EOF
fi

# Luego agrega el plugin con sed si el archivo ya está presente o se acaba de crear
sed -i '/<plugins>/a \
<plugin>\
  <groupId>org.apache.tomcat.maven</groupId>\
  <artifactId>tomcat7-maven-plugin</artifactId>\
  <version>2.2</version>\
  <configuration>\
    <url>http://localhost:8080/manager/text</url>\
    <server>Tomcat</server>\
    <path>/rock-paper-scissors</path>\
  </configuration>\
</plugin>' pom.xml

# Construimos y desplegamos la aplicación
mvn clean install tomcat7:deploy

# Reiniciamos el tomcat
sudo systemctl restart tomcat9

sudo apt-get install -y git


  SHELL
end
