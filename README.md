# 🔐 PasswordPusher Docker - Compartidor Seguro de Contraseñas Autohospedado

[![GitHub](https://img.shields.io/badge/GitHub-Repository-blue?logo=github)](https://github.com/pglombardo/PasswordPusher)
[![Docker](https://img.shields.io/badge/Docker-pglombardo%2Fpwpush-blue?logo=docker)](https://hub.docker.com/r/pglombardo/pwpush)
[![License](https://img.shields.io/badge/License-Apache%202.0-green)](https://github.com/pglombardo/PasswordPusher/blob/master/LICENSE)

## 📋 Descripción general

**PasswordPusher** es una herramienta minimalista y open source para compartir secretos de forma segura sin dejarlos en canales públicos como Slack, email o chat inseguro. En lugar de enviar contraseñas por canales inseguros, generas un link que se auto-destruye después de X vistas o tiempo. El secreto nunca toca email o chat: generas un link seguro, el destinatario lo accede, y después de la expiración se borra permanentemente.

Características clave:
- 🔒 **Encriptación AES-256 en reposo** + HTTPS en tránsito
- ⏱️ **Expiración flexible**: por vistas (ej: 1 view) o por tiempo (ej: 24 horas)
- 📊 **Audit logging completo**: quién vio, cuándo, IP address, navegador
- 👥 **Multi-usuario** con login y panel de administración
- 🤖 **JSON API + CLI oficial** (`pwpush-cli`) para automatización
- 🌐 **31 idiomas** y tema customizable (white-label)
- 🐳 **Docker one-liner deployment** con HTTPS automático vía Let's Encrypt
- 📜 **Apache 2.0 open source** - sin telemetría, auditable

## ✨ Características principales

- **Compartidor de secretos**: Contraseñas, notas, URLs, archivos - todo encriptado y auto-expira
- **Expiración flexible**: Por vistas (ej: 1 view) o por tiempo (ej: 1 hour, 1 day) - o ambos
- **Encriptación AES-256**: En reposo. HTTPS en tránsito. Datos nunca en texto plano
- **Passphrase adicional**: Capa extra - link + passphrase para acceder
- **Audit logging completo**: Quién vio, cuándo, IP address, navegador - tracking exhaustivo
- **Multi-usuario + login**: Equipos con cuentas. Cada usuario ve sus pushes
- **JSON API**: Integra con curl, scripts, tools - automatización completa
- **CLI oficial (pwpush-cli)**: Comparte secrets desde terminal - pipe-friendly
- **Webhooks**: Notificaciones cuando se accede - integra con Slack, Teams, etc.
- **HTTPS automático**: Let's Encrypt vía variable `TLS_DOMAIN` - setup en segundos
- **31 idiomas**: UI multilingüe - Translation.io powered
- **Open source Apache 2.0**: Sin telemetría, sin black box, auditable

## 📋 Requisitos del sistema

- **Docker** y **Docker Compose** instalados
- **512 MB - 1 GB RAM** (Ruby on Rails es ligero)
- **1-2 GB espacio disco** (depende del volumen de pushes)
- **Puerto 443** (HTTPS, recomendado) o **80** (HTTP)
- **Opción 1 (ephemeral)**: Sin DB, datos en memoria (no persiste entre restarts)
- **Opción 2 (production)**: PostgreSQL o MySQL para persistencia
- **TLS_DOMAIN**: Dominio para Let's Encrypt (ej: `secrets.tudominio.com`)
- **PWPUSH_MASTER_KEY** (recomendado): Key de encriptación custom (generar con `openssl rand -hex 32`)

## 🐳 Instalación

### Opción 1: Ephemeral (sin DB, rápido pero sin persistencia)

```bash
docker run -d \
  --name passwordpusher \
  --restart unless-stopped \
  -p 5100:5100 \
  pglombardo/pwpush-ephemeral:release
```

Acceso: `http://localhost:5100` - Dashboard. **Datos NO persisten entre restarts.**

---

### Opción 2: Production con PostgreSQL (recomendado - oficial)

```bash
curl -s -o docker-compose.yml https://raw.githubusercontent.com/pglombardo/PasswordPusher/master/containers/docker/pwpush-postgres/docker-compose.yml
docker compose up -d
```

---

### Opción 3: Docker Compose custom con HTTPS automático (Let's Encrypt)

```bash
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  pwpush:
    image: pglombardo/pwpush:latest
    container_name: passwordpusher
    restart: unless-stopped
    ports:
      - "443:443"
      - "80:80"
    volumes:
      - pwpush-storage:/opt/PasswordPusher/storage
    environment:
      # Domain para Let's Encrypt (IMPORTANTE)
      - TLS_DOMAIN=secrets.tudominio.com
      # Encryption key (generar: openssl rand -hex 32)
      - PWPUSH_MASTER_KEY=your-random-32-char-hex-key-here
      # Email (para notificaciones)
      - PWPUSH_ENABLE_USER_ACCOUNT_EMAILS=false
      # Logging
      - PWP__LOG_LEVEL=info

volumes:
  pwpush-storage:
EOF

docker compose up -d
```

### Generar `PWPUSH_MASTER_KEY`

```bash
openssl rand -hex 32
# Copia resultado y reemplaza en docker-compose.yml
```

### Acceder

```
https://secrets.tudominio.com
```
Dashboard PasswordPusher con **HTTPS automático** vía Let's Encrypt.

## ⚙️ Configuración

1. **TLS_DOMAIN**: Tu dominio/subdominio (ej: `secrets.tudominio.com`) - obligatorio para HTTPS automático
2. **PWPUSH_MASTER_KEY**: Clave de encriptación de 32 chars hex - generar con `openssl rand -hex 32`
3. **PWPUSH_ENABLE_USER_ACCOUNT_EMAILS**: `true`/`false` - habilita emails para recuperación de cuenta
4. **PWP__LOG_LEVEL**: `debug`, `info`, `warn`, `error` - nivel de logging
5. **PWP__PURGE_AFTER**: Ej: `"2 weeks"` - purga automática de pushes expirados
5. **Puertos**: 443 (HTTPS) y 80 (HTTP para challenge Let's Encrypt) deben estar abiertos en firewall/router

## 🚀 Primeros pasos

1. **Crear primer push (desde web)**
   - Abre `https://secrets.tudominio.com`
   - Click **"Create a Push"**
   - Pega secreto: contraseña, URL, nota, o contenido de archivo
   - Set expiry: número de vistas (ej: `1`) + duración (ej: `1 hour`, `1 day`)
   - Opcional: agregar **passphrase** para capa extra
   - Click **"Push It"**
   - Copia link seguro generado
   - Comparte link con recipient (sin compartir passphrase por ese canal)

2. **Recipient accede**
   - Recipient abre link
   - Si hay passphrase, la introduce
   - Ve el secreto
   - Link se auto-destruye (1 view usado)
   - Futuro acceso: *"This push has expired"*

3. **Ver audit log**
   - Dashboard → tu push
   - Verás: quién vio, cuándo, IP address, navegador
   - Tracking exhaustivo

4. **Usar CLI (pwpush-cli)**
   ```bash
   # Instalar CLI
   pip install pwpush
   
   # Compartir secreto desde terminal
   pwpush push --secret "my-password" --days 1 --views 3
   # Resultado: https://secrets.tudominio.com/p/abc123xyz
   ```

5. **Usar API (curl)**
   ```bash
   curl -X POST https://secrets.tudominio.com/api/v1/pushes \
     -H "Content-Type: application/json" \
     -d '{ "push": { "payload": "my-secret-password", "expire_after_views": 1, "expire_after_days": 1 } }'
   ```

6. **Agregar multi-usuario**
   - Settings → Users (si estás en admin)
   - Crea usuarios para equipo
   - Cada usuario: su view de pushes
   - Auditoría: quién creó qué

## 💡 Casos de uso

- **DevOps/SRE**: Compartir credenciales temporales, API keys, database passwords de forma segura
- **Equipos remotos**: Onboarding seguro. Distribuir credenciales sin WhatsApp/email
- **Soporte IT**: Share passwords con clientes sin historial permanente
- **Compliance-heavy**: Auditoría completa: quién vio qué, cuándo, desde dónde
- **Terceros/Contractors**: Temporary access credentials que auto-expiran

## 🔒 Acceso remoto seguro

PasswordPusher maneja **HTTPS nativo** vía `TLS_DOMAIN` (Let's Encrypt automático). Es la opción recomendada.

**Alternativa: Caddy como reverse proxy** (si prefieres gestionar certs externamente):

```caddy
secrets.tudominio.com {
    reverse_proxy localhost:5100
}
```

> **Nota**: Mejor dejar que PasswordPusher maneje su propio TLS vía variable de entorno `TLS_DOMAIN`. Caddy es backup si prefieres manage certs externamente.

## 🛠️ Gestión y mantenimiento

| Acción | Comando |
|--------|---------|
| **Ver logs** | `docker compose logs -f pwpush` |
| **Backup (PostgreSQL)** | `docker compose exec postgres pg_dump -U passwordpusher_user passwordpusher_db > pwpush-$(date +%Y%m%d).sql` |
| **Reiniciar** | `docker compose restart` |
| **Actualizar** | `docker compose pull && docker compose up -d` |
| **Monitorear consumo** | `docker stats passwordpusher` |
| **Purgar pushes expirados** | Configurar `PWP__PURGE_AFTER="2 weeks"` en environment |

## 📝 Licencia

**Apache License 2.0** - Open source, sin telemetría, auditable.

[Ver licencia completa](https://github.com/pglombardo/PasswordPusher/blob/master/LICENSE)

---

> 📖 **Basado en el post:** [Cómo instalar PasswordPusher en Docker - Compartidor seguro de contraseñas autohospedado en Docker](https://genbyte.blogspot.com/2026/07/como-instalar-passwordpusher-en-docker.html)