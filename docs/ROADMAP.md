# ROADMAP — Laboratorio 2: backend multinodo y demostración

**Checkpoint vigente: 25/10/2026.** P13 integrado con tres ESP32; P14-A documenta recuperación Wi-Fi funcional con contraste SQL limitado, y P14-B/C documentan recuperación funcional con dictamen global **INCOMPLETO**. P15 y la verificación retrospectiva T05 acreditan 336 claves de la ventana física; la prueba LOAD caracteriza HTTP en loopback. El informe P16 está versionado. La entrega y demostración presencial del **28/10/2026** aún no han ocurrido. La revisión presencial del 21/10 fue realizada sin observaciones, según confirmó el estudiante.

**Producto actual:** https://github.com/FAJIFBEFJHA/lab2-software-telecomunicaciones

**Origen histórico:** `jmamanilari-cyber/lab1-iot-node`, subdirectorio `lab2/`.

**Ubicación local:** `C:\Proyecto\lab2-software-telecomunicaciones`.

**Continuidad:** `PROJECT_CONTEXT.md` del repositorio de continuidad; su actualización coordinada está pendiente.

**Guía oficial:** reconoce T01–T05. El PDF local citado en el checkpoint antiguo no se presupone versionado aquí; el informe final versionado resume sus criterios. La evidencia del código y de las pruebas prevalece para describir el comportamiento implementado.

**Estado Git antes de esta edición:** `main` y `origin/main` apuntaban a `76b4e52`, con árbol limpio. Este ROADMAP es un cambio local posterior; no se ejecutaron servicios ni pruebas para actualizarlo.

## Alcance y criterios

Backend PHP 8.2/PDO/MariaDB para **tres ESP32**, dos transportes (HTTP REST y MQTT), un **TelemetryService** de dominio, autenticación, ACL MQTT, validación, persistencia, idempotencia, pruebas y evidencias. No modificar el Laboratorio 1 ni añadir Edge Computing/Laboratorio 3 o dashboard fuera de requisitos.

Rutas implementadas: `POST /api/v1/telemetry`, `GET /api/v1/devices`, `GET /api/v1/devices/{id}/telemetry`, `PUT /api/v1/devices/{id}/config`, `GET /api/v1/events`.

**Revisión del miércoles 21/10: realizada, sin observaciones del docente**, según confirmó el estudiante. La evaluación final permanece programada para el **miércoles 28/10**, con demostraciones prácticas. La revisión favorable no acredita por sí sola la entrega final.

## Estado de desarrollo y verificación

| Etapa | Situación | Evidencia |
|---|---|---|
| A. Arquitectura y contratos | Implementada | Controller, Service, Repository y Subscriber en la raíz del repositorio independiente |
| B. MariaDB/PDO | Probada históricamente en escritorio y laptop; nueva comprobación pendiente | Cuatro tablas y tres identidades; MariaDB estaba detenido en la última comprobación tras la migración |
| C1. Telemetría HTTP | Probada históricamente en escritorio y laptop | T01–T04, errores y concurrencia; no repetida desde la nueva raíz |
| C2. API REST restante | Probada históricamente en escritorio y laptop | Cuatro endpoints, filtros, permisos y configuración; no repetida desde la nueva raíz |
| D1. MQTT seguro de loopback | Probada históricamente en escritorio y laptop | ACL, subscriber, duplicados y reanudación; nueva validación operativa pendiente |
| H. Herramienta de demostración | Verificada históricamente en laptop y escritorio | Menú opciones 1–7; corrección PUBACK 0/16 conservada; rutas adaptadas en `76b4e52` |
| D2. Broker MQTT para ESP32 por LAN | Integrado históricamente con #3; configuración estática migrada y validada | MQTT QoS 1/TLS en 8883; nueva validación del broker en ejecución y acreditación de seguridad pendientes |
| E0. Diagnóstico de hardware | Integración funcional validada; caracterización parcial | #1 con PIR físico, BC548, relé/LED; retención funcional ~10 s. Calibración y medición temporal cuantitativa pendientes |
| E1. Firmware de un ESP32 | HTTPS portátil validado | #1/#2 transmiten por HTTPS a laptop mediante hotspot; antecedente de escritorio conservado abajo |
| E2. Tres nodos físicos | P13 ejecutado y observado; acreditación compartida con P15/T05 | #1 PIR físico/HTTPS; #2 PIR simulado/HTTPS; #3 PIR simulado/MQTT QoS 1/TLS |
| F. Pruebas oficiales T01–T05 y LOAD | T01–T04 PASS documentados; T05 PASS retrospectivo; LOAD documentado | T05 contrasta 336 claves SQL/serie; LOAD registra 400 respuestas 201 y correspondencia SQL. T06–T08 fueron antecedentes de planificación, no requisitos oficiales |
| G. Informe y evidencias | Informe P16 creado y versionado; demostración pendiente | [Informe_Laboratorio_2.docx](Informe_Laboratorio_2.docx), [P14_RESULTADOS.md](P14_RESULTADOS.md), [P15_RESULTADOS.md](P15_RESULTADOS.md) |
| M. Migración y colaboradores | Repositorio separado e historial pertinente conservado; validación operativa pendiente | Rutas adaptadas; validar backend/MQTT en ejecución, actualizar `PROJECT_CONTEXT.md` e invitar a dos compañeros |

### Laptop de presentación — verificación histórica 20/10

En esta fecha la laptop aún usaba el antiguo directorio `C:\Proyecto\lab1-iot-node`; la raíz operativa actual es `C:\Proyecto\lab2-software-telecomunicaciones`. XAMPP: PHP CLI **8.2.12** (PDO, pdo_mysql, JSON, OpenSSL); MariaDB **10.4.32**, cuatro tablas `devices/telemetry/configurations/events`, tres dispositivos `esp32-01/02/03`. Mosquitto **2.1.2** instalado y 1883 en escucha; ese listener no está declarado seguro. Este inventario no acredita que los servicios estén activos al 25/10.

Credenciales privadas de PDO, tokens de dispositivo y token administrativo **creados y excluidos por Git**. `check-connection.php` devuelve `OK: conexion PDO, consulta preparada nativa y sesion UTC.`

Ejecuciones exitosas **en la laptop**:
- `tests/c1-http.php`: 201, 401, 422, 200 duplicado, 409 conflicto, 400, 403 y verificación SQL.
- `tests/c2-http.php`: consulta de dispositivos, historia temporal, filtros/paginación, autorización y errores, PUT de configuración, eventos.
- `tests/d1-security.php`: acceso autorizado y rechazo de anónimo/contraseña incorrecta/violaciones ACL.
- `tests/d1-mqtt.php`: MQTT → suscriptor PHP → TelemetryService → MariaDB; duplicado, conflicto, identidad, reanudación controlada.
- `tests/c1-concurrency.php`: cuatro procesos, una creación, tres duplicados y una fila persistida.

Estas ejecuciones prueban **backend local**, no el enlace inalámbrico ni tres nodos físicos. Las pruebas pueden añadir registros a MariaDB; no borrar ni resembrar.

**Antecedente al cierre 23/10:** Apache/HTTPS y acceso en hotspot aún estaban pendientes. **Actualización 24/10:** laptop con PHP, MariaDB, Apache HTTPS y Mosquitto validada como backend portátil para los tres nodos; no confundir esta evidencia física con las suites locales del 20/10.

### Herramienta de demostración — 20–21/10, escritorio y laptop

Los tres archivos están en `scripts/`: `demo.ps1`, `demo-data.php` y `demo-acl.php`. El usuario verificó sus salidas detalladas en **escritorio y laptop** antes de la migración, con las precisiones siguientes:
1. Inventario de ejecutables y prueba real de PDO (avisa si MySQL/XAMPP está detenido).
2. Iniciar PHP HTTP en `127.0.0.1:18082` y Mosquitto D1 en `127.0.0.1:1884`, sin alterar 1883.
3. Mostrar salidas originales de pruebas HTTP, códigos y comparación de registros SQL.
4. Mostrar seguridad/ACL MQTT, ingesta, auditoría y evidencia SQL.
5. Demostrar cuatro procesos concurrentes y una única fila.
6. Consultar dispositivos, últimas mediciones, eventos, configuraciones y auditoría sin imprimir secretos.
7. Detener **solo** procesos iniciados por este menú y comprobar puertos.

**Evidencia en laptop (20/10):** opción 1 comprobó herramientas, PDO y UTC; opción 2 inició REST 18082 y MQTT 1884; opción 3 validó REST C1/C2 y SQL (6→9 filas); opción 4 validó 12 ACL, ingesta MQTT, auditoría y SQL (9→11); opción 5 confirmó concurrencia de cuatro procesos con una sola inserción (11→12); opción 6 consultó telemetría, eventos, configuraciones y auditoría; opción 7 liberó ambos puertos sin detener MariaDB. No confundir los fixtures con tres dispositivos físicos. PowerShell requirió una política temporal de ejecución solo en la sesión.

**Incidencia posterior (21/10, histórica):** sin suscriptor coincidente, tres publicaciones válidas recibieron `PUBACK=16` (MQTT 5: *No matching subscribers*). El diagnóstico de entonces exigía `0` y mostró tres discrepancias falsas; los nueve rechazos `135` fueron correctos. La corrección para aceptar 0 o 16 está conservada en el historial migrado; no volver a implementarla. La seguridad del broker del nuevo entorno debe verificarse de nuevo antes de generar una acreditación local.

**No ejecutar `schema.sql` ni `seed.sql` nuevamente**; las pruebas conservan sus fixtures.

## Inicio E1 — 22/10, primera unidad verificable

Registro histórico del inicio de E1. El despliegue y pruebas físicas posteriores se actualizan en el cierre del 23/10 siguiente; no interpretar los pendientes de este registro como ausencia actual de HTTPS en escritorio.

- Se añadió `firmware/esp32_02_http/` sin modificar `pruebas/pir_rele_10s/`.
  Contrato POST existente, `timestamp=null`, PIR alternado con
  `extensions.pir_simulated=true` y RSSI obtenido del enlace Wi-Fi real.
- Secuencia y JSON pendiente en un blob NVS antes del envío; recuperación tras
  reinicio y reintentos con contenido idéntico. Fallos permanentes detienen el
  cliente; fallos transitorios usan backoff. Sin `delay()`; E/S HTTP en tarea propia.
- HTTPS obligatorio con CA y nombre de servidor verificados; sin modo inseguro,
  fallback HTTP ni redirecciones. Plantilla Apache TLS → PHP loopback y guía
  [FIRMWARE_E1_HTTP.md](FIRMWARE_E1_HTTP.md). Proxy/certificados aún no desplegados en aquel checkpoint inicial; posteriormente probados en escritorio.
- En aquel checkpoint: compilación con core Arduino-ESP32 3.3.8 y marcadores ficticios, sin flashear.
  Entonces estaban pendientes pruebas C++ en host, TLS por LAN, envío desde placa, cortes y evidencia multinodo.
  P13/P14 actualizan integración e interrupciones el 24/10; no acreditan pruebas C++ en host.
- El historial original registra `4328fba`: corrección de diagnóstico PUBACK ya incorporada. La
  incidencia descrita arriba es histórica; no se repitieron aquí las 12 ACL.

## Antecedente histórico 23/10 — E1, diagnóstico Wi-Fi y arquitectura portátil

Los bloqueos e IP de esta sección describen el 23/10. El estado vigente figura en el checkpoint del 25/10; no reutilizar estas IP como configuración actual.

### Evidencia confirmada y límites

- Backend PHP/MariaDB y sus pruebas satisfactorias conservados; tres identidades `esp32-01`, `esp32-02`, `esp32-03` con tokens individuales. Apache HTTPS del **escritorio** configurado y probado, certificado con SAN DNS `telemetry.lab.test`, destino actual del firmware **192.168.0.3:8443**. Nada de esto acredita todavía el nuevo entorno HTTPS de laptop/hotspot.
- Firmware `firmware/esp32_02_http/esp32_02_http.ino`: HTTPS/Bearer con verificación TLS estricta; RSSI real, PIR simulado explícito, SNTP, secuencias y mensaje pendiente persistidos en NVS, reintentos, muestreo aproximado de 10 s y tarea FreeRTOS. Conexión TCP por IP conservando nombre TLS/SNI y HTTP; no depende de resolver `telemetry.lab.test` por DNS.
- En la red anterior se observaron:

  ```text
  HTTP=201 seq=37 ack=1 PIR=SIMULADO
  HTTP=201 seq=38 ack=1 PIR=SIMULADO
  HTTP=201 seq=39 ack=1 PIR=SIMULADO
  ```

  **Validación parcial:** prueba de HTTPS desde la placa en aquella configuración, no validación de la arquitectura definitiva ni de todas las condiciones de fallo.

| Entorno | Resultado confirmado | Interpretación y estado |
|---|---|---|
| Claro CastleNet Infinity601 | Ambas bandas con SSID `RODO`; 2,4 GHz/canal 11 comprobados; sketch diagnóstico conecta | No afirmar ausencia de 2,4 GHz ni fallo de placa/credenciales por defecto |
| E1 sobre Claro | `WIFI_REASON_AUTH_FAIL` 202; un intento tardó cerca de 10 min | Causa no establecida; no atribuir definitivamente a Band Steering |
| E1 tras corrección Wi-Fi | Reconexión exclusiva del core, eventos y SNTP tras IP; compila en Arduino-ESP32 3.3.8; prueba posterior >240 s sin IP | **Comportamiento Claro no solucionado**; compilación no equivale a éxito físico |
| Hotspot Android exclusivo 2,4 GHz | `AP conectado` canal 6, auth 3; IP `10.229.155.191`, tiempo hasta IP **1227 ms**; SNTP iniciado tras IP | Asociación y DHCP exitosos en aproximadamente **1,227 s**; no generalizar a tres arranques o tres nodos |
| HTTPS desde hotspot | `HTTP=-1 seq=82 ack=0 PIR=SIMULADO`; destino seguía en `192.168.0.3` | **No funcional todavía en este entorno**; verificar laptop en la misma red y ruta/acceso/TLS. `-1` no identifica por sí solo la causa. Pendiente seq 82 conservado |

### Decisión principal de demostración

**Teléfono Android (hotspot 2,4 GHz) → laptop + ESP32 #1, #2 y #3.** La laptop ejecutará Apache con HTTPS, API PHP, MariaDB y herramientas/scripts de demostración. El escritorio queda para desarrollo y **no se trasladará a la universidad**. Se descarta continuar separando bandas desde el panel Claro o depender de una intervención del operador. Evaluación final: **miércoles 28 de octubre de 2026**.

## Resultados P13–P16 al 25/10

**P13 integrado funcionalmente:** tres ESP32 físicos transmitieron simultáneamente por Wi-Fi del hotspot Android de 2,4 GHz a la laptop con PHP, MariaDB, Apache HTTPS y Mosquitto. USB se usó para alimentación y captura serie, no para transportar telemetría. La acreditación documental del ensayo conjunto se apoya en la ventana P15 y su verificación T05, sin contarla como otra campaña independiente.

| Nodo | PIR | Transporte | Captura |
|---|---|---|---|
| ESP32 #1 | Físico; relé/LED existentes | HTTPS 8443 | COM5 |
| ESP32 #2 | Simulado declarado; RSSI real | HTTPS 8443 | COM8 |
| ESP32 #3 | Simulado declarado; RSSI real | MQTT QoS 1/TLS 8883 | COM9 |

**P14 ejecutado y documentado:** [P14_RESULTADOS.md](P14_RESULTADOS.md) y el informe final describen los tres escenarios. P14-A muestra recuperación Wi-Fi funcional de los tres nodos, pero no incluye contraste SQL completo ni marcas independientes de la interrupción. P14-B muestra recuperación HTTPS de #1/#2 y continuidad MQTT de #3; su evaluación global es **INCOMPLETO** por líneas serie ilegibles y dos filas SQL de #2 sin evento serie legible. P14-C muestra recuperación MQTT/TLS de #3, incluido un pendiente, pero su evaluación global es **INCOMPLETO**: el informe conserva el hueco SQL de `seq=3496` al corte y continuidad HTTPS parcial durante la caída. Ese hueco no demuestra pérdida irreversible. No convertir estas evaluaciones en PASS ni inferir ausencia absoluta de pérdidas. PUBACK confirma al broker, no el commit SQL.

**P15 documentado:** fuente de métricas y límites en [P15_RESULTADOS.md](P15_RESULTADOS.md), secciones 1–9. La campaña está fechada el 24/10 UTC y comenzó el 23/10 en Perú.

- Campaña física: **1200,3180408 s (~20 min), 336 registros nuevos en MariaDB: 108/108/120** por #1/#2/#3. Throughput agregado aproximado **0,280 mensajes/s**; mide carga ofrecida y persistida, no capacidad máxima. El análisis inicial de P15 cotejó confirmaciones y contadores agregados; la verificación retrospectiva T05 contrastó después las **336 claves individualmente** con SQL y serie, según el informe final versionado. No representa una segunda campaña física.
- Dos campañas posteriores e independientes miden duración interna de `TelemetryService::receive()`: validación, transacción, operaciones SQL y commit. Excluyen Wi-Fi, TLS de Apache, autenticación previa, cola MQTT y auditoría posterior. **No son RTT ni latencia extremo a extremo ESP32→MariaDB.**

| Campaña PHP | Operaciones registradas | p50 (ms) | p95 (ms) | Máximo (ms) |
|---|---|---:|---:|---:|
| ~300 s, HTTP | 52 (26 por nodo #1/#2) | 8,708 | 9,869 | 48,345 |
| ~120 s, HTTP | 22 (11 por nodo #1/#2) | 7,409 | 10,728 | 48,535 |
| ~120 s, MQTT | 12 (#3) | 6,380 | 27,407 | 48,811 |

La primera ventana no registró mediciones MQTT: no significa latencia cero ni ausencia demostrada de tráfico. Todas las operaciones registradas fueron `created`; muestras pequeñas y ventanas cortas, con valores atípicos conservados y causa no identificada. No combinar percentiles entre campañas ni concluir superioridad de un transporte. Detalle por nodo, método de interpolación y limitaciones en la sección 9 de P15.

**LOAD HTTP local:** el informe final versionado documenta una pasada de 100 peticiones por nivel de concurrencia 1, 5, 10 y 20, más un calentamiento; las **400 peticiones medidas** devolvieron HTTP 201 y correspondieron a 400 filas SQL. El generador [`scripts/http-load.php`](../scripts/http-load.php) define los cuatro niveles, los límites y el contraste SQL. Los resultados corresponden al servidor PHP de desarrollo por HTTP en loopback, sin Wi-Fi, TLS ni ESP32:

| Concurrencia | HTTP 201 / SQL | Inserciones confirmadas/s | Respuesta HTTP p50 (ms) | Respuesta HTTP p95 (ms) |
|---:|---:|---:|---:|---:|
| 1 | 100 / 100 | 62,138 | 13,186 | 32,233 |
| 5 | 100 / 100 | 57,081 | 80,531 | 121,441 |
| 10 | 100 / 100 | 50,328 | 188,585 | 239,865 |
| 20 | 100 / 100 | 54,001 | 344,677 | 404,827 |

La duración interna de `TelemetryService::receive()`, el tiempo de respuesta HTTP local medido por LOAD y la latencia extremo a extremo ESP32→MariaDB son magnitudes diferentes. La última no fue medida; una sola pasada por nivel tampoco determina capacidad máxima.

**P16:** [Informe_Laboratorio_2.docx](Informe_Laboratorio_2.docx) fue incorporado en `6c42f85`. Su elaboración no acredita la entrega ni la demostración presencial, previstas para el **28/10/2026**.

**Anomalía COM5:** 63 650 repeticiones de `HTTP=201 seq=292` antes de la ventana física; incompatibles con el caudal serie y de origen no diagnosticado. No representan miles de solicitudes ni inserts. Corregir/verificar el registrador antes de una campaña nueva; conservar logs originales. La anomalía no invalida por sí sola el incremento SQL de los 20 min.

### Estado de prioridades y continuidad

| Etapa | Estado al 25/10 | Límite o seguimiento |
|---|---|---|
| P01–P08: hotspot, laptop y HTTPS | Integración observada en P13/P15 | Conservar TLS, CA/SAN, autenticación y BD; validar mínimamente el arranque desde la nueva raíz |
| P09: localización ante cambio de IP | Revisión documental pendiente | P15 no acredita reproducibilidad ante cualquier cambio de IP |
| P10: tres RESET/EN y tiempos hasta IP | Sin evidencia específica en las fuentes consultadas | No bloquear retrospectivamente la integración observada |
| P11: integración física #1 | Validada funcionalmente | Caracterización cuantitativa y detalles de cableado siguen limitados |
| P12 y D2: #3 por MQTT/TLS | Integrado históricamente | Revalidar broker y seguridad operativa tras la migración |
| P13 | Integración funcional | Acreditación compartida con P15/T05; sin campaña independiente que repetir |
| P14-A/B/C | Recuperación funcional observada y documentada | A con contraste SQL limitado; B/C **INCOMPLETO** |
| P15 y T05 | Campaña física y contraste retrospectivo documentados | 336 filas/claves; no extrapolar capacidad máxima ni ausencia global de pérdidas |
| LOAD | Prueba HTTP local documentada | 400 respuestas 201 con correspondencia SQL; no mide HTTPS físico |
| P16 | Informe versionado | Entrega y demostración presencial del 28/10 pendientes |

**Prioridades vigentes:** (1) validación operativa mínima del backend y MQTT desde el repositorio independiente cuando estén disponibles los servicios; (2) revalidación de seguridad MQTT y nueva acreditación `security.json`; (3) preparación de la demostración presencial del 28/10; (4) actualización coordinada de `PROJECT_CONTEXT.md` en el repositorio de continuidad; (5) revisión posterior de la retirada de `lab2/` del repositorio antiguo, sin operaciones destructivas ahora; (6) invitación pendiente a dos compañeros como colaboradores, sin asignarles tareas. No repetir P13, P14, P15 ni las campañas experimentales documentadas.

## Hardware real — etapa E0

- **Tres ESP32 disponibles y comprobadas mediante diagnósticos básicos**; una identificada como NodeMCU ESP-32S V1.1 (38 pines). La integración simultánea se observó en P13 y se acredita documentalmente mediante P15/T05, sin contarla como una campaña adicional.
- Relé JQC-3FF-S-Z 5V: la conexión directa GPIO26 no liberaba correctamente con nivel HIGH de 3.3 V. Solución **probada aisladamente**: BC548 NPN, resistencia 1 kΩ GPIO26→base, emisor a GND y colector a IN del módulo; relé VCC 5V y GND común. **Prueba inicial histórica:** contactos COM/NO/NC sin carga. **Montaje posterior confirmado por el usuario:** LED de baja tensión con resistencia de **330 Ω** para representar iluminación; no describir el montaje actual como necesariamente sin carga.
- PIR compatible HC-SR501/BISS0001, OUT→GPIO27: prueba independiente PIR/relé completada con **retención funcional aproximada de 10 s**, renovada ante nueva detección. En `firmware/pruebas/pir_rele_10s/pir_rele_10s/pir_rele_10s.ino`, `HOLD_TIME_MS=10000UL`; `lastMotionTime` se renueva mientras OUT está HIGH y el apagado exige OUT LOW y al menos 10000 ms desde la última detección registrada. Esto no caracteriza cuantitativamente el tiempo real ni el temporizador interno del PIR. Calibración final, rango y caracterización temporal pendientes; no hay prueba suficiente de defecto.
- ESP32 #1 con PIR físico, BC548, relé y LED para simular iluminación; integración funcional con firmware y multinodo observada en P13; caracterización cuantitativa pendiente. El sketch de prueba controla GPIO27 (PIR) y GPIO26 (relé), pero no documenta el cableado del LED, el borne conmutado COM/NO/NC ni su alimentación exacta: **pendientes de confirmación**, sin inferirlos de esquemas históricos de Lab1. No equiparar prueba funcional independiente con validación final.
- **Tres ESP32 físicos son obligatorios**, pero solo ESP32 #1 necesita PIR, relé y LED físicos. El profesor **autorizó simular el valor PIR** de ESP32 #2 y #3; no comprar periféricos duplicados. Los tres ESP32 deberán enviar mensajes reales por la red con identidad propia; etiquetar los valores PIR simulados y no presentarlos como mediciones reales.
- Seguridad: solo USB/baja tensión; desconectar antes de modificar conexiones; no conectar red eléctrica a bornes del relé.

## Entrega del 28 de octubre

La demostración presencial sigue programada para el miércoles 28/10/2026 con laptop y tres ESP32. El informe ya está elaborado y versionado; falta la presentación. El entorno previsto sigue siendo el hotspot Android y la laptop. El cliente de computadora no representa una placa física; #2 y #3 usan PIR simulado declarado. La anomalía histórica COM5 queda documentada en P15 y no justifica repetir las campañas concluidas.

## M. Repositorio independiente y colaboración

- El contenido del antiguo `lab2/` de `jmamanilari-cyber/lab1-iot-node` pasó a la raíz de `FAJIFBEFJHA/lab2-software-telecomunicaciones` mediante `git subtree split --prefix=lab2`. Se conservó el historial pertinente, con SHA reconstruidos. El informe final está en el commit migrado `6c42f85`, equivalente al original `6a993ff`; `76b4e52` adaptó rutas a la raíz independiente. En la última comprobación previa a esta edición, `main` estaba sincronizada con `origin/main` y el árbol limpio. El repositorio original permanece intacto; eliminar su carpeta `lab2/` no está autorizado todavía.
- Se transfirieron **10 archivos privados principales**, **ocho auxiliares de Mosquitto** y el estado MQTT (`broker.db`, `inbox.jsonl`, `cursor`, `audit.jsonl`). Se confirmó SHA256 de auxiliares y estado copiados. Las rutas absolutas se ajustaron en `mqtt.local.php`, `mqtt-private/broker.conf` y `mqtt-d2/broker.conf`; la sintaxis PHP de `mqtt.local.php` es válida. La validación estática D2 fue satisfactoria para certificados TLS, ACL y persistencia. Estos elementos están excluidos de Git; no se publican contenidos ni rutas privadas detalladas.
- `security.json` no se transfirió: la acreditación debe regenerarse tras validar la seguridad del broker en ejecución. El cursor coincidía con el tamaño de `inbox.jsonl`, sin bytes posteriores pendientes de lectura. La migración **no está validada completamente en ejecución**: falta comprobar backend/MQTT desde el nuevo repositorio y la nueva verificación PDO, pues MariaDB estaba detenido en la última comprobación.
- Quedan pendientes la actualización coordinada de `PROJECT_CONTEXT.md` en el repositorio de continuidad, la revisión posterior de la retirada de `lab2/` del repositorio antiguo y la invitación a los dos compañeros como colaboradores. No asignarles tareas. No realizar operaciones destructivas, commits ni push por esta actualización documental.

## Reglas de implementación y cierre

- `node_id` vinculado a credencial, `seq` único incluso tras reinicio, reintentos con contenido inalterado; `timestamp` Unix UTC válido o `null`, `received_at` lo asigna el servidor; PIR `value` 0/1, `rssi` -127..0/null. Distinguir telemetría PIR simulada (nodos #2/#3 autorizados) de mediciones PIR físicas (#1).
- MQTT y REST comparten el mismo `TelemetryService` y transacción PDO. **D1 solo 127.0.0.1:1884**; D2 LAN/TLS en 8883 integrado con #3. QoS1/PUBACK del cliente empleado puede anteceder commit SQL; no se garantiza pérdida cero ante corte brusco.
- No publicar `config/database.local.php`, `test-tokens.local.json`, `admin.local.json`, `mqtt.local.php`, `mqtt-private/`, credenciales Wi-Fi, certificados privados, colas ni auditoría privadas. Conservar la separación entre configuración local y archivos versionados.
- Jerarquía de evidencia: código y pruebas reales > guía oficial > resultados documentados > decisiones confirmadas > ROADMAP. No repetir etapas completadas ni presentar una revisión documental como una nueva prueba operativa.
- La guía PDF no está confirmada como archivo versionado en este repositorio. El informe final sí está versionado. Revisar el estado del árbol antes de cualquier operación Git futura; esta actualización no hace commits ni push.
