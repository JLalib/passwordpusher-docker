# 🔐 PasswordPusher: Compartidor seguro de contraseñas y secretos con Docker

[![GitHub](https://img.shields.io/badge/GitHub-Repositorio-blue)](https://github.com/JLalib/passwordpusher-docker) [![Docker](https://img.shields.io/badge/Docker-PasswordPusher-blue)](https://hub.docker.com/r/pglombardo/pwpush) [![License](https://img.shields.io/badge/Licencia-MIT-green)](https://github.com/JLalib/passwordpusher-docker/blob/main/LICENSE)

## 📋 Descripción general

PasswordPusher es una herramienta minimalista de código abierto para compartir contraseñas, notas, URLs y archivos de forma segura, sin dejarlos en canales públicos como Slack, email o chat inseguro. Generas un link que se auto-destruye tras un número de vistas o un tiempo determinado, con auditoría completa de accesos.

Este repositorio contiene la configuración necesaria para desplegar PasswordPusher con Docker Compose, siguiendo el tutorial de Genbyte para dejar de compartir secretos por canales inseguros.

## ✨ Características principales

- **Compartidor de secretos**: contraseñas, notas, URLs y archivos, todo encriptado y con auto-expiración
- **Expiración flexible**: por vistas (ej. 1 vista), por tiempo (ej. 24 horas), o ambos combinados
- **Encriptación AES-256**: en reposo, y HTTPS en tránsito; los datos nunca viajan en texto plano
- **Passphrase adicional**: capa extra de seguridad, además del propio link
- **Audit logging completo**: quién vio el secreto, cuándo, desde qué IP y navegador
- **Multi-usuario con login**: cuentas de equipo, cada usuario gestiona sus propios pushes
- **JSON API + CLI oficial**: automatiza desde scripts o terminal con `pwpush-cli`
- **Webhooks**: notificaciones al Slack, Teams, etc. cuando alguien accede a un push
- **31 idiomas**: interfaz multilingüe traducida vía Translation.io
- **HTTPS automático**: certificado Let's Encrypt vía variable `TLS_DOMAIN`
- **Open source Apache 2.0**: sin telemetría, totalmente auditable

## 📋 Requisitos del sistema

- Docker y Docker Compose instalados
- Al menos 512 MB - 1 GB de RAM (Ruby on Rails es ligero)
- 1-2 GB de espacio en disco (según volumen de pushes)
- Puerto 443 disponible para HTTPS (recomendado), o puerto 80
- Modo ephemeral (sin base de datos, no persiste) o modo producción con PostgreSQL/MySQL
- Un dominio propio si quieres usar `TLS_DOMAIN` para HTTPS automático

## 🐳 Instalación

### Docker Compose (Método recomendado)

Crea un archivo `docker-compose.yml` con el siguiente contenido:

```yaml
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
```

Luego, inicia el servicio:

```bash
docker compose up -d
```

💡 Alternativa rápida sin persistencia (modo ephemeral, ideal para pruebas):

```bash
docker run -d \
  --name passwordpusher \
  --restart unless-stopped \
  -p 5100:5100 \
  pglombardo/pwpush-ephemeral:release
```

## ⚙️ Configuración

Antes de iniciar el contenedor, debes editar el archivo `docker-compose.yml` para establecer tus valores:

1. **TLS_DOMAIN**: dominio propio para el certificado automático de Let's Encrypt (ej. `secrets.tudominio.com`)
2. **PWPUSH_MASTER_KEY**: clave de encriptación personalizada, genérala con `openssl rand -hex 32`
3. **PWPUSH_ENABLE_USER_ACCOUNT_EMAILS**: activa o desactiva las notificaciones por email
4. **PWP__LOG_LEVEL**: nivel de detalle de los logs (`info`, `debug`, etc.)

💡 Consejo: genera siempre una `PWPUSH_MASTER_KEY` aleatoria y única, no uses el valor de ejemplo.

```bash
openssl rand -hex 32
# Copia el resultado y reemplázalo en docker-compose.yml
```

## 🚀 Primeros pasos

1. Asegúrate de tener Docker y Docker Compose instalados en tu sistema
2. Clona este repositorio o copia el archivo `docker-compose.yml` a tu servidor
3. Edita el archivo `docker-compose.yml` y reemplaza:
   - `secrets.tudominio.com` por tu dominio real
   - `your-random-32-char-hex-key-here` por tu clave generada con `openssl rand -hex 32`
4. Ejecuta `docker compose up -d` para iniciar el contenedor
5. Abre tu navegador en `https://secrets.tudominio.com` (HTTPS automático vía Let's Encrypt)
6. Crea tu primer push:
   - Click en "Create a Push"
   - Pega el secreto (contraseña, URL o nota)
   - Configura la expiración: vistas y/o tiempo
   - Opcional: añade una passphrase
   - Click en "Push It" y comparte el link generado

## 💡 Casos de uso

- **DevOps/SRE**: compartir credenciales temporales, API keys o contraseñas de bases de datos de forma segura
- **Equipos remotos**: onboarding seguro, sin distribuir credenciales por WhatsApp o email
- **Soporte IT**: compartir contraseñas con clientes sin dejar historial permanente
- **Compliance-heavy**: auditoría completa de quién vio qué, cuándo y desde dónde
- **Terceros/Contractors**: accesos temporales que caducan automáticamente

## 🔒 Acceso remoto seguro (opcional)

Si prefieres usar Caddy como proxy inverso en lugar de dejar que PasswordPusher gestione su propio TLS, puedes hacerlo así.

### Configuración Caddyfile (ejemplo)

```
secrets.tudominio.com {
    reverse_proxy localhost:5100
    encode gzip
}
```

### Pasos para acceso seguro

1. Instala y configura Caddy, Nginx Proxy Manager o Traefik en tu servidor
2. Obtén un certificado SSL gratuito de Let's Encrypt (Caddy lo hace automáticamente)
3. Configura el proxy inverso para apuntar a `localhost:5100`
4. Accede a PasswordPusher a través de tu dominio seguro (<https://secrets.tudominio.com>)

📝 Nota: PasswordPusher puede gestionar su propio HTTPS de forma nativa vía la variable `TLS_DOMAIN`. Usa Caddy solo si prefieres administrar los certificados externamente.

## 🛠️ Gestión y mantenimiento

### Ver registros

```bash
docker compose logs -f pwpush
```

### Actualizar a la última versión

```bash
docker compose pull
docker compose up -d
```

### Reiniciar servicios

```bash
docker compose restart
```

### Ver estado de salud

```bash
docker stats passwordpusher
```

### Backup de datos (si usas PostgreSQL)

```bash
docker compose exec postgres pg_dump -U passwordpusher_user passwordpusher_db > pwpush-$(date +%Y%m%d).sql
```

### Limpiar y comenzar desde cero

```bash
docker compose down -v  # Elimina contenedores, redes y volúmenes
# Luego vuelve a levantar con: docker compose up -d
```

## 📝 Licencia

Este proyecto se basa en [PasswordPusher](https://github.com/pglombardo/PasswordPusher), licenciado bajo Apache 2.0. La configuración y documentación proporcionada aquí está bajo la [MIT License](https://github.com/JLalib/passwordpusher-docker/blob/main/LICENSE).

---

> ✨ **Nota**: Este repositorio contiene la configuración Docker y documentación extraída del tutorial de Genbyte: <a href="https://genbyte.blogspot.com/2026/07/como-instalar-passwordpusher-en-docker.html" target="_blank" rel="noopener noreferrer">Cómo instalar PasswordPusher en Docker - Compartidor seguro de contraseñas autohospedado en Docker</a>
