# Fase 4: Buenas Prácticas, Errores y Estructura MVC

## 1. Errores Comunes de Windows a Linux (Case Sensitivity)
Uno de los errores más frustrantes al subir un proyecto de XAMPP (Windows) a un VPS (Linux) es el error \Class does not exist\ o \File not found\.

**¿Por qué ocurre?**
* **Windows** ignora las mayúsculas: La carpeta \dmin\ es exactamente igual que \Admin\.
* **Linux (y Docker)** es estricto: Para Linux, \dmin\ y \Admin\ son dos lugares completamente distintos.

Si tu namespace PHP es \App\Controllers\Admin\, pero tu carpeta es \dmin\ (minúscula), funcionará perfecto en local, pero **fallará en el VPS**.

**La Solución:** Renombrar la carpeta en dos pasos usando Git:
\\\ash
git mv app/controllers/admin app/controllers/tmp_admin
git mv app/controllers/tmp_admin app/controllers/Admin
git commit -m "fix: mayúsculas en carpeta Admin"
\\\

## 2. Estructura Típica de un Proyecto (Patrón MVC)
Para estandarizar el desarrollo y facilitar la administración en el VPS, todos nuestros proyectos siguen una arquitectura estricta:

\\\	ext
MiProyecto/
├── app/                     # Carpeta principal del backend protegido (MVC)
│   ├── controllers/         # Lógica de negocio (recibe peticiones y llama al modelo)
│   ├── models/              # Consultas directas a la base de datos MySQL (PDO/MySQLi)
│   ├── services/            # Servicios reutilizables (Manejo de archivos, Emails)
│   └── views/               # Archivos HTML/PHP con el diseño visual de las páginas
├── config/                  # Configuraciones globales (.env, conexion.php)
├── public/                  # Archivos expuestos públicamente a internet (CSS, JS, IMG)
├── uploads/                 # Archivos subidos por los usuarios
├── logs/                    # Registro de errores de PHP
├── deploy.sh                # Script oficial de despliegue automatizado
├── docker-compose.yml       # Orquestador de contenedores
├── BD.txt                   # Volcado SQL inicial de la base de datos
└── index.php                # Front Controller: Punto de entrada único del sistema
\\\

### ¿Por qué esta estructura es clave en el VPS?
1. **Seguridad:** Todas las peticiones web apuntan a \index.php\. El código crítico está encerrado en \pp/\ y no es accesible.
2. **Persistencia (Volúmenes de Docker):** Las carpetas \uploads/\ y \logs/\ se mapean como volúmenes persistentes. Al actualizar (./deploy.sh), no se pierden imágenes ni registros.
3. **Escalabilidad:** Separar \public/\ del backend permite servir estáticos sin invocar a PHP.
