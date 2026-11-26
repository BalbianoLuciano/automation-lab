# 09 — PLCs e industria explicados para un dev web

> Diccionario de traducción: cada concepto industrial explicado con algo del stack que ya usás (React, Node, TypeScript, git, Docker, AWS).
> Las analogías sirven para entender la idea, no son equivalencias exactas. Donde la analogía se rompe, está aclarado con ⚠️.

---

## 1. El PLC: un backend que nunca puede caerse

**Qué es.** Una computadora chiquita que corre **un solo programa en loop infinito**, para siempre.

**Analogía: el game loop, o `requestAnimationFrame`.** Cada vuelta del loop se llama **scan** y dura unos pocos milisegundos:

```ts
// Así "piensa" un PLC, escrito en TypeScript
while (true) {
  const inputs = leerEntradas();         // sensores: ¿hay film? ¿puerta cerrada?
  const outputs = logica(inputs, state); // tu programa
  escribirSalidas(outputs);              // motores, resistencias, válvulas
} // ~1–10 ms por vuelta, siempre igual
```

- Es parecido a un componente React que se re-renderiza miles de veces por segundo. Las entradas son las props, la lógica es el render y las salidas son lo que se pinta.
- **Determinístico:** no hay `async/await`, no hay `fetch`, nada bloquea. Si una vuelta tarda de más, el PLC entra en falla. Es como la regla de "nunca bloquees el render", pero con consecuencias físicas.
- ⚠️ No hay sistema operativo, ni npm, ni internet. Es más parecido a un Arduino muy robusto que a un servidor.

---

## 2. La memoria del PLC: variables globales con nombres raros

En el Delta no declarás `let temperatura = 0`. Usás **zonas de memoria fijas**:

| Delta | Qué es | Analogía TS |
|---|---|---|
| `X0, X1…` | Entradas físicas (sensores) | Props de solo lectura |
| `Y0, Y1…` | Salidas físicas (motores, luces) | Lo que retorna el render |
| `M0…M8191` | Bits internos | `const flags: boolean[] = new Array(8192)` |
| `D0…D65535` | Registros de 16 bits (números) | `const D = new Int16Array(65536)` |
| `T`, `C` | Timers y contadores | `setTimeout` y contadores, pero sincrónicos |
| Memoria retentiva (MRAM) | Sobrevive a un corte de luz | `localStorage` |

- Por ejemplo, la temperatura de la mordaza podría vivir en `D1002 = 1855`, que significa 185,5 °C. Los decimales se guardan multiplicando por 10, como cuando guardás plata en centavos para no usar float.
- **"Steps"** (el AS320T-B tiene 128k) es el tamaño máximo del programa. Es como un límite de bundle size.

---

## 3. Los lenguajes del PLC (norma IEC 61131-3)

| Lenguaje | Analogía | Ejemplo / uso |
|---|---|---|
| **ST (Structured Text)** | TypeScript sin `async`, con sintaxis de Pascal | El más cómodo para vos |
| **Ladder (LD)** | Expresiones booleanas dibujadas como un circuito eléctrico | Lo que el electricista de planta sabe leer |
| **SFC** | Una máquina de estados dibujada, como **XState** o un reducer de Redux | Ideal para los estados de la máquina |
| **FBD / CFC** | Programación visual con cajitas y cables, como **Node-RED** o **n8n** | Lógica de control y PID |

Un ejemplo de la misma lógica en los dos lenguajes:

```pascal
(* ST — en el PLC *)
IF Start AND NOT Stop AND NOT Emergencia THEN
    Motor := TRUE;
END_IF;
```
```ts
// TypeScript — lo mismo
if (start && !stop && !emergencia) motor = true;
```

En Ladder, eso mismo se dibuja como contactos en serie: `--| Start |--|/ Stop |--|/ Emergencia |--( Motor )--`.

### Function Blocks = componentes con estado
- Un **Function Block** (FB) es como un **componente React con su propio estado**, o una clase.
- `FB_TempZone` instanciado 3 veces es igual que hacer `<TempZone id={1} />`, `<TempZone id={2} />` y `<TempZone id={3} />`. Mismo código y estado independiente.
- Una **POU** (Program Organization Unit) es un módulo o archivo.

---

## 4. El IDE y las marcas de PLC

| Industria | Analogía dev |
|---|---|
| **ISPSoft / DIADesigner** (Delta, gratis) | VS Code, pero de un solo fabricante y solo para Windows |
| **TIA Portal** (Siemens, licencia paga) | Visual Studio Enterprise: potente, caro y estándar en empresas grandes |
| **CODESYS** (corre en muchas marcas) | Node.js o Linux: escribís una vez y corre en hardware de distintos fabricantes |
| Simulador de ISPSoft | `localhost` o un emulador de Android: probás sin el equipo real |
| Versión de firmware de la CPU | La versión de Node o el lockfile: si no coincide, el proyecto puede no abrir |
| Módulos de expansión / function cards | Periféricos USB o plugins: agregás entradas, temperatura o comunicación |

### Delta vs Siemens vs CODESYS, explicado con nubes
- **Siemens S7-1200 es AWS.** Es el estándar: todo el mundo lo conoce, hay técnicos e integradores en cualquier lado y el cliente confía. Es más caro y la licencia de software se paga.
- **Delta AS300 es DigitalOcean o Hetzner.** Más barato, alcanza para la mayoría de los casos, con un ecosistema más chico y documentación irregular. Además trae **6 ejes de movimiento integrados**, cosa que el S7-1200 básico no.
- **CODESYS (o Delta AX-3) es Kubernetes.** Una capa estándar que te independiza del hardware y trae extras modernos (OPC UA, EtherCAT), con más complejidad.
- Decisión práctica: **arrancar con Delta**, como eligió Chino, es como arrancar una startup en DigitalOcean. Barato y suficiente. Si un cliente grande exige "Siemens", es como un cliente enterprise que exige AWS: se puede migrar.

---

## 5. Cómo hablar con el PLC: Modbus vs OPC UA

### Modbus TCP: una API REST sin JSON, sin docs y sin contraseña
- Pedís "dame el registro 1002" y te devuelve `1855`. Nada más: ni nombres, ni tipos, ni unidades.
- Es como leer un `Int16Array` por red: **tenés que saber de memoria** que el índice 1002 es la temperatura multiplicada por 10.
- **Holding registers** (R/W) son las `D`. **Coils** (bits R/W) son las `M` y las `Y`.
- ⚠️ **No tiene autenticación.** Exponer el puerto 502 a internet es como exponer Postgres (5432) sin contraseña: cualquiera puede apagarte la máquina.

### El "mapa de registros" es tu contrato de API
- Es una tabla versionada que dice "D1000 = estado, D1002 = temperatura ×10, D1010 = contador de bolsas". Es exactamente un **OpenAPI spec** o una **interface de TypeScript** entre el programa del PLC (backend) y tu software (frontend).
  ```ts
  // El mapa de registros, pensado como tipo
  interface EnvasadoraRegs {
    D1000: Estado;          // 0=parada, 1=produciendo, 2=falla…
    D1002: TempMordaza_x10; // 1855 = 185,5 °C
    D1010: BolsasTotales;
  }
  ```
- Si el programador del PLC mueve un registro sin avisar, rompe tu dashboard, igual que un **breaking change** en una API sin versionar.

### OPC UA: GraphQL con schema, tipos y autenticación
- En vez de "registro 1002", expone objetos con nombre: `Envasadora1.Mordaza.Temperatura` (float, °C).
- Tiene certificados y cifrado, como HTTPS con mTLS.
- El AS300 no lo trae de fábrica. Se agrega un **gateway**, que es como poner un **BFF** (backend-for-frontend) que traduce la API vieja a una moderna.

---

## 6. La pantalla de operador (HMI)

- La **HMI es el frontend** de la máquina: una pantalla táctil de 7" al costado, que lee y escribe registros del PLC.
- **DIAScreen** (el software de las pantallas Delta) es un **builder low-code**, como Webflow o Retool, con scripting en Lua.
- **Recetas** son presets o configuraciones guardadas: "maní 200 g" carga 15 parámetros de una vez, como un `config.json` por producto.
- 💡 Acá está tu fuerte: la mayoría de las HMI industriales tienen UX de los 90. Una pantalla clara, con estados de color y alarmas que dicen **qué hacer**, es un diferencial real.

---

## 7. Máquinas conectadas (IIoT): la arquitectura cloud que ya conocés

```
[PLC]  ──Modbus──>  [Edge]  ──MQTT──>  [Broker]  ──>  [Base de datos]  ──>  [Dashboard]
backend físico      BFF/agente         pub/sub        Postgres series       frontend
```

| Industria | Analogía dev |
|---|---|
| **Edge gateway** (Raspberry o PC industrial en planta) | Un agente o sidecar al lado de la máquina; o Cloudflare Workers: código que corre cerca de la fuente |
| **MQTT** | Pub/sub liviano: Redis Pub/Sub, AWS SNS o canales de WebSocket |
| **Topic MQTT** (`planta1/envasadora1/temp`) | Una ruta o canal: `/api/planta1/envasadora1/temp` |
| **QoS 0 / 1 / 2** | Garantías de entrega: "fire and forget" / at-least-once (como SQS) / exactly-once |
| **Broker** (Mosquitto, EMQX) | Un servidor de mensajería, como RabbitMQ o Redis |
| **Store-and-forward** | Una **PWA offline-first** o el **outbox pattern**: guardás local y sincronizás cuando vuelve internet |
| **TimescaleDB / InfluxDB** | Postgres optimizado para series de tiempo, como una tabla de logs con índices por fecha |
| **Grafana** | Datadog o un dashboard de métricas |
| **Node-RED** | n8n o Zapier: flujos visuales que conectan cosas |
| **Telegraf** | Un agente de logs, como Fluentd o el agente de Datadog |
| **AWS IoT Core / Greengrass** | Lo mismo pero manejado por AWS: IoT Core es el broker y Greengrass el runtime del edge. Te va a sonar del curso |
| **Acceso remoto por VPN** (WireGuard, Tailscale, Ewon) | Un **bastion host** o SSH por VPN para entrar a una red privada |

---

## 8. Control, métricas y seguridad

| Industria | Analogía dev |
|---|---|
| **PID** (control de temperatura) | Un **autoscaler**: mide (CPU o temperatura), compara con el objetivo y ajusta (réplicas o potencia) sin pasarse ni quedarse corto |
| **OEE** = disponibilidad × rendimiento × calidad | Un SLA combinado: uptime × throughput real/esperado × (1 − tasa de error) |
| **PackML** (estados estándar: Idle, Execute, Held, Aborted…) | Los **códigos de estado HTTP**: todo el mundo sabe qué significa un 404, y en PackML todos saben qué es "Held" |
| **Alarma con código** | Un código de error con mensaje, como `ERR_TEMP_HIGH` |
| **Relé de seguridad / STO** (la emergencia no pasa por el PLC común) | **Nunca validar seguridad solo en el frontend.** La parada de emergencia es hardware dedicado, igual que la autorización se valida en el backend aunque el front "ya chequeó" |
| **Zonas y conductos (IEC 62443)** | **VPC con subredes privadas y security groups**: la red de la máquina nunca está expuesta directo |
| **Programa del PLC protegido con contraseña** | Código minificado u ofuscado más licencia: protege el know-how de Chino |
| **Proyecto ISPSoft en git** | Commitear un `.fig` o `.psd`: hay historial pero no diff legible. Por eso conviene exportar a texto y llevar changelog |
| **Backup subido desde la máquina** | Un dump de la base de **producción**, no de tu local: lo que corre en planta es la verdad |
| **Tren de pulsos** (mover un servo) | Llamar `step()` N veces: cada pulso es un paso del motor y la frecuencia es la velocidad |
| **Fieldbus** (CANopen, EtherCAT) | La red interna entre servicios: comunicación de baja latencia entre el PLC y los motores |
| **NPN / PNP** | ⚠️ No hay buena analogía: es la polaridad del cableado eléctrico. Lo que importa es que sensores y PLC sean del mismo tipo, como enchufes del mismo país |

---

## 9. Mapa mental de la envasadora, como app web

```
Sensores / motores        →  hardware (el "cliente" físico)
PLC Delta (ST/SFC)        →  backend crítico, en tiempo real
Mapa de registros Modbus  →  contrato de API (OpenAPI)
HMI (pantalla 7")         →  frontend local
Edge (Raspberry)          →  BFF + agente de sincronización offline-first
MQTT + base + dashboard   →  pub/sub + Postgres + frontend web
Abono mensual             →  SaaS: cobrás por mantener la plataforma
```

Con esto, cuando Chino hable de "registros D", "ladder" o "el HMI", podés traducirlo al instante a algo que ya sabés hacer y aportar desde ahí.
