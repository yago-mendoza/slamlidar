# Subsistema de Sincronizacion Temporal — Referencia Tecnica Exhaustiva

> Fuentes: transcripciones de 3 videos de YouTube (canal 9nl / Subterranean Systems), comentarios de la comunidad (3 videos), posts y captions de Instagram (@subterraneansystems), 3 repositorios GitHub (tthom289), e inferencia de imagenes del proyecto.

---

## 1. Por que la sincronizacion temporal es critica

### 1.1 Tres fuentes independientes confirman que es EL factor tecnico mas critico

La sincronizacion temporal entre sensores es el factor que mas se repite en los comentarios tecnicos de la comunidad como determinante del exito o fracaso del sistema. Tres fuentes independientes con experiencia directa lo confirman:

**Fuente 1 — @PeaceIndustrialComplex** (11 likes, Tyler respondio 5 veces):
> *"if you can get good time sync with the USB IMU and the velodyne via PPS it will significantly help reduce noise in your final cloud"*

Este comentario es notable por el nivel de engagement de Tyler (5 respuestas) y los 11 likes de la comunidad, indicando que la audiencia reconoce la importancia del tema.

**Fuente 2 — @andbondstyle** (trabaja en sistema similar inspirado en paper "Alpha LiDAR", usa Livox MID-360 y motor gimbal con eje hueco):
> *"time sync in such systems are critical to get precise point cloud registration, so you need hardware time sync (most commonly GPS/PPS signal) tied to every motion and data-related component (lidar, IMU, motor encoder etc.)"*

Dato clave: @andbondstyle especificamente menciona el **encoder del motor** como componente que tambien necesita sincronizacion — no solo el LiDAR y la IMU.

**Fuente 3 — @MJ-mw3ni** (intento fallido con hardware superior):
Este usuario intento construir un sistema similar con hardware potencialmente superior (SICK MRS1000 o VLP-16 con IMU de 9 ejes y RTK GPS) y **fallo**. Ningun algoritmo SLAM funcionaba y los datos estaban corruptos incluso con filtro de Kalman. Aunque no lo confirmo explicitamente, una de las causas probables es **mala sincronizacion temporal** entre los sensores.

### 1.2 Cita de Tyler en Instagram

> *"And data timing? If your sensor streams aren't synced to the millisecond, you're not mapping anything, you're just making art."*

Esta cita aparece en un caption de Instagram titulado "What I've learned building my own LiDAR cave mapper (nobody tells you this stuff)" junto con otras lecciones aprendidas:
- *"Getting SLAM reliable is way harder than it looks. Divergence and drift are constant battles, and one bad loop and your whole map falls apart."*
- *"Simulation gives you confidence. Real world testing humbles you. Always."*

---

## 2. PPS (Pulse Per Second) — Desglose tecnico

### 2.1 Que es PPS

PPS (Pulse Per Second) es una **senal electrica que marca el inicio exacto de cada segundo**. Es un pulso digital preciso — tipicamente un flanco de subida de una onda cuadrada que ocurre exactamente una vez por segundo. La precision de la senal PPS es del orden de nanosegundos cuando proviene de un receptor GPS sincronizado con relojes atomicos de satelite.

### 2.2 PPS y el VLP-16

El Velodyne VLP-16 tiene una **entrada PPS** dedicada. Cuando recibe una senal PPS, puede sincronizar sus timestamps internos con esa referencia temporal externa. Esto significa que cada punto de la nube de puntos lleva un timestamp que esta anclado a una referencia temporal absoluta compartida.

### 2.3 PPS y la IMU

Si la IMU tambien esta sincronizada a la **misma senal PPS**, ambos sensores (LiDAR e IMU) comparten una referencia temporal comun. Esto permite:

- Asociar cada punto LiDAR con la orientacion exacta de la IMU en ese instante
- Eliminar la necesidad de interpolacion temporal entre sensores
- Reducir significativamente el ruido en la nube de puntos final

### 2.4 Que pasa sin PPS

Sin sincronizacion PPS, cada sensor usa su **propio reloj interno**. Estos relojes:

- Tienen frecuencias ligeramente diferentes
- Derivan (drift) con el tiempo — microsegundos o milisegundos
- No tienen una referencia comun

**Impacto practico:** Sin PPS, es necesario **interpolar** la orientacion de la IMU para cada punto del LiDAR, lo que introduce ruido. A mayor velocidad de movimiento del sistema, mayor es el error introducido por esta interpolacion. Con PPS, cada punto LiDAR puede asociarse con la orientacion **exacta** de la IMU en ese instante.

---

## 3. Que necesita estar sincronizado

### 3.1 Lista de componentes que requieren sincronizacion

| Componente | Tipo de datos | Frecuencia tipica | Necesidad de sync |
|---|---|---|---|
| VLP-16 (LiDAR) | Puntos 3D (nube de puntos) | 15-31 Hz (frames), 300.000 pts/s | Critica — cada punto tiene timestamp |
| IMU N100 | Aceleracion y rotacion (6/9 ejes) | 100-400 Hz | Critica — orientacion para cada punto |
| AS5600 (encoder) | Angulo de la plataforma rotatoria | 200-451 Hz | Critica — angulo para de-skewing |
| Motor control (Teensy) | Timing de conmutacion | Continuo | Secundaria — afecta indirectamente via encoder |

@andbondstyle enfatizo que la sincronizacion debe cubrir **todos** los componentes relacionados con movimiento y datos: *"you need hardware time sync tied to every motion and data-related component (lidar, IMU, motor encoder etc.)"*

### 3.2 Cadena de sincronizacion ideal

```
Fuente PPS (GPS o generador local)
    |
    +--> VLP-16 (entrada PPS dedicada) --> timestamps de puntos anclados
    |
    +--> IMU N100 --> timestamps de orientacion anclados
    |
    +--> Teensy / AS5600 --> timestamps de angulo anclados
    |
    +--> Raspberry Pi 5 (NTP/PTP local) --> referencia para ROS 2
```

---

## 4. Enfoque de Tyler para la sincronizacion

### 4.1 PPS con RTC

Tyler utiliza **PPS timing con un RTC (reloj de tiempo real)** para la sincronizacion entre subsistemas. Confirmo que **dos senales GPS/PPS pasan a traves del slip ring**, pero que **no usa GPS real** — solo la senal PPS para sincronizacion temporal.

En la transcripcion del Video 3: *"A through it passes the Ethernet line that goes from the sensor to the Raspberry Pi, the power (positive and ground) and two GPS signals. We don't actually use GPS, but we do use the PPS timing with a real-time clock to get precise synchronization between all subsystems: the IMU, the Velodyne points, and the encoder data."*

### 4.2 Senales a traves del slip ring

Las dos senales GPS/PPS pasan a traves del slip ring junto con Ethernet, alimentacion y masa:

| Senal | Tipo | Descripcion |
|---|---|---|
| Ethernet | Gigabit 1000Base-T | Datos de la nube de puntos del VLP-16 hacia el Pi 5 |
| Power | 12V DC | Alimentacion del VLP-16 |
| Ground | GND | Masa |
| GPS/PPS (x2) | Senal digital | PPS timing con RTC para sincronizacion precisa |

### 4.3 Debate sobre GPS en cuevas

**@lordsqueak** cuestiono el uso de GPS dentro de una cueva. Tyler respondio que solo usa la senal PPS del GPS para sincronizacion temporal, no para posicionamiento. Pero @lordsqueak hizo un seguimiento importante:

> *"But how will you get that PPS signal inside a cave? Is there a backup plan?"*

**Esta pregunta quedo sin respuesta.** Es un problema real: si el PPS depende de un receptor GPS, la senal se perdera al entrar en la cueva. Se necesita una fuente PPS alternativa que no dependa de GPS.

---

## 5. Soluciones para sincronizacion temporal sin GPS underground

Estas son las alternativas tecnicas viables para generar o mantener sincronizacion temporal sin senal GPS:

### 5.1 Senal PPS generada localmente

Un microcontrolador (como el Raspberry Pi o un MCU dedicado) puede generar una senal PPS usando su propio oscilador de cristal. La precision no sera de nanosegundos (como GPS), pero puede ser suficiente si el drift del oscilador es pequeno durante la duracion de la mision (minutos a horas).

### 5.2 Hardware timestamping en el bus de datos

Utilizar timestamps de hardware en la capa de comunicacion (por ejemplo, Ethernet PTP — Precision Time Protocol, IEEE 1588) para sincronizar los relojes de los dispositivos conectados al mismo bus. El Pi 5 podria actuar como maestro PTP.

### 5.3 Sincronizacion NTP local entre componentes

Configurar un servidor NTP local en el Raspberry Pi 5 y sincronizar los demas dispositivos contra el. Menos preciso que PPS/PTP pero mas simple de implementar.

### 5.4 Trigger electrico compartido

Una senal de trigger electrico comun (como un GPIO del Pi 5) que se distribuye a todos los sensores simultaneamente, marcando un punto de referencia temporal conocido. Los timestamps de cada sensor se ajustan en post-procesamiento relativo a ese trigger.

---

## 6. Frecuencias de datos observadas (app SLAM Data Recorder)

Tyler construyo una **app movil de control remoto** llamada "SLAM Data Recorder" que se ejecuta en un smartphone y permite monitorear las frecuencias de datos en tiempo real, iniciar/detener grabaciones, y verificar el estado de los sensores. Los datos de frecuencia observados en diferentes sesiones revelan variaciones significativas:

### 6.1 Tabla de frecuencias

| Momento / Sesion | LiDAR (Hz) | IMU (Hz) | Encoder (Hz) | Almacenamiento |
|---|---|---|---|---|
| Campo v1 (IMG_9449) | 23.0 / 30.2 | 199.1 | ~451.1 | — |
| Integracion (IMG_9471) | 17.0 → 15.0 | 100.0 | 320 | 206.4 GB |
| Campo v2 (IMG_9472) | Similar | Similar | Similar | 206.4 GB |
| Handheld (IMG_9458) | 31.0 | 101.0 | ~441.1 | — |
| Campo avanzada (IMG_9474) | — | — | — | 206.8 GB |

### 6.2 Analisis de variaciones

Las variaciones entre sesiones sugieren:

- **Diferentes configuraciones del VLP-16** entre sesiones (el sensor permite configurar la tasa de rotacion interna de 5-20 rev/s, lo que afecta directamente la frecuencia de frames de salida)
- **Evolucion del firmware del Teensy** que cambia la frecuencia de reporte del encoder
- **Configuracion de downsampling de la IMU** (la N100 tiene capacidad de 400 Hz pero se observa a 100-200 Hz, posiblemente downsampled por configuracion)
- **Diferente carga del sistema** que afecta las frecuencias efectivas de publicacion de ROS 2

La frecuencia mas alta del LiDAR (31 Hz) se observa en la version handheld, posiblemente porque Tyler aumento la tasa de rotacion interna del VLP-16 para compensar el movimiento mas rapido del operador.

### 6.3 Datos de Tyler sobre el sistema en operacion

- **Velocidad de rotacion de la plataforma:** ~20 RPM
- **Frecuencia tipica del LiDAR durante grabacion:** 15-16 Hz (banco), hasta 31 Hz (handheld)
- **Datos de un escaneo de 2 minutos:** ~400 MB comprimido, ~1 GB descomprimido, ~21 millones de puntos, submuestreados a ~13.75 millones

---

## 7. Parametros de calibracion temporal en offline_deskew.py

El script `offline_deskew.py` (repositorio `pointcloud-reconstruction`) contiene parametros de calibracion que se editan en la parte superior del archivo. Varios de estos estan directamente relacionados con la sincronizacion temporal y la correccion geometrica:

| Parametro | Valor por defecto | Descripcion |
|---|---|---|
| `ROTATION_AXIS` | `'x'` | Eje alrededor del cual rota la plataforma |
| `ANGLE_OFFSET_DEG` | `184.0` | Offset de calibracion del angulo cero en grados. Compensa la posicion inicial del encoder respecto al sistema de referencia del LiDAR |
| `ROTATION_CENTER` | `[0,0,0]` | Offset del centro optico del LiDAR respecto al eje de rotacion. Compensa el offset de ~5 mm documentado |
| `INVERT_ROTATION` | `False` | Invierte la direccion de rotacion si el encoder reporta en sentido contrario al esperado |
| `ENCODER_TIME_OFFSET_MS` | `0.0` | **Offset temporal entre el reloj del encoder y el reloj del LiDAR en milisegundos.** Este es el parametro clave de sincronizacion temporal — permite compensar cualquier desfase entre los timestamps del encoder (via Teensy/UART) y los timestamps del LiDAR (via Ethernet) |
| `MOUNT_RPY_DEG` | `[0,0,0]` | Roll, pitch y yaw del montaje del LiDAR en grados. Compensa si el sensor no esta perfectamente nivelado en la copa |
| `MOUNT_AXES` | `['+x','+y','+z']` | Remapeo de ejes para el montaje del LiDAR. Permite corregir si la orientacion fisica del sensor no coincide con la orientacion esperada por el software |

### 7.1 Relacion entre parametros y sincronizacion

El parametro `ENCODER_TIME_OFFSET_MS` es el ajuste directo de sincronizacion temporal. Si el encoder y el LiDAR tienen un desfase constante (por ejemplo, porque los datos del encoder pasan por Teensy → UART → Pi 5 mientras que los datos del LiDAR van por Ethernet directo → Pi 5), este parametro lo compensa.

Los parametros `ROTATION_CENTER` y `ANGLE_OFFSET_DEG` son ajustes geometricos, pero estan intimamente relacionados con el timing: si el angulo no esta correctamente alineado temporalmente con los puntos, el de-skewing introduce errores sistematicos que se manifiestan como desalineacion geometrica.

### 7.2 Proceso de de-skewing

El script `offline_deskew.py` corrige el "motion blur" en cada frame de escaneo **interpolando el angulo de rotacion de la plataforma en el momento de adquisicion de cada punto**. Sin este paso, los puntos aparecerian borrosos o duplicados porque el LiDAR se mueve (rota) mientras captura cada revolucion de su espejo interno.

El proceso:
1. Lee los datos crudos del bag ROS 2 (topics `/velodyne_points` y `/rotating_platform/angle`)
2. Para cada punto en cada frame, interpola el angulo exacto de la plataforma en el momento de captura de ese punto
3. Aplica la rotacion inversa para "deshacer" el movimiento de la plataforma
4. El resultado son puntos que aparecen como si hubieran sido capturados con la plataforma estatica

---

## 8. ICP como complemento a la sincronizacion imperfecta

El script `icp_merge.py` aplica el algoritmo ICP (Iterative Closest Point) **sobre** el de-skewing para compensar imperfecciones mecanicas que la sincronizacion temporal no puede resolver: runout del eje, wobble del rodamiento, u otras tolerancias de hardware que causan que las mitades opuestas de 180 grados del escaneo no se alineen perfectamente incluso despues de la correccion de movimiento.

Descripcion del repositorio: *"ICP alignment is applied on top of the deskewing step to compensate for mechanical imperfections — shaft runout, bearing wobble, or other hardware tolerances that cause opposite 180 degree scan halves to not align perfectly even after motion correction. The angular-slice approach gives ICP overlapping geometry between adjacent slices, making registration robust where pure deskewing falls short."*

El ICP merge usa una estrategia de **dos pasadas** (chain → refine → re-chain) para minimizar el drift acumulativo.

---

## 9. Topics ROS 2 relevantes para timing

| Topic ROS 2 | Tipo de mensaje | Descripcion | Fuente |
|---|---|---|---|
| `/velodyne_points` | `sensor_msgs/PointCloud2` | Frames de la nube de puntos del VLP-16 con timestamps | VLP-16 via Ethernet |
| `/rotating_platform/angle` | `std_msgs/Float64` | Angulo de la plataforma en grados | Teensy 4.0 via UART serial |
| (IMU topic, nombre no confirmado) | (tipo no confirmado) | Datos de aceleracion/rotacion de la N100 | IMU N100 via USB serial |

Cada topic en ROS 2 lleva un **header con timestamp** que permite la correlacion temporal entre diferentes fuentes de datos. La precision de estos timestamps depende de:
- La precision del reloj del dispositivo fuente
- La latencia de transmision (Ethernet vs USB vs UART)
- La sincronizacion entre relojes (PPS/RTC)

---

## 10. Contenido de las grabaciones ROS 2

Cada grabacion (bag) de ROS 2 contiene, segun Tyler:
- Datos de la IMU
- Angulo de la plataforma
- Velocidad de la plataforma
- Nube de puntos

Para escaneos estaticos, Tyler usa todo excepto los datos de la IMU (que no son necesarios cuando el sistema no se mueve). Para SLAM en movimiento, todos los datos son necesarios y su sincronizacion temporal es critica.

---

## 11. Impacto de la sincronizacion en el pipeline SLAM

### 11.1 Pipeline actual (escaneos estaticos)

En el pipeline actual de Tyler (escaneos estaticos con de-skewing), la sincronizacion temporal importa principalmente entre el **encoder y el LiDAR**. Si el angulo del encoder no esta temporalmente alineado con los puntos del LiDAR, el de-skewing introduce errores sistematicos. El parametro `ENCODER_TIME_OFFSET_MS` permite ajustar esto manualmente.

### 11.2 Pipeline futuro (SLAM en movimiento)

Cuando Tyler vuelva a integrar SLAM (planea hacerlo una vez que el de-skewing este validado), la sincronizacion temporal sera aun mas critica porque:

- El sistema se mueve (traslacion + rotacion del operador) ademas de la rotacion de la plataforma
- La IMU necesita estar sincronizada con el LiDAR para corregir el movimiento del operador
- El encoder necesita estar sincronizado con ambos para el de-skewing simultaneo
- Cualquier error temporal se amplifica con la velocidad de movimiento

Tyler confirmo: *"Without SLAM, static scanning plus deskewing works great. The problems are with SLAM, where there's massive drift."* El drift masivo podria estar parcialmente causado por sincronizacion temporal insuficiente.

### 11.3 Algoritmo SLAM previsto

El algoritmo SLAM que probablemente usa Tyler es **FAST-LIO** (identificado por @yxlee2676 al reconocer la interfaz en el minuto 2:30 del video 1) o posiblemente un fork de **LIO-SAM** (sugerido por @RYU47376). Tyler confirmo el uso de ROS 2 y FAST-LIO es compatible.

El usuario @franklynd enfatizo que *"drift will always occur without loop closure; closing loops is really important."* La sincronizacion temporal afecta directamente la calidad del loop closure porque determina la precision con la que se pueden comparar puntos capturados en diferentes momentos.

---

## 12. Problemas comunitarios documentados relacionados con timing

| Usuario | Problema | Relevancia |
|---|---|---|
| @MJ-mw3ni | Fallo total con hardware superior (SICK MRS1000/VLP-16 + IMU 9 ejes + RTK GPS). Ningun SLAM funcionaba, datos corruptos incluso con filtro de Kalman | Posible causa: sincronizacion temporal deficiente |
| @weselsmith | Pregunto como asegurar sincronizacion perfecta entre IMU y LiDAR. Sin respuesta del creador | Problema conocido y critico en SLAM |
| @OttoSphalta | IMUs baratos no procesan datos de aceleracion con suficiente frecuencia, causando error acumulativo | Relacionado: frecuencia insuficiente = interpolacion temporal mas pobre |
| @melonenlord2723 | Drift del IMU a largo plazo. Normalmente se corrige con GNSS, no disponible underground | Sin GPS, el drift temporal tambien afecta |
| @janrafflewski5468 | Sorprendido de que el posicionamiento basado en IMU sea confiable. Quiere saber como se reduce el drift | La sincronizacion temporal es parte de la respuesta |

---

## 13. Resumen de la cadena temporal completa

```
                    PPS/RTC (referencia temporal comun)
                              |
         +--------------------+--------------------+
         |                    |                    |
         v                    v                    v
    VLP-16 LiDAR         IMU N100           AS5600 Encoder
    (Ethernet)           (USB serial)       (I2C -> Teensy -> UART)
         |                    |                    |
         v                    v                    v
    /velodyne_points     /imu_data          /rotating_platform/angle
    (con timestamps)     (con timestamps)   (con timestamps)
         |                    |                    |
         +--------------------+--------------------+
                              |
                              v
                    Raspberry Pi 5 (ROS 2)
                              |
                    Grabacion bag ROS 2
                              |
              +---------------+---------------+
              |                               |
              v                               v
    offline_deskew.py                   SLAM (FAST-LIO)
    (ENCODER_TIME_OFFSET_MS)           (requiere sync preciso
    (escaneos estaticos)                de los 3 sensores)
              |
              v
        icp_merge.py
    (compensacion mecanica)
```

La calidad del resultado final — ya sea un escaneo estatico de-skewed o un mapa SLAM completo — depende fundamentalmente de que todos los timestamps en esta cadena esten correctamente alineados. Como dijo Tyler: *"If your sensor streams aren't synced to the millisecond, you're not mapping anything, you're just making art."*
