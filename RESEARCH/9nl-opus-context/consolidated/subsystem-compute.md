# Subsistema de computo -- Raspberry Pi 5 y entorno de desarrollo

Documento de referencia consolidado sobre el subsistema de computo del proyecto Subterranean Systems. Toda la informacion ha sido extraida exhaustivamente de las fuentes primarias: transcripciones de video, comentarios de YouTube, publicaciones de Instagram, READMEs de GitHub, e inferencias de imagenes.

---

## Raspberry Pi 5 -- El cerebro del sistema

| Parametro | Valor |
|---|---|
| Modelo | Raspberry Pi 5 |
| CPU | Quad-core Cortex-A76 a 2.4 GHz |
| RAM | 4 u 8 GB |
| SO / Framework | ROS 2 |
| Rol | Sistema nervioso central: se comunica con VLP-16 (Ethernet), IMU (USB), Teensy (UART serial), graba bags ROS 2, ejecuta pipeline de datos |
| Almacenamiento | Tarjeta SD grande o SSD (~206 GB de espacio libre observado en la app SLAM Data Recorder) |
| Disipador | Naranja/cobre, visible en fotos del interior de la caja |
| Tiempo de arranque | ~1-2 minutos para inicializacion completa. Indicador: "everything running and green" |
| Acceso remoto | SSH desde laptop |
| Precio | ~$80 USD |

---

## Comunicaciones del Raspberry Pi 5

| Interfaz | Conexion | Tipo de datos |
|---|---|---|
| Ethernet | VLP-16 (a traves del slip ring) | Datos de nube de puntos (Gigabit 1000Base-T) |
| USB serial | IMU Wheeltech N100 (cable USB-C) | Datos de aceleracion, giroscopo y magnetometro |
| UART serial (RX) | Teensy 4.0 (pines TX1/RX1 via carrier board) | Datos del encoder angular AS5600 |
| GPIO | Cables conectados visibles en fotos | Funcion especifica no documentada |

### Cadena de datos completa del sistema

VLP-16 (captura puntos) → Ethernet → Slip ring → Raspberry Pi 5 (graba bag ROS 2) ← Serial UART ← Teensy 4.0 ← I2C ← AS5600 (lee angulo del eje) + SimpleFOC (controla motor 4015)

---

## Directorio de datos y grabaciones ROS 2

Los bags crudos de ROS 2 se graban en un directorio dedicado en el almacenamiento del Pi 5. Tyler se conecta al Pi 5 por SSH para acceder al directorio donde se guardan los bags. Cada grabacion contiene:

- Datos de IMU
- Angulo de plataforma
- Velocidad de plataforma
- Nube de puntos

### Topics esperados de ROS 2

| Topic ROS 2 | Tipo de mensaje | Descripcion |
|---|---|---|
| `/velodyne_points` | `sensor_msgs/PointCloud2` | Frames de escaneo del VLP-16 |
| `/rotating_platform/angle` | `std_msgs/Float64` | Angulo del encoder de la plataforma en grados |

### Formato y tamano de los bags

- Formato: `.mcap` o `.mcap.zstd` (comprimido con Zstandard)
- Los bags comprimidos `.mcap.zstd` se descomprimen automaticamente in-place en el primer uso
- Escaneo estatico de 2 minutos: ~400 MB comprimido, ~1 GB descomprimido
- Contiene aproximadamente 21 millones de puntos, submuestreados a ~13.75 millones en el archivo final

---

## Puede el Pi 5 manejar SLAM?

**@Maxjoker98** pregunto directamente si el Raspberry Pi puede manejar ROS 2 y si es suficiente para el procesamiento. El Pi 5 tiene CPU quad-core Cortex-A76 a 2.4 GHz y 4/8 GB de RAM. ROS 2 funciona correctamente, pero el procesamiento SLAM en tiempo real es computacionalmente intensivo.

Lo mas probable es que el Pi se use exclusivamente para el logging (grabacion de datos) y que el SLAM se ejecute en post-procesamiento en un PC mas potente (el laptop de Tyler con Core i9 y GeForce RTX).

Tyler confirmo que se ha alejado del SLAM en tiempo real para centrarse en escaneos estaticos con de-skewing de nubes de puntos. Planea volver a SLAM una vez que valide el proceso de de-skewing.

---

## Alternativas de procesamiento mencionadas por la comunidad

### NVIDIA Jetson Orin Nano Super

Sugerido por **@Prod.M00N** y **@NukeBrosCODM**.

| Parametro | Valor |
|---|---|
| GPU | 1024 CUDA cores |
| RAM | 8 GB |
| Soporte ROS 2 | Nativo |
| Precio | ~$250 USD (vs ~$80 del Pi 5) |

**Beneficios:**
- Permitiria SLAM en tiempo real (no post-procesamiento)
- Procesamiento de nube de puntos acelerado por GPU
- Posibilidad de deep learning SLAM

**Tradeoffs:**
- Mas consumo de energia (critico para sistema portatil con bateria)
- Mas complejo de configurar

**@NukeBrosCODM** tambien sugirio una pantalla mas grande para mostrar el proceso de mapeo 3D en vivo.

**Respuesta de Tyler:** Ambos upgrades (Jetson + pantalla mas grande) serian geniales. Planea hacerlos cuando tenga mas presupuesto.

### Hailo-8L NPU HAT

Sugerido por **@wchutcheson**.

| Parametro | Valor |
|---|---|
| Capacidad | 13 TOPS de procesamiento neural |
| Compatibilidad | HAT para Pi 5 |
| Precio | ~$70 USD |

**Posibles usos:**
- Extraccion de caracteristicas de nubes de puntos
- Clasificacion de superficies
- "Vision AI stuff could probably be adapted to laser"

**Limitacion:** NO es directamente aplicable a SLAM clasico. El SLAM basado en LiDAR-inertial (FAST-LIO, LIO-SAM) usa geometria y algebra lineal, no redes neuronales. Sin embargo, podria ser util para tareas complementarias como reconocimiento de objetos o clasificacion de terreno.

---

## App SLAM Data Recorder -- Interfaz de usuario

Tyler construyo una app de control remoto que se ejecuta en un smartphone (con funda azul) y permite operar el escaner sin necesidad de SSH al Raspberry Pi. La app se comunica con el Pi 5 y permite:

- Iniciar y detener grabaciones
- Monitorear el estado de los sensores
- Verificar las frecuencias de datos
- Ver el almacenamiento disponible

### Elementos de la interfaz

| Elemento | Descripcion |
|---|---|
| Titulo | "SLAM Data Recorder" |
| Estado de conexion | "CONNECTED" (verde) |
| Estado de sensores | "Sensors: ON" |
| Frecuencia LiDAR | Variable: 15.0 Hz, 17.0 Hz, 23.0 Hz, 31.0 Hz (segun version/configuracion) |
| Frecuencia IMU | Variable: 100.0 Hz, 101.0 Hz, 199.1 Hz |
| Frecuencia encoder | Variable: 320 Hz, ~451.1 Hz |
| Almacenamiento disponible | ~206 GB |
| Boton principal | "START RECORD" (verde) / "STOP RECORD" (rojo) |
| Tiempo de grabacion | Contador en segundos (ejemplo: "REC: 038") |
| Lista de Topics | Probablemente ROS topics monitoreados |

### Datos de frecuencia observados en diferentes momentos

| Momento | LiDAR | IMU | Encoder | Almacenamiento |
|---|---|---|---|---|
| Foto campo v1 (IMG_9449) | 23.0 / 30.2 Hz | 199.1 Hz | ~451.1 Hz | -- |
| Foto integracion (IMG_9471) | 17.0 > 15.0 Hz | 100.0 Hz | 320 Hz | 206.4 GB |
| Foto campo v2 (IMG_9472) | Similar | Similar | Similar | 206.4 GB |
| Foto handheld (IMG_9458) | 31.0 Hz | 101.0 Hz | ~441.1 Hz | -- |
| Foto campo avanzada (IMG_9474) | -- | -- | -- | 206.8 GB |

Las variaciones entre sesiones sugieren configuraciones diferentes o evolucion del firmware.

---

## Pipeline de software

### Vision general

Pipeline offline de tres etapas para escaneos estaticos (sin SLAM). Repositorio: `pointcloud-reconstruction` en GitHub (github.com/tthom289/pointcloud-reconstruction). Licencia MIT. Python 100%. 36 stars, 4 forks. Contributors: tthom289 (Tyler) y Claude.

El pipeline esta escrito integramente en Python y se ejecuta en Windows (confirmado por bordes de ventana Windows visibles en capturas). Python 3.11 recomendado (Open3D no soporta Python 3.13+).

Tyler se alejo deliberadamente del SLAM para perfeccionar el de-skewing: "I intentionally took a step back from the SLAM integration to really nail this part down. If the points aren't landing where they should, nothing downstream really matters. Once de-skewing is solid, the SLAM will go back in."

### Dependencias Python

| Paquete | Usado por | Proposito |
|---|---|---|
| numpy | todos | Matematicas de arrays |
| mcap | offline_deskew, icp_merge | Lectura de bags MCAP |
| mcap-ros2-support | offline_deskew, icp_merge | Decodificacion de mensajes ROS 2 |
| zstandard | offline_deskew, icp_merge | Descompresion de bags .mcap.zstd |
| open3d | icp_merge, flythrough | Registro ICP, lectura/escritura PCD |
| glfw | flythrough | Ventana OpenGL e input |
| PyOpenGL | flythrough | Bindings OpenGL |
| PyOpenGL_accelerate | flythrough | Aceleradores C opcionales para PyOpenGL |

### Etapa 1 -- De-skewing (offline_deskew.py)

Los datos crudos del bag ROS 2 contienen los frames del VLP-16 y los angulos del encoder de la plataforma rotatoria. Cada frame de puntos se corrige por el movimiento de rotacion de la plataforma durante la captura de ese frame (motion de-skewing). Sin este paso, los puntos aparecerian borrosos o duplicados porque el LiDAR se mueve mientras captura cada revolucion.

**Parametros de calibracion (editables al inicio del archivo):**

| Parametro | Valor por defecto | Descripcion |
|---|---|---|
| `ROTATION_AXIS` | `'x'` | Eje alrededor del cual rota la plataforma |
| `ANGLE_OFFSET_DEG` | `184.0` | Offset de calibracion del angulo cero en grados |
| `ROTATION_CENTER` | `[0,0,0]` | Offset del centro optico del LiDAR respecto al eje de rotacion |
| `INVERT_ROTATION` | `False` | Invertir direccion de rotacion |
| `ENCODER_TIME_OFFSET_MS` | `0.0` | Offset temporal entre encoder y relojes del LiDAR |
| `MOUNT_RPY_DEG` | `[0,0,0]` | Roll/pitch/yaw del montaje del LiDAR en grados |
| `MOUNT_AXES` | `['+x','+y','+z']` | Remapeo de ejes del montaje del LiDAR |

**Salida (.npz):**

| Clave | Descripcion |
|---|---|
| `points` | Array de arrays XYZ de puntos de-skewed (uno por frame) |
| `timestamps` | Timestamps en nanosegundos por frame |
| `angle_times` | Timestamps de muestras del encoder |
| `angle_values` | Angulos del encoder en radianes |

### Etapa 2 -- ICP Merge (icp_merge.py)

Los frames de-skewed se dividen en slices angulares con solapamiento. Se usa el algoritmo ICP (Iterative Closest Point) para alinear slices adyacentes entre si. Tyler usa una estrategia de dos pasadas (chain → refine → re-chain) para minimizar el drift acumulativo. El resultado es un archivo `.pcd` (Point Cloud Data) con todos los puntos registrados en un marco de referencia comun.

El alineamiento ICP se aplica sobre el paso de de-skewing para compensar imperfecciones mecanicas: runout del eje, oscilacion del rodamiento, u otras tolerancias del hardware que causan que mitades opuestas de 180 grados del escaneo no se alineen perfectamente incluso despues de la correccion de movimiento.

**Opciones de linea de comandos:**

| Argumento / Flag | Defecto | Descripcion |
|---|---|---|
| `bag_path` | -- | Ruta al directorio del bag o archivo .mcap |
| `--slices N` | `12` | Numero de slices angulares |
| `--overlap F` | `2.0` | Ancho de ventana del slice como multiplo del ancho del paso |
| `--fine` | off | Usar tamano de voxel mas fino (0.05 m) para ICP |
| `--min-fitness F` | `0.3` | Puntuacion minima de fitness ICP para aceptar un slice |
| `--no-chain` | off | Alinear cada slice directamente a la referencia (sin encadenamiento) |
| `--map-voxel F` | `0.01` | Tamano de voxel para downsample final en metros; 0 para desactivar |
| `--trim-end N` | `0` | Recortar los ultimos N segundos del bag |
| `-o FILE` | `icp_merged.pcd` | Ruta del archivo PCD de salida |

**Etapas del pipeline:**

1. Leer bag -- decodificar mensajes del encoder y de la nube de puntos
2. De-skew de todos los frames de escaneo
3. Construir slices con solapamiento y alinear con ICP (chain → refine → re-chain)
4. Fusionar puntos core con las transformaciones alineadas y escribir el PCD

### Etapa 3 -- Visor flythrough (flythrough.py)

Visor personalizado de nubes de puntos en primera persona construido con OpenGL (GLFW para ventana, PyOpenGL para rendering). Escrito integramente en Python, parte del repositorio open source. Se ejecuta en Windows.

**Razon de existencia:** CloudCompare y Arvis eran demasiado lentos para el flujo de trabajo iterativo. Tyler necesitaba volar a traves de la nube de puntos rapidamente mientras ajustaba el algoritmo de de-skewing, verificando visualmente si los puntos "aterrrizaban donde debian." El visor elimina la dependencia de CloudCompare u otras herramientas para la inspeccion rapida.

**Aspecto visual:** Fondo negro (void del espacio 3D sin puntos), overlay HUD de controles en la esquina superior izquierda.

**Controles de navegacion:**

| Tecla / Raton | Accion |
|---|---|
| W / S | Mover adelante / atras |
| A / D | Strafe izquierda / derecha |
| Space / LCtrl | Mover arriba / abajo |
| Click izquierdo + arrastrar | Mirar alrededor |
| Click medio/derecho + arrastrar | Pan |
| Scroll | Ajustar velocidad de movimiento |
| + / - | Tamano de punto |
| C | Ciclar modo de color |
| E | Toggle Eye-Dome Lighting (EDL) |
| L / Shift+L | Intensidad EDL arriba / abajo |
| R | Resetear camara al centro |
| P | Imprimir posicion de camara |
| 1-5 | Presets de velocidad |
| X | Filtro de outliers estadistico |
| F12 | Guardar / sobreescribir PCD original |
| F5 / F6 | Rotar escena +-90 grados en X |
| F7 / F8 | Rotar escena +-90 grados en Y |
| F9 / F10 | Rotar escena +-90 grados en Z |
| F1 | Resetear rotacion de escena |
| Esc | Salir |

**Modo comparacion (2+ archivos):**

| Tecla | Accion |
|---|---|
| F2 | Toggle visibilidad nube 1 |
| F3 | Toggle visibilidad nube 2 |
| F4 | Toggle visibilidad nube 3 |

**Modo grab -- alineacion interactiva (2+ archivos, Tab para entrar/salir):**

| Tecla / Raton | Accion |
|---|---|
| Tab | Toggle modo grab on/off (Esc para cancelar y revertir) |
| Click izquierdo + arrastrar | Trasladar nube 2 en el plano de vista |
| Shift + click izquierdo + arrastrar | Rotar nube 2 alrededor de su centroide |
| Scroll | Empujar/tirar nube 2 a lo largo del eje forward de la camara |
| I | Ejecutar ICP para refinar alineacion (luego sale del modo grab) |

**Modo pick -- alineacion por correspondencia (2+ archivos):**

| Tecla / Raton | Accion |
|---|---|
| T | Toggle modo pick on/off |
| Click izquierdo | Seleccionar punto mas cercano (alterna entre nube 1 y nube 2) |
| U | Deshacer ultima seleccion |
| G | Calcular alineacion a partir de los pares actuales (necesita 3+) |
| M | Fusionar todas las nubes y guardar (pide nombre de archivo) |

**Modos de color disponibles:** Color solido, por altura (height colormap), por intensidad, por archivo de origen (cuando hay multiples nubes cargadas).

---

## CloudCompare (herramienta externa)

| Parametro | Valor |
|---|---|
| Version observada | CloudCompare v2.11.1 (Anoia) [64-bit] |
| Uso | Visualizacion avanzada y fusion de multiples escaneos |

**Herramientas usadas dentro de CloudCompare:**
- F2: ver escaneos independientemente
- G: alinear 3+ pares de puntos (point-pair registration)
- I: otra herramienta de alineacion (ICP iterativo; Tyler las confundio inicialmente)
- SOR: Statistical Outlier Removal (visible en barra de herramientas)

**Fusion de escaneos en CloudCompare:** Se abren dos archivos PCD simultaneamente. Se identifican puntos compartidos entre dos escaneos (esquinas de casa, picos de tejado, esquinas de valla, postes de telefono como referencias a larga distancia). Se marcan 3+ pares de puntos correspondientes y se ejecuta el alineamiento. El programa se congela unos segundos mientras procesa. El resultado no es perfecto pero es bastante bueno. Se pueden fusionar los colores y en menos de cinco minutos de trabajo se obtiene un mapa combinado.

Ademas de CloudCompare y el visor custom, **RViz** (el visor estandar de ROS 2) es otra herramienta de visualizacion posible -- se observa una interfaz con fondo azul oscuro y cuadricula azul en algunas capturas que podria ser RViz o el visor personalizado.

---

## Repositorios GitHub del proyecto

### pointcloud-reconstruction

| Parametro | Valor |
|---|---|
| Link | github.com/tthom289/pointcloud-reconstruction |
| Licencia | MIT |
| Lenguaje | Python 100% |
| Stars | 36 |
| Forks | 4 |
| Contributors | tthom289 (Tyler), Claude |
| Contenido | Pipeline de de-skewing + ICP merge + visor flythrough |
| Plataforma probada | Windows 11, Python 3.11 |

### vlp16-spin-controller

| Parametro | Valor |
|---|---|
| Link | github.com/tthom289/vlp16-spin-controller |
| Licencia | MIT |
| Lenguaje | C++ 100% |
| Stars | 18 |
| Forks | 0 |
| Contributors | tthom289 (Tyler), Claude |
| Descripcion | Controlador basado en SimpleFOC para Teensy 4.0 con motor BLDC GM2804 + encoder AS5600 con reduccion 5:1, para rotacion continua a 40 RPM del VLP-16 |
| Ultimo cambio significativo | "Switch to open-loop velocity control with Pi5 UART output" |
| Archivos de configuracion | platformio.ini, VS Code workspace |
| Sin README | Solo contiene la licencia MIT como documentacion |

### TeensyFOC-Carrier

| Parametro | Valor |
|---|---|
| Link | github.com/tthom289/TeensyFOC-Carrier |
| Licencia | MIT |
| Lenguaje | KiCad (diseno de PCB) |
| Stars | 15 |
| Forks | 0 |
| Contributors | tthom289 (Tyler) |
| Descripcion | Carrier board para Teensy 4.0 + SimpleFOC para control de motor brushless en la plataforma SLAM LiDAR rotatoria. Integra encoder magnetico AS5600, passthrough del slip ring, y circuiteria del motor driver GT2 belt drive |

**Archivos del repositorio:**

| Archivo | Descripcion |
|---|---|
| `TeensyFOC.kicad_sch` | Esquematico KiCad |
| `TeensyFOC.kicad_pcb` | Layout PCB |
| `TeensyFOC.kicad_pro` | Proyecto KiCad |
| `TeensyFOC.kicad_prl` | Preferencias locales KiCad |
| `TeensyFOC.step` | Modelo 3D |
| `TeensyFOC.zip` | Gerbers para fabricacion |
| `TeensyFOC.csv` | Netlist CSV |
| `TeensyFOC_bom.csv` | Bill of Materials |
| `TeensyFOC_designators.csv` | Designadores de componentes |
| `TeensyFOC_positions.csv` | Posiciones pick-and-place |
| `TeensyMotorCTRL.zip` | Zip adicional (motor control) |
| `fabrication-toolkit-options.json` | Opciones del toolkit de fabricacion |

**Bill of Materials (conectores):**

| Componente | Fuente |
|---|---|
| Single Row 2.54mm Headers | Amazon |
| JST-XHP Connector Kit | Amazon |

**Workflow de la carrier board:** Diseno en KiCad (open source) → Fabricacion en JLCPCB → Ensamblaje manual (soldadura en casa).

---

## Hardware de desarrollo

### Laptop principal

| Parametro | Valor |
|---|---|
| Marca | Lenovo |
| Procesador | Intel Core i9 |
| GPU | NVIDIA GeForce RTX (pegatinas visibles: "GEFORCE RTX", "Intel CORE i9", "Lenovo") |
| Capacidad | Suficiente para procesamiento de nubes de puntos, renderizado OpenGL, y eventualmente SLAM acelerado por GPU |

La GPU RTX dedicada explica la capacidad de renderizar nubes de puntos con el visor OpenGL personalizado y eventualmente ejecutar procesamiento GPU-acelerado.

### Segundo laptop

| Parametro | Valor |
|---|---|
| Tipo | Laptop gaming |
| Teclado | Retroiluminado verde/amarillo, con teclado numerico |
| Marca probable | ASUS o MSI (diferente del Lenovo) |
| Uso observado | Con CloudCompare |

### Monitores

| Monitor | Detalle |
|---|---|
| Monitor principal | Dell (texto "DELL" legible en bisel inferior) |
| Segundo monitor | Muestra terminal con texto verde sobre fondo oscuro (probablemente output de ROS 2 o logs) |

### Cutting mat

Alfombrilla verde de corte, con cuadricula en milimetros y pulgadas. Es la alfombrilla estandar de una estacion de electronica/maker.

---

## Otros elementos del taller

- **Drone FPV de carreras** colgado en la pared (helices y guardas claramente visibles): confirma experiencia previa en electronica/robotica. Tyler no es un novato que empezo con el LiDAR.
- **Bobina de soldadura dorada** sobre la mesa
- **Documento/poster/datasheet negro** con texto blanco y diagrama de cables visible sobre la mesa (posiblemente pinout del motor o encoder, con etiquetas numeradas tipo "21.", "22.")
- **Anillo de luz o microfono con soporte** (para creacion de contenido)
- **Abrazadera DeWalt amarilla** para fijacion del sistema en la mesa durante pruebas de banco
- **Tripode pequeno con rotula roja marca Ulanzi** (accesorio de camara/video reutilizado como soporte para el VLP-16 durante pruebas sin rotacion)

---

## Datos de rendimiento del bag ROS 2

| Parametro | Valor |
|---|---|
| Escaneo estatico de 2 minutos | ~21 millones de puntos |
| Submuestreados | ~13.75 millones de puntos |
| Tamano comprimido | ~400 MB |
| Tamano descomprimido | ~1 GB |
| Frecuencia del LiDAR (banco) | 15-16 Hz |
| Frecuencia del LiDAR (handheld avanzado) | Hasta 31 Hz |
| Frecuencia de la IMU | 100-200 Hz |
| Frecuencia del encoder | 200-451 Hz |
| Almacenamiento disponible | ~206 GB |

Otro escaneo documentado: 1.126.298 puntos en el archivo `bag2_2024_01_09-14_16_25_merged_map.pcd` (escaneo mas corto o con mas submuestreo). La fecha en el nombre del archivo (2024-01-09) presenta discrepancia con la timeline del proyecto (2026) -- posible error del reloj del sistema o datos de pruebas antiguas.

---

## ROS 2 como barrera de entrada

**@fernandodiaz4092** tiene hardware similar (VLP-32) pero no ha logrado superar los obstaculos de ROS. Literalmente dijo: "Way over my head." Tiene hardware MEJOR (VLP-32 = 32 canales) pero no puede superar la barrera de software.

**@lanealucy** pregunto: "What do I need to Google to just make a 3D point cloud out of a 2D lidar and imu (with ros2)?"

**Implicacion:** ROS 2 es la barrera de entrada principal para la replicacion del proyecto. Un tutorial paso a paso de configuracion de ROS 2 con VLP-16 + IMU tendria alto impacto en la comunidad.

**@cckeysify** pregunto por el repositorio Git y expreso esperanza de que "no sea Python script", argumentando que las empresas comerciales no usan Python para esto. **Matiz:** Las empresas comerciales usan C++ para procesamiento en tiempo real, pero muchos paquetes SLAM modernos (FAST-LIO, LIO-SAM) estan en C++ con wrappers ROS 2 en C++/Python. Python es perfectamente valido para post-procesamiento y scripting.

### Demanda masiva de acceso al codigo

Al menos 6 comentarios piden especificamente repositorio Git: @notexpected, @domenicoacierno2044 (Tyler respondio), @christopaaron, @jsk_0211, @74n3r, @madanm7454. Tyler planea documentar todo y compartirlo completamente open source.

---

## Proceso de encendido del sistema

Documentado en el video 3:

1. Conectar la bateria de 12V -- esto arranca la Raspberry Pi, el HMI (pantalla de control), la IMU, y el sensor LiDAR
2. Conectar la alimentacion separada del Teensy/motor controller (power bank USB)
3. Asegurar que el interruptor del motor esta encendido
4. El sistema inicializa y comienza a girar a ~20 RPM
5. Esperar a que todo arranque en el lado del Pi (indicador: "todo en verde", ~1-2 minutos)
6. Colocar el sistema en vertical
7. Iniciar la grabacion (via la app SLAM Data Recorder o SSH)

Cuando se trabaja en interior, el sistema se alimenta de una fuente de alimentacion DC en vez de la bateria, lo que resulta mas comodo para testing prolongado.

---

## Software y algoritmos SLAM

### Algoritmo principal: FAST-LIO

Identificado por **@yxlee2676** al reconocer la interfaz en el minuto 2:30 del video 2. FAST-LIO es un algoritmo filter-based de fusion LiDAR-inertial desarrollado por HKU. Es probable que sea FAST-LIO2.

### Alternativas mencionadas

- **LIO-SAM**: sugerido por @RYU47376 como posible alternativa
- **LVI-SAM**: si se anade camara, fusiona LiDAR + visual + inercial
- **GraphSLAM con GTSAM**: usado por @benhenderson6966 en Python, lo encuentra dificil de trabajar
- **Gaussian splatting**: Tyler planea anadir camaras RGB para 3DGS y utilizar VIO (Visual Inertial Odometry) para complementar SLAM

### Estado actual del SLAM en el proyecto

Tyler confirmo: "Sin problemas con grabaciones estaticas y deskewing. Los problemas son con SLAM, donde hay drift masivo." Se ha alejado de SLAM hacia escaneos estaticos + de-skewing de nubes de puntos. Planea volver a SLAM una vez que valide el proceso de de-skewing.

Lecciones clave de Instagram: "Getting SLAM reliable is way harder than it looks. Divergence and drift are constant battles, and one bad loop and your whole map falls apart. Simulation gives you confidence. Real world testing humbles you. Always."

---

## Visualizacion y exportacion de datos

### Herramientas mencionadas por la comunidad

- **@JYAA_Williams** pregunto que software usa para visualizar los datos del Velodyne en el Pi. Ha trabajado con HDL-64e pero solo con herramientas propietarias.
- **@Alpha0ne** interesado en el workflow para exportar a archivo LAS o similar para procesamiento posterior. LAS (LIDAR Aerial Survey) es el formato estandar de la industria. Herramientas open source: PDAL, CloudCompare, laspy/pylas en Python.
- **@wvg.** pregunto si se puede exportar la nube de puntos y convertirla en mesh. Ha usado escaneres Leica con extension de Solidworks (propietario). Si hay solucion open source para el meshing, "you're really onto something."
- **@ethansmith244** quiere saber que software genera las imagenes 3D (principiante completo).
- **@hoseja** sugirio Gaussian splatting para las cuevas.
- **@000Krim** sugirio red neuronal para hacer overlap automatico de dos o mas mapas.

### Grabacion de video del point cloud

- **@siteking4289** sugirio hacer pausas entre movimientos para que la codificacion de video pueda construir una imagen limpia. La compresion de video no maneja bien demasiados detalles en movimiento.
- **@austinclark3495** confirmo que tomas estaticas del point cloud serian beneficiosas porque la compresion de YouTube no maneja bien los datos en movimiento.

---

## Nota sobre plataforma

Probado en Windows 11 con Python 3.11. Deberia funcionar en Linux y macOS. En Linux se necesitan las librerias de sistema OpenGL y GLFW (`libglfw3-dev`, `libgl1-mesa-dev`).

---

## Informacion del creador

| Parametro | Valor |
|---|---|
| Nombre | Tyler |
| GitHub | tthom289 |
| Instagram | @subterraneansystems |
| Canal YouTube | 9nl (tambien referido como Subterranean Systems) |
| Presencia adicional | TikTok |
| Logo | "TTHOM" con icono de engranaje (grabado en la PCB carrier board). "Subterranean Systems" (grabado en la carcasa del chasis) |
| Video 1 | "I Built a Scanner that Sees in Total Darkness" (17 enero 2026, ~167,000 vistas, ~448 comentarios) |
| Video 2 | "The LiDAR That Changed Robotics (And Why I Bought One)" |
| Video 3 | "My LIDAR Was Half Blind (so I fixed it)" |

### Videos planeados (anunciados)

1. Como funciona el LiDAR, que ve, que no, y por que el VLP-16
2. Setup del Raspberry Pi y ROS 2, como se comunican las piezas
3. Plataforma rotativa: seleccion de motor, SimpleFOC, Teensy, mecanica
4. SLAM (FAST-LIO): como convierte datos crudos en mapa con tracking de posicion en tiempo real
5. Video dedicado a la seleccion de IMU y por que la calidad importa tanto
