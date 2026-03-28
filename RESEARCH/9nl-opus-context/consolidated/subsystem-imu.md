# Subsistema IMU -- Wheeltech N100

Documento de referencia consolidado sobre la unidad de medicion inercial (IMU) del proyecto Subterranean Systems. Toda la informacion ha sido extraida exhaustivamente de las fuentes primarias: transcripciones de video, comentarios de YouTube, publicaciones de Instagram, READMEs de GitHub, e inferencias de imagenes.

---

## Especificaciones tecnicas del Wheeltech N100

| Parametro | Valor |
|---|---|
| Fabricante | Wheeltec (WHEELTEC FOISYSTEMS, fabricacion china) |
| Modelo | N100 |
| Tipo | 9 ejes (acelerometro de 3 ejes + giroscopo de 3 ejes + magnetometro de 3 ejes). Texto en la etiqueta: "9axisIMU", incluye "9轴IMU" (chino: 9轴 = 9 ejes) |
| Compatibilidad | "ROS" marcado en la etiqueta |
| Frecuencia de muestreo | 400 Hz (especificacion del fabricante), observada entre 100 y 200 Hz en la app SLAM Data Recorder (posiblemente downsampling configurable) |
| Comunicacion | USB serial via cable USB-C negro que sale de la carcasa hacia el Raspberry Pi 5 |
| Apariencia | Placa cuadrada plateada/aluminio, cuatro tornillos de montaje en las esquinas |
| Precio estimado | ~$30-50 USD |

### Frecuencias observadas en la app SLAM Data Recorder

| Momento de captura | Frecuencia IMU observada |
|---|---|
| Foto campo v1 (IMG_9449) | 199.1 Hz |
| Foto integracion (IMG_9471) | 100.0 Hz |
| Foto handheld (IMG_9458) | 101.0 Hz |

Las variaciones en frecuencia entre sesiones sugieren configuraciones diferentes o evolucion del firmware.

---

## Montaje y orientacion

La IMU esta montada en la cara superior interna de la caja de electronica, en una ventana rectangular cortada en el panel. Esta ventana forma parte de una abertura en forma de "T" invertida en la cara superior de la caja: la parte horizontal superior contiene la IMU, y la parte vertical inferior permite el paso de cables entre secciones.

### Diagrama de ejes de la etiqueta

La etiqueta muestra un diagrama de ejes:
- X → izquierda
- Y ↑ arriba
- Z ⊙ saliendo del plano (origen con circulo y punto central)

### Marcas de orientacion

- **Flecha de cinta adhesiva blanca:** pegada sobre la IMU apuntando hacia arriba/hacia el eje. Es una marca visual de la direccion "forward" del sistema para referencia rapida durante el montaje y la depuracion.
- **Cinta adhesiva naranja/ambar alrededor del perimetro:** posiblemente Kapton tape (poliimida, resistente al calor, aislante electrico) o cinta de embalaje usada como aislante improvisado.
- **Pegatina roja circular sobre la IMU:** posiblemente sello de calidad del fabricante o indicador de inspeccion.

### Importancia de la orientacion

La orientacion de montaje de la IMU define como el software interpreta los datos de aceleracion y giroscopo. Si Tyler camina hacia adelante, el movimiento se registra predominantemente en un eje especifico segun esta orientacion de montaje. En fotos mas recientes se observa una orientacion de ejes diferente (Z↑, X←, Y→), lo que podria indicar un remontaje de la IMU o simplemente un efecto del angulo fotografico.

---

## Por que la IMU es tan critica

En entornos sin GPS (subterraneos), la IMU es la UNICA fuente de informacion sobre el movimiento del sistema entre escaneos del LiDAR. Su calidad determina directamente la precision del SLAM. Tyler menciono en la transcripcion del video 1: "La calidad de la IMU importa mucho mas de lo que pensaba inicialmente; hare un video entero dedicado a por que."

En las publicaciones de Instagram, Tyler enfatizo: "data timing? If your sensor streams aren't synced to the millisecond, you're not mapping anything, you're just making art."

### Problema fundamental de los IMUs baratos

**@OttoSphalta** identifico el problema tecnico central con dos componentes:

1. **Frecuencia de muestreo insuficiente:** Si la IMU mide a 100 Hz pero el movimiento tiene componentes de alta frecuencia (vibracion del motor, pasos del usuario), se pierden datos criticos.
2. **Error de integracion:** La posicion se obtiene integrando la aceleracion dos veces (doble integracion). Cualquier error en la aceleracion se acumula cuadraticamente con el tiempo. Un bias de tan solo 0.01 m/s² se convierte en 1.8 metros de error en solo 19 segundos.

El N100 de Wheeltech, con su frecuencia de 400 Hz, mitiga parcialmente el primer problema pero no elimina el segundo.

### Sorpresa sobre la fiabilidad

**@janrafflewski5468** expreso sorpresa de que el posicionamiento basado en IMU sea lo suficientemente confiable para esta aplicacion. Quiere saber como se redujo el drift a un nivel aceptable.

### Drift a largo plazo sin GNSS

**@melonenlord2723** pregunto sobre el drift del IMU a largo plazo. Normalmente el drift se corrige con GNSS (GPS), que no esta disponible bajo tierra. Sin GNSS, se necesitan otras tecnicas para combatir el drift:

- Loop closure en el algoritmo SLAM
- Puntos de referencia conocidos (tags, reflectores)
- IMUs de grado tactico (mucho mas caros)

**@whatilearnttoday5295** senalo que se necesitan puntos de referencia para el loop closing. Sin tags ni reflectores, el loop closing depende del matching de la geometria del entorno, lo cual puede fallar en entornos repetitivos (pasillos largos, cuevas uniformes).

### Fracaso documentado con IMU + filtro de Kalman

**@MJ-mw3ni** documento un fracaso completo: intento construir un sistema similar con SICK MRS1000 o VLP-16 + IMU de 9 ejes + RTK GPS (hardware superior al de Tyler en algunos aspectos). Ningun algoritmo SLAM funciono y los datos "even with kalmannfilter was so corrupted." Posibles causas: mala sincronizacion temporal entre sensores, IMU de baja calidad a pesar de tener 9 ejes, configuracion incorrecta del SLAM. La leccion: el hardware solo no es suficiente; la integracion, sincronizacion y calibracion son igualmente criticos.

---

## Pregunta sobre la ubicacion de la IMU

**@erfindungsbureau** pregunto por que no se coloca la IMU en la parte rotativa del sistema (en vez de la parte fija).

**Implicacion tecnica:** Si la IMU esta en la parte fija, mide el movimiento del operador pero no la orientacion exacta del LiDAR (que esta rotando). Si estuviera en la parte rotativa, mediria todo pero tendria que compensar la rotacion conocida del motor. Cada enfoque tiene tradeoffs de calibracion distintos.

**@____________________________.x** sugirio, como respuesta, montar una IMU en la pieza giratoria y usar un osciloscopio para balancear el conjunto ensamblado.

---

## Alternativas superiores de IMU mencionadas por la comunidad

### VectorNav VN100 (~$800-1,500 USD)

- Grado tactico
- Estabilidad de bias del giroscopo: 10 grados/hora (vs ~100 grados/hora en IMUs baratos como el N100)
- Fusion de sensores integrada con filtro de Kalman
- Compensacion de temperatura
- Representaria una mejora de aproximadamente 10 veces en calidad de datos respecto al Wheeltech N100
- **@Martarts OFRECIO uno a Tyler** (Tyler respondio afirmativamente)

### SBG IMU ($2,000-10,000+ USD)

- Grado industrial/tactico
- **@kjundvr** lo usa para un proyecto de demostracion en secundaria (high school)
- Muy por encima de lo necesario para un proyecto DIY, pero indica que este usuario busca maxima precision

---

## Posicionamiento complementario a la IMU

### RTK GPS

- **@JasonThomasHorn** recomendo RTK GPS para mejorar la precision en areas abiertas. La estacion base actua como servidor GPS y establece un punto home. Sugirio opciones de **SparkFun** (~$200-500 USD).
- Proporciona precision centimetrica (vs ~3 metros del GPS normal), pero SOLO funciona con cielo visible.
- Para la transicion exterior-interior de una cueva: establecer el punto de entrada con precision, y luego el SLAM toma el relevo.
- **@fedro6352** tambien mostro interes en la alineacion de trayectoria con modulo RTK para mejor rendimiento en areas abiertas.

### Puntos de referencia para loop closing

- **@whatilearnttoday5295** senalo que sin puntos de referencia (tags, reflectores), el loop closing depende del matching de la geometria del entorno, lo cual puede fallar en entornos repetitivos (pasillos largos, cuevas uniformes).
- **@PietjeNL** sugirio usar bolas reflectivas pequenas en un palo como puntos de referencia visibles desde multiples angulos para facilitar el stitching de escaneos.
- **@tullgutten** sugirio crear una herramienta de referencia de alineacion usando una piramide o cono que se pueda colocar en un punto visible desde multiples angulos.

---

## Datos de la IMU en las grabaciones

Los datos de la IMU siempre se incluyen en los bags de ROS 2, incluso para escaneos estaticos. Tyler confirmo en el video 3: "la IMU a 200 Hz (no necesaria para escaneo estatico pero se deja encendida)." La grabacion de cada sesion contiene: datos de IMU, angulo de plataforma, velocidad de plataforma, y nube de puntos.

Los topics esperados de ROS 2 incluyen los datos de la IMU como uno de los flujos de datos grabados. Las frecuencias observadas en la app varian entre 100 y 200 Hz segun la configuracion activa.

---

## Sincronizacion temporal IMU-LiDAR

Este es el factor tecnico mas critico para la calidad de los datos, confirmado por multiples fuentes independientes:

### @PeaceIndustrialComplex (comentario con 11 likes, Tyler respondio con 5 respuestas)

Senalo que el VLP-16 no tiene IMU integrado. Si se logra buena sincronizacion temporal con la IMU USB y el Velodyne via PPS (Pulse Per Second), se reducira significativamente el ruido en la nube de puntos final.

**Desglose tecnico del PPS:**
- PPS = Pulse Per Second, una senal electrica que marca exactamente el inicio de cada segundo
- El VLP-16 tiene entrada PPS y puede sincronizar sus timestamps con ella
- Si la IMU tambien se sincroniza con la misma senal PPS, ambos sensores comparten la misma referencia temporal
- Sin PPS, cada sensor usa su propio reloj interno, que puede derivar microsegundos o milisegundos -- suficiente para introducir error cuando el sistema se mueve rapido
- Con PPS, cada punto LiDAR se puede asociar con la orientacion de la IMU exacta en ese instante. Sin PPS, hay que interpolar y esto introduce ruido

### @andbondstyle (sistema similar inspirado en paper "Alpha LiDAR")

Confirmo que la sincronizacion temporal es critica y expando: no solo LiDAR e IMU necesitan time sync, tambien el encoder del motor de rotacion. Si no se sabe exactamente en que angulo estaba el motor cuando se capturo cada punto, la transformacion de coordenadas sera imprecisa.

### @weselsmith

Pregunto como asegurar sincronizacion perfecta entre IMU y LiDAR. Sin respuesta del creador -- es un problema conocido y critico en SLAM.

### Solucion implementada por Tyler

Tyler usa la senal PPS del GPS (a traves de dos lineas GPS que pasan por el slip ring) con un reloj de tiempo real (RTC) para conseguir sincronizacion precisa entre todos los subsistemas: la IMU, los puntos del Velodyne, y los datos del encoder. Como confirmo Tyler a @lordsqueak: "Solo usa la senal PPS del GPS para sincronizacion de tiempo." @lordsqueak pregunto como se obtendria esa senal dentro de una cueva (sin respuesta -- es una pregunta abierta critica).

### Soluciones para time sync sin GPS (underground)

1. Senal PPS generada localmente (por el Pi o un microcontrolador dedicado)
2. Hardware timestamping en el bus de datos
3. Sincronizacion por NTP local entre componentes
4. Trigger electrico compartido

---

## Conexion de la IMU al sistema

La IMU N100 se comunica con el Raspberry Pi 5 via USB serial a traves de un cable USB-C. La cadena de datos completa del sistema es:

VLP-16 (captura puntos) → Ethernet → Slip ring → Raspberry Pi 5 (graba bag ROS 2) ← USB serial ← IMU N100

La IMU forma parte del conjunto de sensores cuya informacion se graba simultaneamente en el bag de ROS 2 junto con los datos del LiDAR y del encoder de la plataforma.

---

## Relevancia de la IMU para el algoritmo SLAM

Tyler menciono en el video 1 que el algoritmo SLAM que usa es probablemente **FAST-LIO** (identificado por @yxlee2676 al reconocer la interfaz en el minuto 2:30 del video 2) o posiblemente un fork de **LIO-SAM** (sugerido por @RYU47376). Ambos algoritmos son de tipo LiDAR-inertial, lo que significa que dependen directamente de los datos de la IMU para funcionar.

- **FAST-LIO** es un algoritmo filter-based disenado especificamente para fusion LiDAR-inertial, desarrollado por HKU
- **LIO-SAM** es otro algoritmo LiDAR-inertial ampliamente usado
- Si se anade camara, existe **LVI-SAM** como opcion que fusiona LiDAR + visual + inercial

### Problemas de SLAM documentados por Tyler

- Debugging de SLAM incluye problemas de drift y fallos en loop closure
- Tyler confirmo a @peterskourup7635: "Sin problemas con grabaciones estaticas y deskewing. Los problemas son con SLAM, donde hay drift masivo"
- Se alejo deliberadamente del SLAM para perfeccionar primero el de-skewing: "I intentionally took a step back from the SLAM integration to really nail this part down. If the points aren't landing where they should, nothing downstream really matters"
- Planea volver a SLAM una vez que valide el proceso de de-skewing

### Drift y loop closure

- **@yxlee2676** pregunto sobre completitud del mapa y si la estimacion de pose sufre drift o falla
- **@franklynd** enfatizo que la deriva siempre ocurrira sin loop closure; cerrar bucles es realmente importante
- **@Tebbit123** pregunto especificamente si es la IMU la que no mantiene precision o la que drifta. Su objetivo inmediato es handheld, y el objetivo final es montarlo en un drone para mapeo aereo

---

## Campo de investigacion SLAM (contexto academico)

- **@RYU47376** (PhD primer semestre) comento que el campo de SLAM ya esta bastante maduro, lo cual dificultaria hacer contribuciones academicas nuevas
- **@yxlee2676** coincidio
- **@bomboclaat9215** menciono que en 2009 SLAM era nuevo, no habia software open source ni hardware accesible, y existia "Rat SLAM" que hacia SLAM con una sola camara
- Tyler confirmo que ahora hay muchos recursos open source excelentes

---

## Planes futuros relacionados con la IMU

Tyler planea:
- Anadir camaras RGB para **Gaussian splatting** y utilizar **VIO (Visual Inertial Odometry)** para complementar SLAM
- Documentacion completa incluyendo un video dedicado a la seleccion de IMU y por que la calidad importa tanto
- Todo sera open source eventualmente
