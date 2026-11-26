# 02 — Envasadoras industriales: tipos, control, fallas y métricas

> Nota de investigación para el referente tecnológico (perfil software). Objetivo: entender cómo funciona una envasadora para pymes alimenticias y dónde puede aportar el software.
> Convención: **[NV]** = no verificado contra una fuente primaria o fabricante; tomarlo como hipótesis.

---

## 1. Glosario rápido (para alguien de software)

- **Film / bobina**: el rollo de plástico (laminado, BOPP, PE, etc.) del que se forma la bolsa.
- **Formar–llenar–sellar (FFS)**: la máquina arma la bolsa a partir del film, la llena y la cierra, todo en línea.
- **Mordazas (jaws)**: las "pinzas" calientes que aprietan el film para sellarlo. Hay mordaza **vertical** (costura longitudinal) y **horizontal** (sellos superior/inferior + corte).
- **Dosificador**: el equipo que mide cuánto producto cae en cada bolsa (por peso o por volumen).
- **Fotocélula de registro / eye mark**: sensor óptico que lee una marquita impresa en el film para cortar siempre en el mismo lugar del diseño. Es el "sync" entre el arte impreso y la máquina.
- **Servo**: motor con encoder y driver que permite controlar posición/velocidad con precisión (lo opuesto a un motor "a lo bruto" on/off).
- **PID**: algoritmo de control en lazo cerrado (proporcional–integral–derivativo). Mide (temperatura), compara con el setpoint y ajusta la salida (potencia de la resistencia). Pensalo como un "controlador de feedback" continuo.
- **Ciclo / golpes por minuto (bpm)**: bolsas por minuto. La métrica de velocidad típica.
- **Receta**: conjunto de parámetros por producto/formato (largo de bolsa, temperaturas, peso objetivo, velocidad). Es "config as data".

---

## 2. Tipos de envasadoras

### 2.1 Vertical — VFFS (Vertical Form Fill Seal)
- El film baja por un **tubo formador**, se cierra en forma de tubo con la mordaza vertical, el producto cae **por gravedad** desde arriba, y la mordaza horizontal sella y corta.
- Ideal para: **productos sueltos que fluyen**: granos, legumbres, azúcar, yerba, café, harinas, snacks, golosinas, congelados (IQF), polvos, líquidos/salsas (con dosificador de pistón/bomba).
- Formatos de bolsa: almohada (pillow), fuelle (gusseted), fondo plano/stand-up (máquinas más caras).
- Es **el caso típico para una pyme alimenticia** y probablemente la primera máquina de Chino **[NV: confirmar con Chino el tipo exacto]**.
- Variantes por movimiento:
  - **Intermitente**: el film avanza, para, sella. Más simple, más barata, ~30–80 bpm **[NV, rango típico de mercado]**.
  - **Continua**: el film no para; las mordazas acompañan el movimiento (movimiento tipo "D" o rotativo). Más rápida, más compleja de controlar (sincronización servo–servo).

### 2.2 Horizontal — HFFS / Flow pack
- El film corre horizontal, el producto **se apoya en una cinta** y entra al tubo de film; sella longitudinal por abajo y transversal con mordazas rotativas.
- Ideal para: **productos sólidos, unitarios o frágiles** que no pueden caer: alfajores, barras de cereal, galletitas en bandeja, panificados, jabones, verduras en bandeja.
- Requiere sincronizar **tres ejes**: cinta de alimentación (con "dedos"/empujadores), arrastre de film y mordaza rotativa (en la práctica se resuelve con **leva electrónica / e-cam**).

### 2.3 Dosificadoras (se montan arriba de una VFFS)
| Dosificador | Cómo mide | Para qué producto | Pros / contras |
|---|---|---|---|
| **Multicabezal (multihead weigher)** | 10–14 tolvas con celda de carga; un algoritmo combina las tolvas cuya suma se acerca más al peso objetivo | Snacks, golosinas, congelados, irregulares/frágiles, mezclas | Muy preciso y rápido; caro. El algoritmo de combinación es literalmente un problema de *subset sum* |
| **Volumétrico de vasos (cup filler)** | Vasos de volumen fijo (ajustable telescópico) que rotan y descargan | Granulados que fluyen sin polvo: arroz, azúcar, legumbres, semillas | Barato, robusto. Impreciso si la densidad varía; malo con polvos finos |
| **Tornillo sinfín (auger)** | Un tornillo gira N vueltas (servo/paso a paso); vueltas ≈ volumen | **Polvos**: harina, leche en polvo, café molido, especias, premezclas | El método práctico para polvos que no fluyen. Se puede combinar con corrección por peso neto |
| **Pistón / bomba** | Desplazamiento volumétrico | Líquidos y semi-líquidos (salsas, aderezos, miel) | Requiere sellado más cuidadoso (contaminación de la zona de sello) |
| **Lineal (balanza lineal)** | 1–4 balanzas con vibradores | Granulados, velocidades medias | Alternativa más barata al multicabezal |

---

## 3. Qué controla el PLC en una VFFS

Pensá el PLC como un **loop de tiempo real determinístico** (scan de ms) que lee entradas, ejecuta lógica y escribe salidas. En una VFFS típica:

### 3.1 Arrastre de film (servo)
- Uno o dos servos mueven **correas de arrastre** (a fricción o por vacío) que tiran del film sobre el tubo formador.
- Modos: **por largo** (avanza X mm fijo) o **por registro** (avanza hasta que la fotocélula ve la marca, con corrección).
- En el AS320T-B esto se puede comandar con **salida de pulsos** (tren de pulsos + dirección al driver del servo) — tiene 6 ejes de 200 kHz (ver nota 03).

### 3.2 Mordazas
- **Vertical**: suele ser neumática (cilindro + electroválvula) o fija con banda caliente en máquinas continuas.
- **Horizontal**: neumática (intermitente, más barata) o **servo** (continua o con control de fuerza/tiempo de cierre). La cuchilla de corte suele ir en la misma mordaza.
- Parámetro clave: **tiempo de permanencia (dwell time)** y **presión**. Temperatura + presión + tiempo = calidad del sello.

### 3.3 Temperatura de sellado (PID)
- Resistencias (cartuchos) en las mordazas + **termocupla (TC) o RTD** como sensor + salida por **SSR** (relé de estado sólido) modulando por ancho de pulso.
- El PLC corre un **PID** por zona (vertical, horizontal delantera/trasera). En Delta: módulos **AS04TC/AS04RTD** e instrucción PID en ISPSoft **[NV el nombre exacto de la instrucción PID en AS]**.
- Problema conocido: cada sello "roba" calor (el film absorbe energía), la temperatura cae y tiene que recuperarse antes del siguiente. Fabricantes como Omron proponen PID con auto-tuning y "pre-boost" sincronizado con el movimiento.
- Alternativa: **sellado por impulso** (calienta solo en el instante del sello, típico en PE) vs **calor constante** (laminados).

### 3.4 Fotocélula de registro
- Sensor de contraste/color que detecta la marca impresa.
- El PLC captura la posición del encoder/pulso en el instante de la marca (**entrada de alta velocidad / captura**) y corrige el largo de la próxima bolsa.
- Ventana de lectura (masking): solo acepta marcas dentro de una ventana para ignorar texto/arte impreso.

### 3.5 Dosificación y handshake
- La dosificadora suele tener su propio controlador. El PLC de la VFFS intercambia señales tipo **"listo para descargar" / "descargá" / "descargado"** (handshake por E/S digitales o Modbus).
- Si el dosificador es tornillo sinfín integrado, el PLC puede controlar directamente el servo del tornillo.

### 3.6 Sincronización y secuencia
- Secuencia típica intermitente: *arrastre → sello vertical → descarga de producto → sello horizontal + corte → (repetir)*.
- En continua: leva electrónica entre eje maestro (film) y esclavo (mordaza).
- Además: tensión/frenado del portabobina, detección de **fin de bobina**, alineación lateral del film (EPC), **fechador/codificador** (hot stamp, inkjet, láser o TTO) que imprime lote y vencimiento, aire/vacío, puertas con interlock, parada de emergencia.

### 3.7 HMI
- Pantalla táctil (Delta DOP-100) para recetas, alarmas, contadores, temperaturas, modo manual/ajuste.

---

## 4. Fallas comunes y causas probables

| Síntoma | Causas típicas | Qué podría detectar/ayudar el software |
|---|---|---|
| **Sellado defectuoso** (abierto, débil, quemado, arrugado) | Temperatura fuera de rango; presión o resortes mal ajustados; mordazas desalineadas o no paralelas al tubo; mordazas sucias/gastadas; **producto atrapado en el sello**; film mal centrado o arrugado | Trend de temperatura por sello; correlación temp/presión vs rechazos; alarma de PID inestable; contador de ciclos de mordaza para mantenimiento |
| **Desfase de registro** (corta fuera del arte; bolsas cortas/largas) | Fotocélula sucia o mal posicionada; poco contraste entre marca y fondo; film brillante; **correas de arrastre patinando** (gastadas, vidriadas, vacío débil); tensión irregular; empalmes de bobina; largo de bolsa mal configurado | Histograma de corrección por bolsa; tendencia creciente = correas gastadas (mantenimiento predictivo) |
| **Atascos** (film, producto) | Producto que "puentea" en tolva/tubo; producto cae tarde (timing de descarga vs cierre de mordaza); film que se pega a la banda caliente; bobina mal bobinada | Registro de paradas con causa; análisis de microparadas |
| **Peso fuera de tolerancia** | Densidad variable (volumétrico); celdas de carga sin calibrar; vibraciones; producto pegajoso; velocidad demasiado alta para el dosificador | Datos de checkweigher → SPC (gráficos de control), auto-ajuste del objetivo, alertas de deriva |
| **Fin de bobina / empalme** | Normal operativa, pero mal manejado genera scrap | Predicción de fin de bobina por consumo |

> Fuente de la mayoría: guías de fabricantes (Viking Masek, Rovema). Son blogs comerciales, pero coinciden entre sí.

---

## 5. Métricas: OEE y PackML

### 5.1 OEE (Overall Equipment Effectiveness)
- **OEE = Disponibilidad × Rendimiento × Calidad**
  - **Disponibilidad** = tiempo produciendo / tiempo planificado (pierde por paradas, cambios de formato, fallas).
  - **Rendimiento** = (ciclo ideal × unidades producidas) / tiempo produciendo (pierde por microparadas y por ir más lento).
  - **Calidad** = unidades buenas / unidades totales (pierde por scrap y retrabajo).
- Ejemplo: 62,5% × 66,7% × 75% ≈ **31%**.
- "Clase mundial" se suele citar como **≥85%** (origen TPM, Nakajima). Ojo: no es universal; una línea típica de packaging (pharma) puede andar en ~30%. Para una pyme, **medir** ya es el salto.
- Para software: es un KPI calculable con pocas señales del PLC (estado, contador total, contador de rechazos, velocidad nominal).

### 5.2 PackML / ISA-TR88.00.02
- **PackML** = estándar de la OMAC/ISA para que todas las máquinas de packaging **hablen el mismo idioma de estados**. Versiones: 2008, 2015, 2022.
- Define una **máquina de estados de 17 estados**: *Stopped, Starting, Idle, Execute, Completing, Complete, Holding, Held, Unholding, Suspending, Suspended, Unsuspending, Stopping, Aborting, Aborted, Clearing, Resetting*.
  - **Execute** = produciendo.
  - **Held** = interrumpida por causa **interna** (ej. se acabó el film).
  - **Suspended** = interrumpida por causa **externa** (ej. el equipo de aguas arriba no manda producto / el de aguas abajo está lleno).
  - **Aborted** = parada de emergencia / falla grave.
- **Modos de unidad**: Producción, Mantenimiento, Manual (y otros definidos por el usuario).
- **PackTags**: nombres estándar de variables — **Command** (comandos), **Status** (estado actual, velocidad), **Admin** (contadores, alarmas, tiempos por estado). Son la base para calcular OEE de forma estándar.
- Para alguien de software: es una **FSM con API estandarizada**. Si el PLC de Chino implementa PackML (aunque sea simplificado), cualquier MES/SCADA/dashboard se integra fácil. También existe el **OPC UA Companion Spec de PackML (OPC 30050)**.

---

## 6. Dónde entran visión artificial y control de peso

### 6.1 Checkweigher (controladora de peso dinámica)
- Balanza en cinta **a la salida** de la envasadora: pesa cada bolsa en movimiento y **rechaza** (soplador/brazo) las que están fuera de tolerancia.
- Lazo de **realimentación (feedback)**: puede mandar al dosificador la tendencia del peso promedio para corregir el objetivo (menos "regalo" de producto = plata directa para el cliente).
- Normativa: el contenido neto de productos premedidos está regulado (en Argentina, metrología legal / Res. Mercosur sobre contenido neto) **[NV: buscar la norma exacta]**.

### 6.2 Visión artificial
Casos de uso concretos en envasado:
- **Verificación de fechado/lote** (OCR/OCV): que el lote y vencimiento estén impresos y legibles. Es un requisito de rotulado (ver nota 07).
- **Inspección de sello**: producto atrapado en el sello, arrugas (cámaras normales o hiperespectrales/termografía).
- **Registro/arte**: verificar que el corte cae bien respecto al diseño; detectar bobina equivocada (código de barras/QR en el film).
- **Lectura de código de barras/EAN** para trazabilidad y evitar mezcla de envases.
- **Conteo / presencia** de producto en flow pack.
- Opciones: smart cameras comerciales (Cognex, Keyence, SICK — caras) vs **cámara industrial + PC/edge con OpenCV/YOLO** (barato, acá es donde un perfil ML suma).

### 6.3 Detector de metales / rayos X
- Muy pedido en alimentos (HACCP). Normalmente equipo de terceros integrado por E/S digitales.

---

## 7. Qué le aporta esto a Chino

- **Diferencial de bajo costo y alto impacto**: un panel/OEE + registro de paradas con causa + contador de producción, leyendo el PLC por Modbus TCP. La mayoría de las envasadoras nacionales para pymes no lo traen **[NV: validar con relevamiento de competencia]**.
- **Estructurar el programa como PackML simplificado** (estados + PackTags mínimos) desde la máquina 1: facilita integración futura, dashboards y venta a clientes más grandes.
- **Recetas por producto** en HMI y exportables/versionadas desde software (cambio de formato más rápido = más disponibilidad).
- **Mantenimiento predictivo simple**: contadores de ciclos de mordaza/correas, tendencia de corrección de registro, estabilidad del PID de temperatura.
- **Módulo opcional de calidad**: checkweigher con feedback al dosificador y/o visión para verificar fechado (lote/vencimiento), apalancado en el perfil de datos/ML.
- **Trazabilidad**: registro de lote, receta, operador y parámetros por turno — conecta directo con lo regulatorio (nota 07).

---

## 8. Fuentes

- Viking Masek — Troubleshooting 10 common VFFS issues: https://vikingmasek.com/packaging-machine-resources/packaging-machine-blog/troubleshooting-10-common-vffs-packaging-machine-issues
- Viking Masek — 5 common film-related issues: https://vikingmasek.com/packaging-machine-resources/packaging-machine-blog/troubleshooting-5-common-film-related-issues-vffs-machines
- Viking Masek — Film registration setup (video): https://vikingmasek.com/blog/video-vffs-machine-maintenance-how-set-film-registration-high-speed-packing-machine
- Rovema — 11 most common film related problems: https://www.rovema-na.com/blog/troubleshooting-the-11-most-common-film-related-problems-with-a-vffs
- Rovema — Short/inconsistent bags, registration: https://www.rovema-na.com/tech-notes/vertical-baggers-short-bags-inconsistent-bag-lengths-or-difficulty-holding-print-registration
- Rovema — Vertical seal troubleshooting: https://www.rovema-na.com/tech-notes/while-running-your-continuous-motion-vertical-form-fill-seal-bagger-you-find-vertical-seal-problems
- iFactory — VFFS maintenance (jaws, film drive, sensors): https://ifactoryapp.com/article/vffs-machine-maintenance-sealing-jaws-film-drive-sensors
- SHKPack — Cup fillers vs auger vs multihead: https://shkpack.com/blogs/news/how-to-choose-the-right-metering-equipment-cup-fillers-vs-auger-fillers-vs-multihead-weighers
- Vtops — Volumetric cup vs auger: https://vtops.com/benefits-of-volumetric-cup-filler-vs-auger-filler
- Wikipedia — Multihead weigher: https://en.wikipedia.org/wiki/Multihead_weigher
- Omron — Temperature control in sync with motion (white paper): https://files.omron.eu/downloads/latest/publication/en/temperature_control_in-sync_with_motion_white_paper_en.pdf
- Automate.org — Synchronized control in packaging: https://www.automate.org/motion-control/editorials/how-synchronized-control-enhances-packaging-performance/boa
- Delta — Application note: packaging wrapping machine (ASDA-A2, flying shear/e-cam): https://filecenter.deltaww.com/solutions/download/02/0201/Tab%202_2_A2-A001%20Application%20to%20Packaging%20Wrapping%20Machine.pdf
- Packaging Digest — OEE dashboard / 85%: https://www.packagingdigest.com/automation/what-should-your-oee-dashboard-look-like-
- Automation World — How to calculate OEE: https://www.automationworld.com/factory/plant-maintenance/article/13309925/how-to-calculate-overall-equipment-effectiveness-a-practical-guide
- PharmTech — packaging line capacity (OEE ~30%): https://www.pharmtech.com/view/your-packaging-line-operating-full-capacity
- ISA — TR88.00.02 preview: https://www.isa.org/getmedia/300dbd50-d549-41ac-b372-a5e52f32fc97/tr_880002_preview.pdf
- OPC Foundation — PackML Companion Spec (OPC 30050): https://reference.opcfoundation.org/specs/OPC-30050/4.2
- Beckhoff — PackML overview: https://beckhoff-usa-community.github.io/SPT-Libraries/Getting_Started/PackML%20Overview.html
- Schneider — PackML base state model: https://product-help.schneider-electric.com/Machine%20Expert/V2.2/en/PackMLli/PackMLli/TPC_PackMLli_FB_UnitModeManager2_StateModel.html
- CODESYS — OMAC PackML State Machine: https://us.store.codesys.com/media/n98_media_assets/files/000060-F/0/OMAC%20PackML%20State%20Machine_en.pdf
