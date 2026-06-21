# Documentación y Comandos del VPS (Sistema Sodicol)

Este repositorio contiene la guía definitiva de administración y comandos útiles para el Mantenimiento del Servidor Virtual Privado (VPS) donde se encuentra alojado el **Sistema Sodicol**.

Dado que el servidor utiliza una arquitectura moderna basada en contenedores (**Docker**), los comandos aquí listados están orientados al flujo de trabajo exacto de nuestro despliegue en producción.

---

## 📁 1. Ubicación del Proyecto

Toda la administración del código y los contenedores debe realizarse situándose primero en la carpeta raíz del proyecto en el servidor:

```bash
# Ingresar a la carpeta principal del proyecto
cd ~/projects/SistemaSodicol
```

---

## 🔒 2. Solución a Problemas de Permisos (Carpetas Bloqueadas)

**¿Por qué sucede?**  
Al ejecutarse PHP y Apache dentro de un contenedor Docker, el sistema asume temporalmente permisos de "Administrador" (root/www-data) para escribir archivos internos (como los logs de errores). Esto bloquea a Git al momento de querer actualizar el código, arrojando errores como `Permission denied`.

**Solución:**  
Ejecutar el siguiente comando estando dentro de `~/projects/SistemaSodicol` para devolverle la propiedad de los archivos al usuario del servidor:

```bash
sudo chown -R $USER:$USER .
```
*(Es probable que te solicite la contraseña de tu usuario 'santiago')*

---

## ⬇️ 3. Actualizar el Código Fuente desde GitHub

Una vez que se han programado nuevos cambios y se han subido a la rama `main` en GitHub, los pasos exactos para descargar esa actualización limpia al VPS son:

```bash
# 1. Asegurarse de tener los permisos correctos (ver sección 2)
sudo chown -R $USER:$USER .

# 2. Descargar la información más reciente de GitHub sin aplicarla todavía
git fetch origin

# 3. Forzar al VPS a que su código sea una copia EXACTA de lo que hay en GitHub
git reset --hard origin/main
```

---

## 🐳 4. Gestión de Contenedores (Docker Compose)

El servidor levanta la base de datos (MySQL) y la aplicación web (Apache/PHP) usando Docker.

**Reconstruir y Levantar el Servidor (Aplicar Cambios)**  
Si se modificó alguna configuración estructural (como el `Dockerfile`, el `php.ini` o se requiere reiniciar totalmente el servicio PHP para borrar la caché):

```bash
sudo docker compose -f docker/docker-compose.yml up -d --build
```

**Apagar el Servidor por Completo**  
```bash
sudo docker compose -f docker/docker-compose.yml down
```

**Ver Logs en Tiempo Real (Para diagnosticar caídas críticas)**  
```bash
sudo docker compose -f docker/docker-compose.yml logs -f
```

---

## 📦 5. Descargar/Actualizar Bibliotecas de PHP (Composer)

Dado que la generación de PDFs usa la librería **DomPDF**, la cual se gestiona a través de Composer, si alguna actualización requiere reinstalar o descargar nuevas librerías, se debe ejecutar este comando. 

*(Ojo: Esto no se ejecuta directamente en el VPS, sino dándole la orden "desde afuera" al contenedor llamado `sodicol_app`)*:

```bash
docker exec -it sodicol_app composer install
```

---

## 🗄️ 6. Acceso Directo a la Base de Datos

Si necesitas ejecutar un comando SQL manualmente (como crear índices o revisar una tabla), puedes inyectar comandos directamente al contenedor de la base de datos (`sodicol_mysql`):

```bash
# Ingresar a la consola interactiva de MySQL
docker exec -it sodicol_mysql mysql -u sodicol -p sistema_sodicol

# (El sistema te pedirá la contraseña de la base de datos definida en el .env)
```

---

## ⚙️ Resumen del Flujo Completo de Actualización Normal

Cuando te digan: *"Los cambios ya están listos en GitHub, actualiza el VPS"*, esta es la "receta mágica" que debes copiar y pegar (línea por línea):

```bash
cd ~/projects/SistemaSodicol
sudo chown -R $USER:$USER .
git fetch origin
git reset --hard origin/main
sudo docker compose -f docker/docker-compose.yml up -d --build
docker exec -it sodicol_app composer install
```

Con esto, garantizas que los permisos, el código, el servidor y las bibliotecas queden funcionando de manera impecable y sincronizada.
