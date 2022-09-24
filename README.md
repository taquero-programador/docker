### Crear una macvlan.
Permite crear un puente a la puerta de enlace, y asignar a cada container una ip desde el router.  
Desde docker
```bash
docker network create -d macvlan \
--subnet=192.168.0.0/24 \
--gatewaye=192.168.0.1 \
-o parent=driver_equipo nombre_red
```
En cada archivo compose se añade así:
```yml
networks:
  nombre_red:
    ipv4_address: ip_range

networks:
  nombre_red:
    external: true
```
Crear directamete desde el compose.
```yml
# asignar a cada container una ip
networks:
  nombre_red:
    ipv4_address: ip_range

networks:
  nombre_red:
    driver: macvlan
    driver_opts:
      parent: driver_equipo
    ipam:
      config:
        - subnet: "192.168.0.0/24"
        - gatewaye: "192.168.0.1"
```

### Crear una red docker para que todos los container se conecten y se puedan ver, usando el nombre de servicio o el container_name.
```bash
docker network create my_net
```
Añadir al final de cada archivo compose.
```yml
networks:
  default:
    name: my_net
    external: true
```
Ahora se pueden comunicar usando el nombre de servicio y en la misma red.

### Variables de entorno.
Se puede crear un archivo `.env` en el mismo lugar donde esta el archivo compose, en cada variable de entorno de compose el valor sera ${VARIABLE}.
```bash
# insatalar el docker compose plugin
sudo apt install docker-compose-plugin
# validar el resultado sin construir el container
docker compose convert
```
Las variables de entorno del shell tienen prioridad sobre los archivos `.env`.  

El archivo se puede colocar en otro directorio diferente al de compose, para mandarlo llamar se usa `env_file:  - ./path/.env`, el archivo tambien puede cambiar de nombre; `.env.test, .env.prod`.
```yml
# quitar enviroment y los valores debajo
# despues de image
env_file:
  - /path/env.tesd
```
Por cada servicio debe existir un archivo `.env`.

### Crear multiple base de datos al levantar un container.
Crear un directorio en la raíz del proyecto junto a compose, dentro un archivo `.sql`.
```bash
mdkir init && touch init/01.sql
```
Dentro del archivo añadir las reglas de creación y permisos de usuario.
```sql
CREATE DATABASE IF NOT EXISTS `name_databse`;
GRANT ALL ON `name_databse`.* TO 'name_user'@'%';
```
En volumes añadir los siguiente:
```yml
volumes:
  - ./init/:/docker-entrypoint-initdb.d
```
Al levantar el archivo compose, se crea la base definida por defecto y las que estan dentro del archivo `.sql`.
```bash
# o acceder al container y ejecutar el proceso manualmente
docker exec -it mariadb bash
mysql -p
```

### Validar que un container se puede comunicar con otro.
```bash
docker exec -it container1 ping container2
```
