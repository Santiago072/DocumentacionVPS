# Documentación y Comandos para Servidores VPS (Linux / Ubuntu) 🚀

Este repositorio centraliza las guías, comandos y buenas prácticas para la administración de un Servidor Virtual Privado (VPS). La información está diseñada de forma general para gestionar proyectos web modernos basados en arquitectura **MVC**, **Docker**, **Nginx** y repositorios **Git**.

## 📑 Índice de Documentación Estructurada

La documentación ha sido refactorizada e integrada en archivos modulares:

1. **[Fase 0: Navegación Básica y Permisos](docs/00_linux_basico.md)**
   * Navegación por carpetas de Linux y uso de Nano.
   * Manejo de usuarios, permisos (\chown\, \chmod\) y errores *Permission Denied*.

2. **[Fase 1: Preparación del VPS y Git](docs/01_preparacion_vps.md)**
   * Clonación de repositorios y descargas forzadas (git reset --hard).
   * Creación, actualización e inyección segura de variables \.env\.
   * Estructura y propósito de \deploy.sh\.

3. **[Fase 2: Docker, Base de Datos y Transferencias](docs/02_docker_y_bd.md)**
   * Orquestación de contenedores y ejecución de comandos internos (\docker exec\).
   * Transferencias directas (\scp\) para evitar subir datos sensibles a GitHub.
   * Importación de bases de datos, prevención de caracteres corruptos y solución al *Modo Estricto* de SQL.

4. **[Fase 3: Proxy Inverso y SSL (Nginx)](docs/03_nginx_y_ssl.md)**
   * Configuración pura de bloques Nginx nativo.
   * Enrutamiento de dominios al puerto interno de Docker.
   * Certificados SSL automatizados (Let's Encrypt) y prevención del *Efecto Fallback*.

5. **[Fase 4: Buenas Prácticas y Errores Típicos](docs/04_buenas_practicas_mvc.md)**
   * Solución al *Case Sensitivity* (diferencias de mayúsculas entre Windows y Linux).
   * Estructura estándar obligatoria para proyectos MVC y cómo se conecta con la persistencia en el VPS.

---
*Mantenido y documentado por Santiago072.*
