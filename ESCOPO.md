# Escopo do Projeto — Atendente IA para Pequenos Negócios

## Status
🟡 Em andamento — Fase 1

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
- [ ] n8n instalado e rodando local (npm)
- [ ] Entendido: Trigger → Node → Node
- [ ] Workflow simples: Webhook recebe requisição → responde JSON
- **Critério de pronto:** workflow testado com sucesso via requisição manual (Postman/curl)

### Fase 2 — Conectar Azure AI Foundry
- [ ] Node HTTP Request configurado com autenticação Azure
- [ ] Workflow recebe pergunta → retorna resposta do `gpt-4.1-mini`
- **Critério de pronto:** resposta correta e consistente em pelo menos 5 perguntas de teste

### Fase 3 — Tool Calling
- [ ] Node AI Agent configurado
- [ ] 1 tool reaproveitada (Open-Meteo) + 1 tool nova (ex: consulta de preços/horários)
- **Critério de pronto:** agente decide corretamente quando chamar cada tool, sem intervenção manual

### Fase 4 — RAG
- [ ] Vector Store configurado (a definir: Pinecone/Qdrant/pgvector)
- [ ] Base de conhecimento carregada (FAQ de negócio fictício ou real)
- **Critério de pronto:** resposta correta baseada em contexto, com top_k=3 validado

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

- *(vazio ainda — preencher a partir da Fase 1)*
