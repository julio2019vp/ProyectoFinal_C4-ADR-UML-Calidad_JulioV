# Sistema de Evaluación y Otorgamiento de Créditos Hipotecarios (SCH)

[cite_start]Este repositorio contiene la documentación arquitectónica viva correspondiente al proyecto final del curso[cite: 7, 24, 28]. El sistema automatiza, agiliza y evalúa de manera digital la pre-aprobación de créditos hipotecarios en el entorno bancario, reduciendo los tiempos de respuesta tradicionales.

---

## 1. Objetivo y Alcance del Sistema
El **Sistema de Créditos Hipotecarios (SCH)** está diseñado para que un cliente solicitante pueda registrar su postulación a un crédito hipotecario desde los canales digitales del banco. 

El sistema se encarga de:
* Recibir y validar los datos financieros del postulante de forma segura.
* Conectarse de manera asíncrona con servicios internos del banco y proveedores externos de riesgo.
* Procesar las reglas de negocio financieras para determinar si el cliente califica.
* Emitir una carta de pre-aprobación digital en tiempo real o diferido según la disponibilidad de los servicios externos.

[cite_start]*Nota: Este proyecto se enfoca estrictamente en la capacidad de análisis, diseño y documentación de la arquitectura de software, abstrayendo la implementación de código según los lineamientos de la evaluación[cite: 11, 12].*

---

## 2. Estructura de la Documentación Viva
[cite_start]Para facilitar la revisión y cumplir con el estándar de arquitectura como código, la documentación se encuentra organizada y segmentada en las siguientes secciones[cite: 24, 26, 28]:

### 🌐 Visualización Arquitectónica (Modelo C4)
[cite_start]El diseño del sistema se ha estructurado utilizando los niveles del Modelo C4 para permitir diferentes niveles de zoom técnico[cite: 8, 17]:
1. [cite_start]**[Nivel 1: Contexto del Sistema](modelo-c4/nivel-1-contexto.md):** Muestra los límites del sistema y cómo interactúa con el Cliente, el Core Bancario y las Centrales de Riesgo externas[cite: 17].
2. [cite_start]**[Nivel 2: Contenedores](modelo-c4/nivel-2-contenedor.md):** Detalla las aplicaciones, APIs, servicios en segundo plano y bases de datos (.NET 8, RabbitMQ, PostgreSQL) que dan soporte a la solución[cite: 17, 24].
3. [cite_start]**[Nivel 3: Componentes](modelo-c4/nivel-3-componente.md):** Hace un zoom interno en el Backend API para mostrar la distribución de responsabilidades y capas de diseño[cite: 17].

### 🔄 Comportamiento Dinámico (UML)
* [cite_start]**[Diagrama de Secuencia UML](uml/diagrama-secuencia.md):** Describe el flujo temporal y la interacción de los componentes durante el caso de uso principal de evaluación asíncrona de un crédito[cite: 21].

### ⚖️ Decisiones y Atributos de Calidad
* [cite_start]**[Architecture Decision Records (ADRs)](adr/ADR-001-procesamiento-asincrono.md):** Registro formal que justifica técnicamente la adopción de una arquitectura dirigida por eventos y procesamiento asíncrono, analizando sus alternativas y consecuencias[cite: 8, 21].
* [cite_start]**[Calidad Arquitectónica](CALIDAD.md):** Análisis detallado de los atributos clave del sistema (Disponibilidad, Escalabilidad y Seguridad) vinculados directamente con las tácticas de diseño elegidas[cite: 8, 21].

---

## 3. Criterios de Sustentación Técnicas Aplicados
[cite_start]La solución documentada en este repositorio cumple con los siguientes requisitos exigidos por la rúbrica de evaluación[cite: 24, 27]:
* [cite_start]**Uso de Modelos Estándar:** Implementación de diagramas C4 y diagramas de secuencia UML claros, coherentes y realistas para el rubro financiero[cite: 8, 17, 21].
* [cite_start]**Justificación Arquitectónica:** Decisiones de diseño respaldadas por trade-offs e impactos en el negocio mediante ADRs[cite: 11, 21].
* [cite_start]**Documentación Viva:** Diagramas integrados mediante lenguaje descriptivo (Mermaid) listos para mantenimiento y evolución continua[cite: 26].
