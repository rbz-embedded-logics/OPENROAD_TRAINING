
# RTL-GDSII de un periférico
Tomar el RTL de este repositorio:

https://github.com/stephanosio/BPSKModem.git

Generar la configuración en sky130hd, ihp-sg13g2 y asap7. Se debe generar un GDS lo más pequeño posible, cumpliento los tiempos de hold y slack y con un objetivo de frecuencia de reloj de 200MHz para sky130hd, ihp-sg13g2 y de 1GHz para asap7.

Recuerde que las unidades de las constraints de tiempos dependen del PDK. En sky130hd y ihp-sg13g2 son en "ns" y en asap7 en "ps".

El proyecto está en systemverilog, deberá añadir lo siguiente al fichero de configuración para que la síntesis sea correcta:

```
export SYNTH_HDL_FRONTEND = slang
```
