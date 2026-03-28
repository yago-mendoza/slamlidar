# Subsistema de Rotacion -- Plataforma Rotatoria

Documento de referencia consolidado sobre el subsistema de PLATAFORMA ROTATORIA del proyecto SLAM LiDAR de Subterranean Systems (Tyler, GitHub: tthom289). Toda la informacion ha sido extraida de transcripciones de video, comentarios de la comunidad, publicaciones de Instagram, READMEs de repositorios GitHub, e inferencias de imagenes.

---

## Concepto

El VLP-16 tiene 40 grados de campo de vision vertical (16 haces laser apilados verticalmente cubriendo +/-15 grados desde el centro optico horizontal). Para mapeo volumetrico de cuevas, esto es insuficiente: no se ve el techo ni el suelo directamente. La solucion consiste en anadir un segundo eje de rotacion al sensor, de modo que los 16 haces barran todos los angulos: techo, suelo, paredes -- cobertura esferica completa desde una sola posicion.

El desafio es hacerlo de forma mecanicamente fiable, electronicamente estable y suficientemente precisa para que el de-skewing de la nube de puntos y los algoritmos SLAM puedan aprovechar los datos. Simple en concepto, pero conseguir que funcione realmente llevo bastante tiempo.

El VLP-16 gira internamente a ~600 RPM. La plataforma rotatoria anade un eje externo perpendicular que gira todo el sensor a ~20 RPM. La combinacion de ambas rotaciones convierte el corte horizontal de 40 grados en cobertura esferica completa.

**Cadena de datos completa del subsistema:**
VLP-16 (captura puntos) -> Ethernet -> Slip ring -> Raspberry Pi 5 (graba bag ROS 2) <- Serial UART <- Teensy 4.0 <- I2C <- AS5600 (lee angulo del eje) + SimpleFOC (controla motor 4015)

---

## Eje de rotacion -- Evolucion

| Parametro | Prototipo inicial | Version actual |
|---|---|---|
| Material | Aluminio | Aluminio |
| Diametro exterior | 3/4" (19 mm) | 1" (25.4 mm) |
| Tipo | Solido | Hueco (permite paso del conector GX12) |

**Razon del cambio:** Se aumento el diametro para poder pasar el conector GX12 del VLP-16 a traves del interior del eje, eliminando la necesidad de cables externos expuestos durante la rotacion. Tyler uso ese conector porque el sensor comprado ya lo traia instalado y no queria desmontarlo para recablearlo. El eje atraviesa la pared lateral de la caja de electronica horizontalmente.

---

## Rodamientos

| Parametro | Prototipo inicial | Version actual |
|---|---|---|
| Tipo | Impresos en 3D con bolas de acero BB | Rodamientos de bolas reales |
| Diametro exterior | -- | 2" (50.8 mm) |
| Diametro interior | -- | 1" (25.4 mm) -- encaja sobre el eje |
| Instalacion | Manual | Prensado con martillo grande y vaso de llave (press-fit) |

**Alojamiento:** Los rodamientos se alojan en un soporte/housing impreso en 3D (gris/blanco, probablemente PLA). La fijacion se realiza con pernos pasantes y tuercas hexagonales.

**Tapa de rodamiento (bearing cap):** Los pernos fueron cortados mas cortos con una sierra para mejor ajuste. Los mismos pernos de la bearing cap se reutilizan para fijar el motor.

**Sugerencia comunitaria -- @tullgutten:** Anadir soporte de rodamiento en el extremo libre para reducir la carga en el rodamiento y el motor.

**Sugerencia comunitaria -- @____________________________.x (respuesta a tullgutten):** Imprimir el montaje del motor y los soportes de rodamiento del eje principal como una sola pieza para eliminar problemas de alineacion. Podria requerir girar el motor 180 grados y anadir un rodamiento adicional.

---

## Copa/horquilla -- Soporte del sensor (PIEZA MAS ITERADA de todo el proyecto)

La copa es la pieza impresa en 3D que sostiene el VLP-16 durante la rotacion. Se prensa sobre el eje. Tyler la describe como "probably one of the most iterated parts" del proyecto completo. Se documentan visualmente al menos 4-5 versiones diferentes en multiples colores y materiales sobre el cutting mat del taller.

### Tabla de TODAS las iteraciones documentadas

| Version | Material | Color | Forma | Resultado |
|---|---|---|---|---|
| Varias tempranas | ASA | Negro | Cuenco cerrado cilindrico | FALLO: se partian por las lineas de capa (delaminacion) |
| Varias tempranas | PETG | Varios colores | Cuenco cerrado cilindrico | FALLO: se partian por las lineas de capa (delaminacion) |
| Intermedia | PLA | Gris/blanco | Cuenco cerrado cilindrico | Funcional pero se deforma con el calor del motor |
| Actual | PLA/PETG | Naranja | Horquilla abierta en U (U-fork) | Funcional |

### Ventajas del diseno en U (horquilla) vs copa cerrada

- Mejor ventilacion: menos acumulacion de calor del motor
- Menos masa = menos momento de inercia = mas facil de balancear
- Mejor acceso al conector del sensor
- Reduce tensiones en lineas de capa y masa total

### Desventaja del diseno en U

- Menor proteccion mecanica del sensor

### Sugerencias comunitarias para contrapeso en la copa

- **@rassulli4532:** Anadir contrapeso al lado deficiente del LiDAR para mantener el centro de masa alineado con el eje de rotacion. Esto reduciria la necesidad de deskewing agresivo.
- **@stocki_esy625:** Mismo concepto: usar un contrapeso pequeno para mantener todo balanceado al mover el laser al eje de rotacion.
- **@Stc442:** Pregunta si se ha considerado un contrapeso en el montaje del laser para alinear el laser con el eje rotacional.

### PROBLEMA DE DEFLEXION

Sin soporte adicional, el peso del sensor (aunque no pesa mucho) creaba deflexion en la copa. Tyler paso un par de dias diagnosticando lo que pensaba que era runout del eje, cuando en realidad era simplemente flexion de la copa. Se anadieron soportes simples que, aunque no son ideales para el ruteo de cables, resuelven el problema.

Cita de Tyler (Instagram): "I spent a couple days fighting what I thought was runout issues in the shaft, which actually turned out to be just deflection of this cup here. So I added some simple supports here. It's not ideal for the wire routing, but it gets the job done and helped reduce all of the issues I had there."

### LECCION APRENDIDA CON MATERIALES

Los materiales de ingenieria (ASA, PETG) que serian ideales por resistencia termica y mecanica fallan por delaminacion en las lineas de capa en esta geometria especifica. PLA funciona mecanicamente pero se deforma con el calor. El rediseno de copa cerrada a horquilla en U reduce tensiones en lineas de capa y masa total.

---

## Consejos de impresion 3D de la comunidad

### @kylek29 -- Adhesion de capas para ABS/ASA

Para evitar que ABS/ASA se separe en las lineas de capa:
- Imprimir en el rango alto de temperatura
- Reducir el ventilador de enfriamiento
- Aumentar los limites de tiempo por capa

Para piezas que necesitan resistencia estructural y no se puede cambiar la orientacion de impresion:
- Anadir ranuras o tubos que atraviesen las lineas de capa
- Rellenarlas con una pasta de acetona + virutas del mismo filamento
- Insertar un "plug" correspondiente -- la acetona crea una union permanente
- Tambien se pueden aplicar placas externas impresas en plano con el mismo metodo

### @holski77 -- Piezas delgadas en ASA

- Usar *draft shields* (escudos de corriente)
- Si no funcionan bien: anadir placas delgadas paralelas a la cama de impresion unidas a la pieza
- Con soportes activados y las placas bien posicionadas, se puede "enjaquetar" las secciones delgadas en soportes, manteniendolas calientes el tiempo suficiente para que las capas se adhieran bien
- Despues retirar los soportes y cortar las placas de 0.5-1 mm

**Respuesta de Tyler (@9nl):** Va a probar la tecnica sugerida por @holski77.

### @TaskerTech

PLA no es adecuado para esta aplicacion. Tyler ya lo reconoce en el video y lo usa por rapidez de prototipado.

### @drewlarson65 -- Precision y materiales avanzados

- Se necesita mas precision de la que ofrecen los materiales usados actualmente
- Recomienda impresion 3D en metal o plasticos de mayor rendimiento como PPS (polisulfuro de fenileno)
- Para rigidez tipo maquina-herramienta, el acero es preferible al aluminio (a menos que el aluminio sea muy grueso)
- Reducir tamano ("go mini") para reducir masa
- Si se balancea bien y es suficientemente rigido, se puede aumentar la velocidad de muestreo/RPM
- Los agujeros desalineados probablemente se deben a sobre-extrusion o mala calibracion de la impresora

---

## Offset del centro optico

| Parametro | Valor |
|---|---|
| Offset entre centro optico del VLP-16 y eje de rotacion | ~5 mm |
| Configuracion ideal | 0 mm (alineados -- de-skewing significativamente mas facil) |
| Razon del offset | Con offset 0, la masa rotacional quedaba desbalanceada causando vibraciones e inestabilidad |

Tyler quiso inicialmente alinear el centro optico del laser con el eje de rotacion, lo que haria el de-skewing de la nube de puntos mucho mas facil. Pero al hacerlo, la masa rotacional quedaba desbalanceada y causaba mas problemas. El offset de 5 mm fue el mejor compromiso entre balance dinamico y simplicidad del de-skewing.

**Cita de Tyler (Instagram):** "Ideally, the rotation axis runs straight through the optical center of the VLP-16... clean math, clean point cloud. But the cup geometry pushed the COG far enough off-axis that vibration and imbalance made that impossible. Had to offset the axis, which means accounting for that in the SLAM pipeline. Engineering is just a series of compromises."

**Cita de Tyler (Video 3):** "If your rotation axis isn't through the optical center, your point cloud map breaks."

El offset de 5 mm se contabiliza en el algoritmo de de-skewing. En el codigo (`offline_deskew.py`), existe un parametro `ROTATION_CENTER` (default `[0,0,0]`) que define el offset del centro optico del LiDAR respecto al eje de rotacion.

**Comentario comunitario -- @taktoa1:** Si se esta haciendo SLAM, el offset rotacional no deberia importar tanto, ya que SLAM tiene que estimar la pose del LiDAR de todos modos. No es como si se dejara girar 360 grados y luego se moviera a la siguiente posicion de captura.

---

## Evolucion del motor

| Parametro | Motor original | Motor actual |
|---|---|---|
| Modelo | 2804 (GM2804) | 4015 |
| Tipo | BLDC gimbal | BLDC gimbal |
| Diametro estator | 28 mm | 40 mm |
| Torque relativo | Base | ~3x mas que el 2804 |
| Generacion de calor | Mayor (causaba deformacion constante de piezas PLA) | Menor que el anterior |

**Razon del cambio:** El motor 2804 generaba demasiado calor, deformando las piezas impresas en PLA cercanas que habia que reemplazar constantemente. Tyler: "The old one would generate so much heat that it would melt my gears, and they would get so warped that it wouldn't even spin anymore." El 4015 tiene mas torque con menor generacion de calor: "definitely overkill in the size, but the control is so much better."

**Montaje:** El motor esta sobre un soporte naranja impreso en 3D. El modelo CAD proporcionado por el fabricante no coincidia exactamente con las perforaciones reales -- hay algun agujero que no se pudo atornillar. Los agujeros de montaje tampoco alineaban perfectamente, pero con dos tornillos es suficientemente robusto y no se mueve. Cuerpo cilindrico negro con cables de fase de colores (rojo, blanco, verde, amarillo) que van al driver SimpleFOC.

**Nota del repositorio GitHub (vlp16-spin-controller):** La descripcion "About" menciona "GM2804 BLDC + AS5600 encoder with 5:1 reduction, driving continuous 40 RPM rotation" -- esto refleja el prototipo original con motor 2804 y ratio 5:1, antes de la actualizacion al motor 4015 y ratio 3:1.

---

## Transmision por correa

| Parametro | Valor |
|---|---|
| Correa | 2GT, 200 mm, 6 mm de ancho |
| Engranaje del sensor (eje) | 60 dientes |
| Engranaje del motor | 20 dientes |
| Ratio de reduccion | 3:1 |
| Velocidad de rotacion resultante | ~20 RPM |

**Diseno:** Incluye espacio para ajuste de tension de la correa (el motor se puede deslizar para tensar). Capturas para tuercas en la parte inferior facilitan el montaje.

**Nota de montaje:** Tyler se olvido de instalar la correa antes de ensamblar y tuvo que desmontar todo para ponerla.

**Nota historica:** El repositorio vlp16-spin-controller menciona un ratio 5:1 en la descripcion original, lo que indica que el ratio fue modificado de 5:1 a 3:1 durante la evolucion del diseno (posiblemente al cambiar del motor 2804 al 4015).

### Sugerencias comunitarias para la correa

**@tullgutten:**
- Reimprimir el adaptador del engranaje del motor para que use los 4 sujetadores y asegurar que este centrado (puede ser el factor del ruido que sube y baja con cada rotacion)
- Anadir soporte de rodamiento en el extremo libre para reducir carga
- Blindar el motor y ESC de los sensores con una lamina de acero para evitar interferencia electromagnetica (EMI)
- Hacer un agujero trasero para rutear cables por detras en vez de contra el sensor, y reforzar justo al lado del eje

**@____________________________.x (respuesta a tullgutten):**
- Una rueda tensora sobre la correa podria ser mas reactiva que mover el motor para tensar
- Imprimir el montaje del motor y los soportes de rodamiento del eje principal como una sola pieza para eliminar problemas de alineacion
- Podria requerir girar el motor 180 grados y anadir un rodamiento

---

## Velocidad de operacion

| Parametro | Valor |
|---|---|
| RPM de la plataforma | ~20 RPM |
| Frecuencia del LiDAR durante grabacion (banco) | 15-16 Hz |
| Frecuencia del LiDAR (version handheld avanzada) | Hasta 31 Hz |
| Frecuencia del encoder (observada en app) | 200-451 Hz (variable entre configuraciones) |

**Datos de la app "SLAM Data Recorder" en diferentes momentos:**

| Momento | LiDAR Hz | IMU Hz | Encoder Hz |
|---|---|---|---|
| Campo v1 | 23.0 / 30.2 | 199.1 | ~451.1 |
| Integracion | 17.0 > 15.0 | 100.0 | 320 |
| Handheld avanzado | 31.0 | 101.0 | ~441.1 |

Las variaciones sugieren configuraciones diferentes entre sesiones o evolucion del firmware.

---

## Balanceo

### Sugerencias comunitarias sobre contrapeso y balanceo

**@rassulli4532:** Anadir contrapeso al lado deficiente del LiDAR para mantener el centro de masa alineado con el eje de rotacion. Reduciria la necesidad de deskewing agresivo.

**@____________________________.x (respuesta):** Ademas del contrapeso, montar un IMU en la pieza giratoria y usar un osciloscopio para balancear el conjunto. Esto permitiria medir las vibraciones directamente y ajustar el contrapeso con precision.

**@stocki_esy625:** Mismo concepto: usar un contrapeso pequeno para mantener todo balanceado al mover el laser al eje de rotacion.

**@Stc442:** Pregunta si se ha considerado un contrapeso en el montaje del laser para alinear el laser con el eje rotacional.

**Nota sobre el estado actual:** Tyler opto por aceptar el offset de 5 mm en lugar de anadir contrapeso, contabilizando la diferencia en el algoritmo de de-skewing. Las sugerencias de contrapeso de la comunidad son una alternativa no implementada que podria simplificar el software a costa de mayor complejidad mecanica.

---

## Ruido del motor y PWM

**@empmachine:** El ruido del motor esta relacionado con la frecuencia de operacion del PWM. Sospecha que hay alguna constante "antigua" en el codigo para la frecuencia PWM.

**@YTuser133:** Podria ser la velocidad de conmutacion de los transistores en la placa controladora. Si operan en rango de kHz, siempre produciran ese sonido. Una placa mas cara con mayor frecuencia de conmutacion eliminaria el ruido porque el sonido estaria fuera del rango audible.

**@____________________________.x (respuesta):** Las secciones de potencia en GPUs operan en rango de MHz y aun pueden producir "coil whine." Hay mas factores involucrados que solo la frecuencia, aunque aumentarla ciertamente ayuda.

**@Spirit532:** Eliminar el ruido es simple: aumentar las frecuencias de conmutacion y del bucle de control. Tambien se reducira si se aumenta la relacion de engranajes y se hace girar el motor mas rapido.

**Contexto del sistema:** El switch manual del motor se anadio precisamente porque "con el testing constante, se deja todo alimentado pero el giro del motor hace mucho ruido."

---

## Validacion comunitaria de la rotacion

**@LordPepeetus** — Tenia un sistema sin gimbal y confirma que la combinacion de rotacion en segundo eje es buena idea. Sin rotacion adicional, la cobertura es insuficiente para mapeo completo.

---

## Disenos alternativos de rotacion propuestos por la comunidad

### @davidrollings5549 -- Espejo a 45 grados

**Propuesta:** Espejo a 45 grados rotando alrededor del sensor fijo (sensor no se mueve, solo el espejo gira). Tyler respondio a esta sugerencia.

**Analisis de ventajas:**
- El sensor queda fijo, eliminando cables rotativos (no se necesita slip ring)
- Menos masa en movimiento
- Menos vibracion

**Analisis de desventajas:**
- El espejo debe ser altamente reflectivo a 905 nm (longitud de onda del VLP-16)
- Introduce distorsion optica
- Dificil de mantener alineado
- Debe ser lo suficientemente grande para cubrir la apertura completa del VLP-16
- **Problema critico:** El VLP-16 tiene 16 haces verticales. Un espejo a 45 grados redirige todos los haces al mismo plano, perdiendo la resolucion angular vertical. Esto elimina la ventaja principal del sensor de 16 canales.

### @projectSurya-r4h -- Tornillo de avance con stepper

**Propuesta:** Tornillo de avance (lead screw) con motor stepper para obtener cortes horizontales a cada nivel de altura y luego hacer mesh de todas las rebanadas.

**Analisis:** Da movimiento vertical lineal, no rotatorio. Funciona para un LiDAR 2D de canal unico. Con el VLP-16 (que ya tiene 16 canales verticales), la rotacion en un segundo eje es mas eficiente: barre el volumen completo por cada revolucion.

### @Scrogan -- Pi y bateria en plataforma giratoria

**Propuesta:** Incluir el Raspberry Pi y la bateria en la plataforma giratoria (eliminando cables rotativos completamente), o usar un motor con mas torque que no necesite reduccion por correa.

**Observacion adicional:** Senala que la comunidad open source traera perspectivas frescas al diseno.

### @tedzbug07 -- Pivoteo en vez de rotacion completa

**Contexto:** Tiene dos Hesai PandarXT (visualmente similares al Velodyne) y pregunto si montandolos en angulos diferentes podria evitar la rotacion.

**Respuesta de Tyler (@9nl):** No es necesaria rotacion 360 grados completa -- un pivoteo de ida y vuelta tambien funciona, y elimina la necesidad del slip ring. Esto es significativo porque el slip ring es uno de los componentes mas criticos y potencialmente problematicos del sistema.

### @eldoncurrey2124 -- WiFi en vez de cable

**Propuesta:** Eliminar el cable central usando WiFi para el transceptor y alimentar via slip ring solo la potencia. Esto permitiria rotacion completa con slip ring mas simple (solo potencia, sin datos de alta velocidad).

---

## Recomendacion de inclinacion (tilt)

### @Spirit532 -- Inclinar el puck 15-30 grados

**Razonamiento detallado:**
- Sin inclinacion: el VLP-16 cubre de -15 a +15 grados. Los 16 haces estan concentrados en esa banda. Una porcion significativa del arco apunta directamente al marco del escaner y al operador, desperdiciando datos.
- Con inclinacion de 15-30 grados: los haces barren una banda diferente en cada posicion rotacional, maximizando la cobertura angular unica. Cada posicion rotacional contribuye datos mas diversos.
- Sin inclinacion: redundancia significativa en los datos.
- Con inclinacion: datos mucho mas utiles.

**Nota adicional de @Spirit532:** Para escaneos lentos, la falta de inclinacion tambien hace que las posiciones de escaneo sean menos aleatorias.

**Estado de implementacion:** Fotos recientes de Tyler muestran que implemento esta recomendacion. Se observa una inclinacion de aproximadamente 15-30 grados del sensor respecto al eje de rotacion en las imagenes del sistema.

---

## Densidad del escaneo

**@DigitalRavels:** Pregunta si configurar la velocidad de escaneo a un valor que no divida limpiamente 360 grados (como 17) tendria algun efecto de suavizado en el banding (bandas visibles) a distancias lejanas. La idea es que una velocidad no-divisora haria que las lineas de escaneo se desplacen ligeramente en cada rotacion, rellenando los huecos naturalmente.

**@antospin4004:** Pregunta si hay funcion de "append" para que mientras captura, procese y agregue datos a la visualizacion en tiempo real. Tambien pregunta si se varia la velocidad de escaneo para aumentar la densidad de puntos desde ciertos angulos.

---

## Control del motor -- SimpleFOC y Teensy

### Teensy 4.0

| Parametro | Valor |
|---|---|
| Modelo | Teensy 4.0 (PJRC) |
| Rol | Controlador del motor BLDC via SimpleFOC + lectura del encoder AS5600 + transmision de datos de posicion angular al Pi 5 |
| Comunicacion con Pi 5 | Serial UART (TX1/RX1) -- transmite datos del encoder |
| Comunicacion con encoder | I2C (SDA/SCL) -- lee posicion angular del AS5600 |
| Comunicacion con motor | 3 senales PWM (IN1, IN2, IN3) + Enable (EN) -> driver SimpleFOC -> fases del motor |
| Alimentacion | Power bank portatil USB (separada del resto del sistema) |

**Razon de alimentacion separada:** Se conecta frecuentemente al laptop para cambios de firmware, tuning del encoder, o ajustes de la funcion de rotacion. Mantener alimentacion independiente facilita este proceso.

### Encoder magnetico AS5600

| Parametro | Valor |
|---|---|
| Modelo | AS5600 |
| Tipo | Encoder magnetico rotatorio absoluto |
| Resolucion | 12 bits (4.096 posiciones/revolucion = ~0.088 grados por paso) |
| Protocolo | I2C (SDA, SCL) |
| Frecuencia de reporte | ~200-451 Hz (variable entre configuraciones observadas) |
| Cables | 4: VCC (rojo), GND (marron/oscuro), SDA (verde), SCL (amarillo) via conector JST blanco |

A 20 RPM con resolucion de 0.088 grados, proporciona mas que suficiente resolucion angular para el de-skewing de la nube de puntos.

### SimpleFOC V1 -- Driver del motor

| Parametro | Valor |
|---|---|
| Modelo | SimpleFOC V1 (driver board del proyecto open source SimpleFOC) |
| Tipo | Driver de potencia para motor BLDC |
| Tamano | ~25-30 mm x 20-25 mm (muy compacto) |
| Fijacion | Cinta de doble cara pegada al interior de la caja |
| Alimentacion | 12V conmutada (a traves del switch manual) |

### Carrier Board -- TeensyFOC-Carrier

| Parametro | Valor |
|---|---|
| Repositorio | github.com/tthom289/TeensyFOC-Carrier |
| Diseno | Tyler (tthom289), logo "TTHOM" con engranaje |
| Software de diseno | KiCad (open source) |
| Fabricante PCB | JLCPCB |
| Capas | 2 |

**Razon de existencia:** Tyler empezo con jumper wires y breadboards; la complejidad del cableado se convirtio en un problema. "When your prototype wiring becomes the problem, you design it out."

**Conectores de la carrier board:**
- Socket DIP hembra para Teensy 4.0
- Al menos 3 conectores JST-XHP blancos (alimentacion/Pi, serial TX/RX al Pi, encoder/motor)
- Header compatible con SimpleFOC motor driver
- Interfaz I2C para AS5600
- Soporte para paso de cables del slip ring
- GPIO no usados expuestos para uso futuro

**Firmware:** Repositorio `vlp16-spin-controller` (github.com/tthom289/vlp16-spin-controller). C++ 100%, licencia MIT. Ultimo cambio documentado: "Switch to open-loop velocity control with Pi5 UART output."

---

## Slip ring

| Parametro | Valor |
|---|---|
| Fabricante | slipring.cn |
| Modelo | LPC-18 (capsule slip ring) |
| Circuitos | 18 independientes (Tyler usa solo 3-4; el resto cortados) |
| Corriente por circuito | Tipicamente 2A senal, algunos canales 5-10A potencia |
| Velocidad max rotacion | Tipicamente 250-300 RPM (sistema opera a ~20 RPM) |
| Vida util tipica | >10 millones de revoluciones |
| Precio | ~$30-50 en eBay |
| Busqueda eBay | "Gigabit Ethernet Slip Ring 1000Base-T 1 Cable Power Signal Flange RJ45 Joint" (confirmado por Tyler) |

**Senales que pasan por el slip ring:**

| Senal | Tipo | Descripcion |
|---|---|---|
| Ethernet | Gigabit 1000Base-T | Datos de nube de puntos VLP-16 -> Pi 5 |
| Power | 12V DC | Alimentacion del VLP-16 |
| Ground | GND | Masa |
| GPS/PPS (x2) | Senal digital | No se usa GPS real, pero si PPS timing con RTC para sincronizacion precisa |

**Nota sobre circuitos:** Tyler menciona 12 circuitos en el video, pero el modelo mostrado es LPC-18 (18 circuitos). Posible margen extra o referencia generica.

**Criticidad para Ethernet:** El VLP-16 transmite datos por Gigabit Ethernet. La integridad de senal a traves de contactos rotativos deslizantes es no trivial -- requiere resistencia baja y consistente con minimo ruido electrico. Slip rings genericos no son suficientes; se necesita uno especifico de Gigabit Ethernet. Un slip ring de mala calidad causaria perdida de paquetes = frames de nube de puntos perdidos o corruptos.

**Duracion:** Tyler confirmo que con el slip ring el sistema podria funcionar indefinidamente, o hasta que el slip ring se desgaste, pero "seguramente algo mas fallaria antes."

**Cableado:** Los cables estan soldados por detras del conector y rellenos con pegamento caliente para evitar que las juntas de soldadura se rompan por vibracion. "No es lo mas bonito, pero funciona." El slip ring tiene malla protectora (wire loom) para evitar el roce del cable.

**Tips comunitarios sobre pegamento caliente:**
- **@EvilSpyBoy:** El alcohol isopropilico despega el hot glue de las superficies de forma limpia. Funciona en impresiones 3D y probablemente en otras superficies.
- **@DonWRII:** Advertencia: algunos tipos de hot glue son parcialmente conductivos y pueden anadir ruido a senales. Lo aprendio en reparaciones USB.

---

## Proceso de encendido y operacion

1. Conectar la bateria de 12V -- esto arranca la Raspberry Pi, el HMI (app en smartphone), la IMU y el sensor LiDAR
2. Conectar la alimentacion separada del Teensy/motor controller
3. Asegurar que el interruptor del motor esta encendido
4. El sistema inicializa y comienza a girar a ~20 RPM
5. Esperar a que todo arranque en el lado del Pi (~1-2 minutos, "todo en verde")
6. Colocar el sistema en posicion (vertical u horizontal segun aplicacion)
7. Iniciar grabacion desde la app "SLAM Data Recorder"

**Datos de grabacion tipicos:**
- Escaneo estatico de 2 minutos = ~400 MB comprimido, ~1 GB descomprimido
- ~21 millones de puntos, submuestreados a ~13.75 millones
- ~206 GB de almacenamiento disponible

**Alimentacion alternativa:** Cuando se trabaja en interior, el sistema se alimenta de una fuente de alimentacion DC en vez de la bateria, lo que resulta mas comodo para testing prolongado.

---

## Cronologia de evolucion del hardware de rotacion

| Fase | Motor | Eje | Copa | Ratio | Notas |
|---|---|---|---|---|---|
| Banco | 2804 | 3/4" solido | Gris/blanco cerrado, PLA | 5:1 | Rodamientos 3D printed con bolas BB |
| Integracion | 4015 | 1" hueco | Varias iteraciones | 3:1 | Rodamientos reales, caja gris aluminio |
| Campo v1 | 4015 | 1" hueco | Naranja, diseno en U | 3:1 | Primera salida exterior |
| Handheld | 4015 | 1" hueco | Naranja, diseno en U, inclinado | 3:1 | Empunaduras ergonomicas, caja negra compacta |

**Patron general:** Cada iteracion mejora simultaneamente robustez mecanica, ergonomia y capacidad de software. Tyler aplica la filosofia de perfeccionar el de-skewing estatico antes de volver a integrar SLAM: "I intentionally took a step back from the SLAM integration to really nail this part down. If the points aren't landing where they should, nothing downstream really matters."

---

## Preguntas frecuentes de la comunidad sobre la rotacion

**Sobre cables enredados:**
- **@KIWIvolt, @bluegizmo1983, @whssavy, @readyrepairs, @derjansan9564, @thomasharper9087, @CrazyCanuckCH, @12mkamran, @MicroRobo, @arias8185:** Multiples usuarios preguntaron como los cables no se enredan al girar. La respuesta es siempre el slip ring.

**Sobre el cable pasando bajo el eje:**
- **@ImaginationToForm:** Preocupacion por el cable rojo que pasa bajo el eje blanco giratorio. Pregunta si el material blanco es abrasivo y podria desgastar el cable.

**Sobre precision de la rotacion por correa:**
- **@manfredthuering9745:** Pregunta sobre la precision de posicionamiento de la rotacion por correa y su control.

**Sobre rediseno completo:**
- **@felderup:** Sugiere un rediseno completo del mecanismo rotador.
- **@Andy-js5jy:** Sugiere hacer una PCB madre para reducir el desorden de cables.

---

## Relacion con el pipeline de software (de-skewing)

La plataforma rotatoria genera los datos angulares necesarios para el de-skewing. El encoder AS5600 proporciona el angulo de rotacion de la plataforma, que se publica como topic ROS 2 (`/rotating_platform/angle`, tipo `std_msgs/Float64`, en grados).

**Parametros de calibracion en `offline_deskew.py`:**

| Parametro | Default | Descripcion |
|---|---|---|
| `ROTATION_AXIS` | `'x'` | Eje alrededor del cual rota la plataforma |
| `ANGLE_OFFSET_DEG` | `184.0` | Offset de calibracion del angulo cero en grados |
| `ROTATION_CENTER` | `[0,0,0]` | Offset del centro optico del LiDAR respecto al eje de rotacion |
| `INVERT_ROTATION` | `False` | Invertir direccion de rotacion |
| `ENCODER_TIME_OFFSET_MS` | `0.0` | Offset temporal entre reloj del encoder y reloj del LiDAR |
| `MOUNT_RPY_DEG` | `[0,0,0]` | Roll / pitch / yaw del montaje del LiDAR en grados |
| `MOUNT_AXES` | `['+x','+y','+z']` | Remapeo de ejes para el montaje del LiDAR |

El ICP merge (`icp_merge.py`) aplica alineacion ICP encima del de-skewing para compensar imperfecciones mecanicas -- runout del eje, wobble de los rodamientos, u otras tolerancias de hardware que causan que las mitades opuestas (180 grados) del escaneo no se alineen perfectamente incluso despues de la correccion por movimiento.
