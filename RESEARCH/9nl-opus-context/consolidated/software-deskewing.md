# Software — Pipeline de Deskewing y Procesamiento de Nube de Puntos

Toda la informacion sobre el pipeline de deskewing y procesamiento de nubes de puntos recopilada de todas las fuentes.

---

## Vision general del pipeline

Pipeline offline de tres etapas para escaneos estaticos (sin SLAM). Repositorio: **pointcloud-reconstruction** en GitHub. Python, probado en **Windows 11 con Python 3.11**. Open3D **no soporta Python 3.13+**.

Tyler deliberadamente se alejo del SLAM para perfeccionar el deskewing primero.

---

## Dependencias

```
pip install numpy mcap mcap-ros2-support zstandard open3d glfw PyOpenGL PyOpenGL_accelerate
```

| Paquete | Usado por | Proposito |
|---|---|---|
| numpy | todos | Matematicas de arrays |
| mcap | offline_deskew, icp_merge | Lectura de bags MCAP |
| mcap-ros2-support | offline_deskew, icp_merge | Decodificacion de mensajes ROS 2 |
| zstandard | offline_deskew, icp_merge | Descomprimir bags .mcap.zstd |
| open3d | icp_merge, flythrough | Registro ICP, I/O de PCD |
| glfw | flythrough | Ventana OpenGL e input |
| PyOpenGL | flythrough | Bindings de OpenGL |
| PyOpenGL_accelerate | flythrough | Aceleradores C opcionales |

Se recomienda entorno virtual (venv).

---

## Topics esperados en el bag

| Topic | Tipo de mensaje | Descripcion |
|---|---|---|
| `/velodyne_points` | `sensor_msgs/PointCloud2` | Frames de escaneo VLP-16 |
| `/rotating_platform/angle` | `std_msgs/Float64` | Angulo del encoder de plataforma en grados |

---

## Etapa 1 — Deskewing (offline_deskew.py)

Corrige el desenfoque de movimiento (motion blur) en cada frame de escaneo interpolando el angulo de rotacion de la plataforma en el momento de adquisicion de cada punto. Puede ser standalone o ser llamado por icp_merge.py.

### Uso
```
python offline_deskew.py <bag_path> [output.npz]
```

### Salida .npz
Contiene:
- `points` — arrays XYZ deskewed por frame
- `timestamps` — nanosegundos por frame
- `angle_times` — timestamps del encoder
- `angle_values` — angulos del encoder en radianes

### Parametros de calibracion (editar al inicio del archivo)

| Parametro | Default | Descripcion |
|---|---|---|
| `ROTATION_AXIS` | `'x'` | Eje de rotacion de la plataforma |
| `ANGLE_OFFSET_DEG` | `184.0` | Offset de calibracion de angulo cero |
| `ROTATION_CENTER` | `[0,0,0]` | Offset del centro optico del LiDAR desde el eje de rotacion |
| `INVERT_ROTATION` | `False` | Invertir direccion de rotacion |
| `ENCODER_TIME_OFFSET_MS` | `0.0` | Offset temporal entre relojes del encoder y LiDAR |
| `MOUNT_RPY_DEG` | `[0,0,0]` | Roll/pitch/yaw del montaje del LiDAR |
| `MOUNT_AXES` | `['+x','+y','+z']` | Remapeo de ejes |

---

## Etapa 2 — ICP Merge (icp_merge.py)

Divide el escaneo completo en slices angulares con superposicion, deskewea cada frame, usa ICP (Iterative Closest Point) para alinear todos los slices. Estrategia de dos pases: **chain -> refine -> re-chain** para minimizar la deriva.

### Por que ICP es necesario
La alineacion ICP compensa imperfecciones mecanicas: runout del eje, oscilacion de rodamientos, tolerancias de hardware que causan que las mitades de escaneo de 180 grados no se alineen perfectamente incluso despues de la correccion de movimiento. El enfoque de slices angulares le da a ICP geometria superpuesta, haciendo el registro robusto.

### Uso
```
python icp_merge.py <bag_path> [opciones]
```

### Argumentos y flags

| Arg/Flag | Default | Descripcion |
|---|---|---|
| `bag_path` | — | Directorio del bag o archivo .mcap |
| `--slices N` | `12` | Numero de slices angulares |
| `--overlap F` | `2.0` | Ancho de ventana del slice como multiplo del ancho de paso |
| `--fine` | off | Voxel mas fino (0.05m) para ICP |
| `--min-fitness F` | `0.3` | Fitness minimo de ICP para aceptar un slice |
| `--no-chain` | off | Alinear todos al de referencia (sin encadenamiento) |
| `--map-voxel F` | `0.01` | Tamano de downsample voxel final (metros); 0 para deshabilitar |
| `--trim-end N` | `0` | Recortar los ultimos N segundos |
| `-o FILE` | `icp_merged.pcd` | Ruta de salida PCD |

### Pipeline interno
Read bag -> Deskew todos los frames -> Construir slices superpuestos + alinear con ICP (chain->refine->re-chain) -> Fusionar puntos centrales con transformaciones alineadas -> Escribir PCD

---

## Etapa 3 — Visor (flythrough.py)

Visor OpenGL en primera persona para archivos .pcd o .npz. Construido a medida porque **CloudCompare y Arvis eran demasiado lentos** para el flujo de trabajo iterativo de deskewing.

### TODOS los controles

#### Navegacion
| Tecla/Accion | Funcion |
|---|---|
| W/S | Avanzar/retroceder |
| A/D | Desplazamiento lateral (strafe) |
| Space/LCtrl | Subir/bajar |
| Left-click drag | Mirar alrededor |
| Mid/right-click drag | Panoramica (pan) |
| Scroll | Velocidad |
| +/- | Tamano de punto |
| C | Ciclar modo de color |
| E | Alternar EDL (Eye-Dome Lighting) |
| L/Shift+L | Intensidad EDL |
| R | Resetear camara |
| P | Imprimir posicion |
| 1-5 | Presets de velocidad |
| X | Filtro de outliers estadistico (SOR) |
| F12 | Guardar/sobrescribir PCD |
| F5-F10 | Rotar escena +-90 grados X/Y/Z |
| F1 | Resetear rotacion |
| Esc | Salir |

#### Modo de comparacion (2+ archivos)
| Tecla | Funcion |
|---|---|
| F2/F3/F4 | Alternar visibilidad de cada nube |

#### Modo Grab (Tab para entrar/salir)
| Accion | Funcion |
|---|---|
| Left-click drag | Trasladar |
| Shift+left-click drag | Rotar |
| Scroll | Empujar/tirar |
| I | Ejecutar ICP |

#### Modo Pick (T para alternar)
| Accion | Funcion |
|---|---|
| Left-click | Seleccionar punto (alterna entre nubes) |
| U | Deshacer |
| G | Calcular alineacion desde 3+ pares (Kabsch SVD + ICP refine) |
| M | Fusionar todas y guardar |

### Modos de color
- Solido
- Mapa de color por altura (height colormap)
- Intensidad
- Origen del archivo (file origin)

### Visual
- Fondo negro
- HUD overlay en la esquina superior izquierda

---

## Resultados de escaneo

### Escaneo estatico de 2 minutos
- ~21 millones de puntos, submuestreado a ~13.75 millones.
- 400MB comprimido, ~1GB descomprimido.

### Otro escaneo
- 1,126,298 puntos en `bag2_2024_01_09-14_16_25_merged_map.pcd` (escaneo mas corto o mas submuestreo).
- La fecha en el nombre del archivo (2024-01-09) tiene discrepancia con la linea temporal del proyecto (2026) — posible error de reloj del sistema o datos de prueba antiguos.

### Objetos distinguibles en escaneo del patio trasero
- Columpio colgando de un arbol
- Cobertizos
- Linea de cerca
- Indentaciones en el suelo del camion de concreto (aparecen exageradas)
- Cubierta de parrilla
- Cooler Yeti
- Lineas electricas
- Arboles con gran detalle

### Arboles
Ramas/hojas como clusters difusos de puntos (el laser penetra parcialmente el follaje, retorna a multiples profundidades). Las copas de los arboles vistas desde abajo forman un dosel rojo (coloreo por altura).

### Edificios
Lineas de tejas del techo visibles. Barandillas y porches distinguibles. Las ventanas aparecen como vacios oscuros/vacios (el vidrio no refleja consistentemente 905nm: la mayor parte de la energia pasa a traves o se refleja especularmente lejos).

---

## Artefactos de escaneo

| Artefacto | Causa |
|---|---|
| Zonas de sombra | Sin linea de vision directa |
| Variacion en densidad de puntos | Mas denso cerca del escaner, mas disperso a distancia |
| Ruido inherente | +-3cm de precision del VLP-16 |
| Ventanas como vacios | El vidrio no refleja consistentemente 905nm |
| Arco circular tenue | Guia de trayectoria de rotacion o esfera de referencia de CloudCompare |
| Imperfecciones exageradas del terreno | Las ondulaciones del suelo se ven mas pronunciadas en la nube de puntos que en persona |

---

## Fusion multi-escaneo (del video 3)

- Se pueden abrir 2+ archivos PCD simultaneamente.
- **F2** para ver independientemente.
- Identificar puntos compartidos (linea de cerca, lado de la casa, pico del techo).
- Marcar **3+ pares de puntos correspondientes** entre escaneos.
- Presionar **G** para alinear (no I — herramienta diferente; Tyler las confundio inicialmente).
- El programa se congela brevemente durante el procesamiento.
- Resultados no perfectos pero buenos: los postes de telefono dan buenas referencias de largo alcance, los lados de la casa coinciden bien con puntos superpuestos.
- Se pueden fusionar colores.
- Menos de 5 minutos de trabajo: frente y lados completos de la casa.

---

## Exportacion y formatos de nube de puntos

| Usuario | Tema | Detalles |
|---|---|---|
| @Alpha0ne | Flujo de trabajo de exportacion LAS | LAS = estandar de la industria. Herramientas open source: PDAL, CloudCompare, laspy/pylas (librerias Python). |
| @wvg. | Nube de puntos a mesh | Tiene experiencia con Leica + extension de Solidworks (propietario). Si existe solucion open source de mesh, "you're really onto something." |
| @fedro6352 | Pipeline completo | Quiere: datos crudos -> alineacion de trayectoria (con RTK opcional) -> suavizado de superficie -> salida final. |

---

## Sugerencias de la comunidad para procesamiento

| Usuario | Sugerencia |
|---|---|
| @_kalia | Pregunta sobre limpiar puntos que un escaneo posterior ve como vacios (problema de smear con ICP+Kinect para objetos en movimiento). |
| @siteking4289 | Para grabaciones de video de nubes de puntos, pausar entre movimientos para mejor compresion de video. |
| @austinclark3495 | Tomas estaticas de nube de puntos son mejores porque la compresion de YouTube no maneja bien datos en movimiento. |
| @PietjeNL | Para stitching, usar pequenas bolas reflectivas en un palo como puntos de referencia visibles desde multiples angulos. |
| @tullgutten | Crear herramienta de referencia de alineacion usando piramide/cono colocable en punto visible desde multiples angulos. |

---

## CloudCompare

- **Version observada**: CloudCompare v2.11.1 (Anoia) [64-bit]
- **Herramientas**:
  - F2 — vista independiente
  - G — registro por pares de puntos (3+ pares)
  - I — ICP iterativo (Tyler lo confundio con G inicialmente)
  - SOR — Statistical Outlier Removal (eliminacion de outliers estadisticos)
- Tambien posible: **RViz** (visor estandar de ROS 2) — se observo interfaz con fondo azul oscuro y cuadricula azul.
