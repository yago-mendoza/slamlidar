# Software — SLAM

Toda la informacion sobre SLAM recopilada de todas las fuentes (videos 1, 2, 3, comentarios y repositorios de GitHub).

---

## Estado actual del SLAM de Tyler

### Algoritmo identificado
- **FAST-LIO** (LiDAR-inertial SLAM basado en filtro) — mencionado en la transcripcion del video 1, identificado por @yxlee2676 a partir de la interfaz visible en el minuto 2:30.
- Posiblemente un fork de **LIO-SAM** — sugerido por @RYU47376.
- Si se agrega camara, **LVI-SAM** es una opcion viable (fusiona LiDAR + visual-inertial).

### Decision deliberada de Tyler
Tyler intencionalmente retrocedio del SLAM para perfeccionar el deskewing:

> "I intentionally took a step back from the SLAM integration to really nail this part down. If the points aren't landing where they should, nothing downstream really matters. Once de-skewing is solid, the SLAM will go back in."

- **Escaneos estaticos + deskewing**: funcionando bien, sin problemas.
- **SLAM**: tiene un problema de **"massive drift"** (deriva masiva).
- **Instagram**: "Getting SLAM reliable is way harder than it looks. Divergence and drift are constant battles, and one bad loop and your whole map falls apart."
- **Plan**: regresar al SLAM una vez que el deskewing este validado.
- **Futuro**: quiere camaras RGB para Gaussian Splatting + VIO (Visual Inertial Odometry) para complementar el SLAM.

---

## Deriva y cierre de bucle (loop closure)

| Usuario | Comentario/Insight |
|---|---|
| @franklynd | La deriva siempre ocurre sin cierre de bucle; cerrar bucles es realmente importante. |
| @peterskourup7635 | Pregunta sobre deriva en grabaciones en movimiento. |
| Tyler (respuesta) | Sin problemas con grabaciones estaticas y deskewing. Los problemas son con SLAM donde hay deriva masiva. |
| @whatilearnttoday5295 | Se necesitan puntos de referencia para un cierre de bucle preciso. Sin tags/reflectores, depende de que la geometria del entorno coincida — puede fallar en entornos repetitivos. |
| @yxlee2676 | Pregunto sobre completitud del mapa y deriva/fallo en la estimacion de pose. |
| @taktoa1 | Si se hace SLAM, por que importa el offset rotacional? El SLAM debe estimar la pose del LiDAR de todos modos. |

---

## Fallos documentados de SLAM en la comunidad

### @MJ-mw3ni — FALLO TOTAL
- **Hardware**: SICK MRS1000 o VLP-16 con IMU de 9 ejes y RTK GPS.
- Hardware **MEJOR** que el de Tyler en algunos aspectos.
- Cita: "I've tried this myself but totally failed. No SLAM was really working and the data even with Kalman filter was so corrupted."
- El filtro de Kalman fue insuficiente.
- **Causas posibles**: mala sincronizacion temporal, IMU de baja calidad a pesar de 9 ejes, configuracion incorrecta del SLAM, falta de segundo eje de rotacion (confirma que necesita "off center rotation axis").
- **LECCION**: el hardware no es suficiente — la integracion, sincronizacion y calibracion son igualmente criticas.

### @loosacpl — FALLO, REPLANIFICANDO
- Construyo con Garmin Lidar Lite v3 + servos Dynamixel.
- Tuvo problemas. Esta replanificando.

### @fernandodiaz4092 — BLOQUEADO
- Tiene VLP-32 pero esta atascado con ROS ("way over my head").

---

## Experiencia SLAM de la comunidad

| Usuario | Experiencia | Detalles |
|---|---|---|
| @_o- | Trabajando con SLAM y ROS 2 por meses | Dos preguntas clave: (1) transformacion de coordenadas del LiDAR rotando en segundo eje (necesita angulo exacto del motor en cada punto), (2) que califica como buen IMU. |
| @benhenderson6966 | Rover + 2D LiDAR + GraphSLAM usando GTSAM en Python | Lo encuentra dificil. GTSAM es enfoque batch vs FAST-LIO que es basado en filtro (incremental). |
| @RYU47376 (PhD primer semestre) | Investigacion academica SLAM | El campo ya esta bastante maduro, dificil hacer nuevas contribuciones academicas. @yxlee2676 estuvo de acuerdo. |
| @bomboclaat9215 | Perspectiva historica | En 2009 el SLAM era nuevo, no habia software de codigo abierto ni hardware accesible. Existia "Rat SLAM" (SLAM con una sola camara). |
| Tyler | Confirmacion | Ahora hay muchos recursos excelentes de codigo abierto. |
| @PeaceIndustrialComplex | Experiencia en investigacion SLAM | Insights tecnicos de alta calidad. |

---

## Software SLAM mencionado

| Software | Tipo | Mencionado por |
|---|---|---|
| **FAST-LIO** | LiDAR-inertial SLAM (basado en filtro, incremental) | 9nl (transcripcion), @yxlee2676 |
| **LIO-SAM** | LiDAR-inertial SLAM | @RYU47376 |
| **LVI-SAM** | LiDAR-visual-inertial SLAM | Mencionado en contexto (si se agrega camara) |
| **GTSAM** | Libreria GraphSLAM (Python/C++, enfoque batch) | @benhenderson6966 |
| **ArduPilot SLAM resources** | Recursos comunitarios | @joshuarespecki1883 |

---

## 3D Gaussian Splatting (3DGS)

- Tyler planea camaras RGB para Gaussian Splatting + VIO.
- LiDAR proporciona geometria precisa; 3DGS proporciona apariencia fotorrealista. Son **complementarios**.

| Usuario | Comentario/Sugerencia |
|---|---|
| @hoseja | Sugiere Gaussian splatting para cuevas. |
| @McCornikus | Usar camara con interruptor remoto grabando timestamp de la posicion del escaner. Lente equivalente a 24mm recomendado (nitido). Supera resultados de PortalCam. Debe ser open source. |
| @ThePrimaFacie | Combinar con fotogrametria Gaussian Splat. |
| @baticzek8 | Evaluando Robosense Airy para 3DGS. |
| @hepburn959 | Agregar texturas con fotogrametria. |

---

## Comparacion con fotogrametria

### @thefosterhousevfx — Experiencia practica
- Uso fotogrametria y LiDAR de telefono para escanear cuevas para VFX.
- La fotogrametria requirio **miles de fotos en HDR de 7 brackets** y **dias de procesamiento**.
- El escaneo con telefono fue mas rapido con escala correcta pero menos detallado y tedioso.
- Un escaner SLAM le habria ahorrado dias.

---

## Sugerencia de red neuronal

- @000Krim: sugiere red neuronal para superposicion automatica de dos o mas mapas.

---

## Madurez del campo SLAM

| Usuario | Perspectiva |
|---|---|
| @RYU47376 (PhD) | El campo ya esta bastante maduro. |
| @yxlee2676 | Esta de acuerdo. |
| @bomboclaat9215 | En 2009 era nuevo, existia "Rat SLAM". |
| Tyler | Ahora hay muchos recursos excelentes de codigo abierto. |
