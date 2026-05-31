# ADR 001: Procesamiento Asíncrono y Mensajería Administrada en la Nube (PaaS)

## 1. Estado
**Aceptado**

## 2. Contexto
El proceso de evaluación y otorgamiento de un crédito hipotecario digital en el Sistema de Créditos Hipotecarios (SCH) requiere integraciones críticas con el Core Bancario y con centrales de riesgo externas (como Equifax o Sentinel). 

Estas llamadas a servicios externos introducen dos problemas principales:
* **Alta e impredecible latencia:** Las consultas a burós de crédito externos pueden demorar entre 5 y 10 segundos por solicitud, dependiendo de la carga del proveedor.
* **Riesgo de indisponibilidad:** Si la central de riesgo externa sufre una caída temporal, todo el proceso de negocio web se interrumpe de inmediato.

Procesar esto de manera síncrona bloquearía los hilos de ejecución del servidor, degradando la experiencia del usuario y saturando los recursos de infraestructura bajo ráfagas de tráfico.

## 3. Alternativas Evaluadas

### Alternativa 1: Procesamiento Síncrono Tradicional (Bloqueante)
Consiste en realizar todas las integraciones dentro del mismo ciclo de vida de la petición HTTP (`POST /v1/solicitudes`).
* **Pros:** Simplicidad de desarrollo y consistencia inmediata en la interfaz gráfica.
* **Contras:** Nula resiliencia ante caídas de terceros. El rendimiento del API se degrada al ritmo del proveedor más lento. Inviable para canales digitales bancarios.

### Alternativa 2: Arquitectura Dirigida por Eventos Autogestionada (RabbitMQ on-premise/IaaS)
Consiste en implementar asincronía desplegando y administrando un clúster de RabbitMQ en máquinas virtuales.
* **Pros:** Resuelve el problema de concurrencia y resiliencia al desacoplar los procesos.
* **Contras:** Alta carga operativa (Ops). El equipo técnico asume la responsabilidad del parchado de servidores, monitoreo del sistema operativo, escalamiento del clúster y recuperación ante desastres (Disaster Recovery).

### Alternativa 3: Arquitectura Dirigida por Eventos con Infraestructura PaaS (Azure Service Bus)
Consiste en implementar el mismo patrón asíncrono, pero delegando la infraestructura a un servicio administrado en la nube de nivel empresarial (Platform as a Service).
* **Pros:** Máxima resiliencia sin carga operativa. Alta disponibilidad respaldada por el SLA de Microsoft. Soporte nativo y optimizado para .NET 8. Cumplimiento de estándares de seguridad financieros e integración transparente mediante el protocolo AMQP 1.0.
* **Contras:** Costo asociado al consumo de servicios en la nube. Posible *vendor lock-in* (dependencia del proveedor tecnológico).

## 4. Decisión
Adoptar la **Alternativa 3 (Arquitectura Dirigida por Eventos con Azure Service Bus)**. 

Priorizamos la **alta disponibilidad**, la **escalabilidad** y la **reducción del TCO (Costo Total de Propiedad)** operativo. Al ser una solución financiera, delegar la administración del *Message Broker* a un proveedor de nube maduro permite que el equipo de ingeniería se enfoque en aportar valor mediante las reglas de negocio crediticio, en lugar de administrar servidores de mensajería.

## 5. Consecuencias

* **Consecuencias Positivas:**
  * **Aislamiento de Fallos y SLA:** Una caída de Equifax no inhabilita la captura de solicitudes. Azure Service Bus garantiza el encolamiento seguro de los prospectos hasta que el proveedor externo se recupere.
  * **Escalabilidad Elástica:** Permite escalar horizontalmente el *Worker* de .NET 8 para procesar las colas más rápido durante campañas inmobiliarias sin saturar el Backend API.
  * **Cero Carga Administrativa:** Eliminación de los costos ocultos de mantenimiento de infraestructura.

* **Consecuencias Negativas (Mitigaciones obligatorias):**
  * **UX Asíncrona:** El equipo de Frontend (SPA) debe implementar un patrón de *polling* para consultar el estado final (`GET /v1/solicitudes/{id}/status`) mientras muestra estados de transición al usuario.
  * **Mitigación de Vendor Lock-in:** Para evitar un acoplamiento duro al SDK nativo de Azure, se utilizará el framework **MassTransit** en la capa de aplicación. Esto permitirá cambiar el broker subyacente en el futuro con un mínimo impacto en el código fuente.
