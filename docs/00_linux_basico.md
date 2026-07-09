# Fase 0: Navegación Básica y Permisos en Linux

## 1. Navegación Básica y Carpetas Comunes
En un servidor Linux, las ubicaciones donde guardas los archivos son fundamentales.

* **/home/usuario/ o ~/**: Es tu carpeta personal. Aquí sueles guardar los proyectos que estás desarrollando o clonando desde GitHub (ej. ~/projects/MiProyecto).
* **/srv/ o /var/www/**: Carpetas reservadas tradicionalmente para datos de servicios web. Si usas Docker con volúmenes persistentes, es muy común crear carpetas como /srv/shared/ para guardar archivos que no se deben borrar al reiniciar contenedores (imágenes, PDFs subidos por usuarios, backups, etc.).
* **/tmp/**: Carpeta para archivos temporales que se borran automáticamente al reiniciar el VPS.

### Comandos de Navegación y Gestión de Carpetas:
\\\ash
cd /ruta/hacia/carpeta  # Entrar a una carpeta
cd ..                   # Subir un nivel
ls -la                  # Listar todos los archivos y carpetas mostrando detalles y permisos
mkdir nombre_carpeta    # Crear una nueva carpeta
rm -rf nombre_carpeta   # Borrar una carpeta y todo su contenido
\\\

### Editar Archivos desde la Consola (Editor Nano):
Cuando necesitas modificar configuraciones de dominios, puertos o proyectos usas el editor de texto integrado llamado **nano**.
\\\ash
cd /srv/shared
sudo nano proyectos.md
\\\

**¿Cómo guardar y salir de Nano?**
1. Escribe o modifica el texto moviéndote con las flechas del teclado.
2. Presiona **Ctrl + O** para guardar (Write Out).
3. Presiona **Enter** para confirmar el nombre del archivo.
4. Presiona **Ctrl + X** para cerrar el editor.

## 2. Manejo de Permisos (Sudo, Chown y Chmod)
**¿Por qué sucede el error "Permission Denied"?**
Cuando usas contenedores Docker, procesos como PHP o Nginx pueden generar archivos internos actuando como el superusuario del sistema (oot). Esto bloquea a otros programas impidiéndoles modificar esos archivos.

**Solución 1: Devolver la propiedad de los archivos a tu usuario**
\\\ash
cd ~/projects/MiProyecto
sudo chown -R $USER:$USER .
\\\

**Solución 2: Dar permisos de escritura a una carpeta compartida**
\\\ash
sudo chmod -R 777 /srv/shared
\\\
"@ -Encoding UTF8

Set-Content -Path docs/01_preparacion_vps.md -Value @"
# Fase 1: Preparación del VPS, Clonación y Control de Versiones

## 1. Clonación del Repositorio
Al descargar el código fuente en el servidor (Ubuntu/Linux), asegúrate de que los archivos tengan los permisos del usuario del sistema.
\\\ash
git clone https://github.com/TuUsuario/TuRepositorio.git
cd TuRepositorio
sudo chown -R $USER:$USER .
\\\

## 2. Control de Versiones (Git) en Producción
La forma correcta de actualizar el código de cualquier proyecto web que ya esté funcionando en el VPS es forzar una descarga exacta desde la rama principal (main) de GitHub, eliminando cualquier archivo temporal o choque de configuración.

\\\ash
sudo chown -R $USER:$USER .
git fetch origin
git reset --hard origin/main
\\\

## 3. Configuración y Actualización de Variables (.env)
Los sistemas profesionales usan variables de entorno para no exponer contraseñas en el código fuente. Se debe crear un archivo .env en la raíz del proyecto.

**Crear archivo .env inicial:**
\\\ash
cat > .env << 'EOF'
DB_HOST=nombre_contenedor_db
DB_USER=usuario_bd
DB_PASS=ClaveSegura123
DB_NAME=nombre_bd
EOF
\\\

**¿Cómo actualizar contraseñas o el archivo .env en producción?**
Recuerda que el archivo .env no se sube a GitHub. Si cambias una clave localmente, el servidor no se enterará mediante un git pull. 
1. Edita el archivo en el VPS (
ano .env).
2. Reinicia el contenedor para inyectar los cambios:
\\\ash
sudo docker compose up -d
\\\
*(No necesitas --build si solo cambiaste el .env).*

## 4. Automatización: El Script deploy.sh
Para evitar escribir esta serie de comandos repetidamente, agruparlos en un script de despliegue automático es la mejor práctica.

**Ejemplo de deploy.sh:**
\\\ash
#!/bin/bash
sudo chown -R $USER:$USER .
git fetch origin
git reset --hard origin/main
docker compose up -d --build
\\\
Ejecútalo de forma segura usando: ash deploy.sh. La bandera --build descarta el contenedor viejo, construye uno fresco leyendo los archivos del disco y lo levanta garantizando una imagen limpia.
