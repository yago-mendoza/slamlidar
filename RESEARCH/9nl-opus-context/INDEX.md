# 9nl Opus Context — Indice de Conocimiento

Base de conocimiento consolidada del proyecto SLAM LiDAR de Subterranean Systems (Tyler / @9nl / tthom289).
Fuentes: 3 videos de YouTube, 3 repos de GitHub, posts de Instagram, comentarios de la comunidad.

---

## consolidated/ — Conocimiento deduplicado y procesado

### Sistema completo
- [system-overview.md](consolidated/system-overview.md) — Arquitectura, BOM completo con precios, cadena de datos, app de control, cronologia de evolucion del hardware, datos cuantitativos del canal

### Subsistemas de hardware
- [subsystem-lidar.md](consolidated/subsystem-lidar.md) — VLP-16 specs completas, anatomia interna, montaje, TODAS las alternativas mencionadas (Hesai, Livox, Ouster, RPLiDAR, iPhone, ToF, Kinect...), comparativas
- [subsystem-rotation.md](consolidated/subsystem-rotation.md) — Plataforma rotatoria: eje, rodamientos, copa (pieza mas iterada), motor 2804->4015, correa 2GT 3:1, balanceo, offset optico 5mm, tips 3D printing, disenos alternativos, ruido PWM
- [subsystem-imu.md](consolidated/subsystem-imu.md) — IMU Wheeltech N100 specs, montaje, problema del drift sin GNSS, alternativas superiores (VectorNav VN100, SBG), posicionamiento complementario (RTK GPS)
- [subsystem-electronics.md](consolidated/subsystem-electronics.md) — Teensy 4.0, carrier board TeensyFOC (KiCad/JLCPCB), SimpleFOC V1, AS5600 encoder, slip ring LPC-18, conector GX12, cableado, alimentacion
- [subsystem-compute.md](consolidated/subsystem-compute.md) — Raspberry Pi 5 + ROS 2, alternativas (Jetson Orin, NPU HAT), estacion de desarrollo, datos de ROS 2 bags
- [subsystem-timing.md](consolidated/subsystem-timing.md) — Sincronizacion temporal (EL factor critico): PPS, time sync IMU-LiDAR-encoder, frecuencias observadas, parametros de calibracion, soluciones para cuevas sin GPS

### Software
- [software-slam.md](consolidated/software-slam.md) — FAST-LIO, estado del SLAM (drift masivo), fracasos documentados de la comunidad, loop closure, 3DGS, comparacion con fotogrametria, madurez del campo
- [software-deskewing.md](consolidated/software-deskewing.md) — Pipeline offline completo: deskewing + ICP merge + flythrough viewer. Dependencias, controles, parametros, resultados de escaneo, artefactos, fusion multi-escaneo, CloudCompare

### Comunidad y preguntas abiertas
- [community-insights.md](consolidated/community-insights.md) — Profesionales en la comunidad, ofertas de colaboracion, validacion de mercado, proyectos paralelos, sectores de aplicacion, contexto cultural (Prometheus), demanda de contenido
- [open-questions.md](consolidated/open-questions.md) — Preguntas sin resolver por prioridad, desafios tecnicos conocidos, recursos mencionados para investigacion

---

## repos/ — READMEs de los repositorios GitHub (referencia)

- [pointcloud-reconstruction.md](repos/pointcloud-reconstruction.md) — Pipeline Python: offline_deskew.py + icp_merge.py + flythrough.py (MIT, 36 stars)
- [vlp16-spin-controller.md](repos/vlp16-spin-controller.md) — Firmware Teensy 4.0 SimpleFOC para motor BLDC (MIT, C++, 18 stars)
- [teensyfoc-carrier.md](repos/teensyfoc-carrier.md) — Diseno KiCad de la PCB carrier board (MIT, 15 stars)

---

## sources-raw/ — Fuentes originales sin procesar (archivo)

- transcription_video1.md — Video 1: Vision general del sistema
- transcription_video2.md — Video 2: El LiDAR (VLP-16 en detalle)
- transcription_video3.md — Video 3: La plataforma rotatoria
- comments_video1.md — Comentarios video 1 (~448 comentarios, exhaustivo)
- comments_video2.md — Comentarios video 2
- comments_video3.md — Comentarios video 3
- instagram_transcriptions_and_bios.md — Posts y bios de Instagram
- readme_githubrepo1.md — README pointcloud-reconstruction
- readme_githubrepo2.md — README vlp16-spin-controller
- readme_githubrepo3.md — README TeensyFOC-Carrier
