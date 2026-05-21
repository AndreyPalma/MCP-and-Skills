Crear un diagrama profesional, ejecutivo y visualmente claro para una presentación a gerentes sobre la arquitectura de agentes para DB2 y RPG, inclúyele iconos. El diagrama de ejemplo mermaid es el siguiente para transformarlo:

flowchart LR
    classDef orchestrator fill:#FFF3D6,stroke:#A56A00,color:#4A3200,stroke-width:1.5px;
    classDef worker fill:#F4F6F8,stroke:#6B7280,color:#111827,stroke-width:1px;

    subgraph RPG["Arquitectura RPG"]
        direction TB
        RPGO["RPG Orchestrator"]
        RPGE["RPG Explorer"]
        RPGA["RPG Applier"]
        RPGV["RPG Verifier"]

        RPGO -->|"explora y analiza"| RPGE
        RPGO -->|"aplica cambios"| RPGA
        RPGO -->|"verifica cumplimiento"| RPGV
    end

    subgraph DB2["Arquitectura DB2"]
        direction TB
        DB2O["DB2 Orchestrator"]
        DB2D["DB2 Designer"]
        DB2V["DB2 Validator"]
        DB2E["DB2 Executor"]

        DB2O -->|"diseña objetos"| DB2D
        DB2O -->|"valida estándares"| DB2V
        DB2O -->|"ejecuta entrega"| DB2E
    end

    class RPGO,DB2O orchestrator;
    class RPGE,RPGA,RPGV,DB2D,DB2V,DB2E worker;

