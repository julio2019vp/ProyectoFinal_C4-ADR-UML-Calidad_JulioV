# Sistema de Evaluación y Otorgamiento de Créditos Hipotecarios (SCH)

Este repositorio contiene la documentación arquitectónica viva correspondiente al proyecto final del curso. El sistema automatiza, agiliza y evalúa de manera digital la pre-aprobación de créditos hipotecarios en el entorno bancario, reduciendo los tiempos de respuesta tradicionales.

---

## 1. Objetivo y Alcance del Sistema
El **Sistema de Créditos Hipotecarios (SCH)** está diseñado para que un cliente solicitante pueda registrar su postulación a un crédito hipotecario desde los canales digitales del banco. 

El sistema se encarga de:
* Recibir y validar los datos financieros del postulante de forma segura.
* Conectarse de manera asíncrona con servicios internos del banco y proveedores externos de riesgo.
* Procesar las reglas de negocio financieras para determinar si el cliente califica.
* Emitir una carta de pre-aprobación digital en tiempo real o diferido según la disponibilidad de los servicios externos.

*Nota: Este proyecto se enfoca estrictamente en la capacidad de análisis, diseño y documentación de la arquitectura de software, abstrayendo la implementación de código según los lineamientos de la evaluación.*

---

## 2. Estructura de la Documentación Viva
Para facilitar la revisión y cumplir con el estándar de arquitectura como código, la documentación se encuentra organizada y segmentada en las siguientes secciones:

### 🌐 Visualización Arquitectónica (Modelo C4)
El diseño del sistema se ha estructurado utilizando los niveles del Modelo C4 para permitir diferentes niveles de zoom técnico:
1. **[Nivel 1: Contexto del Sistema](modelo-c4/nivel-1-contexto.md):** Muestra los límites del sistema y cómo interactúa con el Cliente, el Core Bancario y las Centrales de Riesgo externas.
2. **[Nivel 2: Contenedores](modelo-c4/nivel-2-contenedor.md):** Detalla las aplicaciones, APIs, servicios en segundo plano y bases de datos (.NET 8, RabbitMQ, PostgreSQL) que dan soporte a la solución.
3. **[Nivel 3: Componentes](modelo-c4/nivel-3-componente.md):** Hace un zoom interno en el Backend API para mostrar la distribución de responsabilidades y capas de diseño.

### 🔄 Comportamiento Dinámico (UML)
* **[Diagrama de Secuencia UML](uml/diagrama-secuencia.md):** Describe el flujo temporal y la interacción de los componentes durante el caso de uso principal de evaluación asíncrona de un crédito.

### ⚖️ Decisiones y Atributos de Calidad
* **[Architecture Decision Records (ADRs)](adr/ADR-0
