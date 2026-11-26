# 03 — Stack Delta: PLC AS300 / AS320T-B, HMI, servos y conectividad

> Nota de investigación. **[NV]** = no verificado contra documentación oficial de Delta; confirmar antes de usar en diseño.
> Fuente principal de specs: catálogo Delta "AS Series / Motion CPU" (PDF de distribuidor oficial, ver Fuentes), texto extraído y leído.

---

## 1. Glosario rápido

> 💻 = analogía con el stack web (React, Node, TS, git, Docker, AWS). Es para entender la idea, no una equivalencia exacta. Más en `09-plc-para-devs.md`.

- **PLC**: computadora industrial que ejecuta un programa en loop ("scan") leyendo entradas y escribiendo salidas. Determinístico, robusto, sin SO de propósito general.
  - 💻 Un `while(true)` o game loop: cada vuelta (~1–10 ms) lee sensores, corre tu lógica y escribe a los motores. Como un render de React que nunca para y donde no existe `await`: si algo bloquea, la máquina se detiene.
- **CPU / módulos de expansión**: el AS300 es **modular**: una CPU con algunas E/S propias + módulos que se enchufan al costado (E/S digitales, analógicas, temperatura, celda de carga, comunicación).
  - 💻 La CPU es el servidor base y los módulos son periféricos USB o plugins. Agregás "temperatura" como agregarías un paquete npm, pero físico y atornillado al riel.
- **Steps (pasos)**: unidad de tamaño de programa en PLCs Delta/Mitsubishi-like. ~1 instrucción simple ≈ 1 step.
  - 💻 Un límite de bundle size: 128k steps es el tope de tamaño del programa.
- **Dispositivos / registros**: la "memoria" del PLC tiene nombres por tipo: `X` entradas, `Y` salidas, `M` bits internos, `D` registros de 16 bits (datos), `T` timers, `C` contadores, `S` pasos (SFC), `SM/SR` bits/registros de sistema.
  - 💻 Variables globales prefijadas en vez de `let`:
    - `X` son props de solo lectura e `Y` es lo que retorna el render.
    - `M` es un `boolean[]` y `D` es un `Int16Array(65536)`.
    - `T` es como un `setTimeout` sincrónico y `C` un contador.
    - `SM/SR` son las variables del sistema, como `process.env`.
    - Ejemplo: `D1002 = 1855` puede significar 185,5 °C (se guarda ×10, como plata en centavos).
- **NPN / PNP (sink/source)**: polaridad de las salidas a transistor. El "T" de AS320**T**-B = NPN; la variante **P** = PNP. Importa para cablear sensores/drivers.
  - 💻 ⚠️ No tiene buena analogía de software: es la polaridad del cableado. Es como enchufes de distinto país: sensor y PLC tienen que ser del mismo tipo o no andan.
- **Tren de pulsos**: forma clásica de comandar un servo/paso a paso: cada pulso = un incremento de posición; frecuencia = velocidad.
  - 💻 Llamar `motor.step()` N veces: 1000 pulsos son 1000 pasos, y mandarlos más rápido mueve el motor más rápido.
- **Fieldbus**: red industrial (CANopen, EtherCAT, EtherNet/IP, PROFINET) para hablar con drivers, E/S remotas, etc.
  - 💻 La red interna entre microservicios, como gRPC dentro de un cluster: comunicación rápida y de baja latencia entre el PLC y los motores.
- **Function card**: tarjetita que se inserta en la CPU AS300 (hasta 2) para agregar un puerto (RS-232/485, CANopen, Ethernet, OPC UA, analógica).
  - 💻 Una placa de expansión, como sumarle una placa de red a una PC: le da al PLC un "idioma" nuevo para comunicarse.

---

## 2. Delta AS300 / AS320T-B — specs clave

| Ítem | AS320T-B (dato catálogo Delta) |
|---|---|
| E/S integradas | **8 DI / 12 DO** (salidas transistor **NPN**; AS320P-B = PNP) |
| Salida de pulsos | **6 ejes, 200 kHz** (open collector) |
| Contadores rápidos | **4 canales, 200 kHz** |
| Puertos integrados | **USB, Ethernet, 2× RS-485**, slot **Micro SD** |
| Protocolos | **Modbus RTU/ASCII, Modbus TCP, EtherNet/IP**, **CANopen DS301 (requiere function card AS-FCOPM)** |
| Conexiones Ethernet | Modbus TCP: 32 cliente / 32 servidor; EtherNet/IP (CIP): 32 |
| Programa | **128k steps** |
| Memoria de datos | `D`: 64k words (30k de usuario); FR: 64k words (parámetros) |
| Bits/timers/contadores | M0–M8191, S0–S2047, T0–T511, C0–C511, HC0–HC255 |
| Velocidad | LD 25 ns; MOV 0,15 µs |
| Expansión | hasta **32 módulos** (máx. 16 analógicos / 4 de comunicación), hasta **1.024 E/S** reales |
| Function cards | hasta **2** (RS-232/422/485, CANopen, Ethernet, PROFINET, **OPC UA (AS-FOPC02)**, analógicas) |
| Retención | Programa en Flash; área retentiva en **MRAM** (sin batería); RTC con pila CR1620 |
| Alimentación CPU | 24 VDC (fuente AS-PS02 110–240 VAC) |

💻 **Cómo leer la tabla como dev:**
- **8 DI / 12 DO**: 8 "inputs" (sensores que leés) y 12 "outputs" (cosas que prendés y apagás).
- **6 ejes de pulsos**: podés mover 6 motores con precisión a la vez, como 6 workers independientes.
- **Contadores rápidos**: contadores por hardware para eventos muy rápidos (por ejemplo, un encoder que da miles de pulsos por segundo), como un rate counter que no pasa por el event loop.
- **32 conexiones Modbus TCP servidor**: el PLC acepta hasta 32 clientes simultáneos, como un `max_connections` de Postgres.
- **Flash / MRAM**: el programa vive en Flash, que es el disco. MRAM es la memoria que sobrevive a cortes de luz, como `localStorage` o un volumen persistente de Docker.
- **RTC**: el reloj interno, el equivalente a `Date.now()`, con su pila para no perder la hora.

Notas:
- Hay un listado de un revendedor que lo describe como "módulo de 32 entradas" — es un error; el catálogo de Delta dice 8DI/12DO.
- Para una VFFS: 6 ejes de pulsos alcanzan sobrado para arrastre de film + mordaza servo + sinfín. Faltarán E/S: lo normal es sumar un módulo digital (ej. AS16AP11T-A) y uno de temperatura.
- **Módulos útiles para envasadora**: `AS04TC-A`/`AS06TC` (termocuplas), `AS04RTD-A` (PT100), `AS02LC`/`AS04LC` (**celda de carga** — permitiría hacer el pesaje en el mismo PLC) **[NV modelos exactos disponibles en Argentina]**.
- **Control de ejes por CANopen**: Delta anuncia hasta 8 ejes de posicionamiento CANopen con AS-FCOPM, aunque DigiKey lista AS-FCOPM como "not for new designs" **[NV reemplazo actual]**.
- Funciones de movimiento avanzadas (leva electrónica, flying shear) están bien documentadas para la línea CODESYS de Delta (AX/DXMC) y para servos ASDA-A2/A3; para AS300 con pulsos **[NV]** — consultar manual de instrucciones de movimiento del AS.

### Software: ISPSoft (y DIADesigner)
- **ISPSoft**: IDE gratuito de Delta para DVP, AS y AH. Windows.
  - 💻 VS Code, pero de un solo fabricante y solo para Windows. Escribís, compilás y "deployás" (descargás) el programa al PLC por USB o Ethernet.
- Lenguajes (IEC 61131-3 y afines) según catálogo del AS300: **LD (ladder), ST (texto estructurado), SFC (diagrama secuencial), CFC**. La página de ISPSoft menciona también **FBD e IL**. El catálogo AS lista LD/ST/CFC/SFC.
  - Para alguien de software: **ST** es lo más parecido a Pascal; **SFC** es literalmente una máquina de estados gráfica (ideal para PackML); **LD** es lo que el electricista de planta va a querer leer.
  - 💻 Equivalencias:
    - **ST** es TypeScript sin async con sintaxis Pascal: `IF Start AND NOT Stop THEN Motor := TRUE; END_IF;` equivale a `if (start && !stop) motor = true;`.
    - **SFC** es **XState** o un reducer de Redux dibujado.
    - **LD** son expresiones booleanas dibujadas como circuito eléctrico.
    - **FBD/CFC** es programación visual con cajitas y cables, como Node-RED o n8n.
    - **IL** es tipo assembler. Está en desuso.
- Function blocks (POUs con instancias, tipo "clases"), anidables hasta 32 niveles, parámetros por valor o referencia.
  - 💻 Un Function Block es un **componente React con estado propio**. `FB_TempZone` instanciado 3 veces es como `<TempZone id={1}/>`, `<TempZone id={2}/>` y `<TempZone id={3}/>`: mismo código y estado independiente. Una **POU** es un módulo o archivo.
- **DIADesigner**: el entorno nuevo de Delta lista soporte para AS100/AS200/AS300 además de servos ASDA. Delta todavía linkea ISPSoft desde la página del AS300 → conviven **[NV cuál recomienda Delta hoy para proyectos nuevos]**.
  - 💻 Como cuando conviven dos versiones de un framework (Angular.js y Angular): el nuevo existe, pero mucha documentación y ejemplos siguen en el viejo.
- **HWCONFIG** (dentro de ISPSoft) configura módulos y parámetros de red; **COMMGR** es el driver de comunicación PC–PLC.
  - 💻 HWCONFIG es tu `docker-compose.yml` o `.env` del hardware: declara qué módulos hay, la IP y los puertos. COMMGR es el driver o adaptador de conexión, como el cliente que te conecta a la base.

---

## 3. Ecosistema Delta alrededor

### HMI — serie DOP-100
- Categorías: Basic, Standard, Advanced, Handheld. Tamaños 4,3" a 15" (el típico para envasadora: **7" DOP-107**, variantes EG/EV con Ethernet).
- Software: **DIAScreen** (antes DOPSoft). Editor **Lua** integrado para scripting, alarmas, históricos, usuarios/permisos.
- Funciones de red: FTP, e-mail, **VNC** (monitoreo remoto), recetas.
- Comunicación con AS300 por Ethernet o RS-485 con drivers nativos Delta.
- 💻 Cómo verlo como dev:
  - **La HMI es el frontend de la máquina**: una pantalla táctil que lee y escribe registros del PLC, como un front que consume una API.
  - **DIAScreen** es un builder low-code, como Webflow o Retool, con Lua en lugar de JS.
  - Las **recetas** son presets o un `config.json` por producto: "maní 200 g" carga 15 parámetros de una vez.
  - **VNC** es un "screen share" remoto de la pantalla.
  - **RS-485** es un cable serie: el abuelo de Ethernet, lento pero robusto.
  - La mayoría de las HMI industriales tienen UX de los 90, así que acá tu fuerte en UX es un diferencial directo.

### Servos — ASDA
- **ASDA-B3**: línea estándar actual (pulsos, CANopen, EtherCAT según variante "-L", "-F", "-E"). Compatible con A2/A3.
- **ASDA-A3**: alta gama (e-cam interna, mayor ancho de banda). **ASDA-B2**: anterior, muy común en máquinas existentes.
- Con el AS320T-B la vía más simple es **pulso + dirección** (o CANopen con function card). EtherCAT queda para la línea AX/CODESYS.
- 💻 Cómo verlo como dev:
  - Un **servo** es un motor con feedback: le pedís "andá a la posición 500" y él solo corrige hasta llegar, como un `scrollTo()` con animación que siempre termina exacto.
  - El **driver ASDA** es el "microservicio" que maneja ese motor. El PLC le manda órdenes y el driver resuelve el detalle.
  - La **e-cam** (leva electrónica) sincroniza un eje con otro, por ejemplo la mordaza con el avance del film, como dos animaciones atadas a la misma timeline.
  - **B2/B3/A3** son versiones del mismo producto, como v2/v3 de una librería.

### OPC UA (para "hablar IT")
- **OPC UA** = protocolo industrial moderno con modelo de información, tipado, seguridad (certificados, cifrado). Es el estándar de la Industria 4.0. Modbus es "registros pelados sin semántica"; OPC UA es "objetos con nombre y tipo".
  - 💻 **Modbus vs OPC UA es como un array de números vs GraphQL con schema:**
    - En Modbus pedís `registers[1002]` y te devuelve `1855`. Tenés que saber de memoria que es la temperatura ×10.
    - En OPC UA pedís `Envasadora1.Mordaza.Temperatura` y te devuelve `185.5` (float, °C), con autenticación por certificados, como mTLS.
- **El AS300 no trae OPC UA nativo en la CPU** (solo Modbus TCP y EtherNet/IP). Opciones:
  1. **Function card AS-FOPC02** (OPC UA) — figura en el catálogo AS y en el manual AS (rev. 2020). Compatibilidad específica con AS300 y prestaciones **[NV]**.
  2. **DIAView** (SCADA de Delta) con módulo OPC UA Server / Web API.
  3. **Gateway por software** en PC/Raspberry que lee Modbus TCP y expone OPC UA. 💻 Es un **BFF / adapter**: un servicio que consume la API vieja y expone una moderna. Opciones:
     - **Node-RED** + `node-red-contrib-modbus` + `node-red-contrib-opcua` (open source, el más práctico).
     - **node-opcua** (Node.js) / **open62541** (C) / **asyncua** (Python) para hacer un server propio.
     - **EMQX Neuron**: edición open source solo trae Modbus y MQTT; OPC UA en versión comercial (NeuronEX).
     - **Kepware KEPServerEX** (comercial, driver Modbus TCP), **Softing uaGate MB**, **Advantech EKI-1242** (gateway de hardware).
     - **Apache PLC4X**: librería multi-protocolo (Java/Go/Python) con Modbus.
  4. Si se necesitara OPC UA nativo fuerte: la línea **AX-300N / AX-3 (CODESYS)** de Delta trae servidor OPC UA y usa las mismas E/S de la serie AS.
  - 💻 Herramientas de la lista, traducidas:
    - **Node-RED** es n8n o Zapier.
    - **node-opcua** es un paquete npm, y **asyncua** es su equivalente en Python.
    - **Kepware** es un "conector enterprise" pago, como un iPaaS.
    - **PLC4X** es como un ORM multi-base, pero para protocolos industriales.
    - Pasar a **AX-3/CODESYS** es como migrar de un hosting propietario a Kubernetes: más estándar y más features, con más complejidad.

---

## 4. Leer datos del PLC desde una PC / Raspberry Pi

### 4.1 Camino recomendado: Modbus TCP
- El AS300 actúa como **servidor Modbus TCP** (puerto estándar **502**) sobre su Ethernet integrada. Se habilita en HWCONFIG.
- Conceptos Modbus:
  - **Coils** (bits R/W, FC01 leer / FC05/FC15 escribir) → mapean `M`, `Y`.
  - **Discrete inputs** (bits RO, FC02) → `X`.
  - **Holding registers** (words R/W, FC03 leer / FC06/FC16 escribir) → `D`.
  - Direcciones "40001" son notación humana 1-based; en las librerías se usa el **offset 0-based**.
  - 💻 **Modbus TCP es una API REST minimalista sin JSON, sin docs y sin contraseña:**
    - El **puerto 502** es su "puerto HTTP".
    - Los **function codes** (FC03, FC06…) son los "verbos": FC03 es `GET /registers/100?count=10` y FC06 es un `PUT` de un valor.
    - **Holding registers** son un `Int16Array` remoto de lectura y escritura, y **coils** un `boolean[]` remoto.
    - "40001" vs offset 0 es el clásico *off-by-one* de arrays 1-based contra 0-based, como Lua contra JS.
    - ⚠️ No tiene auth: exponer el 502 a internet es como dejar Postgres en 5432 abierto sin password.

### 4.2 Mapa de direcciones Delta — CUIDADO
- **DVP (serie vieja)**: `D0` = 0x1000 (4096), `M0` = 0x0800 (2048), `Y0` = 0x0500, `X0` = 0x0400. Muy documentado.
- **AS300**: varias fuentes de terceros indican que `D0` = holding register **0** (40001) y `M0` = coil 0, es decir **mapeo directo** distinto al DVP **[NV — confirmar en el apéndice "Modbus communication address" del AS Series Operation/Programming Manual]**. No pude descargar el manual oficial para verificarlo.
- **Prueba empírica recomendada**: escribir un valor conocido en `D100` desde ISPSoft y leer con un cliente Modbus (offset 100 y offset 4196) para ver cuál responde.
- Valores: `D` es 16 bits sin signo en el cable → convertir a signed en el cliente. Valores de 32 bits/float ocupan 2 registros consecutivos (verificar orden de palabras / endianness).
  - 💻 Lo mismo que hacer en JS `new DataView(buffer).getInt16()` contra `getUint16()`: un `-5` llega como `65531` si no lo convertís. Un float de 32 bits viene partido en dos "mitades", y si las armás en el orden equivocado (endianness) te da un número absurdo, como leer un timestamp con los bytes al revés.
- 💻 Por qué el mapa es distinto entre DVP y AS300: dos versiones de la misma API con rutas distintas (`/v1/D/0` vive en 4096 y `/v2/D/0` en 0). La prueba empírica es literalmente "pegarle al endpoint y ver cuál responde".

### 4.3 Librerías
- **Python**: `pymodbus` (completa, async), `pyModbusTCP` (más simple), `minimalmodbus` (RTU por serie).
- **Node.js**: `modbus-serial`, `jsmodbus`; en Node-RED `node-red-contrib-modbus`.
- **Go**: `goburrow/modbus` / `simonvetter/modbus`. **Rust**: `tokio-modbus`.
- Ejemplo mínimo (pymodbus 3.x):
  ```python
  from pymodbus.client import ModbusTcpClient
  c = ModbusTcpClient("192.168.1.5", port=502)
  c.connect()
  rr = c.read_holding_registers(address=100, count=10, slave=1)  # D100..D109 si el mapeo AS es directo [NV]
  print(rr.registers)
  c.close()
  ```
  (En pymodbus ≥3.7 el parámetro `slave` se llama `device_id` **[NV según versión]**.)

### 4.4 Simuladores (para desarrollar sin PLC)
- **ISPSoft** trae simulador de PLC (online simulation) para probar lógica **[NV si expone Modbus TCP a herramientas externas]**.
- **Servidores Modbus de prueba**: `pymodbus.server` (simulador incluido), **diagslave**, **ModbusPal**, **Modbus Slave (Witte, pago)**. Clientes: **Modbus Poll**, **QModMaster** (open source), **mbpoll** (CLI).
- Estrategia: definir un **mapa de registros** (CSV/YAML versionado) y montar un simulador con ese mapa → el dashboard se desarrolla en paralelo al programa del PLC.
  - 💻 Es **API-first con mocks**: el mapa de registros es tu OpenAPI spec, el simulador es un mock server (como MSW o json-server) y el front se desarrolla sin esperar al back. **Modbus Poll / QModMaster / mbpoll** son tu Postman, Insomnia o `curl`.

### 4.5 Arquitectura edge sugerida
```
[AS320T-B] --Modbus TCP--> [Raspberry/mini PC industrial]
                               ├─ colector (Python/Node) → TimescaleDB/InfluxDB/SQLite
                               ├─ Node-RED (opcional) → OPC UA / MQTT
                               └─ web app (OEE, recetas, lotes, alarmas)
```
- Para planta: preferir mini PC industrial o Raspberry con carcasa DIN + fuente 24 V; la Raspberry común con SD es frágil (corrupción de tarjeta) **[criterio, no fuente]**.
- **Solo lectura por defecto**; cualquier escritura al PLC (recetas) con validación de rangos y confirmación en HMI.
  - 💻 El edge es un **BFF + agente**: consume al PLC (el backend físico) y le sirve datos al resto. "Solo lectura por defecto" es como darle a un servicio un usuario de base `SELECT`-only. Escribir recetas es un `UPDATE` en producción: con validación (zod), confirmación y log.

---

## 5. Buenas prácticas de programación y gestión

### Estructura del código
- Separar en **POUs**: `Main` (orquesta), `StateMachine` (PackML/SFC), un FB por **subsistema** (`FB_FilmDrive`, `FB_Jaws`, `FB_TempZone` instanciado 3 veces, `FB_Doser`, `FB_Alarms`).
- **Mapa de memoria documentado**: rangos de `D`/`M` reservados por función (ej. D0–D999 recetas, D1000–D1999 estado/telemetría para Modbus, M1000+ alarmas). Ese bloque "público" es el **contrato API** con el software.
  - 💻 Como separar rutas por recurso (`/recipes`, `/status`, `/alarms`) o, mejor, una **interface de TypeScript** versionada:
    ```ts
    interface EnvasadoraRegs {
      D1000: Estado;          // 0=parada, 1=produciendo, 2=falla
      D1002: TempMordaza_x10; // 1855 = 185,5 °C
      D1010: BolsasTotales;
    }
    ```
    Si el programador del PLC mueve un registro sin avisar, es un **breaking change** y rompe el dashboard.
- 💻 Lo de arriba (`Main`, `StateMachine`, un FB por subsistema) es la misma arquitectura que un front bien hecho: un entry point que orquesta, un store o máquina de estados global y componentes por feature.
- Alarmas con código numérico + texto en HMI + registro con timestamp.
- Nada de "magic numbers": constantes y parámetros en registros de receta.

### Versionado
- Los proyectos ISPSoft son **binarios** → git sirve para historial pero no para diff/merge.
- Prácticas: commit del proyecto + **export a texto** (ST/listado de instrucciones/tabla de símbolos) cuando sea posible **[NV qué exporta ISPSoft en texto]**; changelog manual por versión; tag por máquina entregada (`maq-001-v1.2`).
- **Versión de firmware** de CPU y de ISPSoft anotadas en el repo (una diferencia de versión puede impedir abrir/descargar el proyecto).
- Registrar en el PLC un registro con la **versión del programa** (ej. D en zona pública) para que el software sepa contra qué está hablando.
- 💻 Equivalencias:
  - Un proyecto ISPSoft en git es como commitear un `.fig` o `.psd`: hay historial pero no diff legible. Por eso conviene exportar a texto y llevar changelog.
  - El tag `maq-001-v1.2` es un release tag.
  - La versión de firmware es tu `.nvmrc` o lockfile: si no coincide, el proyecto puede no abrir.
  - La versión del programa en un registro D es un endpoint `/version` o `/health`.

### Backups
- Backup del programa **subido desde la máquina** al entregarla (no solo el del escritorio) + receta + parámetros de servos (ASDA-Soft) + proyecto HMI (DIAScreen).
- La Micro SD del AS300 permite copias de programa/datos **[NV procedimiento exacto]**.
- Proteger el programa con contraseña (protección de know-how de Chino).
- 💻 Equivalencias:
  - "Backup subido desde la máquina" es un **dump de la base de producción**, no de tu local: lo que corre en planta es la verdad, como prod contra tu rama.
  - Los parámetros de servos y el proyecto HMI son "configuración de infraestructura" que también hay que versionar, como el `docker-compose` y los `.env`.
  - La contraseña del programa es como minificar u ofuscar más una licencia.

### Documentación
- Por máquina: esquema eléctrico, lista de E/S (tag, dirección, descripción, cable), mapa Modbus, lista de alarmas, manual de operador, parámetros de servos, versiones.

---

## 6. Comparación breve con alternativas

| | Delta AS300 | Siemens S7-1200 (ej. 1214C) | Otras |
|---|---|---|---|
| Precio | Bajo/medio. AS320T-B **~ARS 824.536** (referencia MercadoLibre, nov 2026, dato provisto; no pude ver el precio en la búsqueda) | Más caro. No encontré precio en ARS; como referencia, RS Online lista variantes 1214C en ~€700–1.150 (sin IVA, otros mercados) **[NV para Argentina]** | **Schneider M221/M241**, **Mitsubishi FX5U**, **Omron NX1P** (mid/high), **Unitronics** (PLC+HMI), **WEG CLIC/ PLC300** (fuerte en Brasil/Argentina), **CODESYS en hardware genérico** |
| Software | ISPSoft/DIADesigner gratis | TIA Portal (licencia paga) | CODESYS gratis |
| Ejes | 6 ejes pulsos 200 kHz integrados | 4 ejes PTO 100 kHz (DC/DC/DC) **[NV]** | — |
| Comunicación | Modbus TCP/RTU, EtherNet/IP, CANopen (card) | PROFINET, S7, Modbus TCP, **OPC UA server nativo (firmware ≥4.4)** **[NV versión]** | — |
| Ecosistema | Bueno en Asia/LatAm, documentación irregular | Estándar de facto en Argentina; muchos integradores y técnicos; la planta del cliente probablemente lo conoce | — |
| **Analogía dev** (ver `09-plc-para-devs.md`) | **DigitalOcean/Hetzner**: barato, alcanza, ecosistema más chico | **AWS**: el estándar que todos conocen, más caro, licencia paga | **CODESYS = Kubernetes**: capa estándar que te independiza del hardware |

- 💻 Términos de la tabla:
  - **TIA Portal** es Visual Studio Enterprise: licencia paga y estándar en empresas grandes.
  - **PROFINET** es el "protocolo propietario de Siemens" para su ecosistema, como iMessage dentro del mundo Apple.
  - **CODESYS gratis** es como Node u open source: corre en hardware de muchas marcas.
  - **Unitronics (PLC+HMI)** es un all-in-one, como un framework fullstack tipo Next.js: front y back en una sola caja.
- Lectura: Delta es coherente para una pyme que necesita costo bajo y movimiento integrado. Siemens se paga en precio pero suma "aceptación" del cliente final y OPC UA nativo.
- Para escalar a máquinas más complejas o con más software: mirar **Delta AX-3 (CODESYS + OPC UA + EtherCAT)** manteniendo E/S AS.

---

## 7. Qué le aporta esto a Chino

- El **AS320T-B es suficiente** para una VFFS intermitente: 6 ejes de pulsos, Modbus TCP y EtherNet/IP integrados, módulos de temperatura y celda de carga.
- **El diferencial de software no requiere cambiar de PLC**: con Modbus TCP + un edge barato se puede ofrecer OEE, recetas, registro de lotes y alarmas remotas.
- **Definir desde ya un "contrato" de registros públicos** (mapa Modbus versionado). Es el puente entre el programador de PLC y el desarrollador.
- Si un cliente pide OPC UA: resolverlo con **gateway (Node-RED/node-opcua)** o function card AS-FOPC02 antes de migrar de plataforma.
- **Proteger el know-how**: contraseña de programa + backups ordenados + versión en registro.
- **Riesgo a cubrir**: confirmar con Delta Argentina (o el distribuidor) el mapa Modbus AS, disponibilidad local de módulos (TC, LC, AS-FOPC02) y soporte de DIADesigner.

---

## 8. Fuentes

- Delta — Catálogo AS Series Motion CPU (PDF distribuidor, tabla de specs AS320T-B): https://www.pneu-hyd.co.th/images/Products/automation/plc/delta/as/motion-cpu/motion-cpu.pdf
- Delta — AS Series Standard CPU: https://www.deltaww.com/en-US/products/060301/3495
- Delta — ISPSoft features: https://www.deltaww.com/en-US/products/PLC-Programmable-Logic-Controllers/3598
- Automate.org — AS300 compact modular mid-range PLC: https://www.automate.org/motion-control/news/as300-series-compact-modular-mid-range-plc
- Control Design — Delta modular series with integrated motion control: https://www.controldesign.com/vendors/products/2018/delta-modular-series-with-integrated-motion-control/
- DigiKey — AS Series highlight: https://www.digikey.com/en/product-highlight/d/delta-electronics/as-series-of-compact-modular-mid-range-plc
- DigiKey — AS-FCOPM (CANopen card): https://www.digikey.com/en/products/detail/delta-electronics-industrial-automation/AS-FCOPM/9862037
- Delta AS-FOPC02 (OPC UA card), distribuidor: https://deltaacdrives.com/delta-as-fopc02-as-series-plc-accessories/
- Delta — Instruction sheet AS-FEN02/FPFN02/FOPC02: https://deltaacdrives.com/Accessories/Delta-AS-FEN02-FPFN02-FOPC02-FECAT-AS-Series-Communication-Card-Instruction-Sheet.pdf
- Delta — AS Series Operation Manual (2020): https://deltronics.ru/images/manual/AS_OM_EN_20200707.pdf
- Delta — DIADesigner specification: https://diastudio.deltaww.com/home/Specification?Type=3
- Delta — DIAView (OPC UA server option): https://www.deltaww.com/IA/downloadcenter/catalogue/7.Software/DIAView/DELTA_%20IA-OMS_DIAView_C_EN_20220719.pdf
- Delta AX-300N/AX-324N (CODESYS, OPC UA): https://engineering-update.co.uk/?p=67350
- Delta — DOP-100 catálogo: https://deltronics.ru/images/catalogue/DOP-100_C_EN_20200109_web.pdf
- DigiKey — DOP-100 series: https://www.digikey.com/en/product-highlight/d/delta-electronics/dop-100-series-hmi
- Delta — ASDA-B3 catálogo: https://www.mouser.com/datasheet/2/632/DELTA_IA_ASDA_ASDA_B3_C_EN_20230523_web-3318784.pdf
- Maple Systems — AS300/DVP-ES3 Modbus connection guide (rangos de dispositivos): https://maplesystems.com/support-center/controller-connection-guides/delta-electronics-as300-dvp-es3-modbus-rtu-ascii-ebpro/
- Tabla Modbus Delta DVP (Scribd): https://www.scribd.com/document/382276093/Modbus-Address-Table-Delta-Plc
- Industrial Monitor Direct — Delta Modbus mapping (tercero, usar con cautela): https://industrialmonitordirect.com/blogs/knowledgebase/delta-plc-modbus-register-mapping-and-function-codes
- pymodbus — grupo/hilo Delta + Python: https://groups.google.com/g/pymodbus/c/zgS9AkbllYo
- Apache PLC4X — Modbus: https://plc4x.apache.org/plc4x/pre-release/users/protocols/modbus.html
- OPCModbusUAServer (node-opcua + jsmodbus): https://github.com/BoBiene/OPCModbusUAServer
- Node-RED Modbus→OPC UA ejemplo: https://github.com/glinvad/Node-Red-OPCuaServer-UR
- EMQX Neuron: https://github.com/emqx/neuron
- Softing uaGate MB: https://industrial.softing.com/products/gateways/gateways-for-access-of-controller-data/uagate-mb.html
- Advantech EKI-1242: https://emt-iiot.advantech.com/products/EKI-1242IOUMS/mod/C0433DFF-3F81-4611-B89E-5360B9AB7E50
- Siemens Industry Mall — S7-1200 1214C: https://mall.industry.siemens.com/mall/es/WW/Catalog/Product/6AG1214-1HG40-4XB0
- RS Online — 1214C precio referencia EUR: https://es.rs-online.com/web/p/controladores-plcs-y-automatas/2330521
- MercadoLibre AR — clip "Plc Delta As320t-b": https://www.mercadolibre.com.ar/shorts/clips/jfxgXp/plc-delta-as320t-b
