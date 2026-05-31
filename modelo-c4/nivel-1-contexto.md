# Modelo C4 - Nivel 1: Contexto del Sistema

Este diagrama de contexto establece los límites del **Sistema de Créditos Hipotecarios (SCH)**, mostrando cómo interactúa con los usuarios del banco y con los sistemas periféricos (tanto internos como externos) que forman parte del ecosistema financiero.

## Diagrama de Contexto Visual

![Diagrama de Contexto](https://www.plantuml.com/plantuml/svg/bLDDRjim4BxxL-WaN46TWej3aB0KYYbO41WJm0DkGL1E6XU88X_Nn5UoD_JlzF12tX-fFWe3eX8X61H7B25u08A9eD3gWl8B5h3aX34Z_5H0u1H2QW8X0H7Z1b28W8X2q4H0u1H2QW8X0H7Z1b28W8X2q4H0u1H2QW8X0H7Z1b28W8X2q4H0u1H2QW8X0H7Z1b28W8X2q4H0u1H2QW8X0H7Z1b28W8X2q4H0u1H2QW8X0H7Z1b28W8X2q4H0u1H2QW8X0H7Z1b28W8X2q4H0u1H2QW8X0H7Z1b28W8X2q4H0u1H2QW8X0H7Z1b28W8X2q4H0u1H2QW8X0H7Z1b28W8X2q4H0u1H2QW8X0H7Z1b28W8X2q4H0u1H2QW8X0H7Z1b28W8X2q4H0u1H2QW8X0H7Z1b28W8X2q4H0u1)
*(Nota: El diagrama superior se genera dinámicamente a partir del código fuente)*

---

## Código Fuente de la Arquitectura (PlantUML)

```plantuml
@startuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Context.puml

LAYOUT_WITH_LEGEND()

title Diagrama de Contexto - Sistema de Créditos Hipotecarios (SCH)

Person(cliente, "Cliente Solicitante", "Usuario del banco que desea adquirir un crédito de vivienda desde canales digitales.")

System(sch, "Sistema de Créditos Hipotecarios (SCH)", "Permite la postulación web, validación financiera automatizada y generación de la carta de pre-aprobación de forma asíncrona.")

System_Ext(core, "Core Bancario", "Sistema transaccional central del banco. Provee datos demográficos del cliente y registra la obligación financiera final.")

System_Ext(centralRiesgo, "Central de Riesgo (Equifax/Sentinel)", "Servicio externo regulado encargado de retornar el score crediticio consolidado y el historial de deudas vigentes.")

Rel(cliente, sch, "Solicita evaluación de crédito", "HTTPS / JSON")
Rel(sch, centralRiesgo, "Consulta historial y score", "REST / JSON API")
Rel(sch, core, "Valida cuentas y registra aprobación", "SOAP / XML")
@enduml
