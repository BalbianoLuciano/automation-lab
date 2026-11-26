# 05 — IA aplicada a las máquinas de Chino

> Para cada caso: qué es, qué tan realista para una pyme, qué pide (datos/sensores/hardware), costo orientativo y casos reales.
> Investigado: noviembre 2026. Lo marcado **(estimación)** es cálculo propio. Muchas cifras de ahorro son **de proveedores**, no auditadas: se aclaran.

## Glosario rápido

- **Edge AI**: correr el modelo en un equipo al lado de la máquina, no en la nube (latencia baja, funciona sin internet).
- **TOPS**: billones de operaciones por segundo. Medida grosera de potencia de un acelerador de IA.
- **YOLO**: familia de modelos de detección de objetos en imágenes, rápidos y fáciles de entrenar (Ultralytics).
- **FFT**: transformada que pasa una señal de vibración a frecuencias; cada falla mecánica "suena" en frecuencias típicas.
- **MCSA**: Motor Current Signature Analysis. Detectar fallas mirando la forma de la corriente del motor.
- **RAG**: Retrieval-Augmented Generation. El LLM responde buscando primero en tus documentos.
- **Detección de anomalías**: el modelo aprende "lo normal" y avisa cuando algo se aparta. No necesita ejemplos de fallas.

---

## 1. Mantenimiento predictivo

**Qué es**: anticipar fallas (rodamientos, motores, correas, resistencias) midiendo vibración, corriente y temperatura, en vez de esperar que se rompa o cambiar piezas por calendario.

**Qué tan realista para una pyme**:
- **Nivel 1 (muy realista)**: umbrales + tendencias. "La corriente del motor de la cinta subió 15% en 3 semanas", "la temperatura del rodamiento pasa de X". Es estadística simple, no ML.
- **Nivel 2 (realista con datos)**: detección de anomalías sobre vibración/corriente (Isolation Forest, autoencoders). Pide meses de datos "normales".
- **Nivel 3 (difícil)**: predecir "le quedan 20 días" (RUL). Pide historial de fallas reales, que una pyme casi nunca tiene.

**Qué pide**:
- **Datos del PLC que ya existen**: horas de marcha, arranques, alarmas, corriente si hay variador (los variadores Delta exponen corriente por Modbus).
- **Vibración**: acelerómetros MEMS en motores/reductores. Inalámbricos comerciales (ABB, Advantech/LoRaWAN) o DIY con ESP32 + acelerómetro.
- **Corriente**: transformadores de corriente (pinzas CT) en los cables del motor, sin intervenir el tablero.
- **Temperatura**: PT100/termocupla en rodamientos y en resistencias de hornos.

**Costo orientativo (estimación)**:
- Usar datos del PLC/variador: USD 0 extra.
- Sensor de vibración DIY (ESP32 + MEMS): USD 20–50 por punto.
- Sensor inalámbrico industrial: USD 150–500 por punto.
- Pinza CT + acondicionamiento: USD 20–80 por motor.

**Casos reales**:
- Tec. de Monterrey: arquitectura de bajo costo para pymes en un torno CNC con temperatura y vibración (académico).
- TI: monitor inalámbrico de motor con FFT a bordo y >10 años de batería (diseño de referencia).
- ABB WiMon 100: piloto en equipos no críticos, captura cada 6 h + FFT.
- Siemens Senseye + Copilot: dicen ~25% menos tiempo de mantenimiento reactivo (cifra de proveedor, pilotos).

**Lo que conviene saber**: el valor real para Chino al principio no es la IA, es **tener los contadores y tendencias** de cada máquina vendida.

---

## 2. Visión artificial en envasado

**Qué es**: una cámara arriba de la línea que, con un modelo tipo YOLO, revisa cada paquete: **sellado** (arrugas, producto en la soldadura), **nivel** de llenado, **etiqueta** (presente, derecha, legible), **presencia** (falta tapa, falta producto). Si falla, el PLC acciona un expulsor.

**Qué tan realista para una pyme**: **bastante**, si se acota bien el problema.
- Presencia, etiqueta torcida, nivel en envase transparente: realista con poco dataset (cientos de imágenes).
- Defectos finos de sellado: difícil. Los sellos transparentes son complicados incluso para proveedores comerciales; a veces hace falta iluminación especial o cámara térmica.
- La parte difícil **no es el modelo**: es la iluminación, el disparo sincronizado (trigger desde el PLC/sensor), y la velocidad de la línea (motion blur).

**Qué pide**:
- Cámara: USB/industrial global shutter idealmente (USD 50–400). Para prototipo, cámara Raspberry o webcam.
- Iluminación controlada (domo o barra LED) y cerramiento para evitar luz ambiente.
- Sensor fotoeléctrico de disparo.
- Edge AI:
  - **Raspberry Pi 5 + AI HAT+ (Hailo-8L, 13 TOPS)**: USD 70 el HAT; YOLOv8s ~60–80 FPS (benchmark secundario). Versión 26 TOPS: USD 110.
  - **Jetson Orin Nano Super**: USD 249 (lista), 67 TOPS INT8. Más flexible para entrenar/probar.
  - Pi 5 sola sin acelerador: YOLOv8n ~5–8 FPS (solo para pruebas).
- Dataset propio etiquetado (Roboflow, CVAT, Label Studio).

**Costo orientativo por estación (estimación)**: USD 300–800 en hardware (Pi 5 + Hailo + cámara + luz + gabinete) contra sistemas comerciales que suelen costar varios miles de USD.

**Casos reales**:
- Paper (PMC, 2025): YOLOv5 mejorado para defectos de packaging de alimentos (rayas, abolladuras, color).
- YOLOv7 en tapas de frascos: buena precisión global pero débil en la clase "dañada" por desbalance de datos (lección: faltan ejemplos de fallas).
- Caso BOA/Automate: sellos de foil con imagen térmica + deep learning, bajó rechazos.
- Mettler-Toledo / Roboflow: inspección de sellado comercial.

---

## 3. Optimización de hornos

**Qué es**: usar datos de temperatura por zona, tiempo, carga y consumo de gas/electricidad para:
- **Recetas**: guardar y repetir perfiles por producto (esto es software, no IA).
- **Curvas**: detectar desvíos de la curva ideal y alertar.
- **Energía**: reducir tiempos muertos (horno encendido sin carga), precalentamientos excesivos, ajustar setpoints.
- **Control avanzado**: ajustar PID o modelo predictivo (MPC) por zona.

**Qué tan realista para una pyme**:
- Recetas + registro de curvas + alertas: **muy realista**, alto valor (calidad repetible, trazabilidad para auditorías de alimentos).
- Detectar horno ocioso y calcular gas por lote: **realista**, ahorro directo.
- ML que ajusta setpoints solo: **poco realista al principio**; riesgo sobre la calidad del producto. Mejor "recomendar" que "actuar".

**Qué pide**: termocuplas por zona (ya existen en el PLC), medidor de gas o energía con salida de pulsos/Modbus, registro de lotes (qué producto, cuánto).

**Costo orientativo (estimación)**: medidor de energía Modbus USD 100–300; caudalímetro de gas con pulsos USD 300–1.500.

**Casos reales / cifras**:
- AMF Bakery: hasta 20% de ahorro de gas con ML (proveedor, sin metodología).
- GEA Imaforni: hornos modernos 10–30% menos energía (proveedor, incluye diseño del horno).
- PrO4Bake (UE): proyección de 10% si se adopta masivamente.
- OST (Suiza): cámara + datos de energía para detectar tiempos ociosos.
- Conclusión: 10–20% es la cifra "típica" de proveedores, **no auditada**.

---

## 4. Control de aireación de granos

**Qué es**: prender los ventiladores del silo solo cuando el aire exterior sirve para **enfriar** (y no humedecer) el grano. Se basa en el **equilibrio higroscópico**: cada combinación temperatura/humedad relativa del aire lleva al grano a una "humedad de equilibrio". Si el aire es muy húmedo, airear moja el grano.

**Datos clave (INTA)**:
- Humedad de almacenamiento segura = la que está en equilibrio con ~67% de HR; por debajo de ~70% de HR de equilibrio los hongos no se desarrollan.
- Ejemplos (app INTA-ISU): maíz 14,0% a 67% HR y 16,6% a 85%; soja 12,8% y 16,5%.
- Aireación reforzada: >0,2 m³/min por tonelada; puede bajar ~1 punto de humedad al mes en maíz de hasta 16,5%.

**Qué tan realista para una pyme**: **el más realista de todos**. Es control con reglas + física conocida, no hace falta ML.
- Nivel 1: termostato (airear cuando T exterior < T grano – X °C). INTA Balcarce desarrolló un controlador de bajo costo que hace eso: elige automáticamente las horas con temperatura bajo el valor programado.
- Nivel 2: agregar HR exterior y calcular humedad de equilibrio (ecuaciones tipo Chung-Pfost/Henderson modificada, que usa la app INTA-ISU) → no airear si humedece.
- Nivel 3: pronóstico del clima (la app INTA estima horas de aireación disponibles a 3–10 días) + termometría en el silo + CO₂.
- Nivel 4 (ML): predecir focos de calentamiento con termometría y CO₂ histórico.

**Qué pide**:
- Sensor exterior de T y HR (USD 30–150, Modbus RS485).
- **Termometría**: cables con sensores colgados del techo a la base del silo; cantidad según diámetro (Grain Systems, CMC; precios a cotizar).
- Opcional: sensor de CO₂ (INTA/IEA Rosario: CO2NTROL usa la aireación existente para detectar deterioro).
- PLC (el AS320T-B alcanza) comandando contactores de ventiladores.

**Costo orientativo (estimación)**: controlador básico + sensor exterior: USD 300–800 sobre el tablero; termometría completa: varios miles de USD por silo (cotizar).

**Casos reales**: INTA Balcarce (controlador económico + app con Iowa State), CO2NTROL (INTA–IEA Rosario), Wiagro (sensores en silo bolsa), Grain Systems/CMC (termometría + aireación integradas).

---

## 5. Asistente LLM para operarios y técnicos

**Qué es**: un chat (web/WhatsApp/tablet en la máquina) que responde "¿qué significa la alarma E12?", "¿cómo cambio la mordaza de sellado?", "¿esto ya pasó antes?" usando **RAG** sobre manuales, planos, listas de repuestos e historial de fallas.

**Ventajas**:
- Chino y la fábrica escalan soporte sin estar 24/7 al teléfono.
- El operario consulta en lenguaje natural, en español, a cualquier hora.
- El historial de fallas se convierte en conocimiento acumulado.
- Es lo que el dev ya sabe hacer (backend + LLM + UX): **esfuerzo bajo, se puede prototipar ya**.

**Riesgos**:
- **Alucinaciones**: inventar un procedimiento o un valor (presión, temperatura, torque). En una máquina con cuchillas y resistencias es un riesgo de seguridad.
  - Mitigación: responder **siempre citando la fuente** (página del manual), que el operario pueda verificar. El estudio de Fincantieri (100+ manuales) llega a la misma conclusión.
  - "No sé" como respuesta válida; prohibir inventar valores fuera del manual.
- **Seguridad**: nunca que el asistente opere la máquina. Solo informa.
- **Calidad de los manuales**: si no hay buen manual, el asistente no sirve. Es una excusa para escribirlos bien.
- **Conectividad**: sin internet no funciona (salvo modelo local, que es mucho más caro y peor).
- **Responsabilidad legal**: disclaimers + procedimientos de bloqueo (LOTO) siempre del manual oficial.

**Costo orientativo (estimación)**: uso de API de LLM para decenas de consultas diarias: USD 5–30/mes; desarrollo de MVP: 2–4 semanas de un dev.

**Casos reales**: Siemens Industrial Copilot para mantenimiento (~25% menos tiempo reactivo, cifra de proveedor); estudio arXiv 2026 con Fincantieri sobre RAG para manuales de troubleshooting; tesis Chalmers sobre asistente con sensores + logs + manuales.

---

## Ranking: qué haría primero

| # | Iniciativa | Impacto | Esfuerzo | Por qué |
|---|---|---|---|---|
| 1 | **Monitoreo + tendencias + alertas** (base para todo, nota 04) | Alto | Bajo | Sin datos no hay IA. Se vende ya. |
| 2 | **Control de aireación por reglas + humedad de equilibrio** | Alto | Bajo-medio | Física conocida, referencias INTA, ahorro de energía y pérdida de grano. Sin ML. |
| 3 | **Asistente LLM con RAG sobre manuales** | Medio-alto | Bajo | El dev lo hace en semanas, se puede demostrar en la venta, mejora postventa. |
| 4 | **Recetas + curvas + energía en hornos** | Medio-alto | Medio | Calidad repetible y trazabilidad; ahorro medible. |
| 5 | **Visión artificial en envasadora** (empezar por presencia/etiqueta/nivel) | Alto | Medio-alto | Gran diferencial, pero iluminación y sincronismo llevan tiempo. Prototipo con Pi 5 + Hailo. |
| 6 | **Mantenimiento predictivo con ML** | Alto a largo plazo | Alto | Necesita meses de datos de varias máquinas. Empezar recolectando desde el día 1. |

## Qué le aporta esto a Chino

- **Argumentos de venta concretos**: "aireación automática que no moja el grano", "envasadora que descarta paquetes mal sellados", "horno que guarda recetas y te dice cuánto gas gastó por lote".
- **Postventa escalable**: asistente que responde dudas y técnico que llega sabiendo qué falla.
- **Barreras de entrada**: la competencia local que fabrica "solo fierro" no ofrece esto.
- **Producto escalonado**: versión base (PLC) + opcionales (monitoreo, visión, asistente) que suben el ticket y generan abono.
- **Empezar barato**: casi todo el nivel 1 se hace con el PLC actual + un gateway de USD 150–300.

## Fuentes

- Mantenimiento predictivo bajo costo pymes (Tec. Monterrey): https://core.ac.uk/download/190345481.pdf
- TI monitor inalámbrico de motor: https://ti.com/lit/pdf/sszt948
- ABB WiMon piloto: https://new.abb.com/news/detail/25556/transforming-condition-monitoring-of-rotating-machines
- ESP32 + MEMS (Sensors 2025): https://mdpi-res.com/d_attachment/sensors/sensors-25-06610/article_deploy/sensors-25-06610.pdf
- Advantech caso LoRaWAN: https://www.advantech.com/en-sg/resources/case-study/tct-integral-wise-iot-predictive-maintenance
- MCSA (WIKA): https://blog.wika.com/us/applications/using-motor-sensors-for-predictive-maintenance-helps-businesses-save-time-and-money/
- Siemens Industrial Copilot mantenimiento: https://press.siemens.com/global/en/pressrelease/siemens-expands-industrial-copilot-new-generative-ai-powered-maintenance-offering
- RAG manuales troubleshooting (arXiv 2026): https://arxiv.org/html/2601.08706v1
- Tesis Chalmers asistente mantenimiento: https://odr.chalmers.se/bitstreams/5f979de4-1e26-4ba0-8d5d-ffa7fad5b260/download
- YOLOv5 packaging alimentos: https://www.ncbi.nlm.nih.gov/pmc/articles/PMC12021230/
- Caso foil seal (Automate): https://www.automate.org/case-studies/tamper-evident-packaging/boa
- Roboflow seal inspection: https://roboflow.com/ai/seal-integrity-inspection
- Mettler-Toledo seal check: https://mt.com/hk/en/home/products/Product-Inspection_1/CI_Vision/seal-check.html
- Raspberry Pi AI HAT+: https://www.raspberrypi.com/products/ai-hat/
- Benchmarks Hailo en Pi 5: https://raspberry.tips/en/raspberrypi-tutorials/raspberry-pi-ai-hat-plus-ai-acceleration
- Jetson Orin Nano Super: https://blogs.nvidia.com/blog/jetson-generative-ai-supercomputer
- AMF ahorro gas: https://amfbakery.com/sustainable-oven-service-2
- GEA hornos eficientes: https://gea.com/es/stories/energy-saving-ovens
- IA en panaderías (PrO4Bake): https://mag.euroshop.de/en/?p=25405
- OST cámara + energía hornos: https://www.ost.ch/fileadmin/dateiliste/97_daten/abstracts/af2049e2-1a5f-4b8a-84b0-7b9740e1ff6c.pdf
- INTA controlador económico de aireación: https://www.engormix.com/agricultura/almacenaje-maiz/diseno-desarrollo-controlador-economico_a29358/
- INTA guía maíz húmedo: https://intainforma.inta.gob.ar/wp-content/uploads/Guia-para-el-manejo-de-maiz-humedo-en-postcosecha.pdf
- INTA poscosecha alta humedad: https://intainforma.inta.gob.ar/wp-content/uploads/2019/02/0018_Poscosecha_en_campañas_con_alta_humedad.pdf
- CO2NTROL (INTA / IEA Rosario): https://argentina.gob.ar/sites/default/files/2024/10/co2ntrol_.pdf
- App aireación INTA-ISU: https://www.agronewscastillayleon.com/desarrollan-una-app-para-la-aireacion-y-el-almacenamiento-de-granos
- Termometría Grain Systems: https://www.grainsystems.com/sp_SA/productos/gestion-de-granos/termometria.html
- CMC Grain Ranger: https://integration.aggrowth.com/globalassets/brands-section/cmc/support-resources/cmc_grain_ranger_spanish_rev_a00.pdf
- Monitoreo y control (MAGyP): https://alimentosargentinos.magyp.gob.ar/contenido/valorAr/redypa/Guia_calidad/pdfs/Monitoreo_y_Control.pdf
