# Decisões técnicas — EngMan Copilot

Sprint 1, Recorte A. Critérios objetivos, comparação e trade-off aceito. A disciplina exige caminho de execução 100% gratuito; todas as opções abaixo têm caminho de custo zero.

## 1. Critérios usados

| Critério | O que foi medido |
|---|---|
| Custo | Preço por milhão de tokens ou custo de hardware para rodar local |
| Licença e termos | Licença do modelo e do banco vetorial |
| Qualidade em pt-BR | Desempenho conhecido em instrução e redação técnica em português |
| Janela de contexto | Tokens disponíveis por chamada |
| Privacidade | Se o dado sai da máquina e quem retém |
| Integração Python | Facilidade de chamar por HTTP/SDK e de rodar no ambiente do grupo |
| Maturidade | Documentação, comunidade e estabilidade da API |

## 2. Modelos de linguagem

| Critério | Qwen 2.5 7B Instruct (Ollama local) | Gemini 2.5 Flash (Google AI Studio, free tier) | inclusionai/ling-3.2-flash:free (OpenRouter) |
|---|---|---|---|
| Custo | R$ 0, roda na máquina do grupo | R$ 0 dentro da cota gratuita; acima disso é pago | US$ 0 no tier free; limite de requisições por dia |
| Licença | Apache 2.0 | Termos do Google AI Studio | Termos do OpenRouter e do provedor do modelo |
| pt-BR | Bom para instrução e redação técnica; erra menos com prompt em português | Muito bom, melhor redação e aderência a formato | Bom para conversa e instrução; menos previsível em texto técnico longo |
| Contexto | 32k tokens | 1M tokens | 128k tokens (suficiente para o recorte) |
| Privacidade | Total: nada sai da máquina | Dados vão para o Google; free tier pode usar para melhoria | Dados vão para o OpenRouter e para o provedor; não usar com dado real |
| Integração Python | HTTP local em `localhost:11434`, sem chave | SDK `google-genai` ou HTTP com chave | HTTP `urllib`/SDK, chave gratuita |
| Maturidade | Alta, Ollama e Qwen bem documentados | Alta, API estável | Média, modelo recente e tier free sujeito a mudança |
| Hardware | ~5 GB de RAM/VRAM para 7B quantizado | Nenhum | Nenhum |
| Veredito | **Escolhido para produção e dado real** | Alternativa de qualidade, se privacidade não for bloqueio | **Escolhido como caminho gratuito de teste**, com dado simulado |

**Escolha:** Qwen 2.5 7B Instruct via Ollama local. Motivos: custo zero real, licença permissiva, contexto de 32k suficiente para o orçamento de 8k tokens, integração HTTP simples e, principalmente, histórico de falhas não sai da rede. O modelo é pequeno para padrão de fronteira; por isso o fluxo usa RAG, tools e guardrail de saída para compensar.

**Trade-off aceito:** o Qwen 7B local escreve um português técnico um pouco pior que o Gemini Flash e pode errar mais em pergunta que exige raciocínio longo. Aceitamos essa perda para manter a privacidade e o custo zero. Em compensação, respostas são mais previsíveis com temperature 0.2 e o eval set existe para medir onde o modelo erra.

**Alternativa gratuita sem instalar modelo — OpenRouter.** Modelo `inclusionai/ling-3.2-flash:free`, página https://openrouter.ai/inclusionai/ling-3.2-flash:free. É o caminho mais rápido para testar o prompt em outra máquina: criar conta, gerar chave gratuita, exportar `OPENROUTER_API_KEY` e chamar `https://openrouter.ai/api/v1/chat/completions`. Limitações: cota diária do tier free, latência de rede e política de retenção do provedor. Por isso, só com dado simulado. O script de validação fica em `ia/extra_openrouter/assistente_openrouter.py`, fora do repositório da Sprint 01, que não pode ter `.py`. O repositório documenta o caminho, não embarca o código.

**Por que não GPT/Claude pagos:** a entrega precisa de caminho 100% gratuito e o histórico é sensível. Um modelo pago pode ser citado como opção, mas não pode ser a única forma de rodar.

## 3. Vector stores

| Critério | ChromaDB | FAISS |
|---|---|---|
| Custo | R$ 0, open source | R$ 0, open source |
| Licença | Apache 2.0 | MIT |
| Privacidade | Local, nada sai da máquina | Local, nada sai da máquina |
| Idioma e contexto | Não se aplica (busca vetorial) | Não se aplica (busca vetorial) |
| Persistência | Nativa, grava em disco sem código extra | Índice em arquivo, mas o CRUD e os metadados ficam por conta do grupo |
| Filtro por metadado | Sim, nativo (`where={"tag": ...}`) | Não nativo; exige índice auxiliar ou pós-filtro manual |
| Integração Python | `pip install chromadb`, API simples | `pip install faiss-cpu`, API de índice |
| Maturidade | Alta e crescendo | Muito alta em busca vetorial, mas é biblioteca de índice, não banco |
| Escala | Boa para dezenas de milhares de chunks | Excelente para milhões, com mais trabalho |
| Operação | Coleção persistente e consulta por similaridade | Exige serializar, versionar e recarregar o índice |

**Escolha:** ChromaDB. O fator decisivo é o filtro por metadado — TAG, tipo de ativo e tipo de documento — que o RAG precisa para não misturar ativos. O volume previsto (cerca de 60 chunks) está muito abaixo do ponto em que FAISS faria diferença. O trade-off é perder um pouco de desempenho bruto em escala grande, que não é o caso deste projeto.

## 4. Embeddings

| Critério | nomic-embed-text via Ollama | Sentence Transformers multilingual local |
|---|---|---|
| Custo | R$ 0 | R$ 0 |
| Privacidade | Local | Local |
| pt-BR | Multilíngue, bom em texto técnico | Bom, depende do modelo escolhido |
| Integração | Mesma stack do LLM | Biblioteca extra |
| Dimensão | 768 | 384 a 1024, conforme modelo |

**Escolha:** nomic-embed-text. Já vem no Ollama, mantém uma stack só e tem bom desempenho multilíngue. Trade-off: um modelo específico de português poderia recuperar melhor termo técnico raro; se o eval de recuperação mostrar falha, trocar é uma linha de configuração.

## 5. Orquestração e parâmetros

- **Orquestração:** chamada direta via HTTP/SDK, sem LangChain. O fluxo tem 8 blocos fixos e 5 tools; um framework adicionaria dependência, versão e camada de abstração sem ganho nesta fase. Se o projeto crescer para múltiplos agentes, LangChain ou LangGraph entram na Sprint 02.
- **Interface:** terminal interativo para validar o prompt. Mockup da interface em `docs/mockup.png`. Streamlit ou Gradio ficam para a próxima fase.
- **Parâmetros do modelo:** temperature 0.2, top_p 0.9, max_tokens 800, frequency_penalty 0.1. Temperature baixa reduz variação em resposta técnica; não elimina alucinação, por isso o guardrail de saída é obrigatório.
- **Histórico:** janela deslizante de 6 turnos, resumo a cada 6, TAG ativo reafirmado, orçamento de 8 mil tokens por chamada.
- **Ambiente:** Python 3.10+, VS Code ou Jupyter local. Ollama para LLM e embedding, ChromaDB para o índice.

## 6. Custo estimado

| Cenário | Custo por chamada | Observação |
|---|---|---|
| Local (escolhido) | R$ 0 | Energia e hardware já existentes; ~6 a 7 mil tokens por chamada |
| OpenRouter free | US$ 0 | Cota diária; sem garantia de disponibilidade |
| Gemini free tier | R$ 0 | Cota gratuita; dado sai da máquina |
| API paga (referência) | Alguns centavos por chamada | Só se o volume estourar a cota free |

Como o assistente faz de 3 a 10 chamadas por atendimento (incluindo retorno de tool), o custo local continua zero e o custo em API paga seria baixo, mas desnecessário para o volume de teste.

## 7. Decisão final

Stack de execução gratuita: **Qwen 2.5 7B Instruct (Ollama) + nomic-embed-text + ChromaDB + chamada direta em Python 3.10+**. Caminho alternativo de teste: **inclusionai/ling-3.2-flash:free via OpenRouter**, com dado simulado e script fora do repositório. O trade-off central é qualidade de redação versus privacidade e custo; escolhemos privacidade e custo, com RAG e guardrails para segurar a qualidade.
