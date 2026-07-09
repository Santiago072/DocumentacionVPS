# Documentación de Despliegue VPS 🚀

Este repositorio contiene las guías técnicas estandarizadas y generalizadas para desplegar aplicaciones web (especialmente arquitecturas MVC, PHP y MySQL/MariaDB) en Servidores Privados Virtuales (VPS) de producción usando contenedores Docker.

## Índice de Documentación

La información ha sido modularizada para facilitar su lectura y consulta técnica:

1. **[Fase 1: Preparación del VPS y Entorno](docs/01_preparacion_vps.md)**
   - Clonación y control de permisos (`chown`).
   - Configuración de variables de entorno `.env`.
   - Lógica y ejecución de scripts de despliegue (`deploy.sh`).

2. **[Fase 2: Docker, Base de Datos y Transferencias](docs/02_docker_y_bd.md)**
   - Orquestación con `docker-compose`.
   - Transferencia segura de archivos vía SCP.
   - Importación de datos (semillas de base de datos) previniendo problemas de codificación (UTF-8).
   - Manejo de Volúmenes Docker para archivos subidos.
   - Solución a errores de *Modo Estricto* en bases de datos.

3. **[Fase 3: Proxy Inverso y SSL (Nginx)](docs/03_nginx_y_ssl.md)**
   - Instalación de Nginx y Certbot.
   - Creación de bloques de servidor para redirigir tráfico al puerto de Docker.
   - Implementación de Certificados SSL seguros mediante Let's Encrypt.

---
*Mantenido por Santiago072.*
