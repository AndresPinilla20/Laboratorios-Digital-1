# Mi primer diseño

## 0. Problema a analizar  

Para este laboratorio, se nos planteó una problemática específica: diseñar un sistema eléctrico para una finca capaz de conmutar automáticamente entre dos fuentes de energía según diversos factores. Estas fuentes incluyen:  

1. **Red eléctrica de la casa** – La principal fuente de suministro.  
2. **Banco de baterías recargadas con energía solar** – Alternativa de respaldo.  

El objetivo es garantizar un suministro continuo de energía, optimizando el uso de la electricidad y asegurando una transición eficiente entre ambas fuentes.

## 1. Proceso del dominio comportamental  

Para abordar la solución del problema, el grupo identificó las entradas que controlarían el sistema y las salidas que servirían como indicadores de las consecuencias de cada entrada. Estas salidas se reflejan en un **tablero de control**.  

Después de un análisis detallado, se definieron las siguientes **entradas y salidas**:  

### Entradas y Salidas del Sistema  

<p align="center">
    <strong>Tabla 1. Entradas y salidas del sistema</strong>  
</p>  
<p align="center">
    <img src="Entradas y salidas.png" width="40%">
</p>  

### Lógica del sistema  
Tras definir las entradas y sus funciones, se elaboró una **tabla de verdad** para establecer cómo variará cada salida en función de las combinaciones de las entradas. Esta tabla permite estructurar el comportamiento del sistema según las diferentes condiciones operativas.  

<p align="center">
    <img src="Lab-2/Tabla de verdad.png" width="60%">
</p>
<p align="center">
    <strong>Tabla 2. Tabla de Verdad </strong>
</p>

A partir de la tabla de entradas y salidas, se generaron los respectivos **mapas de Karnaugh** para definir la lógica combinacional de cada una de las salidas (**Q0.1** a **Q0.5**).  

Una vez determinada la expresión lógica de cada salida, se procedió a elaborar el **diagrama de flujo** del sistema completo, finalizando así la etapa del **dominio comportamental**.  

### Importancia del Diagrama de Flujo  

El **diagrama de flujo** nos permitió visualizar el comportamiento global del sistema, facilitando el análisis y la validación del diseño antes de su implementación. Gracias a esta representación gráfica, pudimos:  

- **Comprender la secuencia de operaciones** y cómo interactúan las entradas y salidas.  
- **Identificar posibles errores lógicos** antes de pasar a la fase de implementación.  
- **Optimizar la toma de decisiones**, asegurando una transición eficiente entre las fuentes de energía.  
- **Facilitar la comunicación** del diseño con otros miembros del equipo, permitiendo mejoras y ajustes antes del desarrollo final.  

### Mapas de Karnaugh  

<p align="center"><strong>Figura 1. Mapa de Karnaugh para Q0.1</strong></p>  
<p align="center"><img src="Lab-2/Karnaugh q1.png" width="60%"></p>  

<p align="center"><strong>Figura 2. Mapa de Karnaugh para Q0.2</strong></p>  
<p align="center"><img src="Lab-2/Karnaugh q2.png" width="60%"></p>  

<p align="center"><strong>Figura 3. Mapa de Karnaugh para Q0.3</strong></p>  
<p align="center"><img src="Lab-2/Karnaugh q3.png" width="60%"></p>  

<p align="center"><strong>Figura 4. Mapa de Karnaugh para Q0.4</strong></p>  
<p align="center"><img src="Lab-2/Karnaugh q4.png" width="60%"></p>  

<p align="center"><strong>Figura 5. Mapa de Karnaugh para Q0.5</strong></p>  
<p align="center"><img src="Lab-2/Karnaugh q5.png" width="60%"></p>  

### Diagrama de Flujo del Sistema  

<p align="center"><strong>Figura 6. Diagrama de flujo del sistema</strong></p>  
<p align="center"><img src="Lab-2/Digrama_De_Flujo_Dig_1-1.png" width="100%"></p>  

## 2. Proceso del dominio estructural

Una vez verificado que la lógica expresada en el diagrama de flujo y los mapas de Karnaugh coincidía, se procedió a simular el circuito completo utilizando compuertas lógicas. Este paso es fundamental porque nos permite:

- **Verificar la integración del sistema**: Comprobar que todas las compuertas interactúan correctamente y que el circuito global cumple con la lógica deseada.
- **Identificar y corregir errores**: Detectar posibles fallas en el diseño antes de pasar a la implementación física, lo cual minimiza riesgos y costos en etapas posteriores.
- **Optimizar el diseño**: Ajustar parámetros y mejorar la eficiencia del circuito, asegurando un rendimiento óptimo bajo diversas condiciones operativas.
- **Facilitar el análisis y la comunicación**: La simulación ofrece una representación visual y detallada del funcionamiento del sistema, permitiendo a todos los miembros del equipo comprender y debatir el diseño de manera efectiva.

<p align="center"><strong>Figura 7. Circuito Digital - Compuertas Logicas</strong></p> 
<p align="center">
    <img src="Lab-2/Compuertas logicas.png" width="60%">
</p>

## 3. Lenguaje Ladder

Después de validar la lógica mediante los diagramas de flujo y los mapas de Karnaugh, se procedió a implementar la solución en **lenguaje Ladder**. Esta decisión refuerza la estructura del sistema y ofrece múltiples beneficios, ya que el formato gráfico del Ladder, similar a un esquema eléctrico, facilita la comprensión y el mantenimiento para ingenieros y técnicos. Además, al ser un estándar ampliamente adoptado en la programación de PLC, garantiza la compatibilidad e integración con otros sistemas de automatización industrial, lo que resulta esencial en proyectos escalables donde se puedan incorporar nuevas funciones o realizar modificaciones sin comprometer la estabilidad del diseño. También simplifica la simulación y el análisis del comportamiento del sistema, permitiendo detectar y corregir errores en etapas tempranas del desarrollo, lo que en conjunto contribuye a un diseño robusto y preparado para futuras ampliaciones.

<p align="center"><strong>Figura 8. Diagrama Ladder</strong></p> 
<p align="center">
    <img src="Lab-2/Diagrama Ladder.png" width="100%">
</p>

## 4. Proceso del dominio físico

Para la implementación física del sistema se empleó una **FPGA** como unidad de control. En esta etapa se programaron las compuertas lógicas utilizando el software **Digital**, que facilitó la generación del código en **Verilog**. Posteriormente, se usó **Lite XL** para programar la FPGA con dicho código, dando lugar a un circuito relativamente sencillo donde se utilizaron pulsadores como entradas y LEDs como salidas. Este proceso permitió validar el diseño de manera tangible y demostrar su funcionamiento en un entorno real.


<p align="center"><strong>Figura 9. Implementación del sistema - FPGA </strong></p> 
<p align="center">
    <img src="Lab-2/circuito fpga.png" width="25%">
</p>

<p align="center"><strong>Video explicativo </strong></p> 
<p align="center">
  <a href="https://youtu.be/C3COS4yHkTs?si=Q6Wph2sFdSqt-Fu1">
    <img src="https://img.youtube.com/vi/C3COS4yHkTs/0.jpg" alt="Ver el video">
  </a>
</p>


