# Guía de Administración y Comandos para Servidores VPS (Linux / Ubuntu)

Este repositorio contiene una guía general de comandos y buenas prácticas para la administración de un Servidor Virtual Privado (VPS). La información aquí descrita está diseñada para gestionar proyectos web modernos basados en contenedores (Docker), repositorios remotos (Git) y arquitecturas típicas de Linux.

---

## 📁 1. Navegación Básica y Carpetas Comunes

En un servidor Linux, las ubicaciones donde guardas los archivos son fundamentales.

* **`/home/usuario/` o `~/`**: Es tu carpeta personal. Aquí sueles guardar los proyectos que estás desarrollando o clonando desde GitHub (ej. `~/projects/MiProyecto`).
* **`/srv/` o `/var/www/`**: Carpetas reservadas tradicionalmente para datos de servicios web. Si usas Docker con volúmenes persistentes, es muy común crear carpetas como `/srv/shared/` para guardar archivos que no se deben borrar al reiniciar contenedores (imágenes, PDFs subidos por usuarios, backups, etc.).
* **`/tmp/`**: Carpeta para archivos temporales que se borran automáticamente al reiniciar el VPS.

### Comandos de Navegación y Gestión de Carpetas:
```bash
cd /ruta/hacia/carpeta  # Entrar a una carpeta
cd ..                   # Subir un nivel (salir de la carpeta actual)
ls -la                  # Listar todos los archivos y carpetas mostrando detalles y permisos
mkdir nombre_carpeta    # Crear una nueva carpeta
rm -rf nombre_carpeta   # Borrar una carpeta y todo su contenido (usar con precaución)
```

### Editar Archivos desde la Consola (Editor Nano):
Cuando necesitas modificar configuraciones de dominios, puertos o proyectos (por ejemplo, los archivos `.md` o `.yml` guardados en `/srv/shared`), usas el editor de texto integrado llamado **nano**.

```bash
# 1. Ve a la carpeta donde están las configuraciones globales
cd /srv/shared

# 2. Lista los archivos para ver qué puedes editar
ls -la

# 3. Abre el archivo deseado con permisos de administrador
sudo nano proyectos.md
```

**¿Cómo guardar y salir de Nano?**
1. Escribe o modifica el texto moviéndote con las flechas del teclado.
2. Presiona **`Ctrl + O`** para decirle al programa que quieres guardar (Write Out).
3. Presiona **`Enter`** para confirmar el nombre del archivo.
4. Presiona **`Ctrl + X`** para cerrar el editor y volver a la terminal normal.

---

## 🔒 2. Manejo de Permisos (Sudo, Chown y Chmod)

**¿Por qué sucede el error "Permission Denied"?**  
Cuando usas contenedores Docker, procesos como PHP o Nginx pueden generar archivos internos (logs, imágenes) actuando como el superusuario del sistema (`root`). Esto bloquea a otros programas (como Git o tu propio usuario) impidiéndoles modificar esos archivos.

**Solución 1: Devolver la propiedad de los archivos a tu usuario**
```bash
# Entra a la carpeta de tu proyecto
cd ~/projects/MiProyecto

# Hazte dueño absoluto de la carpeta actual (.) y todo lo que contiene (-R)
sudo chown -R $USER:$USER .
```

**Solución 2: Dar permisos de escritura a una carpeta compartida (ej. `/srv/shared`)**  
A veces, un contenedor Docker no puede escribir en una carpeta compartida en el host. Para permitir lectura y escritura pública en esa carpeta específica:
```bash
sudo chmod -R 777 /srv/shared
```

---

## ⬇️ 3. Control de Versiones (Git) en Producción

La forma correcta de actualizar el código de cualquier proyecto web que ya esté funcionando en el VPS es forzar una descarga exacta desde la rama principal (`main`) de GitHub, eliminando cualquier archivo temporal o choque de configuración.

```bash
# 1. Asegúrate de tener permisos en la carpeta del proyecto
sudo chown -R $USER:$USER .

# 2. Obtener los últimos metadatos desde GitHub (sin fusionarlos aún)
git fetch origin

# 3. Forzar al VPS a sobreescribir su código para que sea EXACTO al de GitHub
git reset --hard origin/main
```

---

## 🐳 4. Gestión de Contenedores (Docker & Docker Compose)

Docker es el estándar de la industria para ejecutar bases de datos y servidores web encapsulados. El archivo que los orquesta suele llamarse `docker-compose.yml` o `compose.yml`.

**Reconstruir y Levantar los Servicios (Aplicar Actualizaciones)**  
Útil cuando cambias el `Dockerfile`, archivos de configuración o simplemente cuando bajaste nuevo código de GitHub.
```bash
# Construye nuevas imágenes (si es necesario) y levanta los contenedores en segundo plano (-d)
sudo docker compose -f docker-compose.yml up -d --build
```

**Ver el Estado de los Contenedores**  
```bash
sudo docker ps
```

**Apagar Completamente el Proyecto**  
```bash
sudo docker compose -f docker-compose.yml down
```

**Ver Logs (Registro de Actividades y Errores) en Tiempo Real**  
Si tu servidor web falla, revisa este comando.
```bash
sudo docker compose -f docker-compose.yml logs -f
```

---

## 🛠️ 5. Ejecutar Comandos "Dentro" de un Contenedor

No necesitas instalar `PHP`, `Composer`, `Node.js` o `MySQL` en el propio VPS. Todo está dentro de los contenedores. Utilizas `docker exec` para introducir comandos como si estuvieras dentro de ellos.

**Ejemplo 1: Instalar dependencias de PHP (Composer)**  
```bash
# Si tu contenedor de PHP se llama 'mi_app_php'
docker exec -it mi_app_php composer install
```

**Ejemplo 2: Entrar a la consola de la Base de Datos MySQL/MariaDB**  
```bash
# Si tu contenedor se llama 'mi_db_mysql' y el usuario es 'root'
docker exec -it mi_db_mysql mysql -u root -p
```

---

## ⚙️ La "Receta Mágica" para Actualizar un Proyecto General

Cuando tu equipo de desarrollo te indique: *"Los cambios ya están en GitHub"*, ejecuta siempre este bloque de forma secuencial en la carpeta del proyecto:

```bash
cd ~/ruta/de/tu/proyecto
sudo chown -R $USER:$USER .
git fetch origin
git reset --hard origin/main
sudo docker compose -f compose.yml up -d --build
```
*(Y si el proyecto usa dependencias como Composer o NPM, añadirías al final `docker exec -it <nombre_contenedor> composer install`)*

### Automatización: El Script `deploy.sh`
Para evitar escribir esta serie de comandos repetidamente cada vez que hay una actualización, la mejor práctica es agruparlos en un script de despliegue automático en la raíz del proyecto. 

**Paso 1: Dar permisos de ejecución (solo una vez)**
```bash
chmod +x deploy.sh
```

**Paso 2: Ejecutar actualizaciones rápidamente**
```bash
./deploy.sh
```
El script internamente se encarga de realizar el `git pull` y el comando `docker compose up -d --build` para reconstruir los contenedores con la versión más reciente de la imagen, inyectando el código fresco instantáneamente.

---

## 🌐 6. Enrutamiento y Dominios (Nginx Nativo)

Cuando levantas un proyecto con Docker y expones un puerto (ej. `8892:80`), el proyecto funciona internamente, pero necesitas un **Proxy Inverso** para que responda a un dominio real (`midominio.com`). A veces los servidores usan Nginx Proxy Manager, pero la forma más pura y profesional es usar el **Nginx nativo** instalado en Ubuntu.

### Cómo apuntar un Dominio a tu Contenedor Docker
Toda la configuración de Nginx vive en la carpeta `/etc/nginx/sites-available/`.

**Paso 1: Crear el archivo de configuración**
```bash
# Se recomienda usar .conf para mantener el orden
sudo nano /etc/nginx/sites-available/miproyecto.conf
```

**Paso 2: Bloque de Proxy Inverso Básico**
En el editor, pega esto (cambiando el dominio y el puerto expuesto en docker-compose):
```nginx
server {
    listen 80;
    server_name midominio.com;

    location / {
        proxy_pass http://127.0.0.1:8892; # Puerto expuesto hacia el host
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

**Paso 3: Activar y Aplicar los Cambios**
```bash
# 1. Crear el enlace simbólico para habilitarlo
sudo ln -s /etc/nginx/sites-available/miproyecto.conf /etc/nginx/sites-enabled/

# 2. Verificar que no haya errores de sintaxis
sudo nginx -t

# 3. Aplicar los cambios sin apagar el servidor
sudo systemctl reload nginx
```

---

## 🔒 7. Certificados SSL (HTTPS) y el "Efecto Fallback"

**Problema común:** Ingresas a tu nuevo dominio y te muestra la página web de *otro proyecto/compañero* que está alojado en el mismo VPS. 

**¿Por qué sucede? (El Fallback):**
Esto ocurre cuando visitas tu dominio forzando la conexión segura (`https://` - puerto 443), pero tu archivo `.conf` actual solo tiene configurado el puerto 80 (`listen 80;`). Al Nginx no encontrar configuración segura para tu dominio, entra en pánico y muestra el primer sitio web seguro que encuentre alojado en el servidor por defecto.

**La Solución: Instalar el Certificado SSL con Certbot**
Para adueñarte de tu conexión segura y que Certbot añada las líneas necesarias para el puerto 443, debes ejecutar:

```bash
sudo certbot --nginx -d midominio.com
```

> **Nota:** Si pegas una configuración o comando masivo, asegúrate de no saltarte las preguntas interactivas de Certbot en la terminal. Si ya habías emitido un certificado antes, te preguntará si deseas "Attempt to reinstall this existing certificate" (opción 1) o "Renew & replace" (opción 2). Elige la **opción 1**, y recarga tu web.

---

## 🪲 8. Errores Comunes de Windows a Linux (El "Case Sensitivity")

Uno de los errores más frustrantes al subir un proyecto de XAMPP (Windows) a un VPS (Linux) es el error `Class does not exist` o `File not found`.

**¿Por qué ocurre?**
* **Windows** ignora las mayúsculas: Para Windows, la carpeta `admin` es exactamente igual que `Admin`.
* **Linux (y Docker)** es estricto: Para Linux, `admin` y `Admin` son dos lugares completamente distintos.

Si tu código PHP (`Composer PSR-4`) dice que el namespace es `App\Controllers\Admin` (con **A mayúscula**), pero tu carpeta se llama `admin` (con **a minúscula**), funcionará perfecto en local, pero **fallará en el VPS**.

**La Solución:**
La forma correcta de arreglarlo es renombrar la carpeta para que coincida con el código. Dado que Git en Windows a veces ignora los cambios de mayúsculas, debes hacerlo en dos pasos (o usando una carpeta temporal):
```bash
git mv app/controllers/admin app/controllers/tmp_admin
git mv app/controllers/tmp_admin app/controllers/Admin
git commit -m "fix: mayúsculas en carpeta Admin"
```

---

## 🔄 9. Actualización de Variables (.env) y `docker compose`

Cuando usas versiones modernas de Ubuntu y Docker, el comando clásico `docker-compose` (con guion) ha sido reemplazado por **`docker compose`** (con espacio), ya que ahora viene como un plugin oficial de Docker.

**¿Cómo actualizar contraseñas o el archivo `.env` en producción?**
Recuerda que el archivo `.env` **no se sube a GitHub**, por lo que si cambias una clave de correo localmente, el servidor no se enterará mediante un `git pull`. 

Para actualizar una contraseña en el VPS debes:
1. Editar el archivo directamente en el VPS (usando `nano .env` o comandos como `sed`).
2. **Reiniciar el contenedor** para que el sistema operativo interno lea las nuevas variables:
```bash
sudo docker compose up -d
```
No necesitas `--build` si solo cambiaste el `.env`. Con solo levantarlo de nuevo (`up -d`), Docker inyecta las nuevas claves.
