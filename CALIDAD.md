# Reporte de Calidad Arquitectónica y Trade-offs

Este documento detalla los Atributos de Calidad priorizados para el **Sistema de Créditos Hipotecarios (SCH)**. Cada atributo está estructurado bajo un escenario arquitectónico concreto, vinculando los objetivos de negocio con las tácticas y decisiones de diseño adoptadas.

---

## 1. Matriz de Atributos de Calidad

| Atributo de Calidad | Estímulo (Qué pasa) | Táctica Arquitectónica Aplicada | Componente Responsable |
| :--- | :--- | :--- | :--- |
| **Disponibilidad** *(Availability)* | La Central de Riesgo externa (Equifax) se cae por 15 minutos durante horas de alta transaccionalidad. | **Desacoplamiento Asíncrono + Patrón Retry:** El API recibe la solicitud y responde de inmediato. El Worker encola el mensaje y reintenta la comunicación con un backoff exponencial sin afectar al cliente. | `RabbitMQ`, `Evaluation Worker` |
| **Escalabilidad** *(Scalability)* | El tráfico de solicitudes de crédito se triplica durante una campaña inmobiliaria masiva de fin de semana. | **Escalamiento Horizontal de Consumidores (Compensación de Carga):** El API Gateway y el Backend API permanecen ligeros. Se incrementan las instancias del Worker para procesar las colas en paralelo. | `RabbitMQ`, `Evaluation Worker (.NET)` |
| **Seguridad** *(Security)* | Un usuario malintencionado intenta interceptar datos financieros sensibles o realizar operaciones no autorizadas. | **Cifrado de Extremo a Extremo + Control de Acceso Basado en Roles (RBAC):** Uso de TLS 1.3 en tránsito y encriptación de columnas sensibles en reposo. Autenticación centralizada mediante tokens JWT con scopes definidos. | `API Gateway`, `PostgreSQL (TDE)` |

---

## 2. Detalle de Escenarios de Calidad

### Escenario 1: Disponibilidad (Manejo de Fallos de Terceros)
* **Fuente del Estímulo:** Sistema Externo (Central de Riesgo).
* **Estímulo:** El servicio web de consulta de score crediticio no responde (Timeout / HTTP 503).
* **Entorno:** Operación normal en hora punta (14:00 - 18:00 hrs).
* **Artefacto afectado:** Worker de Evaluación.
* **Respuesta del Sistema:** El sistema no bloquea la experiencia del usuario. La solicitud del cliente queda registrada con estado `En Evaluación`. El mensaje se mantiene a salvo en la cola de contingencia.
* **Métrica de Respuesta:** Cero (0) solicitudes perdidas. El sistema procesa de forma diferida el 100% de las evaluaciones encoladas tan pronto como el proveedor externo restablece su servicio.

### Escenario 2: Escalabilidad y Rendimiento (Carga Variable)
* **Fuente del Estímulo:** Clientes concurrentes en la plataforma web.
* **Estímulo:** Incremento abrupto de 50 a 1,500 solicitudes de evaluación por minuto debido
