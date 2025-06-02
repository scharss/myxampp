# ENTORNO DE DESARROLLO PHP MYQSL CON DOCKER


miproyecto/
├── docker-compose.yml
├── php/
│   ├── Dockerfile         # Para personalizar PHP (instalar extensiones, Composer, etc.)
│   └── php.ini            # Configuraciones PHP personalizadas (upload_max_filesize, etc.)
└── src/                   # Aquí irá tu código PHP (tu aplicación)
    └── index.php          # Ejemplo inicial
    └── composer.json      # Si usas Composer para tus dependencias PHP




Accede a tu aplicación: Abre tu navegador y ve a http://localhost:8080.
Accede a phpMyAdmin: Abre http://localhost:8081


docker-compose logs -f web
docker-compose logs -f db


Para ejecutar comandos dentro de un contenedor (ej. Composer, consola de un framework):
docker-compose exec web bash  # Entra a la shell del contenedor web
# Dentro del contenedor:
# composer install
# php artisan --version (si es Laravel)
# exit



docker-compose down

docker-compose down -v

docker-compose up -d --build















