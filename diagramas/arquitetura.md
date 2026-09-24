# Arquitetura de Software — HelpBot

Arquitetura proposta com backend em Go, frontend de chat em TypeScript, painel administrativo em React e LLMs locais executadas pelo Ollama.

```mermaid
flowchart TB
    subgraph Chat["Frontend — Chat (TypeScript + Components)"]
        ChatUI["Chat Widget UI"]
        ChatComponents["Componentes<br/>(Mensagens, Input, Status)"]
        ChatClient["Cliente HTTP/WebSocket"]
        ChatUI --> ChatComponents --> ChatClient
    end

    subgraph Admin["Frontend — ADM (React)"]
        AdmUI["Painel Administrativo"]
        AdmDashboard["Dashboard de Métricas"]
        AdmChamados["Gestão de Chamados"]
        AdmUsuarios["Gestão de Usuários"]
        AdmUI --> AdmDashboard
        AdmUI --> AdmChamados
        AdmUI --> AdmUsuarios
    end

    subgraph Backend["Backend (Go)"]
        API["API Gateway / REST"]
        Auth["Serviço de Autenticação"]
        Tickets["Serviço de Chamados<br/>(Tickets)"]
        Routing["Serviço de Encaminhamento<br/>(Routing)"]
        Orchestrator["Orquestrador de IA"]
        Metrics["Serviço de Métricas"]
    end

    subgraph IA["Camada de IA — LLMs Locais"]
        KEV["KEV<br/>(Classificador de Intenção)"]
        Qwen["Qwen via Ollama<br/>(Geração de Respostas)"]
        Ollama["Ollama Runtime"]
    end

    DB[("PostgreSQL")]
    Queue{{"Fila de Mensagens<br/>(opcional: RabbitMQ/Kafka)"}}

    ChatClient -->|HTTP/WebSocket| API
    AdmDashboard -->|HTTP| API
    AdmChamados -->|HTTP| API
    AdmUsuarios -->|HTTP| API

    API --> Auth
    API --> Tickets
    API --> Orchestrator
    API --> Metrics

    Tickets --> Routing
    Tickets --> DB
    Auth --> DB
    Metrics --> DB

    Orchestrator -->|classificar texto| KEV
    Orchestrator -->|gerar resposta| Qwen
    KEV --> Ollama
    Qwen --> Ollama
    Orchestrator -.->|processamento assíncrono| Queue
    Routing -->|atualiza encaminhamento| Tickets
```

O arquivo fonte PlantUML correspondente está em [`arquitetura.puml`](./arquitetura.puml).
