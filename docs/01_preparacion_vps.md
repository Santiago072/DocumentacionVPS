# Fase 1: Preparación del VPS y Clonación

Esta guía es genérica y aplica para cualquier sistema web (como MVCs en PHP) que se vaya a desplegar en un VPS usando Docker.

## 1. Clonación del Repositorio
Al descargar el código fuente en el servidor (Ubuntu/Linux), es crucial asegurarse de que los archivos tengan los permisos del usuario del sistema y no de `root`. De lo contrario, Docker y el sistema operativo tendrán conflictos de escritura.

```bash
git clone https://github.com/TuUsuario/TuRepositorio.git
cd TuRepositorio
sudo chown -R $USER:$USER .
```

## 2. Configuración de Variables de Entorno (.env)
Los sistemas profesionales usan variables de entorno para no exponer contraseñas en el código fuente. Se debe crear un archivo `.env` en la raíz del proyecto.

**Ejemplo de creación de archivo `.env`:**
```bash
cat > .env << 'EOF'
DB_HOST=nombre_contenedor_db
DB_USER=usuario_bd
DB_PASS=ClaveSegura123
DB_NAME=nombre_bd
MYSQL_ROOT_PASSWORD=ClaveRootSegura!

APP_BASE=/
SESSION_LIFETIME=3600
COOKIE_SECURE=1
EOF
```

## 3. Script de Despliegue (deploy.sh)
Es altamente recomendado tener un script que automatice la obtención de cambios y la reconstrucción de los contenedores. 

**Ejemplo de un script de despliegue (`deploy.sh`):**
```bash
#!/bin/bash
echo "Ajustando permisos locales..."
sudo chown -R $USER:$USER .

echo "Sincronizando con GitHub..."
git fetch origin
git reset --hard origin/main

echo "Levantando contenedores..."
docker compose up -d --build
echo "Despliegue completado."
```

Para ejecutar este script de forma segura sin preocuparse por permisos de ejecución:
```bash
bash deploy.sh
```

**¿Por qué usar `--build`?**
En producción, el código fuente no se lee en tiempo real desde la carpeta por motivos de seguridad. La instrucción `--build` obliga a Docker a tomar una "fotografía" exacta de los archivos recién descargados, construir una imagen inmutable y reemplazar el contenedor viejo.
