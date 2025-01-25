Vagrant.configure("2") do |config|
  config.vm.box = "debian/bullseye64"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"
    vb.cpus = 2
  end

  config.vm.network "forwarded_port", guest: 8080, host: 8080

  config.vm.provision "shell", inline: <<-SHELL
    # Actualizamos los paquetes del sistema
    sudo apt-get update

    # Instalamos OpenJDK, Tomcat9 y Maven
    sudo apt-get install -y openjdk-11-jdk tomcat9 tomcat9-admin maven git

    # Configuramos usuarios para Tomcat
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
<user username="alumno" password="1234" roles="admin,admin-gui,manager,manager-gui"/>
<user username="deploy" password="1234" roles="manager-script"/>
</tomcat-users>
EOF

    # Configuramos Maven para despliegue en Tomcat
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

    # Reiniciamos Tomcat
    sudo systemctl restart tomcat9

    # Clonamos el repositorio y cambiamos a la rama patch-1
    git clone https://github.com/cameronmcnz/rock-paper-scissors.git
    cd rock-paper-scissors
    git checkout patch-1

    # Editamos el pom.xml para agregar el plugin de Maven
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

  SHELL
end
