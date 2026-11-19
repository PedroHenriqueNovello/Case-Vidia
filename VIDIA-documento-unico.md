# Vídia — Análise e modelagem do produto

> Documento único consolidado. App de gestão de medicamentos com foco em **acessibilidade** (WCAG 2.2 / Android / Apple HIG), **leitura de receita por IA** e integração com um **dispenser físico**.
>
> Os diagramas usam **Mermaid** — renderizam automaticamente no GitHub e em https://mermaid.live.

## Sumário

- [Parte 1 — Compreender o funcionamento (funcionalidades e componentes)](#parte-1--compreender-o-funcionamento-funcionalidades-e-componentes)
  - [1. Visão geral](#1-visão-geral)
  - [2. Mapa de funcionalidades (por épico)](#2-mapa-de-funcionalidades-por-épico)
  - [3. Componentes do sistema](#3-componentes-do-sistema)
  - [4. Fluxos principais (casos de uso)](#4-fluxos-principais-casos-de-uso)
  - [5. Observações de risco](#5-observações-de-risco)
- [Parte 2 — Representação visual do fluxo de operação](#parte-2--representação-visual-do-fluxo-de-operação)
  - [A. Arquitetura de componentes](#a-arquitetura-de-componentes)
  - [B. Modelo de dados (entidades)](#b-modelo-de-dados-entidades)
  - [C. Fluxograma — fluxo geral do usuário](#c-fluxograma--fluxo-geral-do-usuário)
  - [D. Fluxograma — UC-01 cadastrar/editar/excluir](#d-fluxograma--uc-01-cadastrareditarexcluir)
  - [E. Fluxograma — UC-03/04 alerta e resposta à dose](#e-fluxograma--uc-0304-alerta-e-resposta-à-dose)
  - [F. Sequência — Autenticação](#f-sequência--autenticação)
  - [G. Sequência — UC-02 leitura de receita por IA](#g-sequência--uc-02-leitura-de-receita-por-ia)
  - [H. Sequência — UC-03/04 alerta e registro da dose](#h-sequência--uc-0304-alerta-e-registro-da-dose)
  - [I. Sequência — Relatório / estatísticas](#i-sequência--relatório--estatísticas)
  - [J. Estados — ciclo de vida da dose](#j-estados--ciclo-de-vida-da-dose)
  - [K. Casos de uso (ator × UC)](#k-casos-de-uso-ator--uc)

---

# Parte 1 — Compreender o funcionamento (funcionalidades e componentes)

## 1. Visão geral

O Vídia ajuda o usuário a **não perder doses de medicamentos**. Ele:

- cadastra remédios (manualmente ou lendo a receita com IA),
- gera um cronograma de doses,
- avisa na hora certa por **vários canais** (som, vibração, notificação em texto e um dispenser físico),
- registra o que foi tomado / adiado / não tomado / perdido,
- mostra histórico, gráficos e estatísticas de adesão ao tratamento.

Tudo isso sob duas restrições fortes que atravessam o produto inteiro:

- **Acessibilidade** (Épico 1): o app precisa ser 100% operável por leitor de tela, teclado, switch, voz, com contraste, texto ampliável etc.
- **Privacidade/LGPD**: dados de medicamento e imagem de receita são **dados pessoais sensíveis** (saúde), com coleta mínima, consentimento específico e comunicação protegida.

## 2. Mapa de funcionalidades (por épico)

| Épico | Funcionalidade | Resumo |
|-------|----------------|--------|
| 1. Design acessível | Acessibilidade transversal | Contraste, texto escalável, leitor de tela, alarme multicanal, alvos de toque grandes, operação sem toque, prevenção de erro, verificação (CA01–CA32) |
| 2. Autenticação | Cadastro, login, recuperação de senha | E-mail + senha, hash seguro, HTTPS/TLS, sessão segura, política de privacidade, proteção contra força bruta, recuperação por e-mail (link expira em 1h) |
| 3. Início | Acompanhamento + Agenda | Caixas de visão geral (ativos, doses do dia, atrasos), lista de doses por horário com status, "Marcar como tomado", notificações |
| 4. Medicamentos | Adicionar/editar/excluir + Leitura de receita (IA) | CRUD lógico com histórico, IA que lê receita e devolve texto para revisão, consentimento específico para o uso de IA |
| 5. Relatório | Histórico, gráfico, estatísticas | Histórico geral e por medicamento, status das doses, gráfico por dia da semana, adesão geral |
| 6. Dispositivos | Dispenser físico | Alarme no dispenser, botões físicos (tomei/adiar/não tomei), comunicação segura, sincronização |

## 3. Componentes do sistema

### 3.1 Interface (app móvel — Android e iOS)
- Telas: **Login/Cadastro/Recuperação**, **Início** (dashboard), **Agenda**, **Medicamentos** (CRUD + leitura de receita), **Relatório** (histórico/gráfico/estatísticas), **Tela de alarme**, **Configurações** (canais de alerta, volume, soneca).
- Requisitos de acessibilidade embutidos em cada controle: nome/função/estado expostos (CA12), alvos ≥ 48dp/44pt (CA21), alternativa de toque único a gestos (CA22), foco visível (CA25), sem limite de tempo oculto (CA28).

### 3.2 Processamento (lógica de negócio)
- **Gerador de cronograma**: a partir de dose, intervalo, data/hora da primeira dose e período, gera as doses futuras (UC-01).
- **Motor de alertas**: agenda notificações locais no aparelho (não depende de internet — UC-03 RN1) e dispara o dispenser.
- **Máquina de status da dose**: `pendente → tomada no horário | tomada com atraso | não tomada | perdida` (UC-03/UC-04, RN1 "cada dose tem exatamente um status final").
- **Janela de tolerância / soneca**: define quando a dose vira "perdida" e limita adiamentos.
- **Resolução de conflitos**: dedup entre registro do app e do dispenser (UC-04 E1).
- **Validação**: regras de negócio do cadastro (dose/intervalo > 0, campos obrigatórios — UC-01 RN1/RN2).

### 3.3 Banco de dados
- Entidades principais: **Usuário**, **Medicamento** (com histórico — exclusão é lógica, RN4), **Dose** (agendada e seus status), **Registro de dose** (origem app/dispenser, data/hora, manual/retroativo), **Dispenser** (vínculo, cronograma, estado), **Auditoria de IA** (imagem da receita + resposta bruta + valores confirmados — UC-02 passo 8).
- Dados de saúde tratados como **sensíveis**: armazenamento e transmissão protegidos (Épico 4 CA03).

### 3.4 Autenticação e segurança
- E-mail + senha; **hash** seguro da senha (CA03), nunca texto puro.
- **HTTPS/TLS** em toda comunicação (CA05/CA11).
- **Sessão** segura; tokens persistentes em armazenamento seguro do SO (CA10); logout invalida token (CA14).
- **Mensagem de erro genérica** no login (não revela se o e-mail existe — CA09).
- **Proteção contra tentativas repetidas** (CA13) e recuperação de senha com link que **expira em 1h**, enviado só se o e-mail existir (CA16).
- Controle de acesso por usuário aos próprios registros (CA12).

### 3.5 Notificações / alertas (multicanal)
- Todo lembrete combina **som + vibração + notificação em texto** e o usuário escolhe os canais (CA18). Som nunca é o único aviso.
- Notificação local no horário (CA01) e **renotificação de atraso** (CA02).
- Privacidade na tela bloqueada: não expor dado de saúde sem opt-in (Módulo Início CA06).
- Tela de alarme sempre com botões "Parar" e "Adiar" visíveis; volume ajustável (CA19).

### 3.6 Integrações externas
- **Serviço de IA (LLM)**: lê a imagem da receita e devolve os dados do medicamento em texto acessível, com nível de confiança por campo (UC-02). Requer internet; consentimento específico (Épico 4 CA07); nunca cria/altera medicamento sem confirmação do usuário (RN1).
- **Dispenser físico (IoT)**: alarme sonoro/luminoso a partir do cronograma armazenado localmente, botões físicos (tomei/adiar/não tomei), **comunicação protegida** (Épico 6 CA03), sincronização quando reconecta.

## 4. Fluxos principais (casos de uso)

- **UC-01** Cadastrar/editar/excluir medicamento → valida, mostra resumo em linguagem natural, salva, gera doses futuras, envia cronograma ao dispenser (UC-05).
- **UC-02** Cadastrar por leitura de receita (IA) → aviso de responsabilidade, foto, IA extrai campos, usuário revisa/confirma, segue UC-01, grava auditoria.
- **UC-03** Alertar horário da dose → dispenser + app alertam, repetem até fim da janela, dose vira "perdida" se não houver resposta.
- **UC-04** Registrar resposta à dose → "Tomei"/"Adiar"/"Não tomei" (app ou dispenser), classifica status, para alertas, atualiza agenda/histórico/estatísticas.
- **UC-05** (referenciado) Sincronizar cronograma com o dispenser.

## 5. Observações de risco

- Erro de digitação ou leitura de dose (ex.: "5 mg" lido como "50 mg") — mitigado por revisão obrigatória e auditoria.
- Edição/exclusão que não chega ao dispenser — marcada como "pendente de sincronização".
- Dose duplicada por registros conflitantes (app + dispenser) — resolução por "vale o primeiro registro".
- Vazamento de dado de saúde — mitigado por LGPD, TLS, consentimento.

---

# Parte 2 — Representação visual do fluxo de operação

## A. Arquitetura de componentes

```mermaid
flowchart TB
    subgraph APP["App móvel (Android / iOS)"]
        UI["Interface acessível<br/>Login · Início · Agenda · Medicamentos<br/>Relatório · Tela de alarme · Configurações"]
        A11Y["Camada de acessibilidade<br/>leitor de tela · teclado · switch · voz<br/>contraste · texto escalável"]
        subgraph PROC["Processamento / lógica de negócio"]
            SCHED["Gerador de cronograma<br/>(doses futuras)"]
            ALERT["Motor de alertas<br/>notificações locais"]
            STATUS["Máquina de status da dose<br/>pendente / tomada / atrasada / perdida"]
            VALID["Validação (regras UC-01)"]
            CONF["Resolução de conflitos<br/>app x dispenser"]
        end
        LOCALDB["Armazenamento local<br/>cronograma · fila de sincronização"]
    end

    subgraph BACK["Backend / servidor"]
        AUTH["Autenticação & sessão<br/>hash de senha · tokens · rate limit"]
        API["API (HTTPS/TLS)"]
        DB[("Banco de dados<br/>Usuário · Medicamento · Dose<br/>Registro · Auditoria IA")]
    end

    IA["Serviço de IA / LLM<br/>leitura de receita"]
    DISP["Dispenser físico (IoT)<br/>alarme + botões tomei/adiar/não tomei"]
    NOTIF["Sistema de notificações do SO<br/>som · vibração · texto"]

    UI <--> A11Y
    UI --> PROC
    PROC --> LOCALDB
    ALERT --> NOTIF
    ALERT <--> DISP
    SCHED --> DISP
    CONF <--> DISP

    APP <-->|HTTPS/TLS| API
    API --> AUTH
    API --> DB
    AUTH --> DB
    APP <-->|imagem + consentimento| IA

    classDef ext fill:#eef,stroke:#669;
    class IA,DISP,NOTIF ext;
```

## B. Modelo de dados (entidades)

```mermaid
erDiagram
    USUARIO ||--o{ MEDICAMENTO : cadastra
    USUARIO ||--o| DISPENSER : possui
    MEDICAMENTO ||--o{ DOSE : gera
    DOSE ||--o| REGISTRO_DOSE : "tem (0..1)"
    MEDICAMENTO ||--o{ AUDITORIA_IA : "origina (se leitura IA)"

    USUARIO {
        id id PK
        string email
        string senha_hash
        bool consentimento_ia
    }
    MEDICAMENTO {
        id id PK
        string nome
        string dosagem
        string intervalo
        date   periodo_inicio
        date   periodo_fim
        string observacoes
        bool   ativo "exclusao logica"
    }
    DOSE {
        id id PK
        datetime horario_previsto
        string status "pendente|no_horario|atraso|nao_tomada|perdida"
    }
    REGISTRO_DOSE {
        id id PK
        datetime data_hora
        string origem "app|dispenser"
        bool   manual_retroativo
    }
    DISPENSER {
        id id PK
        string estado
        datetime cronograma_sync
    }
    AUDITORIA_IA {
        id id PK
        blob   imagem_receita
        text   resposta_bruta
        text   valores_confirmados
    }
```

## C. Fluxograma — fluxo geral do usuário

```mermaid
flowchart TD
    START(["Abrir o app"]) --> HASACC{"Tem conta?"}
    HASACC -->|Não| CAD["Cadastro<br/>e-mail + senha + política de privacidade"]
    HASACC -->|Sim| LOGIN["Login"]
    LOGIN -->|Credenciais inválidas| ERR["Erro genérico<br/>não revela se e-mail existe"]
    ERR --> LOGIN
    LOGIN -->|Esqueci a senha| REC["Recuperação por e-mail<br/>link expira em 1h"]
    REC --> LOGIN
    CAD --> HOME
    LOGIN -->|OK| HOME["Tela Início<br/>ativos · doses do dia · atrasos"]

    HOME --> ACT{"O que fazer?"}
    ACT -->|Cadastrar remédio| UC01["UC-01 manual"]
    ACT -->|Ler receita| UC02["UC-02 leitura por IA"]
    ACT -->|Ver agenda| AGENDA["Agenda do dia<br/>marcar como tomado"]
    ACT -->|Ver relatório| REL["Histórico · gráfico · estatísticas"]
    UC01 --> SCHED["Gera doses futuras<br/>+ envia cronograma ao dispenser"]
    UC02 --> SCHED
    SCHED --> HOME
    AGENDA --> HOME
    REL --> HOME
```

## D. Fluxograma — UC-01 cadastrar/editar/excluir

```mermaid
flowchart TD
    A([Adicionar medicamento]) --> B[Informar nome, dosagem,<br/>intervalo, 1ª dose, período, observações]
    B --> C{"Dados válidos?<br/>RN1 obrigatórios · RN2 dose/intervalo maior que 0"}
    C -->|Não E1| C1[Destaca campo · nada é salvo] --> B
    C -->|Sim| D{Mesmo nome já ativo? A3}
    D -->|Sim| D1[Aviso de duplicidade]
    D1 --> D2{Continuar?}
    D2 -->|Não| X([Cancelar])
    D2 -->|Sim| E
    D -->|Não| E[Resumo em linguagem natural<br/>'500 mg a cada 8h, de 20/09 às 08:00 até 27/09']
    E --> F{Usuário confirma?}
    F -->|Não| X
    F -->|Sim| G[Salvar]
    G -->|E2 falha ao salvar| G1[Nada salvo parcialmente · avisa · tentar de novo] --> B
    G -->|OK| H[Gera doses futuras · agenda alertas]
    H --> I{Dispenser alcançável?}
    I -->|Sim| J([Cronograma enviado ao dispenser · UC-05])
    I -->|Não E3| K([Marca 'pendente de sincronização'<br/>dispenser segue cronograma anterior])

    subgraph Edit["A1 editar / A2 excluir"]
      direction TB
      ED[Editar: vale só p/ doses futuras<br/>histórico mantém valores da época]
      EX[Excluir: exclusão LÓGICA<br/>cancela alertas futuros · mantém histórico · RN4]
    end
```

## E. Fluxograma — UC-03/04 alerta e resposta à dose

```mermaid
flowchart TD
    P([Dose pendente<br/>medicamento ativo c/ cronograma]) --> AL[No horário:<br/>dispenser = som+luz · app = notificação local]
    AL --> W{Usuário respondeu?}
    W -->|Não, dentro da janela| REP[Repete a cada N min] --> W
    W -->|Não, fim da janela RN3| LOST(["Status: PERDIDA · registra"])
    W -->|Sim| R{Qual resposta?}
    R -->|Tomei| T{Dentro da tolerância?}
    T -->|Sim| T1([Tomada NO HORÁRIO])
    T -->|Não| T2([Tomada COM ATRASO])
    R -->|Adiar A1| SNZ{Limite de adiamentos<br/>e ainda na janela?}
    SNZ -->|Sim| AL
    SNZ -->|Não| LOST
    R -->|Não tomei A2| NT([NÃO TOMADA · motivo opcional])
    T1 --> STOP[Interrompe alertas nos 2 canais]
    T2 --> STOP
    NT --> STOP
    STOP --> UPD([Atualiza agenda · histórico · estatísticas])

    CONF{{"E1: registro duplicado app x dispenser?<br/>vale o 1º · se conflitante, usuário decide"}}
```

## F. Sequência — Autenticação

```mermaid
sequenceDiagram
    actor U as Usuário
    participant App
    participant API as Backend (HTTPS/TLS)
    participant Auth as Autenticação
    participant DB as Banco de dados

    Note over U,App: Cadastro (coleta mínima: e-mail + senha)
    U->>App: Preenche e-mail + senha
    App->>U: Exibe Política de Privacidade (antes de concluir)
    U->>App: Consente e envia
    App->>API: POST /cadastro (TLS)
    API->>Auth: Valida formato, e-mail único, força da senha
    Auth->>DB: Salva usuário (senha com hash seguro)
    DB-->>Auth: OK
    Auth-->>App: Cadastro concluído
    App-->>U: "Conta criada, faça login"

    Note over U,App: Login
    U->>App: e-mail + senha
    App->>API: POST /login (TLS)
    API->>Auth: Verifica credenciais + rate limit
    alt Credenciais válidas
        Auth->>DB: Cria sessão / token
        Auth-->>App: Token (guardado em armazenamento seguro do SO)
        App-->>U: Acesso liberado (Início)
    else Inválidas
        Auth-->>App: Erro genérico (não revela se e-mail existe)
        App-->>U: "Não foi possível fazer login"
    end
```

## G. Sequência — UC-02 leitura de receita por IA

```mermaid
sequenceDiagram
    actor U as Usuário
    participant App
    participant IA as Serviço de IA LLM
    participant DB as Banco de dados
    participant D as Dispenser

    U->>App: Ler receita
    App->>U: Aviso IA pode errar, conferência é sua responsabilidade
    U->>App: Aceita consentimento específico p/ IA no 1º uso
    U->>App: Fotografa a receita com orientação por voz
    App->>App: Verifica qualidade foco, luz, corte
    alt Imagem ruim A1
        App-->>U: Pede nova foto
    end
    App->>IA: Envia imagem TLS
    alt IA indisponível, timeout ou sem internet E1/E4
        IA-->>App: Falha
        App-->>U: Oferece cadastro manual UC-01
    else Resposta OK
        IA-->>App: nome, dosagem, intervalo, duração + confiança por campo
        App->>App: Valida regras UC-01, marca ausentes / baixa confiança
        App-->>U: Mostra rascunho para revisão
        U->>App: Corrige e confirma cada medicamento RN1
        App->>DB: Salva medicamento, gera doses, grava auditoria
        App->>D: Envia cronograma UC-05
        App-->>U: Medicamento cadastrado
    end
```

## H. Sequência — UC-03/04 alerta e registro da dose

```mermaid
sequenceDiagram
    actor U as Usuário
    participant D as Dispenser
    participant App
    participant DB as Banco de dados

    Note over D,App: No horário, não depende de internet RN1
    D->>U: Alarme sonoro + luminoso, cronograma local
    App->>U: Notificação local som + vibração + texto

    loop A cada N min, até fim da janela de tolerância
        D->>U: Repete alarme
        App->>U: Repete notificação
    end

    alt Usuário responde UC-04
        U->>D: Aperta Tomei / Adiar / Não tomei
        D->>App: Evento data, hora, origem dispenser
        App->>App: Classifica status no horário / atraso
        App->>D: Interrompe alarme nos 2 canais
        App->>DB: Registra dose, atualiza agenda, histórico, estatísticas
        Note over App,DB: E1 duplo registro, vale o 1º;<br/>se conflitante, usuário decide
    else Dispenser sem conexão A4 / UC-05
        D->>D: Guarda evento com data e hora
        App-->>U: Aguardando sincronização
        D->>App: Sincroniza quando reconecta
    else Sem resposta até fim da janela
        App->>DB: Dose PERDIDA
    end
```

## I. Sequência — Relatório / estatísticas

```mermaid
sequenceDiagram
    actor U as Usuário
    participant App
    participant DB as Banco de dados

    U->>App: Abre Relatório
    App->>DB: Consulta doses (status, data, hora) do usuário
    DB-->>App: Histórico (geral e por medicamento)
    App->>App: Classifica em no horário / com atraso / perdida
    App->>App: Calcula adesão = tomadas / (tomadas + não tomadas)
    App-->>U: Histórico + gráfico por dia da semana + estatísticas
    Note over App,U: RN2 (UC-04): registro reflete o que o usuário informou,<br/>não comprova ingestão — estatística deixa isso claro
```

## J. Estados — ciclo de vida da dose

```mermaid
stateDiagram-v2
    [*] --> Agendada: cronograma gera a dose (UC-01)
    Agendada --> Pendente: chega o horário (UC-03)

    Pendente --> Pendente: sem resposta, repete alerta a cada N min

    Pendente --> Adiada: usuário "Adiar" (A1) dentro da janela e do limite
    Adiada --> Pendente: reagenda o alerta

    Pendente --> TomadaNoHorario: "Tomei" dentro da tolerância
    Pendente --> TomadaComAtraso: "Tomei" fora da tolerância (ainda na janela)
    Adiada --> TomadaComAtraso: "Tomei" após adiamento
    Pendente --> NaoTomada: "Não tomei" (A2)

    Pendente --> Perdida: fim da janela sem resposta (RN3)
    Adiada --> Perdida: janela esgotada / adiamentos esgotados

    TomadaNoHorario --> [*]
    TomadaComAtraso --> [*]
    NaoTomada --> [*]
    Perdida --> [*]

    note right of TomadaNoHorario
        Estados finais param os alertas
        nos dois canais e atualizam
        agenda, histórico e estatísticas
    end note
```

## K. Casos de uso (ator × UC)

```mermaid
flowchart LR
    U(("Usuário"))
    IA(("Serviço de IA"))
    D(("Dispenser"))

    subgraph SYS["Sistema Vídia"]
        UC01(["UC-01<br/>Cadastrar / editar / excluir medicamento"])
        UC02(["UC-02<br/>Cadastrar por leitura de receita (IA)"])
        UC03(["UC-03<br/>Alertar horário da dose"])
        UC04(["UC-04<br/>Registrar resposta à dose"])
        UC05(["UC-05<br/>Sincronizar cronograma c/ dispenser"])
        AUTH(["Autenticar<br/>(cadastro / login / recuperação)"])
        REL(["Ver relatório<br/>(histórico / gráfico / estatísticas)"])
    end

    U --- AUTH
    U --- UC01
    U --- UC02
    U --- UC04
    U --- REL
    IA --- UC02
    D --- UC03
    D --- UC04
    D --- UC05

    UC02 -.->|«include»| UC01
    UC01 -.->|«include»| UC05
    UC03 -.->|«include»| UC04
    UC02 -.->|«extend»| AUTH
    UC01 -.->|«extend»| AUTH

    classDef uc fill:#eef,stroke:#669;
    class UC01,UC02,UC03,UC04,UC05,AUTH,REL uc;
```

---

> **Nota sobre UC-05:** é referenciado na documentação (pós-condição de UC-01 e fluxos de UC-03/UC-04), mas não tem fluxo próprio detalhado; aqui aparece pelas relações de `include` e pelos seus efeitos (envio/sincronização de cronograma).
