
Documentación del Laboratorio de Redes Seguras: AAA, RADIUS y Active Directory
1. Propósito del Laboratorio
Implementar y documentar una topología de red segura integrada con un Router Cisco, un Ethernet Switch, y un Windows Server (Active Directory y NPS). El objetivo es aplicar autenticación centralizada RADIUS, protocolos AAA, cifrado SSH y control de acceso basado en roles para cumplir con rigurosos estándares de seguridad perimetral y auditoría de red.

2. Diagrama de la Topología

<img width="283" height="381" alt="imagen_2026-10-30_223040628" src="https://github.com/user-attachments/assets/8282645d-825b-4fcf-9ea9-62185441fc68" />


[ Cloud1 ] <---> [ Ethernet Switch ] <---> [ Router Cisco (g0/0) ]
                                    <---> [ Windows Server (NPS / AD) ]
                                    
3. Plan de Direccionamiento IP y Segmentación
Dispositivo / Sede	Interfaz / Puerto	Zona / Rol	Dirección IP / Máscara
Cloud	Cloud1	Enlace Externo	N/A (Puente hacia Adaptador físico/virtual)
Ethernet Switch	Ethernet 0 - 2	Capa 2 (Bridge)	N/A (Dispositivo no administrable)
Router Cisco (R1)	GigabitEthernet 0/0	Gateway LAN / AAA / SSH	10.7.91.201 / 255.255.255.128
Windows Server	Ethernet 0	Controlador de Dominio / NPS	10.7.91.200 / 255.255.255.128

4. Configuración e Implementación Paso a Paso
Paso 1: Configuración de Red en el Windows Server
Configurar los parámetros IP estáticos en la interfaz de red del servidor:

Dirección IP: 10.7.91.200

Máscara de subred: 255.255.255.128

Puerta de enlace predeterminada: 10.7.91.201

DNS: 127.0.0.1

<img width="386" height="443" alt="imagen_2026-10-30_223318928" src="https://github.com/user-attachments/assets/4f9e5a2c-c619-458a-8b8a-7ce7c0d51792" />


Paso 2: Creación de Grupos de Seguridad en Active Directory
Abrir Usuarios y equipos de Active Directory (dsa.msc).

Crear tres grupos de seguridad globales para el control de acceso por niveles:

grupo0

grupo5

grupo10

creamos a juanSU de usuario de prueba

<img width="753" height="524" alt="imagen_2026-10-30_223213738" src="https://github.com/user-attachments/assets/0393fed1-b0a3-491c-b007-0a419aff03ce" />


Paso 3: Instalación y Configuración del Servicio NPS (RADIUS)
Instalar el rol de Servicios de acceso y directivas de redes (Network Policy and Access Services) desde el Administrador del servidor.

Abrir la consola de directivas de red (nps.msc).

Registrar el Router Cisco como Cliente RADIUS:

Nombre descriptivo: R1

Dirección IP: 10.7.91.201

Secreto compartido (Shared secret): TuClaveSecretaRadius

Crear una Directiva de red en NPS que condicione el acceso utilizando los grupos de Active Directory creados y seleccione el método de Autenticación sin cifrar (PAP).

<img width="772" height="538" alt="imagen_2026-10-30_223502898" src="https://github.com/user-attachments/assets/2280547b-b6d6-4dfa-8d53-0ca946986a9e" />


Paso 4: Configuración del Router Cisco (AAA, RADIUS, SSH y Seguridad)
Aplicar la siguiente configuración en la consola del Router Cisco (R1):

enable
configure terminal

! 1. Configuracion basica e identidad
hostname R1
ip domain-name red.local

! 2. Configuracion de interfaz de red
interface gigabitethernet 0/0
 description Conexion hacia la Red y Servidor NPS
 ip address 10.7.91.201 255.255.255.128
 no shutdown
 exit

! 3. Habilitar servicio SSH v2 y llaves de cifrado
ip ssh version 2
crypto key generate rsa modulus 2048

! 4. Usuario local de respaldo (Emergencia)
username admin privilege 15 secret AdminPass123*

! 5. Configuracion de AAA y Servidor RADIUS
aaa new-model

radius server NPS-SERVER
 address ipv4 10.7.91.200 auth-port 1812 acct-port 1813
 key TuClaveSecretaRadius
exit

aaa group server radius RADIUS-GROUP
 server name NPS-SERVER
exit

aaa authentication login default local group RADIUS-GROUP
aaa authorization exec default local group RADIUS-GROUP

! 6. Habilitar Logs de auditoria y contabilidad
logging on
logging buffered 32768 informational
aaa accounting delay-start
aaa accounting exec default start-stop group RADIUS-GROUP

! 7. Restringir acceso seguro en lineas de consola, auxiliar y VTY
line con 0
 login authentication default
 authorization exec default
 logging synchronous
line aux 0
 login authentication default
 authorization exec default

line vty 0 4
 transport input ssh
 login authentication default
 authorization exec default
end

write memory

5. Pruebas y Validación de Resultados
Prueba de Conectividad Inicial: Desde el Windows Server, ejecutar un ping 10.7.91.201 para verificar la comunicación bidireccional a través del Ethernet Switch.

<img width="663" height="538" alt="imagen_2026-10-30_223619439" src="https://github.com/user-attachments/assets/11dae7e0-3de2-491d-9181-e6a98673c79c" />


Acceso Seguro por SSH: Conectarse al router mediante SSH utilizando las credenciales del usuario de Active Directory (juanSU) y verificar la correcta redirección de la autenticación hacia el servidor NPS.



Validación de Logs: Comprobar los registros de auditoría y eventos en el Visor de Sucesos del servidor NPS y los búferes de logs del router Cisco.

[!NOTE]
Colocar aquí la captura de pantalla de la terminal mostrando el inicio de sesión SSH exitoso autenticado mediante RADIUS/Active Directory.

📂 Archivos del Repositorio
/configs/running-config.txt: Backup completo de la configuración del Router Cisco.

/docs/: Carpeta contenedora de las capturas de evidencia fotográfica del laboratorio.

🎥 Demostración en Video
El video con la explicación detallada, validación de fecha/hora, exposición de la topología y demostración en vivo del acceso SSH se encuentra disponible en YouTube a través del enlace incrustado al inicio de este documento.
