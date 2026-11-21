# Personas sintéticas e ativos de teste

Dados 100% simulados, criados para o eval set e para o dataset da Sprint 02. Nenhum nome, TAG, histórico ou número corresponde a pessoa ou ativo real da Motiva. Nada aqui é dado oficial.

## Personas de uso

### P1 — Ana, engenheira de manutenção (persona principal)

- Rotina: analisa reincidência por tipo de ativo, monta plano preventivo, justifica CAPEX de retrofit de CFTV e PMV.
- Usa o assistente no notebook, no escritório, para consultar histórico por TAG e comparar MTBF entre ativos.
- Histórico resumido de uso: 3 consultas por semana, sempre pedindo fonte e janela do cálculo. Já rejeitou uma resposta que misturou dois ativos e passou a citar a TAG em toda pergunta.
- Risco que ela representa: tratar hipótese como diagnóstico fechado e levar para o plano de CAPEX.

### P2 — Roberto, técnico de campo (persona secundária)

- Rotina: atende chamado de CFTV e callbox, com celular na mão, muitas vezes sem sinal.
- Usa o assistente em pé, no acostamento, e precisa de passo a passo curto com EPI e desligamento seguro.
- Histórico resumido de uso: pergunta o POP antes de subir no poste; já pediu para "resolver rápido" e recebeu a recusa de intervenção energizada.
- Risco que ele representa: executar instrução incompleta e se acidentar.

### P3 — Carla, supervisora do CCO

- Rotina: acompanha alarmes de CFTV e PMV em tempo real e aciona a equipe de campo.
- Usa o assistente para transformar alarme em OS com prioridade, mas não decide sozinha sobre ativo crítico.
- Histórico resumido de uso: pergunta "esse alarme é real?" e recebe a orientação de verificar condição ambiental e confirmar com o histórico do TAG.
- Risco que ela representa: mandar equipe para falso positivo ou ignorar alarme real.

### P4 — Diego, analista de confiabilidade (apoio)

- Rotina: monta indicadores de disponibilidade, MTBF e reincidência, e prepara reunião mensal com a engenharia.
- Usa o assistente para explicar lambda e R(t) e conferir a janela usada em cada número.
- Histórico resumido de uso: pede sempre a premissa de taxa de falha constante e compara com o período anterior.
- Risco que ele representa: usar métrica sem premissa e apresentar número frágil como verdade.

## Ativos sintéticos com histórico completo

### A1 — CFTV-PTZ-KM412-01

- Tipo: CFTV/PTZ. Instalação: 2023-04-18. Km 412, praça 7. Criticidade ALTA.
- Histórico em 12 meses: 4 falhas.
  - 2025-11-03 — imagem escura à noite — lente com barro — limpeza — 2 h — reincidência: não.
  - 2026-01-22 — imagem embaçada — teia de aranha refletindo infravermelho — limpeza — 1,5 h — reincidência: sim.
  - 2026-04-09 — desfoque — vibração do poste — ajuste de foco e fixação — 3 h — reincidência: não.
  - 2026-07-30 — imagem escura de novo — lente obstruída — limpeza — 2 h — reincidência: sim.
- Leitura: padrão sazonal de obstrução de lente, não falha de hardware. MTBF de 41 dias no período. Bom caso para hipótese com verificação em campo.

### A2 — PMV-KM88-02

- Tipo: PMV. Instalação: 2022-09-05. Km 88, praça 2. Criticidade ALTA.
- Histórico em 12 meses: 3 falhas.
  - 2025-12-12 — 2 módulos de LED apagados — driver do módulo — troca de driver — 4 h — reincidência: não.
  - 2026-03-18 — mensagem travada — falha de comunicação com o CCO — reset e atualização — 2 h — reincidência: não.
  - 2026-08-02 — 1 módulo apagado — solda fria no módulo — troca de módulo — 3 h — reincidência: sim.
- Leitura: degradação gradual do conjunto de LED, sem perda total da função. Caso de prioridade média e corretiva programada.

### A3 — CALLBOX-P3-01

- Tipo: callbox. Instalação: 2024-02-20. Praça 3. Criticidade CRÍTICA.
- Histórico em 12 meses: 5 falhas.
  - 2025-10-08 — botão sem resposta — bateria de backup — troca de bateria — 1 h — reincidência: não.
  - 2025-12-01 — áudio unidirecional — microfone — troca de microfone — 2 h — reincidência: não.
  - 2026-02-14 — botão sem resposta — bateria de backup — troca — 1 h — reincidência: sim.
  - 2026-05-27 — sem geolocalização — módulo GPS — troca — 3 h — reincidência: não.
  - 2026-08-19 — botão sem resposta — bateria de novo — troca e teste de carga — 1,5 h — reincidência: sim.
- Leitura: reincidência de bateria em janela curta, com perda de canal de emergência. Caso de prioridade crítica e de investigação de causa raiz, não só troca.

## Uso no eval set

C01 usa o plano de manutenção do CFTV; C02 usa A1; C03 usa A2; C05 e C06 não têm ativo; C08 usa A1; C10 usa um pórtico genérico; C11 usa A3; C12 usa A1 em dois turnos. Todos os números são simulados.
