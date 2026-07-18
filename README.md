# Atendente IA para Pequenos Negócios

> Automação de atendimento via WhatsApp/Telegram usando n8n como orquestrador e Azure AI Foundry como motor de IA. Projeto construído para aprendizado prático de automação com LLMs e como serviço vendável para pequenos negócios.

## 🎯 Motivação
*(escrever ao final: por que esse projeto, que problema ele resolve, para quem)*

## 🏗️ Arquitetura
*(inserir diagrama simples aqui — pode ser feito no Excalidraw ou mesmo ASCII)*

```
Cliente → WhatsApp/Telegram → n8n (Webhook)
                                  ↓
                          AI Agent (Azure gpt-4.1-mini)
                                  ↓
                    ┌─────────────┴─────────────┐
              Tool Calling                  RAG (Vector Store)
           (preços, horários)            (FAQ do negócio)
                                  ↓
                          Resposta ao cliente
```

## 🛠️ Stack
- **n8n** — orquestração de workflow
- **Azure AI Foundry** (`gpt-4.1-mini`) — modelo de linguagem
- **Telegram / WhatsApp (Evolution API)** — canal de entrada
- **Vector Store** (Pinecone/Qdrant/pgvector) — base de conhecimento
- **Docker** — deploy

## 📸 Demonstração
*(inserir aqui: GIF ou vídeo curto do fluxo funcionando + prints do workflow no n8n)*

## 🚧 Desafios técnicos e soluções
> Essa seção é a mais importante para recrutadores — mostra raciocínio, não só resultado.

*(preencher conforme avança — exemplos do que documentar:)*
- Como resolvi autenticação com Azure dentro do n8n
- Como decidi o valor de top_k na busca vetorial e por quê
- Erros de configuração e como identifiquei a causa

## 📈 Métricas
*(preencher na Fase 6 — custo médio por conversa, latência média, taxa de fallback para humano)*

## 🔜 Próximos passos
- [ ] *(o que ficou de fora do MVP e poderia evoluir)*

## 📚 Aprendizados
*(2-3 parágrafos ao final — o que esse projeto ensinou, conectando com o restante do seu roadmap de LLM+Python)*

---
*Projeto em desenvolvimento — acompanhe o progresso em [ESCOPO.md](./ESCOPO.md)*
