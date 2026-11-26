# 08 — Simular máquina y galpón en 3D/VR para vender

> Objetivo: capturar el galpón del cliente, ubicar las máquinas de Chino (envasadora, horno, cintas) armando el circuito de producción, y mostrárselo al cliente en 3D o con el visor puesto **antes** de vender.
> Investigado: noviembre 2026. Lo marcado **(estimación)** es cálculo propio.

## Glosario rápido

- **Layout de planta**: la disposición física de máquinas, pasillos, depósitos y flujos en el galpón.
- **Simulación de eventos discretos (DES)**: simular la producción en el tiempo (cuellos de botella, colas, tiempos de ciclo). Es otra cosa que "verlo en 3D".
- **VR**: realidad virtual, el visor tapa todo y estás en un mundo 3D.
- **MR / passthrough**: realidad mixta; el Quest 3 muestra el mundo real con cámaras y superpone objetos 3D. Clave para "poner la máquina en el galpón real".
- **LiDAR**: sensor láser que mide distancias; los iPhone/iPad Pro lo tienen.
- **Nube de puntos / malla**: lo que sale de un escaneo; millones de puntos o un modelo de triángulos.
- **glTF/GLB**: el "JPG del 3D", formato estándar para web y apps.
- **STEP**: formato estándar de CAD con geometría exacta (no triángulos).
- **WebXR**: API del navegador para VR/MR; funciona en el navegador del Quest sin instalar app.

## 1. Herramientas existentes y precios

| Herramienta | Qué hace | Precio orientativo | ¿Sirve para Chino? |
|---|---|---|---|
| **Siemens Plant Simulation (X)** | Simulación de eventos discretos + 3D, estándar automotriz | Foundation (≤500 objetos) era USD 250/mes en 2020; hoy Essentials/Standard/Advanced, a cotizar. Versión estudiante gratis (80 objetos) | Exagerado para vender a una pyme |
| **FlexSim** (ahora Autodesk) | DES con 3D lindo y soporte VR | ~USD 6.000/año (revendedores) | Bueno para estudiar capacidad, caro |
| **visTABLE (plavis)** | Layout 2D/3D, flujo de materiales, importa DWG/DXF/PDF | ~€249/mes por usuario (anual) | El más parecido al caso, pero de escritorio |
| **Autodesk Factory Design Utilities** | Layout de fábrica sobre AutoCAD/Inventor | Dentro de Product Design & Manufacturing Collection | Si Chino ya usa Inventor, ojo |
| **Theorem XR / TechSoft3D SpinFire XR** | Ver layouts CAD a escala 1:1 en VR/MR, colaborativo | Enterprise, a cotizar | Caro, pensado para grandes |
| **Fraunhofer HoloLayouts** | MR para mover máquinas sobre el galpón real | HoloLens 2, investigación | Muestra que la idea funciona |
| **Apps genéricas en Quest** (Gravity Sketch, ShapesXR, visualizadores de modelos) | Diseño/visualización general | Gratis a ~USD 20–50/mes | Sirven para mostrar un GLB, no para planificar flujo |

**Conclusión**: no encontré una app de Quest específica para "layout de fábrica pyme" barata. Hay hueco: las herramientas serias son caras y de escritorio; las de VR son enterprise.

## 2. Cómo capturar el galpón (y qué precisión esperar)

| Método | Costo | Precisión | Para qué sirve |
|---|---|---|---|
| **Plano + cinta/medidor láser** | Medidor láser USD 30–100 | ±1–3 mm por medida (del medidor); el error real está en olvidar cosas (columnas, desniveles, cañerías) | Base mínima: paredes, columnas, puertas, alturas, tableros |
| **iPhone/iPad Pro con LiDAR + Polycam / app tipo** | Equipo + Polycam Pro ~USD 20/mes (estimación) | Polycam declara ±½" (~13 mm) en ambientes estándar, **empeora en espacios grandes**. Estudio ISPRS 2026: varía mucho entre apps, el mejor ~5 cm RMSE. Matterport (competidor) dice ±100–200 mm para celulares | Contexto visual y verificación; no para la cota crítica |
| **Matterport Pro3** | Cámara desde USD 5.995 + plan Professional (~USD 245/mes, verificar) | ±20 mm a 10 m | Recorrido virtual del galpón muy bueno, también contratable como servicio |
| **Escáner láser terrestre (Leica BLK360)** | USD 16.800–21.000; o alquiler/servicio de agrimensor | 4 mm a 10 m (1 sigma) | Precisión de obra; caro, solo para proyectos grandes |

**Recomendación práctica**: **medidor láser + plano** como verdad dimensional, y **escaneo con celular** como "fondo" visual para que el cliente reconozca su galpón. Si el proyecto es grande, contratar un servicio de escaneo láser.

Riesgo clave: un escaneo de celular en un galpón de 30×60 m puede acumular errores de varios cm a decenas de cm (deriva). Siempre validar contra 3–4 medidas con láser (largo, ancho, distancia entre columnas).

## 3. Desarrollo propio: web vs Unity

### Opción A — Web: three.js / React Three Fiber + WebXR
- **Qué es**: una app web 3D (React) que corre en el navegador de la PC, celular **y** del Quest 3 (WebXR), sin tiendas de apps.
- **A favor**:
  - El dev ya sabe React → curva baja. React Three Fiber + `@react-three/xr` + `drei`.
  - Mismo link para el cliente en la PC y en el visor. Demo por WhatsApp.
  - El navegador del Quest soporta passthrough (MR), hit-test con sensor de profundidad en Quest 3/3S (ubicar objetos al instante) y detección de planos (con setup de la habitación).
- **En contra**:
  - Menos rendimiento que nativo: hay que optimizar mallas (escaneos pesan mucho).
  - No se puede leer el video del passthrough desde WebXR.
  - Soporte de features MR más inmaduro y con reportes contradictorios en foros.

### Opción B — Unity (o Unreal) + Meta XR SDK
- **A favor**: máximo rendimiento, todas las features del Quest (anclas espaciales, malla de escena, colaboración), muchos ejemplos.
- **En contra**: C# y un ecosistema nuevo para el dev; build/instalación en el visor (sideload o tienda); Unity Personal es gratis hasta USD 200k de ingresos + fondeo. Unreal es más pesado todavía para un dev solo.

### Recomendación: **web primero**.

### MVP para un dev solo (estimación)
| Etapa | Contenido | Tiempo (estimación) |
|---|---|---|
| 1 | Visor 3D web: galpón como caja con medidas reales (paredes, columnas, puertas) cargado desde JSON | 1 semana |
| 2 | Catálogo de máquinas GLB (envasadora, horno, cintas) con footprint y zonas de seguridad; arrastrar, rotar, snap a grilla, medir distancias | 2 semanas |
| 3 | Flujo de producción: flechas/rutas entre máquinas, pasillos, chequeo de colisiones y distancias mínimas | 1–2 semanas |
| 4 | Modo VR/MR en Quest 3: recorrer a escala 1:1 con teleport; en passthrough, ubicar la máquina en el galpón real | 2 semanas |
| 5 | Importar escaneo (GLB de Polycam) como fondo, alinear con el plano | 1 semana |
| — | **Total MVP** | **~6–8 semanas** part-time-intenso |

Extras posteriores: animar la cinta/envasado, mostrar producción/hora con un simulador simple de eventos discretos, cotización automática según layout.

### Hardware
- **Meta Quest 3 (512 GB) en Argentina**: ~ARS 1,2 M en Mercado Libre (mínimo histórico registrado ~ARS 1,205 M, mediana ~ARS 1,39 M); Frávega ~ARS 1,56 M. Precios cambian seguido.
- **Quest 3S (128 GB)**: ~ARS 950.000. Alcanza para VR; el Quest 3 tiene mejores lentes y passthrough a color de mejor calidad (más útil para MR en galpón).
- iPhone/iPad Pro con LiDAR si se va a escanear (o pedir prestado / contratar).

## 4. Importar máquinas desde CAD (STEP → glTF)

- **Problema**: el STEP tiene geometría exacta y miles de piezas (tornillos incluidos). Para VR hay que pasarlo a triángulos y **simplificar** mucho.
- **Herramientas**:
  - **Mayo** (gratis, open source): abre STEP/IGES y exporta glTF manteniendo jerarquía, con control de calidad de teselado.
  - **opencascade-tools** (npm) / OpenCASCADE: conversión automatizable por línea de comando con control de deflexión.
  - **step2glb**: convertidor standalone en GitHub.
  - **Blender**: para limpiar, decimar (reducir triángulos), juntar piezas y asignar materiales; después `gltf-transform` para comprimir (Draco/Meshopt).
  - Convertidores online (MeshInspector, etc.): ojo con subir planos confidenciales de Chino.
- **Pipeline sugerido**: STEP → Mayo/OCCT (teselado medio) → Blender (borrar tornillería interna, decimar, materiales) → GLB comprimido → objetivo < 5–10 MB por máquina **(estimación)**.
- **Unidades**: glTF usa metros; verificar escala (los CAD suelen estar en mm).
- **Propiedad intelectual**: un GLB simplificado no revela el diseño interno; nunca mandar el STEP al cliente.

## 5. Riesgos

- **Precisión**: si el cliente compra porque "entraba" en el modelo y en la realidad no entra por 30 cm, el problema es grave. Mitigación: cotas críticas siempre medidas con láser y con margen; disclaimer de que el layout es orientativo hasta relevamiento técnico.
- **Mareo (cybersickness)**: moverse con joystick en VR marea. Usar **teleport y giro por saltos (snap turn)**, velocidades realistas (~1,4 m/s caminando), frame rate estable (72–90 Hz). En MR el mareo es mucho menor porque el usuario camina de verdad.
- **Tiempo**: es fácil enamorarse del 3D y no vender. Acotar al MVP y medir si ayuda a cerrar ventas.
- **Rendimiento**: escaneos y CAD sin optimizar hacen que el Quest vaya a tirones.
- **Usuario no habituado**: un cliente de 60 años con el visor por primera vez. Tener siempre la versión en pantalla/tablet; el visor como "wow" de 5 minutos.
- **Higiene/logística**: visor compartido, cargado, actualizado, protector facial.

## Recomendación concreta

1. **Ya**: app web con React Three Fiber, galpón paramétrico (medidas tipeadas) + catálogo de 3 máquinas en GLB simplificado. Usable en PC y tablet.
2. **Después**: modo WebXR en un Quest 3 para recorrer a escala 1:1 y modo passthrough para "poner" la envasadora en el galpón real del cliente.
3. **Captura**: medidor láser + fotos + escaneo con celular (Polycam) como fondo. Escaneo láser profesional solo si el proyecto lo justifica.
4. **No comprar** Plant Simulation/FlexSim por ahora; considerar visTABLE solo si el problema es planificar flujo complejo y no "mostrar".
5. **Comprar un Quest 3** (no 3S) cuando el MVP web esté andando, no antes.

## Qué le aporta esto a Chino

- **Cerrar ventas**: el cliente "ve" su galpón con la línea armada antes de pagar; baja la incertidumbre.
- **Evitar errores caros**: detectar antes que la máquina no pasa por la puerta, que choca con una columna o que falta espacio para el operario.
- **Diseño de la línea completa**, no solo la máquina suelta: envasadora + cinta + horno → vende más equipos por cliente.
- **Imagen**: muy pocos fabricantes locales muestran sus máquinas en VR/MR; es un diferencial fuerte en ferias.
- **Reutilización**: los GLB de las máquinas sirven para la web, catálogos, manuales interactivos y el gemelo digital con datos reales (nota 04).

## Fuentes

- visTABLE precios (Software Advice): https://www.softwareadvice.co.uk/software/560664/visTable
- visTABLE (Fraunhofer IML): https://iml.fraunhofer.de/en/fields_of_activity/enterprise-logistics/productionlogistics/fabrikplanung/vistable1.html
- Plant Simulation compra online (2020): https://blogs.sw.siemens.com/tecnomatix/buy-plant-simulation-online/
- Plant Simulation X buyers guide: https://www.siemens.com/en-gb/products/tecnomatix/offerings/plant-simulation-x-buyers-guide/
- FlexSim pricing (comparativa): https://www.rfp.wiki/supply-chain-logistics-transportation/supply-chain-planning-solutions/supply-chain-simulation-software/flexsim/enterprise-dynamics
- FlexSim pricing (TrustRadius): https://www.trustradius.com/products/flexsim/pricing
- Autodesk Factory Design Utilities: https://www.autodesk.com/products/factory-design-utilities
- TechSoft3D factory layout XR: https://www.techsoft3d.com/enterprise/spinfire-xr/experiences/factory-layout
- Theorem XR factory layout: https://www.theorem.com/blog/can-extended-reality-simplify-large-scale-design
- Fraunhofer HoloLayouts: https://www.fraunhofer.de/en/press/research-news/2023/january-2023/working-interactively-with-mixed-reality-factory-layouts.html
- Quest passthrough para diseño de puestos (Springer): https://link.springer.com/content/pdf/10.1007/978-3-031-32767-4_7.pdf
- Tesis Chalmers VR + escaneo en layout: https://research.chalmers.se/publication/526665/file/526665_Fulltext.pdf
- Precisión Polycam: https://learn.poly.cam/hc/en-us/articles/50434745985556-How-Accurate-Are-Polycam-Scans
- Estudio ISPRS 2026 apps LiDAR iPhone: https://isprs-archives.copernicus.org/articles/XLIX-M-1-2026/1/2026/
- iPhone 13 Pro en interiores (FIG 2022): https://www.fig.net/resources/proceedings/fig_proceedings/fig2022/papers/ts03g/TS03G_zaczek-peplinska_kowalska_11458.pdf
- Matterport precisión LiDAR: https://matterport.com/es/blog/lidar-accuracy
- Matterport Pro3: https://matterport.com/pro3
- Leica BLK360: https://leica-geosystems.com/en-in/products/laser-scanners/scanners/blk360
- Meta WebXR mixed reality: https://developers.meta.com/horizon/documentation/web/webxr-mixed-reality/
- Quest browser depth hit-test: https://www.uploadvr.com/quest-browser-depth-api-webxr-hit-testing-instant-placement/
- WebXR plane detection spec: https://immersive-web.github.io/plane-detection
- Plane detection Quest 3 (Babylon forum): https://forum.babylonjs.com/t/webxr-plane-detection-on-quest-3-passthrough/57422
- Unity planes y precios: https://unity.com/products
- Meta locomotion y confort: https://developers.meta.com/horizon/design/locomotion-comfort-usability/
- Meta locomotion best practices: https://developers.meta.com/horizon/design/locomotion-best-practices/
- VR sin mareo (heise): https://heise.de/-10572305
- Quest 3 precio ML (tracker): https://mejorescompras.com.ar/precio/MLA28299479--meta-quest-3-512-gb-color-blanco-visor-de-realidad-virtual
- Quest 3 en Frávega: https://www.fravega.com/meta-quest-3-512gb-22929170/p
- Quest 3S precio ML: https://mejorescompras.com.ar/precio/MLA59328186--meta-quest-3s-128-gb-entra-en-la-realidad-mixta
- STEP → glTF (three.js forum): https://discourse.threejs.org/t/how-to-convert-the-format-step-and-stp-in-gltf/27878
- Mayo: https://blenderartists.org/t/mayo-free-viewer-and-conveter-for-cads-files-iges-step-into-gltf-or-obj/1472531
- opencascade-tools: https://www.npmjs.org/package/opencascade-tools
- step2glb: https://github.com/vegarringdal/step2glb
