<h1 align="center" >Laboratorio OpenVPN - Unitropico</h1></br>
<p align="center"><a href="https://unitropico.edu.co" target="_blank"><img src="https://i.postimg.cc/GtJMcSLD/LOGO-1024x601.png" width=35% alt="Unitropico Logo"></a></p></br>
<p>El taller/Laboratorio de la materia redes convergentes, se siguió el siguiente tutorial para realizar la practica.</p>
- link: https://youtu.be/P7i-oLe2bHk?si=Et1nWIOoXDx15IkF

# Comandos/Pasos para la Instalación y Configuración 

## Actualizar Ubuntu Server
``` bash
sudo apt update
```
## Instalar OpenVpn y Easy-rsa
``` bash
sudo apt install openvpn easy-rsa -y
```
## Verificar Instalación
``` bash
openvpn --help
openvpn --version
```

El CA (Certificate Authority) y la PKI (Public Key Infrastructure) son componentes fundamentales en la seguridad de una red, especialmente en el contexto de VPNs y otros sistemas que requieren autenticación y cifrado.

### Autoridad Certificadora CA
Una Autoridad de Certificación es una entidad confiable que emite certificados digitales utilizados para autenticar la identidad de usuarios, dispositivos o servidores en una red. El CA emite certificados digitales firmados digitalmente, que contienen la clave pública de la entidad a la que pertenecen y otros datos relevantes. Estos certificados son utilizados por otras entidades para verificar la identidad de la parte con la que están comunicando.

### Infrastructura de Clave Publica PKI
La Infraestructura de Clave Pública es el conjunto de hardware, software, políticas y procedimientos necesarios para crear, administrar, distribuir, utilizar y revocar certificados digitales y claves públicas. La PKI proporciona un marco seguro para la gestión de certificados digitales, incluyendo la emisión de certificados por parte de la Autoridad de Certificación, la verificación de la autenticidad de los certificados y el establecimiento de confianza entre las partes que se comunican.

En el contexto de OpenVPN y Easy-RSA, el CA y la PKI son utilizados para establecer una infraestructura de seguridad sólida. Easy-RSA simplifica la creación y administración de la PKI al proporcionar herramientas para generar certificados digitales y administrar la autoridad de certificación. Esto permite a los administradores de sistemas implementar VPNs de manera segura, garantizando que solo los usuarios y dispositivos autorizados puedan acceder a la red protegida.

## Copiar el diectorio de easy-rsa en el directorio de openvpn
``` bash
sudo cp -r /usr/share/easy-rsa/ /etc/openvpn
```
## Acceder al directorio
``` bash
cd /etc/openvpn/easy-rsa
sudo ls -l
```
## Crear la PKI
``` bash
sudo ./easyrsa init-pki
```
## Crear la CA
``` bash
sudo ./easyrsa build-ca
sudo -ls pki/private
```
Esta pedirá establecer una contraseña de administración para poder certificar las diferntes claves de los "clientes"

## Generar claves .key (privada) .req (publica) del servidor
``` bash
sudo ./easyrsa gen-req userver nopass
```
## Firmar la clave .req con el CA para generar certificado .crt
``` bash
sudo ./easyrsa sign-req server userver
```
este pedirá la contraseña al momendo de crear el CA

## Mover las claves privada, publica y el certificado a una carpeta llamada server en la carpeta de openvpn
``` bash
sudo cp /etc/openvpn/easy-rsa/pki/issued/userver.crt /etc/openvpn/server

sudo cp /etc/openvpn/easy-rsa/pki/ca.crt /etc/openvpn/server

sudo cp /etc/openvpn/easy-rsa/pki/private/userver.key /etc/openvpn/server
```
## Crear Clave TLS-Crypt
La clave TLS-Crypt es una característica de seguridad adicional que se puede utilizar en OpenVPN para proteger la integridad de los datos de control (encabezados y mensajes de control) transmitidos durante una sesión VPN. Proporciona una capa adicional de seguridad al cifrar estos datos de control, lo que dificulta la detección y manipulación de paquetes por parte de terceros.

```bash
cd /etc/openvpn/server
sudo openvpn --genkey --secret ta.key
```
## Realizar el procedimiento para claves del cliente
```bash
sudo mkdir /etc/openvpn/cliente/keys
```
## Quitar privilegios a los clientes
```bash
sudo chmod -R 700 /etc/openvpn/cliente
```
## Generar claves .key y .req para el cliente
```bash
cd /etc/openvpn/easy-rsa
sudo ./easyrsa gen-req cliente1-userver nopass
```
## Firmar la clave .req con el CA para generar certificado .crt
```bash
sudo ./easyrsa sign-req cliente cliente1-userver
```
## Mover las claves privada, publica y el certificado a la carpeta creada para el cliente
```bash
sudo cp /etc/openvpn/easy-rsa/pki/issued/cliente1-userver.crt /etc/openvpn/cliente/keys

sudo cp /etc/openvpn/easy-rsa/pki/ca.crt /etc/openvpn/cliente/keys

sudo cp /etc/openvpn/easy-rsa/pki/private/cliente1-userver.key /etc/openvpn/cliente/keys

sudo cp /etc/openvpn/server/ta.key /etc/openvpn/cliente/keys
```
## Buscar fichero de configuracion Server.conf 
```bash
ls /usr/share/doc/openvpn/examples/sample-config-files/
```
## Mover fichero de configuracion a la carpeta server de openvpn
```bash
sudo cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf /etc/openvpn/server

'Si el archivo esta .gz es porque esta comprimido, se descomprime así'
sudo gunzip /etc/openvpn/server/server.config.gz
```
## Editar fichero de configuracion server.conf
```conf
# 1. INTERFAZ DE ESCUCHA
;local a.b.c.d

# 2. PUERTO A UTILIZAR (TCP O UDP) POR DEFECTO ES 1194
port 1194

# 3. PROTOCOLO A UTILIZAR: TCP O UDP
;proto tcp
proto udp

# 4. TIPO DE TUNEL: dev tun(enrutamiento IP) o dev tap (puente Ethernet)
;dev tap
dev tun
;dev-node MyTap

# 5. MODIFICAR EL NOMBRE DE LAS CLAVES POR EL QUE LAS HEMOS GENERADO
ca ca.crt
cert userver.crt
key userver.key

# 6. Desactivar la directiva Diffi-Hellman
;dh dh2048.pem
dh none

# 7. TOPOLOGIA DE LA RED
topology subnet

# 8. DIRECCION DE SUBRED DE LA VPN (ip_servidor) SOLO PARA dev tun
server 10.8.0.0 255.255.255.0

# 9. CONFIGURACION PARA QUE LOS CLIENTES TENGAN LA MISMA IP SIEMPRE
ifconfig-pool-persist /var/log/openvpn/ipp.txt

# 10. PERMITE A LOS CLIENTES TENER ACCESO A INTERNET EN LA VPN
push "redirect-gateway def1 bypass-dhcp"

# 14. HABILITAMOS KEEPALIVE PARA SABER SI EL TUNEL ESTA CAIDO
keepalive 10 120

# 15. ACTIVAR CLAVE EXTRA DE SEGURIDADD TLS-CRYPT
;tls-auth ta.key 0
tls-crypt ta.key

# 16. TIPO DE CIFRADO
;cipher AES-256-CBC
cipher AES-256-GCM
auth SHA512

# 18. NUM MAXIMO DE CLIENTES SIMULTANEOS
max-clients 100

# 19. SIN PERMISOS DE USUARIO Y GRUPO
user nobody
group nogroup

# 20. CLAVE Y TUNEL PERSISTENTE
persist-key
persist-tun

# 21. REGISTRO DE CONEXIONES ACTUALES
status /var/log/openvpn/openvpn-status.log

# 23. VERVOSITY
verb 3

# 25. NOTIFICACION DE REINICIO
explicit-exist-nofify 1
```
## Mover fichero cliente
```bash
sudo cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf /etc/openvpn/server
```
## Modificando fichero cliente
```conf
client
remote my-server-1 1194
;remote my-server-2 1194

proto udp

dev  tun

;remote-random

resolv-retry infinite

nobind

user nobody
group nogroup
persist-key
persist-tun

;http-proxy-rety # retry on connection failures
;http-proxy [proxy server] [ proxy port #]

;ca ca.crt
;cert client.crt
;key client.key
;tls-auth ta.key 1

cipher AES-256-GCM
auth SHA512
```

## Habilidanto net.ipv4.ip_forward en ./etc/sysctl.con
```bash
sysctl -p
net.ipv4.ip_forward = 1
```
## Configurando reglas de firewall con iptables
```bash
iptables -t nat -I POSTROUTING 1 -S 10.8.0.0/24 -0 enp0s3 -j MASQUERADE

iptables -I INPUT 1 -i tun0 -j ACCEPT
iptables -I FORWARD 1 -i enp0s3 -o tun0 -j ACCEPT
iptables -I FORWARD 1 -i tun0 -o enp0s3 -j ACCEPT
iptables -I INPUT 1 -i enp0s3 -p udp --dport 1194 -j ACCEPT
```
## Instalando iptables-persistent
```bash
sudo apt install iptables-persistent -y
```
## Guardando reglas de firewall con netfilter-persistent save
```bash
netfilter-persistent save
```
