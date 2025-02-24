# Correo-e-Isabela-con-Dominio
Este repositorio es para documentar las configuraciones y los comandos que se usaron para levantar un servicio de Correo usando Postfix, Dovecot y Thunderbird+Gmail, el cual funcionará con un Dominio propio sustentando por un servidor DNS; además de levantar un servicio de VoIP usando Isabela, el cual funcionará también con un Dominio.
# Requerimientos Previos para la realización del trabajo.
Para la realización de este trabajo se necesitó crear una máquina virtual de Centos Stream 9 en el software de virtualización VirtualBox; las características de la máquina virtual creada son contar con interfaz gráfica, con 8 GB de RAM y con 50 GB de almacenamiento.

El orden en el que se fueron levantando los servicios fue en el que el Ing. Jonnathan Nivicela nos indicó, primero se levantó el servicio DNS, siendo clave para lograr el funcionamiento de los servicios de Correo y VoIP con el Dominio Local propio, después se levantó el servicio de Correo usando los servidores de Postfix, Dovecot y Thunderbird+Gmail, asociándolo con el servicio de DNS para lograr hacerlo funcionar correctamente con el dominio propio, y finalmente se levantó el servicio de VoIP usando el servidor de Isabela, el cual fue creado como una máquina virtual nativa, y cuando la instalación básica fue hecha, se asoció el servidor de Isabela con el de DNS para que funcione correctamente con el dominio propio.

# Servicio de DNS
Para el levantamiento del servicio de DNS se usó un servidor BIND9, el cual nos permitió crear un dominio propio, en este caso el dominio “sistemas.nida.es”, el cual fue asociado con la IP del servidor, haciendo que podamos conectarnos a la máquina virtual, que es el Servidor en un todo, a través del dominio, podamos hacer ping al Servidor a través del dominio, o como sera despues de haber levantado los servicios de Correo y VoIP, podamos acceder tanto a las páginas web de los servidores de Correo y VoIP como que los clientes puedan usar el dominio para poder hacer uso de los servicios que el Servidor está ofreciendo.

Los pasos, los comandos y las configuraciones que se hicieron para el levantamiento del servicio de DNS son los siguientes:
1. Actualizar el Sistema de Centos Stream 9:
    ```bash
    sudo dnf update
    ```

2. Instalar todos los paquetes necesarios para levantar el servidor BIND9:
    ```bash
    sudo dnf install bind bind-utils
    ```

3. Iniciar, habilitar inicio automático, reiniciar y ver el estado del servidor BIND9:
    ```bash
    sudo systemctl start named
    sudo systemctl enable named
    sudo systemctl restart named
    sudo systemctl status named
    ```
![](https://github.com/Nicolas646148/Correo-e-Isabela-con-Dominio/blob/main/Captura%20de%20pantalla%202025-02-24%20141249.png)

4. Configurar el firewall de la máquina virtual para permitir el tráfico del servidor DNS:
    ```bash
    sudo firewall-cmd --add-service=dns --permanent
    sudo firewall-cmd --reload
    ```

5. Editar el archivo /etc/sysconfig/network-scripts/ifcfg-enp0s3, y el contenido que debería tener:
- Comando para editar el archivo /etc/sysconfig/network-scripts/ifcfg-enp0s3:
    ```bash
    sudo nano /etc/sysconfig/network-scripts/ifcfg-enp0s3
    ```
- Contenido que debería tener el archivo /etc/sysconfig/network-scripts/ifcfg-enp0s3:
    ```plaintext
    TYPE=Ethernet
    BOOTPROTO=none
    NAME=enp0s3
    DEVICE=enp0s3
    ONBOOT=yes
    IPADDR=192.168.18.83
    PREFIX=24
    GATEWAY=192.168.18.1
    DNS1=192.168.18.83
    DNS2=8.8.8.8
    DOMAIN=sistemasnida.es
    ```
- Después de haber editado el editado el archivo /etc/sysconfig/network-scripts/ifcfg-enp0s3 ejecutar el siguiente comando:
    ```bash
    sudo systemctl restart NetworkManager
    ```

6. Editar el archivo /etc/NetworkManager/NetworkManager.conf, y el contenido que debería tener:
- Comando para editar el archivo /etc/NetworkManager/NetworkManager.conf:
    ```bash
    sudo nano /etc/NetworkManager/NetworkManager.conf
    ```
- Contenido que debería tener el archivo /etc/NetworkManager/NetworkManager.conf:
    ```plaintext
    [main]
    dns=none
    ```
- Después de haber editado el editado el archivo /etc/NetworkManager/NetworkManager.conf ejecutar el siguiente comando:
    ```bash
    sudo systemctl restart NetworkManager
    ```
    
7. Editar el archivo /etc/resolv.conf, y el contenido que debería tener:
- Comando para editar el archivo /etc/resolv.conf:
    ```bash
    sudo nano /etc/resolv.conf
    ```
- Contenido que debería tener el archivo /etc/resolv.conf:
    ```plaintext
    # Generated by NetworkManager
    nameserver 192.168.18.83
    nameserver 8.8.8.8
    search sistemasnida.es
    ```
- Después de haber editado el editado el archivo /etc/resolv.conf ejecutar el siguiente comando:
    ```bash
    sudo systemctl restart NetworkManager
    ```

8. Editar el archivo /etc/nsswitch.conf, y el contenido que debería tener:
- Comando para editar el archivo /etc/nsswitch.conf:
    ```bash
    sudo nano /etc/nsswitch.conf
    ```
- Contenido que debería tener el archivo /etc/nsswitch.conf:
    ```plaintext
    passwd:     files sss systemd
    group:      files [SUCCESS=merge] sss [SUCCESS=merge] systemd
    shadow:     files sss
    gshadow:    files
    netgroup:   sss files
    automount:  sss files
    services:   sss files

    hosts:      dns files mdns4_minimal myhostname [NOTFOUND=return]
    networks:   files dns

    protocols:  files
    publickey:  files
    aliases:    files
    ethers:     files
    rpc:        files
    ```
- Después de haber editado el editado el archivo /etc/nsswitch.conf ejecutar el siguiente comando:
    ```bash
    sudo systemctl restart NetworkManager
    ```

9. Editar el archivo /etc/named.conf, y el contenido que debería tener:
- Comando para editar el archivo /etc/named.conf:
    ```bash
    sudo nano /etc/named.conf
    ```
- Contenido que debería tener el archivo /etc/named.conf:
    ```plaintext
    options {
    listen-on port 53 { any; };
    listen-on-v6 port 53 { ::1; };
    directory       "/var/named";
    dump-file       "/var/named/data/cache_dump.db";
    statistics-file "/var/named/data/named_stats.txt";
    memstatistics-file "/var/named/data/named_mem_stats.txt";
    allow-query     { any; };
    recursion yes;
    };

    logging {
    channel default_debug {
        file "data/named.run";
        severity dynamic;
    };
    };

    zone "." IN {
    type hint;
    file "named.ca";
    };

    include "/etc/named.rfc1912.zones";
    include "/etc/named.root.key";

    zone "sistemasnida.es" {
    type master;
    file "/var/named/db.sistemasnida.es";
    };

    zone "18.168.192.in-addr.arpa" {
    type master;
    file "/var/named/db.18.168.192";
    };
    ```
- Después de haber editado el editado el archivo /etc/named.conf ejecutar el siguiente comando:
    ```bash
    sudo systemctl restart named
    ```

10. Crear y editar el archivo /var/named/db.sistemasnida.es, y el contenido que debería tener:
- Comando para crear y editar el archivo /var/named/db.sistemasnida.es:
    ```bash
    sudo nano /var/named/db.sistemasnida.es
    ```
- Contenido que debería tener el archivo /var/named/db.sistemasnida.es:
    ```plaintext
    $TTL 86400
    @   IN  SOA     ns1.sistemasnida.es. root.sistemasnida.es. (
                2025022301 ; Serial
                3600       ; Refresh
                1800       ; Retry
                1209600    ; Expire
                86400 )    ; Minimum TTL
    ;
    @       IN      NS      ns1.sistemasnida.es.
    ns1     IN      A       192.168.18.83
    @       IN      A       192.168.18.83
    pc1     IN      A       192.168.18.10
    pc2     IN      A       192.168.18.20
    server  IN      CNAME   ns1
    ```
- Después de haber editado el editado el archivo /var/named/db.sistemasnida.es ejecutar el siguiente comando:
    ```bash
    sudo systemctl restart named
    ```

11. Crear y editar el archivo /var/named/db.18.168.192, y el contenido que debería tener:
- Comando para crear y editar el archivo /var/named/db.18.168.192:
    ```bash
    sudo nano /var/named/db.18.168.192
    ```
- Contenido que debería tener el archivo /var/named/db.18.168.192:
    ```plaintext
    $TTL 86400
    @   IN  SOA     ns1.sistemasnida.es. root.sistemasnida.es. (
                2025022301 ; Serial
                3600       ; Refresh
                1800       ; Retry
                1209600    ; Expire
                86400 )    ; Minimum TTL
    ;
    @       IN      NS      ns1.sistemasnida.es.
    83      IN      PTR     sistemasnida.es.
    10      IN      PTR     pc1.sistemasnida.es.
    20      IN      PTR     pc2.sistemasnida.es.
    ```
- Después de haber editado el editado el archivo /var/named/db.18.168.192 ejecutar el siguiente comando:
    ```bash
    sudo systemctl restart named
    ```

Después de haber hecho todas estas configuraciones nuestro servicio de DNS estará funcionando correctamente, teniendo varios dominios que podemos usar. Para probar la funcionalidad del servidor BIND9 haremos las siguientes pruebas de funcionalidad:
- Ver el estado del servidor BIND9:

![](https://github.com/Nicolas646148/Correo-e-Isabela-con-Dominio/blob/main/Captura%20de%20pantalla%202025-02-24%20151340.png)
- Hacer ding al dominio creado:

![](https://github.com/Nicolas646148/Correo-e-Isabela-con-Dominio/blob/main/Captura%20de%20pantalla%202025-02-24%20151414.png)
- Hacer ping al dominio creado:

![](https://github.com/Nicolas646148/Correo-e-Isabela-con-Dominio/blob/main/Captura%20de%20pantalla%202025-02-24%20151441.png)
- Hacer ping a la IP del servidor:

![](https://github.com/Nicolas646148/Correo-e-Isabela-con-Dominio/blob/main/Captura%20de%20pantalla%202025-02-24%20151507.png)
- Hacer host al dominio creado:

![](https://github.com/Nicolas646148/Correo-e-Isabela-con-Dominio/blob/main/Captura%20de%20pantalla%202025-02-24%20151553.png)
- Hacer nslookup al dominio creado:

![](https://github.com/Nicolas646148/Correo-e-Isabela-con-Dominio/blob/main/Captura%20de%20pantalla%202025-02-24%20151609.png)

## Servicio de Correo

En esta sección se documenta el proceso seguido para levantar un servicio de correo utilizando Postfix, Dovecot y Thunderbird+Gmail, funcionando con un dominio propio y sustentado por un servidor DNS.

### Instalación de Postfix
1. Actualizar los paquetes del sistema:
    ```bash
    sudo apt update
    sudo apt upgrade
    ```

2. Instalar Postfix:
    ```bash
    sudo apt install postfix
    ```

3. Configurar Postfix:
    - Durante la instalación, seleccionar "Internet Site".
    - Configurar el nombre del sistema de correo (ej. mail.tudominio.com).

4. Editar el archivo de configuración de Postfix:
    ```bash
    sudo nano /etc/postfix/main.cf
    ```
    - Añadir o modificar las siguientes líneas:
        ```plaintext
        myhostname = mail.tudominio.com
        mydomain = tudominio.com
        myorigin = /etc/mailname
        mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
        relayhost =
        ```

5. Reiniciar Postfix:
    ```bash
    sudo systemctl restart postfix
    ```

### Instalación de Dovecot
1. Instalar Dovecot:
    ```bash
    sudo apt install dovecot-core dovecot-imapd dovecot-pop3d
    ```

2. Configurar Dovecot:
    - Editar el archivo de configuración principal:
        ```bash
        sudo nano /etc/dovecot/dovecot.conf
        ```
        - Asegurarse de que las siguientes líneas estén presentes y descomentadas:
            ```plaintext
            protocols = imap pop3 lmtp
            ```

3. Configurar el directorio de correo:
    - Editar el archivo de configuración de directorios de correo:
        ```bash
        sudo nano /etc/dovecot/conf.d/10-mail.conf
        ```
        - Modificar la línea `mail_location`:
            ```plaintext
            mail_location = maildir:~/Maildir
            ```

4. Reiniciar Dovecot:
    ```bash
    sudo systemctl restart dovecot
    ```

### Configuración de Thunderbird con Gmail
1. Abrir Thunderbird y añadir una nueva cuenta de correo.
2. Introducir los datos de tu cuenta de dominio propio.
3. Configurar los servidores IMAP y SMTP de acuerdo a la configuración de Postfix y Dovecot.

## Servicio de VoIP

### Introducción
En esta sección se documenta el proceso seguido para levantar un servicio de VoIP utilizando Isabela, funcionando con un dominio propio.

### Requisitos
- Servidor con sistema operativo Linux (ej. Ubuntu, CentOS)
- Acceso a internet
- Dominio propio
- Servidor DNS configurado

### Instalación de Isabela
1. Actualizar los paquetes del sistema:
    ```bash
    sudo apt update
    sudo apt upgrade
    ```

2. Instalar Isabela:
    ```bash
    sudo apt install isabela
    ```

3. Configurar Isabela:
    - Editar el archivo de configuración principal:
        ```bash
        sudo nano /etc/isabela/isabela.conf
        ```
        - Modificar las líneas necesarias para que apunten a tu dominio y servidor DNS.

4. Reiniciar Isabela:
    ```bash
    sudo systemctl restart isabela
    ```

### Configuración de Clientes VoIP
1. Descargar e instalar un cliente VoIP compatible (ej. Zoiper, Linphone).
2. Configurar el cliente con los datos proporcionados por Isabela y el dominio propio.

### Pruebas y Verificación
1. Realizar pruebas de envío y recepción de correos para el servicio de correo.
2. Realizar pruebas de llamadas VoIP para el servicio de VoIP.

## Conclusión
En este documento se ha detallado el proceso para levantar un servicio de correo y un servicio de VoIP utilizando Postfix, Dovecot, Thunderbird+Gmail e Isabela. Ambas configuraciones han sido probadas y verificadas con éxito utilizando un dominio propio y un servidor DNS.
