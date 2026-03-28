# Subsistema LiDAR -- Velodyne VLP-16 "Puck"

Documento de referencia consolidado sobre el sensor LiDAR del proyecto Subterranean Systems. Toda la informacion ha sido extraida exhaustivamente de las fuentes primarias: transcripciones de video, comentarios de YouTube, publicaciones de Instagram, READMEs de GitHub, e inferencias de imagenes.

---

## Fundamentos del LiDAR

LiDAR significa **Light Detection and Ranging**. Concepto: se dispara un pulso de luz laser, golpea algo, rebota, se mide el tiempo de ida y vuelta, se multiplica por la velocidad de la luz y se divide entre dos. Eso es LiDAR de tiempo de vuelo (ToF). Lo que lo hace poderoso es disparar miles o millones de pulsos en diferentes direcciones, construyendo una imagen 3D del mundo.

### Tipos de LiDAR
- **Aereo**: aviones, drones, satelites. El ICESat-2 de la NASA usa LiDAR desde orbita para medir cambios en elevacion de capas de hielo. Drones han traido esta capacidad a escalas menores (obras, granjas, mineria). Ventaja: cobertura y penetracion de dosel vegetal.
- **Terrestre**: desde el suelo. Sistemas estaticos con tripode (precision milimetrica). Sistemas moviles en vehiculos (asi Google Maps obtiene datos 3D a nivel de calle). Sistemas portatiles de mano — la categoria del proyecto de Tyler.
- **Batimetrico**: laseres de longitud de onda verde que penetran el agua. Mapeo de lechos de rios, zonas costeras, fondos oceanicos poco profundos. @gozznut trabajo 18 anos en GIS/Survey con el sistema LADS de LiDAR batimetrico en Adelaide, Australia.

### Aplicaciones historicas destacadas
- **Arqueologia**: Ciudades mayas masivas descubiertas bajo la selva guatemalteca con LiDAR aereo. Miles de estructuras, caminos y terrazas agricolas invisibles desde el suelo.
- **Documentacion de infraestructura**: Cuando Notre Dame ardio en 2019, investigadores ya tenian escaneos LiDAR detallados de toda la estructura. Esos datos guian ahora la reconstruccion.
- **Ciencia forestal**: Medir alturas de arboles, estimar biomasa y secuestro de carbono, evaluar riesgo de incendios forestales midiendo cargas de combustible.
- **Vehiculos autonomos**: Sensor giratorio encima del coche construye mapa 3D en tiempo real.
- **Topografia**: Modelos digitales de elevacion, planificacion de obras, prediccion de inundaciones.
- **Medicion de volumenes**: Mineria y construccion — calcular metros cubicos de acopios con drones y nubes de puntos.

---

## Especificaciones tecnicas

| Parametro | Valor |
|---|---|
| Fabricante | Velodyne (ahora Ouster tras la adquisicion) |
| Modelo | VLP-16, apodo "Puck" |
| Tipo | LiDAR de tiempo de vuelo (Time-of-Flight), 16 canales |
| Longitud de onda | 905 nm (infrarrojo cercano) |
| Seguridad laser | Clase 1 (seguro para los ojos en condiciones normales de operacion) |
| Puntos por segundo | ~300.000 |
| Canales | 16 haces laser apilados verticalmente |
| Campo de vision vertical | 40 grados (+-15 grados desde el centro optico horizontal). Tyler tambien menciona 30 grados en algunos contextos |
| Campo de vision horizontal | 360 grados (rotacion interna continua) |
| Velocidad de rotacion interna | ~600 RPM (configurable entre 5 y 20 rev/seg; Tyler la usa a ~10 Hz) |
| Precision | +-3 cm (segun datasheet, mencionado por @sharknight11) |
| Proteccion | IP67 |
| Temperatura de operacion | -10 a 60 grados Celsius |
| Datos de salida | Puntos 3D procesados + datos de intensidad (reflectividad de superficie) via Gigabit Ethernet |
| Dimensiones | ~103 mm de diametro x 72 mm de alto |
| Peso | ~830 g |
| Precio original de venta | ~$8.000 USD |
| Precio pagado por Tyler | $321 USD (segunda mano, eBay) |
| Conector | GX12 (conector circular metalico con rosca, ya venia instalado en el sensor comprado) |
| En produccion desde | 2014 |

### Seguridad laser -- explicacion detallada

El VLP-16 es Clase 1, seguro para los ojos bajo condiciones normales de operacion. Los pulsos son de aproximadamente 5 ns de duracion, distribuidos entre 16 haces que rotan a ~600 RPM. La energia de cada pulso individual esta muy por debajo del umbral de dano retinal. Los LiDARs de 1550 nm son aun mas seguros porque esa longitud de onda es absorbida por la cornea antes de llegar a la retina. Preguntas de la comunidad sobre seguridad laser:
- @darth_dan8886: pregunto como los laseres no danan ojos y camaras, ha oido que los LiDAR de coches tienen ese problema.
- @TheEmptyHoliness: pregunto por que estos laseres infrarrojos no son peligrosos para los ojos y que los diferencia de un laser infrarrojo tipico.
- Advertencia de @AdrianInGaming: los pulsos laser eventualmente podrian danar sensores CMOS de camaras con aperturas grandes.
- @dirtynachobuffet: sugirio que seria interesante ver el sistema con gafas de vision nocturna (NVGs), ya que al trabajar en IR cercano (905nm), los NVGs podrian captar los pulsos.

Tyler publico un video separado sobre seguridad laser (enlazado en la descripcion del video 2).

---

## Anatomia interna (Tyler abrio el sensor)

Tyler retiro la tapa del VLP-16 y fotografio su interior. La electronica interna esta intacta y sin modificaciones: no hay cables soldados a la PCB interna ni componentes retirados. Tyler usa el VLP-16 como unidad cerrada que recibe alimentacion y entrega datos procesados por Ethernet.

### PCB superior (placa circular verde)

- Al menos dos ICs grandes (encapsulado QFP o BGA): probablemente FPGA o procesador de senal que gestiona timing de pulsos laser, recepcion de retornos y calculo de distancias ToF
- Motor de rotacion central: orificio central con anillo naranja/cobre (estator) y eje de rotor. Este motor gira a ~600 RPM haciendo que los 16 pares emisor-receptor barran 360 grados horizontales
- Componentes pasivos distribuidos: capacitores amarillos (ceramicos/inductores), transistores/MOSFETs negros, trazas de cobre visibles
- Tornillos de montaje fijando la PCB al cuerpo
- Pegatina triangular amarilla de peligro laser (Clase 1)

### Banda negra central

Ventana optica de policarbonato, transparente al infrarrojo de 905 nm pero opaca a la luz visible. Por aqui salen los pulsos laser y entran los retornos.

### Base gris/plateada

Aluminio mecanizado que proporciona la robustez IP67. Acabado industrial de calidad.

---

## Interfaz electrica y conexion al sistema

Tyler usa el VLP-16 como unidad cerrada. Se conecta directamente via Ethernet al Raspberry Pi 5 a traves del slip ring. **No se necesita una caja de interfaz electrica separada** (confirmado por @biglogclash en comentarios). Tyler no accede directamente al puck ni modifica su electronica interna; usa la electronica de procesamiento integrada del Velodyne (pregunta de @Chekr12).

### Cadena de datos

VLP-16 (captura puntos) --> cable Ethernet --> conector GX12 --> eje hueco de 1 pulgada --> slip ring --> Raspberry Pi 5

---

## Datos de intensidad

Cada punto incluye un valor de intensidad que mide la reflectividad de la superficie impactada:

- Materiales retroreflectivos (senales de trafico, cinta de seguridad) devuelven senales muy brillantes
- Superficies oscuras o anguladas devuelven senales debiles
- El canal de intensidad se puede usar para distinguir tipos de superficie en post-procesamiento

---

## Comportamiento sin rotacion adicional

Sin la plataforma rotatoria, el VLP-16 solo captura un "slice" horizontal: 16 lineas horizontales de puntos formando bandas paralelas separadas. No se ve el techo directamente encima ni el suelo a los pies del operador. Todo lo que este a mas de 15 grados por encima o por debajo del sensor es punto ciego. Esto es perfectamente valido para vehiculos autonomos (importa lo que hay delante en la carretera), pero es insuficiente para mapeo volumetrico de cuevas. Esta es exactamente la razon por la que Tyler construye la plataforma rotatoria.

---

## Montaje en el sistema

El VLP-16 se asienta en una copa/horquilla impresa en 3D (la pieza mas iterada de todo el proyecto) que esta prensada sobre el eje de rotacion de 1 pulgada de diametro. El cable Ethernet sale del sensor a traves del conector GX12, pasa por el interior del eje hueco y a traves del slip ring hasta la Raspberry Pi 5.

Fotos recientes muestran una inclinacion de aproximadamente 15 a 30 grados respecto al eje, consistente con la recomendacion de @Spirit532 para obtener datos mas utiles. Sin inclinacion hay redundancia significativa en los datos; con inclinacion, cada posicion rotacional contribuye datos angulares mas diversos.

Para pruebas sin rotacion, Tyler usa un tripode pequeno con rotula roja marca Ulanzi (accesorio de camara/video reutilizado como soporte estatico).

---

## Por que Tyler eligio el VLP-16

### 1. Disponibilidad y coste

El VLP-16 esta en produccion desde 2014. La industria de vehiculos autonomos paso por miles de estos sensores durante desarrollo y pruebas. Muchas empresas han migrado a sensores mas nuevos, actualizado sus flotas o directamente cerrado. Eso significa que VLP-16 usados aparecen constantemente en el mercado secundario. Tyler pago $321 por el suyo (precio retail original: ~$8.000).

### 2. Ecosistema y soporte

El VLP-16 tiene drivers maduros y probados en batalla para ROS y ROS 2. El paquete del driver de Velodyne lleva anos disponible. Cuando se intentan depurar algoritmos SLAM y el mapa esta derivando o los cierres de bucle estan fallando, lo ultimo que se quiere es tambien estar cuestionando si el driver del sensor esta dando buenos datos. Ademas hay una enorme cantidad de conocimiento comunitario: cualquier problema que se encuentre, probablemente alguien ya lo tuvo y escribio sobre ello.

### 3. Durabilidad

Disenado para ir montado encima de vehiculos circulando por San Francisco. Lluvia, niebla, tormentas, vibracion. Calificacion IP67, temperatura de operacion de -10 a 60 grados Celsius. Para trabajo de campo en cuevas con humedad, grandes cambios de temperatura y golpes ocasionales contra paredes de roca, esa durabilidad importa.

Tyler admite que si empezara de cero o tuviera presupuesto ilimitado, probablemente elegiria un sensor diferente. Pero el VLP-16 da en el clavo en el equilibrio entre capacidad, coste, ecosistema y calidad de construccion.

---

## Alternativas mencionadas en todas las fuentes

### Alternativas directas al VLP-16

**Hesai de 40 canales (~$200 en eBay)**
Mencionado por @LeLehel, quien posee ambas unidades (Hesai y Velodyne). Ofrece 2.5x mas canales por la mitad del precio. Esto significa 2.5x mas resolucion vertical por barrido, lo que podria reducir la necesidad de rotacion en segundo eje o mejorar dramaticamente la densidad de la nube de puntos. Dato critico de precio/rendimiento. El usuario afirma que el Hesai tiene mejor calidad.

**Livox Mid-360**
Patron de escaneo no repetitivo que se rellena con el tiempo. Buena densidad de puntos por dolar. Funciona muy bien para SLAM (compatible con FAST-LIO2). Inconvenientes: el patron de escaneo puede ser complicado de manejar y son bastante mas caros que el VLP-16 de segunda mano. Mencionado por @fx_node, @andbondstyle y @hrmny_.

**Robosense Airy**
LiDAR solid-state compacto, evaluado por @baticzek8 para un proyecto de 3D Gaussian Splatting (3DGS).

**Unitree L2**
Viene con paquetes SLAM integrados, lo que eliminaria gran parte de la complejidad de integracion. Tradeoff: menos control, menos aprendizaje. Mencionado por @Yabatoob.

**Ouster OS1**
Hasta 128 canales verticales y mayor resolucion. Genera imagenes tipo camara junto con la nube de puntos. Mas caro y consume mas energia. Mencionado por Tyler en el video 2.

**Serie RPLiDAR (~$35 a unos pocos cientos de dolares)**
Buenos para mapeo 2D simple pero limitados para cobertura 3D completa en entornos complejos. Mencionado por @Protein_2000gm y por Tyler. Los sensores de ~$35 son probablemente RPLiDAR A1 o similares (2D, rango limitado, baja precision). La diferencia con un VLP-16 es abismal en calidad de datos.

**Hesai PandarXT**
Visualmente similar al Velodyne. @tedzbug07 tiene dos unidades y pregunto si montandolos en angulos diferentes podria evitar la rotacion.

### LiDAR de smartphones

**iPhone Pro LiDAR**
Rango efectivo maximo de aproximadamente 5 metros (~16 pies) segun @mikegrok. @RFNR-72448 (topografo profesional de cuevas en Mid-Missouri) confirma que es insuficiente. @NatetheAceOfficial usa Polycam para scans "MUY rudos" de salas pequenas y trabaja con el desarrollador de CaveWhere para anadir soporte de nube de puntos. @nobilismaximus sugiere conectar en red varios iPhones viejos con LiDAR. @batallapj3 sugiere usar los sensores solid-state del iPhone (Sony IMX590, pero no se venden individualmente al publico para integracion DIY). @emertonom quiere que alguien haga funcionar el LiDAR del iPhone con un Pi. @alien_hd afirma que se pueden lograr resultados similares (incorrecto: el VLP-16 tiene 100 m de rango contra 5 m del iPhone, y 300.000 pts/s contra una resolucion mucho menor).

### Ultra bajo coste / DIY

**Garmin Lidar Lite v3 (~$130)**
Sensor de un solo punto (no escaner 2D/3D). @loosacpl construyo un sistema con dos servos AX12 Dynamixel pero tuvo problemas. Requiere escaneo mecanico completo, velocidad extremadamente lenta, precision del servo insuficiente para registro preciso. Planea migrar a espejos rotativos o LiDAR 2D rotatorio.

**LiDAR de aspiradora robot (robovac)**
2D con resolucion angular de 0.5-1 grado y rango de 6-12 metros. Funciona para navegacion plana pero la densidad para mapeo 3D seria muy baja. Mencionado por @acspider10.

**Xbox Kinect**
Sensor de profundidad (ToF/structured light) con rango de ~4.5 metros y FOV limitado. Funciona para espacios interiores pequenos pero no para cuevas o exteriores. Problemas con superficies reflectantes y luz solar. Mencionado por @arias8185.

**Camaras ToF (Azure Kinect, RealSense L515)**
Rango tipico de 0.5-9 metros contra los 100 metros del VLP-16. @asmotaku pregunto por que no usar camara ToF en vez de LiDAR. La respuesta es clara: rango.

**ArduCam ToF (~$30-80)**
Rango de aproximadamente 4 metros. @ventusprime construyo un sistema con 3 camaras ArduCam ToF.

### Referencia profesional

**Leica BLK360 / RTC360**
$15.000 a $90.000 o mas. Estandar de la industria. @Flumphinator ha usado escaner Leica profesionalmente. @Luci_4 tiene experiencia con BLK360 para modelar una biblioteca escolar. @wvg. ha trabajado con escaneres Leica usando extension de Solidworks (propietario) y pregunta si hay solucion open source para meshing.

**Emesent Hovermap**
$40.000 o mas. Sistema portatil de mapeo LiDAR con SLAM. @peterdickinson1936 es ex-empleado de Emesent y valida el enfoque del proyecto de 9nl. Que un ex-empleado de Emesent apoye el proyecto indica que el enfoque DIY es viable.

---

## Multiples VLP-16 en vez de rotacion

@tourdumonde77 hizo el calculo: para reemplazar un VLP-16 rotatorio con multiples VLP-16 fijos se necesitarian 6 unidades (180 grados / 30 grados de cobertura vertical = 6). Esto seria demasiado caro, pesado, voluminoso y con un consumo de energia 6 veces mayor. No es practico para un sistema portatil de cuevas.

---

## Rendimiento en superficies dificiles

**Roca y arcilla**: @sharknight11 senalo que la roca (especialmente arcillosa) probablemente tiene baja reflectividad, y que un ICP vanilla tendria problemas para extraer detalle preciso de superficies rocosas. Pregunta sin respuesta del creador, pero muy relevante para el caso de uso de cuevas.

**Ambientes con particulas**: @seanmikel pregunto como funciona en ambientes con particulas en suspension (polvo, niebla). Sin respuesta. Es una limitacion conocida del LiDAR de tiempo de vuelo: las particulas generan retornos falsos y reducen el alcance efectivo.

---

## Prevencion del auto-escaneo

@firesnake6311 pregunto como evitar que el escaner se mapee a si mismo. @SchusterMarino respondio: se configuran parametros de filtrado de angulos y se establece un rango minimo para descartar puntos demasiado cercanos al propio sensor.

---

## Comportamiento del laser en superficies especificas

- **Vidrio/ventanas**: Aparecen como huecos oscuros/vacios en la nube de puntos. El vidrio no refleja el laser de 905 nm de forma consistente: la mayoria de la energia pasa a traves o se refleja especularmente lejos del sensor
- **Arboles y follaje**: Las ramas y hojas aparecen como clusters difusos de puntos porque el laser penetra parcialmente el follaje y genera retornos a multiples profundidades
- **Edificios**: Las lineas de tejas del tejado son visibles. Barandillas y porches se distinguen
- **Terreno**: Las imperfecciones del suelo (como marcas de camion de hormigon) se ven mas pronunciadas en la nube de puntos que en persona

---

## Datos de rendimiento medidos

| Parametro | Valor |
|---|---|
| Escaneo estatico de 2 minutos | ~21 millones de puntos, submuestreados a ~13,75 millones |
| Tamano de archivo comprimido | ~400 MB |
| Tamano descomprimido | ~1 GB |
| Frecuencia de grabacion del LiDAR (banco) | 15-16 Hz |
| Frecuencia de grabacion del LiDAR (handheld avanzado) | Hasta 31 Hz |
| Frecuencia del LiDAR segun Tyler (video 2) | ~10 Hz como balance entre densidad de puntos y tasa de actualizacion |

---

## Compra de segunda mano

Tyler compro su VLP-16 en eBay por $321. En el video 1 menciona "unos $400" como precio aproximado. @justanotherape pregunto sobre comprar en eBay; es la via recomendada. La pregunta abierta (sin respuesta) es cual es el mejor test para verificar que un VLP-16 de segunda mano funciona correctamente antes de construir todo el sistema. @lolslim690 advirtio que la popularidad de los videos probablemente hara subir el precio del VLP-16 en eBay.

---

## Observacion con vision nocturna

@dirtynachobuffet sugirio que seria interesante ver el sistema funcionando mientras se llevan gafas de vision nocturna (NVGs). Dado que el LiDAR trabaja en infrarrojo cercano (905 nm), los NVGs podrian captar los pulsos laser, lo cual seria una forma visual de "ver" el LiDAR en accion.

---

## Contexto historico y de mercado

El VLP-16 ha sido el caballo de batalla de la industria de vehiculos autonomos y robotica durante anos. La industria de vehiculos autonomos paso por miles de estos sensores durante desarrollo y pruebas, y ahora muchas empresas han migrado a sensores mas nuevos. Esto crea un flujo constante de unidades usadas en el mercado secundario a precios muy reducidos respecto al precio original.

Velodyne fue adquirida por Ouster. El sensor sigue siendo ampliamente soportado en el ecosistema ROS/ROS2 con drivers probados en batalla.
