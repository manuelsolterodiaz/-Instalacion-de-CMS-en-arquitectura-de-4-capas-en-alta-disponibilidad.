# Instalación de CMS en arquitectura de 4 capas en alta disponibilidad.
Contiene los ficheros VagrantFile y de aprovisionamiento necesarios para esta tarea.
## Capas
- Capa 1: Capa pública. Balanceador de carga con Nginx.
- Capa 2: Backend (Dos servidores web y un servidor NFS y motor PHP_FPM).
- Capa 3: Balanceador de Base de Datos con HAProxy.
- Capa 4: Base de Datos (Dos servidores de base de datos con MariaDB)
## Índice

1. [Introducción](#id1)
2. [Programas y herramientas](#id2)
3. [Scripts](#id3)

# Introducción <a name="id1"></a>

En esta práctica se va a realizar el despliegue de una aplicación web que está alojada en un repositorio público de GitHub en alta disponibilidad.

La infraestructura montada en vagrant se monta un balanceador con Nginx, dos servidores de web , un servidor NFS con la aplicación alojada en una carpeta a la que acceder los servidores para mostrarla en el navegador, después en la misma máquina hay instalado el motor de PHP-FPM, seguidamente hay un servidor que actúa de balanceador en los servidores de base de datos, se ha utilizado HAProxy, y por último dos servidores de base de datos donde se aloja la base de datos.

### Direccionamiento

Para esta práctica se han usado las diferentes redes:
- Red publica balanceador: 192.168.10.0/24
- Red privada servidores web: 192.168.20.0/24
- Red privada nfs: 192.168.30.0/24
- Red privada base de datos: 192.168.40.0/24
##### Por cada máquina
- Balanceador: Utiliza dos redes.
    - Red pública (redpublica): 192.168.10.0/24 - utiliza - 192.168.10.10
    - Red privada (redweb): 192.168.20.0/24 - utiliza - 192.168.20.14
- Server Web 1: Utiliza dos redes.
    - Red privada (redweb): 192.168.20.0/24 - utiliza - 192.168.20.10
    - Red privada (rednfs): 192.168.30.0/24 - utiliza - 192.168.30.11
- Server Web 2: Utiliza dos redes.
    - Red privada (redweb): 192.168.20.0/24 - utiliza - 192.168.20.11
    - Red privada (rednfs): 192.168.30.0/24 - utiliza - 192.168.30.12
- Server NFS: Utiliza dos redes.
    - Red privada (redweb): 192.168.20.0/24 - utiliza - 192.168.20.13
    - Red privada (rednfs): 192.168.30.0/24 - utiliza - 192.168.30.13
- Server HAProxy: Utiliza dos redes.
    - Red privada (rednfs): 192.168.30.0/24 - utiliza - 192.168.30.10
    - Red privada (reddatabase): 192.168.40.0/24 - utiliza - 192.168.40.10
- Server DB1: Utiliza una única red.
    - Red privada (reddatabase): 192.168.40.0/24 - utiliza - 192.168.40.11
- Server DB2: Utiliza una única red.
    - Red privada (reddatabase): 192.168.40.0/24 - utiliza - 192.168.40.12

# Programas / herramientas utilizados <a name="id2"></a>
Programas/herramientas utilizados:
- Vagrant/Virtualbox: para desplegar toda la infraestructura.
- Nginx para el balanceador, servir la aplicación web.
- Preprocesador PHP-FPM: sirven para gestionar de manera eficiente los procesos de PHP y optimizar significativamente el rendimiento de las aplicaciones web.
- NFS: sirve para compartir archivos y directorios a través de una red, permitiendo que múltiples ordenadores (clientes) accedan a ellos como si estuvieran en su propio disco local.
- HAProxy: se usa con bases de datos para distribuir las conexiones y consultas entre múltiples servidores de bases de datos.
- MariaDB: utilizado como sistema gestor de base de datos para poder guardar los registros de la aplicación.
- MariaDB-client: utilizado para acceder desde el server HAProxy a la base de datos.
- Galera: para poder hacer el cluster de las base de datos.

## Scripts <a name="id3"></a>

## Orden para el correcto levantamiento/funcionamiento de las máquinas
```
vagrant up serverdatos1ManuelSoltero  serverdatos2ManuelSoltero proxyBBDDManuelSoltero serverNFSManuelSoltero  serverweb1ManuelSoltero serverweb2ManuelSoltero balanceadorManuelSoltero
```
 ### Vagrantfile
``` ruby
# -- mode: ruby --
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
config.vm.box = "debian/bookworm64"

  config.vm.define "balanceadorManuelSoltero" do |balanceador|
     balanceador.vm.network "private_network", ip:"192.168.10.10", virtualbox__intnet: "redpublica"
     balanceador.vm.network "private_network", ip:"192.168.20.14", virtualbox__intnet: "redweb"
     balanceador.vm.network "forwarded_port", guest: 80, host: 8080
     balanceador.vm.provision "shell", path: "balanceador.sh"
     balanceador.vm.hostname = "balanceadorManuelSoltero"
  end
  
  config.vm.define "serverweb1ManuelSoltero" do |web1|
     web1.vm.network "private_network", ip:"192.168.20.10", virtualbox__intnet: "redweb"
     web1.vm.network "private_network", ip:"192.168.30.11", virtualbox__intnet: "rednfs"
     web1.vm.provision "shell", path: "web1.sh"
     web1.vm.hostname = "serverweb1ManuelSoltero"
  end
  
  config.vm.define "serverweb2ManuelSoltero" do |web2|
     web2.vm.network "private_network", ip:"192.168.20.11", virtualbox__intnet: "redweb"
     web2.vm.network "private_network", ip:"192.168.30.12", virtualbox__intnet: "rednfs"
     web2.vm.provision "shell", path: "web2.sh"
     web2.vm.hostname = "serverweb2ManuelSoltero"
  end
  
  config.vm.define "serverNFSManuelSoltero" do |nfs|
     nfs.vm.network "private_network", ip:"192.168.20.13", virtualbox__intnet: "redweb"
     nfs.vm.network "private_network", ip:"192.168.30.13", virtualbox__intnet: "rednfs"
     nfs.vm.provision "shell", path: "nfs.sh"
     nfs.vm.hostname = "serverNFSManuelSoltero"
  end
  
  config.vm.define "proxyBBDDManuelSoltero" do |proxy|
     proxy.vm.network "private_network", ip:"192.168.30.10", virtualbox__intnet: "rednfs"
     proxy.vm.network "private_network", ip:"192.168.40.10", virtualbox__intnet: "reddatabase"
     proxy.vm.provision "shell", path: "proxy.sh"
     proxy.vm.hostname = "proxyBBDDManuelSoltero"
  end
  
  config.vm.define "serverdatos1ManuelSoltero" do |db1|
     db1.vm.network "private_network", ip:"192.168.40.11", virtualbox__intnet: "reddatabase"
     db1.vm.provision "shell", path: "db1.sh"
     db1.vm.hostname = "serverdatos1ManuelSoltero"
  end
  
  config.vm.define "serverdatos2ManuelSoltero" do |db2|
     db2.vm.network "private_network", ip:"192.168.40.12", virtualbox__intnet: "reddatabase"
     db2.vm.provision "shell", path: "db2.sh"
     db2.vm.hostname = "serverdatos2ManuelSoltero"
  end

  # Para crear las maquinas en orden
  # vagrant up serverdatos1ManuelSoltero  serverdatos2ManuelSoltero proxyBBDDManuelSoltero serverNFSManuelSoltero  serverweb1ManuelSoltero serverweb2ManuelSoltero balanceadorManuelSoltero


  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # vagrant box outdated. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Disable the default share of the current code directory. Doing this
  # provides improved isolation between the vagrant box and your host
  # by making sure your Vagrantfile isn't accessible to the vagrant box.
  # If you use this you may want to enable additional shared subfolders as
  # shown above.
  # config.vm.synced_folder ".", "/vagrant", disabled: true

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Ansible, Chef, Docker, Puppet and Salt are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL
end
```

### Balanceador

```
#!/bin/bash

# Script de aprovisionamiento del Balanceador de Carga Nginx
# Capa 1 - Expuesta a red pública
# Manuel Soltero Díaz


echo "=== Actualizando sistema ==="
apt-get update


echo "=== Instalando Nginx ==="
apt-get install -y nginx

echo "=== Configurando Nginx como balanceador de carga ==="
cat > /etc/nginx/conf.d/load-balancer.conf <<'EOF'
upstream backend_servers {
    server 192.168.20.10:80;
    server 192.168.20.11:80;
}

server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://backend_servers;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
EOF

# Eliminar configuración por defecto
rm -f /etc/nginx/sites-enabled/default

echo "=== Habilitando y reiniciando Nginx ==="
systemctl enable nginx
systemctl restart nginx

echo "=== Configurando IP forwarding ==="
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
sysctl -p

echo "=== Balanceador configurado correctamente ==="
```

### Server Web 1

```
#!/bin/bash

# Script de aprovisionamiento del Servidor Web 1
# Capa 2 - Backend (Nginx + montaje NFS)
# Manuel Soltero Díaz


echo "=== Actualizando sistema ==="
apt-get update


echo "=== Instalando Nginx y NFS client ==="
apt-get install -y nginx nfs-common

echo "=== Creando directorio para montaje NFS ==="
mkdir -p /var/www/html

echo "=== Configurando montaje NFS ==="
echo "192.168.30.13:/var/www/html /var/www/html nfs defaults 0 0" >> /etc/fstab
mount -a

echo "=== Configurando Nginx para usar PHP-FPM remoto ==="
cat > /etc/nginx/sites-available/default <<'EOF'
server {
    listen 80;
    server_name _;
    root /var/www/html;

    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass 192.168.30.13:9000;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }
}
EOF

echo "=== Ajustando permisos ==="
chown -R www-data:www-data /var/www/html
chmod -R 755 /var/www/html

echo "=== Reiniciando Nginx ==="
systemctl enable nginx
systemctl restart nginx

echo "=== Servidor Web 1 configurado correctamente ==="
```
### Server Web 2

```
#!/bin/bash

# Script de aprovisionamiento del Servidor Web 2
# Capa 2 - Backend (Nginx + montaje NFS)
# Manuel Soltero Díaz

echo "=== Actualizando sistema ==="
apt-get update

echo "=== Instalando Nginx y NFS client ==="
apt-get install -y nginx nfs-common

echo "=== Creando directorio para montaje NFS ==="
mkdir -p /var/www/html

echo "=== Configurando montaje NFS ==="
echo "192.168.30.13:/var/www/html /var/www/html nfs defaults 0 0" >> /etc/fstab
mount -a

echo "=== Configurando Nginx para usar PHP-FPM remoto ==="
cat > /etc/nginx/sites-available/default <<'EOF'
server {
    listen 80;
    server_name _;
    root /var/www/html;

    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass 192.168.30.13:9000;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }
}
EOF

echo "=== Ajustando permisos ==="
chown -R www-data:www-data /var/www/html
chmod -R 755 /var/www/html

echo "=== Reiniciando Nginx ==="
systemctl enable nginx
systemctl restart nginx

echo "=== Servidor Web 2 configurado correctamente ==="
```
### Server NFS + PHP-FPM

```
#!/bin/bash

# Script de aprovisionamiento del Servidor NFS con PHP-FPM
# Capa 2 - Backend (Almacenamiento compartido y motor PHP)
# Manuel Soltero Díaz

echo "=== Actualizando sistema ==="
apt-get update

echo "=== Instalando NFS Server, PHP-FPM y extensiones ==="
apt-get install -y nfs-kernel-server php-fpm php-mysql php-cli php-curl php-gd php-mbstring php-xml php-zip unzip git

echo "=== Creando directorio compartido ==="
mkdir -p /var/www/html

echo "=== Configurando exportación NFS ==="
cat > /etc/exports <<'EOF'
/var/www/html 192.168.30.11(rw,sync,no_subtree_check,no_root_squash)
/var/www/html 192.168.30.12(rw,sync,no_subtree_check,no_root_squash)
EOF

exportfs -a

echo "=== Configurando PHP-FPM para escuchar en todas las interfaces ==="
PHP_VERSION=$(php -r 'echo PHP_MAJOR_VERSION.".".PHP_MINOR_VERSION;')

# Escuchar en 0.0.0.0:9000 para aceptar conexiones remotas
sed -i 's/listen = \/run\/php\/php.*-fpm.sock/listen = 0.0.0.0:9000/' /etc/php/$PHP_VERSION/fpm/pool.d/www.conf

# Permitir conexiones solo desde los servidores web
sed -i 's/;listen.allowed_clients/listen.allowed_clients/' /etc/php/$PHP_VERSION/fpm/pool.d/www.conf
sed -i '/listen.allowed_clients/d' /etc/php/$PHP_VERSION/fpm/pool.d/www.conf
echo "listen.allowed_clients = 192.168.30.11,192.168.30.12" >> /etc/php/$PHP_VERSION/fpm/pool.d/www.conf

echo "=== Descargando aplicación de usuarios desde GitHub ==="
cd /tmp
git clone https://github.com/josejuansanchez/iaw-practica-lamp.git
cp -r iaw-practica-lamp/src/* /var/www/html/

echo "=== Configurando la base de datos en la aplicación ==="
cat > /var/www/html/config.php <<'EOF'
<?php
define('DB_HOST', '192.168.30.10:3306');
define('DB_NAME', 'lamp_db');
define('DB_USER', 'manuelsoltero');
define('DB_PASSWORD', 'abcd');

$mysqli = mysqli_connect(DB_HOST, DB_USER, DB_PASSWORD, DB_NAME);

if (!$mysqli) {
    die("Error de conexión: " . mysqli_connect_error());
}
?>
EOF

echo "=== Ajustando permisos ==="
chown -R www-data:www-data /var/www/html
chmod -R 755 /var/www/html

echo "=== Reiniciando servicios ==="
systemctl enable nfs-kernel-server
systemctl restart nfs-kernel-server
systemctl enable php$PHP_VERSION-fpm
systemctl restart php$PHP_VERSION-fpm

echo "=== Verificando configuración PHP-FPM ==="
netstat -tlnp | grep 9000 || ss -tlnp | grep 9000

echo "=== Servidor NFS con PHP-FPM configurado correctamente ==="
echo "PHP-FPM escuchando en: 0.0.0.0:9000"
echo "NFS compartiendo: /var/www/html"
```
### Server HAProxy

```
#!/bin/bash

# Script de aprovisionamiento del Proxy de Base de Datos HAProxy
# Capa 3 - Balanceador de bases de datos
# Manuel Soltero Díaz

echo "=== Actualizando sistema ==="
apt-get update

echo "=== Instalando HAProxy ==="
apt-get install -y haproxy mariadb-client


echo "=== Configurando HAProxy para balanceo de bases de datos ==="
cat > /etc/haproxy/haproxy.cfg <<'EOF'
global
    log /dev/log local0
    log /dev/log local1 notice
    chroot /var/lib/haproxy
    stats socket /run/haproxy/admin.sock mode 660 level admin
    stats timeout 30s
    user haproxy
    group haproxy
    daemon

defaults
    log     global
    mode    tcp
    option  tcplog
    option  dontlognull
    timeout connect 5000
    timeout client  50000
    timeout server  50000

listen mysql-cluster
    bind *:3306
    mode tcp
    balance roundrobin
    option mysql-check user haproxy
    server db1 192.168.40.11:3306 check
    server db2 192.168.40.12:3306 check

listen stats
    bind *:8080
    mode http
    stats enable
    stats uri /stats
    stats refresh 30s
    stats realm HAProxy\ Statistics
    stats auth admin:admin
EOF

echo "=== Habilitando HAProxy ==="
systemctl enable haproxy
systemctl restart haproxy

echo "=== Proxy de base de datos configurado correctamente ==="
echo "=== Estadísticas disponibles en http://192.168.30.10:8080/stats (admin/admin) ==="
```

### Server DB 1

```
#!/bin/bash

# Script de aprovisionamiento del Servidor de Base de Datos 1
# Capa 4 - Datos (MariaDB Galera Cluster - Nodo 1)
# Manuel Soltero Díaz

echo "=== Actualizando sistema ==="
apt-get update

echo "=== Instalando MariaDB Server y Galera ==="
DEBIAN_FRONTEND=noninteractive apt-get install -y mariadb-server mariadb-client galera-4 rsync git

echo "=== Deteniendo MariaDB para configuración ==="
systemctl stop mariadb || true
killall -9 mysqld 2>/dev/null || true
sleep 3

echo "=== Configurando Galera Cluster ==="
cat > /etc/mysql/mariadb.conf.d/60-galera.cnf <<'EOF'
[mysqld]
# Configuración básica
bind-address = 0.0.0.0

# Configuración de Galera
wsrep_on = ON
wsrep_provider = /usr/lib/galera/libgalera_smm.so

# Cluster configuration
wsrep_cluster_name = "galera_cluster"
wsrep_cluster_address = "gcomm://192.168.40.11,192.168.40.12"

# Node configuration
wsrep_node_address = "192.168.40.11"
wsrep_node_name = "serverdatos1ManuelSoltero"

# SST method
wsrep_sst_method = rsync

# Configuración para replicación
binlog_format = row
default_storage_engine = InnoDB
innodb_autoinc_lock_mode = 2
EOF

echo "=== Limpiando estado previo de Galera ==="
rm -f /var/lib/mysql/grastate.dat

echo "=== Iniciando el cluster (bootstrap en primer nodo) ==="
galera_new_cluster

echo "=== Esperando a que MariaDB esté disponible ==="
sleep 10

# Verificar que MariaDB está corriendo
if ! systemctl is-active --quiet mariadb; then
    echo "ERROR: MariaDB no arrancó correctamente"
    systemctl status mariadb
    journalctl -u mariadb -n 50
    exit 1
fi

echo "=== Descargando script SQL de la aplicación ==="
cd /tmp

git clone https://github.com/josejuansanchez/iaw-practica-lamp.git


echo "=== Creando base de datos y usuario ==="
mysql <<'MYSQL_SCRIPT'
-- Crear base de datos
CREATE DATABASE IF NOT EXISTS lamp_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- Crear usuario para la aplicación
CREATE USER IF NOT EXISTS 'manuelsoltero'@'%' IDENTIFIED BY 'abcd';
GRANT ALL PRIVILEGES ON lamp_db.* TO 'manuelsoltero'@'%';

-- Crear usuario para HAProxy health checks
CREATE USER IF NOT EXISTS 'haproxy'@'%';

-- Crear usuario para SST (State Snapshot Transfer)
CREATE USER IF NOT EXISTS 'sstuser'@'localhost' IDENTIFIED BY 'sstpass';
GRANT RELOAD, LOCK TABLES, PROCESS, REPLICATION CLIENT ON *.* TO 'sstuser'@'localhost';

FLUSH PRIVILEGES;
MYSQL_SCRIPT

echo "=== Importando tablas desde el script SQL del repositorio ==="
 
    mysql lamp_db < /tmp/iaw-practica-lamp/db/database.sql
    echo "Tablas importadas correctamente"


echo "=== Verificando estado del cluster ==="
mysql -e "SHOW STATUS LIKE 'wsrep_cluster_size';" || true
mysql -e "SHOW STATUS LIKE 'wsrep_ready';" || true

echo "=== Habilitando MariaDB ==="
systemctl enable mariadb

echo "=== Mostrando tablas creadas ==="
mysql lamp_db -e "SHOW TABLES;" || true

echo "=== Servidor de base de datos 1 (Galera Nodo 1) configurado correctamente ==="
echo "=== Estado del cluster: ==="
mysql -e "SHOW STATUS LIKE 'wsrep_%';" | grep -E "wsrep_cluster_size|wsrep_cluster_status|wsrep_ready|wsrep_connected" || true

```

### Server DB 2

```
#!/bin/bash

# Script de aprovisionamiento del Servidor de Base de Datos 2
# Capa 4 - Datos (MariaDB Galera Cluster - Nodo 2)
# Manuel Soltero Díaz

echo "=== Actualizando sistema ==="
apt-get update


echo "=== Instalando MariaDB Server y Galera ==="
DEBIAN_FRONTEND=noninteractive apt-get install -y mariadb-server mariadb-client galera-4 rsync

echo "=== Deteniendo MariaDB para configuración ==="
systemctl stop mariadb || true
killall -9 mysqld 2>/dev/null || true
sleep 3

echo "=== Configurando Galera Cluster ==="
cat > /etc/mysql/mariadb.conf.d/60-galera.cnf <<'EOF'
[mysqld]
# Configuración básica
bind-address = 0.0.0.0

# Configuración de Galera
wsrep_on = ON
wsrep_provider = /usr/lib/galera/libgalera_smm.so

# Cluster configuration
wsrep_cluster_name = "galera_cluster"
wsrep_cluster_address = "gcomm://192.168.40.11,192.168.40.12"

# Node configuration
wsrep_node_address = "192.168.40.12"
wsrep_node_name = "serverdatos2ManuelSoltero"

# SST method
wsrep_sst_method = rsync

# Configuración para replicación
binlog_format = row
default_storage_engine = InnoDB
innodb_autoinc_lock_mode = 2
EOF

echo "=== Limpiando estado previo de Galera ==="
rm -f /var/lib/mysql/grastate.dat

echo "=== Esperando a que el nodo 1 esté disponible ==="
echo "Esperando 40 segundos para que el cluster se inicialice en db1..."
sleep 40



echo "=== Iniciando MariaDB y uniéndose al cluster ==="
systemctl start mariadb

echo "=== Esperando a que MariaDB esté disponible ==="
sleep 15

# Verificar que MariaDB está corriendo

    
    echo "=== Verificando estado del cluster ==="
    mysql -e "SHOW STATUS LIKE 'wsrep_cluster_size';" || echo "Esperando sincronización..."
    sleep 5
    mysql -e "SHOW STATUS LIKE 'wsrep_ready';" || echo "Esperando sincronización..."
    
    echo "=== Verificando replicación de datos ==="
    mysql -e "USE lamp_db; SHOW TABLES;" 2>/dev/null || echo "Base de datos aún sincronizando..."
    sleep 3
    mysql -e "USE lamp_db; SELECT * FROM usuarios LIMIT 5;" 2>/dev/null || echo "Tablas aún sincronizando..."


echo "=== Habilitando MariaDB ==="
systemctl enable mariadb

echo "=== Servidor de base de datos 2 (Galera Nodo 2) configurado correctamente ==="
echo "=== Estado del cluster: ==="
mysql -e "SHOW STATUS LIKE 'wsrep_%';" 2>/dev/null | grep -E "wsrep_cluster_size|wsrep_cluster_status|wsrep_ready|wsrep_connected" || echo "Nodo uniéndose al cluster..."
```
