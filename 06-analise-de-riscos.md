# Vídia — Análise de riscos

> Guia prático para a equipe de desenvolvimento. Cobre identificação (causa e consequência), priorização (probabilidade × gravidade), medidas de prevenção/controle e o comportamento seguro esperado quando uma função não pode ser executada com segurança.

## Como ler este documento

- **ID**: `R-<categoria><n>`. Categorias: **FN** funcionalidade · **DAT** integridade de dados · **COM** comunicação/disponibilidade · **SEC** segurança/privacidade · **UX** usabilidade/acessibilidade · **IA** processamento automatizado.
- **Probabilidade (P)** e **Gravidade (G)**: escala 1–5.
- **Severidade** = P × G. Prioridade: **Crítico** (≥ 15) · **Alto** (10–14) · **Médio** (5–9) · **Baixo** (≤ 4).
- Contexto que amplifica a gravidade: o app lida com **saúde** (dose errada ou perdida tem consequência clínica) e com **pessoas com deficiência** (uma barreira de acessibilidade pode impedir totalmente o uso).

---

## 1. Registro de riscos

### Funcionalidade (regras de negócio, cálculos, validações, processamento)

| ID | Risco | Por que acontece (causa) | Consequência | P | G | Sev. | Prioridade |
|----|-------|--------------------------|--------------|---|---|------|-----------|
| R-FN1 | Cronograma de doses gerado errado (horários/quantidade) | Erro no cálculo a partir de dose, intervalo, 1ª dose e período; fuso horário/horário de verão (UC-03 A5); borda de fim de período | Usuário é alertado na hora errada ou deixa de ser alertado → dose perdida ou dupla | 3 | 5 | 15 | **Crítico** |
| R-FN2 | Dose com dois status finais ou nenhum (viola RN1) | Corrida entre app e dispenser; falha ao gravar (UC-04 E3) deixa a dose em limbo | Histórico e estatísticas incorretos; usuário confia em dado falso | 3 | 4 | 12 | **Alto** |
| R-FN3 | Classificação "no horário" vs. "com atraso" incorreta | Janela de tolerância mal aplicada; relógio do aparelho/dispenser dessincronizado | Métricas de adesão enganosas; decisão clínica baseada em dado errado | 3 | 3 | 9 | **Médio** |
| R-FN4 | Validação aceita dose/intervalo inválidos | RN2 (valores > 0, limites máximos) não aplicada em todas as entradas (manual e IA) | Cronograma absurdo (ex.: intervalo de 0h) ou overdose agendada | 2 | 5 | 10 | **Alto** |
| R-FN5 | Edição retroage sobre doses já registradas | A1 não respeitado: edição deveria valer só para doses futuras | Perda do valor histórico real da época; auditoria corrompida | 2 | 3 | 6 | **Médio** |
| R-FN6 | Cálculo de adesão enganoso | Fórmula `tomadas/(tomadas+não tomadas)` ignora "perdidas" ou mistura categorias | Usuário/profissional superestima a adesão | 3 | 3 | 9 | **Médio** |

### Integridade de dados (perda, duplicação, inconsistência)

| ID | Risco | Causa | Consequência | P | G | Sev. | Prioridade |
|----|-------|-------|--------------|---|---|------|-----------|
| R-DAT1 | Dose duplicada por registros conflitantes (app + dispenser) | Duplo toque; E1 do UC-04 resolvido errado; sincronização reenvia evento | Histórico falso; estatística inflada; confusão do usuário | 4 | 3 | 12 | **Alto** |
| R-DAT2 | Perda de registro de dose | Falha ao gravar (UC-04 E3); evento offline do dispenser não sincroniza; app encerrado no meio | "Buraco" no histórico; dose tomada aparece como perdida | 3 | 4 | 12 | **Alto** |
| R-DAT3 | Exclusão física de medicamento (viola RN4) | Bug que apaga em vez de inativar (A2 exclusão lógica) | Perda irreversível de histórico; não atende LGPD art. 15/16/18 | 2 | 4 | 8 | **Médio** |
| R-DAT4 | Inconsistência de cronograma app × dispenser | Edição/exclusão não chega ao dispenser (UC-01 E3) | Alarme dispara para medicamento já alterado/excluído | 3 | 4 | 12 | **Alto** |
| R-DAT5 | Dessincronização de relógio entre dispositivos | Dispenser sem sincronização de hora; queda de energia (UC-03 E1) | Registros com data/hora erradas; status mal classificado | 3 | 3 | 9 | **Médio** |

### Comunicação e disponibilidade (quedas de conexão, sincronização)

| ID | Risco | Causa | Consequência | P | G | Sev. | Prioridade |
|----|-------|-------|--------------|---|---|------|-----------|
| R-COM1 | Alerta não dispara por depender de rede | Alerta vinculado ao servidor em vez de local (viola UC-03 RN1) | Dose perdida sem qualquer aviso — falha mais grave do produto | 2 | 5 | 10 | **Alto** |
| R-COM2 | Dispenser offline não sincroniza eventos | Perda de conexão prolongada (UC-03 A3); fila local estoura | Histórico atrasado/perdido; conflitos ao reconectar | 3 | 3 | 9 | **Médio** |
| R-COM3 | Falha/timeout do serviço de IA | Indisponibilidade externa (UC-02 E1/E4) | Cadastro por receita indisponível; frustração; usuário desiste | 3 | 2 | 6 | **Médio** |
| R-COM4 | Dispenser com bateria baixa ou relógio reiniciado | Queda de energia (UC-03 E1/A4) | Alarme em horário errado ou silêncio sem aviso | 2 | 4 | 8 | **Médio** |

### Segurança e privacidade (acesso não autorizado, proteção de dados)

| ID | Risco | Causa | Consequência | P | G | Sev. | Prioridade |
|----|-------|-------|--------------|---|---|------|-----------|
| R-SEC1 | Vazamento de dados de saúde | Transmissão sem TLS; banco sem cifragem; log expõe dado sensível | Exposição de dado pessoal sensível; violação de LGPD (arts. 11/46); dano ao titular | 2 | 5 | 10 | **Alto** |
| R-SEC2 | Acesso a registros de outro usuário | Falta de verificação de autorização no servidor (CA12) | Quebra de confidencialidade; acesso indevido a prontuário de medicação | 2 | 5 | 10 | **Alto** |
| R-SEC3 | Conta invadida por força bruta | Ausência de rate limit / bloqueio progressivo (CA13) | Sequestro de conta; adulteração de medicação | 3 | 4 | 12 | **Alto** |
| R-SEC4 | Token de sessão roubado / não invalidado | Token fora do armazenamento seguro do SO; logout não invalida (CA10/CA14) | Sessão sequestrada mesmo após logout | 2 | 4 | 8 | **Médio** |
| R-SEC5 | Comando forjado ao dispenser | Canal app↔dispenser sem autenticação/cifragem (Épico 6 CA03) | Alarme manipulado; dose induzida ou suprimida por terceiro | 2 | 4 | 8 | **Médio** |
| R-SEC6 | Imagem da receita usada além do consentido | Uso para treinar modelo sem consentimento (UC-02 RN5); retenção indevida | Violação de finalidade (LGPD arts. 9/11); perda de confiança | 2 | 4 | 8 | **Médio** |
| R-SEC7 | Enumeração de contas | Mensagem de erro ou recuperação revela se e-mail existe (CA09/CA16) | Coleta de e-mails válidos para ataque dirigido | 3 | 2 | 6 | **Médio** |

### Usabilidade e acessibilidade (barreiras para pessoas com deficiência)

| ID | Risco | Causa | Consequência | P | G | Sev. | Prioridade |
|----|-------|-------|--------------|---|---|------|-----------|
| R-UX1 | Alerta só por som (canal único) | Implementação não combina som+vibração+texto (CA18); usuário surdo/cego não percebe | Dose perdida por não perceber o alerta — barreira crítica | 3 | 5 | 15 | **Crítico** |
| R-UX2 | Controle sem nome/função/estado para leitor de tela | Componente custom (seletor de horário, interruptor) sem rótulo acessível (CA12) | Pessoa cega não consegue cadastrar remédio nem confirmar dose | 3 | 5 | 15 | **Crítico** |
| R-UX3 | Status comunicado só por cor | "Tomado/atrasado/perdido" sem ícone/texto (CA03) | Pessoa com daltonismo/baixa visão não distingue o status | 3 | 4 | 12 | **Alto** |
| R-UX4 | Ação só por gesto, sem alternativa de toque | Deslizar/arrastar sem botão equivalente (CA22) | Pessoa com tremor/pouca precisão não executa a ação | 3 | 3 | 9 | **Médio** |
| R-UX5 | Confirmação de dose fecha sozinha / timeout oculto | Viola CA28 (sem limite de tempo oculto) | Pessoa com dificuldade de leitura/atenção perde a ação; registro incorreto | 2 | 4 | 8 | **Médio** |
| R-UX6 | Registro de dose por toque acidental | Falta cancelamento ao levantar o dedo e "Desfazer" (CA23) | Histórico falso por toque não intencional | 3 | 2 | 6 | **Médio** |

### Processamento automatizado (IA na leitura de receitas)

| ID | Risco | Causa | Consequência | P | G | Sev. | Prioridade |
|----|-------|-------|--------------|---|---|------|-----------|
| R-IA1 | IA lê dose/unidade errada (ex.: 5 mg → 50 mg) | OCR/LLM impreciso; receita manuscrita; foto ruim | **Overdose/subdose** se salvo sem conferência — risco clínico grave | 4 | 5 | 20 | **Crítico** |
| R-IA2 | IA preenche campo ausente por suposição ("conforme necessidade") | Modelo "alucina" em vez de deixar vazio (UC-02 A2) | Cronograma inventado; usuário confia em dado falso | 3 | 5 | 15 | **Crítico** |
| R-IA3 | Medicamento criado sem confirmação explícita | Falha em RN1 do UC-02 (nunca criar sem confirmação) | Tratamento agendado que o usuário não revisou | 2 | 5 | 10 | **Alto** |
| R-IA4 | Pessoa cega confirma leitura sem poder conferir o papel | Dependência visual para validar a extração | Erro da IA não é detectado por quem mais depende do recurso | 3 | 5 | 15 | **Crítico** |
| R-IA5 | Resposta da IA fora do formato esperado | Mudança no modelo/serviço; parsing frágil (UC-02 E2) | Dados corrompidos ou travamento do fluxo de cadastro | 2 | 3 | 6 | **Médio** |

---

## 2. Priorização (ordem de atenção)

### Críticos (Severidade ≥ 15) — tratar primeiro

1. **R-IA1 (20)** — IA lê dose errada → risco clínico direto.
2. **R-FN1 (15)** — cronograma calculado errado.
3. **R-IA2 (15)** — IA inventa campo ausente.
4. **R-IA4 (15)** — usuário cego não consegue conferir a leitura da IA.
5. **R-UX1 (15)** — alerta em canal único.
6. **R-UX2 (15)** — controle sem rótulo acessível.

### Altos (10–14)

R-FN2, R-DAT1, R-DAT2, R-DAT4, R-SEC3, R-UX3 (12) · R-FN4, R-COM1, R-SEC1, R-SEC2, R-IA3 (10).

### Médios (5–9)

R-FN3, R-FN6, R-DAT5, R-COM2, R-UX4 (9) · R-DAT3, R-SEC4, R-SEC5, R-SEC6, R-UX5, R-COM4 (8) · R-FN5, R-COM3, R-SEC7, R-UX6, R-IA5 (6).

> Observação: três dos seis críticos são de **IA** e dois são de **acessibilidade** — exatamente os dois diferenciais do produto. Concentre QA e testes com usuários aí.

---

## 3. Medidas de prevenção e controle

### Para os riscos críticos

**R-IA1 / R-IA2 / R-IA3 / R-IA4 — Leitura de receita por IA**
- Revisão obrigatória: nada é salvo sem confirmação ativa, campo a campo (UC-02 RN1/RN3). Campo ausente fica **vazio**, nunca preenchido por suposição (A2).
- Exibir **nível de confiança por campo** e destacar baixa confiança para conferência obrigatória.
- Validar a extração com as mesmas regras do cadastro manual (RN2: dose/intervalo > 0, limites plausíveis) antes de permitir salvar.
- **Acessibilidade da conferência (R-IA4):** ler em voz alta cada campo extraído com unidade por extenso ("cinquenta miligramas"), oferecer **releitura** e segunda captura; permitir conferência por terceiro de confiança. Deixar explícito que a conferência com a orientação médica é responsabilidade do usuário e que o Vídia não substitui acompanhamento profissional.
- Testar acurácia do modelo antes de cada versão e registrar auditoria (imagem + resposta bruta + valores confirmados).
- **Comportamento seguro em falha:** IA indisponível, timeout, sem internet ou resposta fora do formato → **não adivinhar**; informar e oferecer **cadastro manual** (UC-02 E1–E4).

**R-FN1 — Cronograma**
- Testes de unidade cobrindo bordas: fim de período, intervalos que cruzam meia-noite, **fuso horário e horário de verão** (usar horário local do usuário — UC-03 A5), uso contínuo vs. nº de dias.
- Mostrar **resumo em linguagem natural** e exigir confirmação antes de salvar (UC-01 passo 4).
- **Em falha:** se o cálculo não for confiável, não agendar silenciosamente; bloquear o salvamento e sinalizar o campo.

**R-UX1 / R-UX2 / R-UX3 — Acessibilidade**
- Todo alerta combina **som + vibração + notificação em texto**; som nunca é o único canal (CA18). Dispenser é canal redundante.
- Todo controle (inclusive custom) expõe **nome, função e estado** (CA12); status usa **ícone + texto além da cor** (CA03).
- Checklist por versão: TalkBack/VoiceOver, Switch, Voz, teclado, texto 200%, alto contraste, modo escuro e **o alarme sem som / sem ver / sem tocar** (CA30); Scanner de acessibilidade + relatório de pré-lançamento (CA31); **testes com o público-alvo** (CA32).
- **Em falha:** se um canal de alerta não estiver disponível (ex.: notificação sem permissão), avisar o usuário e **acionar o canal redundante** (dispenser) — nunca ficar em silêncio (UC-03 A2).

### Para os riscos altos

**Integridade (R-DAT1, R-DAT2, R-DAT4)**
- **Idempotência**: cada evento de dose com ID único (origem + dose + timestamp) para deduplicar na sincronização (regra UC-04 E1: vale o 1º; conflito → usuário decide).
- **Fila local durável** no app e no dispenser; eventos offline guardados com data/hora e reenviados ao reconectar (UC-03 A3).
- Marcar edição/exclusão não propagada como **"pendente de sincronização"** e avisar que o dispenser segue o cronograma anterior (UC-01 E3).
- "Desfazer" em poucos minutos e registro retroativo marcado como **"manual"** (UC-04 A5/A6).

**Funcionalidade (R-FN2, R-FN4)**
- Transação atômica ao registrar dose: **tudo ou nada** (UC-04 E3 — nada salvo parcialmente). Em falha, a dose **continua pendente** e os alertas seguem ativos ("na dúvida, continua avisando").
- Validação server-side além da client-side; limites máximos configuráveis.

**Comunicação (R-COM1)**
- Alertas **100% locais** (notificação agendada no aparelho + cronograma no dispenser), independentes de internet (UC-03 RN1). Teste explícito em modo avião.

**Segurança (R-SEC1, R-SEC2, R-SEC3)**
- TLS em todo tráfego; cifragem em repouso dos dados de saúde; **nunca logar dado sensível**.
- Autorização por usuário em **toda** operação protegida, validada no servidor (CA12) — não confiar no cliente.
- **Rate limit + bloqueio progressivo** e proteção contra automação em login/recuperação (CA13).

### Para os riscos médios (resumo)
- Sincronização de relógio (NTP) no dispenser e detecção de relógio inválido → estado de erro visível, nunca horário errado em silêncio (UC-03 E1).
- Token em armazenamento seguro do SO; logout invalida o token no servidor (CA10/CA14).
- Canal app↔dispenser autenticado e cifrado (Épico 6 CA03).
- Consentimento específico e destacado para a IA; **proibir uso da imagem para treino sem consentimento** e definir retenção (UC-02 RN5; LGPD arts. 9/11).
- Mensagens de erro e recuperação **genéricas** (CA09/CA16) para evitar enumeração.
- Alternativa de toque único a todo gesto (CA22); cancelar registro ao levantar o dedo + "Desfazer" (CA23); sem timeout oculto na confirmação (CA28).

---

## 4. Princípio geral de comportamento seguro ("fail-safe")

Quando uma função **não pode ser executada com segurança**, o Vídia deve, em ordem:

1. **Não inventar e não adivinhar** (vale sobretudo para a IA: campo duvidoso fica vazio).
2. **Preservar o estado seguro**: dose em dúvida continua **pendente** e os alertas seguem ativos — é melhor avisar de novo do que silenciar.
3. **Nada de gravação parcial**: operação é atômica; em falha, nada é salvo pela metade.
4. **Informar o usuário de forma acessível** (texto + leitor de tela, mensagem de status anunciada sem mover o foco — CA16) e oferecer **alternativa** (ex.: cadastro manual quando a IA falha).
5. **Degradar com redundância**: se um canal cai, usar o outro (dispenser ↔ app); se a rede cai, operar localmente e sincronizar depois.
6. **Preservar auditoria**: registrar a falha para diagnóstico (sem expor dado sensível).

---

## 5. Controles transversais recomendados

| Mecanismo | Onde atua | Riscos mitigados |
|-----------|-----------|------------------|
| Validação dupla (client + server) | Cadastro manual e IA | R-FN1, R-FN4, R-IA1, R-IA2 |
| Confirmação ativa + resumo em linguagem natural | Cadastro, leitura de IA | R-IA1–R-IA4, R-FN1 |
| Idempotência + fila durável + dedup | Registro e sincronização de doses | R-DAT1, R-DAT2, R-DAT4, R-COM2 |
| Transação atômica + "na dúvida, avisa" | Registro de resposta à dose | R-FN2, R-DAT2 |
| Alerta multicanal local | Notificações e dispenser | R-UX1, R-COM1 |
| Checklist de acessibilidade + teste com público-alvo | Toda a UI e o alarme | R-UX1–R-UX6, R-IA4 |
| TLS + cifragem em repouso + sem dado em log | Comunicação e banco | R-SEC1, R-SEC5, R-SEC6 |
| Autorização por usuário no servidor | Toda operação protegida | R-SEC2 |
| Rate limit + mensagens genéricas | Autenticação e recuperação | R-SEC3, R-SEC7 |
| Logs de auditoria (sem dado sensível) + backup | Dados e eventos | R-DAT2, R-DAT3, R-IA1 |
| Suite de testes (unidade/integração/e2e/acessibilidade) | Antes de cada versão | transversal |

> **Respostas às perguntas-guia**
> - *O que pode comprometer o projeto e como controlar?* Os pontos críticos são leitura por IA (dose errada) e acessibilidade (alerta/controle inacessível); controle = revisão obrigatória, multicanal e testes com o público-alvo.
> - *Como o sistema pode ser atacado/explorado/usado indevidamente?* Força bruta e sequestro de conta (R-SEC3/R-SEC4), acesso cruzado a registros (R-SEC2), interceptação de dados de saúde (R-SEC1), comando forjado ao dispenser (R-SEC5), enumeração de contas (R-SEC7) e uso indevido da imagem da receita (R-SEC6); controle = TLS/cifragem, autorização server-side, rate limit, canal cifrado com o dispenser e consentimento com finalidade limitada.
