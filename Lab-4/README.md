# Laboratorio 4 - Máquina de Estados
En el ámbito de los sistemas digitales, las máquinas de estados representan una herramienta fundamental para el control y la gestión de procesos secuenciales, permitiendo diseñar sistemas complejos con comportamientos predecibles y estructurados. Su aplicación abarca desde sistemas embebidos hasta protocolos de comunicación, destacando por su eficiencia en la transición entre estados definidos bajo condiciones específicas. En este contexto, el presente informe documenta el diseño, implementación y validación de una máquina de estados finitos (FSM, por sus siglas en inglés) desarrollada como parte de un proyecto integrador, cuyo objetivo es controlar un sistema mediante una FPGA (Field-Programmable Gate Array).

La elección de una FPGA como plataforma de implementación se fundamenta en su flexibilidad y capacidad de reconfiguración, características que permiten emular circuitos digitales personalizados sin necesidad de fabricar hardware dedicado. Esto facilita la optimización de recursos lógicos, la paralelización de operaciones y la validación iterativa del diseño. Durante el desarrollo, se empleó la herramientaa de descripción de hardware Verilog, para modelar la máquina de estados, así como software de síntesis y simulación (Quartus) para garantizar su correcto funcionamiento antes de la programación física del dispositivo.

# Dominio Comportamental 


Dentro de la electrónica digital muchas veces existen desafíos de sistemas que necesitan tener en cuenta cuál era su función anteriormente para realizar una nueva acción, es acá donde entran las máquinas de estado, pues gracias a la integración de latch tipo D se logra realizar el guardado de un dato durante un ciclo, el cual dura según el clock implementado, esto ayuda a que la máquina pueda verificar que está realizando paso a paso y decida según sus estados y a veces entradas externas, qué debe realizar. Así dentro del contexto del presente proyecto que trata de automatizar el sistema de luminarias de determinado espacio, se ve necesario la implementación de una máquina de estados que permita establecer el tiempo que se mantedrá la salida activa al momento de que se registre o lea una señal de entrada.

## Circuito de Compuertas lógicas en Digital

<div align="center">
 
![acople](acople.png)

</div>
## Ajuste de tensión para el ADC

Dado que el ADC tiene una referencia de 5V, este valor correspondería a 255 Vp en su conversión, lo cual no es el objetivo. Para establecer la tensión adecuada, se emplea un divisor de tensión. A través de una regla de 3, se determina que la tensión deseada es 3.33 V para una entrada de 170 Vp. Dado que la tensión disponible después de la rectificación es de 7.3 V, se implementa un divisor de tensión utilizando un trimmer de 100 KΩ. La resistencia R2, donde debe caer la tensión de 3.33 V, se ajusta a 45.6 KΩ dentro del trimmer.


  $$\text{V}_{sal} = V \cdot \cfrac{R2}{R2 + R1}$$

## Estados presentes
  
  <div align="center">
   
  ![Esquematico ADC0808](Imagen/adc0808.png)
  
  </div>

Para la conversión en el ADC, primero se selecciona la entrada de medición a través del multplexor interno, configurando los selectores de dirección (ADD) a tierra, lo que  habilita la entrada 0. Luego, se establece la referencia de voltaje del ADC en 5 V, coincidiendo con su tensión de alimentación.

Las señales START y EOC se conectan en retroalimentación para sincronizar la toma y procesamiento de datos en el momento adecuado. Las líneas ALE y OE se fijan en 5 V, para garantizar una lectura continua de las entradas analógicas y mantener activas las salidas digitales del ADC. Finalmente, los datos digitalizados se envían a la FPGA, donde serán procesados y desplegados en tres displays de 7 segmentos, mostrando los valores en centenas, decenas y unidades.

## Entradas

El sistema con 4 entradas en total, dos van directamente a la máquina de estados, y los otros dos hacen parte de un multiplexor que permiten que el sistema opere de forma diferente.

## Salidas

El sistema cuenta con una única salida.

# Dominio estructural 

## Descripción en lenguaje HDL (Hardware Description Language) 

El funcionamiento del circuito ha sido implementado directamente en HDL, en la carpeta **Codigo** donde se definen distintos modulos para seguir el principio de separación de entidades, y se reune todo en el *top*.

## Divisor de Frecuencia
Este módulo reduce la frecuencia del reloj de la FPGA, que opera a 50 MHz, a una frecuencia de 1 kHz, adecuada para el ADC0808.

```verilog
module clk_divider #(
    parameter integer FREQ_IN = 50000000,
    parameter integer FREQ_OUT = 1000,
    parameter integer INIT = 0
) (
    input CLK_IN,
    output reg CLK_OUT = 0
);

  localparam integer COUNT = (FREQ_IN / FREQ_OUT) / 2;
  localparam integer SIZE = $clog2(COUNT);
  localparam integer LIMIT = COUNT - 1;

  reg [SIZE-1:0] count = INIT;

  always @(posedge CLK_IN) begin
    if (count == LIMIT) begin
      count   <= 0;
      CLK_OUT <= ~CLK_OUT;
    end else begin
      count <= count + 1;
    end
  end
endmodule
```

## Conversión de Binario a BCD
Este módulo convierte un valor binario en su representación en BCD.

```verilog
module binary_to_bcd (
    input wire [7:0] binary,
    output reg [3:0] hundreds,
    output reg [3:0] tens,
    output reg [3:0] unit
);

always @(*) begin
    hundreds = binary / 100;
    tens = (binary % 100) / 10;
    unit = binary % 10;
end

endmodule
```

## Controlador de Display de 7 Segmentos
Este módulo maneja la multiplexación y activación del display de 7 segmentos.

```verilog
module seven_segment_driver (
    input wire clk,
    input wire reset_n,
    input wire [3:0] digit0,
    input wire [3:0] digit1,
    input wire [3:0] digit2,
    output reg [6:0] segmentos,
    output reg [2:0] anodos
);

reg [1:0] contador;
reg [15:0] prescaler;
reg [3:0] digito_actual;

always @(*) begin
    case (digito_actual)
        4'h0: segmentos = 7'b1000000;
        4'h1: segmentos = 7'b1111001;
        4'h2: segmentos = 7'b0100100;
        4'h3: segmentos = 7'b0110000;
        4'h4: segmentos = 7'b0011001;
        4'h5: segmentos = 7'b0010010;
        4'h6: segmentos = 7'b0000010;
        4'h7: segmentos = 7'b1111000;
        4'h8: segmentos = 7'b0000000;
        4'h9: segmentos = 7'b0010000;
        default: segmentos = 7'b1111111;
    endcase
end

always @(posedge clk or negedge reset_n) begin
    if (!reset_n) begin
        prescaler <= 0;
        contador <= 0;
        anodos <= 3'b111;
    end else begin
        if (prescaler == 16'd50000) begin
            prescaler <= 0;
            case (contador)
                2'b00: begin
                    digito_actual <= digit0;
                    anodos <= 3'b110;
                end
                2'b01: begin
                    digito_actual <= digit1;
                    anodos <= 3'b101;
                end
                2'b10: begin
                    digito_actual <= digit2;
                    anodos <= 3'b011;
                end
            endcase
            contador <= contador + 1;
        end else begin
            prescaler <= prescaler + 1;
        end
    end
end

endmodule
```

## Módulo *top.v*
Este módulo integra todas las funciones anteriores.

```verilog
`include "seven_segment_driver.v"
`include "binary_to_bcd.v"
`include "adc_control.v"
`include "clk_divider.v"

module top (
    input wire clk,
    input wire reset_n,
    input wire [7:0] adc_data,
    input wire eoc,
    output wire adc_clk,
    output wire start,
    output wire oe,
    output wire add_a,
    output wire add_b,
    output wire add_c,
    output wire [6:0] segmentos,
    output wire [2:0] anodos
);

clk_divider clk_div_adc (
    .CLK_IN(clk),
    .CLK_OUT(adc_clk)
);

adc_control adc_ctrl (
    .clk(clk),
    .reset_n(reset_n),
    .eoc(eoc),
    .start(start),
    .oe(oe)
);

wire [3:0] hundreds, tens, unit;
binary_to_bcd bcd_conv (
    .binary(adc_data),
    .hundreds(hundreds),
    .tens(tens),
    .unit(unit)
);

seven_segment_driver display (
    .clk(clk),
    .reset_n(reset_n),
    .digit0(unit),
    .digit1(tens),
    .digit2(hundreds),
    .segmentos(segmentos),
    .anodos(anodos)
);

endmodule
```

## Asignación de Pines
Configuración de la FPGA *Cyclone IV E* en `top.qsf`.

```tcl
set_global_assignment -name FAMILY "Cyclone IV E"
set_global_assignment -name DEVICE EP4CE10E22C8
set_global_assignment -name TOP_LEVEL_ENTITY top
set_global_assignment -name PROJECT_OUTPUT_DIRECTORY build

set_location_assignment PIN_23 -to clk

set_location_assignment PIN_28 -to adc_data[0]
set_location_assignment PIN_30 -to adc_data[1]
set_location_assignment PIN_31 -to adc_data[2]
set_location_assignment PIN_32 -to adc_data[3]
set_location_assignment PIN_33 -to adc_data[4]
set_location_assignment PIN_34 -to adc_data[5]
set_location_assignment PIN_38 -to adc_data[6]
set_location_assignment PIN_39 -to adc_data[7]

set_location_assignment PIN_52 -to adc_clk

set_location_assignment PIN_127 -to segmentos[0]
set_location_assignment PIN_126 -to segmentos[1]
set_location_assignment PIN_125 -to segmentos[2]
set_location_assignment PIN_124 -to segmentos[3]
set_location_assignment PIN_121 -to segmentos[4]
set_location_assignment PIN_120 -to segmentos[5]
set_location_assignment PIN_119 -to segmentos[6]

set_location_assignment PIN_128 -to anodos[0]
set_location_assignment PIN_129 -to anodos[1]
set_location_assignment PIN_132 -to anodos[2]

set_global_assignment -name LAST_QUARTUS_VERSION "23.1std.1 Lite Edition"
```

## Diagrama RTL 

<div align="center">

 ![rtl](Imagen/rtl.png)
</div>


# Dominio Fisico 

<div align="center">

 ![circuito](Imagen/circuito.png)
</div>

## Sensor pir hc-sr501

Es un sensor infrarojo que detecta movimiento, la idea es usarlo para poder automatizar el encendido de luces, y gracias a su modo de funcionamiento con temporizador interno, es posible mantener a salida en 1, durante determinado tiempo. Su rango de alcance de lectura, va de los 3 mts hasta los 7 mts y posee dos modos de operación; un disparo o múltiples disparos. En el caso del proyecto se decide usar la función de múltiples disparos dado que va sumando el tiempo del temporizador cada vez que realiza una lectura, esto aunque una gran solución al problema del proyecto, presenta una limitante y es justamente la suma de los tiempos en lugar de un reset cada vez que registra una lectura, pues si hay mcuahs personas moviéndose, el tiempo de activación podría durar mucho tiempo. Esto es una limitante del sensor que es muy complejo de resolver, pues no posee un clock para que la fpga dirija el temporizador.
## Módulo de fotocelda

Se trata de un sensor que tiene la capacidad de dar un valor alto cuando no es capaz de leer luz, la sensibilidad a la iluminación con la que vuelve a un nivel bajo, se puede ajustar gracias a la presencia de una resistencia en el módulo. Esta entrada es la prioritaria para poder encender las luces, pues se proyecta que sea el que avise al sistema cuando es de noche, junto con el pir son las entradas que van conectadas a la máquina de estados.
## Interruptor 

En este caso el interruptor va a tener la función de actuar como un selector de un multiplexsor, el cual si tiene un valor de 0, permite que el circuito opere con la lógica programada; mientras que si tiene un valor de 1, conecta la salida directamente a una fuente de energía, emulando que las luces están siempre encendidas durante la noche.
