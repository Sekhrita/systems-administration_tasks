# Administración de sistemas - Tarea 1: Servicios y paquetes

- Estudiante: Felipe Carreño Aravena
  
- Profesor: Francisco Aravena Oñate

- Fecha: Septiembre 10, 2024



## Descripción del proyecto

El objetivo de este proyecto es la configuración y demostración de una serie de servicios esenciales en un entorno Debian, ejecutado en una máquina virtual en VirtualBox. Los servicios configurados incluyen:

--> Apache y PHP: Para servir aplicaciones web.
--> MySQL: Para la gestión de bases de datos con usuarios administradores y lectores.
--> SSH: Para permitir conexiones remotas seguras a la máquina virtual.
--> Samba: Para compartir carpetas entre la máquina virtual y el host.
--> Node.js: Para desplegar una aplicación web simple en el puerto 3000.

Cada servicio ha sido configurado para que sea accesible desde el host, y se han documentado los pasos necesarios para replicar el entorno, así como los errores comunes encontrados y sus soluciones.

## Intrucciones:
### Preparación del Entorno
#### Entorno de la máquina virtual 
Se utiliza Oracle VM VirtualBox v7.0.20 para emular el OS Debian 12 en su versión de terminal pura, complementada con el gestor de ventanas BSPWM.

#### Configuración de red 
La máquina virtual está configurada en modo puente para asegurar acceso a la red y permitir la comunicación con el host y otros dispositivos en la red local.

#### Usuario administrativo
Dada la instalación minimalista, se tuvo que crear manualmente un usuario con permisos de administrador (sudo) utilizando los siguientes pasos:
```bash
  sudo adduser nombre_usuario
  sudo usermod -aG sudo nombre_usuario
```
El comando "usermod" permite modificar caracteristicas de un usuario creado. La opción "-aG" permite añadir el usuario creado al grupo de administradores "sudo", sin necesidad de eliminarlo de otros grupos a los que ya pertenezca.

### Instalación y Configuración de Servicios
1.- Con el usuario creado, se inicia sesión en el Debian emulado por VirtualBox.

2.- Luego, se actualizan los repositorios para asegurarse de que el sistema tiene las versiones más recientes de los paquetes:
```bash
  sudo apt update -y && sudo apt upgrade -y
```
En este caso, la opción -y permite que la acción continúe automáticamente, sin requerir confirmación por parte del usuario para cada actualización. La && se utiliza para encadenar comandos, de modo que si el primero (apt update) se ejecuta correctamente, el segundo (apt upgrade) se ejecuta de inmediato.

3.- A continuación, se instalan los servicios requeridos para el proyecto:
#### PHP y Apache
##### Instalación:
Se logró instalar ambos programas desde los repositorios de APT. Ahora en cuanto a 'libapache2-mod-php', este es una librería que permite la conexión entre un interprete PHP y apache, por lo que para el desarrollo de este proyecto es necesario.
```bash
  sudo apt install apache2 
  sudo apt install php
  sudo apt install libapache2-mod-php
```

##### Verificación de la instalación de Apache2 y PHP:
Se espera encontrar el servicio de apacha2 corriendo, y una respuesta por parte de 'php -v' para corroborar que se encuentra entre las aplicaciones. Para esto se utiliza 'systemctl' que permite administrar los servicios.
```bash
  sudo systemctl start apache2
  sudo systemctl enable apache2
  sudo php -v
```

#### MySQL
##### Instalar el servidor de base de datos MySQL
Para abordar este punto se requiere instalar mysql-server, sin embargo este no se encuentra por defecto en el repositorio de APT. Por lo que, para realizar su instalación se tuvo que descargar el empaquetado '.deb' desde la página oficial de MySQL y añadirlo localmente al APT. Una vez realizado eso, se logró instalar lo requerido:
```bash
  wget https://dev.mysql.com/get/mysql-apt-config_0.8.32-1_all.deb
  sudo apt install ./mysql-apt-config_0.8.32-1_all.deb
  sudo apt update -y && sudo apt upgrade -y
  sudo apt install mysql-server
```

#### SSH
##### Instalación:
La instalación de SSH se logró desde el repositorio de APT:
```bash
  sudo apt install openssh-server
```

##### Validación:
Su validación puede visualizarse gracias a 'systemctl', el cual luego de habilitarlo, con el uso de 'status' deberia mostrar que está corriendo.
```bash
  sudo systemctl start ssh
  sudo systemctl enable ssh
  sudo systemctl status ssh
```

#### Samba:
##### Instalación:
La instalación nuevamente se logró gracias a los repositorios de APT ('systemctl status' para verificar):
```bash
  sudo apt install samba

  sudo systemctl start smbd
  sudo systemctl enable smbd
  sudo systemctl status smbd
```
##### Compartir una carpeta con enlace simbolico desde el host
Primero se crea la ruta que utilizará samba para las carpetas compartidas. En este caso se hará junto con 'la_carpeta_shared' el cual sera compartida con el host. El comando para esto sera mkdir con la opción -p, lo que hará que cree todas las carpetas que no existen de manera recursiva hasta llegar a la creación de 'la_carpeta_shared':
```bash
  sudo mkdir -p /srv/samba/la_carpeta_shared
```

Luego se le darán los permisos requeridos a esta carpeta con chmod 777, lo que le otorga capacidad de lectura, escritura y ejecución a todos los usuarios e instancias sobre la carpeta:
```bash
  sudo chmod 777 /srv/samba/la_carpeta_shared
```

También se debe de modificar la configuración de samba para que se adapte a las necesidades del proyecto:
```bash
  sudo vim /etc/samba/smb.conf
```

La configuración necesaria se resume en que el usuario 'sekhrita' tenga la capacidad de realizar cambios en la carpeta compartida del siguiente paso con el enlace simbolico en el home:
```bash
  [la_carpeta_shared]
    path = /home/sekhrita/la_carpeta/la_carpeta_shared
    available = yes
    read only = no
    public = yes
    browseable = yes
    writable = yes
    guest ok = yes
    valid users = sekhrita
    force user = sekhrita
    create mask = 0755
    directory mask = 0755
```
Finalmente, se debe reiniciar el servicio de samba para incorporar los cambios. Además se puede utilizar nuevamente 'systemctl status' para verificar:
```bash
  sudo systemctl restart smbd
  sudo systemctl status smbd
```

##### Enlace simbolico con la carpeta en el home con ln
```bash
  ln -s /srv/samba/la_carpeta_shared /home/sekhrita/la_carpeta/la_carpeta_shared
```

#### Node.js:
##### Instalación:
La instalación de Node.js fue sencilla, dado que se pudo obtener desde el repositorio de APT. También se instalara npm, el gestor de paquetes de Nodejs, en el caso de necesitarlo:
```bash
  sudo apt install nodejs -y && sudo apt install npm -y
```

##### Verificación:
```bash
  node -v
  npm -v
```

## Configuración Específica de Servicios
### MySQL
#### Crear una base de datos de prueba:
Una vez instalado, se debe acceder a MySQL con el root para la creación de la base de datos y de más usuarios:
```bash
  sudo mysql -u root -p
```

Primero que todo, se generará una base de datos de prueba llamada 'trials' para este proyecto. Los usuarios creados tendran permisos de gestión, ya sea de administrador o de lector, asociados a esa base en concreto; esto por seguridad.
```sql
  CREATE DATABASE trials
```

#### Configurar el acceso remoto desde el host para los dos usuarios (administrador y lector):
Ahora dentro de MySQL, y luego de la creación de la base de datos, se pueden crear distintos usuarios con diversos permisos:
```sql
  CREATE USER 'admin_host'@'%' IDENTIFIED BY '******';
  GRANT ALL PRIVILEGES ON trials.* TO 'admin_host'@'%';

  CREATE USER 'lector_host'@'%' IDENTIFIED BY '******';
  GRANT SELECT ON trials.* TO 'lector_host'@'%';

  FLUSH PRIVILEGES;
```
En líneas generales, se creo dos usuarios con permisos distintos, de lector y de administrador, para la gestión de la base de datos 'trials'. Estos usuarios no estan asociados a ninguna dirección IP concreta (con el uso de '%' luego del '@') para posteriormente conectarse al servicio de MySQL fuera de la maquina virtual.

Tambien, se ha de mencionar que se puede verificar la creación de los usuarios, y sus permisos, con los siguientes comandos MySQL:
```sql
  SELECT user, host FROM mysql.user;
  SHOW GRANTS FOR 'nombre_del_a_revisar'@'%';
```

Imagen:
[aaa]

### PHP/Apache
#### Crear una aplicación PHP simple que se muestre a través de Apache (por ejemplo, un script PHP que muestre "Hello, World!" y la fecha actual):
Para comprobar el funcionamiento de php se elaboro una apliación simple alojada en apache para que sea visible desde el navegador. Para esto se creo y el script:
Archivo:
```bash
  sudo vim /var/www/html/index.php
```

Script:
```php
  echo "¡¡Hola, mundo!! La fecha actual es: " . date('Y-m-d H:i:s');
```

Imagen:
[]

#### Creacion de un script de PHP que demuestre una conexion a la base de datos MySQL.
Luego para comprobar una conexión entre PHP y MySQL se tuvo que realizar la instalación de una dependecia:
```bash
  sudo apt install php-mysql
```

Una vez realizado eso, se pudo generar un script capaz de mostrar un mensaje de exitó al conectarse al servicio de MySQL.
Archivo:
```bash
  sudo vim /var/www/html/connection_mysql-db_user-admin.php
```

Script:
```php
  // login
  $serverName = "localhost";
  $userName = "admin_host";
  $password = "533748";
  $dbName = "trials";

  // connect
  $conn = mysqli_connect($serverName,
                         $userName,
                         $password,
                         $dbName);

  // feedback
  if ($conn) {
    echo "Connected ";
    echo "to " . $dbName . " database ";
    echo "with " . $userName . " profile";
  }
  else {
    echo "Error!!";
  }
```

#### Asegurar que esta aplicación sea accesible desde el navegador web del host.
Dada la conexión en puente a la misma red local, el host pudo conectarse gracias a la IP que registra la maquina virtual: http://192.168.100.31/index.php
Imagen:
[]

### Samba
#### Compartir una carpeta ubicada en el home del usuario usando Samba.
Para comprobar que la instalación y configuración de la carpeta compartida con samba fue exitosa, y dado el entorno minimalista que se esta usando, se tuvo que instalar smbclient para poder realizar la conexión por la terminal:
```bash
  sudo apt install smbclient 
```

Imagen de la conexión exitosa a la carpeta compartida desde el host más modificaciones (para esto se tuve que realizar el enlace al servidor apache de los pasos posteriores):
[]

#### Configurar el acceso para que pueda ser modificada desde el host.
La configuración necesaria se realizó junto a la instalación en el paso anterior.

#### Enlazar simbólicamente esta carpeta compartida a la raíz del servidor web Apache.
De la misma manera que se realizo el la vinculación desde samba a una carpeta del home, ahora se realizara lo mismo desde la carpeta del home al servicio de apache (con ln -s):
```bash
  sudo ln -s /home/usuario/compartida_publica /var/www/html/compartida_publica
```



### SSH
#### Asegurar que se pueda acceder a la máquina virtual usando SSH desde el host con las claves SSH para la autenticación segura.
Luego de haber realizado la instalación de SSH en la maquina virtual y en el host, en este último se pueden utilizar las claves para generar una autenticación segura:
```bash
  ssh-keygen
```

Una vez generada la clave, esta se envia al punto donde se realizara la conexión. En este caso se envia desde el host a la maquina virtual:
```bash
  ssh-copy-id sekhrita@192.168.100.31
```

Imagen de referencia de la conexión. En mi caso no me pide contraseña porque además hice una conexión rápida con ssh:
[]

### Node.js
#### Creación de una aplicación Node.js simple (e.g., un servidor HTTP que responda con "Hello, Node.js!") y su ejecución en el puerto por defecto (3000):
Posterior a la instalación de Node.js, se puede elaborar una apliación que se desplegara en el puerto 3000 como servicio. Para se crea un código en js:
Archivo:
```bash
  vim ~/la_carpeta/la_carpeta_shared/node_task/app.js
```

Código:
```js
  const http = require('http');
  const port = 3000;

  const requestHandler = (request, response) => {
          response.end('Hello, Node.js!');
  };

  const server = http.createServer(requestHandler);

  server.listen(port, () => {
          console.log(`Server is running on http://localhost:${port}`);
  });
```
Despliegue:
```bash
  node ~/la_carpeta/la_carpeta_shared/node_task/app.js
```

Imagen:
[]

## Errores Comunes y Soluciones: Lista de problemas encontrados y cómo se resolvieron.
Durante el proceso de instalación y configuración, encontré varios problemas que tuvieron que ser resueltos. En primer lugar, la instalación de MySQL no fue sencilla porque no estaba disponible en el repositorio de apt, lo que me obligó a descargar el empaquetado desde la página oficial de MySQL. Además, hubo problemas para habilitar la conexión remota mediante la dirección IP, ya que los usuarios habían sido configurados inicialmente solo para acceder desde localhost. Esto se resolvió al permitir que los usuarios se conectaran desde direcciones IP variables y habilitando el puerto 3306, que es el puerto predeterminado para MySQL, para permitir dichas conexiones. También surgieron dificultades con PHP, especialmente al intentar desarrollar un script que pudiera conectarse de manera remota a MySQL y devolver una respuesta. El script no generaba ninguna salida, pero, tras seguir guías más recientes, identifiqué que el problema residía en errores de sintaxis que finalmente logré corregir.

Tambien se ha de mencionar que al estar utilizando bspwm, y tener varios escritorios y terminales abiertas donde estoy trabajando en paralelo, el historial de los comandos utilizados no es el que deberia; muchos se pierden o sobrelapan. Por este motivo no podre entregar un history.log completo.