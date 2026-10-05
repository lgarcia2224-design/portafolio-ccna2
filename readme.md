Configuración Global y Creación de VLANs
Estos comandos se utilizan para ingresar al modo de configuración y crear las redes virtuales (VLANs) asignándoles un nombre de identificación.
ios
enable
configure terminal

vlan 10
 name LAN10
 exit

vlan 20
 name LAN20
 exit

vlan 99
 name Management
 exit
Usa el código con precaución.
• enable: Cambia al modo de ejecución privilegiado (permite ver y modificar configuraciones globales).
• configure terminal: Entra al modo de configuración global desde el cual se pueden aplicar cambios al equipo.
• vlan [número]: Crea una interfaz virtual de Capa 2 con el número de ID especificado y entra a su submodo.
• name [nombre]: Asigna un nombre descriptivo a la VLAN para facilitar su administración.
• exit: Sale del submodo actual y regresa al modo de configuración global.
🌐 Configuración de la Interfaz de Administración y Gateway
Permite asignar una dirección IP al switch para poder gestionarlo de forma remota (por ejemplo, vía SSH o Telnet).
ios
interface vlan 99
 ip add 192.168.99.2 255.255.255.0
 no shut
 exit

ip default-gateway 192.168.99.1
Usa el código con precaución.
• interface vlan 99: Entra a la interfaz virtual del switch (SVI) correspondiente a la VLAN 99.
• ip add [dirección_ip] [máscara]: Asigna la dirección IP 192.168.99.2 con máscara de subred 255.255.255.0 (Clase C) a esta interfaz.
• no shut (o no shutdown): Enciende o activa la interfaz (por defecto vienen apagadas administrativamente).
• ip default-gateway 192.168.99.1: Configura la puerta de enlace predeterminada del switch para que este pueda comunicarse con redes fuera de su propia subred.
💻 Configuración de Puertos en el Switch (Acceso y Troncal)
Define el comportamiento de los puertos físicos del switch, ya sea para conectar dispositivos finales (Acceso) o para conectar otros switches/routers (Troncal).
ios
interface fa0/6
 switchport mode access
 switchport access vlan 10
 no shut
 exit

interface fa0/1
 switchport mode trunk
 no shut
 exit

interface fa0/5
 switchport mode trunk
 no shut
 end
Usa el código con precaución.
• interface fa0/6: Selecciona el puerto físico FastEthernet 0/6 para configurarlo.
• switchport mode access: Configura el puerto en modo acceso, lo que significa que solo se conectará un dispositivo final (como una PC).
• switchport access vlan 10: Asocia de forma exclusiva el tráfico de este puerto a la VLAN 10.
• interface fa0/1 / fa0/5: Selecciona los puertos FastEthernet 0/1 y 0/5.
• switchport mode trunk: Configura los puertos en modo troncal (Trunk), permitiendo que el tráfico de múltiples VLANs viaje a través de un solo cable físico hacia otro switch o router.
• end: Finaliza el modo de configuración por completo y regresa al modo privilegiado de manera directa.
🔀 Enrutamiento Inter-VLAN (Router on a Stick)
Esta sección configura un router para comunicar las diferentes VLANs entre sí, dividiendo una interfaz física en subinterfaces lógicas.
ios
interface gigabitEthernet0/0
 no shutdown
 exit

interface gigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 exit

interface gigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 exit

interface gigabitEthernet0/0.99
 encapsulation dot1Q 99
 ip address 192.168.99.1 255.255.255.0
 exit
Usa el código con precaución.
• interface gigabitEthernet0/0: Entra a la interfaz física principal GigabitEthernet 0/0 del router.
• interface gigabitEthernet0/0.[número]: Crea una subinterfaz lógica asignada al número indicado (es buena práctica que coincida con el ID de la VLAN).
• encapsulation dot1Q [VLAN]: Aplica el protocolo de encapsulación estándar de la industria IEEE 802.1Q y le indica a la subinterfaz a qué VLAN debe etiquetar o destetiquetar el tráfico.
• ip address [dirección_ip] [máscara]: Configura la IP de la subinterfaz, la cual servirá como el Gateway (Puerta de enlace) para los dispositivos que pertenezcan a esa VLAN específica.
💾 Almacenamiento de Cambios
Garantiza que la configuración no se borre si el equipo llega a apagarse o reiniciarse.
ios
write memory
Usa el código con precaución.
• write memory (o copy running-config startup-config): Guarda de forma permanente la configuración activa de la memoria RAM en la memoria NVRAM del dispositivo.
