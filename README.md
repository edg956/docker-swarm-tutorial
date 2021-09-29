# Cómo crear tu propio cluster de Docker Swarm
## Montar docker-machine para una máquina linux
- Configurar host remoto
	- Crear usuario y asignarle contraseña
		```bash
		sudo useradd <user>
		sudo passwd <user>
		```
	- Añadir usuario a `/etc/sudoers`
		- Ejecutar `sudo visudo`
		- Añadir la siguiente línea: `<user>	ALL=(ALL) NOPASSWD: ALL`

- Configurar credenciales en host local
	- Crear llaves y copiar llave pública a host remoto
		```bash
		ssh-keygen <tu configuración de confianza>
		ssh-copy-id -i <llave_rsa publica> <user>@<domain> # user ha de ser el mismo que el configurado arriba
		```

- Provisionar máquinas con `docker-machine`
	- Ejecutar `docker-machine` para crear el comando
		```bash
		docker-machine create --driver generic \
			--generic-ip-address=<ip or domain> \
			--generic-ssh-key <llave_rsa privada> \
			--generic-ssh-user <user> \		     # user ha de ser el configurado para el host remoto
			<machine>
		```

- Por seguridad, realizar un backup cifrado de las credenciales creadas por `docker-machine`
	- Ir a la carpeta de las credenciales de docker machine
		```bash
		cd ~/.docker/
		tar -cf machine.tar machine/
		gzip machine.tar
		gpg -c machine.tar.gz 		# Pedirá una contraseña. Anótala por si no la recuerdas al necesitarlo.
		```

Notas:
- Este ejemplo de `docker-machine` está orientado para usuarios que ya tengan una máquina lista para provisionar con docker, etc. Puedes dirigirte a la documentación oficial para ver que otras opciones existen.
- `docker-machine` puede pisar la configuración de docker del sistema systemd.
La documentación de docker hace referencia al directorio en /etc/systemd/system/docker.service.d/,
pero también puedes encontrar la unidad de docker en /lib/systemd/system/docker.service
https://docs.docker.com/machine/drivers/generic/.
- docker-machine no reconoce Raspbian en una raspberry, así que hay que cambiar temporalmente el ID del sistema operativo en /etc/os-release para engañar a docker-machine.
- Hay que asegurarse de que el usuario que se haya creado para docker-machine tenga su propio directorio en /home y sea de su propiedad.

## Crear docker swarm con un manager y añadir workers
- Crear manager
	- `docker-machine ssh \<manager-machine\> docker swarm init --advertise-addr \<ip|inet\>[:\<port\>]`
		- Este comando imprimirá el token para unir workers al swarm
- Añadir workers
	- `docker-machine ssh \<worker-machine\> docker swarm join --token \<token-paso-anterior\> \<ip|addr manager\>:\<port\>`

## Desplegar servicios (stacks)
- Configurar entorno para controlar manager
	- `eval $(docker-machine env \<manager-machine\>)`
- Desplegar servicios
	- `docker stack deploy -c \<stack-configuration\>.yml \<stack-name\>`

# TODO:
- Añadir plugins
- Añadir otros métodos alternativos a docker-machine para gestionar el clúster (¿ansible?)
- Ampliar el scope de esta guía para explicar componentes de interés básicos en un cluster (monitorización, enrutamiento con traefik, interfaz gráfica de gestión como Portainer o Swarmkit)
- Añadir ejemplos de docker-machine con proveedores cloud
- Extras?
