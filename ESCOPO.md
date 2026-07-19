# Escopo do Projeto — Atendente IA para Pequenos Negócios

## Status
🟡 Em andamento — Fase 6

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
- [x] Telegram funcionando ponta a ponta
- [ ] WhatsApp via Evolution API (opcional, se houver tempo/recurso)
- **Critério de pronto:** conversa completa funcionando no canal escolhido — validado: pergunta sobre horário de funcionamento respondida corretamente via Telegram, com RAG ativo

### Fase 6 — Produção (dividida em 2 etapas)

**Etapa 6.1 — Agora (sem Docker, sem banco de dados)**
- [x] Memory por usuário (histórico de conversa) — via Simple Memory do n8n, sessionId = chat.id do Telegram
- [x] Fallback para humano — System Message instrui o modelo a prefixar respostas incertas com "FALLBACK", node IF (contains) desvia para mensagem padrão de transferência
- [x] Log de custo/latência — registrado em arquivo CSV local (`C:\n8n-logs\logs-atendente.csv`), via Edit Fields → Convert to File → Read/Write Files from Disk. Campos: data/hora, chat_id, pergunta, tokens (pendente de captura correta, hoje "N/A"), latência (pendente, hoje "N/A"), uso de fallback (sim/nao)
- **Critério de pronto:** as 3 funcionalidades operando via Telegram, sem intervenção manual

**Etapa 6.2 — Depois (Docker básico, quando houver tempo/disposição)**
- [ ] Instalar Docker Desktop (Windows, possivelmente via WSL2)
- [ ] Entender conceitos de imagem vs container
- [ ] Rodar o n8n dentro de um container Docker básico, sem persistência avançada ainda
- **Critério de pronto:** n8n rodando via Docker localmente, com o workflow funcionando igual à versão via npm
- **Fora do escopo desta etapa:** persistência de dados via volumes Docker e deploy em produção real (servidor/VPS) — fica para uma **v3**, planejada para depois que o conteúdo de banco de dados do curso avançar, já que os dois conceitos (dados persistentes em container + banco relacional) se conectam naturalmente.

**Critério de pronto da Fase 6 completa:** projeto rodando de forma estável, documentado como template reutilizável, com Docker básico validado

## Fora de escopo (por enquanto)
- Multi-idioma
- Painel administrativo visual
- Cobrança/pagamento automatizado

## Como reiniciar o ambiente (a cada nova sessão de trabalho)
> Necessário porque o Simple Vector Store perde dados ao reiniciar o n8n, e o ngrok gratuito gera uma URL nova a cada execução.

1. **Iniciar o túnel ngrok** (em um terminal):
   ```powershell
   .\ngrok.exe http 5678
   ```
   Copiar a nova URL `https://...ngrok-free.dev` gerada.

2. **Iniciar o n8n** (em outro terminal, com as variáveis de ambiente):
   ```powershell
   $env:NODE_OPTIONS="--dns-result-order=ipv4first"
   $env:WEBHOOK_URL="https://SUA-NOVA-URL-NGROK.ngrok-free.dev/"
   $env:N8N_RESTRICT_FILE_ACCESS_TO="C:\n8n-logs"
   n8n start
   ```
   (não esquecer a barra `/` no final da URL)

3. **Repopular o Vector Store**: abrir o workflow, clicar em "Execute workflow" a partir do node **"When clicking 'Execute workflow'"** (topo do canvas) para reinserir o FAQ na memória.

4. **Reativar o workflow**: conferir se o toggle/Publish do workflow está ativo — isso reconecta o Telegram Trigger à nova URL do ngrok automaticamente.

5. **Testar**: mandar uma mensagem no bot do Telegram (`@atendenteassump_bot`) e confirmar resposta.

## Decisões técnicas (atualizar conforme o projeto avança)
> Registrar aqui escolhas importantes e o porquê — útil para o README final e para entrevistas.

- **Fase 2:** Optamos pelo endpoint OpenAI do Azure (`/openai/v1/chat/completions`) em vez do endpoint de "projeto" do AI Foundry, por ser mais direto para chamadas HTTP REST puras (o endpoint de projeto é otimizado para uso via SDK). Autenticação feita via Credential do n8n (Header Auth), mantendo a API Key fora do JSON exportável do workflow — segurança para versionamento no GitHub.
- **Fase 3:** Parâmetros de tools que exigem cálculo/conhecimento do modelo (ex: coordenadas geográficas) precisam de descrições explícitas instruindo o modelo a fazer a conversão sozinho — sem isso, ele tenta passar o valor bruto (nome da cidade) e a API externa rejeita. O node "Code Tool" do n8n exige retorno em formato string, não objeto — diferente do HTTP Request Tool, que aceita JSON estruturado da API externa.
- **Fase 4:** Iniciada com **In-Memory (Simple) Vector Store** em vez de Postgres/pgvector, pois o conteúdo de banco de dados do curso ainda não foi cursado. Decisão consciente: valida o conceito de RAG (chunking, embeddings, similarity search) sem bloquear o progresso. Migração para pgvector prevista como evolução natural (v2), assim que SQL for estudado no semestre — sem necessidade de retrabalho da lógica de RAG em si.
- **Fase 4 (debugging):** Vector Stores em memória não compartilham dados entre workflows diferentes — inserção (Insert Documents) e busca (Retrieve Documents) precisam estar no MESMO workflow para acessar a mesma instância de memória. Sub-nodes de Tool (Vector Store como Tool, HTTP Request Tool, etc.) não devem ser testados isoladamente com "Execute step" a partir do zero do fluxo — isso trava esperando o node Webhook disparar de verdade. Testar via modal próprio ("Test Vector Store") ou via execução completa disparada pela Test URL do Webhook.
- **Fase 4 (armadilha de fluxo):** Depois de reconectar o AI Agent, sempre conferir se nodes de fases anteriores (ex: o HTTP Request fixo da Fase 2, ou o Edit Fields com valores fixos da Fase 1) não ficaram presos em série no meio do fluxo, descartando silenciosamente o output do node novo.
- **Fase 5:** Telegram Trigger exige webhook HTTPS — n8n local em `localhost` (HTTP puro) não é aceito pelo Telegram. Solução para desenvolvimento: túnel HTTPS via **ngrok**, apontando pra `localhost:5678`, com a variável de ambiente `WEBHOOK_URL` do n8n configurada para a URL pública do ngrok antes de iniciar (`n8n start`). Em produção (Fase 6), isso será substituído por domínio próprio com certificado real.
- **Fase 5 (debugging - rede):** Conexões do processo Node.js do n8n para `api.telegram.org` davam timeout, mesmo com o mesmo domínio acessível via navegador e PowerShell (`curl`/`Invoke-WebRequest`). Causa: Node.js tentando IPv6 primeiro em uma rede/provedor com IPv6 mal configurado. Solução: variável de ambiente `NODE_OPTIONS="--dns-result-order=ipv4first"` antes de iniciar o n8n, forçando IPv4 primeiro. Firewall do Windows e antivírus (Avast) foram descartados como causa após testes isolados.
- **Fase 5 (debugging - fluxo):** O modo de teste do editor n8n ("Listen for test event" + "Execute step" manual por node) não substitui o teste real — só o modo **Ativo/Published** do workflow processa mensagens do Telegram automaticamente, de ponta a ponta, sem intervenção manual em cada node.
- **Fase 5 (debugging - expressão):** O campo "Prompt (User Message)" do AI Agent precisa estar no modo **Expression** (não "Fixed"), senão `{{ $json.message.text }}` é enviado como texto literal para o modelo em vez de ser resolvido para o valor real — causando respostas genéricas e nenhuma tool sendo chamada.
- **Fase 6.1 (Fallback):** Modelos de linguagem não reproduzem formatos rígidos com 100% de consistência (variação de espaços, hífen, etc. em torno da palavra-chave definida no System Message). Detecção via node IF deve usar "contains" (não "is equal to"), procurando apenas a palavra-chave (ex: "FALLBACK"), nunca uma string exata — isso absorve variações naturais do modelo sem quebrar a lógica de desvio.
- **Fase 6.1 (Log de arquivo — debugging):** O node "Read/Write Files from Disk" retorna erro genérico "The file is not writable" mesmo com permissões de sistema corretas (confirmado via PowerShell `Add-Content` funcionando normalmente) — não é bug de Windows, antivírus ou permissão de pasta. Desde versões recentes, o n8n restringe por padrão o acesso a caminhos de disco por segurança. Correção: definir a variável de ambiente `N8N_RESTRICT_FILE_ACCESS_TO` apontando para a pasta desejada (ex: `C:\n8n-logs`) antes de iniciar o n8n. Nodes "Convert to File" (Text Input Field) e "Read/Write Files from Disk" (Input Binary Field) usam nomes de campo (ex: `linha_csv`, `data`), não expressões `{{ }}`.
- **Fase 6.1 (Log — pendências conhecidas):** A quebra de linha `\n` inserida via expressão no Edit Fields é gravada como caractere literal, não como quebra de linha real — cosmético, não impede o registro dos dados. Também pendente: captura de tokens reais (`tokenUsage`) e latência real (hoje "N/A" nos dois), a resolver em iteração futura sem bloquear o restante do projeto.
