# Video 2 — El LiDAR: cómo funciona y por qué el VLP-16

**Link:** https://www.youtube.com/watch?v=R4zUKJqoBcI

## [0:00] Introducción

Este pequeño sensor dispara 300.000 puntos láser por segundo, puede ver en oscuridad completa, mide con precisión centimétrica y va a ser los ojos de mi plataforma de mapeo de cuevas. Llevo unos 6 meses trabajando en este proyecto. El objetivo inicial era simplemente aprender más sobre sistemas autónomos, SLAM y robótica. Hace unas semanas conseguí sacar los primeros datos de escaneo del sistema y quedaron muy bien. Publiqué algunos vídeos y resultó que a la gente le encantó. Así que aquí estamos con el segundo vídeo de la serie, profundizando en la parte más importante del sistema: el Velodyne VLP-16.

## [0:35] Fundamentos del LiDAR

LiDAR significa Light Detection and Ranging. En esencia es un concepto simple: disparas un pulso de luz láser, golpea algo, rebota, mides cuánto tardó, multiplicas por la velocidad de la luz y divides entre dos (porque la luz viajó ida y vuelta). Ya tienes la distancia. Eso es el LiDAR de tiempo de vuelo. Lo que lo hace poderoso es cuando disparas miles o incluso millones de estos pulsos en diferentes direcciones. Cada uno te da una medición de distancia y juntos construyen una imagen 3D del mundo. Estos sensores son clase 1, seguros para los ojos bajo condiciones normales de operación. Publiqué recientemente un vídeo sobre seguridad láser, enlazado en la descripción.

## [1:14] Tipos de LiDAR

**LiDAR aéreo.** Vuela en aviones, drones, incluso satélites. El ICESat-2 de la NASA usa LiDAR para medir cambios en la elevación de capas de hielo desde órbita. Aviones de topografía pueden mapear cientos de kilómetros cuadrados en un solo vuelo. Los drones han traído esta capacidad a escalas más pequeñas: obras, granjas, operaciones mineras. La gran ventaja es la cobertura y la penetración de dosel: los pulsos láser se cuelan entre los huecos de los árboles y la vegetación y mapean el terreno debajo.

**LiDAR terrestre.** Funciona desde el suelo. Los sistemas estáticos con trípode capturan escaneos increíblemente detallados de edificios, infraestructura, escenas del crimen, con precisión milimétrica de grado topográfico. Los sistemas móviles terrestres van montados en vehículos: así es como Google Maps obtiene sus datos 3D a nivel de calle, conduciendo por la carretera y capturando todo a ambos lados. Y luego están los sistemas portátiles de mano, que puedes llevar a espacios donde los vehículos no caben. Esa es la categoría para la que estamos construyendo, al menos por ahora, hasta que lo pongamos en un dron.

**LiDAR batimétrico.** Usa láseres de longitud de onda verde que penetran el agua. Útil para mapear lechos de ríos, zonas costeras y fondos oceánicos poco profundos. Herramienta diferente, mismo principio fundamental.

## [2:29] Aplicaciones del LiDAR

**Topografía y levantamiento.** La aplicación clásica. Crear modelos digitales de elevación, planificar obras, modelar cómo fluye el agua a través de un paisaje para predicción de inundaciones. Cada vez que quieres saber cómo es la superficie de la tierra con precisión, probablemente hay LiDAR involucrado.

**Arqueología.** Cuando los arqueólogos descubrieron ciudades mayas masivas ocultas bajo la selva guatemalteca, fue con LiDAR aéreo. Miles de estructuras, caminos, terrazas agrícolas, invisibles desde el suelo e invisibles en fotografía aérea convencional, pero los pulsos láser encontraron el terreno a través del dosel y revelaron una civilización entera.

**Medición de volúmenes.** Enorme en minería y construcción. Tienes un acopio de grava o mineral, necesitas saber cuántos metros cúbicos hay: vuelas un dron, procesas la nube de puntos, calculas el volumen. Lo que antes llevaba días a un equipo de topografía ahora se hace en horas. El mismo principio para rastrear capacidad de vertederos o calcular cantidades de movimiento de tierras en una obra.

**Documentación de infraestructura.** Cuando Notre Dame ardió en 2019, investigadores ya tenían escaneos LiDAR detallados de toda la estructura de un proyecto años anterior. Esos datos se están usando ahora para guiar la reconstrucción. Sabían exactamente cómo era cada piedra y cada viga antes del incendio. El mismo principio se aplica a puentes, túneles e instalaciones industriales: capturas la condición as-built en 3D y tienes un registro digital permanente.

**Vehículos autónomos.** Probablemente lo que la mayoría piensa al oír LiDAR. Ese sensor giratorio encima de un coche construye un mapa 3D en tiempo real de todo su entorno: peatones, otros vehículos, marcas de carril, obstáculos. El sistema de percepción del coche usa esos datos para entender el mundo y tomar decisiones de conducción.

**Ciencia forestal y ambiental.** Medir alturas de árboles en bosques enteros, estimar biomasa y secuestro de carbono, evaluar riesgo de incendios forestales midiendo cargas de combustible (cuánta vegetación seca hay disponible para arder). El LiDAR aéreo puede ver a través del dosel para medir tanto los árboles como la superficie del suelo, lo que te da el panorama completo.

## [4:27] Mapeo de cuevas

Y esto nos lleva a por qué estoy haciendo esto. Los levantamientos tradicionales de cuevas se hacen con cintas métricas, brújulas e inclinómetros. Funciona, los espeleólogos han mapeado miles de kilómetros de pasajes así durante el último siglo, pero es lento, intensivo en mano de obra y la resolución está limitada a cuántas estaciones estés dispuesto a colocar. El LiDAR cambia la ecuación por completo: no importa que no haya luz ambiente, traemos nuestros propios fotones. Geometría 3D compleja con voladizos, pozos verticales, formaciones delicadas, capturada perfectamente sin tocar nada. Entorno sin GPS donde no puedes obtener señal de satélite, no necesitamos GPS cuando tenemos algoritmos SLAM construyendo el mapa mientras nos movemos. Esa es la visión de este proyecto: desarrollar un sistema que pueda llevar a través de una cueva y salir con modelos 3D extremadamente precisos de todo el sistema de cuevas que recorra. Los datos pueden usarse para investigación científica, conservación, o simplemente compartir estos modelos 3D con cualquiera que esté interesado en ver cómo es una cueva.

## [5:23] El VLP-16 en detalle

Este es el VLP-16, conocido como el "Puck". Ha sido el caballo de batalla de la industria de vehículos autónomos y robótica durante años, y hay buenas razones para ello. Todo el conjunto óptico gira dentro de la carcasa. Puedes configurar la tasa de rotación entre 5 y 20 revoluciones por segundo. Yo lo corro a unos 10 Hz, que da un buen equilibrio entre densidad de puntos y tasa de actualización. A esa velocidad genera aproximadamente 300.000 mediciones de distancia por segundo.

Cada punto viene con más que solo posición. También obtenemos datos de intensidad, que básicamente miden lo reflectante que es la superficie. Los materiales retroreflectivos como señales de tráfico o cinta de seguridad se iluminan muy brillantes. Las superficies más oscuras o anguladas devuelven señales más débiles. El canal de intensidad se puede usar para distinguir diferentes tipos de superficie en post-procesamiento.

Ahora, la limitación que necesitamos discutir: 30° de cobertura vertical. Todo lo que esté a más de 15° por encima del sensor es punto ciego. Todo lo que esté a más de 15° por debajo, punto ciego. Si estás de pie en un pasaje de cueva con el sensor a la altura del pecho, no estás viendo el techo si está directamente encima, y no estás viendo el suelo a tus pies. Para aplicaciones automotrices esto es perfectamente válido: te importa lo que hay delante en la carretera, no lo que hay encima o debajo del coche. Pero para mapeo volumétrico de entornos 3D es una restricción real. Esta es exactamente la razón por la que estoy construyendo la plataforma rotatoria.

## [6:42] Por qué el VLP-16 específicamente

Hay montones de sensores LiDAR en el mercado. ¿Por qué elegí el VLP-16?

**Disponibilidad y coste.** El VLP-16 está en producción desde 2014. La industria de vehículos autónomos pasó por miles de estos sensores durante desarrollo y pruebas. Ahora muchas de esas empresas han pasado a sensores más nuevos, actualizado sus flotas o directamente cerrado. Eso significa que VLP-16 usados aparecen constantemente en el mercado secundario. Pagué 321 dólares por el mío.

**Ecosistema y soporte.** El VLP-16 tiene drivers maduros y probados en batalla para ROS y ROS 2. El paquete del driver de Velodyne lleva años disponible. Cuando estás intentando depurar tus algoritmos SLAM y tu mapa está derivando o los cierres de bucle están fallando, lo último que quieres es también estar cuestionando si el driver del sensor te está dando buenos datos. Además hay una tonelada de conocimiento comunitario ahí fuera: cualquier problema que te encuentres, probablemente alguien ya lo tuvo y escribió sobre ello.

**Durabilidad.** Está diseñado para ir montado encima de vehículos circulando por San Francisco. Lluvia, niebla, tormentas, vibración. La calidad de construcción es bastante impresionante. Calificación IP67, temperatura de operación de unos -10 a 60°C. Para trabajo de campo en cuevas con humedad, grandes cambios de temperatura y golpes ocasionales contra paredes de roca, esa durabilidad importa.

## [8:00] Alternativas al VLP-16

Siendo honesto, si empezara de cero o tuviera presupuesto ilimitado, probablemente elegiría un sensor diferente. Pero el VLP-16 va perfectamente bien para este proyecto.

Los sensores **Livox** como el Mid-360 ofrecen increíble densidad de puntos por dólar, con un patrón de escaneo no repetitivo que se rellena con el tiempo y funciona muy bien para SLAM. El inconveniente: el patrón de escaneo puede ser complicado de manejar y son bastante más caros.

Los sensores **Ouster** como el OS1 dan más canales verticales (hasta 128) y mayor resolución, además generan imágenes tipo cámara junto con la nube de puntos, pero son mucho más caros y consumen más energía.

Las opciones económicas como la serie **RPLiDAR** se consiguen por unos pocos cientos de dólares y son geniales para mapeo 2D simple, pero limitadas para cobertura 3D completa en entornos complejos.

Para este proyecto específico, el VLP-16 da en el clavo en el equilibrio entre capacidad, coste, ecosistema y calidad de construcción.

## [8:53] Cierre

El sensor es solo un sensor. Necesita un cerebro para hacer algo útil con todos esos datos. En el próximo vídeo hablaremos de la plataforma de cómputo: cómo estoy usando una Raspberry Pi 5 con ROS 2 para capturar, procesar y eventualmente ejecutar SLAM sobre estas nubes de puntos en tiempo real. Veo en todos los comentarios que todo el mundo quiere una guía how-to, documentación, modelos CAD y código. Estoy trabajando en ello, voy a documentarlo todo y compartirlo con todos. Simplemente lleva mucho tiempo.
