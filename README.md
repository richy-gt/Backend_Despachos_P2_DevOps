Sistema de gestión de ventas y despachos compuesto por dos APIs REST (Spring Boot) y un frontend React, orquestados con Docker Compose.

---

## Estructura del proyecto

```
/
├── Springboot-API-REST/          # Backend de Ventas (puerto 8080)
│   └── Dockerfile
├── Springboot-API-REST-DESPACHO/ # Backend de Despachos (puerto 8081)
│   └── Dockerfile
├── frontend/                     # Frontend React + Vite (puerto 80)
│   ├── Dockerfile
│   └── nginx.conf
├── docker-compose.yml
├── init-db.sql
├── .env.example
└── README.md
```


## Justificación técnica

Para la persistencia de datos de MySQL se eligió un **Named Volume** (`mysql_data`) en lugar de un Bind Mount, por las siguientes razones técnicas:

### ¿Por qué Named Volume?

1. **Portabilidad:** Un named volume es gestionado completamente por Docker y no depende de la estructura de directorios del sistema operativo host. Esto significa que el `docker-compose.yml` funciona igual en Linux, macOS y Windows sin modificaciones.

2. **Rendimiento en I/O:** Los named volumes tienen mejor rendimiento de lectura/escritura que los bind mounts en sistemas macOS y Windows, ya que Docker los gestiona en su propio sistema de archivos interno, evitando la capa de traducción entre el host y el contenedor.

3. **Seguridad:** Al no exponer una ruta del sistema de archivos del host, se reduce la superficie de ataque. Un bind mount podría permitir al contenedor acceder a archivos sensibles del host si no se configura correctamente.

4. **Gestión del ciclo de vida:** Docker gestiona automáticamente el ciclo de vida del volumen. Se puede inspeccionar, respaldar y restaurar de forma independiente al contenedor con comandos estándar (`docker volume inspect`, `docker volume ls`).

5. **Persistencia real:** Los datos sobreviven a `docker-compose down`, `docker-compose up --build` y recreaciones de contenedores. Solo se eliminan con `docker-compose down -v` (explícitamente).

### ¿Cuándo usar Bind Mount en su lugar?

Un bind mount sería más apropiado para **archivos de configuración** (como `init-db.sql`) o para **código fuente en desarrollo** donde se requiere hot-reload, ya que permite mapear directamente un directorio del host al contenedor. En este proyecto se usa bind mount de solo lectura (`:ro`) para el script SQL de inicialización, lo que es una práctica segura y apropiada para ese caso de uso.

---

## Instrucciones de despliegue

### Prerrequisitos

- Docker Engine 24+
- Docker Compose v2.x

### 1. Clonar y configurar variables de entorno

```bash
clonar todos los repos a una carpeta unica!
git clone <url-del-repositorio>
cd <nombre-del-proyecto>

cp .env.example .env
nano .env
```

### 2. Construir e iniciar todos los servicios

```bash
docker-compose up --build -d

docker-compose logs -f

docker-compose ps
```

### 3. Verificar que todo funciona

```bash
curl http://localhost:80/health

curl http://localhost:8080/api/v1/ventas

curl http://localhost:8081/api/v1/despachos
```

### 4. Apagar los servicios

```bash
docker-compose down

docker-compose down -v
```

---

## Gestión del volumen de base de datos

```bash

docker volume inspect <nombre_proyecto>_mysql_data


docker volume ls

docker exec mysql-db mysqldump -u root -p<password> ventas_db > backup_ventas.sql
docker exec mysql-db mysqldump -u root -p<password> despacho_db > backup_despachos.sql
```

---

## Seguridad implementada

Todos los Dockerfiles implementan las siguientes buenas prácticas:

- **Multi-stage build:** La imagen final no contiene herramientas de compilación (Maven, npm, etc.), reduciendo drásticamente la superficie de ataque.
- **Usuario no root:** Todos los servicios se ejecutan con usuarios sin privilegios (`appuser` en Java, `nginx` en el frontend).
- **Imágenes Alpine/ligeras:** Se usan variantes Alpine para minimizar el tamaño de la imagen y la cantidad de paquetes instalados.
- **Limpieza de capas:** Se eliminan cachés de paquetes (`apk cache`, `npm cache`, `.m2/repository`) en la misma instrucción `RUN` para no añadir tamaño a las capas de Docker.
- **Health checks:** Todos los servicios tienen health checks definidos para que Docker Compose gestione el orden de arranque de forma confiable.
