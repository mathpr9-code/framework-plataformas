# 3. Fase 1 — Arquitetura e fronteiras de confiança

Desenhar o fluxo antes da implementação:

- [ ] Frontend e componentes que rodam no navegador.
- [ ] Backend/API/Edge Functions e regras de negócio.
- [ ] Provedor de identidade e ciclo de sessão.
- [ ] Banco, schemas, roles, RLS, views, funções e storage.
- [ ] Integrações, filas, webhooks e automações n8n.
- [ ] Serviços de terceiros, LLMs, RAG e ferramentas de agentes, se existirem.
- [ ] Secret manager, chaves de criptografia e variáveis de ambiente.
- [ ] Observabilidade: logs, métricas, alertas e auditoria.

Documentar quais componentes são confiáveis, quais recebem entrada não confiável e onde cada autorização é aplicada. Não presumir que uma chamada originada do frontend é confiável.

**Entregável:** diagrama de arquitetura, contratos entre componentes e ADRs das decisões relevantes.

# 3.1 Estilo de arquitetura

A primeira decisão de estrutura. Cada estilo existe para resolver um problema; escolher um estilo sem ter o problema traz só o custo.

| Estilo | O que é | Para que serve | Quando não usar | O que exige | Risco de segurança que traz |
| --- | --- | --- | --- | --- | --- |
| Frontend mais BaaS | Aplicação web falando com Supabase (banco, auth, storage, funções), quase sem backend próprio | CRUD com papéis, painéis, primeira versão rápida | Regra de negócio complexa, integrações pesadas, processamento longo | RLS impecável, porque o banco é a única barreira | Toda falha de policy é vazamento direto |
| Monolito | Uma aplicação, um deploy, um banco | Domínio pequeno, time de uma a três pessoas | Quando partes diferentes precisam escalar ou publicar em ritmos diferentes | Organização interna mínima | Uma falha expõe tudo; superfície única |
| Monolito modular | Um deploy, mas dividido em módulos por domínio, cada um com interface pública e dados próprios | Domínio médio a grande com um time; é o ponto de partida recomendado para plataforma com backend próprio | Quando um módulo precisa de runtime, escala ou isolamento que o processo único não dá | Disciplina de fronteira: módulo não lê tabela de outro, só chama a interface | Fronteira é lógica, não de rede: um módulo comprometido alcança os outros |
| Microsserviços | Serviços independentes, cada um com deploy e banco próprios, conversando por rede | Partes com escala, ritmo de publicação, tecnologia ou exigência de isolamento realmente diferentes | Time pequeno, domínio ainda mudando, sem automação de deploy e de monitoramento | Deploy automatizado, monitoramento, tratamento de falha de rede, consistência eventual | Autenticação entre serviços, mais segredos, mais superfície de rede |
| Funções serverless | Código que roda por evento, sem servidor para manter | Webhooks, tarefas curtas, picos esporádicos | Processamento longo, estado em memória, muitas conexões de banco sem pool | Pool de conexões em modo transação, idempotência | Cada função é um endpoint público a proteger |

## O que a literatura diz

Martin Fowler observa que quase todas as histórias de sucesso com microsserviços começaram com um monolito que ficou grande demais e foi dividido, e que microsserviços cobram um prêmio de custo e risco que só se paga em sistemas complexos ([MonolithFirst](https://www.martinfowler.com/bliki/MonolithFirst.html); [guia de microsserviços](https://www.martinfowler.com/microservices/)). Ele registra também que o caminho gradual para microsserviços funcionou onde já havia um bom desenho modular. O mesmo guia lista pré-requisitos: quem não tem competências de base de operação não deveria adotar o estilo.

A opinião não é unânime, e Fowler diz isso. Para o framework ela vale como ponto de partida, não como regra.

## O que direciona a escolha

| Pergunta | Aponta para |
| --- | --- |
| É CRUD com papéis e pouca regra fora do banco? | Frontend mais BaaS |
| Há regra de negócio, integrações e um time só? | Monolito modular |
| O domínio ainda está sendo descoberto? | Monolito modular; fronteira errada custa caro em microsserviço |
| Uma parte tem carga muito diferente das outras (importação, relatório, IA)? | Extrair só essa parte como worker ou serviço |
| Uma parte precisa de outra linguagem ou runtime? | Extrair só essa parte |
| Uma parte trata dado que exige isolamento (financeiro, saúde)? | Serviço separado com banco e credenciais próprios |
| Há times diferentes publicando em ritmos diferentes? | Microsserviços passam a se pagar |
| Há deploy automatizado, monitoramento e alguém para operar? | Se não, microsserviços estão fora |

Regra de extração: um serviço só sai do monolito com um motivo da tabela acima, nomeado no ADR. "Para ficar mais organizado" não é motivo; organização se resolve com módulo.

## Regras do monolito modular

- Um módulo por domínio do negócio, não por camada técnica.
- Cada módulo expõe uma interface pública; o resto é privado.
- Cada módulo é dono das suas tabelas, de preferência em schema próprio. Outro módulo não faz JOIN nelas; pede pela interface.
- Comunicação entre módulos por chamada de interface ou por evento interno.
- Um teste automatizado verifica as dependências permitidas entre módulos e quebra o build quando alguém fura a fronteira.
- Um módulo bem delimitado é o que torna a extração futura barata.

# 3.2 Comunicação síncrona e assíncrona

Síncrono é quando quem chama espera a resposta. Assíncrono é quando o pedido é registrado e processado depois. A escolha é por operação, não por plataforma.

| Sinal na operação | Escolha | Por quê |
| --- | --- | --- |
| Resposta em menos de um segundo e o usuário precisa dela para continuar | Síncrono | Mais simples de construir, testar e depurar |
| Leva mais que alguns segundos (relatório, importação, IA) | Assíncrono | Requisição longa estoura timeout e prende conexão |
| Depende de sistema externo instável | Assíncrono com retry | A falha do terceiro não vira falha para o usuário |
| Chega em picos (campanha, lote, webhook em rajada) | Assíncrono com fila | A fila absorve o pico e o processamento segue no ritmo que o sistema aguenta |
| Um evento dispara várias reações (e-mail, auditoria, integração) | Evento publicado, vários consumidores | Quem publica não precisa conhecer quem reage |
| Precisa ser tudo ou nada, na mesma transação | Síncrono, mesmo banco | Consistência forte só existe dentro de uma transação |

Como o usuário recebe o resultado de algo assíncrono:

| Forma | Como funciona | Serve quando |
| --- | --- | --- |
| Polling | O cliente pergunta de tempos em tempos se terminou | Simples; resultado em segundos ou minutos; poucos usuários esperando |
| Webhook | O servidor avisa o sistema do cliente quando termina | Integração entre sistemas |
| Tempo real (realtime, SSE, websocket) | O servidor empurra a atualização para a tela | Acompanhamento ao vivo, muitos eventos |
| Notificação | E-mail ou mensagem com o link do resultado | Processos longos, usuário não fica na tela |

O custo do assíncrono: o resultado não é imediato, a mesma mensagem pode ser processada duas vezes, a ordem pode não ser garantida, e depurar exige rastrear o pedido entre componentes. Por isso ele entra por operação e com os padrões da seção 3.3.

# 3.3 Filas, jobs e mensageria

## Peças

| Peça | O que é | Para que serve | Quando não usar |
| --- | --- | --- | --- |
| Q1 — Fila no próprio banco | Tabela de jobs lida com bloqueio de linha, ou extensão de fila do Postgres | Volume baixo a médio; um componente a menos para operar; o job entra na mesma transação do dado | Volume alto e contínuo, que passa a competir com as consultas da aplicação |
| Q2 — Fila em armazenamento rápido | Biblioteca de filas sobre Redis | Jobs com retry, atraso, prioridade e concorrência controlada | Quando perder um job é inaceitável e a persistência do Redis não foi configurada |
| Q3 — Broker gerenciado | Serviço de fila do provedor de nuvem | Volume alto, garantia de entrega, sem operar servidor | Plataforma pequena em VPS única |
| Q4 — Stream de eventos | Log de eventos ordenado e reproduzível | Muitos consumidores, reprocessamento histórico, análise | Quase sempre demais para plataforma pequena |
| Q5 — Agendador | Tarefas por horário | Rotinas, expiração, reenvio, fechamento | Não substitui fila para trabalho disparado por evento |

## Padrões obrigatórios em qualquer fila

| Padrão | O que faz | Por que é obrigatório |
| --- | --- | --- |
| Consumidor idempotente | Processar a mesma mensagem duas vezes produz o mesmo efeito que uma | Filas entregam ao menos uma vez; duplicata é certa, não exceção |
| Retry com espera crescente e variação aleatória | Tenta de novo esperando cada vez mais | Retry imediato derruba o serviço que está tentando voltar |
| Fila de mensagens mortas (DLQ) | Depois de N tentativas, a mensagem sai da fila principal e fica guardada | Uma mensagem defeituosa não pode travar as outras nem sumir |
| Tempo de visibilidade ou trava | Enquanto um worker processa, os outros não pegam a mesma mensagem | Evita processamento duplicado simultâneo |
| Outbox transacional | O evento é gravado na mesma transação do dado e publicado depois | Sem isso, o dado salva e o evento se perde, ou o contrário |
| Contexto na mensagem | A mensagem leva tenant, usuário de origem e id de correlação | O worker roda sem sessão; sem contexto, não há como autorizar nem rastrear |

## Padrões por situação

| Situação | Padrão | Fonte |
| --- | --- | --- |
| Pico de carga derruba o serviço | Nivelamento de carga por fila | [Queue-Based Load Leveling](https://learn.microsoft.com/en-us/azure/architecture/patterns/queue-based-load-leveling) |
| Fila cresce mais rápido do que é consumida | Vários consumidores na mesma fila | [Competing Consumers](https://learn.microsoft.com/en-us/azure/architecture/patterns/competing-consumers) |
| Alguns jobs não podem esperar atrás dos outros | Fila por prioridade | [Priority Queue](https://learn.microsoft.com/en-us/azure/architecture/patterns/priority-queue) |
| Operação atravessa vários serviços e precisa desfazer se falhar | Saga com compensação | [Saga](https://learn.microsoft.com/en-us/azure/architecture/patterns/saga) |
| Dependência fora do ar | Disjuntor | [Circuit Breaker](https://learn.microsoft.com/en-us/azure/architecture/patterns/circuit-breaker) |
| Falha passageira | Nova tentativa | [Retry](https://learn.microsoft.com/en-us/azure/architecture/patterns/retry) |

Os padrões são do catálogo do Azure Architecture Center, que os descreve como aplicáveis a qualquer sistema distribuído, não só ao Azure. Li o índice do catálogo, não cada página.

## O que direciona a escolha da fila

| Pergunta | Aponta para |
| --- | --- |
| Poucos jobs por minuto e já existe Postgres? | Q1 |
| Precisa de atraso, prioridade e muitos workers, e já existe Redis? | Q2 |
| Perder mensagem é inaceitável e não há quem opere servidor de fila? | Q3 |
| Vários sistemas precisam reler o histórico de eventos? | Q4 |
| É só uma rotina diária? | Q5, sem fila |

# 3.4 Banco: conexões, pool e multi-tenant

## Pool de conexões

Cada conexão ao Postgres custa memória e há um teto. Aplicação que abre uma conexão por requisição esgota o banco em pico. O pool mantém poucas conexões abertas e as empresta.

| Modo | Como funciona | Para que serve | Limite |
| --- | --- | --- | --- |
| Conexão direta | A aplicação fala com o Postgres | Backend persistente, com poucas instâncias | Não serve para serverless |
| Pool em modo sessão | Cada cliente segura uma conexão enquanto estiver conectado | Ferramentas e backends que precisam de recursos de sessão | Não multiplica conexões de verdade |
| Pool em modo transação | A conexão é emprestada só durante a transação | Serverless e edge functions, muitas conexões curtas | Recursos de sessão não funcionam |

No Supabase, segundo a [documentação de conexão](https://supabase.com/docs/guides/database/connecting-to-postgres): o modo transação é o indicado para serverless e edge functions e atende na porta 6543; o modo sessão do pooler compartilhado atende na 5432; o pooler compartilhado não suporta prepared statements em modo transação, então eles precisam ser desligados no driver (por exemplo `prepare: false` no Postgres.js e no Drizzle, `pgbouncer=true` no Prisma, `statement_cache_size=0` no asyncpg). A documentação de self-hosting acrescenta que o modo transação não suporta `SET` de sessão, `LISTEN/NOTIFY` nem advisory locks.

Consequência para segurança: em modo transação, o contexto de tenant ou de usuário usado pelas policies não pode ser definido por sessão, porque a próxima requisição pode receber a mesma conexão. Ele é definido dentro da transação e vale só para ela.

Outros controles de conexão: tamanho do pool compatível com o teto do banco; tempo máximo por consulta; tempo máximo de transação parada; usuário de banco da aplicação sem privilégio de administrador.

## Modelos de multi-tenant

| Modelo | Como funciona | Para que serve | Custo |
| --- | --- | --- | --- |
| Tabelas compartilhadas com `tenant_id` e RLS | Todos os clientes nas mesmas tabelas; a policy filtra | Muitos clientes pequenos; operação simples | Uma policy errada vaza entre clientes; exige teste de isolamento |
| Schema por cliente | Mesmas tabelas repetidas em um schema por cliente | Poucos clientes com customização; isolamento mais visível | Migrações multiplicadas; papéis e grants por schema |
| Banco por cliente | Um banco ou projeto por cliente | Exigência contratual ou regulatória de isolamento | Custo e operação por cliente |

O que direciona: número de clientes, exigência de isolamento em contrato, grau de customização por cliente e quem vai rodar as migrações.

## Migrações

Toda mudança de schema é um arquivo versionado, aplicado por pipeline, nunca pelo painel em produção. Mudança que quebra compatibilidade é feita em duas publicações: primeiro adicionar o novo, depois remover o antigo.

# 3.5 Tratamento e processamento de dados

| Situação | Peça | Por quê |
| --- | --- | --- |
| Dado digitado pelo usuário | Validação por schema na borda da API, e de novo no banco por constraint | Frontend valida para ajudar; servidor e banco validam para proteger |
| Planilha ou CSV de cliente (preços, cadastro, estoque) | Importação assíncrona em etapas: receber, validar, mostrar erros, confirmar, aplicar | Arquivo grande estoura requisição; aplicar sem conferência corrompe a base |
| Mesma importação enviada duas vezes | Chave de idempotência por arquivo ou por linha | Evita duplicar registro |
| Integração periódica com outro sistema | Carga em lote com marca do último ponto sincronizado e reconciliação | Recomeça de onde parou; detecta divergência |
| Evento que precisa refletir em segundos | Processamento por evento, com fila | Lote não atende o prazo |
| Lista grande na tela ou na API | Paginação por cursor, com limite máximo | Paginação por número de página degrada e permite extrair a base inteira |
| Exportação grande | Job assíncrono, arquivo em storage privado, link assinado com expiração | Exportação é o caminho mais fácil de vazamento em massa |
| Upload de arquivo | Envio direto ao storage com URL assinada, validação de tipo e tamanho, processamento em worker | Arquivo não passa pela memória da API |
| Relatório pesado | Tabela ou visão pré-calculada, atualizada por job | Consulta analítica não disputa com a operação |
| Dado pessoal em ambiente de teste | Dado fictício ou anonimizado | Regra da Fase 2 |

Toda importação, exportação e carga registra quem fez, quando, quantos registros e o resultado.

# 3.6 Resiliência

| Controle | O que faz | Por quê |
| --- | --- | --- |
| Timeout em toda chamada externa | Desiste depois de um tempo definido | Sem timeout, uma dependência lenta prende todas as conexões |
| Retry só em operação idempotente | Repete o que é seguro repetir | Repetir um pagamento não é seguro sem chave de idempotência |
| Disjuntor | Para de chamar a dependência que está falhando e testa de tempos em tempos | Evita agravar a falha e esgotar recursos |
| Compartimentos | Limite de recursos por tipo de trabalho ou por cliente | Um cliente ou uma rota não consome tudo |
| Limite de taxa e contrapressão | Recusa ou enfileira quando passa do que aguenta | Melhor recusar cedo do que cair |
| Degradação controlada | Com uma parte fora, o resto continua, com aviso | Indisponibilidade parcial em vez de total |
| Verificação de saúde | Endpoint que diz se o serviço e suas dependências respondem | Deploy e monitoramento dependem disso |
| Falha fechada | Erro inesperado resulta em negação e registro | O OWASP Top 10:2025 criou a categoria A10, tratamento incorreto de condições excepcionais, para esse problema |

# 3.7 Cache

| Regra | Por quê |
| --- | --- |
| Cache entra depois de medir, não antes | Cache esconde consulta ruim e adiciona um modo novo de falha |
| A chave inclui o tenant e, quando o dado varia por pessoa, o usuário | Sem isso, um cliente recebe o dado de outro |
| Todo item tem validade e regra de invalidação | Dado velho apresentado como atual é erro de negócio |
| Resposta privada nunca é cacheada na borda como pública | CDN serve a mesma resposta a todos |
| Decisão de autorização não é cacheada sem prazo curto e invalidação na revogação | Usuário removido continuaria com acesso |
| Segredo e dado sensível não entram em cache compartilhado | Cache costuma ter menos controle de acesso que o banco |
