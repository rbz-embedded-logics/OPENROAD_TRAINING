
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

# Sintetizar un procesador que ejecuta uClinux desde memorias RAM/FLASH por SPI
Tomar el código RTL de este repositorio:

https://github.com/splinedrive/KianV_rv32ia_uLinux_SoC

Se debe generar un proyecto para ihp-sg13g2 con PADs de I/O y alimentación.

El objetivo de reloj es de 20MHz.

Solo hay un reloj, en el top de ese teo el reloj es `clk`.

Las señales `ui` son solo de entrada, las `uo` de salida y las `uio` de entrada salida consolada por la señal `oe`.

El fichero `config.tcl` debe ignorarse y no se usa en este entorno.

# Ficheros de ejemplo

## Wrapper de interfaz tinytapeout
```
`timescale 1ns/1ps

// definimos las señales que estarán conectadas a los I/O
module tt_pwm (
  inout wire io_ui_in_0_PAD,
  inout wire io_ui_in_1_PAD,
  inout wire io_ui_in_2_PAD,
  inout wire io_ui_in_3_PAD,
  inout wire io_ui_in_4_PAD,
  inout wire io_ui_in_5_PAD,
  inout wire io_ui_in_6_PAD,
  inout wire io_ui_in_7_PAD,
  inout wire io_uo_out_0_PAD,
  inout wire io_uo_out_1_PAD,
  inout wire io_uo_out_2_PAD,
  inout wire io_uo_out_3_PAD,
  inout wire io_uo_out_4_PAD,
  inout wire io_uo_out_5_PAD,
  inout wire io_uo_out_6_PAD,
  inout wire io_uo_out_7_PAD,
  inout wire io_uio_0_PAD,
  inout wire io_uio_1_PAD,
  inout wire io_uio_2_PAD,
  inout wire io_uio_3_PAD,
  inout wire io_uio_4_PAD,
  inout wire io_uio_5_PAD,
  inout wire io_uio_6_PAD,
  inout wire io_uio_7_PAD,
  inout wire io_ena_PAD,
  inout wire io_clk_PAD,
  inout wire io_rst_n_PAD,
);

  wire sg13g2_IOPad_io_ena_p2c;
  wire sg13g2_IOPad_io_clock_p2c;
  wire sg13g2_IOPad_io_rst_n_p2c;

  wire sg13g2_IOPad_io_ui_in_0_p2c;
  wire sg13g2_IOPad_io_ui_in_1_p2c;
  wire sg13g2_IOPad_io_ui_in_2_p2c;
  wire sg13g2_IOPad_io_ui_in_3_p2c;
  wire sg13g2_IOPad_io_ui_in_4_p2c;
  wire sg13g2_IOPad_io_ui_in_5_p2c;
  wire sg13g2_IOPad_io_ui_in_6_p2c;
  wire sg13g2_IOPad_io_ui_in_7_p2c;

  wire sg13g2_IOPad_io_uo_out_0_c2p;
  wire sg13g2_IOPad_io_uo_out_1_c2p;
  wire sg13g2_IOPad_io_uo_out_2_c2p;
  wire sg13g2_IOPad_io_uo_out_3_c2p;
  wire sg13g2_IOPad_io_uo_out_4_c2p;
  wire sg13g2_IOPad_io_uo_out_5_c2p;
  wire sg13g2_IOPad_io_uo_out_6_c2p;
  wire sg13g2_IOPad_io_uo_out_7_c2p;

  wire sg13g2_IOPad_io_uio_in_0_p2c;
  wire sg13g2_IOPad_io_uio_in_1_p2c;
  wire sg13g2_IOPad_io_uio_in_2_p2c;
  wire sg13g2_IOPad_io_uio_in_3_p2c;
  wire sg13g2_IOPad_io_uio_in_4_p2c;
  wire sg13g2_IOPad_io_uio_in_5_p2c;
  wire sg13g2_IOPad_io_uio_in_6_p2c;
  wire sg13g2_IOPad_io_uio_in_7_p2c;

  wire sg13g2_IOPad_io_uio_out_0_c2p;
  wire sg13g2_IOPad_io_uio_out_1_c2p;
  wire sg13g2_IOPad_io_uio_out_2_c2p;
  wire sg13g2_IOPad_io_uio_out_3_c2p;
  wire sg13g2_IOPad_io_uio_out_4_c2p;
  wire sg13g2_IOPad_io_uio_out_5_c2p;
  wire sg13g2_IOPad_io_uio_out_6_c2p;
  wire sg13g2_IOPad_io_uio_out_7_c2p;

  wire sg13g2_IOPad_io_uio_oe_0_c2p;
  wire sg13g2_IOPad_io_uio_oe_1_c2p;
  wire sg13g2_IOPad_io_uio_oe_2_c2p;
  wire sg13g2_IOPad_io_uio_oe_3_c2p;
  wire sg13g2_IOPad_io_uio_oe_4_c2p;
  wire sg13g2_IOPad_io_uio_oe_5_c2p;
  wire sg13g2_IOPad_io_uio_oe_6_c2p;
  wire sg13g2_IOPad_io_uio_oe_7_c2p;

  assign clock = sg13g2_IOPad_io_clock_p2c;

 // Clock, ena, reset
 sg13g2_IOPadIn sg13g2_IOPad_io_clock (.p2c (sg13g2_IOPad_io_clock_p2c), .pad (io_clk_PAD));
 sg13g2_IOPadIn sg13g2_IOPad_io_ena   (.p2c (sg13g2_IOPad_io_ena_p2c),   .pad (io_ena_PAD));
 sg13g2_IOPadIn sg13g2_IOPad_io_rst_n (.p2c (sg13g2_IOPad_io_rst_n_p2c), .pad (io_rst_n_PAD));

 //ui_in
 sg13g2_IOPadIn sg13g2_IOPad_io_ui_in_0 (.p2c (sg13g2_IOPad_io_ui_in_0_p2c), .pad (io_ui_in_0_PAD));
 sg13g2_IOPadIn sg13g2_IOPad_io_ui_in_1 (.p2c (sg13g2_IOPad_io_ui_in_1_p2c), .pad (io_ui_in_1_PAD));
 sg13g2_IOPadIn sg13g2_IOPad_io_ui_in_2 (.p2c (sg13g2_IOPad_io_ui_in_2_p2c), .pad (io_ui_in_2_PAD));
 sg13g2_IOPadIn sg13g2_IOPad_io_ui_in_3 (.p2c (sg13g2_IOPad_io_ui_in_3_p2c), .pad (io_ui_in_3_PAD));
 sg13g2_IOPadIn sg13g2_IOPad_io_ui_in_4 (.p2c (sg13g2_IOPad_io_ui_in_4_p2c), .pad (io_ui_in_4_PAD));
 sg13g2_IOPadIn sg13g2_IOPad_io_ui_in_5 (.p2c (sg13g2_IOPad_io_ui_in_5_p2c), .pad (io_ui_in_5_PAD));
 sg13g2_IOPadIn sg13g2_IOPad_io_ui_in_6 (.p2c (sg13g2_IOPad_io_ui_in_6_p2c), .pad (io_ui_in_6_PAD));
 sg13g2_IOPadIn sg13g2_IOPad_io_ui_in_7 (.p2c (sg13g2_IOPad_io_ui_in_7_p2c), .pad (io_ui_in_7_PAD));
 
 //uo_out
 sg13g2_IOPadOut4mA sg13g2_IOPad_io_uo_out_0 (.c2p (sg13g2_IOPad_io_uo_out_0_c2p), .pad (io_uo_out_0_PAD));
 sg13g2_IOPadOut4mA sg13g2_IOPad_io_uo_out_1 (.c2p (sg13g2_IOPad_io_uo_out_1_c2p), .pad (io_uo_out_1_PAD));
 sg13g2_IOPadOut4mA sg13g2_IOPad_io_uo_out_2 (.c2p (sg13g2_IOPad_io_uo_out_2_c2p), .pad (io_uo_out_2_PAD));
 sg13g2_IOPadOut4mA sg13g2_IOPad_io_uo_out_3 (.c2p (sg13g2_IOPad_io_uo_out_3_c2p), .pad (io_uo_out_3_PAD));
 sg13g2_IOPadOut4mA sg13g2_IOPad_io_uo_out_4 (.c2p (sg13g2_IOPad_io_uo_out_4_c2p), .pad (io_uo_out_4_PAD));
 sg13g2_IOPadOut4mA sg13g2_IOPad_io_uo_out_5 (.c2p (sg13g2_IOPad_io_uo_out_5_c2p), .pad (io_uo_out_5_PAD));
 sg13g2_IOPadOut4mA sg13g2_IOPad_io_uo_out_6 (.c2p (sg13g2_IOPad_io_uo_out_6_c2p), .pad (io_uo_out_6_PAD));
 sg13g2_IOPadOut4mA sg13g2_IOPad_io_uo_out_7 (.c2p (sg13g2_IOPad_io_uo_out_7_c2p), .pad (io_uo_out_7_PAD));

 //uio_in
  sg13g2_IOPadInOut4mA sg13g2_IOPad_uio_uo_0 (.pad(io_uio_0_PAD), .c2p(sg13g2_IOPad_io_uio_out_0_c2p), .c2p_en(sg13g2_IOPad_io_uio_oe_0_c2p), .p2c(sg13g2_IOPad_io_uio_in_0_p2c));
  sg13g2_IOPadInOut4mA sg13g2_IOPad_uio_uo_1 (.pad(io_uio_1_PAD), .c2p(sg13g2_IOPad_io_uio_out_1_c2p), .c2p_en(sg13g2_IOPad_io_uio_oe_1_c2p), .p2c(sg13g2_IOPad_io_uio_in_1_p2c));
  sg13g2_IOPadInOut4mA sg13g2_IOPad_uio_uo_2 (.pad(io_uio_2_PAD), .c2p(sg13g2_IOPad_io_uio_out_2_c2p), .c2p_en(sg13g2_IOPad_io_uio_oe_2_c2p), .p2c(sg13g2_IOPad_io_uio_in_2_p2c));
  sg13g2_IOPadInOut4mA sg13g2_IOPad_uio_uo_3 (.pad(io_uio_3_PAD), .c2p(sg13g2_IOPad_io_uio_out_3_c2p), .c2p_en(sg13g2_IOPad_io_uio_oe_3_c2p), .p2c(sg13g2_IOPad_io_uio_in_3_p2c));
  sg13g2_IOPadInOut4mA sg13g2_IOPad_uio_uo_4 (.pad(io_uio_4_PAD), .c2p(sg13g2_IOPad_io_uio_out_4_c2p), .c2p_en(sg13g2_IOPad_io_uio_oe_4_c2p), .p2c(sg13g2_IOPad_io_uio_in_4_p2c));
  sg13g2_IOPadInOut4mA sg13g2_IOPad_uio_uo_5 (.pad(io_uio_5_PAD), .c2p(sg13g2_IOPad_io_uio_out_5_c2p), .c2p_en(sg13g2_IOPad_io_uio_oe_5_c2p), .p2c(sg13g2_IOPad_io_uio_in_5_p2c));
  sg13g2_IOPadInOut4mA sg13g2_IOPad_uio_uo_6 (.pad(io_uio_6_PAD), .c2p(sg13g2_IOPad_io_uio_out_6_c2p), .c2p_en(sg13g2_IOPad_io_uio_oe_6_c2p), .p2c(sg13g2_IOPad_io_uio_in_6_p2c));
  sg13g2_IOPadInOut4mA sg13g2_IOPad_uio_uo_7 (.pad(io_uio_7_PAD), .c2p(sg13g2_IOPad_io_uio_out_7_c2p), .c2p_en(sg13g2_IOPad_io_uio_oe_7_c2p), .p2c(sg13g2_IOPad_io_uio_in_7_p2c));


  tt_um_spi_pwm_djuara tt_pwm_mod (
    .ui_in  ({sg13g2_IOPad_io_ui_in_7_p2c,   sg13g2_IOPad_io_ui_in_6_p2c,   sg13g2_IOPad_io_ui_in_5_p2c,   sg13g2_IOPad_io_ui_in_4_p2c,   sg13g2_IOPad_io_ui_in_3_p2c,   sg13g2_IOPad_io_ui_in_2_p2c,   sg13g2_IOPad_io_ui_in_1_p2c,   sg13g2_IOPad_io_ui_in_0_p2c}),    // Dedicated inputs
    .uo_out ({sg13g2_IOPad_io_uo_out_7_c2p,  sg13g2_IOPad_io_uo_out_6_c2p,  sg13g2_IOPad_io_uo_out_5_c2p,  sg13g2_IOPad_io_uo_out_4_c2p,  sg13g2_IOPad_io_uo_out_3_c2p,  sg13g2_IOPad_io_uo_out_2_c2p,  sg13g2_IOPad_io_uo_out_1_c2p,  sg13g2_IOPad_io_uo_out_0_c2p}),    // Dedicated inputs
    .uio_in ({sg13g2_IOPad_io_uio_in_7_p2c,  sg13g2_IOPad_io_uio_in_6_p2c,  sg13g2_IOPad_io_uio_in_5_p2c,  sg13g2_IOPad_io_uio_in_4_p2c,  sg13g2_IOPad_io_uio_in_3_p2c,  sg13g2_IOPad_io_uio_in_2_p2c,  sg13g2_IOPad_io_uio_in_1_p2c,  sg13g2_IOPad_io_uio_in_0_p2c}),    // Dedicated inputs
    .uio_out({sg13g2_IOPad_io_uio_out_7_c2p, sg13g2_IOPad_io_uio_out_6_c2p, sg13g2_IOPad_io_uio_out_5_c2p, sg13g2_IOPad_io_uio_out_4_c2p, sg13g2_IOPad_io_uio_out_3_c2p, sg13g2_IOPad_io_uio_out_2_c2p, sg13g2_IOPad_io_uio_out_1_c2p, sg13g2_IOPad_io_uio_out_0_c2p}),    // Dedicated inputs
    .uio_oe ({sg13g2_IOPad_io_uio_oe_7_c2p,  sg13g2_IOPad_io_uio_oe_6_c2p,  sg13g2_IOPad_io_uio_oe_5_c2p,  sg13g2_IOPad_io_uio_oe_4_c2p,  sg13g2_IOPad_io_uio_oe_3_c2p,  sg13g2_IOPad_io_uio_oe_2_c2p,  sg13g2_IOPad_io_uio_oe_1_c2p,  sg13g2_IOPad_io_uio_oe_0_c2p}),    // Dedicated inputs
    .ena(sg13g2_IOPad_io_ena_p2c),
    .clk(clock),
    .rst_n(sg13g2_IOPad_io_rst_n_p2c)
);

endmodule
```

## PADs tcl
```
set IO_LENGTH 180
set IO_WIDTH 80
set BONDPAD_SIZE 70
set SEALRING_OFFSET 70

proc calc_horizontal_pad_location {index total} {
    global IO_LENGTH
    global IO_WIDTH
    global BONDPAD_SIZE
    global SEALRING_OFFSET

    set DIE_WIDTH [expr {[lindex $::env(DIE_AREA) 2] - [lindex $::env(DIE_AREA) 0]}]
    set PAD_OFFSET [expr {$IO_LENGTH + $BONDPAD_SIZE + $SEALRING_OFFSET}]
    set PAD_AREA_WIDTH [expr {$DIE_WIDTH - ($PAD_OFFSET * 2)}]
    set HORIZONTAL_PAD_DISTANCE [expr {($PAD_AREA_WIDTH / $total) - $IO_WIDTH}]

    return [expr {$PAD_OFFSET + (($IO_WIDTH + $HORIZONTAL_PAD_DISTANCE) * $index) + ($HORIZONTAL_PAD_DISTANCE / 2)}]
}

proc calc_vertical_pad_location {index total} {
    global IO_LENGTH
    global IO_WIDTH
    global BONDPAD_SIZE
    global SEALRING_OFFSET

    set DIE_HEIGHT [expr {[lindex $::env(DIE_AREA) 3] - [lindex $::env(DIE_AREA) 1]}]
    set PAD_OFFSET [expr {$IO_LENGTH + $BONDPAD_SIZE + $SEALRING_OFFSET}]
    set PAD_AREA_HEIGHT [expr {$DIE_HEIGHT - ($PAD_OFFSET * 2)}]
    set VERTICAL_PAD_DISTANCE [expr {($PAD_AREA_HEIGHT / $total) - $IO_WIDTH}]

    return [expr {$PAD_OFFSET + (($IO_WIDTH + $VERTICAL_PAD_DISTANCE) * $index) + ($VERTICAL_PAD_DISTANCE / 2)}]
}

make_fake_io_site -name IOLibSite -width 1 -height $IO_LENGTH
make_fake_io_site -name IOLibCSite -width $IO_LENGTH -height $IO_LENGTH

set IO_OFFSET [expr {$BONDPAD_SIZE + $SEALRING_OFFSET}]
# Create IO Rows
make_io_sites \
    -horizontal_site IOLibSite \
    -vertical_site IOLibSite \
    -corner_site IOLibCSite \
    -offset $IO_OFFSET

# WEST
place_pad -row IO_WEST -location [calc_vertical_pad_location 7 8] {sg13g2_IOPadVdd_west_0}   -master sg13g2_IOPadVdd
place_pad -row IO_WEST -location [calc_vertical_pad_location 6 8] {sg13g2_IOPadIOVdd_west_1} -master sg13g2_IOPadIOVdd
place_pad -row IO_WEST -location [calc_vertical_pad_location 5 8] {sg13g2_IOPad_io_rst_n}    -master sg13g2_IOPadIn
place_pad -row IO_WEST -location [calc_vertical_pad_location 4 8] {sg13g2_IOPad_io_clock}    -master sg13g2_IOPadIn
place_pad -row IO_WEST -location [calc_vertical_pad_location 3 8] {sg13g2_IOPad_io_ena}      -master sg13g2_IOPadIn

place_pad -row IO_WEST -location [calc_vertical_pad_location 1 8] {sg13g2_IOPadVss_west_6}   -master sg13g2_IOPadVss
place_pad -row IO_WEST -location [calc_vertical_pad_location 0 8] {sg13g2_IOPadIOVss_west_7} -master sg13g2_IOPadIOVss

# NORTH
place_pad -row IO_NORTH -location [calc_horizontal_pad_location 0 8] {sg13g2_IOPad_io_ui_in_0} -master sg13g2_IOPadIn
place_pad -row IO_NORTH -location [calc_horizontal_pad_location 1 8] {sg13g2_IOPad_io_ui_in_1} -master sg13g2_IOPadIn
place_pad -row IO_NORTH -location [calc_horizontal_pad_location 2 8] {sg13g2_IOPad_io_ui_in_2} -master sg13g2_IOPadIn
place_pad -row IO_NORTH -location [calc_horizontal_pad_location 3 8] {sg13g2_IOPad_io_ui_in_3} -master sg13g2_IOPadIn
place_pad -row IO_NORTH -location [calc_horizontal_pad_location 4 8] {sg13g2_IOPad_io_ui_in_4} -master sg13g2_IOPadIn
place_pad -row IO_NORTH -location [calc_horizontal_pad_location 5 8] {sg13g2_IOPad_io_ui_in_5} -master sg13g2_IOPadIn
place_pad -row IO_NORTH -location [calc_horizontal_pad_location 6 8] {sg13g2_IOPad_io_ui_in_6} -master sg13g2_IOPadIn
place_pad -row IO_NORTH -location [calc_horizontal_pad_location 7 8] {sg13g2_IOPad_io_ui_in_7} -master sg13g2_IOPadIn

# EAST
place_pad -row IO_EAST -location [calc_vertical_pad_location 7 8] {sg13g2_IOPad_io_uo_out_0} -master sg13g2_IOPadOut4mA
place_pad -row IO_EAST -location [calc_vertical_pad_location 6 8] {sg13g2_IOPad_io_uo_out_1} -master sg13g2_IOPadOut4mA
place_pad -row IO_EAST -location [calc_vertical_pad_location 5 8] {sg13g2_IOPad_io_uo_out_2} -master sg13g2_IOPadOut4mA
place_pad -row IO_EAST -location [calc_vertical_pad_location 4 8] {sg13g2_IOPad_io_uo_out_3} -master sg13g2_IOPadOut4mA
place_pad -row IO_EAST -location [calc_vertical_pad_location 3 8] {sg13g2_IOPad_io_uo_out_4} -master sg13g2_IOPadOut4mA
place_pad -row IO_EAST -location [calc_vertical_pad_location 2 8] {sg13g2_IOPad_io_uo_out_5} -master sg13g2_IOPadOut4mA
place_pad -row IO_EAST -location [calc_vertical_pad_location 1 8] {sg13g2_IOPad_io_uo_out_6} -master sg13g2_IOPadOut4mA
place_pad -row IO_EAST -location [calc_vertical_pad_location 0 8] {sg13g2_IOPad_io_uo_out_7} -master sg13g2_IOPadOut4mA

# SOUTH
place_pad -row IO_SOUTH -location [calc_horizontal_pad_location 0 8] {sg13g2_IOPad_uio_uo_0} -master sg13g2_IOPadInOut4mA
place_pad -row IO_SOUTH -location [calc_horizontal_pad_location 2 8] {sg13g2_IOPad_uio_uo_1} -master sg13g2_IOPadInOut4mA
place_pad -row IO_SOUTH -location [calc_horizontal_pad_location 4 8] {sg13g2_IOPad_uio_uo_2} -master sg13g2_IOPadInOut4mA
place_pad -row IO_SOUTH -location [calc_horizontal_pad_location 6 8] {sg13g2_IOPad_uio_uo_3} -master sg13g2_IOPadInOut4mA
place_pad -row IO_SOUTH -location [calc_horizontal_pad_location 1 8] {sg13g2_IOPad_uio_uo_4} -master sg13g2_IOPadInOut4mA
place_pad -row IO_SOUTH -location [calc_horizontal_pad_location 3 8] {sg13g2_IOPad_uio_uo_5} -master sg13g2_IOPadInOut4mA
place_pad -row IO_SOUTH -location [calc_horizontal_pad_location 5 8] {sg13g2_IOPad_uio_uo_6} -master sg13g2_IOPadInOut4mA
place_pad -row IO_SOUTH -location [calc_horizontal_pad_location 7 8] {sg13g2_IOPad_uio_uo_7} -master sg13g2_IOPadInOut4mA

# Place Corner Cells and Filler
place_corners sg13g2_Corner

set iofill {
    sg13g2_Filler10000
    sg13g2_Filler4000
    sg13g2_Filler2000
    sg13g2_Filler1000
    sg13g2_Filler400
    sg13g2_Filler200
}

place_io_fill -row IO_NORTH {*}$iofill
place_io_fill -row IO_SOUTH {*}$iofill
place_io_fill -row IO_WEST {*}$iofill
place_io_fill -row IO_EAST {*}$iofill

connect_by_abutment

place_bondpad -bond bondpad_70x70 sg13g2_IOPad* -offset {5.0 -70.0}

remove_io_rows
```

## Constraints simplificado
```
current_design SG13G2Top
set_units -time ns -resistance kOhm -capacitance pF -voltage V -current uA
set_max_fanout 8 [current_design]
set_max_capacitance 0.5 [current_design]
set_max_transition 3 [current_design]
set_max_area 0

set_ideal_network [get_pins sg13g2_IOPad_io_clock/p2c]
create_clock [get_pins sg13g2_IOPad_io_clock/p2c] -name clk_core -period 20.0 -waveform {0 10.0}
set_clock_uncertainty 0.15 [get_clocks clk_core]
set_clock_transition 0.25 [get_clocks clk_core]

set clock_ports [get_ports { 
	io_clk_PAD 
}]
set_driving_cell -lib_cell sg13g2_IOPadIn -pin pad $clock_ports
```