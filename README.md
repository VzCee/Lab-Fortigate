# Documentacion Fortigate

**Documentación Técnica: Implementación de Seguridad Perimetral y Segmentación de Red con FortiGate**

**Fecha:** 12 de Marzo de 2026

**Plataforma:** PNETLab

**Video:**
https://youtu.be/GtF5vRE4KxM

## 1. Topología y Direccionamiento de Red

Se ha implementado una topología de red segmentada utilizando un Firewall FortiGate como núcleo de seguridad para gestionar el tráfico entre la red LAN de usuarios, la red de servidores y la salida a Internet (WAN).

<img width="613" height="464" alt="image" src="https://github.com/user-attachments/assets/13a7de7f-f8b7-45ae-8274-68c377a7b537" />



| **Interfaz** | **Descripción** | **Red / IP** | **Máscara** | **Gateway** |
| --- | --- | --- | --- | --- |
| **Port 1** | WAN (Internet) | 192.168.133.173 | 255.255.255.0 | 192.168.133.x |
| **Port 2** | LAN Usuarios | 23.72.1.1 | 255.255.255.128 (/25) | N/A (Firewall) |
| **Port 3** | LAN Servidores | 23.72.1.129 | 255.255.255.240 (/28) | N/A (Firewall) |

# **2. Configuración de Servicios Base**

**2.1 DHCP en LAN Usuarios**
Para facilitar la administración, se configuró un servidor DHCP en el **Port 2** para asignar IPs automáticamente en el rango `23.72.1.2 - 23.72.1.126`.

<img width="1378" height="279" alt="image" src="https://github.com/user-attachments/assets/a26b0f55-ba9e-495f-8827-70f92ad6927b" />


**2.2 Ruta por Defecto (Static Route)**
Se estableció una ruta estática hacia `0.0.0.0/0.0.0.0` apuntando al Gateway de la nube (Port 1) para permitir la salida a Internet de los usuarios.

<img width="920" height="438" alt="image" src="https://github.com/user-attachments/assets/0347757f-fcad-41c3-bd37-dc360ff2c91d" />


**3. Políticas de Seguridad y Filtrado de Contenido**
Se crearon políticas específicas para cumplir con el principio de **Menor Privilegio**.
**3.1 Política: Internet Usuarios (Salida)**
Permite la navegación general aplicando los siguientes perfiles de seguridad:
• **Web Filter:** Bloqueo de dominios y subdominios de `itla.edu.do`.
• **Application Control:** Bloqueo de Redes Sociales y llamadas de WhatsApp.
• **Inspección:** Flow-based (para optimizar la compatibilidad con servicios de Google/HTTPS).

**3.2 Política: Acceso a Servidores (Interna)**
Restringe el tráfico desde la LAN de usuarios hacia la LAN de servidores:
• **Servicio:** Únicamente **HTTP (Puerto 80)**. Todo lo demás (incluyendo Ping/ICMP) está denegado.
• **Seguridad:** Aplicación de **IPS** y **WAF (Web Application Firewall)**.
• **Inspección:** Proxy-based (necesario para el funcionamiento del WAF).

**4. Verificación de Requisitos (Pruebas de Funcionamiento)
4.1 Bloqueo de Dominio ITLA**
Al intentar acceder a `itla.edu.do`, el motor de **Web Filter** intercepta la petición y muestra la página de reemplazo.

<img width="952" height="780" alt="image" src="https://github.com/user-attachments/assets/bbcefa60-5145-48ad-8582-694b54dcfcb5" />


# Visualizacion del log en fortigate

<img width="1265" height="228" alt="image" src="https://github.com/user-attachments/assets/4cf8f63f-d2f3-4c92-a8ef-00d39d470824" />


**4.2 Bloqueo de Redes Sociales (YouTube/WhatsApp)**
El motor de **Application Control** detecta las firmas de tráfico de aplicaciones prohibidas y las descarta.

<img width="1671" height="497" alt="image" src="https://github.com/user-attachments/assets/1966342e-d9a0-47d0-8714-571570e3e807" />


**4.3 Protección del Servidor (IPS y WAF)**
Se verifica que el tráfico hacia la IP `23.72.1.130` es inspeccionado. Aunque el servidor sea simulado, los logs de **Forward Traffic** confirman que la política permite el tráfico y los perfiles de seguridad se aplican.
• **Resultado del Log:** `TCP reset from server` (Confirma que el Firewall entregó el paquete al destino).

<img width="995" height="592" alt="image" src="https://github.com/user-attachments/assets/d5cff781-d286-4e44-b029-386d2cd02667" />


<img width="1590" height="295" alt="image" src="https://github.com/user-attachments/assets/3d529780-07da-4d1b-9658-f837e28b24d3" />


**4.4 Bloqueo de Escáneres de Red**
Se configuró un sensor de IPS específicamente para detectar categorías de **Scanner.**

<img width="880" height="461" alt="image" src="https://github.com/user-attachments/assets/43053460-8f79-4749-8d6f-582f35b7da70" />

**5. Conclusiones**
La implementación cumple con los estándares de seguridad perimetral al aislar la red de servidores del internet público y restringir el acceso de los usuarios finales únicamente a los servicios necesarios. El uso de **WAF** e **IPS** asegura que el servidor web esté protegido contra ataques de capa 7 (aplicación), mientras que el filtrado de contenido optimiza el uso del ancho de banda y cumple con las políticas institucionales.
