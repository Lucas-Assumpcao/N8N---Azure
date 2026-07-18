# Escopo do Projeto — Atendente IA para Pequenos Negócios

## Status
🟡 Em andamento — Fase 5

## Objetivo
Construir um atendente automatizado (WhatsApp/Telegram) que responde clientes de pequenos negócios usando IA, orquestrado via n8n e Azure AI Foundry (`gpt-4.1-mini`). Serve como projeto de portfólio e como serviço vendável (setup + mensalidade).

## Stack
| Camada | Ferramenta |
|---|---|
| Linguagem | JavaScript (Code nodes do n8n, nativo do ecossistema Node.js) |
| Orquestração | n8n (self-host via npm) |
| LLM | Azure AI Foundry — `gpt-4.1-mini` |
| Canal de entrada | Telegram (fase inicial) → WhatsApp (Evolution API, fase final) |
| Base de conhecimento | Markdown/FAQ → Vector Store (Pinecone/Qdrant/pgvector) |
| Deploy final | Docker (fase 6, ainda não iniciada) |

> Nota: Python fica de fora deste projeto de propósito — mantém o roadmap de LLM+Python separado e sem sobreposição de aprendizado.

## Estrutura do repositório
```
atendente-ia-n8n/
├── README.md
├── ESCOPO.md
├── faq-loja.md         ← base de conhecimento fictícia usada no RAG (Fase 4)
├── workflows/
│   ├── 01-fundamentos.json
│   ├── 02-azure-integration.json
│   ├── 03-tool-calling.json
│   ├── 04-rag.json
│   └── 05-canal-entrada.json
├── docs/
│   └── (prints/GIFs do workflow rodando, um por fase)
└── .env.example        ← variáveis de ambiente sem valores reais (nunca commitar credenciais)
```
> Cada fase concluída = exportar o workflow (JSON) e commitar em `workflows/`. Isso cria um histórico visual de evolução do projeto, útil para o portfólio.

## Fases e critério de "pronto"

### Fase 1 — Fundamentos do n8n
- [x] n8n instalado e rodando local (npm)
- [x] Entendido: Trigger → Node → Node
- [x] Workflow simples: Webhook recebe requisição → responde JSON
- **Critério de pronto:** workflow testado com sucesso via requisição manual (Postman/curl)

### Fase 2 — Conectar Azure AI Foundry
- [x] Node HTTP Request configurado com autenticação Azure
- [x] Workflow recebe pergunta → retorna resposta do `gpt-4.1-mini`
- **Critério de pronto:** resposta correta e consistente em pelo menos 5 perguntas de teste

### Fase 3 — Tool Calling
- [x] Node AI Agent configurado
- [x] 1 tool reaproveitada (Open-Meteo) + 1 tool nova (ex: consulta de preços/horários)
- **Critério de pronto:** agente decide corretamente quando chamar cada tool, sem intervenção manual

### Fase 4 — RAG
- [x] Vector Store configurado — **v1: Simple Vector Store (in-memory)**, nativo do n8n, sem SQL/Docker. Migração futura para Postgres/pgvector planejada para quando o conteúdo de banco de dados do curso avançar.
- [x] Base de conhecimento carregada (FAQ fictício de loja — horário, contato, endereço, pagamento, troca/devolução, entrega, garantia, fidelidade)
- **Critério de pronto:** resposta correta baseada em contexto — validado com pergunta sobre política de troca, resposta gerada bateu 100% com o conteúdo do FAQ indexado

### Fase 5 — Canal de entrada real
- [ ] Telegram funcionando ponta a ponta
- [ ] WhatsApp via Evolution API (opcional, se houver tempo/recurso)
- **Critério de pronto:** conversa completa funcionando no canal escolhido

### Fase 6 — Produção
- [ ] Memory por usuário (histórico de conversa)
- [ ] Fallback para humano
- [ ] Log de custo/latência
- [ ] Deploy via Docker
- **Critério de pronto:** projeto rodando de forma estável, documentado como template reutilizável

## Fora de escopo (por enquanto)
- Multi-idioma
- Painel administrativo visual
- Cobrança/pagamento automatizado

## Decisões técnicas (atualizar conforme o projeto avança)
> Registrar aqui escolhas importantes e o porquê — útil para o README final e para entrevistas.

- **Fase 2:** Optamos pelo endpoint OpenAI do Azure (`/openai/v1/chat/completions`) em vez do endpoint de "projeto" do AI Foundry, por ser mais direto para chamadas HTTP REST puras (o endpoint de projeto é otimizado para uso via SDK). Autenticação feita via Credential do n8n (Header Auth), mantendo a API Key fora do JSON exportável do workflow — segurança para versionamento no GitHub.
- **Fase 3:** Parâmetros de tools que exigem cálculo/conhecimento do modelo (ex: coordenadas geográficas) precisam de descrições explícitas instruindo o modelo a fazer a conversão sozinho — sem isso, ele tenta passar o valor bruto (nome da cidade) e a API externa rejeita. O node "Code Tool" do n8n exige retorno em formato string, não objeto — diferente do HTTP Request Tool, que aceita JSON estruturado da API externa.
- **Fase 4:** Iniciada com **In-Memory (Simple) Vector Store** em vez de Postgres/pgvector, pois o conteúdo de banco de dados do curso ainda não foi cursado. Decisão consciente: valida o conceito de RAG (chunking, embeddings, similarity search) sem bloquear o progresso. Migração para pgvector prevista como evolução natural (v2), assim que SQL for estudado no semestre — sem necessidade de retrabalho da lógica de RAG em si.
- **Fase 4 (debugging):** Vector Stores em memória não compartilham dados entre workflows diferentes — inserção (Insert Documents) e busca (Retrieve Documents) precisam estar no MESMO workflow para acessar a mesma instância de memória. Sub-nodes de Tool (Vector Store como Tool, HTTP Request Tool, etc.) não devem ser testados isoladamente com "Execute step" a partir do zero do fluxo — isso trava esperando o node Webhook disparar de verdade. Testar via modal próprio ("Test Vector Store") ou via execução completa disparada pela Test URL do Webhook.
- **Fase 4 (armadilha de fluxo):** Depois de reconectar o AI Agent, sempre conferir se nodes de fases anteriores (ex: o HTTP Request fixo da Fase 2, ou o Edit Fields com valores fixos da Fase 1) não ficaram presos em série no meio do fluxo, descartando silenciosamente o output do node novo.
