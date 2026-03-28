# Subsistema Electronico — Referencia Tecnica Exhaustiva

> Fuentes: transcripciones de 3 videos de YouTube (canal 9nl / Subterranean Systems), comentarios de la comunidad, posts y captions de Instagram (@subterraneansystems), 3 repositorios GitHub (tthom289), e inferencia de imagenes del proyecto.

---

## 1. Teensy 4.0

### 1.1 Rol en el sistema

El Teensy 4.0 (fabricado por PJRC) cumple tres funciones simultaneas:

1. **Controlador del motor BLDC** mediante la libreria SimpleFOC — genera las senales PWM trifasicas que el driver SimpleFOC convierte en corriente para las fases del motor gimbal 4015.
2. **Lectura del encoder magnetico AS5600** — obtiene la posicion angular absoluta del eje de rotacion de la plataforma en tiempo real via I2C.
3. **Transmision de datos de posicion angular al Raspberry Pi 5** — envia los datos del encoder al Pi 5 por serial UART para que ROS 2 los registre junto con los datos del LiDAR y la IMU. Estos datos son fundamentales para el de-skewing de la nube de puntos.

### 1.2 Interfaces de comunicacion

| Interfaz | Protocolo | Pines | Destino | Datos transmitidos |
|---|---|---|---|---|
| UART serial | TX1/RX1 | TX1, RX1 | Raspberry Pi 5 | Datos del encoder (angulo de la plataforma en grados) |
| I2C | SDA/SCL | SDA, SCL | Encoder AS5600 | Posicion angular absoluta (12 bits) |
| PWM trifasico | 3x PWM + Enable | IN1, IN2, IN3, EN | Driver SimpleFOC V1 | Senales de conmutacion para motor BLDC |

El topic ROS 2 publicado por el Pi 5 a partir de los datos UART del Teensy es `/rotating_platform/angle` (tipo `std_msgs/Float64`, angulo en grados).

### 1.3 Alimentacion

El Teensy 4.0 se alimenta desde un **power bank portatil USB**, separado del resto del sistema (que usa la bateria de 12V de ion de litio). Tyler explico la razon de esta separacion: el Teensy se conecta frecuentemente al laptop para cambios de firmware, tuning del encoder, ajuste de la funcion de rotacion y lectura de datos del encoder. Mantenerlo en USB independiente simplifica este flujo de trabajo de desarrollo.

### 1.4 Firmware

El firmware esta escrito en **C++ 100%** y se gestiona con **PlatformIO** (archivo `platformio.ini`). El repositorio GitHub es `vlp16-spin-controller` (github.com/tthom289/vlp16-spin-controller, licencia MIT, 18 stars). El ultimo cambio documentado fue: "Switch to open-loop velocity control with Pi5 UART output". El workspace de desarrollo es VS Code (`TeensyFOC_Init.code-workspace`).

La descripcion del repositorio indica: "Teensy 4.0 SimpleFOC-based controller for GM2804 BLDC + AS5600 encoder with 5:1 reduction, driving continuous 40 RPM rotation of Velodyne VLP-16 LiDAR." Nota: la descripcion menciona motor GM2804 y ratio 5:1, pero el sistema actual usa motor 4015 con ratio 3:1 (20T motor / 60T eje). Esto refleja la evolucion del hardware — el repositorio fue creado con el motor original.

---

## 2. Carrier Board — TeensyFOC-Carrier

### 2.1 Informacion general

| Parametro | Valor |
|---|---|
| Disenador | Tyler (GitHub: tthom289) |
| Logo | "TTHOM" con icono de engranaje grabado en la serigrafia de la PCB |
| Software de diseno | KiCad (open source) |
| Fabricante de la PCB | JLCPCB (Shenzhen JLC Electronics Co.) — caja azul con slogan "Accelerating Your Innovation" |
| Coste tipico | ~$2-8 por 5 PCBs + envio |
| Capas | 2 (relativamente simple, sin vias densas ni planos de masa complejos) |
| Color del soldermask | Verde (estandar JLCPCB) |
| Agujeros de montaje | 4 en las esquinas |
| Repositorio | github.com/tthom289/TeensyFOC-Carrier (licencia MIT, 15 stars) |

### 2.2 Razon de existencia

Tyler comenzo el proyecto con jumper wires y breadboards. A medida que anadio mas subsistemas (motor, encoder, serial al Pi, alimentacion), la complejidad del cableado se convirtio en el problema principal — puntos de fallo multiples, cables que se desconectaban por vibracion, dificultad de depuracion. La solucion fue disenar una PCB dedicada.

Cita de Tyler en Instagram: *"Designed this PCB to integrate a Teensy 4.0 with SimpleFOC motor control for my cave mapping project. Main goal? Eliminate the rats nest of jumper wires and breakout boards. Clean power delivery, compact footprint, and way fewer points of failure."*

Filosofia resumida: *"When your prototype wiring becomes the problem, you design it out."*

Descripcion honesta de Tyler sobre el resultado: *"Not the prettiest, very simple. Just breaks out all GPIO currently using."*

### 2.3 Serigrafia de pines visible

| Etiqueta | Funcion |
|---|---|
| GND | Masa / tierra |
| IN1, IN2, IN3 | Inputs del driver motor BLDC (3 senales PWM para conmutacion trifasica) |
| EN | Enable (habilitacion del driver del motor) |
| SDA, SCL | Bus I2C para el encoder magnetico AS5600 |
| VDD | Alimentacion positiva para encoder/logica |
| TX1, RX1 | Serial UART hacia el Raspberry Pi 5 (transmision de datos del encoder) |
| Vin | Alimentacion general de entrada |
| GPIO 13-17 | GPIO del Teensy no utilizados, expuestos para uso futuro. Tyler: *"all of this right here is GPIO that I'm not using. Figured just in case I need it in the future, I can solder some wires in right around here and use it if needed"* |
| SND | Etiqueta no estandar — posible error de serigrafia o senal especifica no documentada |

### 2.4 Conectores

| Tipo | Cantidad | Funcion |
|---|---|---|
| Socket DIP hembra | 1 (dos filas paralelas de headers negros) | Alojamiento del Teensy 4.0 en el centro de la placa. Tyler lo diseno asi para poder extraer o reemplazar el Teensy facilmente |
| JST-XHP blancos | 3+ en los bordes | Alimentacion/Pi, serial TX/RX al Pi 5, encoder/motor |
| Header SimpleFOC | 1 | Interfaz con el driver SimpleFOC V1 (se enchufa directamente) |
| Interfaz I2C | 1 | Conexion al encoder AS5600 |
| Soporte passthrough slip ring | Presente | Facilita el paso de cables del slip ring |

Tyler en video: *"CNC here. I made this like this, so if I ever need to pull it out or replace it, it's a little bit easier."* (refiriendose al socket DIP del Teensy).

### 2.5 Componentes SMD

En la cara inferior de la placa se observan componentes de montaje superficial. Probablemente son **resistencias de pull-up para el bus I2C** del AS5600 (tipicamente 4.7k ohm a VDD) y/o **capacitores de desacoplo** (tipicamente 100nF ceramico cerca de los pines de alimentacion).

### 2.6 Descripcion del repositorio GitHub

El repo `TeensyFOC-Carrier` contiene:

| Archivo | Descripcion |
|---|---|
| `TeensyFOC.kicad_sch` | Esquematico KiCad |
| `TeensyFOC.kicad_pcb` | Layout de la PCB |
| `TeensyFOC.kicad_pro` | Archivo de proyecto KiCad |
| `TeensyFOC.kicad_prl` | Preferencias locales KiCad |
| `TeensyFOC.step` | Modelo 3D del ensamblaje |
| `TeensyFOC.zip` | Gerbers para fabricacion |
| `TeensyFOC.csv` | Netlist CSV |
| `TeensyFOC_bom.csv` | Bill of Materials |
| `TeensyFOC_designators.csv` | Designadores de componentes |
| `TeensyFOC_positions.csv` | Posiciones pick-and-place |
| `TeensyMotorCTRL.zip` | Zip adicional (motor control) |
| `fabrication-toolkit-options.json` | Opciones del toolkit de fabricacion |

BOM de conectores documentado:
- Single Row 2.54mm Headers (Amazon)
- JST-XHP Connector Kit (Amazon)

### 2.7 Caption de Instagram

*"Built a custom carrier board for my rotating LiDAR Scanner. Designed this PCB to integrate a Teensy 4.0 with SimpleFOC motor control for my cave mapping project. Main goal? Eliminate the rats nest of jumper wires and breakout boards. Clean power delivery, compact footprint, and way fewer points of failure. When your prototype wiring becomes the problem, you design it out. Full build series coming soon. #PCBDesign #robotics #ai #engineering"*

*"Unboxing the board for my LiDAR Scanner Motor Controller. More info coming soon! #techdiy #engineering #robotics #ros2"*

---

## 3. SimpleFOC V1 — Driver del motor

### 3.1 Especificaciones

| Parametro | Valor |
|---|---|
| Modelo | SimpleFOC V1 (placa del proyecto open source SimpleFOC — simplefoc.com) |
| Tipo | Driver de potencia para motor BLDC — convierte las senales PWM de 3 fases del Teensy en corriente trifasica que alimenta las bobinas del motor |
| Tamano | ~25-30 mm x 20-25 mm (muy compacto) |
| Fijacion en el sistema | Cinta adhesiva de doble cara pegada al interior de la caja |
| Alimentacion | 12V conmutada (a traves del switch manual) |
| Salida | Cables a las 3 fases del motor (rojo, blanco, verde, amarillo) |

### 3.2 Componentes visibles en la placa

- **IC central** (encapsulado QFN): probablemente half-bridge/gate driver que genera las senales de alta corriente para los MOSFETs
- **Conector JST miniatura**: para conexion con la carrier board o alimentacion
- **MOSFETs de potencia**: conmutan la corriente de 12V hacia las fases del motor
- **LED rojo**: indicador de estado/alimentacion
- **Componentes pasivos SMD**: resistencias, capacitores de desacoplo, posiblemente inductores

### 3.3 Conexion en el sistema

El driver SimpleFOC V1 se enchufa directamente en el header de la carrier board TeensyFOC. Recibe las senales PWM (IN1, IN2, IN3) y la senal Enable (EN) del Teensy 4.0, y entrega corriente trifasica al motor 4015.

### 3.4 Ruido del motor y PWM

Varios miembros de la comunidad comentaron sobre el ruido audible del motor:

- **@empmachine**: El ruido esta relacionado con la frecuencia de operacion del PWM. Sospecha que hay alguna constante "antigua" en el codigo para la frecuencia PWM.
- **@YTuser133**: Podria ser la velocidad de conmutacion de los transistores en la placa controladora. Si operan en rango de kHz, siempre produciran ese sonido. Una placa mas cara con mayor frecuencia de conmutacion eliminaria el ruido porque estaria fuera del rango audible.
- **@Spirit532**: Eliminar el ruido es simple: aumentar las frecuencias de conmutacion y del bucle de control. Tambien se reduciria al aumentar la relacion de engranajes y hacer girar el motor mas rapido.
- **@____________________________.x**: Las secciones de potencia en GPUs operan en rango de MHz y aun pueden producir "coil whine". Hay mas factores involucrados que solo la frecuencia, aunque aumentarla ciertamente ayuda.

---

## 4. Encoder magnetico — AS5600

### 4.1 Especificaciones

| Parametro | Valor |
|---|---|
| Modelo | AS5600 |
| Tipo | Encoder magnetico rotatorio absoluto |
| Resolucion | 12 bits (4.096 posiciones/revolucion = ~0,088 grados por paso) |
| Protocolo de comunicacion | I2C (SDA, SCL) |
| Frecuencia de reporte observada | ~200-320 Hz (varia entre configuraciones; hasta ~451 Hz en algunas sesiones segun la app SLAM Data Recorder) |
| Principio de funcionamiento | Mide la posicion angular del eje mediante un iman montado en el eje de rotacion. El sensor detecta el campo magnetico y calcula el angulo absoluto |
| Montaje | Breakout board pequena de color oscuro/negro con header de pines macho |

### 4.2 Cableado

4 cables con conector JST blanco:

| Color del cable | Senal |
|---|---|
| Rojo | VCC (alimentacion) |
| Marron/oscuro | GND (masa) |
| Verde | SDA (datos I2C) |
| Amarillo | SCL (reloj I2C) |

### 4.3 Rendimiento en el sistema

A la velocidad de rotacion del sistema (~20 RPM), con una resolucion de 0,088 grados por paso, el encoder proporciona mas que suficiente resolucion angular para el de-skewing de la nube de puntos. El dato del encoder se transmite al Pi 5 via UART serial desde el Teensy y se publica como topic ROS 2 `/rotating_platform/angle`.

### 4.4 Importancia para la sincronizacion temporal

El usuario **@andbondstyle** especificamente senalo que el encoder tambien necesita sincronizacion temporal con el LiDAR y la IMU para un registro preciso de la nube de puntos. El parametro `ENCODER_TIME_OFFSET_MS` en `offline_deskew.py` existe exactamente para compensar cualquier desfase temporal entre el encoder y el reloj del LiDAR.

---

## 5. Slip Ring — LPC-18

### 5.1 Especificaciones completas

| Parametro | Valor |
|---|---|
| Fabricante | slipring.cn (fabricante chino) |
| Modelo | LPC-18 (capsule slip ring) |
| Circuitos | 18 independientes |
| Circuitos usados por Tyler | Solo 3-4: alimentacion (power), masa (ground), Ethernet data+, Ethernet data-. El resto cortados y apartados |
| Discrepancia | Tyler menciona "12 circuits" en el video de Instagram, pero el modelo mostrado es LPC-18 (18 circuitos). Posible margen extra, o el diagrama es referencia generica |
| Corriente por circuito | Tipicamente 2A para senales, algunos canales 5-10A para potencia |
| Velocidad maxima de rotacion | Tipicamente 250-300 RPM (el sistema opera a ~20 RPM, muy dentro del margen) |
| Vida util tipica | >10 millones de revoluciones (modelos con escobillas de metal precioso) |
| Precio referencia | ~$30-50 en eBay |
| String de busqueda en eBay (confirmado por Tyler) | "Gigabit Ethernet Slip Ring 1000Base-T 1 Cable Power Signal Flange RJ45 Joint" |

### 5.2 Anatomia del slip ring (de arriba a abajo en el despiece)

1. **Carcasa superior del rotor** (azul oscuro): Parte que rota con el LiDAR. Cilindro de plastico/resina. Los cables del rotor salen por arriba (naranja, rojo, verde, amarillo, azul) y van al VLP-16 (Ethernet, alimentacion, senales PPS/GPS).

2. **Anillos de contacto y escobillas** (zona central dorada): Bandas horizontales doradas (anillos conductores). Las escobillas (contactos de carbon o metal precioso) presionan contra estos anillos durante la rotacion, manteniendo la conexion electrica continua. Se observa un muelle/resorte helicoidal dorado.

3. **Rodamiento central miniatura** (plateado): Permite la rotacion suave entre el rotor y el estator.

4. **Base/flange del estator** (azul oscuro): Brida de montaje con forma de sombrero — se fija al chasis con tornillos. Los cables del estator salen por abajo y van a la Raspberry Pi 5 y la fuente de alimentacion.

### 5.3 Tipo de slip ring

El LPC-18 mostrado en el diagrama de Instagram es un slip ring de **cables individuales** (sin conector RJ45 integrado). Sin embargo, Tyler podria estar usando uno con conector RJ45 integrado que simplifica la conexion Ethernet. El diagrama parece ser una referencia generica para explicar el concepto al publico.

### 5.4 Criticidad para Ethernet

El VLP-16 transmite toda su nube de puntos via **Gigabit Ethernet (1000Base-T)**. La integridad de la senal Ethernet a traves de contactos rotativos deslizantes es **no trivial** — requiere:

- Resistencia baja y consistente entre los contactos
- Minimo ruido electrico
- Capacidad para las frecuencias de senal de Gigabit Ethernet

Los slip rings genericos de senal **no son suficientes**; se necesita uno especificamente disenado para Gigabit Ethernet. Un slip ring de mala calidad causaria **perdida de paquetes = frames de nube de puntos perdidos o corruptos**.

### 5.5 Funcionamiento indefinido

Tyler confirmo que el sistema puede funcionar indefinidamente: *"it can run indefinitely, or until the slip ring wears out, but something else would probably fail first."* Esto lo confirmo en respuesta a multiples usuarios que preguntaron como evitar que los cables se retuerzan (@thomasharper9087, @CrazyCanuckCH, @12mkamran).

### 5.6 Cita de Tyler en Instagram

*"You've probably seen everyone keeps asking how the wires don't get tangled up. The secret is this little guy, a slip ring. The wires run down through this tube to the slip ring at the base. The top half spins with the LiDAR sensor, the bottom half stays put, so nothing ever gets twisted. Inside, you've got these rotating metal rings with little brushes that just ride along and stay in contact. Each ring gets its own circuit, so power and data keep flowing while the whole thing spins. It's the same thing you find in wind turbines, radar dishes, even CT scanners. Basically, anything that needs to spin forever without strangling itself."*

### 5.7 Alternativa sin slip ring

Tyler confirmo a @tedzbug07 que **no es necesaria rotacion 360 grados continua** — un pivoteo de ida y vuelta tambien funciona y eliminaria la necesidad del slip ring. El usuario @eldoncurrey2124 sugirio eliminar el cable central usando WiFi para el transceptor y alimentar solo via slip ring.

### 5.8 Comentarios de la comunidad sobre el slip ring

| Usuario | Comentario |
|---|---|
| @2ndgameX | Pregunto que slip ring se usa y si uno barato es suficiente para 100Base-T |
| @MicroRobo | Pregunto marca/modelo del slip ring. Tyler respondio con el string de busqueda de eBay |
| @DonWRII | Advertencia: algunos tipos de hot glue son parcialmente conductivos y pueden anadir ruido a senales. Lo aprendio en reparaciones USB |
| @EvilSpyBoy | El alcohol isopropilico remueve el hot glue limpiamente de las superficies |
| @Nikkuuu69 | Confirmo el tip del isopropilico: convierte el hot glue en herramienta de fijacion temporal |
| @KIWIvolt | Recomendo no retorcer cables; usar slip ring |
| @eldoncurrey2124 | Sugirio eliminar el cable usando WiFi para transceptor, alimentar via slip ring |
| @Andy-js5jy | Sugirio hacer una PCB madre para reducir el desorden de cables |

---

## 6. Conector GX12

Conector circular metalico con rosca que conecta el cable del VLP-16. Tyler lo usa porque el sensor comprado de segunda mano ya lo traia instalado y no queria desmontar ni recablear el sensor. El conector pasa a traves del interior del **eje hueco de 1" (25.4 mm)** — esta fue la razon de aumentar el diametro del eje de 3/4" a 1", precisamente para permitir el paso del GX12.

---

## 7. Senales que pasan a traves del slip ring

| Senal | Tipo | Descripcion |
|---|---|---|
| Ethernet | Gigabit 1000Base-T | Datos de la nube de puntos del VLP-16 hacia el Pi 5 |
| Power | 12V DC | Alimentacion del VLP-16 |
| Ground | GND | Masa |
| GPS/PPS (x2) | Senal digital | No se usa GPS real. Se utiliza PPS timing con RTC para sincronizacion precisa entre la IMU, los puntos del Velodyne y los datos del encoder |

---

## 8. Cableado general

### 8.1 Estado del cableado

El interior del sistema muestra un cableado **denso pero funcional**. Cables de multiples colores: amarillo, naranja, rojo, negro, verde, azul, blanco. Un **cable plano blanco** (flat ribbon cable o flat flex cable) pasa por la zona superior — posiblemente cable de datos de la pantalla (HMI) o cable GPIO. Un conector **RJ45/Ethernet metalico** rectangular es visible donde el cable Ethernet del VLP-16 se conecta despues de pasar por el slip ring.

Tyler lo describe como *"absolute mess of wires"* y *"really needs to be cleaned up."*

### 8.2 Practicas de cableado observadas

| Practica | Descripcion |
|---|---|
| **Cables trenzados** (verde-amarillo, twisted pair) | Tecnica para reducir interferencia electromagnetica (EMI). Observados probablemente en senales del encoder AS5600 (I2C: SDA/SCL) o senales del motor |
| **Pegamento caliente en juntas de soldadura** | Los cables estan soldados por detras del conector y rellenos con hot glue para evitar que las juntas se rompan por vibracion. Tyler: *"No es lo mas bonito, pero funciona"* |
| **Malla protectora (wire loom/mesh)** | Sobre los cables del slip ring para mitigar el roce por friccion |
| **Cable Ethernet apantallado** | Cables azules gruesos tipo CAT6 visibles en versiones avanzadas del sistema |

### 8.3 Comentarios de la comunidad sobre cableado

| Usuario | Comentario |
|---|---|
| @ImaginationToForm | Preocupacion por el cable rojo que pasa bajo el eje blanco giratorio — potencial abrasion |
| @arias8185 | Senalo que existen cables disenados especificamente para rotacion (slip rings/rotary connectors) |
| @Andy-js5jy | Sugirio una PCB madre para reducir el desorden, pero recomendo pedir ayuda de expertos para reducir riesgo |

---

## 9. Arquitectura de alimentacion

### 9.1 Fuentes de energia

| Componente | Fuente | Notas |
|---|---|---|
| VLP-16, Pi 5, IMU, HMI (pantalla) | Bateria de ion de litio 12V (celdas azules LiPo/Li-ion visibles en el interior de la caja) | Fuente principal del sistema |
| Teensy 4.0 / motor controller | Power bank portatil USB | Separado para facilitar la conexion frecuente al laptop durante desarrollo |
| Sistema en interior/banco | Fuente de alimentacion DC | Mas comodo que la bateria para testing prolongado |

### 9.2 Switch manual

Tyler anadio un **interruptor manual** (naranja, visible en fotos cenitales recientes) porque durante el testing constante se deja todo el sistema alimentado, pero el giro del motor hace mucho ruido. El switch permite **cortar solo la rotacion** (motor) sin apagar el resto del sistema (Pi 5, VLP-16, IMU). El switch actua sobre la linea de 12V que alimenta el driver SimpleFOC.

### 9.3 Alimentacion del VLP-16 a traves del slip ring

El VLP-16 recibe sus 12V DC a traves del slip ring. La alimentacion pasa por dos de los circuitos del slip ring (power y ground), junto con las senales de Ethernet y PPS.

---

## 10. Cadena de datos completa del subsistema electronico

```
AS5600 (encoder) --[I2C: SDA/SCL]--> Teensy 4.0 --[UART: TX1/RX1]--> Raspberry Pi 5
                                          |
                                    [PWM: IN1/IN2/IN3 + EN]
                                          |
                                          v
                                    SimpleFOC V1 (driver)
                                          |
                                    [3-phase current]
                                          |
                                          v
                                    Motor BLDC 4015
                                          |
                                    [Belt drive 3:1]
                                          |
                                          v
                                    Eje de rotacion (1")
                                          |
                                    [Inside hollow shaft]
                                          |
                                          v
                                    Slip ring LPC-18
                                          |
                              [Ethernet + 12V + GND + PPS x2]
                                          |
                                          v
                                    VLP-16 "Puck"
```

El Pi 5 recibe datos de tres fuentes:
1. **VLP-16** via Ethernet (a traves del slip ring): nube de puntos (~300.000 pts/s)
2. **IMU N100** via USB serial: aceleracion y rotacion (~100-400 Hz)
3. **Teensy 4.0** via UART serial: angulo de la plataforma (~200-451 Hz)
