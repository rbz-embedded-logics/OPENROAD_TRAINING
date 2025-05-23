# Introducción a la práctica

El objetivo de la práctica es la síntesis en silicio de un SoC basado en un procesador RISC-V con varios periféricos sencillos.

Esta práctica se divide en dos partes, en la primera se generará la macro del SoC y en la segunda parte se añadirán los pads y el I/O necesario para tener cun componente completamente diseñado y listo para mandarse a fabricar.

El proyecto se va a realizar usando el PDK ihp-sg13g2

# Descripción del picosoc
El picosoc es un pequeño SoC basado en el procesador RISC-V picorv32 con un pequeño juego de periféricos e interfaces de memoria.

![picorv32](images/picorv32_overview.svg)

Las fuentes están disponibles dentro del proyecto del picorv32 en github:

[https://github.com/YosysHQ/picorv32.git](https://github.com/YosysHQ/picorv32.git)

Dispone de configuraciones para su síntesis en FPGAs o en ASIC. Para este último caso es posibel usarlo sin necesidad de primitivas de FPGA y la memoria se implementara como DFFRAM (RAM sintentizada usando Flip flops y latches). Esta memoria no se tratará como una macro independiente sino como lógica dentro del SOC.

# Descarga y preparación de las fuentes del proyecto picosoc
El primer paso es preparar las fuentes y modificarlas para reducir el número de I/O que se expone en el SOC. Por defecto exporta todo el bus de memoria del procesador, pero estas señales no vamos a llevarlas hacia el exterior.

Estos pasos se van a ejecutar en la máquina host, una vez todo esté preparado se iniciarán los pasos en el sistema docker que contienen las herramientas instaladas.

1. Descarga de las fuentes
Entramos en el directorio en donde otros proyectos almacenan las fuentes y descargamos el proyecto:

``` text
cd ~/Proyectos/demo_openroad/OpenROAD-flow-scripts/flow/designs/src
git clone https://github.com/YosysHQ/picorv32
cd picorv32
```

2. Modificación de las fuentes
Dentro del directorio en donde están las fuentes del picorv32 existe una carpeta llamada picosoc, dentro de ésta el fichero picosoc.v contiene el top module del SoC.

Editamos este fichero y haremos los siguientes cambios:

1. Eliminar el bus de memoria hacia fuera
```
module picosoc (
	input clk,
	input resetn,

	output        iomem_valid,
	input         iomem_ready,
	output [ 3:0] iomem_wstrb,
	output [31:0] iomem_addr,
	output [31:0] iomem_wdata,
	input  [31:0] iomem_rdata,
```

se cambia por:

```
module picosoc (
	input clk,
	input resetn,

	//output        iomem_valid,
	//input         iomem_ready,
	//output [ 3:0] iomem_wstrb,
	//output [31:0] iomem_addr,
	//output [31:0] iomem_wdata,
	//input  [31:0] iomem_rdata,
```

2. Añadimos más señales de irq:

```
	//input  [31:0] iomem_rdata,

	input  irq_5,
	input  irq_6,
	input  irq_7,

	output ser_tx,
	input  ser_rx,
```

se cambia por:

```
	//input  [31:0] iomem_rdata,

	input  irq_5,
	input  irq_6,
	input  irq_7,
	input  irq_8,
	input  irq_9,
	input  irq_10,
	input  irq_11,

	output ser_tx,
	input  ser_rx,
```

3. Modificamos la configuración del core para tener todas las opciones:

``` 
       parameter [0:0] ENABLE_DIV = 1;
       parameter [0:0] ENABLE_FAST_MUL = 0;
       parameter [0:0] ENABLE_COMPRESSED = 1;
       parameter [0:0] ENABLE_COUNTERS = 1;
       parameter [0:0] ENABLE_IRQ_QREGS = 0;
```

se cambia por:

```
       parameter [0:0] ENABLE_DIV = 1;
       parameter [0:0] ENABLE_FAST_MUL = 1;
       parameter [0:0] ENABLE_COMPRESSED = 1;
       parameter [0:0] ENABLE_COUNTERS = 1;
       parameter [0:0] ENABLE_IRQ_QREGS = 1;
```

4. Reducimos la memoria del SOC, esto ayudará a mantener los tiempos de place y route contenidos. La memoria se define en palabras de 32 bits, bajamos de 1kByte a 512byte.

```
       parameter integer MEM_WORDS = 256;
```

se cambia por:

```
       parameter integer MEM_WORDS = 128;
```

5. Conectamos las señales de irq externas con las del procesador:

```
always @* begin
		irq = 0;
		irq[3] = irq_stall;
		irq[4] = irq_uart;
		irq[5] = irq_5;
		irq[6] = irq_6;
		irq[7] = irq_7;
```

se cambia por:

```
always @* begin
		irq = 0;
		irq[3] = irq_stall;
		irq[4] = irq_uart;
		irq[5] = irq_5;
		irq[6] = irq_6;
		irq[7] = irq_7;
		irq[8] = irq_8;
		irq[9] = irq_9;
		irq[10] = irq_10;
		irq[11] = irq_11;
``` 

Una vez hechos los cambios podemos ejecutar el comando `git diff` para ver que se han aplicado todos ellos.

# Generación de macro inicial

## Objetivo
El objetivo es realizar una primera síntesis completa de RTL a GDSII del diseño pero sin que se añadan los I/O o los pads para hacer bonding. Al final de esta parte tendremos un proyecto que sintentiza y cumple los tiempos requeridos.

## Creación de configuración del proyecto
Seguimos trabajando en la máquina host. Volvemos al directorio de partida y entramos dentro del directorio en donde se alojan los proyectos basados en el proceso ihp-sg13g2.

```
cd ~/Proyectos/demo_openroad/flow/
cd designs
cd ihp-sg13g2
```

Crearemos un directorio en donde alojaremos nuestro proyecto:

```
mkdir picosoc
cd picosoc
```

Una vez dentro del directorio generamos dos ficheros vacíos que serán el punto de partida del proyecto:

```
touch config.mk
touch constraint.sdc
```

## Configuración del entorno de trabajo
En los siguientes pasos se va a ir alternando entre la máquina host y la máquina docker. Recomendamos tener dos terminales, uno a cada lado para poder trabajar en cada uno de estos entornos:

![linux_shell](images/linux_shell.png)

Dentro de la consola en la que se ejecuta docker entramos con los siguientes comandos:

```
cd ~/Proyectos/demo_openroad/OpenROAD-flow-scripts/
sh docker_gui.sh
source env.sh 
cd flow
```

Recordemos el contenido del script docker_gui.sh:

```
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

El valor `8287a5`es el identificador de la imagen de docker que se obtiene al ejecutar el comando `docker images`. Pueden exitir múltiples instalaciones de las herramientas en el sistema sin interferir entre ellas.

Recomendamos editar el fichero y copiar el contenido en vez de usar el comando `echo`, dado que este comando va a resolver las variables del sistema cuando se escriba el fichero y éstas pueden verse modificadas.

## Configuración de la síntesis
### Preparación de los ficheros
Vamos al directorio en donde está la configuración del proyecto

```
cd ~/Proyectos/demo_openroad/OpenROAD-flow-scripts/flow/designs/ihp-sg13g2/picosoc
```

Primero editamos el fichero `config.mk`. Añadimos el siguiente contenido al principio:

```
export PLATFORM               = ihp-sg13g2

export DESIGN_NAME            = picosoc

# synth
export SYNTH_MEMORY_MAX_BITS  = 128000

export VERILOG_FILES          = $(DESIGN_HOME)/src/picorv32/picosoc/picosoc.v \
                                $(DESIGN_HOME)/src/picorv32/picosoc/simpleuart.v \
                                $(DESIGN_HOME)/src/picorv32/picosoc/spimemio.v \
                                $(DESIGN_HOME)/src/picorv32/picorv32.v
export SDC_FILE               = $(DESIGN_HOME)/$(PLATFORM)/$(DESIGN_NAME)/constraint.sdc
```

La variable `PLATFORM` debe tener el nombre del proceso tecnológico que se va a usar y ayudará a la herramienta a encontrar ficheros importantes:

[https://openroad-flow-scripts.readthedocs.io/en/latest/user/FlowVariables.html#platform](https://openroad-flow-scripts.readthedocs.io/en/latest/user/FlowVariables.html#platform)

La variable `DESIGN_NAME` debe ser igual que el directorio en donde se aloja el fichero de configuración, es recomendable que coincida también con el nombre del objeto del Top Module del diseño.

[https://openroad-flow-scripts.readthedocs.io/en/latest/user/FlowVariables.html#design-specific-configuration-variables](https://openroad-flow-scripts.readthedocs.io/en/latest/user/FlowVariables.html#design-specific-configuration-variables)

Los valores de configuración para la síntesis se pueden consultar aquí:

[https://openroad-flow-scripts.readthedocs.io/en/latest/user/FlowVariables.html#synth-variables](https://openroad-flow-scripts.readthedocs.io/en/latest/user/FlowVariables.html#synth-variables)

La variable `SYNTH_MEMORY_MAX_BITS` se debe ajustar porque el diseño presenta memoria en forma de flip flops y Yosys (la herramienta de síntesis) requiere de esta variable cuando debe optimizar igual o más de 4096 bits (512 bytes).

Posteriormente editamos el fichero `constraint.sdc`, incluyendo el siguiente contenido:

```
current_design picosoc

set clk_name  core_clock
set clk_port_name clk
set clk_period 10
set clk_io_pct 0.2

set clk_port [get_ports $clk_port_name]

create_clock -name $clk_name -period $clk_period  $clk_port

set non_clock_inputs [lsearch -inline -all -not -exact [all_inputs] $clk_port]

set_input_delay  [expr $clk_period * $clk_io_pct] -clock $clk_name $non_clock_inputs
set_output_delay [expr $clk_period * $clk_io_pct] -clock $clk_name [all_outputs]
```

Del fichero anterior el nombre `clk` es el que coincide con la señal de clock el el top del diseño. Se fija un periodo objetivo de 10ns (100MHz) con 2ns (10*0.2) de incertidumbre en este reloj a la entrada  salida.

La definición de los constraints es similar a la existente para FPGAs.

### Ejecución de la síntesis
Entramos en la consola en la que está el sistema docker, es importante que el comando `source env.sh` sea el primero en ejecutarse nada más entrar en el sistema docker.

Si no hemos lanzado la máquina docker estos serían los pasos:

```
cd ~/Proyectos/demo_openroad/OpenROAD-flow-scripts/
sh docker_gui.sh
source env.sh 
cd flow
```

Ahora configuramos el diseño que se va a sintetizar:

```
export DESIGN_CONFIG=./designs/ihp-sg13g2/picosoc/config.mk
make clean_synth
make synth
```

Una vez finalizado veremos que ha generado el fichero `results/ihp-sg13g2/picosoc/base/1_1_yosys.v ./results/ihp-sg13g2/picosoc/base/1_synth.v`, este fichero es el Netlist que ya no contiene un verilog que describe el diseño sino el diseño completo a nivel lógico.

```
exec cp /OpenROAD-flow-scripts/flow/designs/ihp-sg13g2/picosoc/constraint.sdc ./results/ihp-sg13g2/picosoc/base/1_synth.sdc
Warnings: 2 unique messages, 2 total
End of script. Logfile hash: 7b0a3f3e67, CPU: user 8.65s system 0.12s, MEM: 222.58 MB peak
Yosys 0.51 (git sha1 UNKNOWN, g++ 11.4.0-1ubuntu1~22.04 -fPIC -O3)
Time spent: 80% 2x abc (33 sec), 5% 43x opt_clean (2 sec), ...
Elapsed time: 0:41.42[h:]min:sec. CPU time: user 41.13 sys 0.27 (99%). Peak memory: 227920KB.
mkdir -p ./results/ihp-sg13g2/picosoc/base ./logs/ihp-sg13g2/picosoc/base ./reports/ihp-sg13g2/picosoc/base
cp ./results/ihp-sg13g2/picosoc/base/1_1_yosys.v ./results/ihp-sg13g2/picosoc/base/1_synth.v
```

## Configuración del floorplan
### Preparación de los ficheros
Vamos a la consola del host y editamos el fichero `config.mk`. Añadiremos dos variables para hacer una estimación del floorplan:

```
# floorplan
export CORE_UTILIZATION = 40
export PLACE_DENSITY    = 0.50
```

Este enlace tiene la configuración de las variables del `floorplan`:

[https://openroad-flow-scripts.readthedocs.io/en/latest/user/FlowVariables.html#floorplan-variables](https://openroad-flow-scripts.readthedocs.io/en/latest/user/FlowVariables.html#floorplan-variables)

En esta configuración se ha decidido que las celdas lógicas se estime que ocupen un 40% de la superficie y que la densidad de ocupación esa de 0.5 (1.0 = celdas muy proximas; 0.0 = celdas muy separadas). Estos son valores iniciales de trabajo.

### Ejecución del floorplan
Entramos en la consola en la que está el sistema docker y ejecutamos el floorplan:

```
make floorplan
```

Una vez terminado veremos los resultados:

```
==========================================================================
floorplan final report_design_area
--------------------------------------------------------------------------
Design area 1009977 u^2 40% utilization.
Elapsed time: 0:12.94[h:]min:sec. CPU time: user 18.87 sys 19.87 (299%). Peak memory: 371944KB.
/OpenROAD-flow-scripts/flow/scripts/flow.sh 2_2_floorplan_macro macro_place
Running macro_place.tcl, stage 2_2_floorplan_macro
source /OpenROAD-flow-scripts/flow/platforms/ihp-sg13g2/setRC.tcl
No macros found: Skipping macro_placement
Elapsed time: 0:00.33[h:]min:sec. CPU time: user 0.26 sys 0.06 (99%). Peak memory: 191260KB.
/OpenROAD-flow-scripts/flow/scripts/flow.sh 2_3_floorplan_tapcell tapcell
Running tapcell.tcl, stage 2_3_floorplan_tapcell
source /OpenROAD-flow-scripts/flow/platforms/ihp-sg13g2/setRC.tcl
Elapsed time: 0:00.26[h:]min:sec. CPU time: user 0.22 sys 0.04 (100%). Peak memory: 147744KB.
/OpenROAD-flow-scripts/flow/scripts/flow.sh 2_4_floorplan_pdn pdn
Running pdn.tcl, stage 2_4_floorplan_pdn
source /OpenROAD-flow-scripts/flow/platforms/ihp-sg13g2/setRC.tcl
[INFO PDN-0001] Inserting grid: grid
Elapsed time: 0:00.57[h:]min:sec. CPU time: user 0.51 sys 0.06 (100%). Peak memory: 199144KB.
cp ./results/ihp-sg13g2/picosoc/base/2_4_floorplan_pdn.odb ./results/ihp-sg13g2/picosoc/base/2_floorplan.odb
cp ./results/ihp-sg13g2/picosoc/base/2_1_floorplan.sdc ./results/ihp-sg13g2/picosoc/base/2_floorplan.sdc
```

Lanzamos la interfaz gráfica para verificar las dimensiones del componente:

```
make gui_floorplan
```

Podemos escoger la herramienta de medida para ver el tamaño `Tools->Ruler`:

![Tamaño picosoc floorpan](images/picosoc_1.png)

Aquí vemos que el tamaño es de aproximadamente 1622um, ambas dimensiones son iguales por defecto.

## Configuración del place
### Preparación de los ficheros
Las variables de configuración para el proceso de `place` están en:

[https://openroad-flow-scripts.readthedocs.io/en/latest/user/FlowVariables.html#place-variables](https://openroad-flow-scripts.readthedocs.io/en/latest/user/FlowVariables.html#place-variables)

La variable `PLACE_DENSITY` del fichero `config.mk`, que ya se ajustó en el apartado de `floorplan`, también es usada en este proceso.

en este diseño no necesitamos añadir ninguna variable más.

### Ejecución del place
Entramos en la consola en la que está el sistema docker y ejecutamos el place:

```
make place
```

Una vez terminado veremos los resultados de esta colocación:

```
==========================================================================
detailed place report_design_area
--------------------------------------------------------------------------
Design area 1006451 u^2 40% utilization.
Elapsed time: 0:24.64[h:]min:sec. CPU time: user 32.82 sys 27.58 (245%). Peak memory: 670004KB.
cp ./results/ihp-sg13g2/picosoc/base/3_5_place_dp.odb ./results/ihp-sg13g2/picosoc/base/3_place.odb
cp ./results/ihp-sg13g2/picosoc/base/2_floorplan.sdc ./results/ihp-sg13g2/picosoc/base/3_place.sdc
```

Este proceso ha conseguido mantener la utilización en el 40%, que es lo solicitado en la variable de configuración. Podría darse el caso que en este proceso se aumente ligeramente el espacio utilizado al colocar finalmente las celdas lógicas.

## Configuración del CTS
### Preparación de los ficheros
Las variables de configuración para el proceso de `CTS` están en:

[https://openroad-flow-scripts.readthedocs.io/en/latest/user/FlowVariables.html#cts-variables](https://openroad-flow-scripts.readthedocs.io/en/latest/user/FlowVariables.html#cts-variables)

En el fichero `config.mk` vamos a añadir una variable:

```
# cts
export TNS_END_PERCENT = 100
```

Ese valor indica que debe resolver el 100% de los problemas de reloj que se encuentre, aunque no estén en el camino crítico.

### Ejecución del CTS
Entramos en la consola en la que está el sistema docker y ejecutamos el CTS:

```
make cts
```

Una vez terminado veremos los resultados del reloj:

```
==========================================================================
cts final report_design_area
--------------------------------------------------------------------------
Design area 1192063 u^2 47% utilization.
Elapsed time: 0:23.24[h:]min:sec. CPU time: user 39.43 sys 43.95 (358%). Peak memory: 991164KB.
cp ./results/ihp-sg13g2/picosoc/base/4_1_cts.odb ./results/ihp-sg13g2/picosoc/base/4_cts.odb
```

Se observa que ha aumentado el espacio usitlizado de un 40% a un 47% debido a las celdas lógicas y buffers que se han insertado para distribuir el árbol de relojes del diseño.

## Configuración del route
### Preparación de los ficheros
Este proceso se realiza en dos fase, una de rutado inicial y luego el rutado detallado, por ello tiene dos apartados con documentación de las variables involucradas:

[https://openroad-flow-scripts.readthedocs.io/en/latest/user/FlowVariables.html#grt-variables](https://openroad-flow-scripts.readthedocs.io/en/latest/user/FlowVariables.html#grt-variables)
[https://openroad-flow-scripts.readthedocs.io/en/latest/user/FlowVariables.html#route-variables](https://openroad-flow-scripts.readthedocs.io/en/latest/user/FlowVariables.html#route-variables)

La variable `TNS_END_PERCENT` del fichero `config.mk`, que ya se ajustó en el apartado de `CTS`, también es usada en este proceso.

No se hace ninguna configuración adicional.

### Ejecución de la síntesis
Entramos en la consola en la que está el sistema docker y ejecutamos el route:

```
make route
```

Una vez terminado veremos los resultados del rutado:

```
[INFO DRT-0180] Post processing.
Took 298 seconds: detailed_route -output_drc ./reports/ihp-sg13g2/picosoc/base/5_route_drc.rpt -output_maze ./results/ihp-sg13g2/picosoc/base/maze.log -bottom_routing_layer Metal2 -top_routing_layer Metal5 -droute_end_iter 64 -verbose 1 -drc_report_iter_step 5
[INFO ANT-0002] Found 141 net violations.
[INFO ANT-0001] Found 149 pin violations.
Elapsed time: 5:01.01[h:]min:sec. CPU time: user 5988.05 sys 21.11 (1996%). Peak memory: 7283676KB.
/OpenROAD-flow-scripts/flow/scripts/flow.sh 5_3_fillcell fillcell
Running fillcell.tcl, stage 5_3_fillcell
source /OpenROAD-flow-scripts/flow/platforms/ihp-sg13g2/setRC.tcl
filler_placement "sg13g2_fill_1 sg13g2_fill_2 sg13g2_decap_4 sg13g2_decap_8"
[INFO DPL-0001] Placed 148164 filler instances.
Elapsed time: 0:01.50[h:]min:sec. CPU time: user 1.30 sys 0.19 (99%). Peak memory: 599796KB.
cp ./results/ihp-sg13g2/picosoc/base/5_3_fillcell.odb ./results/ihp-sg13g2/picosoc/base/5_route.odb
cp ./results/ihp-sg13g2/picosoc/base/5_1_grt.sdc ./results/ihp-sg13g2/picosoc/base/5_route.sdc
```

## Finalización
### Preparación de los ficheros
Las variables de configuración para el proceso de `final` están en:

[https://openroad-flow-scripts.readthedocs.io/en/latest/user/FlowVariables.html#final-variables](https://openroad-flow-scripts.readthedocs.io/en/latest/user/FlowVariables.html#final-variables)

En el fichero `config.mk` ajustamos la variable que indica la tensión a la que se calcula la caída de tensión y el consumo del componente. En este tecnología por defecto son 1.2V y lo ponemos en 1.8V.

```
# final
export PWR_NETS_VOLTAGES = VDD 1.8
```

### Ejecución del diseño final
Entramos en la consola en la que está el sistema docker y ejecutamos el final:

```
make final
```

Una vez terminado veremos los resultados finales:

```
Elapsed time: 0:03.78[h:]min:sec. CPU time: user 3.52 sys 0.26 (99%). Peak memory: 1043596KB.
cp results/ihp-sg13g2/picosoc/base/6_1_merged.gds results/ihp-sg13g2/picosoc/base/6_final.gds
./logs/ihp-sg13g2/picosoc/base
Log                            Elapsed seconds Peak Memory/MB
1_1_yosys                                   41            222
1_1_yosys_canonicalize                       0             37
2_1_floorplan                               12            363
2_2_floorplan_macro                          0            186
2_3_floorplan_tapcell                        0            144
2_4_floorplan_pdn                            0            194
3_1_place_gp_skip_io                        22            291
3_2_place_iop                                0            187
3_3_place_gp                                70            870
3_4_place_resized                           17            555
3_5_place_dp                                24            654
4_1_cts                                     23            967
5_1_grt                                     46           1716
5_2_route                                  301           7112
5_3_fillcell                                 1            585
6_1_fill                                     0            293
6_1_merge                                    3           1019
6_report                                    98           2175
Total                                      658           7112
```

Una vez terminado lanzamos la visualización para ver el resultado:

```
make gui_final
```

![Picosoc macro final](images/picosoc_2.png)

# Generación de un SOC completo

## Objetivo
Los pasos anteriores han generado una macro y sobre todo han permitido obtener unos parámetros iniciales de espacio utilizado por el diseño. No se han hecho una optimización de espacio.

El siguiente punto consiste en la modificación del diseño para poder incorporar los PADs de conexión al exterior, los PADs de bonding y ajustar la alimentación.

## Descripción del I/O del ihp-sg13g2
El proceso ihp-sg13g2 posee una librería de componentes aptos para realizar procesos de I/O. Estos componentes tienen la siguiente características:
- Pueden funcionar con una tensión distinta de la del Core. Cuando hay I/O tendremos VDD para el core y VDDIO para la entrada salida, ambos valores no tienen porque coincidir. En este proceso el Core está probado a 1.2V y el I/O hasta 3.3V.
- Poseen protecciones contra electricidad estática usando diodos de clamping contra alimentación y masa
- Tienen transistores más grandes que las celdas lógicas para poder entregar o consumir más corriente.

Dentro de la consola del host podemos consultar los siguientes ficheros:

```
cd ~/Proyectos/demo_openroad/OpenROAD-flow-scripts/flow/platforms/ihp-sg13g2
```

Ahí dentro podemos ver el fichero `verilog/sg13g2_io.v` en donde podemos ver las definiciones en verilog de los bloques de I/O y las señales de control que tienen 

Por ejemplo la siguiente celda de I/O:

```
// type: TriStateOutput4mA
`timescale 1ns/10ps
`celldefine
module sg13g2_IOPadTriOut4mA (pad, c2p, c2p_en);
        inout pad;
        input c2p;
        input c2p_en;

        // Function
        assign pad = (c2p_en) ? c2p : 1'bz;

        // Timing
        specify
                if (c2p_en == 1'b1)
                        (c2p => pad) = 0;
        endspecify
endmodule
```

El nombre indica que es un Pad de salida triestado con hasta 4mA de corriente de salida. Tiene tres señales de control:
- pad: se corresponde con el pad que hace de interfaz con el exterior
- c2p: señal a la entrada que puede o no conectarse con la salida
- c2p_en: señal de habilitación de la salida, escoge entre conectar c2p a pad o dejar la salida en alta impedancia

Al final del fichero también se definen otras celdas como son las de alimentación para que estén accesibles en el flujo de trabajo aunque no se conectan a ninguna señal. Esta definición es propia de cada PDK.

```
// type: IOVss
`timescale 1ns/10ps
`celldefine
module sg13g2_IOPadIOVss ();
endmodule
`endcelldefine

// type: IOVdd
`timescale 1ns/10ps
`celldefine
module sg13g2_IOPadIOVdd ();
endmodule
`endcelldefine

// type: Vss
`timescale 1ns/10ps
`celldefine
module sg13g2_IOPadVss ();
endmodule
`endcelldefine

// type: Vdd
`timescale 1ns/10ps
`celldefine
module sg13g2_IOPadVdd ();
endmodule
`endcelldefine
```

En el fichero `lef/sg13g2_io.lef` podemos ver las definiciones de la geometría de las celdas. Por ejemplo para el caso del sg13g2_IOPadTriOut4mA podemos ver las capas en las que están las señales:

```
MACRO sg13g2_IOPadTriOut4mA
    CLASS PAD OUTPUT ;
    ORIGIN 0.000 0.000 ;
    FOREIGN sg13g2_IOPadTriOut4mA 0.000 0.000 ;
    SIZE 80.000 BY 180.000 ;
    SYMMETRY X Y R90 ;
    SITE sg13g2_ioSite ;
  PIN c2p
    DIRECTION INPUT ;
    USE SIGNAL ;
    PORT
      LAYER Metal2 ;
        RECT 38.330 178.090 38.620 180.000 ;
      LAYER Metal3 ;
        RECT 38.225 179.710 38.725 180.000 ;
    END
  END c2p
  PIN c2p_en 
    DIRECTION INPUT ;
    USE SIGNAL ;
    PORT
      LAYER Metal2 ;
        RECT 41.380 174.045 41.670 180.000 ;
      LAYER Metal3 ;
        RECT 41.275 179.710 41.775 180.000 ;
    END 
  END c2p_en
  ```

  La definición continúa con los pines de alimentación de core y de I/O. Podemos ver que usan todas las capas de metalización hasta la TopMetal2 que es la superior en donde se colocan los pads de conexión al exterior. En este diagrama es la metalización superior con un grosor de 300um.

  ![ihp-sg13g2](images/ihp-sg13g2.png)

Deberemos ajustar el diseño para que ahora las señales de salida vayan a PADs de este proceso.

## Creación de la estructura de ficheros

Vamos a crear el proyecto desde la consola del host, similar a lo ya generado con el diseño anterior:

```
cd ~/Proyectos/demo_openroad/flow/
cd designs
cd ihp-sg13g2
mkdir picosoc_ihp
cd picosoc_ihp
```

Una vez dentro del directorio generamos dos ficheros vacíos que serán el punto de partida del proyecto:

```
touch config.mk
touch constraint.sdc
touch pad.tcl
touch pdn.tcl
touch picosoc_ihp.v
```

El wrapper de verilog del SoC se genera en la carpeta del proyecto para tenerlo asociado al proyecto de este PDK.

## Creación del wrapper de verilog del SoC
Vamos a editar el fichero `picosoc_ihp.v` con este contenido:

```

```


## Configuración de la síntesis


## Configuración del floorplan


## Configuración del place


## Configuración del PDN


## Configuración del CTS


## Configuración del route


## Finalización
