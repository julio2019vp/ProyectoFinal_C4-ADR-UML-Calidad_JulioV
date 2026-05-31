# Reporte de Calidad Arquitectónica y Trade-offs

## 1. Estrategia de Aseguramiento de Calidad (QA)
La estrategia para el **Sistema de Créditos Hipotecarios (SCH)** garantiza la robustez mediante:
* **Pruebas Automatizadas:** Pirámide de pruebas (Unitarias con xUnit, Integración con Testcontainers).
* **Análisis Estático:** Uso de SonarQube para detectar vulnerabilidades y deuda técnica.
* **Observabilidad:** Implementación de OpenTelemetry y Application Insights.
* **Definición de "Hecho" (DoD):** Ningún componente pasa a producción sin cobertura >80% y validación de seguridad.

## 2. Análisis de Trade-offs Arquitectónicos
Toda decisión técnica implica un compromiso. A continuación, los trade-offs más críticos del diseño:

| Decisión Arquitectónica | Ventaja (Pros) | Sacrificio (Cons / Trade-off) |
| :--- | :--- | :--- |
| **Arquitectura Basada en Eventos (Azure Service Bus)** | Alta disponibilidad y resiliencia ante fallos de sistemas externos. | Mayor complejidad en el debugging y necesidad de UX asíncrona. |
| **Uso de Base de Datos PaaS (PostgreSQL)** | Menor costo operativo y gestión automática (respaldos/parches). | Ligera dependencia del proveedor (Vendor Lock-in). |
| **API Gateway (Azure APIM)** | Centralización de seguridad, throttling y analítica de tráfico. | Latencia adicional (mínima) por el salto de red adicional. |
| **Migración a .NET 8 (LTS)** | Estabilidad empresarial, soporte extendido y ecosistema maduro. | Se renuncia a las funcionalidades de versiones experimentales (.NET 10). |

## 3. Justificación de los Trade-offs
El objetivo principal del SCH es la **Resiliencia**. Sacrificamos la simplicidad del código (complejidad de la asincronía) para ganar robustez. Preferimos procesar las solicitudes de forma desacoplada a través de **Azure Service Bus** para evitar que una caída en el Core Bancario detenga la recepción de nuevas solicitudes. Asimismo, optamos por **servicios PaaS** para delegar la carga operativa a Microsoft, permitiendo que el equipo se enfoque en el negocio y no en la administración de servidores, aceptando el trade-off de la dependencia del proveedor.
