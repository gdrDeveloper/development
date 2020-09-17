# Herramientas útiles para el desarrollo
En el presente documento explicare como usar estos docker-compose para un ambiente de desarrollo.
> **Nota: No me hago responsable del mal uso de estas tecnologías o las pérdidas de datos que se pudieran ocasionar por su uso.**
> **Deben tener presente que para alguna de ellas se deben contar con las respectivas licencias de docker hub o en caso de usarlas de forma productivas las correspondientes licencias de sus fabricantes si así hiciera falta.**

El repositorio está organizado según las tecnologías a utilizar. Cada directorio cuenta con un archivo docker-compose que es el que se encarga de crear los contenedores con las imágenes que he configurado.
> **Todas las imágenes son las oficiales sin cambios de ningún tipo y las últimas al momento de crear los docker-compose.**

En la carpeta "Librerias" se encuentra el driver para poder usar la base de datos oracle 12c. En caso de agregar otras bases de datos, los drivers se agregarán en esta carpeta.

Dentro de la carpeta "Directorios" se encuentran agrupadas las distintas tecnologías.
> **Glosario:**
>  - EL:    **Elastisearch**
>  - KB:    **Kibana**
>  - FB:    **Filebeat**
>  - NF:    **Apache Nifi**
>  - NR:    **Apache Nifi Registry**

El ejemplo más simple es el de Apache Nifi
```
version: '3.8'
services:
  nifi1114:
    image: 'apache/nifi:1.11.4'
    container_name: nifi1114
    hostname: nifi1114
    user: root
    ports:
      - "8084:8084"
    environment:
      - NIFI_WEB_HTTP_PORT=8084
      - NIFI_ELECTION_MAX_WAIT=1 min
      - TZ=America/Argentina/Buenos_Aires
      - LANG=es_AR.UTF-8
      - NIFI_JVM_HEAP_MAX=2048m      
    volumes:
      - ../../Librerias/:/home/nifi/
      - ./conf/:/opt/nifi/nifi-1.11.4/conf/    
      - ./content_repository/:/opt/nifi/nifi-1.11.4/content_repository/
      - ./database_repository/:/opt/nifi/nifi-1.11.4/database_repository/
      - ./flowfile_repository/:/opt/nifi/nifi-1.11.4/flowfile_repository/
      - ./provenance_repository/:/opt/nifi/nifi-1.11.4/provenance_repository/
      - ./logs/:/opt/nifi/nifi-1.11.4/logs/
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 2048M
    logging:
      driver: "json-file"
      options:
        max-size: 100m
        max-file: "3"
```

Todos los archivos docker-compose están organizados de la misma forma y configurados para usar la zona horaria de Argentina, por lo que si no es tu zona horaria podrás cambiarla modificando la **TZ** por la que desees utilizar.
Podrás ver una lista [aqui](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)

Solo he cambiado los puertos de Apache Nifi, Apache Nifi Registry y el de Oracle 12c por una cuestión de uso. Los puertos por defecto son el 8080, el 18080 y 1521 respectivamente que deberás actualizarlos en caso de querer usarlos.
También configure el uso de CPU y Memoria, en caso de no contar con mucha memoria ram, deberás cambiar el valor de **memory** por uno acorde a tus requerimientos, **el valor mínimo es de 512 MB**.

Dentro de **volumes**, los path que están a la izquierda representan los directorios que tendrás en el host (son los directorios que has clonado), y los de la derecha son los propios de las imágenes. Estos se separan por los dos puntos (:).
```
host_path:image_path
```

> Para el caso de la imágen de Apache Nifi, si decidieras usar otra versión de la misma, es necesario que actualices la carpeta **conf** por la respectiva carpeta que se encuentra dentro del archivo comprimido de descarga de la versión que intentas utilizar. Dicho de otro modo, tendrás primero que descargar el archivo comprimido de la [página oficial de apache nifi](https://nifi.apache.org/download.html) y luego copiar los archivos que se encuentran en la carpeta **conf** del archivo comprimido dentro del directorio **conf** que has clonado. Esto se debe a que la imágen no realiza este proceso y si intentas ejecutar el contenedor creado sin estos archivos dentro de la carpeta **conf** el mismo te dará un error.

> Para el caso de la imágen de Oracle 12c, la misma cuenta con problemas de persistencia de datos. Si bien los directorios estan mapeados y son creados al momento de la primera ejecución, si por algún motivo tienes que detener la imágen y luego volver a iniciarla, el contenedor no interpretará que ha sido creado antes y dará un error de que no puede encontrar cierta carpeta. De todas formas, si cuentas con los script de creación tanto de usuarios como de esquema de datos (y un export de los datos) podrás volver a crear el contenedor borrando el contenido del directorio **data**.

> **Este problema está en la lista de pendientes a resolver**

# Modo de uso
## Modo de Ejecución
### Windows
Es necesario contar con el programa [Docker Desktop para windows](https://docs.docker.com/docker-for-windows/install/) para poder ejecutar docker en windows. Al mismo tiempo deberás contar con una consola de comandos, te recomiendo que uses [git bash](https://git-scm.com/downloads) dado que podras usarla también para la gestión de tus repositorios git.
### Linux
Es necesario instalar [Docker Engine](https://docs.docker.com/engine/install/ubuntu/) para poder ejecutar docker en linux y luego instalar docker-compose.
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.25.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose version
```
### MAC
No tuve la oportunidad de probar el sistema operativo MAC por lo que en lo único que puedo ayudarte es en compartirte esta [guia oficial](https://docs.docker.com/docker-for-mac/install/).


Para poder ejecutar los contenedores deberás usar el siguiente comando posicionado en la carpeta que desees:

> Por ejemplo, si solo quisieras utilizar el contenedor de Apache Nifi, deberás posicionarte en ~\development\Docker\Directorios\Nifi-1114

- Sin ver los logs
```
docker-compose --compatibility up -d
```
- Viendo los logs
```
docker-compose --compatibility up
```

## URLs
- Apache Nifi: http://localhost:8084/nifi/
- Apache Nifi Registry: http://localhost:18084/nifi-registry
- Elasticsearch: http://localhost:9200/
- Kibana: http://localhost:5601/
## String de conexión
- Host: localhost
- Port: 1525
- SID: ora12cdk
- Usuario: sys
- Contraseña: Oradoc_db1

### Script para la creacion de un usuario
Deberás actualizar **USUARIO**  por el nombre que quieras dar a tu usuario y **CONTRASENA**  por la contraseña para el mismo.
```
ALTER SESSION SET "_ORACLE_SCRIPT"=true;
CREATE USER USUARIO IDENTIFIED BY CONTRASENA;
ALTER USER USUARIO DEFAULT TABLESPACE users QUOTA UNLIMITED ON users;
ALTER USER USUARIO TEMPORARY TABLESPACE temp;
GRANT CREATE SESSION, CREATE VIEW, ALTER SESSION, CREATE SEQUENCE TO USUARIO;
GRANT CREATE SYNONYM, CREATE DATABASE LINK, RESOURCE , UNLIMITED TABLESPACE TO USUARIO;
COMMIT;
```

## Lista de pendientes:
- [ ] Validar comportamiento de filebeat para el patrón y path de logs configurados
- [ ] Corregir dentro del contenedor de Oracle 12C la pérdida del vínculo entre los path configurados
- [ ] Incorporar git a Apache Nifi Registry
