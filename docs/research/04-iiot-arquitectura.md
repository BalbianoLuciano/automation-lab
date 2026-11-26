# 04 — Arquitectura IIoT: del PLC a la nube

> Contexto: envasadoras, hornos, cintas y aireación de granos con PLC Delta AS320T-B (Modbus TCP), clientes pymes con internet inestable. Todo se prototipa simulado.
> Investigado: noviembre 2026. Los precios en USD son de listas de distribuidores del exterior; en Argentina hay que sumar importación/impuestos (en general bastante más). Lo marcado **(estimación)** es cálculo propio.

## Glosario rápido

> 💻 = analogía con el stack web (React, Node, TS, git, Docker, AWS). Es para entender la idea, no una equivalencia exacta. Más en `09-plc-para-devs.md`.

- **IIoT**: Industrial Internet of Things. Conectar máquinas industriales para sacar datos y, a veces, controlarlas a distancia.
  - 💻 Observabilidad (como Datadog o Sentry), pero de máquinas físicas en vez de servidores: métricas, logs y alertas de una envasadora.
- **Edge / edge gateway**: computadora chica que vive *al lado de la máquina*. Lee el PLC, guarda datos local y los manda a la nube.
  - 💻 Un **sidecar o agente** (como el agente de Datadog en un server), o un **BFF**: está pegado a la fuente, traduce y reenvía. Si conocés Cloudflare Workers o Lambda@Edge, es la misma idea de "procesar cerca del origen".
- **Modbus TCP**: protocolo viejo y simple para leer/escribir registros del PLC por Ethernet. Sin cifrado ni autenticación.
  - 💻 Una API REST minimalista que devuelve números pelados de un array, sin JSON, sin docs y sin auth. Como si `GET /registers/1002` devolviera `1855` y tuvieras que saber que es la temperatura ×10.
- **MQTT**: protocolo de mensajería liviano tipo "publicar/suscribir". El **broker** es el servidor que recibe y reparte mensajes.
  - 💻 Pub/sub como **Redis Pub/Sub, AWS SNS o canales de Socket.IO**:
    - El **topic** (`planta1/envasadora1/temp`) es como una ruta o nombre de canal.
    - El **broker** es el servidor de mensajería, como Redis o RabbitMQ.
    - Publicar es `io.emit()` y suscribirse es `socket.on()`.
- **QoS (MQTT)**: nivel de garantía de entrega. QoS 0 = "tiro y me olvido", QoS 1 = "al menos una vez" (puede duplicar).
  - 💻 Las garantías de una cola:
    - QoS 0 es un `fetch` sin retry.
    - QoS 1 es **SQS**: at-least-once, así que tu consumidor tiene que ser **idempotente**.
    - QoS 2 es exactly-once: más caro y casi nunca necesario.
- **Store-and-forward**: guardar datos localmente cuando no hay internet y reenviarlos cuando vuelve.
  - 💻 Una **PWA offline-first** (service worker + IndexedDB que sincroniza al volver la red) o el **outbox pattern** de backend.
- **Base de series temporales (TSDB)**: base optimizada para datos "valor + timestamp" (InfluxDB, TimescaleDB).
  - 💻 Una tabla de logs o métricas gigante, siempre consultada por rango de fechas. **TimescaleDB es Postgres** con una extensión, así que usás SQL normal. InfluxDB es más como un Prometheus con su propio lenguaje.

## Arquitectura de referencia

```
[PLC Delta AS320T-B] --Modbus TCP--> [Edge gateway] --MQTT/TLS--> [Broker en nube] --> [TSDB] --> [Dashboards / alertas / ML]
                                         |
                                  buffer local (disco)
                                  dashboard local (opcional)
                                  router 4G de respaldo
```

- **Capa 1 — Máquina**: el PLC hace el control. *Nunca* se mueve el control a la nube; la nube solo observa (y como mucho cambia parámetros con mucho cuidado).
- **Capa 2 — Edge**: lee registros Modbus cada 1–5 s, normaliza (nombres, unidades), guarda en disco y publica por MQTT.
- **Capa 3 — Transporte**: MQTT sobre TLS, saliente desde la planta (no hace falta abrir puertos en el router del cliente).
- **Capa 4 — Nube**: broker + base de datos + dashboards + alertas. Acá después entra la IA (ver nota 05).

💻 **La misma arquitectura en términos web:**
```
PLC                →  backend crítico en tiempo real (no se toca desde la nube)
Edge               →  agente/BFF: lee, normaliza (como un serializer/DTO), guarda local y publica
MQTT/TLS saliente  →  un WebSocket que el cliente abre hacia afuera: no hace falta abrir puertos, igual que un navegador no necesita IP pública
Broker + TSDB      →  Redis/RabbitMQ + Postgres
Dashboards/alertas →  tu frontend + webhooks (Telegram, mail)
```
- "Nunca mover el control a la nube" es como **no validar pagos en el frontend**: la lógica crítica vive donde no depende de la red.
- "Normalizar" en el edge es mapear `D1002 = 1855` a `{ "tempMordaza": 185.5, "unit": "°C" }`, como un DTO o un `zod.transform`.

## Hardware de edge: opciones

| Opción | Qué es | Precio orientativo | Pros | Contras |
|---|---|---|---|---|
| **Raspberry Pi 5 + gabinete DIN + fuente 24V** | Pi común en caja | ~USD 100–150 (estimación) | Barato, mucha comunidad, ideal para prototipo | No es industrial: SD que se corrompe, temperatura, ruido eléctrico |
| **Revolution Pi Connect 4 (Kunbus)** | Raspberry Pi CM4 industrial, riel DIN, 24V | ~£410–425 (RS UK), USD 509–600 (DigiKey) | Industrial, eMMC, Modbus TCP/RTU nativo, Node-RED soportado | Caro en Argentina, importación |
| **PC industrial fanless (x86)** | Mini PC sin ventilador | USD 300–800 (estimación) | Potencia para visión artificial, Docker sin drama | Más caro, más consumo |
| **Router industrial 4G con edge (Teltonika RUT956)** | Router con Modbus y MQTT incorporado | ~€270–300 en Europa; ~ARS 812.000 en Mercado Libre (fecha incierta) | Una sola caja: 4G + Modbus→MQTT + VPN | Lógica limitada, poco almacenamiento |
| **Delta DX-2300 + DIACloud** | Router "cloud" de Delta para sus PLC | Consultar | Integrado con ISPSoft para programar remoto | Atado a Delta, algunos distribuidores lo dan como discontinuado |

- 💻 Cómo leer la tabla:
  - **Raspberry Pi** es tu VPS barato de USD 5: anda, pero sin garantías.
  - **RevPi o PC industrial** es una instancia "production grade", con hardware que aguanta calor, vibración y ruido eléctrico.
  - **eMMC vs SD**: la SD es como guardar la base en `/tmp`: se corrompe con cortes de luz. La eMMC es un disco de verdad.
  - **Riel DIN** es el estándar donde se atornilla todo en el tablero eléctrico, como un rack de servidores en miniatura.
  - **Teltonika** es un "router + Lambda" en una caja: conectividad y un poco de lógica.
  - **DIACloud** es un PaaS del propio fabricante (como Vercel para Next): cómodo, pero te ata a Delta.
- **Para el lab (todo simulado)**: una Raspberry Pi o directamente la notebook con Docker.
- **Para la primera máquina real**: Pi industrial (RevPi) o Teltonika RUT956 si el cliente no tiene internet.

## Software: opciones por capa

### En el edge
- **Node-RED**: programación visual por "flujos". Tiene nodos Modbus, MQTT, InfluxDB. Muy rápido para prototipar; mantenible si se versionan los flows en git.
- **Telegraf** (InfluxData): agente que lee de Modbus/MQTT y escribe en InfluxDB. Configuración por archivo, muy liviano. Tiene buffer en memoria por salida (`metric_buffer_limit`), pero **se pierde si se reinicia** el equipo.
- **Mosquitto local + bridge**: broker MQTT en el edge que reenvía a la nube. Puede encolar mensajes con `persistence true`, `cleansession false`, QoS 1 y `max_queued_messages`/`max_queued_bytes`. Ojo: por defecto encola solo 100 mensajes y **no encola QoS 0**.
- **NanoMQ / EMQX Edge**: broker liviano de EMQ para el edge. EMQX Edge (comercial) documenta store-and-forward a SQLite.
- **AWS IoT Greengrass v2**: runtime de AWS en el edge. Su *stream manager* hace buffer persistente y sube a S3/Kinesis/SiteWise. USD 0,16 por dispositivo activo/mes (3 gratis el primer año).
- 💻 Las herramientas del edge, traducidas:
  - **Node-RED** es **n8n o Zapier** que corre local. Los "flows" son JSON y se versionan en git.
  - **Telegraf** es un agente de métricas, como Fluentd o el agente de Datadog: lee de A y escribe en B por config. ⚠️ Su buffer es memoria RAM, como guardar la cola en una variable: se pierde al reiniciar.
  - **Mosquitto local + bridge** es un Redis local que replica a uno remoto. `persistence true` es como activar AOF en Redis para no perder la cola si se reinicia.
  - **Greengrass** es como "Lambda + ECS que corre en tu Raspberry", manejado desde la consola de AWS. Te va a sonar del curso.

### En la nube
- **Broker**: EMQX (open source), Mosquitto, HiveMQ, o **AWS IoT Core** (gestionado).
- **Base de datos**: InfluxDB (series temporales nativo) o **TimescaleDB** (Postgres con superpoderes; ventaja: es SQL y un dev ya lo sabe).
- **Dashboards**: **Grafana** (el estándar). Grafana Cloud free: 10k series activas, 14 días de retención.
- **Plataformas "todo en uno"**:
  - **ThingsBoard**: open source (Community Edition) con dispositivos, dashboards, alarmas y reglas. Cloud: Free (5 dispositivos), Prototype USD 49/mes (50 dispositivos). ThingsBoard Edge funciona sin conexión y sincroniza.
  - **AWS IoT SiteWise**: modelado de activos industriales + SiteWise Edge. Potente pero cobra por cada dato ingerido, cómputo y exportación; curva de aprendizaje alta.
- 💻 Las opciones de nube, traducidas:
  - **EMQX / Mosquitto / HiveMQ** son RabbitMQ o Redis: elegís el broker como elegís la cola.
  - **AWS IoT Core** es la versión managed, como SQS/SNS pero para dispositivos, con un certificado por máquina (mTLS).
  - **Grafana** es Datadog, Kibana o Metabase: dashboards conectados a la base.
  - **ThingsBoard** es un **Supabase o Firebase del IoT**: auth, dispositivos, dashboards y reglas ya hechos. Rápido para arrancar, pero menos control.
  - **SiteWise** es como DynamoDB con un modelo de datos industrial: potente, con un pricing por uso que hay que mirar de cerca.

### AWS IoT Core: precios de referencia
- Conectividad: USD 0,08 por millón de minutos de conexión.
- Mensajes: ~USD 1 por millón (medido en bloques de 5 KB).
- Reglas: ~USD 0,15 por millón de reglas + 0,15 por millón de acciones.
- Free tier 12 meses: 2,25 M minutos, 500 k mensajes.

## Store-and-forward: cómo no perder datos

- **Regla de oro**: el edge escribe **primero en disco**, después intenta mandar.
- Patrones posibles (de más simple a más robusto):
  1. Mosquitto local con persistencia + bridge QoS 1 a la nube.
  2. Node-RED → SQLite/archivo local → worker que reenvía y marca lo enviado.
  3. InfluxDB/TimescaleDB **local** en el edge + replicación a la nube (el cliente ve sus datos aunque no haya internet).
  4. Greengrass stream manager o EMQX Edge (buffer persistente "de fábrica").
- **Dimensionar el buffer (estimación)**: 50 variables × 1 muestra/s × ~50 bytes ≈ 216 MB/día sin comprimir. Una SSD de 32 GB aguanta semanas de corte. Con agregación a 10 s baja 10×.
- **Timestamp en el origen**: el dato tiene que llevar la hora de cuando se midió, no de cuando llegó. Sincronizar hora con NTP (y GPS del router 4G si no hay internet).
- 💻 Los 4 patrones, traducidos:
  1. Una cola local que replica: Redis con AOF + réplica.
  2. El **outbox pattern** de libro: insert en SQLite, un worker hace `SELECT WHERE sent=false`, envía y marca `sent=true`.
  3. Como una app **local-first** (Replicache, PouchDB/CouchDB): el cliente tiene su propia base y sincroniza.
  4. Un SDK que te da el outbox hecho.
  - "Timestamp en el origen" es el clásico `createdAt` del cliente vs `receivedAt` del server: si usás el del server, un corte de 3 horas aplasta todos los datos en el mismo instante.

## 4G como respaldo

- Router industrial dual SIM (ej. Teltonika RUT956) con **failover**: usa el WiFi/Ethernet del cliente y si se cae pasa a 4G.
- Ventaja comercial: no depender de la red del cliente (y del área de sistemas del cliente).
- Consumo de datos (estimación): con agregación a 10 s y MQTT comprimido, **< 1 GB/mes por máquina**. Un plan de datos chico alcanza.
- Planes M2M en Argentina: no hay precios públicos; hay que cotizar con Claro / Personal / Movistar empresas, o SIM IoT multi-operador (Onomondo, Things Mobile, que operan sobre Claro y Movistar).
- 2G/3G se están apagando: comprar equipos 4G (Cat 1 o Cat 4) como mínimo.
- 💻 El **failover** es como un load balancer con health check: si el primario (WiFi del cliente) no responde, rutea al secundario (4G). La **SIM M2M** es un plan de datos "para máquinas", como una API key con cuota mensual.

## Acceso remoto seguro (para que Chino/fábrica programen o diagnostiquen)

| Opción | Costo orientativo | Notas |
|---|---|---|
| **Ewon Flexy / Cosy (HMS)** | Flexy 205 ~£730 (RS UK); USD 800–1.200 | Estándar de la industria, Talk2M gratuito para acceso básico |
| **Tosibox** | USD 400–700 (fuente secundaria) | VPN "enchufar y listo", popular en pymes |
| **Delta DX + DIACloud** | Consultar | Túnel integrado con ISPSoft |
| **WireGuard / Tailscale en el edge** | Casi gratis (Tailscale personal es solo no comercial; planes pagos ~USD 8/usuario/mes) | Requiere saber lo que se hace; el edge rutea a la subred del PLC |

**Riesgos y reglas**:
- **Nunca** exponer el puerto Modbus (502) ni el de programación a internet. Modbus no tiene contraseña.
- La VPN da acceso a la red de la máquina, no a toda la red del cliente: VLAN o red separada para la máquina.
- Cambiar credenciales por defecto de routers/gateways (vector típico de ataque).
- Programar un PLC a distancia con la máquina andando es peligroso: exigir que haya alguien en planta y la máquina en modo seguro.
- Logs de quién entró y cuándo.
- 💻 Todo esto en términos de AWS:
  - La VPN al edge es un **bastion host o SSM Session Manager** para entrar a una subnet privada.
  - "VLAN separada" es una **VPC con subnet privada + security groups**: la máquina no ve la red del cliente y viceversa.
  - **Ewon, Tosibox y DIACloud** son "Tailscale managed por el fabricante".
  - "Credenciales por defecto" es dejar `admin/admin` en producción.
  - "Logs de acceso" es CloudTrail.
  - "Alguien en planta para programar a distancia" es como exigir un par de ojos antes de un deploy a prod a las 3 AM, porque acá un error mueve motores.

## Costos orientativos por máquina (estimación)

| Concepto | Opción mínima | Opción "industrial" |
|---|---|---|
| Gateway | Pi 5 + caja + fuente: USD 150 | RevPi Connect 4: USD 550–650 |
| Router 4G | (usa red del cliente) | Teltonika RUT956: USD 300 / ~ARS 800k |
| Sensores extra | 0 | Ver nota 05 |
| **Hardware total** | **~USD 150–250** | **~USD 900–1.300** (antes de impuestos de importación) |
| Nube / máquina / mes | USD 0–5 (free tiers, VPS compartido) | USD 5–15 |
| Datos móviles / mes | 0 | Plan chico M2M (cotizar) |

- Un VPS chico (USD 5–20/mes) con EMQX + TimescaleDB + Grafana aguanta decenas de máquinas.
- En AWS IoT Core una máquina mandando 1 mensaje cada 10 s ≈ 260 k mensajes/mes ≈ USD 0,30 (estimación). Lo caro no es IoT Core, es lo que se arma alrededor (bases, cómputo, SiteWise).

## Arquitectura recomendada para empezar (simple y barata)

1. **Simulador**: un script (Python/Node) que expone un servidor Modbus TCP imitando el mapa de registros del AS320T-B de la envasadora.
2. **Edge (Docker en una Pi o en la notebook)**:
   - **Node-RED** lee Modbus cada 1 s, arma JSON con nombres claros.
   - **Mosquitto local** con persistencia y bridge QoS 1 a la nube.
   - (Opcional) Grafana local para mostrar en planta sin internet.
3. **Nube (un VPS o una instancia chica en AWS)**:
   - **EMQX o Mosquitto** como broker.
   - **Telegraf** suscrito a MQTT → **TimescaleDB** (o InfluxDB).
   - **Grafana** con dashboards por máquina y alertas por mail/Telegram/WhatsApp.
4. **Acceso remoto**: WireGuard/Tailscale en el edge, solo para el equipo técnico.
5. **Después**, cuando haya varias máquinas reales: evaluar migrar el broker a AWS IoT Core (certificados por dispositivo, escalado) o ThingsBoard si se quiere un panel multi-cliente sin desarrollarlo.

Por qué así: todo es open source, corre en Docker local para el lab, el dev ya domina la mitad (Node, SQL, dashboards), y nada te ata a un proveedor.

💻 **Resumen en una línea:** es un `docker-compose` con un mock server (simulador Modbus), un worker (Node-RED), Redis local (Mosquitto), un Redis remoto (EMQX), un ingestor (Telegraf), Postgres (TimescaleDB) y un dashboard (Grafana). Migrar después a AWS IoT Core es como pasar de un VPS a servicios managed cuando escala.

## Qué le aporta esto a Chino

- **Diferencial de venta**: "la máquina viene con monitoreo": producción, paradas, alarmas y consumo visibles desde el celular.
- **Servicio postventa**: diagnosticar a distancia antes de mandar un técnico (menos viajes, más rápido).
- **Ingreso recurrente**: abono mensual de monitoreo/soporte por máquina.
- **Feedback de diseño**: datos reales de cómo se usan sus máquinas para mejorar los planos.
- **Base para la IA** (nota 05): sin datos históricos no hay mantenimiento predictivo.
- Costo de entrada bajo: ~USD 150–300 de hardware por máquina para empezar.

## Fuentes

- RevPi Connect 4 (DigiKey): https://www.digikey.com/es/product-highlight/k/kunbus/revpi-connect-4
- RevPi Connect 4 (RS UK): https://uk.rs-online.com/web/p/industrial-computers/2713625
- AWS IoT Core pricing: https://aws.amazon.com/iot-core/pricing/
- AWS blog de precios IoT: https://aws.amazon.com/blogs/aws/aws-iot-update-better-value-with-new-pricing-model
- AWS Greengrass pricing: https://aws.amazon.com/greengrass/pricing/
- AWS IoT SiteWise pricing: https://aws.amazon.com/iot-sitewise/pricing/
- SiteWise + Greengrass stream manager: https://docs.aws.amazon.com/iot-sitewise/latest/userguide/greengrass-stream-manager.html
- Telegraf buffer (comunidad): https://community.influxdata.com/t/telegraf-metric-buffer-limit-cant-exceed-20/39315
- Telegraf MQTT pérdida de datos: https://oneuptime.com/blog/post/2026-08-24-prevent-telegraf-mqtt-data-loss/view
- Node-RED edge→cloud: https://discourse.nodered.org/t/edge-to-cloud-computing/79815
- Mosquitto conf (max_queued_messages): https://manpages.debian.org/jessie/mosquitto/mosquitto.conf.5
- Mosquitto persistent queue (Cedalo): https://docs.cedalo.com/mosquitto/2.9/broker/Mosquitto%20Manual/mosquitto-persistent-queue
- Teltonika bridge QoS0: https://community.teltonika.lt/t/mqtt-bridge-message-caching/17605
- EMQX vs NanoMQ: https://www.emqx.com/en/blog/emqx-vs-nanomq-2023-mqtt-broker-comparison
- EMQX Edge: https://docs.emqx.com/en/emqx-edge/latest/
- ThingsBoard pricing: https://thingsboard.io/pricing/
- ThingsBoard Edge: https://thingsboard.io/products/thingsboard-edge/
- Grafana Cloud pricing: https://grafana.com/products/cloud/pricing/
- Ewon Flexy 205 (RS UK): https://uk.rs-online.com/web/p/routers/2502742
- Comparativa acceso remoto PLC Delta: https://industrialmonitordirect.com/blogs/knowledgebase/delta-plc-hmi-remote-access-solutions-comparison
- Delta DX-2300 manual: https://filecenter.deltaww.com/Products/download/06/060308/Manual/DELTA_IA-IES_DX-2300LN-WW_UM_EN_20180301.pdf
- Teltonika RUT956: https://teltonika-networks.com/es/products/routers/rut956
- RUT956 en España (precio): https://www.pccomponentes.com/router-teltonika-rut956-4g-lte-wifi-4-150-mbps-doble-sim-y-gnss
- SIM IoT Argentina (Onomondo): https://onomondo.com/iot-sim/the-best-iot-sim-cards-for-argentina
- Tailscale pricing: https://tailscale.com/pricing/
