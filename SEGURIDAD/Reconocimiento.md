---
title: "Reconocimiento y OSINT"
type: note
status: active
tags:
  - seguridad
  - recon
  - osint
  - bugbounty
aliases:
  - Recon
  - OSINT
  - Passive Reconnaissance
created: 2026-06-13
updated: 2026-06-13
---

# Reconocimiento y OSINT

> [!INFO] Fuente
> Basado en metodologias de Jhaddix, STOK, TomNomNom, y ProjectDiscovery.

---

## Recon Pasivo (sin interactuar con el target)

### Subdominios

```bash
# Certificate Transparency
curl -s "https://crt.sh/?q=%25.target.com&output=json" | jq -r '.[].name_value' | sort -u

# subfinder (ProjectDiscovery)
subfinder -d target.com -all -o subdomains.txt

# amass
amass enum -passive -d target.com
```

### URLs Historicas

```bash
# gau (TomNomNom)
gau target.com | tee urls.txt

# waybackurls
waybackurls target.com | tee wayback_urls.txt

# katana
katana -u https://target.com -jc -kf -aff -o urls.txt
```

### Parametros

```bash
# paramspider
paramspider -d target.com

# arjun (mas lento pero exhaustivo)
arjun -u https://target.com/api/endpoint
```

### Tecnologias

| Herramienta | Uso |
|-------------|-----|
| Wappalyzer | Extension de browser |
| whatweb | whatweb target.com |
| BuiltWith | builtwith.com |

### Google & GitHub Dorks

```
site:target.com intitle:"index of"
site:target.com inurl:admin
site:target.com filetype:pdf

"target.com" "api_key"
"target.com" "password" extension:env
org:target "aws_secret_access_key"
```

---

## Recon Activo

### Escaneo de puertos

```bash
# nmap basico
nmap -sC -sV -p- --min-rate=1000 target.com

# rustscan (mas rapido)
rustscan -a target.com -b 2000 -- -sC -sV

# masscan (muy rapido, requiere sudo)
sudo masscan -p1-65535 --rate=10000 target.com
```

### Fuzzing de directorios

```bash
# ffuf (recomendado)
ffuf -u https://target.com/FUZZ -w /usr/share/wordlists/directory-list-2.3-medium.txt

# con extensiones
ffuf -u https://target.com/FUZZ -w wordlist.txt -e .php,.asp,.aspx,.jsp,.txt,.bak

# con filtros
ffuf -u https://target.com/FUZZ -w wordlist.txt -fc 404,403
```

### Crawling

```bash
# katana
katana -u https://target.com -d 3 -jc -o crawled.txt

# gospider
gospider -s https://target.com -o output -t 3
```

---

## Screenshots y visualizacion

```bash
# aquatone (toma screenshots de todos los subdominios vivos)
cat subdomains.txt | aquatone -out screenshots/

# httpx + nuclei (ProjectDiscovery)
cat subdomains.txt | httpx -status-code -title -tech-detect | tee live.txt
nuclei -l live.txt -t ~/nuclei-templates/ -o vulnerabilities.txt
```

---

## Flujo de trabajo recomendado

```
1. Obtener subdominios (subfinder + crt.sh + amass)
2. Filtrar hosts vivos (httpx)
3. Crawlear URLs (katana + gospider)
4. Extraer parametros (paramspider, arjun)
5. Fuzzear directorios en hosts clave (ffuf)
6. Escanear puertos en IPs relevantes (nmap)
7. Nuclei para deteccion automatica
8. Screenshots para revision visual (aquatone)
```

> [!TIP] Automatizacion
> Crea scripts que encadenen estos pasos. Ej: `recon.sh target.com` que corra todo el pipeline y deje los resultados organizados en carpetas.

---

## Herramientas esenciales

| Herramienta | Instalacion |
|-------------|-------------|
| subfinder | `go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest` |
| httpx | `go install -v github.com/projectdiscovery/httpx/cmd/httpx@latest` |
| nuclei | `go install -v github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest` |
| ffuf | `go install github.com/ffuf/ffuf/v2@latest` |
| katana | `go install github.com/projectdiscovery/katana/cmd/katana@latest` |
| gau | `go install github.com/lc/gau/v2/cmd/gau@latest` |

---

## Ver tambien

- [[SEGURIDAD/Roadmap Bug Bounty]]
- [[SEGURIDAD/Vulnerabilidades Web]]
- [[SEGURIDAD/Hacking Activo]]
- [[SEGURIDAD/00 - INDICE]]
