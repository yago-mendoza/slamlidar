# Referencia Consolidada: Visión General del Sistema SLAM LiDAR — Subterranean Systems

Documento de referencia exhaustivo sobre el proyecto de escáner LiDAR 3D portátil DIY de Tyler (9nl / Subterranean Systems). Toda la información ha sido extraída de las fuentes primarias: transcripciones de tres vídeos de YouTube, comentarios de la comunidad, publicaciones de Instagram, READMEs de tres repositorios GitHub, e inferencias de análisis visual de imágenes.

---

## 1. Visión del proyecto y motivación

### 1.1 El problema

El GPS no funciona bajo tierra: cuevas, minas, parkings subterráneos. Los teléfonos son inútiles. Incluso en exteriores, el GPS solo indica por dónde se pasó — una línea en un mapa. Lo que Tyler busca es capturar el entorno real: modelos 3D completos, cada superficie, cada dimensión, el espacio físico completo.

### 1.2 El mercado actual

Existen empresas que fabrican sistemas de escaneo portátiles con SLAM (Simultaneous Localization and Mapping) que rastrean su propia posición mientras escanean todo alrededor. El problema:

| Aspecto | Sistemas comerciales | Proyecto de Tyler |
|---|---|---|
| Precio | ~$40.000 USD | <$1.000 USD |
| Acceso al funcionamiento | Completamente propietarios, "ninguna comparte su salsa secreta" | Open source, todo documentado |
| Ingeniería inversa | No viable ("no tengo ese dinero") | Código, PCB, CAD publicados |
| Formatos de datos | Propietarios, software de suscripción, sin escalabilidad | Estándares abiertos (PCD, MCAP, ROS 2 bags) |

Un ex-empleado de **Emesent** (fabricante del Hovermap, ~$40.000+) — @peterdickinson1936 — validó el proyecto con un "Great work - subscribed!", lo cual indica que el enfoque DIY es viable según alguien con experiencia interna en la industria.

### 1.3 Aplicación principal: mapeo de cuevas

Los levantamientos tradicionales de cuevas se hacen con cintas métricas, brújulas e inclinómetros. Funciona (miles de kilómetros mapeados en el último siglo), pero es lento, intensivo en mano de obra y con resolución limitada. El LiDAR cambia la ecuación:

- No necesita luz ambiente — trae sus propios fotones (905 nm)
- Captura geometría 3D compleja: voladizos, pozos verticales, formaciones delicadas, sin tocar nada
- No necesita GPS — los algoritmos SLAM construyen el mapa mientras el operador se mueve
- Datos utilizables para investigación científica, conservación, o compartir modelos 3D con cualquier persona interesada

### 1.4 Origen y duración del proyecto

Tyler lleva aproximadamente 6 meses trabajando en el proyecto (dato de la transcripción del vídeo 2). El objetivo inicial era "simplemente aprender más sobre sistemas autónomos, SLAM y robótica." Publicó los primeros datos de escaneo y la respuesta de la comunidad fue muy positiva, lo que motivó la serie de vídeos.

---

## 2. Información del autor

| Campo | Valor |
|---|---|
| Nombre | Tyler |
| GitHub | tthom289 |
| YouTube | 9nl (canal: @9nl) |
| Instagram | @subterraneansystems |
| Presencia adicional | TikTok (mencionado por @ceater7726) |
| Logo en PCB | "TTHOM" con icono de engranaje, grabado en serigrafía de la carrier board |
| Logo en chasis | "Subterranean Systems" grabado en la carcasa exterior (posiblemente grabado láser o vinyl) |
| Marca del proyecto | "Subterranean Systems" |
| YouTube del proyecto | https://www.youtube.com/@SubterraneanSystems |
| Colaborador de código | Claude (aparece como contributor en repos de GitHub) |

### 2.1 Filosofía del proyecto

- **Open source total**: código, diseños PCB (KiCad), modelos CAD, documentación — todo será publicado
- **Prototipado iterativo**: muchas versiones de cada pieza, especialmente la copa del sensor ("probably one of the most iterated parts")
- **Perfeccionar antes de avanzar**: "I intentionally took a step back from the SLAM integration to really nail this part down. If the points aren't landing where they should, nothing downstream really matters." En español: "perfeccionar deskewing antes de volver a SLAM"
- **"Engineering is just a series of compromises"** — frase del propio Tyler sobre el offset de 5 mm del centro óptico
- **"When your prototype wiring becomes the problem, you design it out"** — sobre la transición de jumper wires a carrier board PCB

---

## 3. Cadena de datos completa

```
VLP-16 (captura 300K pts/s)
    │
    │ Gigabit Ethernet (1000Base-T)
    │
    ▼
Slip ring (LPC-18, contactos rotativos)
    │
    │ Ethernet
    │
    ▼
Raspberry Pi 5 (ROS 2 — graba bags)
    ▲           ▲
    │           │
    │ USB       │ Serial UART (TX1/RX1)
    │ serial    │
    │           │
IMU N100    Teensy 4.0
            ▲           │
            │           │ 3x PWM + EN
            │ I2C       │
            │(SDA/SCL)  ▼
            │       SimpleFOC V1
        AS5600          │
        (encoder)       │ 3 fases
                        ▼
                    Motor 4015
                    (BLDC gimbal)
```

### 3.1 Descripción del flujo

1. El **VLP-16** dispara 300.000 pulsos láser/segundo, gira internamente a ~600 RPM, y entrega puntos 3D procesados + datos de intensidad vía **Gigabit Ethernet**
2. Los datos Ethernet pasan por el **slip ring** (parte superior rota con el sensor, parte inferior fija al chasis)
3. La **Raspberry Pi 5** recibe los datos Ethernet del LiDAR, datos USB serial de la **IMU N100**, y datos UART serial del **Teensy 4.0** (ángulo del encoder). Graba todo en **bags ROS 2** (formato MCAP, opcionalmente comprimidos con zstandard)
4. El **Teensy 4.0** lee la posición angular del eje mediante el encoder magnético **AS5600** por **I2C**, y transmite esos datos al Pi 5 por **UART serial**. Simultáneamente, controla el motor **4015** mediante la placa **SimpleFOC V1** con señales **PWM** trifásicas
5. Las señales **PPS** (Pulse Per Second) del GPS (no se usa GPS real, solo la señal de timing con reloj de tiempo real) pasan por el slip ring para conseguir sincronización precisa entre IMU, puntos del Velodyne y datos del encoder

---

## 4. Lista de materiales (Bill of Materials) completa

### 4.1 Componentes principales con precios

| Componente | Modelo / Especificación | Precio | Fuente | Notas |
|---|---|---|---|---|
| **LiDAR** | Velodyne VLP-16 "Puck", 16 canales, ToF, 905 nm, Clase 1 | $321 USD (segunda mano) | eBay | Retail nuevo: ~$8.000. En producción desde 2014. IP67, -10 a 60°C |
| **IMU** | Wheeltech N100, 9 ejes (accel+gyro+mag), "ROS" compatible | ~$30-50 | Online | Mide aceleración y rotación ~400 veces/segundo |
| **Computadora** | Raspberry Pi 5 | ~$80 | — | Con ROS 2. Disipador naranja/cobre visible |
| **Microcontrolador** | Teensy 4.0 (PJRC) | ~$25-30 | — | Controlador del motor + lectura encoder + transmisión UART al Pi |
| **Driver del motor** | SimpleFOC V1 | ~$20-30 | SimpleFOC.com | Placa open source, convierte PWM en corriente trifásica |
| **Encoder magnético** | AS5600, 12 bits (4096 posiciones/rev), I2C | ~$2-5 | — | Breakout board con header de pines |
| **Motor BLDC** | 4015 gimbal motor | ~$15-30 | — | Reemplazó al 2804 original (3x más torque, menos calor) |
| **Slip ring** | Gigabit Ethernet, RJ45, 12-18 circuitos | ~$30-50 | eBay: "Gigabit Ethernet Slip Ring 1000Base-T 1 Cable Power Signal Flange RJ45 Joint" | Búsqueda exacta confirmada por Tyler en comentarios |
| **Correa** | 2GT, 200 mm, 6 mm de ancho | ~$2-5 | — | Transmisión por correa dentada |
| **Engranaje 60T** | 60 dientes, 2GT | ~$3-8 | — | Prensado sobre el eje del sensor |
| **Engranaje 20T** | 20 dientes, 2GT | ~$2-5 | — | En el eje del motor |
| **Carrier board PCB** | TeensyFOC-Carrier (diseño propio, KiCad) | ~$2-8 por 5 PCBs + envío | JLCPCB | 2 capas, soldermask verde. Logo "TTHOM" en serigrafía |
| **Batería** | 12V Li-ion (celdas azules LiPo/Li-ion) | ~$20-40 | — | Alimenta VLP-16, Pi, IMU, HMI |
| **Power bank USB** | Portátil, para Teensy | ~$10-20 | — | Separado para facilitar conexión al laptop |
| **Rodamientos** | Bolas reales, 2" OD × 1" ID | ~$5-15 par | — | Reemplazan los rodamientos impresos en 3D del prototipo |
| **Eje de aluminio** | 1" OD, hueco | ~$5-10 | — | Permite paso del conector GX12 por el interior |
| **Conector GX12** | Circular metálico con rosca | — | Ya instalado en el VLP-16 comprado | Pasa por el eje hueco |
| **Conectores** | Headers 2.54mm + Kit JST-XHP | ~$5-10 | Amazon | Para la carrier board |
| **Piezas impresas 3D** | Copa del sensor, soportes motor, housing rodamientos, brackets varios | Coste de filamento | — | PLA, ASA, PETG probados. PLA funciona mejor para la copa |
| **Interruptor manual** | Para motor | ~$1-3 | — | Corta solo la rotación sin apagar el resto |
| **Fuente DC** | Para testing de banco | — | — | Más cómodo que batería para sesiones largas |

### 4.2 Resumen de costes

| Categoría | Coste estimado |
|---|---|
| LiDAR (VLP-16 usado) | $321 |
| Computación (Pi 5 + Teensy) | ~$105-110 |
| Sensores (IMU + encoder) | ~$32-55 |
| Electrónica (SimpleFOC + PCB + conectores) | ~$30-50 |
| Mecánica (motor + correa + engranajes + rodamientos + eje) | ~$30-75 |
| Transferencia señal (slip ring) | ~$30-50 |
| Alimentación (batería + power bank) | ~$30-60 |
| **TOTAL** | **< $1.000 USD** |

Tyler: "En total, estamos hablando de poco menos de 1.000 dólares en hardware, pero demasiadas horas invertidas."

### 4.3 Alternativas de LiDAR mencionadas

| Sensor | Canales | Precio aprox. | Ventajas | Desventajas |
|---|---|---|---|---|
| **Velodyne VLP-16** (usado) | 16 | $321 eBay | Ecosistema maduro, drivers ROS probados, IP67, comunidad masiva | Solo 30° FOV vertical, tecnología de 2014 |
| **Hesai 40ch** | 40 | ~$200 eBay | 2.5x más canales por la mitad del precio | Menos documentación comunitaria (según disponibilidad) |
| **Livox Mid-360** | N/A (non-repetitive) | Más caro que VLP-16 | Increíble densidad de puntos/dólar, patrón no-repetitivo bueno para SLAM | Patrón de escaneo complejo de manejar |
| **Ouster OS1** | Hasta 128 | Mucho más caro | Más resolución, imágenes tipo cámara junto con point cloud | Mayor consumo, mayor coste |
| **RPLiDAR serie** | 1 (2D) | Pocos cientos USD | Barato, bueno para mapeo 2D simple | Limitado para cobertura 3D completa |
| **Robosense Airy** | Solid-state | — | Compacto | Menos documentado para este uso |
| **Unitree L2** | — | — | Paquetes SLAM integrados | Menos control, menos aprendizaje |
| **Hesai PandarXT** | — | — | Visualmente similar al Velodyne | — |

---

## 5. Arquitectura de alimentación

### 5.1 Diagrama de alimentación

| Fuente | Componentes alimentados | Tipo | Notas |
|---|---|---|---|
| **Batería 12V Li-ion** | VLP-16, Raspberry Pi 5, IMU N100, HMI (pantalla/app) | DC 12V | Fuente principal del sistema. Celdas azules LiPo/Li-ion visibles en interior de la caja |
| **Power bank USB** | Teensy 4.0 / SimpleFOC motor controller | USB 5V | Separado del sistema principal para facilitar la conexión frecuente al laptop para cambios de firmware, tuning del encoder, o ajustes de la función de rotación |
| **Fuente DC de banco** | Todo el sistema | DC 12V | Sustituto de la batería para testing en interior ("más cómodo") |
| **Interruptor manual** | Motor (vía SimpleFOC) | Interruptor ON/OFF | "Con todo el testing se deja el sistema encendido y el giro constante hace mucho ruido" — el switch corta solo la rotación |

### 5.2 Secuencia de encendido

1. Conectar la batería de 12V → arranca Raspberry Pi, HMI, IMU y sensor LiDAR
2. Conectar la alimentación separada del Teensy/motor controller
3. Asegurar que el interruptor del motor está encendido
4. El sistema inicializa y comienza a girar a ~20 RPM
5. Esperar a que todo arranque en el lado del Pi ("todo en verde")
6. Colocar el sistema en posición deseada (vertical para escaneo)
7. Iniciar la grabación desde la app "SLAM Data Recorder"

---

## 6. Buses de comunicación

### 6.1 Tabla de buses

| Bus | Origen | Destino | Señal / Datos | Medio físico | Notas |
|---|---|---|---|---|---|
| **Gigabit Ethernet (1000Base-T)** | VLP-16 | Raspberry Pi 5 | Nube de puntos 3D + intensidad, ~300K pts/s | Cable Ethernet → slip ring → cable Ethernet | Pasa por el eje hueco y el slip ring. Requiere slip ring específico de Gigabit Ethernet |
| **USB serial** | IMU Wheeltech N100 | Raspberry Pi 5 | Aceleración + rotación + magnetómetro, ~100-400 Hz | Cable USB-C negro | Comunicación típica de la N100 |
| **UART serial (TX1/RX1)** | Teensy 4.0 | Raspberry Pi 5 | Datos del encoder (ángulo de la plataforma rotatoria) | Cable con conector JST, serial 3 pines (GND, TX, RX) | "This transmits the encoder serial data to the Pi 5" |
| **I2C (SDA/SCL)** | AS5600 encoder | Teensy 4.0 | Posición angular absoluta, 12 bits (4096 posiciones/rev) | 4 cables: VCC (rojo), GND (marrón), SDA (verde), SCL (amarillo) | Conector JST blanco |
| **PWM trifásico** | Teensy 4.0 | SimpleFOC V1 → Motor 4015 | 3 señales PWM (IN1, IN2, IN3) + Enable (EN) | Cables de la carrier board al SimpleFOC, luego cables de fase al motor | Conmutación de motor BLDC |
| **PPS/GPS** | (señal de timing) | A través del slip ring | 2 señales GPS/PPS para sincronización temporal | Cables dentro del slip ring | No es GPS real — se usa PPS timing con RTC para sincronizar IMU, Velodyne y encoder |

### 6.2 Tópicos ROS 2 (bags grabados)

| Tópico ROS 2 | Tipo de mensaje | Descripción |
|---|---|---|
| `/velodyne_points` | `sensor_msgs/PointCloud2` | Frames de nube de puntos del VLP-16 |
| `/rotating_platform/angle` | `std_msgs/Float64` | Ángulo del encoder de la plataforma en grados |
| IMU (nombre no confirmado) | — | Datos de la IMU N100 |
| Velocidad de plataforma (nombre no confirmado) | — | Velocidad de rotación de la plataforma |

---

## 7. Aplicación móvil "SLAM Data Recorder"

Tyler construyó una app de control remoto que se ejecuta en un smartphone (con funda azul) para operar el escáner sin necesidad de SSH al Raspberry Pi. Permite iniciar/detener grabaciones, monitorear el estado de los sensores y verificar las frecuencias de datos. En la interfaz se observa una lista de "Topics:" (probablemente ROS topics monitoreados).

### 7.1 Elementos de la interfaz

| Elemento | Descripción |
|---|---|
| Título | "SLAM Data Recorder" |
| Estado de conexión | "CONNECTED" (indicador verde) |
| Estado de sensores | "Sensors: ON" |
| Frecuencia LiDAR | Variable según configuración: 15.0 Hz, 17.0 Hz, 23.0 Hz, 31.0 Hz |
| Frecuencia IMU | Variable: 100.0 Hz, 101.0 Hz, 199.1 Hz |
| Frecuencia encoder | Variable: 320 Hz, ~451.1 Hz |
| Almacenamiento disponible | ~206 GB (observado en múltiples fotos) |
| Botón principal | "START RECORD" (verde) / "STOP RECORD" (rojo) |
| Tiempo de grabación | Contador en segundos (ej: "REC: 038") |

### 7.2 Datos de frecuencia observados en diferentes momentos/configuraciones

| Momento / Foto | LiDAR (Hz) | IMU (Hz) | Encoder (Hz) | Almacenamiento |
|---|---|---|---|---|
| Campo v1 (IMG_9449) | 23.0 / 30.2 | 199.1 | ~451.1 | — |
| Integración (IMG_9471) | 17.0 > 15.0 | 100.0 | 320 | 206.4 GB |
| Campo v2 (IMG_9472) | Similar | Similar | Similar | 206.4 GB |
| Handheld (IMG_9458) | 31.0 | 101.0 | ~441.1 | — |
| Campo avanzada (IMG_9474) | — | — | — | 206.8 GB |

Las variaciones en frecuencias sugieren configuraciones diferentes entre sesiones o evolución del firmware del Teensy y del pipeline ROS 2.

---

## 8. Cronología de evolución del hardware

### 8.1 Fases documentadas

| Fase | Descripción | Indicadores clave |
|---|---|---|
| **Banco (prototipo inicial)** | Mecanismo rotatorio desnudo fijado con abrazadera DeWalt a la mesa del taller. Cup gris/blanco cerrado (cilíndrico). Motor 2804. Eje de aluminio 3/4". Rodamientos impresos en 3D con bolas de acero tipo BB | Chasis gris, soportes grises, cableado desordenado con jumper wires y breadboards |
| **Integración** | Todo dentro de la caja gris de aluminio con texturizado fino. Pantalla de control funcional. Carrier board soldada. Motor actualizado de 2804 a 4015 (3x más torque, menos calor). Eje actualizado de 3/4" a 1" (hueco). Rodamientos de bolas reales (2" OD × 1" ID). Engranaje 60T prensado. Logo "Subterranean Systems" en carcasa | Caja gris aluminio, piezas azules aparecen, carrier board JLCPCB verde |
| **Campo v1** | Primera salida exterior. Caja abierta con cables expuestos. Primeras pruebas estáticas en el patio trasero de Tyler | Exterior, caja gris, configuración de campo |
| **Campo v2** | Soporte handheld azul. Cup naranja (rediseño de cilindro cerrado a horquilla abierta en U). Motor 4015 ya instalado. Soportes adicionales para corregir deflexión de la copa | Piezas naranjas dominantes, diseño más robusto |
| **Refinamiento** | Herramientas custom de de-skewing. Visor flythrough OpenGL personalizado. "Motor upgrade + building custom tools to get the deskewing right before moving back to SLAM. No shortcuts." Publicado alrededor del 27 de febrero | Post Instagram con fotos en cocina, desarrollo de software intensivo |
| **Handheld (versión actual)** | Versión compacta con empuñaduras ergonómicas (izquierda de apariencia madera, derecha posiblemente impresa en 3D). Caja negra compacta. Smartphone montado/sujeto al escáner con pantalla visible mostrando "SLAM Data Recorder". Cables Ethernet azules gruesos tipo CAT6. LiDAR operando a 31 Hz | Diseño más pulido, ergonómico y usable en campo |

### 8.2 Evolución de piezas específicas

**Copa del sensor (pieza más iterada del proyecto):**

| Versión | Material | Color | Forma | Resultado |
|---|---|---|---|---|
| Tempranas | ASA | Negro | Cuenco cerrado cilíndrico | FALLO: se partían por las líneas de capa |
| Tempranas | PETG | Varios | Cuenco cerrado cilíndrico | FALLO: se partían por las líneas de capa |
| Intermedia | PLA | Gris/blanco | Cuenco cerrado cilíndrico | Funcional pero se deformaba con el calor del motor 2804 |
| Actual | PLA/PETG | Naranja | Horquilla abierta en U | Funcional — mejor ventilación, menos masa, mejor acceso al conector |

**Motor:**

| Parámetro | Motor original (2804) | Motor actual (4015) |
|---|---|---|
| Tipo | BLDC gimbal, 28 mm estator | BLDC gimbal, 40 mm estator |
| Torque relativo | Base | ~3x más |
| Generación de calor | Excesiva (deformaba piezas PLA constantemente) | Menor — "I can run it a whole lot slower, generates less heat" |

**Eje de rotación:**

| Parámetro | Prototipo | Versión actual |
|---|---|---|
| Diámetro exterior | 3/4" (19 mm), sólido | 1" (25.4 mm), hueco |
| Razón del cambio | — | Permite paso del conector GX12 del VLP-16 por el interior |

**Rodamientos:**

| Parámetro | Prototipo | Versión actual |
|---|---|---|
| Tipo | Impresos en 3D con bolas de acero BB | Rodamientos de bolas reales |
| Dimensiones | — | 2" OD × 1" ID |
| Instalación | Manual | Prensado con martillo grande y vaso de llave |

### 8.3 Chasis y estructura fisica — detalles adicionales

**Caja de electronica:**
- Fase temprana: aluminio gris con texturizado fino de pintura. Tornillos Allen negros de cabeza cilindrica. Logo "Subterranean Systems" grabado en el exterior.
- Fase actual: carcasa negra/oscura mas compacta.
- Abertura en forma de "T" invertida en la cara superior: parte horizontal superior contiene la IMU, parte vertical inferior permite paso de cables entre secciones.
- **Prensaestopas/cable glands** (componentes negros roscados): permiten paso de cables a traves de la pared manteniendo proteccion mecanica.
- Bordes de aberturas cortados de forma funcional pero no perfectamente acabados (estilo prototipado).

**Piezas impresas en 3D por color/funcion:**

| Color | Funcion observada |
|---|---|
| Naranja | Soporte del motor, cup del sensor (version actual en U), soportes internos, brackets internos. Color popular en prototipos por alta visibilidad |
| Azul | Soportes o guias de cables cerca del eje, piezas internas de la caja. Posiblemente componentes auxiliares como handle/mango |
| Gris/blanco | Housing del rodamiento, bearing cap, soportes del eje, iteraciones tempranas del cup |
| Negro | Iteraciones tempranas del cup (ASA) |

---

## 9. Estación de desarrollo

### 9.1 Hardware de Tyler

| Componente | Detalle |
|---|---|
| Laptop principal | Lenovo, Intel Core i9, NVIDIA GeForce RTX (pegatinas visibles). Máquina potente adecuada para procesamiento de nubes de puntos, renderizado OpenGL, y eventualmente SLAM en PC |
| Segundo laptop | Laptop gaming con teclado retroiluminado verde/amarillo, teclado numérico. Posiblemente ASUS o MSI. Observado con CloudCompare abierto |
| Monitor principal | Dell (texto "DELL" legible en bisel inferior) |
| Segundo monitor | Observado mostrando terminal con texto verde sobre fondo oscuro (posiblemente output ROS 2 o logs del sistema) |
| Cutting mat | Verde, con cuadrícula en milímetros y pulgadas (alfombrilla estándar de estación de electrónica/maker) |
| Software de visualización | CloudCompare v2.11.1 (Anoia) [64-bit], visor flythrough OpenGL personalizado (Python + GLFW + PyOpenGL) |
| Software de procesamiento | Python 3.11, Open3D, numpy, MCAP libraries |
| Plataforma de desarrollo firmware | PlatformIO (para Teensy 4.0) |
| Diseño PCB | KiCad (open source) |
| Fabricante PCB | JLCPCB (caja azul con slogan "Accelerating Your Innovation") |

### 9.2 Otros elementos del taller observados

- Drone FPV de carreras colgado en la pared (hélices y guardas visibles) — confirma experiencia previa en electrónica/robótica
- Bobina de soldadura dorada sobre la mesa (estación de soldadura activa)
- Documento/poster/datasheet negro con texto blanco y diagrama de cables (posiblemente pinout del motor o encoder)
- Anillo de luz o micrófono con soporte (para creación de contenido)
- Abrazadera DeWalt amarilla para fijación del sistema durante pruebas de banco
- Trípode pequeño con rótula roja marca Ulanzi (accesorio de cámara/vídeo reutilizado como soporte estático para el VLP-16)

---

## 10. Datos cuantitativos del vídeo 1

| Métrica | Valor | Fecha de referencia |
|---|---|---|
| Suscriptores del canal | 24.300 | Al momento de la extracción de comentarios |
| Visualizaciones (vídeo 1) | ~167.000 | Al momento de la extracción |
| Likes (vídeo 1) | ~9.100 | Al momento de la extracción |
| Comentarios (vídeo 1) | ~448 | Total analizados |
| Fecha de publicación (vídeo 1) | 17 enero 2026 | Confirmado en metadata |

### 10.1 Datos de rendimiento de escaneo

| Métrica | Valor |
|---|---|
| Puntos por segundo del VLP-16 | ~300.000 |
| Escaneo estático de 2 minutos | ~21 millones de puntos crudos |
| Puntos submuestreados (archivo final) | ~13,75 millones |
| Tamaño comprimido (bag) | ~400 MB |
| Tamaño descomprimido | ~1 GB |
| Velocidad de rotación de la plataforma | ~20 RPM |
| Frecuencia del LiDAR durante grabación de banco | 15-16 Hz |
| Frecuencia del LiDAR en versión handheld avanzada | Hasta 31 Hz |
| Almacenamiento disponible observado | ~206 GB |

---

## 11. Presencia online y publicaciones

### 11.1 Tres vídeos de YouTube publicados

| # | Título / Tema | URL | Contenido principal |
|---|---|---|---|
| 1 | "I Built a Scanner that Sees in Total Darkness" — Visión general del sistema | https://www.youtube.com/watch?v=sGpfHk-vEkg | Problema y visión, componentes, por qué rotar, próximos pasos |
| 2 | "The LiDAR That Changed Robotics (And Why I Bought One)" — El LiDAR en detalle | https://www.youtube.com/watch?v=R4zUKJqoBcI | Fundamentos LiDAR, tipos, aplicaciones, VLP-16 en detalle, alternativas |
| 3 | "My LIDAR Was Half Blind (so I fixed it)" — Plataforma rotatoria | https://www.youtube.com/watch?v=JH7F9xr6yL0 | Diseño mecánico, construcción, transmisión, carrier board, pruebas estáticas, resultados, fusión de escaneos |

### 11.2 Tres repositorios de GitHub

| Repositorio | URL | Lenguaje | Stars | Forks | Licencia | Descripción |
|---|---|---|---|---|---|---|
| **pointcloud-reconstruction** | https://github.com/tthom289/pointcloud-reconstruction | Python 100% | 36 | 4 | MIT | Offline deskewing de nubes de puntos VLP-16 con encoder angles + ICP merge + visor flythrough OpenGL |
| **vlp16-spin-controller** | https://github.com/tthom289/vlp16-spin-controller | C++ 100% | 18 | 0 | MIT | Firmware Teensy 4.0 SimpleFOC para motor GM2804/4015 BLDC + AS5600 encoder, open-loop velocity control con output UART al Pi 5 |
| **TeensyFOC-Carrier** | https://github.com/tthom289/TeensyFOC-Carrier | KiCad (PCB) | 15 | 0 | MIT | Carrier board Teensy 4.0 + SimpleFOC para motor control. Integra AS5600, slip ring passthrough, GT2 belt drive circuitry |

### 11.3 Contenido de Instagram (@subterraneansystems)

Publicaciones documentadas incluyen:

- Copa rotacional y alineación del centro óptico
- Upgrade del motor (2804 → 4015) y visor custom de nubes de puntos
- Selección del VLP-16 y razones
- Custom PCB / carrier board para Teensy 4.0 (unboxing de JLCPCB)
- Slip ring (explicación de funcionamiento)
- Overview / pitch general del proyecto
- Captions técnicos sobre deskewing, registro de 14 millones de puntos, lecciones aprendidas, sincronización de datos

### 11.4 Hashtags utilizados

`#robotics` `#engineering` `#ros2` `#LiDAR` `#3dscanning` `#techdiy` `#PCBDesign` `#ai`

---

## 12. Pipeline de software detallado

### 12.1 Procesamiento offline (escaneos estáticos, sin SLAM)

El pipeline está escrito íntegramente en **Python** y se ejecuta en **Windows 11** (Python 3.11 recomendado — Open3D no soporta Python 3.13+).

**Etapa 1 — De-skewing (`offline_deskew.py`):**
Corrige el motion blur en cada frame interpolando el ángulo de rotación de la plataforma en el tiempo de adquisición de cada punto. Parámetros de calibración editables incluyen: eje de rotación, offset angular, centro de rotación, inversión de rotación, offset temporal encoder-LiDAR, y orientación de montaje.

**Etapa 2 — ICP Merge (`icp_merge.py`):**
Divide el escaneo completo en slices angulares con solapamiento, aplica deskewing a cada frame, y usa ICP (Iterative Closest Point) para alinear todos los slices. Estrategia de dos pasadas (chain → refine → re-chain) para minimizar drift acumulativo. Compensa imperfecciones mecánicas: runout del eje, wobble del rodamiento, tolerancias del hardware.

**Etapa 3 — Visor flythrough (`flythrough.py`):**
Visor de primera persona construido con OpenGL (GLFW + PyOpenGL). Soporta: navegación WASD, modos de color (sólido, por altura, por intensidad), Eye-Dome Lighting (EDL), filtro de outliers estadístico, carga de múltiples nubes, modo de comparación, modo grab (alineación manual interactiva), modo pick (alineación por correspondencia de puntos con SVD de Kabsch + refinamiento ICP), fusión y guardado.

### 12.2 SLAM (estado actual)

Tyler se alejó deliberadamente del SLAM para perfeccionar el deskewing estático:

> "I intentionally took a step back from the SLAM integration to really nail this part down. If the points aren't landing where they should, nothing downstream really matters. Once de-skewing is solid, the SLAM will go back in."

> "Sin problemas con grabaciones estáticas y deskewing. Los problemas son con SLAM, donde hay drift masivo."

El algoritmo SLAM planeado/probado es **FAST-LIO** (mencionado en la transcripción del vídeo 1). La interfaz del minuto 2:30 fue identificada por @yxlee2676 como FAST-LIO. Un fork de **LIO-SAM** también fue sugerido como alternativa por @RYU47376.

### 12.3 Planes futuros de software

- Cámaras RGB para **Gaussian splatting** (confirmado por Tyler en comentarios del vídeo 3)
- **VIO (Visual Inertial Odometry)** para complementar SLAM (confirmado por Tyler)
- Integración de SLAM una vez validado el deskewing
- Upgrade a **Jetson Orin Nano Super** para procesamiento en tiempo real (cuando haya presupuesto)

### 12.4 Herramientas externas utilizadas

| Herramienta | Versión / Detalle | Uso |
|---|---|---|
| CloudCompare | v2.11.1 (Anoia) [64-bit] | Visualización avanzada, fusión de múltiples escaneos, SOR (Statistical Outlier Removal) |
| ROS 2 | En Raspberry Pi 5 | Framework de comunicación entre sensores, grabación de bags |
| PlatformIO | En VS Code | Desarrollo del firmware del Teensy 4.0 |
| SSH | Desde laptop al Pi 5 | Acceso remoto al sistema de captura |

---

## 13. Detalles técnicos del subsistema de rotación

### 13.1 Especificaciones del VLP-16

| Parámetro | Valor |
|---|---|
| Fabricante | Velodyne (ahora Ouster tras adquisición) |
| Modelo | VLP-16 "Puck" |
| Tipo | LiDAR de tiempo de vuelo (ToF), 16 canales |
| Longitud de onda | 905 nm (infrarrojo cercano) |
| Clase de seguridad láser | Clase 1 (seguro para los ojos) |
| Puntos por segundo | ~300.000 |
| Canales | 16 haces láser apilados verticalmente |
| FOV vertical | 40° (±15° desde el centro óptico horizontal, o 30° según otras fuentes — discrepancia entre vídeos: Tyler dice 30° en vídeo 1, 40° en vídeo 3) |
| FOV horizontal | 360° (rotación interna continua) |
| Velocidad de rotación interna | ~600 RPM (configurable 5-20 rev/s; Tyler usa ~10 Hz) |
| Precisión | ±3 cm (según datasheet, confirmado por @sharknight11) |
| Protección | IP67 |
| Temperatura de operación | -10 a 60°C |
| Datos de salida | Puntos 3D + intensidad vía Gigabit Ethernet |
| Dimensiones | ~103 mm diámetro × 72 mm alto |
| Peso | ~830 g |
| Conector | GX12 (ya instalado en unidad comprada) |

### 13.2 Offset del centro óptico

| Parámetro | Valor |
|---|---|
| Offset entre centro óptico del VLP-16 y eje de rotación | ~5 mm |
| Configuración ideal | 0 mm (alineados — deskewing mucho más fácil) |
| Razón del offset | Con offset 0, la masa rotacional quedaba desbalanceada causando vibraciones |

Tyler: "Ideally, the rotation axis runs straight through the optical center of the VLP-16... clean math, clean point cloud. But the cup geometry pushed the COG far enough off-axis that vibration and imbalance made that impossible. Had to offset the axis, which means accounting for that in the SLAM pipeline. Engineering is just a series of compromises."

### 13.3 Transmisión por correa

| Parámetro | Valor |
|---|---|
| Correa | 2GT, 200 mm, 6 mm de ancho |
| Engranaje del sensor (eje) | 60 dientes |
| Engranaje del motor | 20 dientes |
| Ratio de reducción | 3:1 |
| Velocidad resultante de la plataforma | ~20 RPM |

### 13.4 Slip ring

| Parámetro | Valor |
|---|---|
| Modelo referencia visual | LPC-18 (capsule slip ring, fabricante slipring.cn) |
| Circuitos totales | 18 (Tyler menciona 12 en un vídeo — posible discrepancia o modelo diferente) |
| Circuitos usados | 3-4: power, ground, Ethernet data+, Ethernet data- (resto cortados y apartados) |
| Velocidad máx. | Típicamente 250-300 RPM (sistema opera a ~20 RPM) |
| Vida útil típica | >10 millones de revoluciones |
| Precio | ~$30-50 en eBay |
| Búsqueda exacta en eBay | "Gigabit Ethernet Slip Ring 1000Base-T 1 Cable Power Signal Flange RJ45 Joint" (confirmado dos veces por Tyler: a @2ndgameX y a @MicroRobo) |
| Señales que pasan | Ethernet Gigabit (datos VLP-16), Power 12V DC, Ground, GPS/PPS ×2 (timing) |

Tyler sobre duración: "Con el slip ring podría funcionar indefinidamente, o hasta que el slip ring se desgaste, pero seguramente algo más fallaría antes."

### 13.5 Carrier board — TeensyFOC-Carrier

| Parámetro | Valor |
|---|---|
| Diseñador | Tyler (tthom289) |
| Logo en serigrafía | "TTHOM" con icono de engranaje |
| Software de diseño | KiCad (open source) |
| Fabricante | JLCPCB (~$2-8 por 5 PCBs + envío) |
| Capas | 2 |
| Color soldermask | Verde (estándar JLCPCB) |
| Agujeros de montaje | 4 (esquinas) |

**Pines expuestos en serigrafía:**

| Etiqueta | Función |
|---|---|
| GND | Masa |
| IN1, IN2, IN3 | Inputs del driver motor BLDC (3 PWM para conmutación trifásica) |
| EN | Enable (habilitación del driver) |
| SDA, SCL | Bus I2C para encoder AS5600 |
| VDD | Alimentación positiva encoder/lógica |
| TX1, RX1 | Serial UART hacia Raspberry Pi 5 (datos encoder) |
| Vin | Alimentación general |
| 13-17 | GPIO del Teensy no usados, expuestos para uso futuro |

**Conectores:**

- Socket DIP hembra para Teensy 4.0 (centro de la placa)
- Al menos 3 conectores JST-XHP blancos (alimentación/Pi, serial TX/RX, encoder/motor)
- Header compatible con SimpleFOC motor driver
- Interfaz I2C para AS5600
- Soporte para paso de cables del slip ring

### 13.6 IMU Wheeltech N100

| Parámetro | Valor |
|---|---|
| Fabricante | Wheeltec (WHEELTEC FOISYSTEMS) |
| Modelo | N100 |
| Ejes | 9 (acelerómetro 3 + giróscopo 3 + magnetómetro 3). Etiqueta: "9axisIMU" / "9轴IMU" |
| Compatibilidad | "ROS" marcado en etiqueta |
| Frecuencia de muestreo | 400 Hz (especificación), observada 100-200 Hz en la app |
| Comunicación | USB serial vía cable USB-C negro |
| Montaje | Cara superior interna de la caja, en ventana rectangular |
| Marcas de orientación | Flecha de cinta adhesiva blanca apuntando hacia el eje (dirección "forward"), cinta naranja/ámbar alrededor del perímetro |

Tyler sobre la importancia de la IMU: "La calidad de la IMU importa mucho más de lo que pensaba inicialmente; haré un vídeo entero dedicado a por qué."

### 13.7 Encoder AS5600

| Parámetro | Valor |
|---|---|
| Modelo | AS5600 |
| Tipo | Encoder magnético rotatorio absoluto |
| Resolución | 12 bits (4096 posiciones/revolución = ~0,088° por paso) |
| Protocolo | I2C (SDA, SCL) |
| Frecuencia de reporte | ~200-451 Hz (varía entre configuraciones observadas en la app) |
| Principio | Mide posición angular del eje del motor mediante imán montado en el eje |

---

## 14. Resultados de escaneo observados

### 14.1 Objetos distinguibles en escaneos del patio trasero

Los escaneos estáticos del patio de Tyler (2 minutos, ~13,75 millones de puntos) muestran resolución suficiente para distinguir:

- Columpio colgando de un árbol
- Cobertizos
- Línea de la valla (usada como punto de referencia para fusión de escaneos)
- Hendiduras/indentaciones en el suelo dejadas por un camión de hormigón (aparecen exageradas en la nube de puntos)
- Cubierta de la parrilla
- Nevera Yeti
- Líneas eléctricas
- Árboles con gran detalle (ramas y hojas como clusters difusos de puntos)
- Líneas de tejas del tejado
- Barandillas y porches
- Postes de teléfono (buenas referencias a larga distancia para fusión)
- Laterales de la casa

### 14.2 Artefactos y limitaciones observadas

| Artefacto | Causa |
|---|---|
| Zonas de sombra (sin datos) | LiDAR sin línea de visión directa |
| Variación en densidad | Más densa cerca, más dispersa a distancia |
| Ruido inherente | Precisión de ±3 cm del VLP-16 |
| Ventanas como huecos vacíos | El vidrio no refleja consistentemente el láser 905 nm |
| Imperfecciones del terreno exageradas | Las ondulaciones del suelo se ven más pronunciadas en la nube de puntos |

### 14.3 Fusión de múltiples escaneos

En el vídeo 3, Tyler demuestra la fusión de un escaneo frontal y uno trasero de su casa usando CloudCompare:

1. Abrir dos archivos PCD simultáneamente (F2 para ver independientemente)
2. Identificar puntos compartidos: línea de valla, lateral de casa, pico del tejado
3. Marcar 3+ pares de puntos correspondientes entre ambos escaneos
4. Pulsar G para alinear (Kabsch SVD + ICP refinement). No I, que es otra herramienta de alineación diferente — Tyler las confundió inicialmente
5. Resultado: en menos de cinco minutos, casa completa con frente y laterales fusionados

---

## 15. Desafíos técnicos clave identificados

### 15.1 Sincronización temporal (el factor más crítico según la comunidad)

Tres fuentes independientes confirman su importancia:

| Fuente | Insight |
|---|---|
| @PeaceIndustrialComplex (11 likes, 5 respuestas de Tyler) | PPS sync entre USB IMU y Velodyne reduce ruido significativamente |
| @andbondstyle (sistema similar con Livox Mid-360) | Time sync necesario para TODOS los componentes: LiDAR, IMU, encoder del motor |
| @MJ-mw3ni (fracasó con hardware superior) | Hardware MEJOR que Tyler + filtro de Kalman no fueron suficientes — posible causa: mala sincronización |

Tyler: "Y data timing? If your sensor streams aren't synced to the millisecond, you're not mapping anything, you're just making art."

### 15.2 Drift del SLAM

Tyler confirma: "Sin problemas con grabaciones estáticas y deskewing. Los problemas son con SLAM, donde hay drift masivo." @franklynd enfatiza que "la deriva siempre ocurrirá sin loop closure; cerrar bucles es realmente importante."

### 15.3 Materiales de impresión 3D

Los materiales de ingeniería (ASA, PETG) fallan por delaminación en líneas de capa. PLA funciona mecánicamente pero se deforma con calor. El rediseño de copa cerrada a horquilla en U fue la solución.

### 15.4 Calor del motor

El motor 2804 original generaba tanto calor que deformaba las piezas PLA cercanas constantemente. El upgrade al 4015 (3x más torque, funcionamiento más lento) resolvió el problema.

---

## 16. Algoritmo SLAM y campo de investigación

### 16.1 Algoritmos mencionados

| Algoritmo | Contexto |
|---|---|
| **FAST-LIO** | Algoritmo SLAM probado/planeado por Tyler (identificado por @yxlee2676 en interfaz del minuto 2:30 del vídeo 1) |
| **LIO-SAM** | Alternativa sugerida por @RYU47376 |
| **LVI-SAM** | Fusión LiDAR + visual, sugerida si se añade cámara |
| **GraphSLAM (GTSAM)** | Usado por @benhenderson6966, descrito como "tough to work with" |
| **ICP (Iterative Closest Point)** | Usado por Tyler para alineación de slices angulares en el pipeline estático |

### 16.2 Estado del campo

@RYU47376 (PhD primer semestre) comentó que SLAM ya está bastante maduro, dificultando contribuciones académicas nuevas. @yxlee2676 coincidió. @bomboclaat9215 mencionó que en 2009 SLAM era nuevo, no había software open source ni hardware accesible, y existía "Rat SLAM" (SLAM con una sola cámara). Tyler confirmó que ahora hay muchos recursos open source excelentes.

---

## 17. Comunidad y demanda

### 17.1 Demanda más frecuente en comentarios

| Petición | Frecuencia / Usuarios |
|---|---|
| Acceso al código / repositorio Git | Al menos 6+ comentarios directos (@notexpected, @domenicoacierno2044, @christopaaron, @jsk_0211, @74n3r, @madanm7454, @cckeysify) |
| Lista de hardware / BOM | @jeffs9503, @101pcgamer, @357pandora, @rayxfinkle8328, @petemenuez |
| Tutorial / guía de construcción completa | @daftpunk1270, @cmoreno41, @CrunchySandwich, @bigsmoke8377, @KS97RLA, @madanm7454, @RandomVideos-bq2xn |
| Open source confirmado | @Alice.59, @Tebbit123, @nuclearbwld, @kanunssol1246, @icarusneverfalls1, @Karol.Szykula |
| Servidor Discord / comunidad | @ManuelHouben |
| Patreon para contribuir | @357pandora |

### 17.2 Ofertas de colaboración recibidas

| Usuario | Oferta |
|---|---|
| @_relaxtation | Trabaja en el mismo problema, ofrece colaborar |
| @JohnnysaidWhat | Startup de automatización e IA en upstate NY |
| @nicklaeder5628 | Ingeniero civil y de datos, desarrollo open source para industria |
| @wilsonwj | Acceso a una cueva para pruebas |
| @Martarts | Hardware: VectorNav VN100 (IMU de grado táctico, ~$800-1500) |
| @seaboom7877 | Cotización para tooling de plástico y metal + precios unitarios |
| @000Krim | Colaboración con otro canal de YouTube |
| @NatetheAceOfficial | Testing en cuevas específicas, trabaja con dev de CaveWhere |
| @technicboss9348 | Mecanizado/diseño de piezas metálicas |
| @aerospacengineer1 | Diseño mecánico |
| @LifesAScript | Venderle prototipo a GhostTownLiving como beta test → llevar al mercado |

### 17.3 Colaboración con GhostTownLiving

Múltiples usuarios (@PrebleStreetRecords, @ablazedave, @backacheache, @minerharry, @LifesAScript) sugirieron colaboración con el canal **Ghost Town Living** (2M subs) que tiene cuevas/minas en **Cerro Gordo** y busca mapearlas con LiDAR. Tyler respondió que le gustaría trabajar con él una vez el sistema sea más fiable.

---

## 18. Aplicaciones propuestas por la comunidad (catálogo completo)

### 18.1 Por sector

| Sector | Usuarios / Propuestas |
|---|---|
| **Espeleología** | @user-ed7gm7ol8k, @RFNR-72448 (topógrafo profesional Mid-Missouri), @NatetheAceOfficial (con CaveWhere), @ones_flow5652, @fluffandchuckbro868 |
| **Minería** | @PrebleStreetRecords / @LifesAScript (Ghost Town Living / Cerro Gordo), @wibblywobblyidiotvision (minas pizarra 100 años), @TrashfireLab (pozos mineros) |
| **Drones** | @dbillionaer, @TomS699, @PencilParasite (3 LiDARs en anillo), @thestyleofbeyond3, @Zapato56, @euemeubarco, @Tebbit123 (objetivo final: drone), @ViperGtr91 |
| **Arquitectura / Infraestructura** | @finnsjustanotherday1481 (edificio +1M sq ft), @MePeterNicholls (carreteras), @nicklaeder5628 (ingeniero civil), @Luci_4 (experiencia con Leica BLK360) |
| **VFX / Cine** | @thefosterhousevfx (fotogrametría + LiDAR para cuevas en VFX) |
| **Simulación / Gaming** | @HugoTremolo (etapas rally para RBR), @nightmisterio (calles para juego rally), @ItsTanmoyYT-k1q |
| **Emergencias** | @borkuz (bomberos), @XAirForcedotcom (rescate), @elmichellangelo (militar) |
| **Submarina** | @danielmcmath2754, @RandomAdhdQuest, @scarface7859, @Najsnwjsbdn (buceo) — nota: LiDAR 905 nm no funciona bajo agua |
| **VR/AR** | @Púca-h2q (arnés + gafas Bigscreen V2), @Fishdonotexist (cámara 360) |
| **Agricultura** | @kbrere14 (cortacésped autónomo), @MilovanLoon (geoespacial) |
| **Investigación de fauna** | @JochenRoth (murciélagos, proyecto NEXUS) |
| **Construcción renovable** | @mitchie8974 (LiDAR en drones construidos en casa) |
| **Arqueología** | @LadyChesalot (herramienta de survey arqueológico) |

---

## 19. Recursos y referencias mencionados

| Recurso | Contexto |
|---|---|
| Paper "Alpha LiDAR" | Inspiración para sistema similar (@andbondstyle) |
| Comunidad ArduPilot | Recursos de SLAM (@joshuarespecki1883) |
| GTSAM (Georgia Tech) | Librería Python para GraphSLAM (@benhenderson6966) |
| CaveWhere | Software de topografía espeleológica, en desarrollo de soporte para nube de puntos (@NatetheAceOfficial) |
| SparkFun | Módulos RTK GPS (~$200-500) (@JasonThomasHorn) |
| NSS Convention (Indiana) | Evento de la National Speleological Society (@NatetheAceOfficial) |
| Ghost Town Living (YouTube, 2M subs) | Mina Cerro Gordo, potencial colaboración |
| Polycam | App de escaneo 3D para iPhone (scans "VERY rough") |
| App 3D Scanner | Alternativa iPhone (~$70) (@nobilismaximus) |
| SimpleFOC | Librería y hardware open source para motor control (https://simplefoc.com/) |
| LADS (sistema LiDAR batimétrico) | Referente profesional mencionado por @gozznut (18 años en GIS, Adelaide, Australia) |
| Leica BLK360 / RTC360 | Escáneres profesionales ($15.000-$90.000+) usados como referencia de comparación |

---

## 20. Tips técnicos clave de la comunidad

1. **Time sync es crítico** — PPS hardware entre LiDAR, IMU y encoder reduce drásticamente el ruido (@PeaceIndustrialComplex, @andbondstyle)
2. **Inclinar el puck 15-30°** produce datos significativamente más útiles (@Spirit532)
3. **IMUs baratos tienen problema de frecuencia de muestreo** — datos de aceleración no procesados suficientemente rápido causan drift acumulativo (@OttoSphalta)
4. **Loop closure es esencial** — sin él, la deriva siempre ocurre (@franklynd, @whatilearnttoday5295)
5. **Ruido de motor** — aumentar frecuencia PWM y frecuencia del bucle de control lo elimina (@Spirit532, @empmachine, @YTuser133)
6. **Contrapeso** — añadir contrapeso al lado deficiente del LiDAR para mantener centro de masa alineado (@rassulli4532, @stocki_esy625, @Stc442)
7. **ASA/ABS delaminación** — imprimir en rango alto de temperatura, reducir ventilador, técnica de acetona+virutas como "soldadura química" (@kylek29)
8. **Draft shields** — para piezas delgadas en ASA, "enjaquetar" secciones en soportes para mantener calor (@holski77). Tyler confirmó que probará la técnica
9. **Alcohol isopropílico** despega el hot glue limpiamente (@EvilSpyBoy)
10. **Algunos hot glue son parcialmente conductivos** y pueden añadir ruido a señales (@DonWRII)
11. **RTK GPS como complemento** para áreas abiertas mejora la precisión (@JasonThomasHorn)
12. **Motor con eje hueco** como alternativa al slip ring para pasar cables (@andbondstyle)
13. **Bolas reflectivas** en un palo como puntos de referencia para facilitar stitching de escaneos (@PietjeNL)
14. **Hesai 40ch por ~$200** puede ser mejor opción que VLP-16 por $321 (@LeLehel, dice poseer ambos)
15. **Blindar motor y ESC** con lámina de acero para evitar interferencia electromagnética con sensores (@tullgutten)
