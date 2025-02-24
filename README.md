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

