# 06 · Modelo de abono mensual (servicio sobre la máquina)

> Investigación hecha el 24/11/2026. **[DATO]** = con fuente al final. **[ESTIMACIÓN]** = cálculo o propuesta nuestra, a validar. Los precios en USD de proveedores extranjeros son de lista y pueden variar.

---

## 1. Modelos de servicio que existen en maquinaria

- **Soporte remoto por incidente o pack:** el cliente compra N intervenciones remotas al año. Por ejemplo, Ishida Sentinel incluye 3 intervenciones remotas por año y por máquina en un pack [DATO].
- **Monitoreo + reportes:** dashboard web 24/7 con rendimiento en vivo y reportes diarios, semanales o mensuales por mail (Ishida Sentinel, reporting pack) [DATO].
- **Service agreement a precio fijo:** mantenimiento planificado, repuestos y soporte por una cuota fija. Syntegon los vende "a costo fijo" combinados con su nube Synexio [DATO].
- **Plataforma de datos en niveles:** Synexio tiene tres (Monitor, Uptime, Empower). Ishida Sentinel 5.0 tiene base, Premium y Premium+, y el nivel más alto incluye servicio monitoreado por Ishida [DATO].
- **Equipment / Machine as a Service (EaaS / MaaS):** el fabricante conserva la máquina y cobra por uso o por suscripción [DATO].
  - Heidelberg cobra por hoja impresa, con máquina, software, consumibles y servicio incluidos.
  - Gebr. Becker cobra una suscripción plana por bombas de vacío, con mantenimiento y monitoreo incluidos.
  - HELLER ofrece pago por pieza.
  - Atlas Copco vende "aire comprimido como servicio".
  - Pearson Packaging aparece citado como caso de EaaS en envasado, sin detalle.
- **[ESTIMACIÓN]** Para una fábrica nueva en Argentina, el EaaS completo (sin vender la máquina) exige mucho capital y financiamiento. **No es para arrancar.** Lo realista es vender la máquina y sumar un abono de servicio.

## 2. Precios reales de plataformas de monitoreo industrial

| Plataforma | Precio | Comentario | Fuente |
|---|---|---|---|
| **MachineMetrics** | Paquete de 10 máquinas + 1 edge: **USD 12.750 por año**, unos **USD 106 por máquina por mes** (cálculo nuestro) | Un revendedor dice "desde USD 150 por máquina por mes" (no verificado) | AWS Marketplace / Techjockey |
| **Guidewheel** | **USD 59 a 99 por sensor por mes**, más **USD 75 por mes** por router LTE | Listado de terceros de 12/2024. Antes publicaban "desde USD 15.000 por año, 10 máquinas" y lo sacaron en 09/2026 | Zoftwarehub / PricingSaaS |
| **Rango de la categoría** | Software: **USD 240 a 1.200 por máquina por año**. Hardware inicial: **USD 500 a 2.000 por máquina** | Estimación publicada por un vendor (Guidewheel) | Guidewheel blog |
| **Ubidots** (plataforma IoT genérica) | **USD 99 por mes**, 50 dispositivos incluidos, **+USD 2** por dispositivo extra | Plan Professional. Industrial y Enterprise a medida | ubidots.com |
| **Ewon Talk2M Pro** (acceso remoto VPN) | **75 € por mes** (Light: 40 € por mes). Distribuidor: USD 899 por año | Hasta 5 conexiones VPN simultáneas, 20 GB por mes | HMS / Gexpro |
| **Weintek EasyAccess 2.0** | Licencia única por HMI: **~USD 104**. Recarga de 5 GB: ~USD 184 | Precio de minorista | Wolf Automation |
| **Syntegon Synexio / Ishida Sentinel** | **No publican precio** | Hay que cotizar | — |

**Lectura:** el mercado internacional cobra entre **USD 20 y 150 por máquina por mes** solo por monitoreo y software, sin incluir soporte humano.

## 3. Costos operativos para nosotros (por máquina)

| Ítem | Costo | Tipo | Fuente |
|---|---|---|---|
| **Router 4G Delta DX-2400L9** (nativo del ecosistema Delta, con DIACloud y sin VPN aparte) | **~540 a 590 €** | Único | Mouser / DigiKey (Europa) |
| **Router 4G Teltonika RUT241** (alternativa genérica, MQTT) | **~USD 199** | Único | Linovision / Klarna |
| **SIM M2M Argentina** (multioperador, Conecty) | **ARS 3.100 por mes (5 MB)**, ARS 3.600 (10 MB), **ARS 13.100 (100 MB)**. Excedente: ARS 97 por MB | Mensual | conecty.com.ar |
| **Servidor cloud** | DigitalOcean desde **USD 4 por mes**. 2 vCPU y 4 GB, ~USD 24 por mes. Hetzner, ~5 a 6 € por mes | Mensual, compartido entre todas las máquinas | digitalocean.com / reviews de Hetzner |
| **Ubidots** (si no armamos plataforma propia) | ~USD 2 por máquina por mes a escala | Mensual | ubidots.com |

**[ESTIMACIÓN] Costo marginal por máquina por mes:**
- Conectividad (100 MB): **~USD 10 a 13**, según el tipo de cambio. Con MQTT y telemetría liviana alcanza 10 a 50 MB, o sea menos.
- Cloud prorrateado (un servidor de ~USD 24 para 50 máquinas): **~USD 0,5**.
- Hardware (gateway de USD 200 a 650 amortizado en 36 meses): **~USD 6 a 18**.
- **Total técnico: ~USD 15 a 30 por máquina por mes.** Lo caro no es la infraestructura, son las **horas de soporte humano**.

## 4. Beneficios

**Para la pyme cliente:**
- Menos horas de máquina parada: diagnóstico remoto en lugar de esperar al técnico.
- Ve su producción real (unidades, paradas, OEE) desde el celular, sin papel.
- Costo de mantenimiento predecible (cuota fija) en lugar de sorpresas.
- Respaldo de parámetros y recetas: si se rompe la HMI, se restaura.
- Soporte en castellano y en su huso horario, algo que ni la china ni la europea le dan.

**Para el fabricante (Chino + fábrica):**
- **Ingreso recurrente.** BCG (2025) relevó ~100 fabricantes de maquinaria y encontró que **solo el 4%** logra monetizar servicios digitales recurrentes [DATO]. Hay mucho espacio y poca competencia.
- **Más venta de repuestos.** Según BCG, los top performers venden **16 puntos porcentuales más** de repuestos [DATO]. Con datos de uso se anticipa el desgaste (mordazas, resistencias, correas) y se ofrece el repuesto antes de que falle.
- **Datos para mejorar el diseño:** qué fallas se repiten, qué parámetros usan los clientes y qué componentes duran menos. Eso vuelve a los planos de Chino.
- **Menos viajes de soporte:** muchos problemas se resuelven con un cambio de parámetro o un video.
- **Lock-in sano:** el cliente con abono vuelve a comprar al mismo fabricante.
- Referencia de mercado: PTC (un vendor) afirma que ~29% de los ingresos de empresas industriales viene de servicios. Es un dato de marketing, sin metodología.

## 5. Propuesta: 3 niveles de abono

> **Todo esta sección es [ESTIMACIÓN].** Los precios salen de comparar con el mercado internacional (USD 20 a 150 por máquina por mes solo en software) y de nuestro costo técnico (~USD 15 a 30). Hay que validarlos con 3 a 5 pymes reales antes de publicarlos.

### Nivel 1 · "Conectada" · ~USD 39 por máquina por mes
- Gateway 4G y SIM incluidos (en comodato).
- Dashboard web y móvil: producción, paradas, contador de ciclos, OEE básico.
- Alertas por WhatsApp o mail (máquina parada, falla, fin de film o material).
- Backup automático de parámetros y recetas del PLC.
- Soporte remoto en horario laboral: hasta 2 incidentes por mes.
- *Margen bruto estimado: USD 10 a 25 por máquina por mes.*

### Nivel 2 · "Soporte" · ~USD 89 por máquina por mes
- Todo lo del Nivel 1.
- Soporte remoto ilimitado en horario extendido y tiempo de respuesta garantizado (por ejemplo, 4 horas hábiles).
- Reporte mensual de OEE con recomendaciones.
- Mantenimiento preventivo por horas o ciclos reales, con aviso de cambio de piezas de desgaste.
- 10% de descuento en repuestos.
- 1 visita preventiva por año (viáticos aparte fuera del radio).

### Nivel 3 · "Planta" · ~USD 149 por máquina por mes (o precio por línea)
- Todo lo del Nivel 2.
- 2 visitas preventivas por año y kit de repuestos de desgaste incluido.
- Integración multi-máquina y de toda la línea (envasadora + cinta + horno).
- Asistente con IA sobre el historial de la máquina y los manuales (estilo chatbot de Sentinel 5.0).
- Exportación de datos o API al ERP del cliente.
- Soporte prioritario 7x24 para paradas críticas.

**Reglas comerciales sugeridas [ESTIMACIÓN]:**
- **Primeros 6 o 12 meses del Nivel 1 incluidos en el precio de la máquina.** Así el cliente prueba y nosotros juntamos datos desde el día uno.
- Descuento por pago anual (ej. 2 meses gratis).
- Precio en USD y facturación en pesos al tipo de cambio del día. Si no, la inflación se come el margen.
- Sin abono, la máquina funciona igual: el abono **suma**, nunca bloquea. Eso genera confianza.

## Qué le aporta esto a Chino

- **Transforma una venta única en una relación.** Con 50 máquinas en Nivel 1 hay ~USD 1.950 por mes de ingreso recurrente **[ESTIMACIÓN]**, y casi todo es margen, porque la infraestructura cuesta poco.
- **Hay poca competencia:** según BCG, solo 4% de los fabricantes lo hace bien. Los nacionales no lo ofrecen y los europeos lo cobran caro y en inglés.
- **Datos de campo para sus planos:** sabe qué falla, dónde y cuándo, y mejora el diseño de la próxima versión.
- **Encaja con su PLC:** el Delta AS320T-B tiene camino directo al router Delta DX-2400L9 con DIACloud, o a un gateway Teltonika vía Modbus TCP y MQTT. No hace falta cambiar el hardware de control.
- **Arranque barato:** con ~USD 200 a 650 por máquina en hardware y ~USD 25 por mes de cloud total se monta un piloto. El software lo construye el developer.
- **Propuesta para la charla:** "Cada máquina sale conectada de fábrica. Los primeros 12 meses de monitoreo van incluidos, y después el cliente elige el nivel."

## Fuentes

- Ishida Sentinel: https://www.ishidaeurope.com/en/products/reporting-software/reporting-software/ishida-sentinel
- Ishida Sentinel 5: https://ishidaeurope.com/en/landing-pages/sentinel-5
- Ishida Sentinel, packs (brochure): https://www.ishidaeurope.com/media/3d4lzqlo/ishida_sentinel-50-brochure_v6-0425.pdf
- Syntegon Synexio: https://www.syntegon.com/synexio-on-demand
- Syntegon digital solutions: https://www.syntegon.com/services/digital-solutions
- Synexio, paquetes (Packaging South Asia): https://packagingsouthasia.com/2023/09/going-digital-for-greater-efficiency/
- EaaS, ejemplos (Automation World): https://www.automationworld.com/factory/plant-maintenance/article/22353256/machines-as-a-service-this-trend-is-just-getting-started
- EaaS (IoT Analytics): https://iot-analytics.com/entering-the-decade-of-equipment-as-a-service/
- Bain, Machinery as a Service: https://www.bain.com/insights/machinery-as-a-service-global-machinery-and-equipment-report-2022/
- Capgemini, EaaS: https://www.capgemini.com/us-en/insights/expert-perspectives/equipment-as-a-service-can-be-a-winning-solution-in-manufacturing-but-only-if-its-financially-viable/
- BCG 2025, aftermarket: https://www.bcg.com/ja-jp/publications/2025/aftermarket-services-drive-growth-for-industrial-manufacturers
- McKinsey, aftermarket: https://www.mckinsey.com/industries/industrials-and-electronics/our-insights/industrial-aftermarket-services-growing-the-core
- PTC, servitization: https://www.ptc.com/en/industries/industrials/campaigns/servitization
- MachineMetrics pricing: https://machinemetrics.com/pricing
- MachineMetrics en AWS Marketplace: https://aws.amazon.com/marketplace/pp/prodview-nnc6qc5mcufmy
- MachineMetrics (Techjockey): https://www.techjockey.com/ae/detail/machinemetrics
- Guidewheel (Zoftwarehub): https://zoftwarehub.com/en-ae/products/guidewheel/pricing
- Guidewheel (PricingSaaS): https://pricingsaas.com/companies/guidewheel
- Guidewheel, costo de la categoría: https://www.guidewheel.com/blog/machine-monitoring-software-cost
- Ubidots pricing: https://ubidots.com/pricing
- Ewon Talk2M Pro: https://hms-networks.com/software-and-tools/talk2m/talk2m-pro
- Talk2M Pro (Gexpro): https://www.gexpro.com/p/801564/ewon/talk2m-pro-yearly-fee/tm50041
- Weintek EasyAccess 2.0: https://wolfautomation.com/easyaccess-2-0-activation-card
- Delta DX-2400L9 (Delta EMEA): https://www.delta-emea.com/en-GB/products/IIoT-Routers/DX-2400L9
- Delta DX-2400L9 (DigiKey): https://www.digikey.at/en/products/detail/delta-electronics-industrial-automation/DX-2400L9/23028606
- Delta DX-2400L9 (Mouser): https://www.mouser.in/new/delta-electronics/delta-dx-2400l9-routers/
- Teltonika RUT241: https://linovision.com/products/teltonika-rut241-industrial-4g-lte-cellular-wi-fi-router
- Conecty, planes M2M: https://conecty.com.ar/solution/m2m-nacional/
- Conecty, tienda: https://shop.conecty.io/tienda/sims/sim-m2m-movistar/
- DigitalOcean pricing: https://digitalocean.com/pricing/droplets
- Hetzner, review 2026: https://betterstack.com/community/guides/web-servers/hetzner-cloud-review/
