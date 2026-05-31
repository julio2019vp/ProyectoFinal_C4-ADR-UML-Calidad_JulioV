# ADR 001: Procesamiento Asíncrono de Evaluaciones Crediticias Hipotecarias

## 1. Estado
**Aceptado**

## 2. Contexto
El proceso de evaluación y otorgamiento de un crédito hipotecario digital en el Sistema de Créditos Hipotecarios (SCH) requiere integraciones críticas con el Core Bancario (para validar cuentas del cliente) y con centrales de riesgo externas (como Sentinel o Equifax). 

Estas llamadas a servicios externos introducen dos problemas principales:
* **Alta e impredecible latencia:** Las consultas a burós de crédito externos pueden demorar entre 5 y 10 segundos por solicitud, dependiendo de la carga del proveedor.
* **Riesgo de indisponibilidad:** Si la central de riesgo externa sufre una caída temporal, todo el proceso de negocio se interrumpe de inmediato.

Si procesamos este flujo de manera síncrona (bloqueante) directamente en el hilo de la solicitud HTTP principal:
1. El usuario experimentará lentitud extrema en el navegador (pantalla de carga congelada).
2. El servidor web agotará rápidamente su pool de hilos de ejecución bajo ráfagas de tráfico, provocando la denegación del servicio y tirando abajo el API de cara al cliente.

## 3. Alternativas Evaluadas

### Alternativa 1: Procesamiento Síncrono Tradicional (Bloqueante)
Consiste en realizar las llamadas a la Central de Riesgo y al Core Bancario dentro del mismo ciclo de vida de la petición HTTP (`POST /v1/solicitudes`) y no responder al cliente hasta tener el resultado final.
* **Pros:** Más simple de implementar a nivel de desarrollo de software; no requiere infraestructura adicional ni gestión de mensajería; consistencia inmediata de los datos en la pantalla del usuario.
* **Contras:** Nula resiliencia ante caídas de terceros. Acoplamiento temporal severo. El rendimiento general del sistema se degrada al ritmo del proveedor externo más lento, saturando los recursos del servidor.

### Alternativa 2: Arquitectura Dirigida por Eventos y Procesamiento Asíncrono (RabbitMQ)
Consiste en separar la recepción de la solicitud de la evaluación real de negocio. El Backend API recibe los datos del cliente, los persiste en la base de datos PostgreSQL con un estado `Pendiente`, publica un evento llamado `SolicitudRegistradaEvent` en un Message Broker (RabbitMQ) y responde de inmediato al cliente con un código de estado `HTTP 202 Accepted`. Un servicio en segundo plano (Worker en .NET) consume el mensaje de la cola de forma aislada y ejecuta las integraciones pesadas.
* **Pros:** Máxima disponibilidad y resiliencia del sistema. Si la central de riesgo se cae, las solicitudes no se pierden; se quedan seguras en la cola de RabbitMQ y se reintentan automáticamente mediante políticas de reintento (*Retry Pattern*). El API web se libera en milisegundos, soportando picos masivos de tráfico.
* **Contras:** Incrementa la complejidad técnica del ecosistema de software. Requiere desplegar y monitorear infraestructura adicional (RabbitMQ) y manejar escenarios de consistencia eventual. El frontend deberá implementar consultas periódicas (*polling*) o WebSockets para conocer cuándo terminó el proceso en segundo plano.

## 4. Decisión
Adoptar la **Alternativa 2 (Arquitectura Dirigida por Eventos con RabbitMQ)**. 

Dado que estamos diseñando una solución para el rubro financiero, priorizaremos la **alta disponibilidad**, la **escalabilidad** y la **resiliencia del negocio** sobre la simplicidad del código de programación. El desacoplamiento táctico protegerá los canales digitales del banco frente a la inestabilidad de los sistemas legados o externos.

## 5. Consecuencias

* **Consecuencias Positivas:**
  * **Aislamiento de Fallos:** Una caída del Core Bancario o de Equifax no inhabilita la opción de que nuevos clientes sigan registrando sus solicitudes en el portal web.
  * **Escalabilidad Elástica:** Permite escalar horizontalmente el Worker en .NET (encender más instancias del servicio) para procesar los mensajes de la cola más rápido durante campañas masivas de marketing, sin necesidad de tocar o alterar el API principal.
  * **Garantía de Entrega:** Los mensajes no procesados por errores fatales se derivan a una cola de descarte (*Dead Letter Queue - DLQ*) para su posterior análisis y reprocesamiento manual, evitando la pérdida de prospectos comerciales.

* **Consecuencias Negativas (Mitigaciones obligatorias):**
  * **Experiencia de Usuario (UX) Asíncrona:** El equipo de Frontend (Single Page Application) se ve obligado a diseñar una interfaz con estados intermedios ("Estamos evaluando tus datos...") y consumir un endpoint de consulta de estado (`GET /v1/solicitudes/{id}/status`) mediante *polling* cada 3 segundos hasta obtener el veredicto final.
  * **Idempotencia:** El Worker debe ser diseñado para ser estrictamente idempotente; si consume dos veces el mismo mensaje por un reintento de red, no debe duplicar la evaluación ni generar registros erróneos en la base de datos.
