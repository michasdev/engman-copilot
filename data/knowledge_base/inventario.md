# Inventário da base de conhecimento — EngMan Copilot

Versão sprint 1. Recorte A (Diagnóstico e Confiabilidade), ativos CFTV/PTZ, PMV, callbox, SAT e SCA. Origem: o briefing da Motiva é real; todo documento operacional abaixo é **simulado**, com dados fictícios plausíveis, e está identificado como simulado. Nenhum dado real da Motiva foi usado.

## Como a base é usada

A base alimenta o RAG (bloco 3 da arquitetura). Catálogo, histórico de falhas e métricas **não** são recuperados como chunk: vêm das tools, para não existir duas versões do mesmo dado. A base guarda o que é texto técnico: procedimento, ficha, norma, glossário, matriz de criticidade e contexto do briefing.

- **Chunk:** 600 tokens, overlap de 80. Justificativa: passo a passo e tabela técnica perdem sentido em chunk curto, e chunk longo gasta janela e piora a precisão da recuperação.
- **Recuperação:** top-4 trechos por similaridade, com filtro de metadados quando o usuário cita TAG ou tipo de ativo.
- **Embedding:** nomic-embed-text via Ollama, 768 dimensões, local.
- **Tamanho estimado:** 10 documentos, cerca de 60 chunks, 36 mil tokens. Não cabe inteiro no contexto operacional de 8 mil tokens, por isso o RAG é obrigatório.

## Metadados guardados em cada trecho

| Metadado | Exemplo | Uso |
|---|---|---|
| tag | CFTV-PTZ-KM412-01 | Filtra histórico e ficha do ativo citado |
| tipo_ativo | cftv | Restringe o POP e a ficha ao tipo correto |
| tipo_documento | pop / ficha_tecnica / norma / matriz / glossario / briefing | Prioriza o tipo de fonte na resposta |
| data | 2026-09-16 | Resolve conflito entre versões, priorizando a mais recente |
| versao | v1.0 | Auditoria e rastreabilidade |
| criticidade | alta | Apoia a priorização sem inventar |
| origem | real / simulado | Impede que dado fictício seja tratado como real |

## Documentos previstos

| # | Documento | Descrição e campos | Origem | Status | Chunks |
|---|---|---|---|---|---|
| 1 | Catálogo de ativos | Ficha por ativo: TAG, tipo, fabricante, modelo, número de série, km, praça ou pórtico, data de instalação, vida útil, criticidade, status operacional. 40 ativos simulados cobrindo os 5 tipos. Backend da tool consultar_ativo. | simulado | a criar | 12 |
| 2 | Plano de manutenção preventiva | Tarefa, ativo ou tipo, periodicidade, responsável, tempo estimado, recursos, condição de execução e critério de aceite. Cobre limpeza de lente, teste de comunicação, calibração e inspeção elétrica. | simulado | a criar | 8 |
| 3 | Histórico de falhas do FSM | TAG, data, sintoma, causa identificada, tempo de reparo, peça trocada, reincidência, condição ambiental. 60 registros simulados em 12 meses. Backend da tool consultar_historico_falhas. | simulado | a criar | 10 |
| 4 | Procedimentos operacionais (POPs) | Passo a passo de inspeção, limpeza, ajuste de foco, teste de comunicação e desligamento seguro, com pré-requisitos, EPI, ferramenta e critério de parada. 5 POPs, um por tipo de ativo. Backend da tool consultar_procedimento. | simulado | a criar | 12 |
| 5 | Fichas técnicas dos equipamentos | Especificação de CFTV, PMV, callbox, SAT e SCA: alimentação, comunicação, faixa de operação, limite de calibração, peças e vida útil. | real + simulado | parcial | 6 |
| 6 | Matriz de criticidade | Critério de impacto em arrecadação, segurança viária e disponibilidade, com peso e prioridade resultante (crítica, alta, média, baixa). | simulado | a criar | 2 |
| 7 | Glossário de siglas | CFTV, PMV, SAT, SCA, CCO, CBM, RCM, FSM, RGA, LDA, lambda, MTBF, MTTR, WIM, OCR, POP, OS. | real | pronto | 2 |
| 8 | Normas e requisitos de segurança | Resumo dos trechos aplicáveis de NR-10 e NR-35 e das regras internas de trabalho em via, sempre com aviso de que o documento oficial prevalece. | real (resumo) | pronto | 4 |
| 9 | Documentos do briefing Motiva | Contexto do desafio, descrição dos temas 1, 2 e 2.1, dores e números do cenário de 300 câmeras. | real | pronto | 3 |
| 10 | Boletins de ocorrência de campo | Registro textual do que o técnico relatou no atendimento, incluindo condição ambiental, suspeita e desfecho. 10 boletins simulados. | simulado | a criar | 1 |

Status: **pronto** = conteúdo já extraído do briefing ou escrito; **a criar** = documento especificado nesta Sprint, será produzido na Sprint 02; **parcial** = parte real do briefing, parte simulada.

## Regra de conflito entre fontes

Quando duas fontes discordam, o assistente mostra as duas, prioriza a mais recente e a mais específica para o ativo, informa a data de cada uma e pede validação humana. Norma oficial sempre prevalece sobre resumo interno. Tool sempre prevalece sobre texto solto da base para dado de ativo.

## O que NUNCA entra na base

- Nome, matrícula, cargo, salário, contato ou qualquer dado de empregado.
- Imagem de ocupante de veículo e vídeo de câmera com pessoa identificável.
- Placa de veículo capturada por OCR sem anonimização; quando necessária para contexto, entra mascarada (ABC-1***).
- Dado contratual sigiloso, proposta comercial, valor de arrecadação por praça.
- Credencial, token, chave de API ou `.env`.
- Laudo, ART ou parecer com assinatura de responsável técnico.
- Histórico de falha real da Motiva: nesta fase, só dado simulado.

Essa lista vale também para o prompt. A regra 10 do system prompt proíbe tratar dado pessoal, e o risco R05 da matriz aponta a mitigação.

## Como cada documento vira chunk

1. Extrair o texto e normalizar título, seção e tabela.
2. Quebrar em blocos de 600 tokens com overlap de 80, preservando o cabeçalho da seção.
3. Anexar os metadados da tabela acima.
4. Gerar embedding com nomic-embed-text.
5. Gravar no ChromaDB com `id = documento + seção + índice`.
6. Marcar `origem: simulado` em todo chunk operacional para o guardrail de saída poder avisar o usuário.
