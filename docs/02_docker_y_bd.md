# Fase 2: Docker, Base de Datos y Transferencias

## 1. Gestión de Contenedores (Docker Compose)
Docker es el estándar para ejecutar bases de datos y servidores web encapsulados.

* **Reconstruir y Levantar los Servicios:** `docker compose -f docker-compose.yml up -d --build`
* **Ver el Estado de los Contenedores:** `docker ps`
* **Apagar el Proyecto:** `docker compose down`
* **Ver Logs en Tiempo Real:** `docker compose logs -f`

---

## 2. Ejecutar Comandos "Dentro" de un Contenedor
Utilizas `docker exec` para introducir comandos como si estuvieras dentro de ellos sin instalar nada en el VPS.
* Instalar dependencias PHP: `docker exec -it <nombre_contenedor_app> composer install`
* Entrar a MySQL / MariaDB (Modo Interactivo):
  ```bash
  docker exec -it <nombre_contenedor_db> mariadb -u <usuario_bd> -p<contraseña> <nombre_bd>
  ```
  *(Sustituye los valores entre `<...>` por los definidos en tu archivo `.env` o `docker-compose.yml`)*
  
  **Comandos útiles dentro de MySQL / MariaDB (`MariaDB [...]>`):**
  - Ver tablas: `SHOW TABLES;`
  - Ver estructura de tabla: `DESCRIBE nombre_tabla;`
  - Ver datos: `SELECT * FROM nombre_tabla LIMIT 5;`
  - Salir: `exit;`

---

## 3. Copias de Seguridad (Backups) y Restauración de Base de Datos

Guía general para respaldar y recuperar cualquier base de datos que corra dentro de un contenedor Docker en el VPS.

### A. Generar Backup Comprimido (.sql.gz) en el VPS
Este comando genera un volcado completo (estructura y datos) directamente comprimido con fecha y hora:

```bash
docker exec <nombre_contenedor_db> mariadb-dump \
  -u <usuario_bd> -p"<contraseña>" \
  <nombre_bd> | gzip > ~/backup_<sistema>_$(date +%Y%m%d_%H%M%S).sql.gz
```

> **Nota:** Si la imagen de base de datos usa MySQL tradicional en lugar de MariaDB, puedes reemplazar `mariadb-dump` por `mysqldump`.

Verifica el archivo generado:
```bash
ls -lh ~/backup_<sistema>_*.sql.gz
```

### B. Descargar el Backup a tu PC Local (PowerShell)
Ejecuta este comando en la terminal local de tu computadora (no dentro del VPS):

```powershell
scp <usuario_vps>@<IP_DEL_VPS>:~/backup_<sistema>_*.sql.gz C:\Users\Usuario\Desktop\
```

### C. Restaurar una Base de Datos desde un Backup (.sql.gz)
Si necesitas restablecer la base de datos a un punto anterior o restaurarla tras recrear el contenedor:

```bash
zcat ~/backup_<sistema>_FECHA.sql.gz | docker exec -i <nombre_contenedor_db> mariadb -u <usuario_bd> -p"<contraseña>" <nombre_bd>
```

---

## 4. Transferencia Segura de Archivos Locales (SCP)
Nunca subas archivos con datos sensibles o hashes de usuarios reales a repositorios públicos. Usa `scp`:

```powershell
scp ruta\local\data_seed.sql usuario@IP_DEL_VPS:~/projects/MiProyecto/
```

---

## 5. Importar Base de Datos (Semilla / Script Inicial)
Inyecta un archivo SQL plano al contenedor. Si ocurren errores de duplicidad de registros, puedes reemplazar por `REPLACE INTO` y forzar codificación UTF-8:

```bash
sed -i 's/INSERT INTO/REPLACE INTO/g' data_seed.sql
docker exec -i <nombre_contenedor_db> mariadb -u <usuario_bd> -p"<contraseña>" --default-character-set=utf8mb4 <nombre_bd> < data_seed.sql
```

---

## 6. Resolución de Errores Comunes de BD

### Error 500: Data too long for column (Modo Estricto)
Aumentar el límite de la columna en vivo sin reiniciar el servicio:
```bash
docker exec -i <nombre_contenedor_db> mariadb -u <usuario_bd> -p"<contraseña>" -e "ALTER TABLE mi_tabla MODIFY columna VARCHAR(255) DEFAULT NULL;" <nombre_bd>
```

### Extraer archivos en Volúmenes de Docker (Subidas / Uploads)
Para transferir carpetas de imágenes, PDFs o archivos adjuntos a un contenedor:
```bash
docker cp uploads.zip <nombre_contenedor_app>:/tmp/
docker exec -i <nombre_contenedor_app> bash -c "unzip -o /tmp/uploads.zip -d /var/www/html/uploads/ && chown -R www-data:www-data /var/www/html/uploads/"
```
