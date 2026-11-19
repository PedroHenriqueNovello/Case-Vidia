# Vídia — Diagrama de estados e casos de uso

## 1. Diagrama de estados da dose

Representa o ciclo de vida de **uma dose** desde que é gerada pelo cronograma até seu status final. Reflete UC-03/UC-04 e a regra RN1 ("cada dose tem exatamente um status final").

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

## 2. Diagrama de casos de uso (ator × UC)

Mostra os atores do sistema e os casos de uso documentados, com as relações `include`/`extend`.

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

> Observação: `UC-05` é referenciado na documentação (pós-condição de UC-01 e fluxos de UC-03/UC-04), mas não tem fluxo próprio detalhado; aqui é representado pelas relações de `include`/sincronização.
