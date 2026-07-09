# Fase 2: Docker, Base de Datos y Transferencia de Archivos

Esta guía explica cómo levantar los servicios de orquestación, transferir información confidencial (como semillas de base de datos e imágenes de usuarios) e importarlas de forma segura.

## 1. Levantar Contenedores (Docker Compose)
Se asume el uso de un archivo `docker-compose.yml`. Para sistemas que interactúan entre sí (Ej: múltiples aplicaciones en el mismo VPS detrás de un Nginx), se recomienda crear una red compartida primero.

```bash
docker network create mi_red_global 2>/dev/null || true
docker compose up -d --build
```

## 2. Transferencia Segura de Archivos Locales (SCP)
Nunca subas archivos como `data_seed.sql` (que contengan hashes de usuarios reales o datos sensibles) a GitHub. Usa `scp` desde tu consola local (Windows/Mac) para transferirlos directamente por SSH.

**Ejemplo de transferencia de archivo SQL:**
```powershell
scp ruta\local\data_seed.sql usuario@IP_DEL_VPS:~/projects/MiProyecto/
```

**Ejemplo de transferencia de imágenes (comprimir primero):**
```powershell
Compress-Archive -Path ruta\local\uploads\* -DestinationPath uploads.zip -Force
scp uploads.zip usuario@IP_DEL_VPS:~/projects/MiProyecto/
```

## 3. Importar Base de Datos (Semilla)
Una vez el archivo SQL esté en el VPS, se debe inyectar al contenedor de base de datos. 

Si el sistema ya generó datos iniciales y el archivo contiene `INSERT INTO`, puedes tener errores de duplicidad. Es recomendable cambiar a `REPLACE INTO` para sobreescribir:
```bash
sed -i 's/INSERT INTO/REPLACE INTO/g' data_seed.sql
```

**Comando de inyección (Forzando UTF-8):**
Es crítico forzar `--default-character-set=utf8mb4` para que las tildes y caracteres especiales (ñ) no se corrompan (Ej: `tama├▒o`).
```bash
docker exec -i nombre_contenedor_db mariadb -u usuario_bd -pClaveSegura --default-character-set=utf8mb4 nombre_bd < data_seed.sql
```

## 4. Resolución de Errores Comunes de BD

### 4.1. Error 500: Data too long for column (Modo Estricto)
Las bases de datos modernas en la nube operan en **Modo Estricto**. Si la estructura de la base de datos dice que un texto admite 80 caracteres, e intentas guardar 81, el sistema arrojará un error 500 en lugar de truncarlo en silencio.
**Solución:** Aumentar el límite de la columna en vivo:
```bash
docker exec -i nombre_contenedor_db mariadb -u usuario_bd -pClave -e "ALTER TABLE mi_tabla MODIFY columna VARCHAR(255) DEFAULT NULL;"
```

### 4.2. Extraer archivos dentro de un Volumen Docker
Las imágenes subidas por usuarios se guardan en Volúmenes de Docker, no en el host. Para pasar el `uploads.zip` a la carpeta real del servidor web dentro de Docker:

```bash
docker cp uploads.zip contenedor_app:/tmp/
docker exec -i contenedor_app bash -c "apt-get update && apt-get install -y unzip && unzip -o /tmp/uploads.zip -d /var/www/html/uploads/ && chown -R www-data:www-data /var/www/html/uploads/"
```
