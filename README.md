### Crear una macvlan.
Permite crear una puente a la puerta de enlace, y asignar a cada container una ip desde el router.  
Desde docker
```bash
docker network create -d macvlan \
--subnet=192.168.0.0/24 \
--gatewaye=192.168.0.1 \
-o parent=driver_equipo nombre__red
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

### Crear una red docker para que todos los container se conecten y se pueden ver, usando el nombre de servicio o el container_name.
```bash
docker network create my_net
```
Añadir al final de cada archivo compose.
```yml
networks:
  default:
    external:
      name: my_net
```
Ahora se pueden comunicar usando el nombre de servicio y en la misma red.
