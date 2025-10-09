# Fluxograma do Workflow Multiagente PAP

## Visão Geral
Este fluxograma representa o workflow multiagente para processamento de licitações (PAP - Processo de Análise de Propostas), mostrando dois fluxos principais: **Standard** e **Classified**.

## Fluxograma Principal

```mermaid
flowchart TD
    START([INÍCIO]) --> PDF[Agente Extrator PDF para TXT]
    
    PDF --> DECISION1{Tipo de Workflow?}
    
    DECISION1 -->|Standard| EXTRATOR[Agente Extrator de Requisitos]
    DECISION1 -->|Classified| EXTRATOR_CLASS[Agente Extrator Requisitos com Classificação]
    
    EXTRATOR --> FORMATADOR[Agente Formatador de Requisitos]
    EXTRATOR_CLASS --> FORMATADOR
    
    FORMATADOR --> DECISION2{Tipo de Workflow?}
    
    DECISION2 -->|Standard| BUSCADOR[Agente Buscador Especificações]
    DECISION2 -->|Classified| BUSCADOR_CLASS[Agente Buscador Especificações Classificadas]
    
    BUSCADOR --> ROUTER[Router de Fornecedores<br/>workflow_type_router]
    BUSCADOR_CLASS --> ROUTER
    
    ROUTER --> COMPARADOR[Agente Comparador Requisitos Edital-Fornecedor]
    
    COMPARADOR --> EXCEL[Agente Exportar Excel]
    EXCEL --> END([FIM])
    
    %% Styling para modo escuro
    classDef agentClass fill:#1e3a8a,stroke:#60a5fa,stroke-width:2px,color:#ffffff
    classDef decisionClass fill:#dc2626,stroke:#fca5a5,stroke-width:2px,color:#ffffff
    classDef flowClass fill:#7c3aed,stroke:#c4b5fd,stroke-width:2px,color:#ffffff
    classDef startEndClass fill:#059669,stroke:#6ee7b7,stroke-width:3px,color:#ffffff
    
    class PDF,EXTRATOR,EXTRATOR_CLASS,FORMATADOR,BUSCADOR,BUSCADOR_CLASS,ROUTER,COMPARADOR,EXCEL agentClass
    class DECISION1,DECISION2 decisionClass
    class START,END startEndClass
```

## Detalhamento dos Fluxos de Fornecedores

### Fluxo Standard
```mermaid
flowchart TD
    STANDARD_START([Início Fluxo Standard]) --> PROCESS_ALL[Processar Todos os Datasheets]
    
    PROCESS_ALL --> DATASHEET_LOOP[Para cada Fornecedor]
    DATASHEET_LOOP --> CHECK_TYPE{Tipo de Requisito?}
    
    CHECK_TYPE -->|Técnico| PROCESS_TECH[Processar Datasheet Técnico]
    CHECK_TYPE -->|Teórico| PROCESS_THEO[Processar Hedex Teórico]
    
    PROCESS_TECH --> SAVE_MODEL[Salvar Modelo do Produto]
    SAVE_MODEL --> NEXT_SUPPLIER1[Próximo Fornecedor]
    
    PROCESS_THEO --> CHECK_ASSOCIATED{Tem Datasheet Associado?}
    CHECK_ASSOCIATED -->|Sim| USE_MODEL[Usar Modelo do Datasheet]
    CHECK_ASSOCIATED -->|Não| PROCESS_NORMAL[Processamento Normal]
    
    USE_MODEL --> MODIFY_TEXT[Modificar Texto com Modelo]
    MODIFY_TEXT --> EXECUTE_LLM1[Executar LLM Teórico]
    EXECUTE_LLM1 --> REPLACE_MODEL[Substituir Modelo]
    REPLACE_MODEL --> NEXT_SUPPLIER2[Próximo Fornecedor]
    
    PROCESS_NORMAL --> EXECUTE_LLM2[Executar LLM Normal]
    EXECUTE_LLM2 --> NEXT_SUPPLIER3[Próximo Fornecedor]
    
    NEXT_SUPPLIER1 --> MORE_SUPPLIERS1{Mais Fornecedores?}
    NEXT_SUPPLIER2 --> MORE_SUPPLIERS1
    NEXT_SUPPLIER3 --> MORE_SUPPLIERS1
    
    MORE_SUPPLIERS1 -->|Sim| DATASHEET_LOOP
    MORE_SUPPLIERS1 -->|Não| STANDARD_END([Fim Fluxo Standard])
    
    classDef processClass fill:#1e40af,stroke:#93c5fd,stroke-width:2px,color:#ffffff
    classDef decisionClass fill:#dc2626,stroke:#fca5a5,stroke-width:2px,color:#ffffff
    classDef startEndClass fill:#059669,stroke:#6ee7b7,stroke-width:3px,color:#ffffff
    
    class PROCESS_ALL,DATASHEET_LOOP,PROCESS_TECH,PROCESS_THEO,SAVE_MODEL,USE_MODEL,MODIFY_TEXT,EXECUTE_LLM1,EXECUTE_LLM2,REPLACE_MODEL,PROCESS_NORMAL processClass
    class CHECK_TYPE,CHECK_ASSOCIATED,MORE_SUPPLIERS1 decisionClass
    class STANDARD_START,STANDARD_END startEndClass
```

### Fluxo Classified
```mermaid
flowchart TD
    CLASSIFIED_START([Início Fluxo Classificado]) --> IDENTIFY_PAIRS[Identificar Pares Datasheet-Hedex]
    
    IDENTIFY_PAIRS --> PROCESS_DATASHEETS[Processar Todos os Datasheets]
    PROCESS_DATASHEETS --> DATASHEET_RESULTS[Armazenar Resultados dos Datasheets]
    
    DATASHEET_RESULTS --> PAIR_LOOP[Para cada Par Datasheet-Hedex]
    PAIR_LOOP --> CHECK_DATASHEET{Datasheet Atende?}
    
    CHECK_DATASHEET -->|Atende/Atende Parcialmente/Superioridade| COPY_RESULT[Copiar Resultado do Datasheet para Hedex]
    CHECK_DATASHEET -->|Não Atende| PROCESS_HEDEX[Processar Hedex]
    
    COPY_RESULT --> NEXT_PAIR1[Próximo Par]
    
    PROCESS_HEDEX --> GET_MODEL[Obter Modelo do Datasheet]
    GET_MODEL --> MODIFY_HEDEX[Modificar Texto do Hedex]
    MODIFY_HEDEX --> EXECUTE_HEDEX[Executar LLM no Hedex]
    EXECUTE_HEDEX --> ENSURE_MODEL[Garantir Consistência do Modelo]
    ENSURE_MODEL --> NEXT_PAIR2[Próximo Par]
    
    NEXT_PAIR1 --> MORE_PAIRS{Mais Pares?}
    NEXT_PAIR2 --> MORE_PAIRS
    
    MORE_PAIRS -->|Sim| PAIR_LOOP
    MORE_PAIRS -->|Não| CLASSIFIED_END([Fim Fluxo Classificado])
    
    classDef processClass fill:#7c3aed,stroke:#c4b5fd,stroke-width:2px,color:#ffffff
    classDef decisionClass fill:#dc2626,stroke:#fca5a5,stroke-width:2px,color:#ffffff
    classDef startEndClass fill:#059669,stroke:#6ee7b7,stroke-width:3px,color:#ffffff
    
    class IDENTIFY_PAIRS,PROCESS_DATASHEETS,DATASHEET_RESULTS,PAIR_LOOP,COPY_RESULT,PROCESS_HEDEX,GET_MODEL,MODIFY_HEDEX,EXECUTE_HEDEX,ENSURE_MODEL processClass
    class CHECK_DATASHEET,MORE_PAIRS decisionClass
    class CLASSIFIED_START,CLASSIFIED_END startEndClass
```

## Estados do Workflow

```mermaid
classDiagram
    class State {
        +string edital_pdf_path
        +list fornecedores_collections
        +string extracted_text_edital
        +string requisitos_llm_response
        +dict requisitos_dict
        +string output_filename
        +string workflow_type
        +dict especs_info
    }
    
    State : Representa o estado compartilhado entre agentes
    State : Contém dados do edital, fornecedores e resultados
```

## Agentes Disponíveis

```mermaid
mindmap
  root((Agentes PAP))
    Extração
      AgentEditalPdfToStrs
      AgenteExtratorRequisitos
      AgenteExtratorRequisitosComClassificacao
    Formatação
      AgentFormatador
    Busca
      AgenteBuscadorEspecificacoes
      AgenteBuscadorEspecificacoesClassificadas
      BuscadorEspecificacoesFornecedorLLM
    Comparação
      AgenteComparadorRequisitosEditalFornecedor
    Exportação
      AgentExcelFromDictMultiSupplier
    Base
      AgentBase
      AgentFactory
```

## Configurações de Modelo

O workflow suporta diferentes configurações de modelo:

- **Extração**: Modelos para extração de requisitos
- **Comparação**: Modelos para comparação detalhada
- **Busca**: Modelos para busca semântica
- **Formatação**: Modelos para formatação de dados

### Provedores Suportados:
- **Google (Gemini)**: Modelos Gemini
- **Anthropic**: Claude 3.5 Sonnet
- **OpenAI**: GPT-4o
- **Groq**: Modelos otimizados para velocidade

## Funcionamento do Router de Fornecedores

O **Router de Fornecedores** (`workflow_type_router`) é um componente central que executa internamente os fluxos Standard ou Classified baseado no `workflow_type` definido no estado. Ele não é uma decisão condicional no grafo, mas sim um agente que contém a lógica de ambos os fluxos.

```mermaid
graph TD
    ROUTER[workflow_type_router] --> CHECK{workflow_type no estado}
    CHECK -->|"standard"| STANDARD[Executa suppliers_work_type.standard]
    CHECK -->|"classified"| CLASSIFIED[Executa suppliers_work_type.classified]
    
    STANDARD --> RESULT[Retorna requisitos_dict processados]
    CLASSIFIED --> RESULT
```

## Fluxo de Decisão Principal

```mermaid
graph LR
    A[Entrada] --> B{workflow_type?}
    B -->|standard| C[Fluxo Padrão]
    B -->|classified| D[Fluxo Classificado]
    
    C --> E[Extrator Padrão]
    D --> F[Extrator com Classificação]
    
    E --> G[Buscador Padrão]
    F --> H[Buscador Classificado]
    
    G --> I[Router Executa Fluxo Interno]
    H --> I
    
    I --> J[Comparador]
    J --> K[Excel]
    K --> L[Saída]
```

## Observações Importantes

1. **Dois Fluxos Principais**: O sistema suporta fluxo `standard` (original) e `classified` (otimizado)
2. **Processamento Inteligente**: No fluxo classificado, se o datasheet atende aos requisitos, os hedex associados não são processados
3. **Consistência de Modelos**: O sistema garante que o modelo do produto seja consistente entre datasheet e hedex
4. **Configuração Flexível**: Suporte a múltiplos provedores de LLM com configuração dinâmica
5. **Logging Detalhado**: Cada etapa é logada para rastreabilidade e debugging
