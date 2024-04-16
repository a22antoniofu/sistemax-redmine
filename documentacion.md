# 1. Introducción

El propósito de esta documentación es documentar el proceso de actualización de Redmine, una plataforma de gestión de proyectos de código abierto, desde la versión 3.4.6 a la versión más reciente, 5.1.2 en un entorno en producción gestionado mediante el orquestador Docker Compose.

# 2. Configuración del entorno

## 2.1 Creación del fichero Docker Compose

Para facilitar el proceso de actualización, se utiliza un archivo "compose.yaml" que se adjunta junto con esta documentación. Este archivo contiene la configuración necesaria para crear los contenedores Docker requeridos para Redmine (depende de un gestor de bases de datos, en este caso Mysql 5.7) y, adicionalmente, Adminer, una herramienta de código abierto que permite gestionar bases de datos Mysql, Sqlite y otras programada en Php mediante una interfaz web.

## 2.2 Definición de variables en el archivo .env

Para simplificar el proceso, se proporcionan varios archivos ".env":

- "adminer.env": Configuración para levantar un contenedor con Adminer, facilitando la inspección y exportación de la base de datos.
- ".env": Configuración para iniciar Redmine en la versión 3.4.6. Será el que de no especificar al desplegar iniciará por defecto.
- "app-updated.env": Configuración para poner en marcha Redmine en su última versión disponible. En este caso, no se pasaría por las versiones intermedias, por lo que habría un mayor riesgo de pérdida de datos.

Estos archivos permiten simplificar el proceso de despliegue, facilitando probar múltiples versiones del programa a desplegar en entornos de pruebas.
Para hacer esto, se debe indicar en el comando de compose la ruta al archivo de variables utilizado (si no se indica, carga por defecto ".env) con la opción "--env-file" antes de la orden de despliegue.

# 3. Actualización de Redmine

Se detallan las actualizaciones realizadas en Redmine, junto con los cambios y consideraciones importantes.

## 3.1. Cómo realizar las actualizaciones

Una vez definidas las variables en el archivo ".env" para iniciar las pruebas se modifica la siguiente:

- IMAGE_VERSION_APP=x.y.z

Donde x, y y z serán los números de la versión a la que se desea actualizar en cada momento.

## 3.2. Actualización a la versión 3.4.13

- **Proceso:** Realizada sin problemas, modificando la variable "app_image_version".
- **Datos:** Se mantienen intactos, conservando la compatibilidad con MySQL 5.7.

## 3.3. Actualización a la versión 4.0.0

- **Proceso:** Sin inconvenientes, ajustando la variable "app_image_version".
- **Datos:** Permanecen sin cambios, manteniendo la compatibilidad con MySQL 5.7.

## 3.4. Actualización a la versión 4.2.0

- **Proceso:** Completada exitosamente, modificando "app_image_version".
- **Datos:** No se alteran, y se mantiene la compatibilidad con MySQL 5.7.

## 3.5.  Actualización a la versión 4.2.10

- **Proceso:** Ejecutada sin contratiempos, ajustando "app_image_version".
- **Datos:** Conservados sin pérdidas, manteniendo la compatibilidad con MySQL 5.7. Se destaca que el tiempo desde que se inician los contenedores hasta que la aplicación web responde  es mayor que en actualizaciones anteriores, posiblemente debido a actualizaciones en la base de datos.

## 3.6. Actualización a la versión 5.0.0

- **Datos:** Mantenidos sin pérdidas y compatibilidad preservada con MySQL 5.7. Es importante destacar que se realiza una actualización de la base de datos, lo que puede aumentar el tiempo de inicio.

## 3.7. Actualización a la versión más reciente ("latest")

- **Proceso:** Realizada satisfactoriamente, sin afectar los datos y manteniendo la compatibilidad con MySQL 5.7.

# 4. Actualización de la base de datos MySQL a la versión 8.0.36

Para realizar este procedimiento, se siguieron las instrucciones del manual de referencia citado en fuentes. Los principales pasos seguidos son:

- Realización de un volcado de la base de datos al anfitrión para evitar pérdidas de datos. Este proceso puede realizarse con herramientas como mysqldump o Adminer.
- Comprobar que el gestor no tiene dependencias que impidan la actualización.
- Realizar un apagado controlado del motor de bases de datos indicando que se va a actualizar.
- Apagado y eliminación del contenedor.

Una vez realizados estos pasos, los datos se preservan sin sufrir cambios, aunque el tiempo de inicio puede aumentar debido a la optimización de la base de datos al iniciar el contenedor de MySQL con la nueva versión.

Es de destacar que de no realizar correctamente el proceso, las tablas pueden corromperse y por consecuencia, el contenedor no arrancará, lo que hace necesario un especial cuidado en entornos de producción para evitar pérdidas y tiempos de inactividad.

# 5. Almacenamiento y copias de seguridad

Dado que el despliegue se está realizando en contenedores (efímeros) se debe tener especial cuidado con los datos que necesitan conservarse. 

Para el manejo de datos conservados en el host y empleados por los contenedores existen dos opciones principales, a saber:

- Empleo de volúmenes gestionados por Docker. Una opción transparente para el usuario, el software busca el mejor lugar para almacenar los datos, gestionando permisos y montándolos en los contenedores.
- Montaje de directorios dentro del contenedor. En este caso, es el usuario quien debe definir el directorio en el host que estará montado en la ruta necesaria dentro del container. Es responsabilidad del usuario gestionar los permisos para que los datos sean accesibles por la aplicación ejecutada en el contenedor.

La opción empleada en este despliegue es la gestión de directorios montados dentro de un contenedor. En el anfitrión, el directorio debe existir y la aplicación dentro del invitado debe tener permisos (por ejemplo, si precisa crear ficheros escritura) para que este enfoque funcione correctamente.

## 5.1. Copias de seguridad de los ficheros

Una consideración particular merecen los ficheros que suben los usuarios a Redmine que se almacenan en la ruta /usr/src/redmine/files dentro del contenedor. 

Al margen de que los archivos se mantienen pese a que los contenedores puedan eliminarse o intercambiarse, una buena opción sería automatizar copias de seguridad con software para tal efecto (por ejemplo Duplicaty o gestión de shell scripting con un Crontab).

## 5.2. Copias de seguridad de las bases de datos

La aplicación desplegada, para su correcto funcionamiento utiliza una base de datos. Es importante mantener copias de la base de datos, pese a que un cambio de contenedor o actualización no debiera afectarla.

Aquí, la opción más simple puede ser realizar un volcado con mysqldump, que se puede automatizar mediante un Crontab.

# 6. Fuentes

- Consideraciones a la hora de realizar el proceso de actualización de Redmine (en su wiki): https://www.redmine.org/projects/redmine/wiki/redmineupgrade
- Imagen oficial de Redmine para Docker: https://hub.docker.com/_/redmine/
- La persistencia de datos en Docker: https://learn.microsoft.com/es-es/virtualization/windowscontainers/manage-containers/persistent-storage
- Imagen oficial de Mysql para Docker: https://hub.docker.com/_/mysql
- MySQL :: Manual de referencia de MySQL 8.3 :: 3 Actualización de MySQL: https://dev.mysql.com/doc/refman/8.3/en/upgrading.html
- Implementación de Mysql en contenedores Docker (manual oficial): https://dev.mysql.com/doc/refman/8.3/en/docker-mysql-getting-started.html#docker-upgrading
- MySQL :: MySQL 8.3 Manual de referencia :: 3.6 Preparando su instalación para la actualización: https://dev.mysql.com/doc/refman/8.3/en/upgrade-prerequisites.html
- MySQL :: Manual de referencia de MySQL 8.3 :: Solución de problemas de actualización 3.13: https://dev.mysql.com/doc/refman/8.3/en/upgrade-troubleshooting.html
- MySQL :: MySQL 8.3 Manual de referencia :: 3.12 Actualización de una instalación Docker de MySQL: https://dev.mysql.com/doc/refman/8.3/en/upgrade-docker-mysql.html
