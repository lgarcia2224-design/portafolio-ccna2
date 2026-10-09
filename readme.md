flowchart TB
    R1["R1-Core<br/>G0/0/0<br/>Subinterfaces .10 .20 .30 .99 .111"]
    CORE["SW-Core<br/>Núcleo de red"]
    MGMT["PC-Gestión<br/>Fa0/9 · VLAN 99"]

    LAB1["SW-Lab1<br/>Edificio A"]
    LAB2["SW-Lab2<br/>Edificio B"]

    R1 <-->|"Trunk 802.1Q"| CORE
    CORE <-->|"Trunk VLAN 10,20,30,99"| LAB1
    CORE <-->|"Trunk VLAN 10,20,30,99"| LAB2
    CORE --- MGMT

    subgraph A["Edificio A"]
        LAB1 --- A10["3 PCs · VLAN 10"]
        LAB1 --- A20["3 PCs · VLAN 20"]
        LAB1 --- A30["1 PC · VLAN 30"]
    end

    subgraph B["Edificio B"]
        LAB2 --- B10["3 PCs · VLAN 10"]
        LAB2 --- B20["3 PCs · VLAN 20"]
        LAB2 --- B30["1 PC · VLAN 30"]
    end

    classDef router fill:#1d4ed8,color:#fff,stroke:#1e40af
    classDef core fill:#0f766e,color:#fff,stroke:#115e59
    classDef access fill:#dbeafe,color:#172554,stroke:#2563eb
    classDef endpoint fill:#f3f4f6,color:#111827,stroke:#9ca3af

    class R1 router
    class CORE core
    class LAB1,LAB2 access
    class MGMT,A10,A20,A30,B10,B20,B30 endpoint

























# 🛠️ GUÍA TÉCNICA DE CONFIGURACIÓN DE REDES: VLANs Y INTER-VLAN
<img width="1148" height="472" alt="Captura de pantalla 2026-10-05 083352" src="https://github.com/user-attachments/assets/145aed56-819c-45ed-bb37-cfd536dbcb41" />

Este documento contiene la sintaxis exacta de Cisco IOS para configurar una red segmentada con tres VLANs y enrutamiento a través de un esquema **Router-on-a-Stick**.

---

## 📊 1. Resumen del Direccionamiento IP y Segmentación

A continuación se detallan las subredes y las funciones asignadas a cada una de las redes virtuales:

* **VLAN 10:** Datos de usuarios en la red local. (`LAN10`)
* **VLAN 20:** Datos de usuarios secundarios. (`LAN20`)
* **VLAN 99:** Red de administración interna. (`Management`)

### 📌 Tabla de Asignación de Interfaces y Puertas de Enlace

| Elemento de Red | ID / Interfaz | Dirección IP | Máscara de Subred | Función Principal |
| :--- | :--- | :--- | :--- | :--- |
| **VLAN 10** | Subinterfaz Gi0/0.10 | 192.168.10.1 | 255.255.255.0 | Gateway LAN 10 |
| **VLAN 20** | Subinterfaz Gi0/0.20 | 192.168.20.1 | 255.255.255.0 | Gateway LAN 20 |
| **VLAN 99** | Subinterfaz Gi0/0.99 | 192.168.99.1 | 255.255.255.0 | Gateway Administración |
| **Switch SVI** | Interfaz Virtual VLAN 99 | 192.168.99.2 | 255.255.255.0 | IP de Gestión Remota |

---

## 🎛️ 2. Configuración General en el Switch

> ⚠️ **Importante:** Asegúrate de ejecutar el comando `enable` y `configure terminal` antes de ingresar las siguientes instrucciones en la consola.

### 🔹 Bloque A: Creación y nombramiento de VLANs
```ios
vlan 10
 name LAN10
 exit

vlan 20
 name LAN20
 exit

vlan 99
 name Management
 exit
```

### 🔹 Bloque B: Asignación de IP de Gestión y Puerta de Enlace (SVI)
```ios
interface vlan 99
 ip add 192.168.99.2 255.255.255.0
 no shut
 exit

ip default-gateway 192.168.99.1
```

### 🔹 Bloque C: Modos de Puertos (Acceso y Troncales)
```ios
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
```

---

## 🔀 3. Configuración del Enrutamiento en el Router (Router-on-a-Stick)

Para habilitar la comunicación entre diferentes VLANs, dividimos la interfaz física `gigabitEthernet0/0` en subinterfaces lógicas y aplicamos la encapsulación `dot1Q`.

```ios
enable
configure terminal

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

end
```

---

## 💾 4. Almacenamiento en Memoria No Volátil

Una vez verificada la topología, debes guardar los cambios en la **NVRAM** para prevenir pérdidas tras un reinicio físico del hardware:

```ios
write memory
```

---

## 📋 5. Tareas Pendientes para el Administrador de Red

- [ ] Verificar la tabla de VLANs con el comando `show vlan brief`.
- [ ] Validar el estado de los enlaces troncales usando `show interfaces trunk`.
- [ ] Hacer pruebas de conectividad (*ping*) desde un equipo de la VLAN 10 hacia la VLAN 20.
