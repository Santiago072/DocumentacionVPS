# Fase 2: Docker, Base de Datos y Transferencias

## 1. Gestión de Contenedores (Docker Compose)
Docker es el estándar para ejecutar bases de datos y servidores web encapsulados.

* **Reconstruir y Levantar los Servicios:** \docker compose -f docker-compose.yml up -d --build\
* **Ver el Estado de los Contenedores:** \docker ps\
* **Apagar el Proyecto:** \docker compose down\
* **Ver Logs en Tiempo Real:** \docker compose logs -f\

## 2. Ejecutar Comandos "Dentro" de un Contenedor
Utilizas docker exec para introducir comandos como si estuvieras dentro de ellos sin instalar nada en el VPS.
* Instalar dependencias PHP: \`docker exec -it mi_app_php composer install\`
* Entrar a MySQL (Modo Interactivo):
  ```bash
  docker exec -it <nombre_contenedor_db> mysql -u <usuario_bd> -p<contraseña> <nombre_bd>
  ```
  *(Sustituye `<nombre_contenedor_db>`, `<usuario_bd>`, `<contraseña>` y `<nombre_bd>` por los valores de tu `.env`)*
  
  **Comandos útiles dentro de MySQL (`MariaDB [...]>`):**
  - Ver tablas: `SHOW TABLES;`
  - Ver estructura de tabla: `DESCRIBE nombre_tabla;`
  - Ver datos: `SELECT * FROM nombre_tabla LIMIT 5;`
  - Salir: `exit;`

## 3. Transferencia Segura de Archivos Locales (SCP)
Nunca subas archivos con hashes de usuarios reales a GitHub. Usa \scp\:
\\\powershell
scp ruta\local\data_seed.sql usuario@IP_DEL_VPS:~/projects/MiProyecto/
\\\

## 4. Importar Base de Datos (Semilla)
Inyecta el archivo SQL al contenedor. Si ocurren errores de duplicidad, usa \REPLACE INTO\ y fuerza \UTF-8\:
\\\ash
sed -i 's/INSERT INTO/REPLACE INTO/g' data_seed.sql
docker exec -i nombre_contenedor_db mariadb -u usuario_bd -pClave --default-character-set=utf8mb4 nombre_bd < data_seed.sql
\\\

## 5. Resolución de Errores Comunes de BD
**Error 500: Data too long for column (Modo Estricto)**
Aumentar el límite de la columna en vivo:
\\\ash
docker exec -i nombre_contenedor_db mariadb -u usuario_bd -pClave -e "ALTER TABLE mi_tabla MODIFY columna VARCHAR(255) DEFAULT NULL;"
\\\

**Extraer archivos en Volúmenes de Docker:**
\\\ash
docker cp uploads.zip contenedor_app:/tmp/
docker exec -i contenedor_app bash -c "unzip -o /tmp/uploads.zip -d /var/www/html/uploads/ && chown -R www-data:www-data /var/www/html/uploads/"
\\\
"@ -Encoding UTF8

Set-Content -Path docs/03_nginx_y_ssl.md -Value @"
# Fase 3: Proxy Inverso (Nginx) y Seguridad SSL (HTTPS)

## 1. Enrutamiento y Dominios (Nginx Nativo)
Para que un proyecto en Docker (puerto \8892\) responda a un dominio real, debes configurar Nginx como proxy inverso.

**Crear el archivo de configuración:**
\\\ash
sudo nano /etc/nginx/sites-available/miproyecto.conf
\\\

**Bloque de Proxy Inverso Básico:**
\\\
ginx
server {
    listen 80;
    server_name midominio.com;
    location / {
        proxy_pass http://127.0.0.1:8892;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
\\\

**Activar y Aplicar:**
\\\ash
sudo ln -s /etc/nginx/sites-available/miproyecto.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
\\\

## 2. Certificados SSL (HTTPS) y el "Efecto Fallback"
**Problema del Fallback:** Ingresas a tu dominio con \https://\ y muestra el sitio de otro proyecto. Ocurre porque Nginx no tiene configuración segura para tu dominio, así que entra en pánico y muestra el primer sitio web seguro que encuentre en el servidor por defecto.

**La Solución:** Usar Certbot para adueñarte de tu conexión segura.
\\\ash
sudo certbot --nginx -d midominio.com
\\\
Elige la opción para redirigir todo el tráfico a HTTPS. Si ya tenías un certificado, te preguntará si deseas "Attempt to reinstall this existing certificate" (opción 1).
