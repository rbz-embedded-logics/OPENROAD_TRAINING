# Tutorial de manejo de OpenROAD

## Introducción
Este ejercicio busca replicar los pasos de uso de las herramientas con un ejemplo ya existente.

## Enlaces de interés

- [Fuentes de OpenROAD](https://github.com/The-OpenROAD-Project/OpenROAD/)
- [Documentación de OpenROAD](https://openroad.readthedocs.io/en/latest/index.html)
- [Fuentes del flujo de trabajo digital de OpenROAD](https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts)
- [Dcumentación del flujo de trabajo digital de OpenROAD](https://openroad-flow-scripts.readthedocs.io/en/latest/index2.html)
- [Documentación del PDK ihp13sg2](https://github.com/IHP-GmbH/IHP-Open-PDK)
- [Curso del ETH de Zurich, incluye más detalles sobre diseño VLSI](https://vlsi.ethz.ch/wiki/VLSI_Lectures)

# Instalación

## Si docker no está instalado
Si es necesario instalamos docker y docker compose:

``` text
sudo apt  install docker.io
sudo apt-get install docker-buildx
sudo usermod -aG docker $USER
newgrp docker
```

## Instruciones especiales laboratorio cátedra chip

Instrucciones necesarias para terminar de prepara el entorno Linux instalado:

``` text
docker context use default
sudo xhost +local:docker
docker system prune -a
```

## Descarga de fuentes y compilación 

Bajamos las fuentes:

``` text
mkdir -p ~/Proyectos/demo_openroad
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
docker run --rm -it \
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
```

## Flujo de trabajo de ejemplo

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
make final
make gui_final
make clean_all
make
```

# Ejercicio final

## Flujos RTL-GDSII de un ejemplo más complejo
En los ejemplos, dentro del ihp-sg13g2, está el proyecto i2c-gpio-expander, este proyecto representa el flujo completo incluyendo i/o. Este proyecto es una referencia que ha sido fabricada en silicio.

Regenere el ejemplo para visualizarlo en los distintos pasos.

### RTL-GDSII con ejemplo modificado
En los ejemplos, dentro del sky130hd está el proyecto riscv32i.

Ajuste el valor de CORE_UTILIZATION a valores del 25% y 75%, observe el impacto en el tamaño de la pieza y la dificultad en conseguir cumplir los requisitos de tiempo.





