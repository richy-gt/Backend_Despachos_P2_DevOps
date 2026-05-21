## Instrucciones de Ejecución Local

Para levantar este servicio localmente con su base de datos aislada:

1. Clonar este repositorio.
2. Posicionarse en la carpeta raíz del proyecto.
3. Ejecutar el siguiente comando para construir y levantar los contenedores en segundo plano:
   `docker compose up -d --build`

Una vez iniciado, la API estará disponible en `http://localhost:8081` (puedes verificar la documentación Swagger en `/swagger-ui.html`).

## Justificación Técnica de Volúmenes (Persistencia de Datos)

Para asegurar la persistencia de la base de datos de este microservicio, se ha elegido utilizar **Volúmenes Nombrados (Named Volumes)** gestionados por Docker, en lugar de *Bind Mounts*. 

**Razón de la elección:**
Los Named Volumes son la mejor práctica para bases de datos en entornos de producción y servidores en la nube como AWS EC2. Al estar completamente administrados por Docker, se aíslan del sistema de archivos del host, previniendo problemas de permisos de lectura/escritura cruzados entre el contenedor y el sistema operativo de la instancia. Esto garantiza que si el contenedor de la base de datos se destruye o reinicia, la información crítica de despachos se mantenga íntegra y disponible para el nuevo contenedor.