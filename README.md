# Laravel con Docker

## Descargar Laradock

1. Clonar el repositorio:

    ```
    git clone https://github.com/Laradock/laradock.git
    ```

2. Copiar el fichero `env-example` a `.env`:

    Windows

    ```
    cd laradock & copy env-example .env
    ```

3. Editar el fichero `.env` de la carpeta `laradock`:

    - Modificar la ruta de la aplicación para que apunte a la carpeta `laradock` poniendo `APP_CODE_PATH_HOST=../laradock`
    - Si disponemos de más de una instalación de Laradock, modificar la variable `COMPOSE_PROJECT_NAME` y asignarle un nombre único para que los contenedores tengan nombres diferentes.
    - Seleccionar la versión de PHP: `PHP_VERSION=7.4`
    - Modificar el driver de base de datos de phpMyAdmin: `PMA_DB_ENGINE=mariadb`

## Nuevo proyecto de Laravel

(Opcional) Reemplazar `app` por el nombre del proyecto a partir de aquí.

### Generar el proyecto

> :warning: En estos comandos, si se ha renombrado app, cambiar solo la última ocurrencia, después de laravel/laravel.

Linux y macOS

```
docker run -it --rm --name php-cli \
    -v composer_cache:/home/docker/.composer/cache \
    -v "$PWD:/usr/src/app" thecodingmachine/php:7.4-v3-slim-cli \
    composer create-project --prefer-dist laravel/laravel app
```

Windows

```
docker run -it --rm --name php-cli ^
    -v composer_cache:/home/docker/.composer/cache ^
    -v "%CD%:/usr/src/app" thecodingmachine/php:7.4-v3-slim-cli ^
    composer create-project --prefer-dist laravel/laravel app
```

### Configurar un nuevo sitio en Laradock

1. Ir a `laradock/nginx/sites` y duplicar `laravel.conf.example` a `app.conf`.

2. Modificar en el fichero `app.conf` estas dos líneas, cambiado `laravel` por el nombre del proyecto:

    ```
    server_name app.test;
    root /var/www/app/public;
    ```

### Editar fichero de hosts

Añadir a `/etc/hosts` (en Windows `C:\Windows\System32\Drivers\etc\hosts`):

```
127.0.0.1	app.test
```

### (Re)arrancar los contenedores

Los comandos de `docker-compose` se lanzan en la carpeta `laradock`.

Arrancar los contenedores necesarios:

```
docker-compose up -d nginx mariadb phpmyadmin workspace
```

Y para reiniciar un contenedor concreto:

```
docker-compose restart nginx
```

### Crear la base de datos

1. Acceder a [phpMyAdmin](http://localhost:8081)

    - Servidor `mariadb` y usuario `root/root`.
    - Crear la base de datos `app` y el usuario `app/app`.

2. Editar el .env de la aplicación

    ```
    DB_CONNECTION=mysql
    DB_HOST=mariadb
    DB_PORT=3306
    DB_DATABASE=app
    DB_USERNAME=app
    DB_PASSWORD=app
    ```

#### Parche para Windows

La base de datos no arranca correctamente debido al sistema de ficheros en que corre Docker. Para solucionarlo, editar el fichero `laradock/docker-compose.yml` y modificar:

1. En la sección `volumes`, alrededor de la línea 24, añadir un volumen nuevo para alojar los datos de mariadb:

    ```yml
      mariadb_data:
        driver: ${VOLUMES_DRIVER}
    ```

2. En la sección correspondiente a `mariadb`, alrededor de la línea 390, editar la sección `volumes` y reemplazar:

    ```yml
    ### MariaDB ##############################################
        ...
          volumes:
            - ${DATA_PATH_HOST}/mariadb:/var/lib/mysql
        ...
    ```

    Por:

    ```yml
    ### MariaDB ##############################################
        ...
          volumes:
            - mariadb_data:/var/lib/mysql
        ...
    ```

3. Reiniciar los contenedores.

### Acceder al sitio web

Página principal: http://app.test

<a href="https://ibb.co/nC7xkRg"><img src="https://i.ibb.co/jRyn4H6/Captura-de-pantalla-2020-11-15-a-las-11-21-17.png" alt="Captura-de-pantalla-2020-11-15-a-las-11-21-17" border="0"></a>

> Si en Windows da un error de permiso denegado, entrar al workspace (ver siguiente sección) y lanzar el comando: `chown -R laradock:laradock /var/www`.
> Para hacer el comando anterior, hay que meterse dentro del workspace, eso se hace con el primer comando en la sección de utilidades, tambien tendréis que meteros en la carpeta del proyecto, en el ejemplo se usa app.

## Utilidades

### Lanzar comandos en el proyecto (composer, artisan, npm...)

```
docker-compose exec workspace /bin/bash

cd app
```

```
Aqui poner los comandos del block de notas
```

### Añadir soporte para fechas en castellano

Editar el fichero `.env` de laradock y activar la opción `PHP_FPM_INSTALL_ADDITIONAL_LOCALES=true`.

En la variable `PHP_FPM_ADDITIONAL_LOCALES` escribir la lista de idiomas adicionales, como por ejemplo `es_ES.UTF-8` para castellano.

```
Información sacade de: https://gist.github.com/ijaureguialzo/bf10504c742b44122ba62bfafe772c1c
```

## Todos esos son los comandos para crear un proyecto desde 0, pero y si quiero seguir con un proyecto ya empezado, de otra persona.

Lo que tenemos que hacer es lo siguiente:

### Configurar un nuevo sitio en Laradock

Ir a `laradock/nginx/sites` y duplicar `laravel.conf.example` a `app.conf`.

Modificar en el fichero `app.conf` estas dos líneas, cambiado `laravel` por el nombre del proyecto:

    ```
    server_name app.test;
    root /var/www/app/public;
    ```

### Crear la base de datos

Acceder a [phpMyAdmin](http://localhost:8081)

    - Servidor `mariadb` y usuario `root/root`.
    - Crear la base de datos `app` y el usuario `app/app`.

### Generar el proyecto con el comando:

Linux y macOS

```
docker run -it --rm --name php-cli \
    -v composer_cache:/home/docker/.composer/cache \
    -v "$PWD:/usr/src/app" thecodingmachine/php:7.4-v3-slim-cli \
    composer create-project --prefer-dist laravel/laravel app
```

Windows

```
docker run -it --rm --name php-cli ^
    -v composer_cache:/home/docker/.composer/cache ^
    -v "%CD%:/usr/src/app" thecodingmachine/php:7.4-v3-slim-cli ^
    composer create-project --prefer-dist laravel/laravel app
```

### Ir al directorio del proyecto y clonar el proyecto allí.

```
    cd app

    Clonar la ruta en el directorio del proyecto
    
    Editar el .env de la aplicación

    DB_CONNECTION=mysql
    DB_HOST=mariadb
    DB_PORT=3306
    DB_DATABASE=app
    DB_USERNAME=app
    DB_PASSWORD=app
 ´´´
 
 ### Ejecutar este comando en la carpeta donde tienes instalado laradock
 
 ´´´
    docker-compose exec workspace /bin/bash
    cd app
    composer install
    php artisan key:generate
```
> Tambien tienes que tener modificado el fichero de hosts.

Ya debería de estar funcionando.
