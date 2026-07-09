# 🚀 Guía de Despliegue en Producción (Docker + Nginx + SSL)

Esta guía documenta el proceso detallado para el despliegue del **Sistema Impobiomedical** (y otros sistemas MVC similares) en un servidor VPS Ubuntu usando contenedores Docker.

## 1. Clonación y Permisos Iniciales
Al clonar el repositorio, es fundamental asegurarse de que el usuario actual tenga permisos sobre los archivos y no queden a nombre de `root` (lo cual causaría fallos de lectura/escritura o errores `Permission denied` en el `.env`).

```bash
git clone https://github.com/Santiago072/SistemaImpobiomedical.git
cd SistemaImpobiomedical
sudo chown -R $USER:$USER .
```

## 2. Configuración del Entorno (.env)
Se crea el archivo `.env` necesario para que el sistema funcione. En producción debe asegurarse el uso de contraseñas fuertes y preparar las variables para HTTPS (`COOKIE_SECURE=1`).

```bash
cat > .env << 'EOF'
DB_HOST=impobiomedical_db
DB_USER=impo_user
DB_PASS=ContraseñaSegura123
DB_NAME=sistema_impobiomedical
MYSQL_ROOT_PASSWORD=RootSeguro!

APP_BASE=/
SESSION_LIFETIME=3600
COOKIE_SECURE=1

UPLOAD_MAX_SIZE=5242880
ALLOWED_EXTENSIONS=jpg,jpeg,png,gif,webp
EOF
```

## 3. Orquestación con Docker Compose
1. **Red Dedicada:** Se debe garantizar la existencia de una red compartida si otros sistemas (como Sodicol) necesitan interactuar o usar un proxy global.
   ```bash
   docker network create sodicol_network 2>/dev/null || true
   ```
2. **Levantar el Servicio:** Usamos `--build` para que PHP reconstruya el código dentro del contenedor (el código fuente no se "lee" del disco en vivo por seguridad).
   ```bash
   docker compose up -d --build
   ```

## 4. Inicialización de Datos (Seed de la Base de Datos)
Por motivos de seguridad, los archivos de importación (`data_seed.sql`) que contienen hashes de usuarios, correos y clientes **no deben estar en el repositorio de GitHub**. 

### 4.1 Método recomendado de transferencia
Descarga segura (si el archivo temporal está en un entorno controlado) o copiarlo vía SCP desde XAMPP local al VPS:
```powershell
# En la PC Local
scp c:\xampp\htdocs\SistemaImpobiomedical\data_seed.sql santiago@IP_VPS:~/projects/SistemaImpobiomedical/
```

### 4.2 Importación en MariaDB
Si ocurre el error `ERROR 1062 (23000) Duplicate entry '1'`, significa que la BD creó al usuario administrador por defecto (`BD.txt`). En ese caso, transformamos los `INSERT` en `REPLACE` para sobreescribir la info:

```bash
sed -i 's/INSERT INTO/REPLACE INTO/g' data_seed.sql
docker exec -i impobiomedical_db mariadb -u impo_user -pContraseñaSegura123 sistema_impobiomedical < data_seed.sql
```

> **NOTA SOBRE STRICT MODE:** Si ocurre el error `Data too long for column`, se debe a que un texto supera el límite de un campo VARCHAR bajo el modo estricto de MySQL/MariaDB. Solución en caliente:
> ```bash
> docker exec -i impobiomedical_db mariadb -u impo_user -pClave123 sistema_impobiomedical -e "ALTER TABLE cotizacion_items MODIFY tiempo_entrega VARCHAR(255) DEFAULT NULL;"
> ```

## 5. Transferencia de Imágenes Subidas (Uploads)
Las imágenes de productos se almacenan en un *volumen de Docker*. No basta con dejarlas en la carpeta del host; hay que inyectarlas al contenedor.

1. **PC Local (PowerShell):** Comprimir y enviar
   ```powershell
   Compress-Archive -Path c:\xampp\htdocs\SistemaImpobiomedical\uploads\* -DestinationPath uploads.zip -Force
   scp uploads.zip santiago@IP_VPS:~/projects/SistemaImpobiomedical/
   ```
2. **VPS (SSH):** Inyectar y extraer en el contenedor
   ```bash
   docker cp uploads.zip impobiomedical_app:/tmp/
   docker exec -i impobiomedical_app bash -c "apt-get update && apt-get install -y unzip && unzip -o /tmp/uploads.zip -d /var/www/html/uploads/ && chown -R www-data:www-data /var/www/html/uploads/"
   ```

## 6. Configuración del Proxy Inverso (Nginx) y SSL Seguro
Para exponer el sistema mediante un dominio (`sistemaimpobiomedical.dominio.online`) y utilizar el puerto interno de Docker (Ej: `8895`), Nginx actuará de mediador.

### 6.1 Crear el bloque de servidor
Si la terminal corta los comandos por portapapeles, usar `nano` para crearlo a mano:
```bash
sudo nano /etc/nginx/sites-available/sistemaimpobiomedical
```
**Contenido:**
```nginx
server {
    listen 80;
    server_name sistemaimpobiomedical.slscode.online;

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

### 6.2 Activar y obtener Certificado Let's Encrypt
```bash
sudo ln -s /etc/nginx/sites-available/sistemaimpobiomedical /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
sudo certbot --nginx -d sistemaimpobiomedical.slscode.online
```

## 7. Notas Finales de Mantenimiento (`deploy.sh`)
* El uso de `bash deploy.sh` evita depender de permisos locales de ejecución.
* El proceso toma tiempo porque la bandera `--build` descarta el contenedor viejo, construye uno fresco leyendo los archivos del disco y lo levanta garantizando una imagen limpia. No hay recarga "en caliente" en producción por normas arquitectónicas.
