# Extracción Completa de Comentarios Relevantes — Proyecto DIY LiDAR Scanner (9nl)

Link: https://www.youtube.com/watch?v=sGpfHk-vEkg

> Fuente: Video "I Built a Scanner that Sees in Total Darkness" — Canal 9nl (YouTube)
> Componentes base del proyecto: Velodyne VLP-16 (~$400 eBay), IMU Wheeltech N100, Raspberry Pi 5, ROS 2, motor gimbal, microcontrolador Teensy, algoritmo SLAM (FAST-LIO)

---

## 1. Hardware — Alternativas y Recomendaciones

### LiDAR

- **@LeLehel**: Sugiere usar **Hesai de 40 canales por ~$200 USD** en lugar del Velodyne VLP-16 de 16 canales a $400. Afirma tener ambos y que el Hesai tiene mejor calidad.
- **@fx_node**: Considera usar el **Livox Mid-360** como alternativa al VLP-16.
- **@baticzek8**: Evalúa el **Robosense Airy** para un proyecto similar de 3D Gaussian Splatting (3DGS).
- **@lolslim690**: Advierte que la popularidad del video probablemente hará subir el precio del VLP-16 en eBay. Pregunta por alternativas.
- **@Protein_2000gm**: Planea usar **sensores de $35** para replicar el proyecto, pero reconoce que podría necesitar actualizar tras ver los resultados del VLP-16.
- **@acspider10**: Plantea usar el **LiDAR de un robovac (aspiradora robot)** reutilizado, cuestionando si la resolución sería suficiente.
- **@Yabatoob**: Menciona el **Unitree L2 LiDAR** que ya viene con paquetes SLAM integrados.
- **@nobilismaximus**: Sugiere usar **iPhone con LiDAR** (app 3D Scanner, ~$70) o conectar en red varios iPhones viejos con LiDAR.
- **@mikegrok**: Aclara que el **LiDAR del iPhone Pro tiene rango efectivo máximo de ~16 pies (~5m)**. Útil para objetos cercanos, no para mapeo a distancia.
- **@batallapj3**: Sugiere usar los **sensores solid-state del iPhone**, que son baratos y se podrían integrar en un casco.
- **@Chekr12**: Pregunta si usa la electrónica de procesamiento del Velodyne o si accede al puck directamente.
- **@biglogclash**: Pregunta si se puede usar el VLP-16 solo como sensor o si también se necesita la caja de interfaz eléctrica.
- **@loosacpl**: Construyó algo similar con **Lidar Lite v3 de Garmin** y dos servos **AX12 Dynamixel**, pero tuvo problemas. Planea migrar a espejos rotativos o LiDAR 2D rotatorio como el de 9nl.
- **@DHANESHWARANSANKARSTUDENT-AERO**: Pregunta dónde conseguir el LiDAR o si hay forma de hacerlo DIY.

### IMU (Unidad de Medición Inercial)

- **@ChristophFretter**: Expresa especial interés en la **selección del IMU** y el algoritmo SLAM.
- **@janrafflewski5468**: Sorprendido de que el posicionamiento basado en IMU sea lo suficientemente confiable. Quiere saber cómo se reduce el drift a un nivel aceptable.
- **@kjundvr**: Usa un **SBG IMU** para un proyecto similar orientado a demos de secundaria.
- **@Martarts**: Ofrece un **VectorNav VN100** al creador (IMU de grado superior).
- **@OttoSphalta**: Identifica el problema principal de IMUs baratos (Arduino/ESP): **no procesan datos de aceleración con suficiente frecuencia**, lo que causa error acumulativo donde el movimiento medido es menor y se desfasa con el tiempo.
- **@melonenlord2723**: Pregunta sobre el **drift del IMU a largo plazo**. Normalmente se corrige con GNSS, que no está disponible underground.
- **@erfindungsbureau**: Pregunta por qué no colocar el IMU en la parte rotativa (en vez de la fija).

### Posicionamiento y GPS

- **@JasonThomasHorn**: Recomienda **RTK GPS** para mejorar precisión en áreas abiertas. La estación base actúa como servidor GPS y establece un punto home. Sugiere opciones de **SparkFun**.
- **@whatilearnttoday5295**: Señala que se necesitan **puntos de referencia** para el loop closing, que de otro modo puede ser impreciso.

### Computación

- **@Prod.M00N**: Pregunta qué se podría lograr con un **NVIDIA Jetson Nano Orin Super** en lugar del Raspberry Pi.
- **@wchutcheson**: Sugiere explorar el **NPU hat para el Pi** — el hardware de Vision AI podría adaptarse para láser.
- **@Maxjoker98**: Pregunta si el **Raspberry Pi puede manejar ROS 2** y si es suficiente para el procesamiento.

### Motor / Plataforma Rotativa

- **@manfredthuering9745**: Pregunta sobre la **precisión de posicionamiento de la rotación por correa** y su control.
- **@LordPepeetus**: Comenta que tenía un sistema sin gimbal y que la **combinación de rotación es buena idea**.
- **@Spirit532**: Recomienda **inclinar el puck 15-30 grados** para obtener datos mucho más útiles.
- **@davidrollings5549**: Propone usar un **espejo a 45° rotando alrededor del sensor fijo** como alternativa mecánicamente más simple. El creador respondió a esto.
- **@projectSurya-r4h**: Propone alternativa: **tornillo de avance (lead screw) con stepper** para obtener cortes horizontales a cada nivel de altura y luego hacer mesh de todas las rebanadas.

### Cableado / Mecánica

- **@bluegizmo1983**: Destaca el desafío de las **conexiones de cable que pasan a través del eje de rotación**.
- **@whssavy**: Igualmente intrigado por cómo logra girar sin enredar los cables.
- **@readyrepairs**: Pregunta si usa **PCB/disco con contactos de cobre/carbón** para transmisión de datos/energía a través de la parte rotativa (slip ring).
- **@derjansan9564**: Pide que explique cómo funciona la **conexión de datos con el LiDAR rotando**.

---

## 2. Software y Algoritmos

### SLAM

- **@_o-**: Trabaja con **SLAM y ROS 2** desde hace meses. Interesado en cómo maneja el LiDAR rotativo y qué califica como buen IMU.
- **@benhenderson6966**: Trabaja en un proyecto con rover, LiDAR 2D y SLAM. Usa **GTSAM en Python para GraphSLAM** pero lo encuentra difícil. Pregunta qué variedad de SLAM usa 9nl y qué librerías.
- **@MJ-mw3ni**: **Intentó lo mismo y falló**. Ningún SLAM funcionaba y los datos estaban corruptos incluso con filtro de Kalman. Usó SICK MRS1000 o VLP-16 con IMU de 9 ejes y RTK GPS. Necesita la rotación en eje descentrado.
- **@PeaceIndustrialComplex**: Nota técnica importante: El VLP-16 **no tiene IMU integrado**. Si logra buena **sincronización temporal (time sync) vía PPS** entre el IMU USB y el Velodyne, reducirá significativamente el ruido en la nube de puntos final.
- **@andbondstyle**: Trabaja en sistema similar inspirado en el paper **"Alpha LiDAR"**. Usa Livox MID-360 y motor gimbal con eje hueco. Señala que la **sincronización temporal es crítica** para registro preciso de nube de puntos. Normalmente se usa señal **GPS/PPS** ligada a cada componente (LiDAR, IMU, encoder del motor). Pregunta cómo maneja time sync.
- **@joshuarespecki1883**: Recomienda recursos de **SLAM en la comunidad ArduPilot** y comunidades adyacentes.
- **@fernandodiaz4092**: Tiene hardware similar (VLP-32) pero no ha logrado superar los obstáculos de ROS ("way over my head").
- **@privateCitizen-x5g**: Experiencia con datasets LiDAR grandes capturados con helicóptero. Confirma que usan **formatos propietarios** y software de suscripción sin escalabilidad. Espera que los paquetes SLAM open source funcionen.
- **@fedro6352**: Interesado en el **post-procesamiento**: alineación de trayectoria con módulo RTK para mejor rendimiento en áreas abiertas, y suavizado de superficies.
- **@cckeysify**: Pregunta por el repositorio Git. Espera que **no sea Python**, ya que las empresas comerciales no usan Python para esto.
- **@lanealucy**: Pregunta cómo se llama el sistema sin plataforma rotativa (LiDAR 2D + IMU + ROS 2 para nube de puntos 3D) y qué buscar en Google.

### Software de Visualización / Exportación

- **@JYAA_Williams**: Pregunta qué **software usa para visualizar** los datos del Velodyne en el Pi. Ha trabajado con HDL-64e pero solo con herramientas propietarias.
- **@Alpha0ne**: Interesado en el **workflow para exportar a archivo LAS** o similar para procesamiento posterior.
- **@wvg.**: Pregunta si puede **exportar la nube de puntos y convertirla en mesh**. Ha usado escáneres Leica con extensión de Solidworks (propietario). Si hay solución open source para el meshing, "you're really onto something."
- **@ethansmith244**: Quiere saber qué **software genera las imágenes 3D**. No tiene experiencia previa relevante.

### Tiempo de Sincronización (Time Sync)

- **@PeaceIndustrialComplex** y **@andbondstyle** coinciden: la **sincronización temporal hardware (PPS)** entre todos los componentes es crítica para calidad de datos.

---

## 3. Aplicaciones Propuestas por la Comunidad

### Cuevas y Minas

- **@user-ed7gm7ol8k**: "El proyecto open source que todos los nerds de cuevas esperaban en silencio."
- **@RFNR-72448**: **Trabajador profesional como cartógrafo y topógrafo de cuevas** en Mid-Missouri. Algunos colegas usan LiDAR de iPhone pero buscan un sistema dedicado económico.
- **@PrebleStreetRecords**: Sugiere colaboración con el canal **"Ghost Town Living"** (mina Cerro Gordo), que intenta mapear toda la mina con LiDAR.
- **@wibblywobblyidiotvision**: Quiere documentar **minas de pizarra abandonadas de 100 años**.
- **@TrashfireLab**: Quiere mapear **pozos mineros** en su área.
- **@NatetheAceOfficial**: **Actualmente captura scans LiDAR con iPhone** para datos de cuevas complementando topografía digital y tradicional. Trabaja con el desarrollador de **CaveWhere** para añadir soporte de nube de puntos. Limitado a scans muy rudos de Polycam en salas pequeñas. Pregunta si irá a la **NSS Convention** en Indiana.

### Drones

- **@dbillionaer**: Sugiere **montarlo bajo un dron** (no a velocidad máxima).
- **@TomS699**: Aclara que esto es lo que la mayoría de drones de mapeo LiDAR ya hacen, pero en versión DIY.
- **@PencilParasite**: Propone **3 LiDARs hemisféricos montados equidistantes en un anillo** perpendicular al suelo, con el dron pasando por el centro.
- **@thestyleofbeyond3**: Quiere esto conectado a un dron sin pagar $80,000.
- **@Zapato56**: Pregunta si se puede adaptar a un dron.
- **@euemeubarco**: Ha volado DJI Matrice con LiDAR pero quiere construir el propio.

### Otras Aplicaciones

- **@finnsjustanotherday1481**: Edificio de **+1 millón de pies cuadrados** construido en los 70, remodelado múltiples veces, planos desordenados. Quiere mapear pasillos y salas.
- **@danielmcmath2754**: Quiere construir **versión submarina**.
- **@HugoTremolo**: Quiere **escanear etapas de rally locales con LiDAR** para construirlas en simuladores de carreras (RBR y otros).
- **@MePeterNicholls**: Necesitan **solución de bajo coste para escanear superficies de carreteras** para ayuntamientos/autoridades locales.
- **@borkuz**: Sugiere hacerlo **resistente al calor para bomberos**.
- **@Púca-h2q**: Propone arnés de pecho conectado a **gafas VR/AR** (Bigscreen V2).
- **@elmichellangelo** y **@Google_Does_Evil_Now**: Aplicaciones **militares, caza, y carreras de motor**.
- **@kbrere14**: Quiere construir **cortacésped autónomo** para granja usando LiDAR.
- **@AkiiiiDesu**: Para **mapear montañas rusas en la oscuridad** (Disneyland).
- **@Najsnwjsbdn**: Para **buceo** — el mayor miedo es no poder ver las propias manos en baja visibilidad.
- **@MilovanLoon**: Aplicaciones **geoespaciales**.
- **@Fishdonotexist**: Sugiere añadir **slot para cámara QD mount** para superponer tour 360 con datos LiDAR. Quiere mantenerlo open source para evitar suscripciones mensuales.
- **@nicklaeder5628**: **Ingeniero civil y de datos** que usa fotogrametría con dron. Quiere colaborar en algo open source usable por la industria.
- **@XAirForcedotcom**: Equipos de rescate que no pueden permitirse los sistemas comerciales.

---

## 4. Mejoras Técnicas Propuestas

### Cobertura y Sensores Adicionales

- **@Nanamowa**: Propone **láseres con diferente profundidad de penetración** disparando en dirección opuesta al láser principal del LiDAR, para capas superpuestas que construyan imagen más completa.
- **@prob_io7299**: Sugiere añadir **función infrarroja** como capa adicional alineada con la regular.
- **@hepburn959**: Sugiere añadir **texturas con fotogrametría**.
- **@McCornikus**: Para usar datos SLAM con **3D Gaussian Splatting (3DGS)**: usar cámara con disparo remoto que registre timestamp de la posición actual del escáner. El posicionamiento espacial preciso es importante. Con lente sharp (24mm eq recomendado) se superan fácilmente los resultados de PortalCam.
- **@ThePrimaFacie**: Sugiere combinar con **fotogrametría Gaussian Splat**.
- **@ventusprime**: Creó algo similar pero con **3 cámaras ArduCam ToF**.

### Navegación en Cuevas

- **@Nanamowa**: Propone crear un **"GPS de cueva"**: asumir posición conocida fuera (GPS), luego medir distancia y rotación del dispositivo para localización dentro de la cueva.
- **@Maxjoker98**: Sugiere implementar **"minimapa" para cuevas** y path finding.

### Mecánica / Compacidad

- **@Scrogan**: Sugiere hacer más compacto: **incluir Pi y batería en la plataforma giratoria**, o usar motor con más torque que no necesite reducción por correa. Señala que la comunidad open source traerá perspectivas frescas.

### Láser / Seguridad

- **@darth_dan8886**: Pregunta cómo los láseres **no dañan ojos y cámaras**. Ha oído que los LiDAR de coches tienen ese problema.
- **@TheEmptyHoliness**: Pregunta por qué estos láseres infrarrojos **no son peligrosos para los ojos** y qué los diferencia de un láser infrarrojo típico.
- **@AdrianInGaming**: Advierte que eventualmente podría **dañar una cámara**.

---

## 5. Preguntas Técnicas Específicas Relevantes

| Pregunta | Usuario |
|----------|---------|
| ¿Hay repo de código? | @notexpected, @domenicoacierno2044, @christopaaron, @jsk_0211 |
| ¿Lista de hardware y software completa? | @jeffs9503, @101pcgamer |
| ¿Se publicará open source? | @Alice.59, @Tebbit123, @nuclearbwld, @kanunssol1246, @icarusneverfalls1, @Karol.Szykula |
| ¿Tutorial / video de construcción completo? | @daftpunk1270, @cmoreno41, @CrunchySandwich, @bigsmoke8377, @KS97RLA, @madanm7454, @RandomVideos-bq2xn |
| ¿Precesión o nutación al girar LiDAR en segundo eje? | @emertonom |
| ¿Verificación de distancia para objetos pequeños como drones? | @andreykonovalov7306 |
| ¿Se puede usar datos para juegos 3D? | @ItsTanmoyYT-k1q |
| ¿Funciona bajo agua con carcasa estanca? | @RandomAdhdQuest, @scarface7859, @armiman123 |
| Corrección de refracción en agua salada | @scarface7859 |
| ¿Se puede construir un GPR (Ground-Penetrating Radar)? | @baitullahkhan4485 |
| ¿Puede detectar tumbas bajo tierra (LiDAR subterráneo)? | @geovannygarcia7685 |
| ¿Rango máximo del LiDAR? | @f00busb4r-g4i |
| ¿Se puede construir un SAR (Synthetic Aperture Radar)? | @OXYPLIK |

---

## 6. Proyectos y Experiencia Relacionada de la Comunidad

- **@peterdickinson1936**: Ex-empleado de **Emesent** (empresa de mapeo LiDAR autónomo).
- **@Flumphinator**: Ha usado **escáner Leica** profesionalmente.
- **@marcmapeke**: Trabaja en **depth estimation** en industria e investigación. No sabía que los LiDARs eran tan baratos en eBay.
- **@JonasBareiss**: Tiene un **VLP-32** desde hace años para proyecto similar, nunca lo empezó.
- **@MJ-mw3ni**: Intentó y falló con SICK MRS1000/VLP-16 + IMU 9 ejes + RTK GPS. Ningún SLAM funcionaba.
- **@loosacpl**: Construyó versión con Garmin Lidar Lite v3 + servos Dynamixel, tuvo problemas.
- **@_o-**: Lleva meses trabajando con SLAM y ROS 2.
- **@benhenderson6966**: Proyecto activo con rover, LiDAR 2D y GraphSLAM (GTSAM Python).
- **@emertonom**: Busca que alguien haga funcionar el LiDAR del iPhone con un Pi.
- **@andbondstyle**: Sistema en progreso inspirado en paper "Alpha LiDAR", usando Livox MID-360 y motor gimbal con eje hueco.

---

## 7. Tips y Lecciones Clave de la Comunidad

1. **Time sync es crítico** — Sincronización hardware (PPS) entre LiDAR, IMU y encoder de motor reduce drásticamente el ruido en la nube de puntos. (@PeaceIndustrialComplex, @andbondstyle)

2. **Inclinar el puck 15-30°** produce datos significativamente más útiles. (@Spirit532)

3. **IMUs baratos tienen problema de frecuencia de muestreo** — los datos de aceleración no se procesan lo suficientemente rápido, causando drift acumulativo. (@OttoSphalta)

4. **El drift del IMU sin GNSS es un problema no resuelto trivialmente** — se necesitan técnicas de compensación. (@melonenlord2723, @janrafflewski5468)

5. **Puntos de referencia son necesarios** para loop closing preciso. (@whatilearnttoday5295)

6. **El LiDAR del iPhone tiene rango de solo ~5m** — no apto para mapeo a distancia. (@mikegrok)

7. **Hesai 40ch puede ser mejor opción que VLP-16** por precio y calidad. (@LeLehel)

8. **RTK GPS como complemento** para áreas abiertas mejora la precisión. (@JasonThomasHorn, @fedro6352)

9. **El paso de cables a través del eje rotativo** es un desafío de ingeniería significativo (slip rings). (@bluegizmo1983, @readyrepairs)

10. **GraphSLAM con GTSAM en Python es difícil de trabajar** — considerar alternativas. (@benhenderson6966)

11. **Kalman filter no fue suficiente** para corregir datos corruptos en al menos un intento. (@MJ-mw3ni)

12. **Para 3DGS, añadir cámara con timestamp sincronizado** y lente de 24mm eq. (@McCornikus)

13. **Los formatos propietarios son un problema real** en la industria — open source es necesario. (@privateCitizen-x5g, @Fishdonotexist)

14. **Espejo rotativo a 45° como alternativa mecánica** más simple a rotar todo el sensor. (@davidrollings5549)

15. **Considerar motor con eje hueco** para pasar cables sin slip ring. (@andbondstyle)

---

## 8. Ofertas de Colaboración

- **@_relaxtation**: Trabaja en el mismo problema, ofrece colaborar.
- **@JohnnysaidWhat**: Startup de automatización e IA en upstate NY, ofrece asociación.
- **@nicklaeder5628**: Ingeniero civil y de datos, ofrece desarrollo open source para industria.
- **@wilsonwj**: Ofrece acceso a una cueva para pruebas.
- **@Martarts**: Ofrece un VectorNav VN100.
- **@seaboom7877**: Ofrece cotización para tooling de plástico y metal, así como precios unitarios.
- **@000Krim**: Ofrece colaboración con otro canal de YouTube.
- **@NatetheAceOfficial**: Ofrece testing en cuevas específicas y trabaja con dev de CaveWhere.

---

## 9. Recursos Mencionados

| Recurso | Contexto |
|---------|----------|
| Paper "Alpha LiDAR" | Inspiración para sistema similar (@andbondstyle) |
| Comunidad ArduPilot | Recursos de SLAM y proyectos relacionados (@joshuarespecki1883) |
| GTSAM (librería Python) | GraphSLAM implementation (@benhenderson6966) |
| CaveWhere | Software de topografía de cuevas, en desarrollo de soporte para nube de puntos (@NatetheAceOfficial) |
| SparkFun | Opciones de RTK GPS (@JasonThomasHorn) |
| NSS Convention (Indiana) | Evento de espeleología para networking (@NatetheAceOfficial) |
| Canal "Ghost Town Living" | Mina Cerro Gordo, potencial colaboración de mapeo (@PrebleStreetRecords, @ablazedave, @backacheache, @minerharry) |
| App Polycam | Scans rudos con iPhone (@NatetheAceOfficial) |
| App 3D Scanner (~$70) | Alternativa iPhone (@nobilismaximus) |

---

## 10. Contexto del Video (Transcripción)

### Componentes del Sistema

| Componente | Detalles |
|------------|----------|
| **LiDAR** | Velodyne VLP-16, 16 canales, ~600 RPM internos, 300,000 puntos/segundo, ~$400 eBay (retail ~$8,000) |
| **IMU** | Wheeltech N100, mide aceleración y rotación ~400 veces/segundo |
| **Computadora** | Raspberry Pi 5 con ROS 2, gestiona LiDAR + IMU + logging para SLAM |
| **Plataforma rotativa** | Motor gimbal + microcontrolador Teensy, gira todo el LiDAR en un segundo eje |
| **Coste total** | Menos de $1,000 USD (+ muchas horas de trabajo) |

### Por qué Rotar

El VLP-16 cubre solo ~30° verticalmente. Estático, produce un anillo de datos. Rotando en un segundo eje, los 16 haces barren todos los ángulos (techo, suelo, paredes) — cobertura volumétrica completa desde una sola posición.

### Videos Planeados (por el creador)

1. Cómo funciona el LiDAR, qué ve, qué no, y por qué el VLP-16
2. Setup del Raspberry Pi y ROS 2, cómo se comunican las piezas
3. Plataforma rotativa: selección de motor, FOC simple, Teensy, mecánica
4. SLAM (FAST-LIO): cómo convierte datos crudos en mapa con tracking de posición en tiempo real

---

## 11. Respuestas del Creador (9nl) Mencionadas en Comentarios

- Planea llevar el escáner a una **cueva pronto** y grabar el viaje.
- Publicará más videos detallados sobre cada componente.
- Interactúa positivamente con la comunidad (múltiples respuestas marcadas).
- Tiene presencia también en **Instagram y TikTok** (@ceater7726 menciona verlo repetidamente).
- Ya publicó video posterior: **"My LIDAR Was Half Blind (so I fixed it)"** y **"The LiDAR That Changed Robotics (And Why I Bought One)"**.