# Video 1 — Visión general del sistema SLAM LiDAR

**Link:** https://www.youtube.com/watch?v=sGpfHk-vEkg

## [0:00] El problema y la visión

Este sistema dispara 300.000 pulsos láser por segundo. Funciona en oscuridad total. No necesita GPS. Existen sistemas comerciales capaces de hacer esto, pero cuestan unos 40.000 dólares. Yo construí el mío por menos de 1.000. El GPS no funciona bajo tierra en absoluto: cuevas, minas, parkings subterráneos... tu teléfono es básicamente inútil. E incluso en exteriores, el GPS solo te dice por dónde pasaste, es solo una línea en el mapa. Lo que yo busco es muy diferente: capturar el entorno real. Modelos 3D completos, cada superficie, cada dimensión, el espacio físico completo.

Hay empresas que ya han resuelto esto. Fabrican sistemas de escaneo portátiles que usan SLAM (Simultaneous Localization and Mapping), que rastrea su propia posición mientras escanea todo a su alrededor. El problema es que cuestan unos 40.000 dólares y son completamente propietarios. No puedes ver cómo funciona nada. Y no es que pueda comprar uno y hacer ingeniería inversa, no tengo ese dinero. Ninguna de estas empresas comparte su salsa secreta. Así que decidí ver hasta dónde podía llegar construyendo uno yo mismo con herramientas open source y un presupuesto mucho más reducido.

## [0:54] Los componentes del sistema

**El LiDAR — Velodyne VLP-16.** Es un LiDAR de 16 canales. Gira internamente a unas 600 RPM, dispara láseres en todas las direcciones y genera unos 300.000 puntos por segundo. Encontré este de segunda mano en eBay por unos 400 dólares. Nuevo costaba unos 8.000.

**La IMU — Wheeltech N100.** Mide aceleración y rotación unas 400 veces por segundo. Así es como el sistema sabe cuál es arriba y cómo se está moviendo entre cada escaneo del LiDAR. La calidad de la IMU importa mucho más de lo que pensaba inicialmente; haré un vídeo entero dedicado a por qué.

**El cerebro — Raspberry Pi 5 con ROS 2.** Se comunica con el LiDAR y la IMU, y graba datos para el algoritmo SLAM. Registra todo. Es básicamente el sistema nervioso central.

**La plataforma rotatoria.** Un motor gimbal y un microcontrolador Teensy que hacen girar todo el LiDAR en un segundo eje. Así es como se consigue cobertura completa de 360° en lugar de solo un corte horizontal.

En total, estamos hablando de poco menos de 1.000 dólares en hardware, pero demasiadas horas invertidas.

## [2:07] Por qué rotar el sensor

Los 16 láseres del VLP-16 están apilados verticalmente, pero solo cubren unos 30°. Cuando está quieto, obtienes un anillo de datos alrededor del sensor. Genial para un coche autónomo que siempre se mueve hacia adelante, pero no tanto si estás de pie en una cueva intentando mapear el techo. Si rotas toda la unidad en un segundo eje, los 16 haces barren todos los ángulos: techo, suelo, paredes, todo. Cobertura volumétrica completa desde una sola posición. Esa es toda la idea. Simple en concepto, pero conseguir que funcione realmente me llevó bastante tiempo.

## [2:42] Próximos vídeos

En los próximos vídeos profundizaré en cada pieza: cómo funciona realmente el LiDAR, qué ve y qué no, y por qué el VLP-16 específicamente. Cubriré la Raspberry Pi y la configuración de ROS 2, cómo se comunican todas las piezas entre sí. La plataforma rotatoria tendrá su propio vídeo: selección del motor, Simple FOC, el Teensy, y toda la parte mecánica. Y después SLAM: cómo FAST-LIO toma todos estos datos crudos del sensor y los convierte en un mapa mientras rastrea la posición en tiempo real.
