# Tutorial de manejo de OpenROAD

## 0. Enlaces de interés

- [Fuentes de OpenROAD](https://github.com/The-OpenROAD-Project/OpenROAD/)
- [Documentación de OpenROAD](https://openroad.readthedocs.io/en/latest/index.html)
- [Fuentes del flujo de trabajo digital de OpenROAD](https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts)
- [Dcumentación del flujo de trabajo digital de OpenROAD](https://openroad-flow-scripts.readthedocs.io/en/latest/index2.html)
- [Documentación del PDK ihp13sg2](https://github.com/IHP-GmbH/IHP-Open-PDK)

## 1. Instalación

Si es necesario instalamos docker y docker compose:

``` text
sudo apt  install docker.io
sudo apt-get install docker-buildx
sudo usermod -aG docker $USER
newgrp docker
```

Bajamos las fuentes:

``` text
cd ~/Proyectos/demo_openroad
git clone https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts.git
cd OpenROAD-flow-scripts
git checkout 6f51ec8c93e1edeb3be4d2747894fd28f711f28f
```

Compilamos:

``` text
./build_openroad.sh
```

``` text
user@user:~/Proyectos/demo_openroad/OpenROAD-flow-scripts$ docker images
REPOSITORY                          TAG      IMAGE ID       CREATED         SIZE
openroad/flow-ubuntu22.04-builder   8287a5   e3f097c7b632   7 hours ago     4.56GB
openroad/flow-ubuntu22.04-dev       8287a5   5b2edb392ada   7 hours ago     3.26GB
```

En el ejemplo anterior la imagen generada tiene el tag 8287a5. Usamos este comando para entrar en la máquina docker:

``` text
# Ejecutamos el comando para entrar en la imagen de docker
user@user:~/Proyectos/demo_openroad/OpenROAD-flow-scripts$ docker run --rm -it \
           -u $(id -u ${USER}):$(id -g ${USER}) \
           -v $(pwd)/flow:/OpenROAD-flow-scripts/flow \
                   -v /etc/passwd:/etc/passwd:ro \
                   -v /etc/group:/etc/group:ro \
           -e DISPLAY=${DISPLAY} \
           -v /tmp/.X11-unix:/tmp/.X11-unix \
           -v ${HOME}/.Xauthority:/.Xauthority \
           --network host \
           --security-opt seccomp=unconfined \
           openroad/flow-ubuntu22.04-builder:8287a5

# comprobamos que yosys está instalado
user@user:/OpenROAD-flow-scripts$ yosys --version
Yosys 0.51 (git sha1 c4b519022, g++ 11.4.0-1ubuntu1~22.04 -fPIC -O3)

# ajustamos el entorno de trabajo para que las rutas de los comandos sean adecuadas
user@user:/OpenROAD-flow-scripts$ source env.sh 
OPENROAD: /OpenROAD-flow-scripts/tools/OpenROAD

# ejecutamos la consola de OpenROAD
user@user:/OpenROAD-flow-scripts$ openroad 
OpenROAD HEAD-HASH-NOTFOUND 
Features included (+) or not (-): +GPU +GUI +Python
This program is licensed under the BSD-3 license. See the LICENSE file for details.
Components of this program may be licensed under more restrictive licenses which must be honored.
warning: `/home/user/.tclsh-history' is not writable.
openroad>
```

Para salir de la consola de OpenROAD se debe ejecutar el comando 'exit'.

Recomendamos grabar el contenido del comando de ejecución del docker en un script llamado 'docker_gui.sh' para poder ejecutarlo de manera más sencilla.

``` text
user@user:~/Proyectos/demo_openroad/OpenROAD-flow-scripts$ echo docker run --rm -it \
           -u $(id -u ${USER}):$(id -g ${USER}) \
           -v $(pwd)/flow:/OpenROAD-flow-scripts/flow \
                   -v /etc/passwd:/etc/passwd:ro \
                   -v /etc/group:/etc/group:ro \
           -e DISPLAY=${DISPLAY} \
           -v /tmp/.X11-unix:/tmp/.X11-unix \
           -v ${HOME}/.Xauthority:/.Xauthority \
           --network host \
           --security-opt seccomp=unconfined \
           openroad/flow-ubuntu22.04-builder:8287a5 > docker_gui.sh
user@user:~/Proyectos/demo_openroad/OpenROAD-flow-scripts$ chmod +x docker_gui.sh
```

## 2. Flujo de trabajo de ejemplo

``` text
sh docker_gui.sh
source env.sh 
cd flow
export DESIGN_CONFIG=./designs/asap7/aes/config.mk
make synth
make gui_synth
make floorplan
make gui_floorplan
make place
make gui_place
make cts
make gui_cts
make route
make gui_route
make finish
make gui_finish
make clean_all
make
```

## 3. Prácticas

### 3.1 RTL-GDSII de un perférico

Tomar el RTL de este repositorio:

https://github.com/stephanosio/BPSKModem.git

Generar la configuración en sky130hd, ihp-sg13g2 y asap7. Se debe generar un GDS lo más pequeño posible, cumpliento los tiempos de hold y slack y con un objetivo de frecuencia de reloj de 200MHz para sky130hd, ihp-sg13g2 y de 1GHz para asap7.

Recuerde que las unidades de las constraints de tiempos dependen del PDK. En sky130hd y ihp-sg13g2 son en "ns" y en asap7 en "ps".

### 3.2 RTL-GDSII de un SOC

Tomar el RTL de este repositorio:

https://github.com/YosysHQ/picorv32

Generar una configuración para sintetizar el picosoc en el PDK ihp-sg13g2 con un objetivo de frecuencia de reloj de 50MHz.

### 3.2 RTL-GDSII de un SOC modificado

Modificar el SOC anterior para cambiar la implementación del procesador con el bus de memoria simple por un wishbone, añadir un periférico por wishbone para el manejo de 8 señales de salida y 8 de entrada. Aumentar la memoria RAM del procesador hasta 1Kbyte.

Generar una configuración para sintetizar este picosoc en el PDK ihp-sg13g2 con un objetivo de frecuencia de reloj de 50MHz.
