# Matriz de riscos, ética e LGPD — EngMan Copilot

Versão sprint 1. Recorte A. Toda mitigação abaixo tem endereço: bloco e regra do system prompt, etapa do fluxograma, caso do eval set ou regra da base. Risco sem endereço não entra na matriz.

## Escala usada

- **Impacto:** 1 baixo, 2 moderado, 3 alto, 4 crítico (segurança viária, vazamento legal, perda financeira relevante).
- **Probabilidade:** 1 rara, 2 possível, 3 provável, 4 quase certa sem controle.
- **Severidade:** impacto x probabilidade. 12 ou mais exige controle obrigatório antes de qualquer uso em produção.

## Matriz

| ID | Risco | Como se manifesta | Impacto | Prob. | Sev. | Mitigação | Prompt | Fluxograma | Eval | Base |
|---|---|---|---|---|---|---|---|---|---|---|
| R01 | Alucinação de ativo, métrica ou histórico | Inventa TAG, modelo, número de OS, MTBF, trecho de norma ou registro que não existe | Segurança e financeiro (3) | 4 | 12 | Só responder com base ou tool; citar fonte; admitir ausência; guardrail de saída confere TAG e métrica | Regras 1, 2, 5, 6 e 9 | Blocos 5, 6 e 7 | C02, C06 | Metadados de origem e versão; tool como fonte única |
| R02 | Prompt injection direta e indireta | "Ignore as instruções" no chat ou comando escondido em laudo colado | Segurança e reputacional (3) | 3 | 9 | Instruções imutáveis; conteúdo colado é dado; guardrail de entrada; recusa curta | Regra 8 e BORDA | Blocos 2 e 7 | C08, C09 | Boletim e laudo entram como texto, nunca como instrução |
| R03 | Acidente elétrico em intervenção | Usuário pede sequência de fios em painel energizado e segue a resposta | Segurança e legal (4) | 3 | 12 | Recusar orientação elétrica; encaminhar POP, NR-10 e profissional habilitado; sem instrução de execução | Regra 3 e BORDA | Blocos 2, 7 e 8 | C10 | POP de desligamento seguro; resumo NR-10 |
| R04 | Acidente em via por trabalho sem sinalização | Técnico faz serviço no acostamento sem fechar faixa | Segurança e legal (4) | 3 | 12 | Recusar execução sem sinalização; reforçar NR-35 e regra interna; acionar responsável | Regras 3 e 4 e TOM | Blocos 2 e 7 | C11 | Resumo NR-35 e regra de trabalho em via |
| R05 | LGPD: dado pessoal na base ou no prompt | Placa de OCR, imagem de ocupante, nome ou matrícula de empregado entram no contexto | Legal e reputacional (4) | 3 | 12 | Lista explícita do que não entra; mascarar placa; pedir remoção; não reproduzir o dado | Regra 10 | Blocos 1, 2, 3 e 7 | C09 | Seção "O que nunca entra" do inventário |
| R06 | Vazamento de histórico para API externa | Histórico de falhas e arrecadação enviados a provedor público | Legal e financeiro (3) | 3 | 9 | Modelo local como padrão; API externa só com dado simulado e chave em variável de ambiente | Regra 10 e TOM | Bloco 4 | C08 | Base marcada como simulada; nada real versionado |
| R07 | Viés na priorização | Priorizar sempre a mesma região, praça ou tipo de ativo | Legal e reputacional (3) | 2 | 6 | Critério explícito de criticidade x segurança x arrecadação, auditável, sem "sensação" | Few-shot e FORMATO | Blocos 3 e 7 | C03 | Matriz de criticidade documentada |
| R08 | Excesso de confiança do usuário | Técnico trata hipótese como diagnóstico confirmado e executa sem inspeção | Segurança e operacional (4) | 3 | 12 | Rótulo FATO vs HIPÓTESE; nível de confiança obrigatório; validação humana antes da OS | Regras 2 e 4 e FORMATO | Blocos 7 e 8 | C02, C03, C12 | POP exige inspeção antes de corretiva |
| R09 | Contexto estourado e troca de ativo | Conversa longa perde a TAG ou mistura dois equipamentos | Operacional (2) | 3 | 6 | Janela de 6 turnos; resumo a cada 6; TAG ativo reafirmado; orçamento de 8 mil tokens | BORDA | Blocos 3 e 8 | C12 | — |
| R10 | Dado simulado usado como real | Número fictício vira decisão de CAPEX ou de manutenção | Financeiro e legal (3) | 2 | 6 | Marcar "simulado" em todo registro; aviso no README e no prompt | Regra 5 e FORMATO | Blocos 6 e 7 | C02 | Campo origem em cada chunk e registro |
| R11 | Tool indisponível ou com erro | Timeout ou ativo não encontrado e o modelo completa a lacuna | Operacional e financeiro (2) | 3 | 6 | Erro estruturado; informar falha; não estimar; encaminhar responsável | Regras 1 e 9 | Blocos 5, 6 e 7 | C06 | Contrato de erro no tools_spec |

## Riscos por impacto

- **Segurança:** R01, R03, R04, R08.
- **Financeiro:** R01, R06, R10, R11.
- **Legal:** R03, R04, R05, R06, R07, R10.
- **Reputacional:** R02, R05, R07.
- **Operacional:** R08, R09, R11.

## LGPD — o que entra e o que nunca entra

A base é operacional, não de pessoas. O dado de ativo (TAG, km, criticidade, falha, procedimento) não identifica pessoa natural e pode entrar. Placa de veículo, imagem de ocupante e dado de empregado são dado pessoal e ficam fora.

| Entra | Nunca entra |
|---|---|
| TAG, tipo, modelo, km, praça, criticidade | Nome, matrícula, cargo, salário ou contato de empregado |
| Sintoma, causa, tempo de reparo, peça trocada | Imagem de ocupante de veículo |
| POP, ficha técnica, norma resumida | Placa completa sem anonimização |
| Métrica agregada por ativo | Vídeo ou frame com pessoa identificável |
| Boletim de campo sem dado pessoal | Dado contratual sigiloso ou arrecadação por praça |
| Dado simulado marcado como simulado | Credencial, token, chave de API, `.env` |

**Princípios aplicados:** finalidade (manutenção de ativo), minimização (só o necessário para diagnosticar), necessidade (não pedir dado pessoal), segurança (modelo local e base versionada sem dado real) e transparência (origem e fonte em cada resposta). Placa, quando indispensável para contexto, entra mascarada no formato ABC-1\*\*\* e não é reproduzida na resposta. Não há decisão automatizada sobre pessoa: a saída é recomendação técnica revisada por humano. Retenção: histórico de conversa fica no ambiente local e é descartado ao fim do atendimento; nada é enviado a provedor externo com dado real.

## Ética e responsabilidade

- A decisão final é sempre humana. O assistente recomenda, documenta e aponta a fonte; não libera, bloqueia ou intervém.
- Hipótese não é diagnóstico. O nível de confiança e o rótulo FATO/HIPÓTESE existem para impedir que uma inferência vire ordem de serviço fechada.
- Prioridade segue critério publicado na matriz de criticidade, não a preferência do modelo.
- Toda recusa é curta, educada e aponta o caminho correto (POP, responsável ou profissional habilitado).
- Uso de IA para redigir documentação foi permitido pelo enunciado, mas nenhum output de modelo foi declarado como execução real. Os dados da base são simulados e identificados.

## Rastreabilidade rápida

| Se este risco acontecer... | ...olhe aqui primeiro |
|---|---|
| Resposta inventou MTBF | Regra 5, bloco 7 do fluxo, caso C02, tool calcular_metricas_confiabilidade |
| Modelo obedeceu texto colado | Regra 8, bloco 2, caso C09, guardrail de entrada |
| Resposta mandou mexer no painel | Regra 3, bloco 7, caso C10, POP de desligamento |
| Resposta trouxe placa ou nome | Regra 10, bloco 7, lista de exclusão do inventário |
| Conversa trocou de ativo | BORDA, blocos 3 e 8, caso C12 |
| Número simulado foi tratado como real | Regra 5, origem do chunk, aviso do README |
