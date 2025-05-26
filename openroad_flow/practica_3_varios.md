
# RTL-GDSII de un periférico
Tomar el RTL de este repositorio:

https://github.com/stephanosio/BPSKModem.git

Generar la configuración en sky130hd, ihp-sg13g2 y asap7. Se debe generar un GDS lo más pequeño posible, cumpliento los tiempos de hold y slack y con un objetivo de frecuencia de reloj de 200MHz para sky130hd, ihp-sg13g2 y de 1GHz para asap7.

Recuerde que las unidades de las constraints de tiempos dependen del PDK. En sky130hd y ihp-sg13g2 son en "ns" y en asap7 en "ps".

El proyecto está en systemverilog, deberá añadir lo siguiente al fichero de configuración para que la síntesis sea correcta:

```
export SYNTH_HDL_FRONTEND = slang
```

# Crear un componente sencillo
Tomar el código RTL de este repositorio:

https://github.com/djuara-rbz/tt_spi_pwm/

Se debe generar un proyecto para ihp-sg13g2 con PADs de I/O y alimentación.

El objetivo de reloj es de 20MHz.

Solo hay un reloj, en el top de ese teo el reloj es `clk`.

Las señales `ui` son solo de entrada, las `uo` de salida y las `uio` de entrada salida consolada por la señal `oe`.

El fichero `config.tcl` debe ignorarse y no se usa en este entorno.

# Sintentizar un procesador que ejecuta uClinux desde memorias RAM/FLASH por SPI
Tomar el código RTL de este repositorio:

https://github.com/splinedrive/KianV_rv32ia_uLinux_SoC

Se debe generar un proyecto para ihp-sg13g2 con PADs de I/O y alimentación.

El objetivo de reloj es de 20MHz.

Solo hay un reloj, en el top de ese teo el reloj es `clk`.

Las señales `ui` son solo de entrada, las `uo` de salida y las `uio` de entrada salida consolada por la señal `oe`.

El fichero `config.tcl` debe ignorarse y no se usa en este entorno.