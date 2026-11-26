# 07 — Normas: alimentos, seguridad de máquinas y ciberseguridad (Argentina)

> Nota de investigación, **no es asesoramiento legal**. **[NV]** = no verificado contra texto oficial vigente. Las normas argentinas cambiaron bastante en 2024–2025 (desregulación de reglamentos técnicos), así que todo se debe revalidar antes de comprometer algo con un cliente.

---

## 1. Glosario rápido

- **CAA**: Código Alimentario Argentino (Ley 18.284). Es "el reglamento" de alimentos: qué se puede fabricar, con qué materiales, cómo se rotula.
- **ANMAT / INAL**: autoridad sanitaria nacional de alimentos (INAL es el instituto dentro de ANMAT). Las provincias/municipios aplican el CAA vía sus bromatologías.
- **SENASA**: autoridad para productos de origen animal/vegetal primarios (carnes, lácteos en ciertos casos, frutas y hortalizas frescas, etc.).
- **RNE / RNPA**: Registro Nacional de Establecimiento / Registro Nacional de Producto Alimenticio. Números que tienen que figurar en el envase.
- **BPM / HACCP**: Buenas Prácticas de Manufactura / Análisis de Peligros y Puntos Críticos de Control. Sistemas de gestión de inocuidad; el CAA exige BPM.
- **Reglamento técnico**: norma obligatoria del Estado (vs. una **norma técnica** IRAM/IEC/ISO que es voluntaria salvo que un reglamento la haga obligatoria).
- **Declaración jurada de conformidad (DJC)**: el fabricante declara bajo su responsabilidad que el producto cumple. Modelo "estilo CE".
- **Función de seguridad**: función de la máquina cuya falla aumenta el riesgo (ej.: que al abrir la puerta se corte el movimiento).
- **PL / SIL**: niveles de confiabilidad de funciones de seguridad (ISO 13849-1 usa PL a–e; IEC 62061 usa SIL).

---

## 2. Alimentos (lo que afecta al diseño de la envasadora y a lo que la máquina imprime)

### 2.1 Materiales en contacto con alimentos — CAA Capítulo IV
- **Capítulo IV (arts. 184 a 219)**: "Utensilios, recipientes, envases, envolturas, aparatos y accesorios". Texto consolidado actualizado a 2026.
- **Art. 184**: lista materiales aptos para aparatos/utensilios, incluyendo **acero inoxidable**, acero, hierro fundido, etc.
- **Art. 184 bis (2023)**: envases y utensilios en contacto con alimentos deben cumplir los requisitos aplicables.
- **Art. 195 bis**: incorpora la Res. **GMC (Mercosur) 46/06** sobre envases y equipamientos **metálicos** en contacto con alimentos (límites de impurezas como plomo, arsénico, cadmio; aleaciones de acero inoxidable admitidas). Hay inconsistencia en las fuentes sobre qué resolución lo incorporó **[NV]**.
- Plásticos, elastómeros (juntas, correas), etc. tienen sus propias resoluciones Mercosur **[NV cuáles aplican a correas y piezas plásticas de la máquina]**.
- **Implicancias de diseño (práctica de la industria, no texto literal del CAA)**:
  - Partes en contacto con el producto (tolva, tubo formador, embudo, sinfín) en **AISI 304** (o **316/316L** si hay sal/ácidos/limpieza agresiva).
  - Superficies lisas, soldaduras pulidas, sin rincones; fácil desarme y lavado (**diseño higiénico** — referencia internacional: EHEDG, 3-A, ISO 14159) **[NV aplicabilidad formal en Argentina]**.
  - Lubricantes grado alimentario (NSF H1) en zonas con posible contacto.
  - ANMAT pide, para metales sin recubrimiento, la composición química y la norma (ej. AISI) en sus trámites → **certificado de colada/material del acero** es documentación que conviene guardar por máquina.
- Existe un **Registro Nacional de Establecimientos de envases y materiales en contacto con alimentos** (art. 13). Aplica a fabricantes de envases; para fabricantes de *máquinas* no queda claro **[NV]**.

### 2.2 SENASA vs ANMAT
- Reparto (Decreto 815/1999 y posteriores): SENASA sobre ciertos productos/establecimientos de origen animal y vegetal; ANMAT/INAL sobre alimentos acondicionados para venta al público. Decreto 538/2025 menciona superposición de funciones **[NV cambios recientes]**.
- Trámites RNE/RNPA por **SIFeGA** (sistema federal de gestión de alimentos).
- Para el cliente pyme de la envasadora: la máquina **no se registra**, pero el cliente necesita que su proceso cumpla BPM y que lo que la máquina imprime en el envase sea correcto.

### 2.3 Rotulado y trazabilidad de lotes — CAA Capítulo V
- **Lote** (Cap. V, 2.11): conjunto de artículos del mismo tipo, procesados por el mismo elaborador, en un período determinado y bajo condiciones esencialmente iguales.
- El rótulo obligatorio incluye (Res. GMC 26/03 incorporada al CAA, **[NV numeración exacta]**): denominación, lista de ingredientes, **contenido neto**, identificación de origen, RNE/RNPA, **identificación del lote**, **fecha de duración**, instrucciones de conservación, información nutricional; y desde 2022 los **sellos de advertencia** de la Ley 27.642 (Etiquetado Frontal).
- **Lote y vencimiento** se imprimen normalmente **en la envasadora** (codificador hot-stamp, TTO, inkjet o láser). Se permite tipografía distinta al resto del rótulo (Res. Conj. 41/2003 y 345/2003).
- **Trazabilidad y retiro de productos**: ANMAT tiene un manual de gestión de incidentes y retiro del mercado (recall). Para hacer un recall, la empresa tiene que saber qué lote salió, con qué materia prima, cuándo y a quién → lo que la máquina registre es parte de esa cadena.
- **Contenido neto**: los productos premedidos están sujetos a metrología legal (tolerancias de contenido neto, Res. Mercosur) **[NV norma exacta y autoridad — probablemente Secretaría de Comercio / INTI]**.

---

## 3. Seguridad de máquinas

### 3.1 Marco argentino
1. **Resolución SIC 17/2025** (Secretaría de Industria y Comercio, Min. Economía; BO 25/02/2025, vigente desde 26/02/2025) — **la norma clave para un fabricante**:
   - Aprueba un **reglamento técnico de requisitos esenciales de calidad y seguridad de máquinas y equipos** comercializados en Argentina. Reemplaza el esquema de la Res. 169/2018 (certificados viejos válidos 12 meses de transición).
   - **Anexo II, Apéndice I**: "Máquinas y herramientas eléctricas accionadas por motor" de 50–1.000 VCA / 75–1.500 VCC. Ítem 4: **"Máquinas industriales y equipamiento para máquinas" → norma IEC 60204 (EN 60204)**.
   - **Exclusiones**: entre otras, máquinas con corriente nominal > **63 A** o potencia > **5 kVA**. Una VFFS chica puede quedar dentro o fuera según su potencia instalada (resistencias + servos + compresor si lo trae) → **calcularlo** **[NV interpretación: consultar con un organismo de certificación]**.
   - **Anexo I (requisitos esenciales)**, resumido del texto oficial:
     - Diseño sin riesgo incluso ante **mal uso razonablemente previsible**; informar riesgos residuales.
     - Ante falla de energía: no arrancar intempestivamente, no variar parámetros sin control, no impedir la detención.
     - Sistemas de mando: ubicación segura; **parada de emergencia identificable** y parada normal **con prioridad sobre la puesta en marcha**; activación solo por acción voluntaria.
     - **Resguardos**: robustos, **difíciles de anular**, que **impidan el funcionamiento si no están en posición**, que permitan mantenimiento sin desmontaje total.
     - Mantenimiento: acceso seguro, **separación segura de fuentes de energía** (bloqueo/consignación).
     - Información: advertencias con pictogramas, **sistema de alerta ante falla**, **manual de instrucciones** (uso, montaje, mantenimiento, limpieza).
   - Conformidad: **Declaración Jurada de Conformidad** + procedimiento del Anexo III (marco general Res. SIC 237/2024 y Disposición DNRT 1/2024). Los comercializadores deben tener copia de la DJC. Trámites por TAD. Algunas fuentes mencionan módulos de certificación (1b, 2, 3, 4, 5) **[NV qué módulo aplica a máquinas industriales]**.
2. **Ley 19.587 + Decreto 351/79, Cap. 15 "Máquinas y herramientas" (arts. 103–110)** — obligación del **empleador** (el cliente que usa la máquina):
   - Máquinas seguras; si hay riesgo, no usarlas sin protecciones.
   - Interruptores para detener el motor desde lugar seguro en emergencia.
   - Protección de transmisiones/partes móviles; mantenimiento con la máquina detenida; señalizar máquinas averiadas.
3. **SRT (Superintendencia de Riesgos del Trabajo)**: guías técnicas no obligatorias, p. ej. *Guía 03 "Protecciones en máquinas, equipos y herramientas"* y ficha de atrapamiento: resguardos en transmisiones, paradas de emergencia tipo hongo/cable al alcance. La guía remite a **IRAM 3578** (protecciones de seguridad en maquinaria). Res. SRT 38/1996 (autoevaluación) **[NV vigencia]**.
   - No encontré una "Resolución SRT de seguridad de máquinas" específica tipo Directiva de Máquinas; el instrumento para fabricantes es la Res. SIC 17/2025.

### 3.2 Normas internacionales de referencia (de facto el "manual" de diseño)
- **ISO 12100** — Principios generales de diseño y **evaluación/reducción de riesgos**. Es el punto de partida: identificar peligros → estimar riesgo → reducir con (1) diseño inherentemente seguro, (2) protecciones, (3) información. El entregable es un **documento de análisis de riesgos** por máquina.
- **IEC 60204-1** — Equipamiento eléctrico de máquinas: tablero, puesta a tierra, colores de pulsadores, **categorías de parada (0, 1, 2)**, parada de emergencia (debe ser cat. 0 o 1), protección contra rearranque, documentación eléctrica. Es la norma que **Res. 17/2025 menciona explícitamente**.
- **ISO 13850** — Función de parada de emergencia (hongo rojo sobre fondo amarillo, enclavamiento, rearme manual que no rearranca la máquina).
- **ISO 13849-1** — Partes de los sistemas de mando relativas a la seguridad (**PL**). Relevante: **la parada de emergencia y los interlocks de puertas NO deben depender solo del PLC estándar**; se usan **relés de seguridad** o PLC de seguridad, contactores redundantes, STO en servos.
- **ISO 14119** (interlocks de resguardos), **ISO 14120** (resguardos), **ISO 13857** (distancias de seguridad).
- **Específica de envasadoras**: **EN 415** (serie de seguridad de máquinas de embalaje; EN 415-3 para form-fill-seal) **[NV parte exacta para VFFS]**.
- Puntos críticos típicos en una VFFS: **mordazas (aplastamiento + quemadura + cuchilla)**, correas de arrastre (atrapamiento), portabobina, sinfín/multicabezal (corte). Solución usual: puertas con interlock + STO en servos + descarga de neumática al abrir.
- **Servos Delta ASDA-B3/A3** tienen entrada **STO (Safe Torque Off)** **[NV certificación SIL/PL del modelo elegido]**.

---

## 4. Ciberseguridad industrial — IEC 62443 (lo básico)

- **Qué es**: familia de normas para seguridad de sistemas de automatización y control industrial (IACS). Divide responsabilidades: **dueño del activo** (cliente), **integrador**, **fabricante de componentes/máquina**.
  - **62443-3-2**: análisis de riesgo y partición en **zonas y conductos**.
  - **62443-3-3**: requisitos de sistema y **Security Levels (SL 0–4)** sobre 7 requisitos fundacionales (identificación/autenticación, control de uso, integridad, confidencialidad, flujo restringido de datos, respuesta a eventos, disponibilidad).
  - **62443-4-1**: ciclo de vida de desarrollo seguro (para quien desarrolla el producto/software).
  - **62443-4-2**: requisitos técnicos de **componentes** (PLC, HMI, gateway).
- **Zona** = grupo de activos con requisitos de seguridad comunes (ej. "la máquina": PLC + HMI + servos). **Conducto** = canal de comunicación entre zonas (ej. "máquina ↔ red de la planta"), protegido típicamente con firewall.
- **Por qué importa acá**: Modbus TCP **no tiene autenticación ni cifrado** — cualquiera en la red puede escribir registros (cambiar temperatura, arrancar). Al conectar la máquina, eso es un riesgo real de seguridad física y de calidad.
- **Aplicado a una máquina conectada (mínimos razonables)**:
  - La máquina es **su propia zona**: PLC/HMI en una red interna de la máquina; hacia afuera solo un **gateway/edge** (conducto) con firewall.
  - El edge **lee** del PLC; escrituras solo por un servicio controlado con validación de rangos, y nunca funciones de seguridad (paradas, interlocks) por red.
  - Exponer hacia IT/cloud por **OPC UA con seguridad** o **MQTT sobre TLS**, nunca el puerto 502 del PLC.
  - Acceso remoto (soporte de Chino) por VPN saliente con usuario por persona y registro, no por "abrir puertos" ni VNC expuesto de la HMI.
  - Cambiar contraseñas por defecto (HMI, PLC, router), deshabilitar servicios no usados (FTP/VNC de la HMI), contraseña de programa del PLC.
  - Inventario de versiones de firmware y procedimiento de actualización.
  - Log de auditoría de cambios de receta/parámetros (quién, qué, cuándo).
- **Contexto regulatorio (fuera de Argentina, para exportar)**: la UE tiene el **Cyber Resilience Act** y el nuevo **Reglamento de Máquinas (UE) 2023/1230** (aplicable desde 2027) que incorpora requisitos de protección contra corrupción/ataques **[NV detalle]**. Si Chino quiere exportar, IEC 62443 será el lenguaje común.

---

## 5. Qué se puede resolver o ayudar con software

| Necesidad | Solución de software | Norma que apoya |
|---|---|---|
| **Registro de lote** | Al iniciar producción, el operador (o un lector QR/código de barras) carga lote, producto, materia prima, bobina de film; el sistema guarda inicio/fin, cantidad producida, rechazos, receta usada, operador | CAA Cap. V (lote), BPM, recalls ANMAT |
| **Impresión de lote/vencimiento sin error** | El software calcula el vencimiento desde la receta (vida útil) y **envía el texto al codificador** (en vez de tipeo manual) | CAA rotulado |
| **Verificación de impresión** | Visión (OCR/OCV) que confirma lote/vencimiento legibles; rechazo automático y registro | CAA rotulado, BPM |
| **Etiquetas de bulto / caja** | Generar etiquetas (ZPL para Zebra, GS1-128 con lote y fecha) para cajas/pallets | Trazabilidad aguas abajo |
| **Control de contenido neto** | Registro por bolsa del checkweigher, estadísticas por lote (media, desvío, % fuera de tolerancia), reporte exportable | Metrología legal (contenido neto) |
| **Auditoría de cambios** | Log inmutable de quién cambió qué parámetro (temperatura, peso objetivo) y cuándo; usuarios con roles en HMI/web | BPM, IEC 62443 (FR2/FR6) |
| **Registros de limpieza/mantenimiento** | Checklists digitales (limpieza de partes en contacto, cambio de correas/teflones), con firma del operario | BPM/HACCP, Decreto 351/79 (mantenimiento) |
| **Manual y documentación** | Manual digital accesible desde la HMI/QR en la máquina; registro de alarmas con texto y causa sugerida | Res. 17/2025 Anexo I (información, alerta ante falla) |
| **Expediente técnico de la máquina** | Repositorio por número de serie: análisis de riesgos ISO 12100, esquemas IEC 60204, DJC, certificados de acero, versiones de programa | Res. 17/2025 (DJC), trazabilidad del fabricante |
| **Ciberseguridad** | Edge con firewall, solo lectura, OPC UA/MQTT con TLS, VPN de soporte, inventario de firmware | IEC 62443 |

- **Qué NO resolver con software**: la parada de emergencia y los interlocks de puertas deben ser **hardware de seguridad** (relé/PLC de seguridad, STO). El software solo **monitorea y registra** su estado.

---

## 6. Qué le aporta esto a Chino

- **Obligación concreta hoy**: con la **Res. SIC 17/2025**, como fabricante de máquinas eléctricas debería poder emitir una **Declaración Jurada de Conformidad** basada en **IEC 60204** (y los requisitos esenciales del Anexo I). Conviene armar desde la máquina 1 un **expediente técnico** (análisis de riesgos ISO 12100, esquemas, lista de componentes de seguridad, manual). Verificar con un organismo de certificación si su envasadora entra por potencia (≤5 kVA / ≤63 A).
- **Diseño higiénico + documentación de materiales** (AISI 304/316, certificados) es un argumento de venta frente a pymes que tienen inspecciones de bromatología.
- **Seguridad**: emergencia y puertas por **relé de seguridad + STO** en los servos, no por PLC estándar. Barato y evita el principal riesgo legal (accidente del operario).
- **Diferencial de software alineado a normas**: "**la máquina que te arma el registro de lote**" — lote + vencimiento + contenido neto + auditoría de cambios. Le resuelve al cliente pyme el dolor de BPM/recall y es difícil de copiar para un fabricante puramente mecánico.
- **Conectividad segura por diseño** (IEC 62443 en versión mínima) evita que el diferencial digital se transforme en un riesgo; y prepara el terreno si algún día exporta (UE: Reglamento de Máquinas 2023/1230 + CRA).

---

## 7. Fuentes

**Alimentos**
- CAA Capítulo IV (MAGyP): https://alimentosargentinos.magyp.gob.ar/HomeAlimentos/Marco_Regulatorio/ultimas%20modificaciones/Capitulo_IV.pdf
- CAA Capítulo IV (INTI): https://www.inti.gob.ar/assets/uploads/files/certificaciones/manipuladores-de-alimentos/02/Capitulo_IV.pdf
- ANMAT — Código Alimentario Argentino: https://www.argentina.gob.ar/anmat/codigoalimentario
- ANMAT — Instructivo envases (ENV): https://www.anmat.gob.ar/webanmat/formularios/alimentos/Instructivo_ENV_(May-2014).pdf
- CAA Capítulo V — Rotulación (ANMAT 2022): https://www.argentina.gob.ar/sites/default/files/capitulo_v_rotulacion_actualiz_2022-08.pdf
- CAA Capítulo V (MAGyP): https://alimentosargentinos.magyp.gob.ar/HomeAlimentos/marco-regulatorio/capitulos-caa/CAPITULO%20V%20-%20NORMAS%20PARA%20LA%20ROTULACI%C3%93N%20Y%20PUBLICIDAD%20DE%20LOS%20ALIMENTOS.pdf
- Decreto 815/1999 (Sistema Nacional de Control de Alimentos): https://www.argentina.gob.ar/normativa/nacional/59060/actualizacion
- Decreto 538/2025: https://www.argentina.gob.ar/normativa/nacional/decreto-538-2025-415942/texto
- MAGyP — Trámites para elaboradores: https://alimentosargentinos.magyp.gob.ar/contenido/revista/pdfs/56/56_13_Tramites.pdf
- ANMAT — Preguntas frecuentes (RNE/RNPA en envase): https://www.argentina.gob.ar/node/297040
- ANMAT — Manual de gestión de incidentes y retiro de productos: https://argentina.gob.ar/sites/default/files/anmat_manual_de_recomendaciones_para_la_gestion_de_incidentes_y_retiro_productos_autoridades_sanitarias.pdf

**Seguridad de máquinas**
- Res. SIC 17/2025 — Boletín Oficial: https://www.boletinoficial.gob.ar/detalleAviso/primera/321815/20250225
- Res. 17/2025 — Anexo I (requisitos esenciales): https://servicios.infoleg.gob.ar/infolegInternet/anexos/410000-414999/410053/res17-1.pdf
- Res. 17/2025 — Anexo II (productos, IEC 60204, exclusiones): https://servicios.infoleg.gob.ar/infolegInternet/anexos/410000-414999/410053/res17-2.pdf
- Res. 17/2025 — Anexo III: https://servicios.infoleg.gob.ar/infolegInternet/anexos/410000-414999/410053/res17-3.pdf
- CIRA — resumen Res. 17/2025: https://www.cira.org.ar/es/servicios/normativas-servicios/resolucion/resolucion-17-2025
- Abogados.com.ar — Modificación y derogación de reglamentos técnicos: https://abogados.com.ar/modificacion-y-derogacion-de-reglamentos-tecnicos/36395
- Decreto 351/79 Anexo I: https://www.argentina.gob.ar/normativa/recurso/32030/dto351-1979-anexo1/htm
- SRT — Guía 03 Protecciones en máquinas: https://www.argentina.gob.ar/sites/default/files/03_guia_protecciones_en_maquinas_equipos_y_herramientas_ok.pdf
- SRT — Ficha técnica atrapamiento: https://argentina.gob.ar/sites/default/files/ft_atrapamiento_ago2021.pdf
- Res. SRT 38/1996: https://www.fio.unicen.edu.ar/extension/seh/images/pdf/ley24557/Resol.38-96.pdf

**Ciberseguridad**
- IEC — 62443 background: https://syc-se.iec.ch/?p=4375
- Cisco — ISA/IEC 62443-3-3 white paper: https://cisco.com/c/en/us/products/collateral/security/isaiec-62443-3-3-wp.html
- Cisco — IoT Security Lab: What is IEC 62443: https://cisco.com/c/en/us/td/docs/solutions/Verticals/IoT_Security_Lab/IEC62443_WP.html
- Trend Micro — IEC 62443 security levels in smart factories: https://www.trendmicro.com/en_us/research/20/g/guidelines-related-to-security-in-smart-factories-part-2-system-design-and-security-level-of-iec62443.html

> Las normas ISO 12100, IEC 60204-1, ISO 13849-1, ISO 13850, EN 415 e IEC 62443 son pagas; lo dicho sobre su contenido es conocimiento general del área y resúmenes públicos, no lectura del texto completo **[NV]**.
