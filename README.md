## Docker

### Crear token para Vaultwarden

    echo -n 'password' | argon2 "$(openssl rand -base64 32)" -e -id -k 19456 -t 2 -p 1

Ese `token` siempre lo va a pedir en la página del admin.

### Crear un token y salt para navidrome API Homepage
```sh
SALT=$(openssl rand -hex 8)
PASSWORD="pass"

TOKEN=$(echo -n "${PASSWORD}${SALT}" | md5sum | cut -d' ' -f1)

echo $SALT
echo $TOKEN
```

Probar un requests de la url:
`http://your-server/rest/ping.view?u=joe&t=26719a1196d2a940705a59634eb18eab&s=c19b2d&v=1.12.0&c=myapp`

Debe retornar un status `ok`.

### Crear y usar una red existente
`docker network create --driver bridge {network_name}`

Desde compose usar la red existente:
```yaml
networks:
  hub: # este solo es un alias y también debe estar declarado arriba
    external: true
    name: network_name # nombre real de la red
```

O hacer que docker la cree directamente al levantar el compose:
```yaml
networks:
  hub: # alias
    name: network_name
    driver: bridge
```

### Levantar contenedor Docker+Tailscale (HTTPS+MagicDNS)
- Requiere obtener una APIkey y un `tag:container` (elimina la expiración de autenticación).
- Archivo `config/{servicio}.json` para hacer el reverse proxy con Tailscale.

Archivo de ejemplo en `tailscale_navidrome/docker-compose.yaml`

### Usar certificados con Caddy
Para servicios que no aceptan HTTPS plano y requieren autenticación por certificado.
- Caddy requiere acceso al sock de Tailscale
- `Caddyfile` para hacer reverse proxy

Pasar el formato correcto a caddy:
```sh
docker exec caddy_vault caddy fmt --overwrite /etc/caddy/Caddyfile
# hacer un reaload
docker exec caddy_vault caddy reload --config /etc/caddy/Caddyfile
```

### Crear una macvlan
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

### Variables de entorno
Se puede crear un archivo `.env` en el mismo lugar donde esta el archivo compose, en cada variable de entorno de compose el valor sera ${VARIABLE}.
```bash
# insatalar el docker compose plugin
sudo apt install docker-compose-plugin
# validar el resultado sin construir el container
docker compose convert
```
Las variables de entorno del shell tienen prioridad sobre los archivos `.env`.

Ver los logs es un contenedor en ejecución.
```bash
# usando docker. el comando se puede ejecutar desde cualquier lugar/directorio.
docker logs nombre_container
# usando el plugin. el comando se ejecuta en el directorio del proyecto
docker compose logs
```

El archivo se puede colocar en otro directorio diferente al de compose, para mandarlo llamar se usa `env_file:  - ./path/.env`, el archivo tambien puede cambiar de nombre; `.env.test, .env.prod`.
```yml
# quitar enviroment y los valores debajo
# despues de image
env_file:
  - /path/env.tesd
```
Por cada servicio debe existir un archivo `.env`.

### Crear multiple base de datos al levantar un container
Crear un directorio en la raíz del proyecto junto a compose, dentro un archivo `.sql`.
```bash
mdkir init && touch init/01.sql
```

Dentro del archivo añadir las reglas de creación y permisos de usuario.
```sql
CREATE DATABASE IF NOT EXISTS `name_databse`;
GRANT ALL ON `name_databse`.* TO 'name_user'@'%';
```

Caso adicional para crear user, db y asignarle permisos.
```bash
# crear password para mysql
mysqladmin --user=root --password "pass"
# phpmyadmin no soporta el inicio con root, crear un nuevo user
CREATE USER 'phpmyadmin'@'localhost' IDENTIFIED BY 'password';
# permisos de super admin (acceso general)
GRANT ALL PRIVILEGES ON *.* TO 'phpmyadmin'@'localhost' WITH GRANT OPTION;

# crear DB y usuario con permisos root solo para esa DB
CREATE DATABASE 'docker_test';
# user para esa DB
CREATE USER 'userdockertest'@'localhost' IDENTIFIED BY 'passuserdocker';
# permiso root solo para esa DB
GRANT ALL ON 'docker_test'.* TO 'userdockertest'@'localhost' IDENTIFIED BY 'passuserdocker' WITH GRANT OPTION;
FLUSH PRIVILEGES;
EXIT;
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

### Validar que un container se puede comunicar con otro
```bash
docker exec -it container1 ping container2
```
