# Cómo montar docker-machine para una máquina linux

- Configurar host remoto
	- Crear usuario y asignarle contraseña
		- sudo useradd \<user\>
		- sudo passwd \<user\>
	- Añadir usuario a sudoers
		- \<user\>	ALL=(ALL) NOPASSWD: ALL

- Configurar credenciales en host local
	- Crear llaves y copiar llave pública a host remoto
		- ssh-keygen
		- ssh-copy-id -i \<llave_rsa publica\> \<user\>@\<domain\> # user ha de ser el mismo que el configurado arriba

- Crear maquina con docker-machine
	- Ejecutar docker-machine para crear el comando
		- docker-machine create --driver generic \
			--generic-ip-address=\<ip or domain\> \
			--generic-ssh-key <llave_rsa privada> \
			--generic-ssh-user \<user\> \		     # user ha de ser el configurado para el host remoto
			\<machine\>

- Asegurar las credenciales
	- Ir a la carpeta de las credenciales de docker machine
		- cd ~/.docker/
	- Crear archivo, comprimirlo y encriptarlo
		- tar -cf machine.tar machine/
		- gzip machine.tar
		- gpg -c machine.tar.gz 		# Pedirá una contraseña. Anótala por si no la recuerdas al necesitarlo.

Notas:

- docker-machine puede pisar la configuración de docker del sistema systemd.
La documentación de docker hace referencia al directorio en /etc/systemd/system/docker.service.d/,
pero también puedes encontrar la unidad de docker en /lib/systemd/system/docker.service
https://docs.docker.com/machine/drivers/generic/

- docker-machine no reconoce Raspbian en una raspberry, así que hay que cambiar temporalmente el ID del sistema operativo en /etc/os-release para engañar a docker-machine

# Crear docker swarm con un manager y un worker
- Crear manager
	- docker-machine ssh \<manager-machine\> docker swarm init --advertise-addr \<ip|inet\>[:\<port\>]
		- Este comando imprimirá el token para unir workers al swarm
- Añadir workers
	- docker-machine ssh \<worker-machine\> docker swarm join --token \<token-paso-anterior\> \<ip|addr manager\>:\<port\>

# Desplegar servicios (stacks)
- Configurar entorno para controlar manager
	- eval $(docker-machine env \<manager-machine\>)
- Desplegar servicios
	- docker stack deploy -c \<stack-configuration\>.yml \<stack-name\>


Última modificación 04 de abril, 2021
