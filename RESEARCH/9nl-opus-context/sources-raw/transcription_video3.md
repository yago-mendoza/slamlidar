# Video 3 — La plataforma rotatoria: diseño, construcción y pruebas

**Link:** https://www.youtube.com/watch?v=JH7F9xr6yL0

## [0:00] El problema y la solución

El VLP-16 es un gran sensor, pero tiene 40 grados de campo de visión. Para la mayoría de aplicaciones es suficiente, pero para mapear cuevas es una limitación real. La solución es conceptualmente simple: si tenemos 40° de campo de visión del sensor y lo rotamos sobre un eje adicional, podemos conseguir cobertura esférica completa del área que nos rodea. La parte complicada es hacerlo de forma mecánicamente fiable, electrónicamente estable y suficientemente precisa para que el de-skewing de la nube de puntos y los algoritmos SLAM puedan aprovechar los datos.

## [0:38] Del prototipo al diseño actual

El diseño original fue un proof of concept mínimo viable. Usaba un eje de aluminio de 3/4" con rodamientos impresos en 3D hechos con bolas de acero tipo BB. Funcionó bien para demostrar que la idea era viable, pero después evolucioné a algo mucho más refinado: rodamientos de bolas reales con 2" de diámetro exterior y 1" de interior, que encajan sobre el eje (bueno, hay que meterlos a presión con un martillo grande y un vaso de llave). Luego está el engranaje de 60 dientes, también prensado sobre el eje. Y la copa donde se asienta el sensor, igualmente prensada. Descubrí que tenía que imprimir esta copa en PLA porque con cualquier otro material (ASA, PETG) siempre se partía por las líneas de capa.

La razón de pasar de un eje de 3/4" a 1" fue poder pasar un conector GX12 a través del interior del eje. Usé ese conector porque el sensor que compré ya lo traía instalado y no quería desmontarlo para recablearlo.

## [2:10] Cableado y slip ring

El sistema tiene un slip ring con malla protectora para evitar el roce del cable. A través de él pasa la línea Ethernet que va del sensor a la Raspberry Pi, la alimentación (positivo y tierra) y dos señales GPS. En realidad no usamos GPS, pero sí utilizamos el PPS timing con un reloj de tiempo real para conseguir sincronización precisa entre todos los subsistemas: la IMU, los puntos del Velodyne y los datos del encoder. El conector fue elegido específicamente porque pasa a través del eje hueco. Los cables están soldados por detrás y rellenos con pegamento caliente para evitar que las juntas de soldadura se rompan. No es lo más bonito, pero funciona.

## [4:04] El motor y la transmisión

Se usa un motor 4015 con soporte impreso en 3D. El modelo CAD proporcionado para el motor no coincidía exactamente con las perforaciones reales, así que hay algún agujero que no pudo atornillarse, pero funciona. Lo mismo con los agujeros de montaje: no alineaban perfectamente, pero con dos tornillos es bastante robusto y no se va a ningún sitio. La transmisión usa una correa 2GT de 200 mm y 6 mm de ancho, con un engranaje de 60 dientes en el eje del sensor y uno de 20 dientes en el motor, lo que da una relación de reducción 3:1. El diseño incluye espacio para ajuste de tensión de la correa y capturas para las tuercas en la parte inferior para facilitar el montaje. Tiene conector para el encoder y para la alimentación del motor. Nota: se olvidó de poner la correa antes de montar y hubo que desmontar todo para instalarla. Los pernos de la tapa del rodamiento se cortaron más cortos con una sierra para que encajasen mejor.

Un problema encontrado: las piezas impresas en 3D cerca del motor se deforman por el calor. El motor anterior generaba aún más calor, y las piezas en PLA había que reemplazarlas constantemente por deformación térmica. La placa Simple FOC se fijó con cinta de doble cara.

## [5:24] Alimentación

Todo se alimenta con una batería de ion de litio de 12V, excepto el Teensy, que todavía usa un power bank portátil por comodidad (se conecta frecuentemente al portátil para cambios de firmware, tuning del encoder, o ajustes de la función de rotación). Hay un interruptor manual para el motor porque con todo el testing se deja el sistema encendido y el giro constante hace mucho ruido.

## [7:45] El tren motriz completo

Correa 2GT de 200 mm y 6 mm de ancho. Engranaje de 60 dientes en el sensor, 20 dientes en el motor: ratio 3:1. Velocidad configurada a unos 20 RPM con el LiDAR grabando a 15 Hz.

## [8:07] La placa carrier y el Teensy

Una placa carrier simple que rompe algunos conectores y aloja el Teensy 4.0. La razón de esta placa es que el Teensy es tan pequeño que colocarlo suelto dentro de la caja sería arriesgado. La placa mantiene todo más limpio y reduce el cableado. Un conector va a la Raspberry Pi 5 y transmite los datos del encoder. Otro conector alimenta los datos del encoder directamente al Teensy. También está la placa Simple FOC con alimentación de 12V conmutada y los cables que van al motor.

## [9:29] El eje de rotación y el balanceo

Inicialmente la idea era alinear el centro óptico del láser con el eje de rotación, lo que haría el de-skewing de la nube de puntos mucho más fácil. Pero al hacerlo, la masa rotacional quedaba desbalanceada y causaba más problemas. Finalmente el offset quedó en unos 5 mm, que fue lo mejor que se pudo conseguir en términos de equilibrio.

La copa rotacional es probablemente la pieza más iterada de todo el proyecto: muchas versiones diferentes con distintos materiales y alturas. Otro problema importante fue que sin un soporte adicional, el peso del sensor (aunque no es mucho) creaba deflexión en la copa. Pasé un par de días peleando con lo que pensaba que era runout del eje, y resultó ser simplemente flexión de la copa. Se añadieron soportes simples que, aunque no son ideales para el ruteo de cables, resolvieron el problema.

## [10:53] Prueba estática en el patio

Proceso de encendido: conectar la batería de 12V (esto arranca la Raspberry Pi, el HMI, la IMU y el sensor LiDAR). Luego conectar la alimentación separada del Teensy/motor controller, asegurar que el interruptor está encendido, y el sistema inicializa y comienza a girar a unos 20 RPM. Se espera a que todo arranque en el lado del Pi (todo en verde). Se coloca el sistema en vertical y se inicia la grabación.

El LiDAR graba a 16 Hz, la IMU a 200 Hz (no necesaria para escaneo estático pero se deja encendida), los encoders a 200 Hz. Quedan unos 200 GB de almacenamiento disponible para grabar. Un escaneo de 2 minutos generó 400 MB comprimidos (poco más de 1 GB descomprimido). Cuando se trabaja en interior, el sistema se alimenta de una fuente de alimentación en vez de la batería, lo que resulta más cómodo.

## [13:45] Procesamiento y resultados

Se conecta al Pi 5 por SSH, se accede al directorio donde se guardan los bags crudos. La grabación contiene datos de IMU, ángulo de plataforma, velocidad de plataforma y nube de puntos. Para el escaneo estático se necesita todo excepto los datos de IMU. Tras descargar la grabación, se ejecuta el ICP merge que genera un archivo de nube de puntos (hace dos pasadas). El escaneo de 2 minutos produjo casi 21 millones de puntos, submuestreados a 13.75 millones — más de 13 millones de puntos en 120 segundos. El resultado es impresionante: se distinguen claramente un columpio colgando de un árbol, cobertizos, la línea de la valla, las hendiduras en el suelo dejadas por un camión de hormigón, la cubierta de la parrilla, una nevera Yeti, líneas eléctricas y árboles con gran detalle.

## [16:04] Fusionando múltiples escaneos

Se pueden abrir dos archivos PCD simultáneamente. Con F2 se pueden ver los escaneos de forma independiente. En este caso, un escaneo de la parte delantera de la casa y otro de la trasera, con poca geometría compartida entre ambos. Se identifican puntos compartidos: la línea de la valla, el lateral de la casa, el pico del tejado. Se marcan tres pares de puntos correspondientes entre ambos escaneos. Se pulsa G para alinear 3+ pares de puntos (no I, que es otra herramienta de alineación diferente; inicialmente se confundieron). El programa se congela unos segundos mientras procesa. El resultado no es perfecto pero es bastante bueno: los postes de teléfono dan buenas referencias a larga distancia y los laterales de la casa coinciden bien, con puntos superponiéndose. Se pueden fusionar los colores y en menos de cinco minutos de trabajo se tiene la casa completa, frente y laterales.

## [20:26] Cierre

Ese es todo el stack: transmisión por correa, firmware FOC y de-skewing de nubes de puntos. Las pruebas en banco y en el patio han ido geniales. Aunque me he alejado un poco de usar un algoritmo SLAM para centrarme más en escaneos estáticos y de-skewing de nubes de puntos, todo está funcionando bien. Los repositorios de GitHub para el de-skewing de nubes de puntos y el controlador del motor están compartidos en la descripción. El objetivo es seguir documentando y tener toda la información online y completamente open source.
