# Diagrama de Secuencia UML - Proceso Asíncrono de Evaluación

Este diagrama detalla la interacción temporal y el flujo de mensajes entre los distintos componentes de la arquitectura para resolver el caso de uso principal: **"Postulación y Evaluación Automatizada de un Crédito Hipotecario"**.

El flujo está diseñado de manera asíncrona para garantizar la alta disponibilidad del canal digital frente a sistemas de alta latencia o caídas de proveedores externos.

## Diagrama de Secuencia (Mermaid)

```mermaid
sequenceDiagram
    autonumber
    actor Cliente as Cliente (Usuario)
    participant SPA as SPA Web (Angular/React)
    participant API as Backend API (.NET 8)
    database DB as Base de Datos (PostgreSQL)
    participant Broker as Message Broker (RabbitMQ)
    participant Worker as Worker de Evaluación
    participant Ext as Central de Riesgo (Equifax)

    %% Fase 1: Recepción de la Solicitud
    Note over Cliente, API: Fase 1: Registro y Liberación del Hilo HTTP
    Cliente->>SPA: Completa formulario y da clic en "Evaluar"
    SPA->>API: POST /v1/solicitudes (Datos Financieros)
    activate API
    API->>DB: INSERT INTO solicitudes (Estado: 'Pendiente')
    API->>Broker: Publicar Evento 'SolicitudRegistradaEvent'
    API-->>SPA: HTTP 202 Accepted (Retorna SolicitudId)
    deactivate API
    
    SPA-->>Cliente: Muestra pantalla: "Estamos evaluando tu perfil..."

    %% Fase 2: Procesamiento en Segundo Plano
    Note over Broker, Ext: Fase 2: Procesamiento Asíncrono (Background)
    Broker->>Worker: Despacha 'SolicitudRegistradaEvent'
    activate Worker
    
    Worker->>Ext: GET /v1/scores/{dni} (Consulta deudas)
    activate Ext
    Note over Worker, Ext: Integración con alta latencia (5-10 segundos)
    Ext-->>Worker: Retorna Historial y Score Crediticio
    deactivate Ext

    Note over Worker: El Worker ejecuta las Reglas de Negocio <br/> (Cálculo de capacidad de pago e ingresos)
    
    alt Cliente Califica
        Worker->>DB: UPDATE solicitudes SET estado = 'PreAprobado' WHERE id = SolicitudId
    else Cliente No Califica
        Worker->>DB: UPDATE solicitudes SET estado = 'Rechazado' WHERE id = SolicitudId
    end
    deactivate Worker

    %% Fase 3: Consulta de Estado (Polling)
    Note over Cliente, DB: Fase 3: Sincronización del Frontend (Polling)
    loop Cada 3 segundos hasta finalizar
        SPA->>API: GET /v1/solicitudes/{id}/status
        activate API
        API->>DB: SELECT estado FROM solicitudes WHERE id = id
        DB-->>API: Retorna Estado ('PreAprobado')
        API-->>SPA: HTTP 200 OK (Estado: 'PreAprobado')
        deactivate API
    end

    SPA-->>Cliente: Muestra: "¡Felicidades! Tu crédito hipotecario fue Pre-Aprobado"
