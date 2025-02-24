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

## Servicio de Correo

### Introducción
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
