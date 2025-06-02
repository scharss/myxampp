# Proyecto PHP con Docker y MySQL

Este proyecto proporciona una configuración de Docker lista para usar para desarrollar y ejecutar aplicaciones PHP con una base de datos MySQL. Incluye un servidor web Nginx, PHP-FPM y un servicio MySQL, junto con PhpMyAdmin para una fácil gestión de la base de datos.

## Estructura del Proyecto

```
miproyecto/
├── docker-compose.yml  # Define los servicios de Docker (PHP, Nginx, MySQL, PhpMyAdmin)
├── php/
│   ├── Dockerfile      # Para personalizar PHP (instalar extensiones, Composer, etc.)
│   └── php.ini         # Configuraciones PHP personalizadas (upload_max_filesize, etc.)
└── src/                # Aquí va tu código PHP (tu aplicación)
    └── index.php       # Ejemplo inicial o punto de entrada de tu aplicación
    └── composer.json   # Si usas Composer para tus dependencias PHP
```

## Requisitos Previos

*   Docker Desktop instalado en tu sistema.

## Cómo Empezar

1.  **Clona o descarga este proyecto.**
2.  **Coloca tu código PHP existente en el directorio `src/`.** Si estás comenzando un nuevo proyecto, puedes empezar a crear tus archivos PHP dentro de esta carpeta.
3.  **Configura las variables de entorno (opcional):**
    *   Puedes modificar el archivo `docker-compose.yml` para cambiar las contraseñas de MySQL, los puertos expuestos u otras configuraciones según tus necesidades. Busca las secciones `environment` en los servicios `db` y `phpmyadmin`.

## Uso

### 1. Construir e Iniciar los Contenedores

Abre una terminal en la raíz del proyecto (donde se encuentra el archivo `docker-compose.yml`) y ejecuta el siguiente comando:

```bash
docker-compose up -d --build
```

*   `up`: Crea e inicia los contenedores.
*   `-d`: Ejecuta los contenedores en segundo plano (modo detached).
*   `--build`: Construye las imágenes antes de iniciar los contenedores (útil si has modificado el `Dockerfile` de PHP).

### 2. Acceder a tu Aplicación

Una vez que los contenedores estén en funcionamiento:

*   **Tu aplicación PHP:** Abre tu navegador y ve a `http://localhost:8080` (o el puerto que hayas configurado para el servicio `web` en `docker-compose.yml`).
*   **PhpMyAdmin:** Abre tu navegador y ve a `http://localhost:8081` (o el puerto que hayas configurado para `phpmyadmin`). Utiliza las credenciales de MySQL definidas en `docker-compose.yml` para iniciar sesión. Por defecto (si no has cambiado nada), el servidor es `db`, el usuario es `root` y la contraseña es la que hayas puesto en `MYSQL_ROOT_PASSWORD`.

### 3. Conectar PHP a MySQL

Dentro de tu código PHP en el directorio `src/`, puedes conectarte a la base de datos MySQL utilizando los siguientes detalles:

*   **Host:** `db` (este es el nombre del servicio MySQL definido en `docker-compose.yml`)
*   **Nombre de la base de datos:** La que hayas definido en `MYSQL_DATABASE` en `docker-compose.yml` (por ejemplo, `mydatabase`).
*   **Usuario:** El que hayas definido en `MYSQL_USER` en `docker-compose.yml` (por ejemplo, `myuser`).
*   **Contraseña:** La que hayas definido en `MYSQL_PASSWORD` en `docker-compose.yml`.

**Ejemplo de conexión a base de datos con .env**

```
DB_SERVER=db
DB_USERNAME=mi_usuario_db
DB_PASSWORD=root
DB_NAME=mi_base_de_datos
```

Coloca este código (o similar) en un archivo `.env` dentro del directorio `src/miporyecto` para probar la conexión a Mysql.

## Comandos Útiles de Docker

*   **Ver los logs de los contenedores:**
    ```bash
    docker-compose logs -f web  # Logs del contenedor web (Nginx/PHP)
    docker-compose logs -f db   # Logs del contenedor MySQL
    ```
    (Reemplaza `web` o `db` con el nombre del servicio que quieras inspeccionar).

*   **Ejecutar comandos dentro de un contenedor:**
    Esto es útil para usar Composer, ejecutar migraciones de bases de datos, o acceder a la consola de un framework PHP.
    ```bash
    docker-compose exec web bash
    ```
    Esto te dará una sesión de shell dentro del contenedor `web`. Una vez dentro, puedes ejecutar comandos como:
    ```bash
    # composer install
    # composer update
    # exit (para salir del contenedor)
    ```

*   **Detener los contenedores:**
    ```bash
    docker-compose down
    ```

*   **Detener y eliminar los volúmenes (borrará los datos de MySQL):**
    **¡ADVERTENCIA!** Este comando eliminará permanentemente los datos de tu base de datos MySQL si están almacenados en un volumen gestionado por Docker.
    ```bash
    docker-compose down -v
    ```

*   **Reconstruir e iniciar (si has hecho cambios en Dockerfile o docker-compose.yml):**
    ```bash
    docker-compose up -d --build
    ```

## Personalización

*   **PHP:** Modifica `php/Dockerfile` para instalar extensiones PHP adicionales (ej. `gd`, `zip`, `intl`) o herramientas como Composer. Ajusta `php/php.ini` para configuraciones específicas de PHP (ej. `upload_max_filesize`, `memory_limit`). Después de realizar cambios, recuerda reconstruir las imágenes con `docker-compose up -d --build`.
*   **Nginx:** La configuración de Nginx se encuentra dentro de la imagen de PHP (o puedes montar una configuración personalizada si es necesario). Para la mayoría de los casos, la configuración por defecto debería funcionar.
*   **MySQL:** Puedes cambiar la versión de MySQL, contraseñas, y nombres de bases de datos directamente en el archivo `docker-compose.yml`.


## Para desarrollar con Worpdrees y poder instalar plugins, seguir esto:
* Definir FS_METHOD en wp-config.php (La más recomendada para Docker)
* Esta es la forma más limpia y segura para entornos Docker. Le dices a WordPress que intente escribir directamente en el sistema de archivos, asumiendo que los permisos son correctos.
Edita tu archivo wp-config.php: Este archivo se encuentra en la raíz de tu instalación de WordPress (dentro de tu carpeta src/ que está montada en /var/www/html/).
Añade la siguiente línea en alguna parte del archivo, preferiblemente antes de la línea /* That's all, stop editing! Happy publishing. */:
```
define('FS_METHOD', 'direct');
```
* Guarda el archivo wp-config.php.
Asegúrate de los permisos de archivo: Después de añadir esta línea, necesitas asegurarte de que el usuario con el que se ejecuta Apache/PHP (generalmente www-data dentro del contenedor) tenga permisos de escritura sobre las carpetas wp-content/plugins/ y wp-content/upgrade/ (y a veces wp-content/uploads/ para otras operaciones).
Dentro del contenedor (método preferido para ajustar permisos si FS_METHOD es direct):
Entra al contenedor web:

```
docker-compose exec web bash
```

* Esto te dará una línea de comandos dentro del contenedor web. Verás un prompt diferente, algo como root@mi_app_web_avanzado:/var/www/html# o www-data@mi_app_web_avanzado:/var/www/html$.
Navega al directorio raíz de WordPress dentro del contenedor (si no estás ya ahí):
El prompt ya debería indicarte que estás en /var/www/html, que es donde está montada tu carpeta src/. Si no, ejecuta:
```
cd /var/www/html
```
* Cambia el propietario de las carpetas relevantes a www-data:
Ejecuta los siguientes comandos uno por uno:

```
chown -R www-data:www-data wp-content/plugins
chown -R www-data:www-data wp-content/upgrade
chown -R www-data:www-data wp-content/themes
chown -R www-data:www-data wp-content/uploads 
```
* Si alguna de estas carpetas no existe aún (por ejemplo, upgrade o uploads a veces se crean la primera vez que se necesitan), el comando simplemente no hará nada para esa carpeta específica, lo cual está bien. La más importante para instalar plugins es wp-content/plugins.

  
* chown cambia el propietario.
-R significa recursivo (aplica el cambio a la carpeta y todo su contenido).
www-data:www-data establece el usuario www-data y el grupo www-data.
Ajusta los permisos de las carpetas para que el propietario (www-data) pueda escribir:
Ejecuta los siguientes comandos uno por uno:

```
chmod -R 755 wp-content/plugins
chmod -R 755 wp-content/upgrade
chmod -R 755 wp-content/themes
chmod -R 755 wp-content/uploads
```
* chmod cambia los permisos.
-R es recursivo.
755 significa:
Propietario (www-data): Leer, Escribir, Ejecutar (rwx)
Grupo (www-data): Leer, Ejecutar (r-x)
Otros: Leer, Ejecutar (r-x)
Para archivos dentro de estas carpetas, WordPress a veces necesita 644. Si los comandos anteriores para los directorios no son suficientes, podrías necesitar algo más granular, pero 755 para los directorios suele ser el primer paso.
Sal del contenedor:
Escribe exit y presiona Enter.
```
exit
```
¡Feliz desarrollo!















