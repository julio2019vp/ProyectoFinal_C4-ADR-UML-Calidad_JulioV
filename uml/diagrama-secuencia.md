# Diagrama de Secuencia UML - Proceso Asíncrono de Evaluación

Este diagrama detalla la interacción temporal y el flujo de mensajes entre los distintos componentes de la arquitectura para resolver el caso de uso principal: **"Postulación y Evaluación Automatizada de un Crédito Hipotecario"**.

El flujo está diseñado de manera asíncrona para garantizar la alta disponibilidad del canal digital frente a sistemas de alta latencia o caídas de proveedores externos.

## Diagrama de Secuencia (Mermaid)

```mermaid
sequenceDiagram
    autonumber
    actor Cliente as Cliente
    participant APIM as Azure API Management
    participant API as Backend API (.NET 8)
    participant DB as Azure DB for PostgreSQL
    participant SB as Azure Service Bus
    participant Worker as Worker de Evaluación
    participant Ext as Central de Riesgo (Equifax)
    participant CB as Core Bancario

    %% Fase 1: Recepción de la Solicitud
    Note over Cliente, API: Fase 1: Registro y Liberación del Hilo HTTP
    Cliente->>APIM: POST /v1/solicitudes (Datos)
    APIM->>API: Enruta petición
    activate API
    API->>DB: INSERT INTO solicitudes (Pendiente)
    API->>SB: Publicar Evento SolicitudRegistrada
    API-->>APIM: HTTP 202 Accepted
    APIM-->>Cliente: Retorna SolicitudId
    deactivate API
    
    Cliente->>APIM: Muestra: "Estamos evaluando tu perfil..."

    %% Fase 2: Procesamiento en Segundo Plano
    Note over SB, CB: Fase 2: Procesamiento Asíncrono (Background)
    SB->>Worker: Despacha SolicitudRegistrada
    activate Worker
    
    Worker->>Ext: GET /v1/scores (Latencia alta)
    activate Ext
    Ext-->>Worker: Retorna Score
    deactivate Ext

    Worker->>CB: Validar / Registrar Pre-aprobación
    activate CB
    CB-->>Worker: Confirmación Operación
    deactivate CB

    Worker->>DB: UPDATE solicitudes SET estado = Finalizado
    deactivate Worker

    %% Fase 3: Sincronización (Polling)
    Note over Cliente, DB: Fase 3: Sincronización del Frontend
    loop Cada 3 segundos
        Cliente->>APIM: GET /v1/solicitudes/{id}/status
        APIM->>API: Consultar estado
        API->>DB: SELECT estado FROM solicitudes
        DB-->>API: Retorna Estado
        API-->>Cliente: HTTP 200 OK (Estado Final)
    end

    Cliente->>Cliente: Muestra: "¡Felicidades! Tu crédito fue Pre-Aprobado"
