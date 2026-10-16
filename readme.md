# Topología de Red: Router-on-a-Stick con VLANs

## 1. Diagrama de la topología

```text
                 [ R1-Core-LuisMGL (Router) ]
                          |
                          | G0/0/0
                          | Trunk 802.1Q
                          | Router-on-a-Stick
                          | VLANs 10, 20, 30, 99
                          | VLAN 111 nativa
                          |
                     [ SW-Core-LuisMGL ]
                       /       \
              (Trunk) /         \ (Trunk)
          VLAN 111 nativa     VLAN 111 nativa
                    /             \
             [ SW-Lab1-LuisMGL ]       [ SW-Lab2-LuisMGL ]
             Edificio A        Edificio B

     Dispositivos en SW-Lab1-LuisMGL:       Dispositivos en SW-Lab2-LuisMGL:

     ├── VLAN 10: 3 PCs Admin       ├── VLAN 10: 3 PCs Admin
     │   Fa0/1 - Fa0/3              │   Fa0/1 - Fa0/3
     │                              │
     ├── VLAN 20: 3 PCs Alumnos     ├── VLAN 20: 3 PCs Alumnos
     │   Fa0/4 - Fa0/6              │   Fa0/4 - Fa0/6
     │                              │
     └── VLAN 30: 1 PC Dirección   └── VLAN 30: 1 PC Dirección
         Fa0/7                          Fa0/7


     Conexión local de SW-Core-LuisMGL:

     └── VLAN 99: 1 PC Gestión
         Puerto Fa0/9
```

## 2. Diagrama gráfico con Mermaid

```mermaid
flowchart TB
    R1["R1-Core-LuisMGL<br/>Router Central<br/>G0/0/0<br/>Router-on-a-Stick"]

    CORE["SW-Core-LuisMGL<br/>Switch Núcleo"]

    PCG["PC Gestión<br/>VLAN 99<br/>Fa0/9"]

    LAB1["SW-Lab1-LuisMGL<br/>Edificio A"]
    LAB2["SW-Lab2-LuisMGL<br/>Edificio B"]

    R1 <-->|"Trunk 802.1Q<br/>VLAN 10, 20, 30, 99<br/>VLAN 111 nativa"| CORE

    CORE <-->|"Trunk<br/>VLAN 10, 20, 30<br/>VLAN 111 nativa"| LAB1
    CORE <-->|"Trunk<br/>VLAN 10, 20, 30<br/>VLAN 111 nativa"| LAB2

    CORE --- PCG

    subgraph A["Edificio A - SW-Lab1"]
        A10["VLAN 10<br/>3 PCs Administrativos<br/>Fa0/1 - Fa0/3"]
        A20["VLAN 20<br/>3 PCs Alumnos<br/>Fa0/4 - Fa0/6"]
        A30["VLAN 30<br/>1 PC Dirección<br/>Fa0/7"]

        LAB1 --- A10
        LAB1 --- A20
        LAB1 --- A30
    end

    subgraph B["Edificio B - SW-Lab2"]
        B10["VLAN 10<br/>3 PCs Administrativos<br/>Fa0/1 - Fa0/3"]
        B20["VLAN 20<br/>3 PCs Alumnos<br/>Fa0/4 - Fa0/6"]
        B30["VLAN 30<br/>1 PC Dirección<br/>Fa0/7"]

        LAB2 --- B10
        LAB2 --- B20
        LAB2 --- B30
    end

    classDef router fill:#2563eb,color:#ffffff,stroke:#1e40af
    classDef core fill:#0f766e,color:#ffffff,stroke:#115e59
    classDef access fill:#dbeafe,color:#1e3a8a,stroke:#2563eb
    classDef pc fill:#f3f4f6,color:#111827,stroke:#6b7280

    class R1 router
    class CORE core
    class LAB1,LAB2 access
    class PCG,A10,A20,A30,B10,B20,B30 pc
```

## 3. Tabla de direccionamiento

Todas las subredes indicadas para las VLAN 10, 20, 30 y 99 usan máscara
`/24` (`255.255.255.0`). En cada subred, la primera dirección utilizable
es `.1` y la última es `.254`.

### 3.1 Tabla A. Infraestructura

  ### 3.1 Tabla de Direccionamiento de Segmentos / VLANs

| Segmento | VLAN | Red/CIDR | Máscara | Primera IP utilizable | Última IP utilizable | Gateway |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| Administración | 10 | `192.168.10.0/24` | `255.255.255.0` | `192.168.10.1` | `192.168.10.254` | `192.168.10.254` |
| Laboratorios | 20 | `192.168.20.0/24` | `255.255.255.0` | `192.168.20.1` | `192.168.20.254` | `192.168.20.254` |
| Dirección | 30 | `192.168.30.0/24` | `255.255.255.0` | `192.168.30.1` | `192.168.30.254` | `192.168.30.254` |
| Gestión | 99 | `192.168.99.0/24` | `255.255.255.0` | `192.168.99.1` | `192.168.99.254` | `192.168.99.254` |
| VLAN nativa | 111 | No especificada en el documento | No especificada | No asignada | No asignada | No asignado |
| Blackhole / puertos inactivos | 999 | No se especifica una subred | No especificada | No asignada | No asignada | No asignado |

*Las puertas de enlace de las VLAN 10, 20, 30 y 99 se configuran con la última IP utilizable de cada subred en las subinterfaces del router.*

---

### 3.2 Direcciones SVI de los switches

| Dispositivo | Interfaz VLAN | Dirección IP/CIDR | Gateway predeterminado |
| :--- | :---: | :---: | :---: |
| `SW-Core-LuisMGL` | `Vlan99` | `192.168.99.1/24` | `192.168.99.254` |
| `SW-Lab1-LuisMGL` | `Vlan99` | `192.168.99.2/24` | `192.168.99.254` |
| `SW-Lab2-LuisMGL` | `Vlan99` | `192.168.99.3/24` | `192.168.99.254` |

---

### 3.4 Tabla B. Hosts

*Se asignan direcciones estáticas consecutivas dentro de cada VLAN. La PC de gestión usa `192.168.99.4`, porque las direcciones `.1`, `.2` y `.3` se reservan para las SVI de los tres switches.*

| Dispositivo | VLAN | Dirección IP/CIDR | Máscara | Gateway predeterminado |
| :--- | :---: | :---: | :---: | :---: |
| PC-Admin-Lab1-1 | 10 | `192.168.10.1/24` | `255.255.255.0` | `192.168.10.254` |
| PC-Admin-Lab1-2 | 10 | `192.168.10.2/24` | `255.255.255.0` | `192.168.10.254` |
| PC-Admin-Lab1-3 | 10 | `192.168.10.3/24` | `255.255.255.0` | `192.168.10.254` |
| PC-Admin-Lab2-1 | 10 | `192.168.10.4/24` | `255.255.255.0` | `192.168.10.254` |
| PC-Admin-Lab2-2 | 10 | `192.168.10.5/24` | `255.255.255.0` | `192.168.10.254` |
| PC-Admin-Lab2-3 | 10 | `192.168.10.6/24` | `255.255.255.0` | `192.168.10.254` |
| PC-Alumno-Lab1-1 | 20 | `192.168.20.1/24` | `255.255.255.0` | `192.168.20.254` |
| PC-Alumno-Lab1-2 | 20 | `192.168.20.2/24` | `255.255.255.0` | `192.168.20.254` |
| PC-Alumno-Lab1-3 | 20 | `192.168.20.3/24` | `255.255.255.0` | `192.168.20.254` |
| PC-Alumno-Lab2-1 | 20 | `192.168.20.4/24` | `255.255.255.0` | `192.168.20.254` |
| PC-Alumno-Lab2-2 | 20 | `192.168.20.5/24` | `255.255.255.0` | `192.168.20.254` |
| PC-Alumno-Lab2-3 | 20 | `192.168.20.6/24` | `255.255.255.0` | `192.168.20.254` |
| PC-Direccion-Lab1 | 30 | `192.168.30.1/24` | `255.255.255.0` | `192.168.30.254` |
| PC-Direccion-Lab2 | 30 | `192.168.30.2/24` | `255.255.255.0` | `192.168.30.254` |
| PC-Gestion | 99 | `192.168.99.4/24` | `255.255.255.0` | `192.168.99.254` |
> En cada PC: **Desktop → IP Configuration → Static**. Introduce la
> dirección IP, la máscara y el gateway indicados en la tabla.

------------------------------------------------------------------------

## 4. Asignar nombre a los dispositivos con las iniciales LuisMGL

Ejecuta en cada dispositivo, en modo privilegiado y luego en
configuración global:

### Router --- `R1-Core-LuisMGL`
``` text
Router>enable
Router#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#hostname R1-Core-LuisMGL
R1-Core-LuisMGL(config)#
```

### Switch central --- `SW-Core-LuisMGL`
``` text
Switch>enable
Switch#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)#hostname SW-Core-LuisMGL
SW-Core-LuisMGL(config)#
```

### Switch de acceso 1 --- `SW-Lab1-LuisMGL`

``` text
Switch>enable
Switch#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)#hostname SW-Lab1-LuisMGL
SW-Lab1-LuisMGL(config)#
```
### Switch de acceso 2 --- `SW-Lab2-LuisMGL`
``` text
Switch>enable
Switch#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)#hostname SW-Lab2-LuisMGL
SW-Lab2-LuisMGL(config)#

```
## 5. Configuración básica de seguridad en router y switches

El documento solicita `enable secret`, cifrado de contraseñas de texto
plano y `banner motd`. Usa una contraseña de práctica propia y
consistente en los cuatro dispositivos;

Ejecuta en **router y switchS**, después de establecer el
`hostname`:

```bash
enable
config
enable secret MODULO_PREFERIDO_MANEJOTE
service password-encryption
banner motd #ACCESO RESTRINGIDO: solo personal autorizado#
```

Para configurar una contraseña en la consola:

``` text
line console 0
password MODULO_PREFERIDO_MANEJOTE
login
exit
```
Para la línea VTY, destinada al acceso remoto por terminal:
``` text
line vty 0 4
password <CLAVE_VTY>
login
exit
```

Guarda la configuración al terminar el equipo:
``` text
end
copy running-config startup-config
```

Cuando Packet Tracer pregunte por el nombre del archivo de destino,
pulsa **Enter** para aceptar el valor predeterminado.

------------------------------------------------------------------------
## 5. Crear las VLAN 10, 20, 30, 99, 111 y 999

Crea las VLAN en **los tres switches** (`SW-Core-LuisMGL`, `SW-Lab1-LuisMGL` y
`SW-Lab2-LuisMGL`).

``` text
configure terminal
vlan 10
 name Administracion
exit
vlan 20
 name Laboratorios
exit
vlan 30
 name Direccion
exit
vlan 99
 name Gestion
exit
vlan 111
 name Nativa
exit
vlan 999
 name Blackhole
exit
end
```

Verifica la creación:

``` text
show vlan brief
```

Comprueba que aparezcan las seis VLAN. La VLAN 999 se usará para los
puertos que no estén en uso; no asignes a ella los puertos conectados a
PCs ni los enlaces troncales.

------------------------------------------------------------------------

## 6. Asignar los puertos a las VLAN

### 6.1 SW-Lab1-LuisMGL

IRAN A:

-   `Fa0/1` a `Fa0/3`: Administración, VLAN 10.
-   `Fa0/4` a `Fa0/6`: Laboratorios/Alumnos, VLAN 20.
-   `Fa0/7`: Dirección, VLAN 30.

``` text
SW-Lab1-LuisMGL#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
SW-Lab1-LuisMGL(config)#interface range fastEthernet 0/1-3
SW-Lab1-LuisMGL(config-if-range)# switchport mode access
SW-Lab1-LuisMGL(config-if-range)# switchport access vlan 10
SW-Lab1-LuisMGL(config-if-range)#exit
SW-Lab1-LuisMGL(config)#interface range fastEthernet 0/4-6
SW-Lab1-LuisMGL(config-if-range)# switchport mode access
SW-Lab1-LuisMGL(config-if-range)# switchport access vlan 20
SW-Lab1-LuisMGL(config-if-range)#exit
SW-Lab1-LuisMGL(config)#interface fastEthernet 0/7
SW-Lab1-LuisMGL(config-if)# switchport mode access
SW-Lab1-LuisMGL(config-if)# switchport access vlan 30
SW-Lab1-LuisMGL(config-if)#exit
SW-Lab1-LuisMGL(config)#end
```

### 6.2 SW-Lab2-LuisMGL

``` text
SW-Lab2-LuisMGL#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
SW-Lab2-LuisMGL(config)#interface range fastEthernet 0/1-3
SW-Lab2-LuisMGL(config-if-range)# switchport mode access
SW-Lab2-LuisMGL(config-if-range)# switchport access vlan 10
SW-Lab2-LuisMGL(config-if-range)#exit
SW-Lab2-LuisMGL(config)#interface range fastEthernet 0/4-6
SW-Lab2-LuisMGL(config-if-range)# switchport mode access
SW-Lab2-LuisMGL(config-if-range)# switchport access vlan 20
SW-Lab2-LuisMGL(config-if-range)#exit
SW-Lab2-LuisMGL(config)#interface fastEthernet 0/7
SW-Lab2-LuisMGL(config-if)# switchport mode access
SW-Lab2-LuisMGL(config-if)# switchport access vlan 30
SW-Lab2-LuisMGL(config-if)#exit
SW-Lab2-LuisMGL(config)#end
SW-Lab2-LuisMGL#
%SYS-5-CONFIG_I: Configured from console by console
```

### 6.3 SW-Core-LuisMGL: PC de gestión

Coloca la PC de gestión en `Fa0/9`, VLAN 99.

``` text
SW-Core-LuisMGL#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
SW-Core-LuisMGL(config)#interface fastEthernet 0/9
SW-Core-LuisMGL(config-if)# switchport mode access
SW-Core-LuisMGL(config-if)# switchport access vlan 99
SW-Core-LuisMGL(config-if)# no shutdown
SW-Core-LuisMGL(config-if)#exit
SW-Core-LuisMGL(config)#end
```

**No asignes otros puertos de acceso del SW-Core a una VLAN de usuario
sin que corresponda a un dispositivo del diagrama.** Los puertos del
núcleo que conectan con el router y con los switches de laboratorio se
configurarán como troncales en el siguiente apartado.

### 6.4 Verificar la pertenencia a VLAN

En cada switch:

``` bash
show vlan brief
```

Comprueba que `Fa0/1-Fa0/3`, `Fa0/4-Fa0/6` y `Fa0/7` estén en las VLAN
indicadas en cada switch de laboratorio, y que `SW-Core-luismgl Fa0/9` esté
en VLAN 99.

------------------------------------------------------------------------
## 7. Habilitar los enlaces troncales

La VLAN nativa requerida es la **VLAN 111**. Configura los puertos que
conectan switch con switch y el puerto del SW-Core que conecta al router
como troncales, permitiendo las VLAN 10, 20, 30, 99 y 111.

### 7.1 SW-Core-LuisMGL

``` text
SW-Core-LuisMGL(config)#interface Gig0/1
SW-Core-LuisMGL(config-if)#switchport mode trunk
SW-Core-LuisMGL(config-if)# switchport trunk native vlan 111
SW-Core-LuisMGL(config-if)# switchport trunk allowed vlan 10,20,30,99,111
SW-Core-LuisMGL(config-if)# no shutdown
SW-Core-LuisMGL(config-if)#exit
SW-Core-LuisMGL(config)#interface f0/24
SW-Core-LuisMGL(config-if)# switchport mode trunk

SW-Core-LuisMGL(config-if)# switchport trunk native vlan 111
SW-Core-LuisMGL(config-if)# switchport trunk allowed vlan 10,20,30,99,111
SW-Core-LuisMGL(config-if)# no shutdown
SW-Core-LuisMGL(config-if)#exit
SW-Core-LuisMGL(config)#interface f0/23
SW-Core-LuisMGL(config-if)# switchport mode trunk

SW-Core-LuisMGL(config-if)# switchport trunk native vlan 111
SW-Core-LuisMGL(config-if)# switchport trunk allowed vlan 10,20,30,99,111
SW-Core-LuisMGL(config-if)# no shutdown
SW-Core-LuisMGL(config-if)#exit
SW-Core-LuisMGL(config)#end
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/24, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/24, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/23, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/23, changed state to up

SW-Core-LuisMGL#
%SYS-5-CONFIG_I: Configured from console by console
```

Ejemplo de sustitución: si el cable hacia SW-Lab1 está conectado a
`GigabitEthernet0/1`, escribe `interface gigabitEthernet 0/1` en lugar
de `interface <INTERFAZ_LAB1>`. **No copies literalmente los marcadores
`<...>` en la CLI.**

### 7.2 SW-Lab1-LuisMGL

Por el puerto que conecta al SW-Core.

``` text
SW-Lab1-LuisMGL#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
SW-Lab1-LuisMGL(config)#interface fa0/24
SW-Lab1-LuisMGL(config-if)# switchport mode trunk
SW-Lab1-LuisMGL(config-if)# switchport trunk native vlan 111
SW-Lab1-LuisMGL(config-if)# switchport trunk allowed vlan 10,20,30,99,111
SW-Lab1-LuisMGL(config-if)# no shutdown
SW-Lab1-LuisMGL(config-if)#exit
SW-Lab1-LuisMGL(config)#end
SW-Lab1-LuisMGL#
%SYS-5-CONFIG_I: Configured from console by console
```

### 7.3 SW-Lab2-LuisMGL
Por el puerto que conecta al SW-Core.

``` text
SW-Lab2-LuisMGL#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
SW-Lab2-LuisMGL(config)#interface fa0/23
SW-Lab2-LuisMGL(config-if)# switchport mode trunk
SW-Lab2-LuisMGL(config-if)# switchport trunk native vlan 111
SW-Lab2-LuisMGL(config-if)# switchport trunk allowed vlan 10,20,30,99,111
SW-Lab2-LuisMGL(config-if)# no shutdown
SW-Lab2-LuisMGL(config-if)#exit
SW-Lab2-LuisMGL(config)#end
SW-Lab2-LuisMGL#
%SYS-5-CONFIG_I: Configured from console by console

```

> Si el modelo de switch seleccionado no admite alguno de los comandos
> `switchport` mostrados, verifica el modelo y las capacidades de la
> interfaz en Packet Tracer. No cambies la topología para ocultar el
> error.

### 7.4 Verificar los troncales
En cada switch:

``` text
show interfaces trunk
show interfaces <PUERTO_TRONCAL> switchport
```

Comprueba que los enlaces aparezcan como troncales, que la VLAN nativa
sea 111 y que la lista permitida incluya `10,20,30,99,111`.

------------------------------------------------------------------------


## 5. Consideraciones importantes
1. Crear las VLANs necesarias en los switches.
2. Configurar los enlaces troncales con la VLAN nativa 111 en ambos extremos.
3. Configurar las subinterfaces del router con `encapsulation dot1Q`.
4. Configurar la subinterfaz de la VLAN 111 con `encapsulation dot1Q 111 native` si el router también utilizará esa VLAN para enrutamiento.
5. Asignar las direcciones IP de gateway y las direcciones de los dispositivos.
6. Configurar el puerto Fa0/9 de SW-Core como acceso a la VLAN 99.

**Nota:** La VLAN nativa 111 y la VLAN 99 de gestión cumplen funciones distintas. La VLAN 111 transporta tráfico sin etiquetar en los trunks, mientras que la VLAN 99 se utiliza para la red de gestión.****






