# Resumen para la charla con Chino

> Una página. El detalle y las fuentes están en las notas 01–08. Los precios marcados "~" son estimaciones.
> Para traducir la jerga industrial a conceptos de desarrollo web: `09-plc-para-devs.md`.

## La idea en una frase
**Máquinas nacionales con el software y el servicio de una europea, al precio de una nacional.**
Los fabricantes argentinos venden "el fierro". Las europeas cobran caro el software y el servicio. Las chinas llegan sin soporte. El hueco del medio está vacío.

## 5 diferenciales, en orden

1. **Máquina conectada de fábrica** (`04`, `01`)
   - Ninguno de los fabricantes nacionales que revisamos ofrece monitoreo remoto, OEE ni alertas.
   - Costo técnico: ~USD 150–1.300 de hardware por máquina y ~USD 15–30 por mes de nube y SIM.
   - Para el cliente: ve producción, paradas y alarmas desde el celular.
   - Para nosotros: diagnosticamos sin viajar.

2. **Abono mensual de soporte** (`06`)
   - Resuelve el dolor que Chino ya conoce: el soporte lento.
   - Propuesta de 3 niveles: ~USD 39, 89 y 149 por máquina por mes.
   - El primer año del nivel básico va incluido en el precio de la máquina.
   - Solo el 4% de los fabricantes de maquinaria logra cobrar servicios digitales recurrentes (BCG 2025). Hay poca competencia.

3. **"La máquina que te arma el registro de lote"** (`07`, `02`)
   - Junta lote, vencimiento, peso controlado, recetas y auditoría de cambios.
   - Le sirve a la pyme alimenticia para cumplir el Código Alimentario.
   - Sale por Modbus TCP, sin cambiar de PLC.

4. **Software de control profesional** (`03`, `02`)
   - Programa del PLC Delta estructurado con el estándar PackML desde la máquina 1.
   - Código versionado y con backups.
   - Pantalla de operador clara: acá entra el fuerte en UX.
   - Seguridad bien resuelta: emergencias con relé de seguridad y corte seguro de los servos.
   - Cumple la Res. SIC 17/2025, que exige la IEC 60204.

5. **Ventas con 3D/VR del galpón** (`08`)
   - Se le muestra al cliente su galpón con la línea ya ubicada, antes de comprar.
   - App web propia: ~6–8 semanas de MVP. El visor VR (Quest 3) es opcional: ~ARS 1,2–1,5 M.
   - Precisión: medidas con láser y escaneo con celular como fondo visual.
   - Lo profesional cuesta ~€249 por mes (visTABLE) o ~USD 6.000 por año (FlexSim).

## IA: dónde sí y dónde todavía no (`05`)
- **Ya:**
  - Alertas y tendencias.
  - Control automático de aireación de granos, con base INTA y sin machine learning. El maní de Chino puede ser el piloto.
  - Asistente para técnicos que responde con los manuales y cita la fuente.
- **Después:**
  - Visión artificial para sellado y etiquetas. El hardware cuesta ~USD 70–250; lo difícil es la iluminación.
  - Mantenimiento predictivo. Necesita meses de datos de máquinas reales.

## Para preguntarle a Chino
1. ¿Quién programa hoy el PLC y la pantalla? ¿Usa ISPSoft?
2. ¿Qué potencia tiene la envasadora? Si supera los 63 A o 5 kVA, cambia la norma que aplica.
3. ¿Qué tipo de envasadora y dosificador hace primero (vertical, multicabezal, sinfín)? ¿Para qué producto?
4. ¿Le gusta la idea del abono? ¿Quién haría el soporte en campo?
5. ¿Podemos usar una máquina de su empresa de maní como piloto para conectar y medir?
6. ¿Cómo se reparte mi parte (socio, por proyecto, sueldo)?

## Demo propuesta
**Gemelo digital web de una envasadora:**
- Un simulador del Delta AS320T-B por Modbus TCP.
- Un panel en vivo con OEE, alarmas y una falla simulada.
- La máquina en 3D, y un modo galpón para ubicar máquinas.

Mensaje: *"esto, conectado a tu PLC real, es lo que le vendemos al cliente como abono"*.

## Pendientes antes de la charla
- Pedir 2–3 cotizaciones de envasadoras nacionales: no hay precios publicados.
- Confirmar el mapa de registros Modbus del AS300 con el manual oficial (marcado [NV] en `03`).
- Cotizar planes de SIM M2M en Argentina.
