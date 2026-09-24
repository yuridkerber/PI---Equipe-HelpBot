# Arquitetura de LLMs de Baixo Custo

Arquitetura alternativa com modelos locais como primeira opção e APIs econômicas como fallback.

> Os modelos locais não cobram por token, mas exigem infraestrutura própria. Os custos das APIs podem variar conforme o provedor, modelo e data de consulta.

```mermaid
flowchart TB
    Orchestrator["Backend Go<br/>Orquestrador de IA<br/>(Strategy Selector)"]

    subgraph Local["Camada Local — custo por token: zero"]
        KEV["KEV<br/>(Classificador)"]
        Qwen["Qwen 4B/7B<br/>via Ollama"]
        Llama["Llama 3 8B<br/>via Ollama"]
        Phi["Phi-3 Mini<br/>via Ollama"]
    end

    subgraph Cloud["Camada Cloud — baixo custo (fallback)"]
        Gemini["Gemini Flash"]
        DeepSeek["DeepSeek"]
        LlamaAPI["Llama 3.1 8B<br/>(API hospedada)"]
        GPTMini["GPT-4o-mini"]
    end

    Orchestrator -->|1. classificar intenção| KEV
    Orchestrator -->|2a. resposta padrão| Qwen
    Orchestrator -->|2b. alternativa local| Llama
    Orchestrator -->|2c. tarefa leve/rápida| Phi

    Orchestrator -->|3. fallback / alta demanda| Gemini
    Orchestrator -->|3. fallback / custo reduzido| DeepSeek
    Orchestrator -->|3. fallback / escala| LlamaAPI
    Orchestrator -->|3. fallback / qualidade-custo| GPTMini
```

## Estratégia sugerida

1. Usar o **KEV** para classificar a intenção e a complexidade do chamado.
2. Encaminhar solicitações simples para o **Qwen local via Ollama**.
3. Utilizar **Phi** ou **Llama** localmente quando houver necessidade de menor consumo ou modelo alternativo.
4. Acionar uma API de fallback somente quando o modelo local estiver indisponível, sobrecarregado ou apresentar baixa confiança.
5. Registrar modelo utilizado, latência, custo estimado e taxa de encaminhamento para monitoramento.

O arquivo fonte PlantUML correspondente está em [`arquitetura_llms_baratas.puml`](./arquitetura_llms_baratas.puml).
