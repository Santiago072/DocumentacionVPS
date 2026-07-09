# Fase 3: Proxy Inverso (Nginx) y Seguridad SSL (HTTPS)

Para exponer el sistema de forma segura mediante un dominio o subdominio, se utiliza Nginx como proxy inverso, redirigiendo el tráfico del puerto interno de Docker hacia el puerto 80 (HTTP) y 443 (HTTPS).

## 1. Instalación de Nginx y Certbot
Si el VPS es nuevo, asegúrate de tener las herramientas necesarias:
```bash
sudo apt update
sudo apt install -y nginx certbot python3-certbot-nginx
```

## 2. Creación del Bloque de Servidor (Server Block)
Es fundamental mapear el dominio al puerto correcto donde está expuesto el contenedor (Ej: `8895`).

1. Crea o edita el archivo de configuración:
   ```bash
   sudo nano /etc/nginx/sites-available/misistema
   ```
2. Agrega la configuración de proxy. Reemplaza `misistema.midominio.com` por el dominio real y `8895` por el puerto expuesto en el `docker-compose.yml`:
   ```nginx
   server {
       listen 80;
       server_name misistema.midominio.com;

       location / {
           proxy_pass http://localhost:8895;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
           proxy_read_timeout 120s;
       }
   }
   ```

## 3. Activación y Reinicio de Nginx
Crea un enlace simbólico para activar la configuración y recarga Nginx:
```bash
sudo ln -s /etc/nginx/sites-available/misistema /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```
*(Nota: Si `nginx -t` reporta algún error, revisa que no te falte ningún punto y coma `;` en el archivo).*

## 4. Generación de Certificado SSL (Let's Encrypt)
Usa Certbot para instalar automáticamente el certificado SSL y forzar la redirección HTTP a HTTPS.

```bash
sudo certbot --nginx -d misistema.midominio.com
```

Certbot preguntará si deseas redirigir todo el tráfico a HTTPS; selecciona la opción para redirigir (opción 2) para asegurar el tráfico. 

### 4.1. Renovación Automática
El certificado expira en 90 días, pero Certbot configura un cron job automáticamente en Ubuntu. Puedes simular la renovación para garantizar que funcione:
```bash
sudo certbot renew --dry-run
```
