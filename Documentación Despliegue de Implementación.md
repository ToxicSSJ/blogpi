## **Proyecto 2**

Documentación Tecnica.

 - Abraham Miguel Lora Vargas (amlorav@eafit.edu.co)
 - Mario Alejandro Muñetón Durango mamunetond@eafit.edu.co

Grupo 001
Universidad EAFIT
Departamento de informática y sistema
ST0263-001 Tópicos especiales en telemática
*Medellín*

---

Los requisitos de la implementación (prueba piloto) se debe contar con lo siguiente:

- 3 maquinas cloud que corran con un sistema Linux (Preferiblemente Ubuntu 20.04)
- 1 Cuenta de Cloudflare
- 1 Cuenta de GCP con creditos

Pasos para la instalación:

 1. Instalar docker en las 3 maquinas, para ello se debe rejecutar la siguiente serie de comandos en cada una de las 3 maquinas:
	 - `sudo apt update`
	 - sudo apt install apt-transport-https ca-certificates curl software-properties-common`
	 - `curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -`
	 - `sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"`
	 - `sudo apt-get update`
	 - `sudo apt install docker-ce`
	 - `systemctl status docker`

2. Inicializar el Manager en la maquina que se destinará para dicho rol:
	- `sudo docker swarm init --advertise-addr <ip>`
	- Copiar el token generado e introducirlo en los workers (ver paso 3).

3. Inicializar y enlazar los dos workers:
	- `sudo docker swarm join --token <token> <ip>:<port>`

4. (Solo en el manager) Clonar y ejecutar el stack de docker swarm:
   - Acceder y clonar desde el siguiente repositorio de github: `https://github.com/ToxicSSJ/blogpi-stack`
   - Cambiar `ejemplo.me` a su dominio personal en el siguiente comando `sed -i -e 's/ejemplo.me/yourdomain.com/g' ./docker-stack.yml`.
   - Ejecutar el comando anterior.
   - Crear las carpetas requeridas (tal cual configuradas en el yml) `mkdir -p /data/{wp,mysql,letsencrypt}_data`
   - Crear dos networks para tener un mayor control y organización sobre los servicios a ejecutar:
	   - `docker network create --driver overlay --scope swarm nw-web`
	   - `docker network create --driver overlay --scope swarm nw-backend`
   - Cambiar prefix name por el nombre del stack (a preferencia) `docker stack deploy -c docker-stack.yml <prefix_name>`

5. Revisar la ejecución:
   - `docker service ls`
   - `docker service logs -f <prefix_name>_nginx`

---
Pasos para escalado:

1. Para escalar simplemente es cuestión de ajustar el numero de replicas:
   - `docker service scale my_wordpress_nginx=<replicas>`
   - `docker service scale my_wordpress_wordpress=<replicas>`

2. Se puede revisar en todo momento dicha cantidad de replicas utilizando: `docker service ls`

---
Pasos para certificación SSL por medio de Cloudflare:
1. Lo unico que se debe de hacer es asegurarse que los dominios coincidan, donde el email del dominio en la configuración de Traefik apunte a <mi_dominio>. Por otro lado se debe configurar el token de Cloudflare remplazando la variable **CF_DNS_API_TOKEN**.

---
Mecanismo de Cache:
> La imagen introducida para nginx tiene capacidad de usar fastcgi_cache, lo cual nos dispone de un fpm completamente configurable, haciendo tiempos de respuesta mas rápidos. Toda esta configuración debe editarse desde la ruta de configuración de nginx: `./apps/nginx/nginx_conf/conf.d/default.conf`