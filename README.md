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
    STANDARD_START([Início Fluxo Standard]) --> PHASE1[FASE 1: Processar Datasheets]
    
    %% FASE 1: Processamento de Datasheets
    PHASE1 --> DATASHEET_LOOP[Para cada Fornecedor]
    DATASHEET_LOOP --> CHECK_TECH{É Técnico?}
    
    CHECK_TECH -->|Sim| PROCESS_TECH[Processar Datasheet Técnico]
    CHECK_TECH -->|Não| SKIP_TECH[Pular para Próximo]
    
    PROCESS_TECH --> SAVE_MODEL[Salvar Modelo do Produto<br/>modelos_produtos[fornecedor]]
    SAVE_MODEL --> NEXT_TECH[Próximo Fornecedor]
    SKIP_TECH --> NEXT_TECH
    
    NEXT_TECH --> MORE_TECH{Mais Fornecedores?}
    MORE_TECH -->|Sim| DATASHEET_LOOP
    MORE_TECH -->|Não| PHASE2[FASE 2: Processar Hedex]
    
    %% FASE 2: Processamento de Hedex
    PHASE2 --> HEDEX_LOOP[Para cada Fornecedor]
    HEDEX_LOOP --> CHECK_HEDEX{É Teórico/Hedex?}
    
    CHECK_HEDEX -->|Sim| CHECK_ASSOCIATED{Tem Datasheet Associado?}
    CHECK_HEDEX -->|Não| SKIP_HEDEX[Pular para Próximo]
    
    CHECK_ASSOCIATED -->|Sim| USE_MODEL[Usar Modelo Salvo<br/>modelo_datasheet = modelos_produtos[datasheet_associado]]
    CHECK_ASSOCIATED -->|Não| PROCESS_NORMAL[Processamento Normal]
    
    USE_MODEL --> MODIFY_TEXT[Modificar Texto com Modelo<br/>Adicionar modelo ao texto e busca]
    MODIFY_TEXT --> EXECUTE_LLM1[Executar LLM Teórico]
    EXECUTE_LLM1 --> REPLACE_MODEL[Substituir Modelo no Resultado<br/>resultado["produto_fornecedor"] = modelo_datasheet]
    REPLACE_MODEL --> NEXT_HEDEX[Próximo Fornecedor]
    
    PROCESS_NORMAL --> EXECUTE_LLM2[Executar LLM Normal]
    EXECUTE_LLM2 --> NEXT_HEDEX
    SKIP_HEDEX --> NEXT_HEDEX
    
    NEXT_HEDEX --> MORE_HEDEX{Mais Fornecedores?}
    MORE_HEDEX -->|Sim| HEDEX_LOOP
    MORE_HEDEX -->|Não| STANDARD_END([Fim Fluxo Standard])
    
    classDef processClass fill:#1e40af,stroke:#93c5fd,stroke-width:2px,color:#ffffff
    classDef decisionClass fill:#dc2626,stroke:#fca5a5,stroke-width:2px,color:#ffffff
    classDef startEndClass fill:#059669,stroke:#6ee7b7,stroke-width:3px,color:#ffffff
    classDef phaseClass fill:#7c2d12,stroke:#fed7aa,stroke-width:3px,color:#ffffff
    
    class DATASHEET_LOOP,PROCESS_TECH,SAVE_MODEL,HEDEX_LOOP,USE_MODEL,MODIFY_TEXT,EXECUTE_LLM1,EXECUTE_LLM2,REPLACE_MODEL,PROCESS_NORMAL,SKIP_TECH,SKIP_HEDEX,NEXT_TECH,NEXT_HEDEX processClass
    class CHECK_TECH,CHECK_HEDEX,CHECK_ASSOCIATED,MORE_TECH,MORE_HEDEX decisionClass
    class STANDARD_START,STANDARD_END startEndClass
    class PHASE1,PHASE2 phaseClass
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
