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
