# Grupo 3 — Base de Datos

## Proyecto: servidor MariaDB con Alpine Linux y Docker

Implementación de un servicio de base de datos MariaDB en una máquina virtual Alpine Linux. El proyecto cubre el despliegue del contenedor, la persistencia de datos, la administración web con Adminer y las pruebas de conexión desde otros equipos de la red.

**Materia:** Sistemas Operativos  
**Institución:** Universidad Franz Tamayo (UNIFRANZ), sede Cochabamba  
**Docente:** Ing. Diego Cárdenas

## Equipo y roles

| Integrante | Rol | Responsabilidades |
|---|---|---|
| **Juan Pablo Coca Soliz** | Responsable de documentación | Organizar el repositorio, recopilar evidencias y documentar las pruebas y configuración. |
| **Gabriel Ortega** | Operador técnico | Desplegar los contenedores Docker y configurar volúmenes, puertos y red en Alpine Linux. |
| **Evelin Torrico** | Coordinadora | Organizar las tareas, dar seguimiento al tiempo y coordinar las pruebas de inserción remota. |
| **Leandro Gonzales Mur** | Analista e investigador | Investigar la optimización de memoria y analizar las pruebas de conexión y persistencia. |

**Responsabilidad compartida:** todo el equipo participa en la continuidad del servicio, la bitácora de incidentes y la validación de los cambios.

## Objetivos

- Desplegar MariaDB en Alpine Linux mediante Docker.
- Crear y estructurar la base de datos `soporte_ti_db` y la tabla `incidencias_ti`.
- Mantener los datos al reiniciar el contenedor mediante un volumen Docker.
- Permitir la administración y las pruebas desde otros equipos con Adminer.
- Registrar las decisiones técnicas, incidencias, soluciones y evidencias del proyecto.

## Arquitectura del proyecto

- **Sistema operativo:** Alpine Linux en una máquina virtual sobre Proxmox VE.
- **Motor de base de datos:** MariaDB, ejecutado en Docker.
- **Base de datos:** `soporte_ti_db`.
- **Tabla principal:** `incidencias_ti`.
- **Persistencia:** volumen Docker `datos_mariadb`, montado en `/var/lib/mysql`.
- **Administración web:** Adminer, publicado en el puerto `8081`.
- **Conexión MariaDB:** puerto `3306`.
- **Optimización para recursos limitados:** InnoDB buffer pool configurado en 64 MB.

> Las direcciones IP del laboratorio pueden cambiar. Consulta la configuración vigente de la máquina virtual antes de conectarte. No publiques contraseñas, tokens ni direcciones privadas de VPN.

## Despliegue resumido

Crea el volumen persistente:

```sh
docker volume create datos_mariadb
```

Inicia MariaDB y reemplaza los valores de ejemplo por credenciales propias:

```sh
docker run -d \
  --name servidor-mariadb \
  -p 3306:3306 \
  -v datos_mariadb:/var/lib/mysql \
  -e MARIADB_ROOT_PASSWORD='<PASSWORD_ROOT>' \
  -e MARIADB_DATABASE=soporte_ti_db \
  -e MARIADB_USER=equipo_ti \
  -e MARIADB_PASSWORD='<PASSWORD_EQUIPO>' \
  mariadb:lts \
  --innodb-buffer-pool-size=64M
```

Verifica el estado del contenedor:

```sh
docker ps
```

Adminer se puede desplegar en el mismo host Docker para administrar la base de datos desde un navegador:

```sh
docker run -d \
  --name interfaz-db \
  --link servidor-mariadb:db \
  -p 8081:8080 \
  adminer
```

Abre `http://<IP_DEL_SERVIDOR>:8081` y conecta con el servidor `db`, el usuario `equipo_ti` y la base de datos `soporte_ti_db`.

## Esquema de datos

La tabla `incidencias_ti` registra el integrante, su rol, la actividad realizada, el estado y la fecha de registro. Su estructura incluye:

| Campo | Descripción |
|---|---|
| `id` | Identificador incremental y clave primaria. |
| `integrante` | Nombre de la persona registrada. |
| `rol` | Rol asignado en el equipo. |
| `actividad` | Tarea o actividad documentada. |
| `estado` | Estado del registro; por defecto, `Completado`. |
| `fecha_registro` | Fecha y hora de creación automática. |

## Validaciones realizadas

- Inicio del contenedor MariaDB y publicación del puerto 3306.
- Creación de la base de datos, la tabla y los registros iniciales.
- Acceso remoto a través de Adminer desde equipos del laboratorio.
- Inserción y consulta de registros desde distintos equipos.
- Persistencia de los datos después de reiniciar el contenedor.
- Verificación desde la consola del servidor de los registros añadidos remotamente.

Después de un reinicio, espera a que MariaDB termine de iniciar antes de ejecutar consultas. Una consulta inmediata puede fallar mientras el servicio prepara su socket.

## Evidencias y documentación

El informe principal del proyecto contiene el procedimiento detallado, la bitácora de incidencias y las capturas de las pruebas. Agrega las evidencias al repositorio con nombres descriptivos y sin incluir credenciales reales.

## Continuidad del servicio

Como siguientes pasos se propone automatizar copias de seguridad con `mariadb-dump` y documentar el despliegue con un archivo `docker-compose.yml`.
