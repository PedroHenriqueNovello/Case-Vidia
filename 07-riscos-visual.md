# Vídia — Riscos em formato visual (guia da equipe)

Resumo visual da [análise de riscos](06-analise-de-riscos.md). Diagramas em Mermaid (renderizam no GitHub).

## 1. Mapa de calor — Probabilidade × Gravidade

Linhas = Probabilidade (P), colunas = Gravidade (G). Cada célula mostra a severidade (P×G) e os riscos naquela posição. Canto inferior direito = maior prioridade.

Legenda: 🟥 Crítico (≥15) · 🟧 Alto (10–14) · 🟨 Médio (5–9) · 🟩 Baixo (≤4)

| P \ G | G=2 | G=3 | G=4 | G=5 |
|-------|-----|-----|-----|-----|
| **P=4** | 🟨 8 | 🟧 12 · R-DAT1 | 🟧 16 | 🟥 20 · **R-IA1** |
| **P=3** | 🟨 6 · R-COM3 R-SEC7 R-UX6 | 🟨 9 · R-FN3 R-FN6 R-DAT5 R-COM2 R-UX4 | 🟧 12 · R-FN2 R-DAT2 R-DAT4 R-SEC3 R-UX3 | 🟥 15 · **R-FN1 R-IA2 R-IA4 R-UX1 R-UX2** |
| **P=2** | 🟩 4 | 🟨 6 · R-FN5 R-IA5 | 🟨 8 · R-DAT3 R-SEC4 R-SEC5 R-SEC6 R-UX5 R-COM4 | 🟧 10 · R-FN4 R-COM1 R-SEC1 R-SEC2 R-IA3 |

> Leitura rápida: tudo na coluna **G=5** merece atenção porque é saúde; a célula **P=4, G=5** (R-IA1) é o risco número 1.

## 2. Distribuição dos riscos por categoria

```mermaid
flowchart LR
    R["36 riscos mapeados"] --> FN["Funcionalidade<br/>6 riscos"]
    R --> DAT["Integridade de dados<br/>5 riscos"]
    R --> COM["Comunicação e disponibilidade<br/>4 riscos"]
    R --> SEC["Segurança e privacidade<br/>7 riscos"]
    R --> UX["Usabilidade e acessibilidade<br/>6 riscos"]
    R --> IA["IA leitura de receita<br/>5 riscos"]

    FN --> FNc["1 crítico<br/>R-FN1"]:::crit
    UX --> UXc["2 críticos<br/>R-UX1 R-UX2"]:::crit
    IA --> IAc["3 críticos<br/>R-IA1 R-IA2 R-IA4"]:::crit

    classDef crit fill:#f8cecb,stroke:#b5504f,color:#5a1a18;
```

## 3. Top 6 riscos críticos → controle principal

```mermaid
flowchart LR
    subgraph CRITICOS["Riscos críticos"]
        direction TB
        A["R-IA1 sev 20<br/>IA lê dose errada"]:::crit
        B["R-FN1 sev 15<br/>cronograma errado"]:::crit
        C["R-IA2 sev 15<br/>IA inventa campo"]:::crit
        D["R-IA4 sev 15<br/>cego não confere IA"]:::crit
        E["R-UX1 sev 15<br/>alerta canal único"]:::crit
        F["R-UX2 sev 15<br/>controle sem rótulo"]:::crit
    end

    A --> A1["Revisão obrigatória campo a campo<br/>+ validação + confiança por campo"]
    B --> B1["Testes de borda fuso/HV<br/>+ resumo e confirmação"]
    C --> C1["Campo ausente fica vazio<br/>nunca por suposição"]
    D --> D1["Leitura em voz + releitura<br/>+ conferência por terceiro"]
    E --> E1["Som + vibração + texto<br/>+ dispenser redundante"]
    F --> F1["Nome, função e estado<br/>em todo controle"]

    classDef crit fill:#f8cecb,stroke:#b5504f,color:#5a1a18;
```

## 4. Comportamento seguro quando não dá para executar com segurança

```mermaid
flowchart TD
    START{"Função pode ser<br/>executada com segurança?"} -->|Sim| GO["Executa normalmente"]:::ok
    START -->|Não| SAFE["Modo seguro"]:::warn

    SAFE --> S1["Não inventa / não adivinha<br/>campo duvidoso fica vazio"]
    SAFE --> S2["Preserva estado seguro<br/>dose fica pendente, alerta continua"]
    SAFE --> S3["Sem gravação parcial<br/>operação atômica"]
    SAFE --> S4["Informa de forma acessível<br/>texto + leitor de tela"]
    SAFE --> S5["Oferece alternativa<br/>ex. cadastro manual"]
    SAFE --> S6["Degrada com redundância<br/>outro canal / offline + sync"]
    SAFE --> S7["Registra auditoria<br/>sem expor dado sensível"]

    classDef ok fill:#d8f0dd,stroke:#2e8b57,color:#113a22;
    classDef warn fill:#fde3c4,stroke:#c8821d,color:#5a3a0a;
```

## 5. Superfícies de ataque e defesas (segurança)

```mermaid
flowchart LR
    ATT(("Atacante"))
    ATT -->|força bruta| L["Login / recuperação"]
    ATT -->|interceptação| N["Tráfego de rede"]
    ATT -->|acesso cruzado| S["Registros de outro usuário"]
    ATT -->|comando forjado| DP["Dispenser"]
    ATT -->|enumeração| EM["E-mails válidos"]
    ATT -->|uso indevido| IMG["Imagem da receita"]

    L -.->|defesa| L1["Rate limit + bloqueio progressivo"]:::def
    N -.->|defesa| N1["TLS + cifragem em repouso"]:::def
    S -.->|defesa| S1["Autorização por usuário no servidor"]:::def
    DP -.->|defesa| DP1["Canal autenticado e cifrado"]:::def
    EM -.->|defesa| EM1["Mensagens genéricas"]:::def
    IMG -.->|defesa| IMG1["Consentimento + finalidade limitada + retenção"]:::def

    classDef def fill:#d8f0dd,stroke:#2e8b57,color:#113a22;
```
