SISPROCON_UG2
=============

Descripcion tecnica
-------------------
Aplicacion web en PHP con arquitectura MVC simple. El flujo principal es un
frontend PHP (controladores, modelos y vistas) que consume servicios externos
para autenticacion y para ejecucion de procedimientos almacenados en SQL Server.

Nombre del proyecto
-------------------
SISPROCON_UG2

Objetivo del modulo
-------------------
Gestion y captura de propuestas mediante formularios (Formulario 1 a 7), con
validaciones en servidor y persistencia a traves de procedimientos almacenados.

Arquitectura general
--------------------
- MVC monolitico en PHP (carpeta `app/` con controllers, models, views).
- Enrutamiento por parametros GET `?c=` (controlador) y `?m=` (metodo).
- Consumo de APIs externas para autenticacion y acceso a datos.

Estructura de carpetas
----------------------
```
SISPROCON_UG2/
  app/
    controllers/
    models/
    views/
      layout/
  config/
  core/
  public/
    assets/
      css/
      img/
      js/
  vendor/
  composer.json
  composer.lock
  README.md
```

Tecnologias utilizadas
----------------------
- Lenguaje: PHP 8.3
- Framework: MVC propio (no framework externo).
- Dependencias: `vlucas/phpdotenv` (carga de variables de entorno).
- Base de datos: SQL Server (via procedimientos almacenados, consumidos por API).
- Servidor web: Apache (public en `public/`).

Instrucciones de despliegue
---------------------------
Requisitos del entorno
- PHP con extensiones: `curl`, `pdo_sqlsrv` (o driver SQL Server si se usa acceso directo).
- Composer instalado.
- Servidor web (Apache recomendado 2.4).
- Acceso a las APIs externas configuradas en variables de entorno.

Pasos para instalacion
1) Clonar o copiar el proyecto en el document root del servidor web.
2) Ejecutar `composer install` en la raiz del proyecto.
3) Crear el archivo `.env` en la raiz con las variables requeridas.
4) Ajustar `config/app.php` para la `base_url` si cambia la ruta de public.
5) Verificar que `public/` sea el entrypoint del servidor web.

Comandos de ejecucion
- En entornos con Apache: levantar el servicio y navegar a la URL configurada.
- En desarrollo con PHP built-in (opcional):
  - `php -S localhost:8000 -t public`

Configuracion de variables de entorno
-------------------------------------
Crear `.env` en la raiz con:

API_BASE_URL=...
API_DB_BASE_URL=...
API_ADMIN_USER=...
API_ADMIN_PASS=...
API_DB_USER=...
API_DB_PASS=...

Notas:
- `WEB_URL` y `API_DB_METODO` estan definidos en `config/constants.php` y
  pueden necesitar ajuste si cambia el host.

APIs y servicios
----------------
Este proyecto consume servicios externos. Los endpoints se derivan de las
variables de entorno en `config/constants.php`.

1) Autenticacion / Sesion (API_BASE_URL)
   - Endpoint: `/Sesion/Login`
   - Metodo HTTP: POST
   - Headers: `Authorization: Bearer <token>`
   - Body (JSON):
     - ip
     - username
     - password
     - tipoUsuario
   - Respuesta esperada: JSON con `respuesta` y datos de usuario.

2) Token de aplicacion (API_BASE_URL)
   - Endpoint: `/GeneraToken/UsuarioToken`
   - Metodo HTTP: POST
   - Body (JSON):
     - usuario
     - clave
   - Respuesta esperada: JSON con `result` (token).

3) Datos academicos (API_BASE_URL)
   - Endpoint: `/ConsultaDatos/GetFacultades`
     - Metodo HTTP: POST
     - Body (JSON): usuario, token
   - Endpoint: `/ConsultaDatos/GetCarreras`
     - Metodo HTTP: POST
     - Body (JSON): codFacultad, usuario, token
   - Endpoint: `/ConsultaDatos/GetDocentes`
     - Metodo HTTP: POST
     - Body (JSON): codFacultad, codCarrera, usuario, token

4) API BD (API_DB_BASE_URL)
   - Endpoint: `/Auth/login`
     - Metodo HTTP: POST
     - Body (JSON): aplicacion, usuario, clave
     - Respuesta esperada: JSON con `token`
   - Endpoint: `/db/metodo` (configurado en `API_DB_METODO`)
     - Metodo HTTP: POST
     - Body (JSON):
       - conexion
       - sentencia (nombre SP)
       - parametros (@Transaccion, @Xml)

Endpoints del sistema (web)
---------------------------
Los endpoints se acceden por querystring:
- Login: `/?c=auth&m=index` (vista)
- Login POST: `/?c=auth&m=login` (procesa credenciales)
- Home: `/?c=auth&m=home` (requiere sesion)
- Logout: `/?c=auth&m=logout`

Metodos HTTP
------------
- Vistas: GET
- Autenticacion: POST en `/?c=auth&m=login`
- Las acciones de formularios se exponen como endpoints en sus respectivos
  controladores (ver `app/controllers/`).

Parametros y respuestas
-----------------------
- Los formularios manejan payloads JSON enviados por fetch/XHR.
- Las respuestas de controllers retornan JSON con `success`/`Exito` y `Mensaje`.
- Para detalles de payloads ver los modelos en `app/models/`.

Autenticacion
-------------
- El sistema usa sesion PHP (`$_SESSION`) para mantener el usuario autenticado.
- Las rutas publicas estan definidas en `public/index.php`.

