# Atendente IA para Pequenos Negócios

> Automação de atendimento via Telegram usando n8n como orquestrador e Azure AI Foundry como motor de IA. Projeto construído para aprendizado prático de automação com LLMs (tool calling, RAG, deploy containerizado) e como serviço vendável para pequenos negócios.

## 🎯 Motivação

Donos de pequenos negócios (lojas, clínicas, salões) gastam tempo respondendo as mesmas perguntas repetidamente: horário de funcionamento, formas de pagamento, política de troca, endereço. Este projeto automatiza esse primeiro nível de atendimento via Telegram, usando um LLM (Azure `gpt-4.1-mini`) capaz de:
- Responder com base em uma base de conhecimento real (RAG), sem inventar informação
- Consultar ferramentas externas quando necessário (ex: previsão do tempo)
- Reconhecer os próprios limites e transferir para um atendente humano quando não sabe responder
- Lembrar o contexto da conversa de cada cliente

O projeto nasceu como parte de um roadmap de estudo de automação com IA, e evoluiu para um MVP funcional, containerizado, documentado e pronto para ser oferecido como serviço.

## 🏗️ Arquitetura

```
Cliente → Telegram → n8n (Telegram Trigger)
                          ↓
                    AI Agent (Azure gpt-4.1-mini)
                          ↓
        ┌─────────────────┼─────────────────┐
   Tool Calling          RAG              Memory
  (clima, horário)   (Vector Store)    (por chat_id)
        └─────────────────┼─────────────────┘
                          ↓
                  IF (detecta FALLBACK?)
                    ↓             ↓
              Resposta normal   Transferência humana
                    ↓             ↓
                    Telegram (resposta ao cliente)
                          ↓
              Log de conversa (CSV local)
```

Todo o pipeline roda dentro de um container Docker do n8n, com túnel HTTPS (ngrok) para expor o webhook do Telegram durante o desenvolvimento local.

## 🛠️ Stack

- **n8n** — orquestração de workflow (rodando em container Docker)
- **Azure AI Foundry** — `gpt-4.1-mini` (chat) e `text-embedding-3-small` (embeddings)
- **Telegram** — canal de entrada e saída (Bot API)
- **Simple Vector Store** (n8n, in-memory) — base de conhecimento para RAG
- **Docker Desktop + WSL2** — ambiente de execução containerizado
- **ngrok** — túnel HTTPS para desenvolvimento local
- **JavaScript** — linguagem dos Code Nodes do n8n

## 📸 Demonstração

```
Cliente: Qual o horário de funcionamento?
Bot: Nosso horário de funcionamento é de segunda a sexta, das 9h às 18h,
     e aos sábados, das 9h às 13h. Posso ajudar com mais alguma coisa?

Cliente: E aos sábados?
Bot: Aos sábados, nosso horário de funcionamento é das 9h às 13h.
     Posso ajudar com mais alguma informação?

Cliente: Vocês vendem carros?
Bot: No momento não consigo te ajudar com essa informação por aqui.
     Um de nossos atendentes vai continuar sua conversa em breve! 😊
```

*(sugestão: substituir por prints reais ou um GIF curto do chat no Telegram, em `docs/`)*

## 🚧 Desafios técnicos e soluções

Esta seção documenta os problemas reais enfrentados durante o desenvolvimento — o valor está tanto no raciocínio de diagnóstico quanto na solução em si.

**Rede (IPv6 vs IPv4).** Conexões do processo Node.js do n8n para `api.telegram.org` davam timeout, mesmo o mesmo domínio sendo acessível via navegador e PowerShell. Causa: Node.js tentando IPv6 primeiro numa rede com IPv6 mal configurado. Resolvido com `NODE_OPTIONS="--dns-result-order=ipv4first"`.

**Webhook HTTPS.** O Telegram exige HTTPS para registrar webhooks, mas o n8n local roda em HTTP puro (`localhost`). Resolvido com túnel ngrok apontando para a porta local, combinado com a variável `WEBHOOK_URL` do n8n.

**Escopo de memória do Vector Store.** O Simple Vector Store (in-memory) não compartilha dados entre workflows diferentes — inserção e busca precisam estar no mesmo workflow para acessar a mesma instância de memória. Um debugging longo revelou isso ao investigar por que a busca vetorial retornava vazio mesmo após a inserção ter sido confirmada com sucesso.

**Consistência de LLM em formatos rígidos.** A estratégia de fallback humano depende do modelo prefixar respostas incertas com a palavra "FALLBACK". Como LLMs não são 100% determinísticos na formatação exata, a detecção no fluxo usa correspondência por "contains" em vez de igualdade exata — absorvendo variações naturais do modelo sem quebrar a lógica.

**Restrição de segurança do n8n para escrita em disco.** O node de escrita em arquivo retornava erro genérico de "arquivo não gravável" mesmo com todas as permissões de sistema corretas (confirmado testando escrita manual via PowerShell). A causa real: versões recentes do n8n restringem por padrão o acesso a caminhos de disco por segurança, exigindo a variável de ambiente `N8N_RESTRICT_FILE_ACCESS_TO` explicitamente.

**Migração para Docker.** Containerizar o n8n exigiu repensar a relação do processo com o sistema de arquivos: nada do Windows é visível de dentro do container a menos que seja montado explicitamente. Resolvido com um volume Docker gerenciado para dados internos do n8n (workflows, credenciais) e um bind mount para a pasta de logs, permitindo acesso tanto pelo container quanto pelo Windows.

*(lista completa e detalhada de decisões técnicas, incluindo debugging de WSL2 e PATH do Windows, disponível em [ESCOPO.md](./ESCOPO.md))*

## 📈 Métricas

- Tempo médio de resposta observado: ~2-8s por mensagem (varia conforme uso de tool calling/RAG)
- Log de conversas: data/hora, chat_id, pergunta, uso de fallback (registrado em CSV local)
- Captura de tokens e latência exatos: pendente de refinamento (ver Próximos Passos)

## 🔜 Próximos passos (v3 — não iniciada)

Planejados para quando o conteúdo de banco de dados do meu curso técnico for cursado, já que os dois conceitos se conectam naturalmente:

- Migrar o Vector Store de in-memory para **Postgres + pgvector**
- Trocar o banco interno do n8n (SQLite) por **Postgres** via `docker-compose`
- Persistência de execuções e histórico de conversas em banco relacional
- Deploy em servidor real (VPS) com domínio e certificado HTTPS próprios, substituindo o túnel ngrok
- Captura correta de tokens e latência reais no log
- Canal WhatsApp via Evolution API

## 📚 Aprendizados

Este projeto me levou por um ciclo completo de automação com IA: da chamada básica a uma API de LLM até tool calling, RAG, deploy containerizado e debugging de infraestrutura real (rede, permissões de sistema, containers). A parte mais valiosa não foi fazer o "caminho feliz" funcionar — foi diagnosticar, de forma metódica, cada travamento inesperado (timeouts de rede, memória isolada entre workflows, restrições de segurança do n8n) até a causa raiz, sem simplesmente tentar soluções aleatórias.

Também aprendi a importância de documentar decisões técnicas *durante* o processo, não depois — cada entrada no arquivo [ESCOPO.md](./ESCOPO.md) foi escrita no momento em que o problema foi resolvido, o que tornou este README muito mais fácil (e honesto) de escrever ao final.

---
*Escopo original (Fases 1–6) concluído. Acompanhe decisões técnicas e a v3 planejada em [ESCOPO.md](./ESCOPO.md)*
