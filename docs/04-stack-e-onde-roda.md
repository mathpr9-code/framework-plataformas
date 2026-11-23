# 3.8 Linguagem e stack

A linguagem é escolhida pelo tipo de trabalho e por quem vai manter, não por preferência. Esta seção é critério de escolha, sem fonte oficial que a sustente como regra.

| Tipo de trabalho | Candidatas | Por quê |
| --- | --- | --- |
| Interface web e backend de aplicação | TypeScript | Mesma linguagem no navegador e no servidor; tipos compartilhados entre os dois |
| Funções de borda e webhooks | TypeScript | É o que os runtimes de edge executam |
| Agentes de IA, dados, automação, integração com modelos | Python | Ecossistema de bibliotecas de IA e de dados |
| Serviço de alta concorrência e baixa latência | Go | Binário único, concorrência barata, consumo baixo |
| Regra que precisa ficar colada ao dado | SQL e funções no Postgres | Policies, constraints e triggers valem para qualquer cliente do banco |

Critérios de decisão:

1. O que o time já domina e consegue revisar.
2. Onde o código vai rodar e o que esse ambiente executa.
3. Se existe biblioteca madura para a integração principal.
4. Tipagem e teste: código escrito com agente de programação precisa de tipos e de suíte para ser revisado com segurança.
5. Número de linguagens: no máximo duas por plataforma sem ADR, uma por fronteira.

Regra de negócio em função de banco tem um custo: é mais difícil de testar e de versionar. Vale para o que é proteção de dado; não vale para fluxo de negócio extenso.

## Padrão de stack — proposta para aprovação

Ainda não existe padrão definido. Esta é uma primeira proposta, montada a partir do que já está em uso, para ser aprovada, ajustada ou recusada linha a linha. Até a aprovação, vale o critério da tabela acima.

| Camada | Padrão proposto | Sai do padrão quando |
| --- | --- | --- |
| Frontend | React com TypeScript | A interface é só conteúdo estático |
| Dados, autenticação e storage | Supabase (Postgres, Auth, Storage), com RLS | O cliente exige outro banco ou hospedagem própria |
| Backend de plataforma | Monolito modular em TypeScript | Há motivo de extração da seção 3.1 |
| Funções curtas e webhooks | Edge Functions em TypeScript | O trabalho passa do tempo máximo de execução |
| Agentes de IA e processamento de dados | Python, como serviço ou worker separado | O fluxo é simples o bastante para um orquestrador visual |
| Automação e integrações simples | n8n | A lógica precisa de teste automatizado e laços |
| Fila | Q1 (no banco) como primeira opção; Q2 (Redis) quando houver atraso, prioridade ou volume | Perder mensagem é inaceitável e não há quem opere |
| Onde roda | Escolhido por projeto entre as peças de deploy da seção 3.9; ponto de partida: D1 para frontend simples, Droplet da DigitalOcean (D3) quando houver backend, workers ou n8n | O cliente define o ambiente |
| Segredos | V1 e V2 em plataforma gerenciada; V3 quando houver VPS com mais de um serviço; V7 para pessoas | O cliente exige cofre específico |
| Observabilidade | Logs estruturados e alertas na plataforma; tracing de IA em LangSmith ou Langfuse | — |

Depois de aprovado, o padrão vira um ADR geral, e cada plataforma registra só os desvios.

# 3.9 Onde roda

| Opção | Para que serve | Custo e risco |
| --- | --- | --- |
| Hospedagem de frontend mais BaaS | Plataforma sem backend próprio ou com poucas funções | Limites do provedor; processamento longo não cabe |
| VPS ou droplet com containers | Backend próprio, workers, filas, n8n | Operação é sua: atualização, firewall, backup, monitoramento |
| Containers gerenciados | Backend próprio sem operar servidor | Custo maior; menos controle |
| Funções serverless | Trabalho por evento e picos | Tempo máximo de execução; conexões de banco por pool |

O que direciona: se há processamento longo ou worker; quem opera; onde os dados podem ficar; custo previsível contra custo por uso.

## Peças de deploy

Nenhum provedor é padrão do framework. A hospedagem gerenciada costuma ser o caminho mais fácil, não o único, e a escolha é por plataforma.

| Peça | O que é | Para que serve | Quando não usar | O que passa a ser sua responsabilidade |
| --- | --- | --- | --- | --- |
| D1 — Hospedagem gerenciada de frontend | Plataforma que publica a partir do repositório, com CDN, HTTPS e ambientes de preview (ex.: Vercel, Netlify, Cloudflare Pages) | Frontend e funções curtas; publicar rápido sem operar servidor | Processamento longo, worker, custo por uso imprevisível, exigência de hospedar em ambiente do cliente | Variáveis por ambiente, proteção dos previews, permissões da conta |
| D2 — Estático em storage com CDN | Arquivos do build servidos por armazenamento de objetos e CDN | Site ou aplicação de página única sem servidor próprio | Renderização no servidor, funções | Cabeçalhos de segurança, cache, invalidação, pipeline de publicação |
| D3 — VPS com containers e proxy reverso | Servidor próprio com Docker e um proxy na frente (ex.: Nginx, Caddy, Traefik) | Backend, workers, filas, n8n e também o frontend, tudo no mesmo ambiente; custo fixo | Sem ninguém para manter servidor atualizado | Sistema operacional, firewall, TLS, backup, monitoramento, atualização, rollback |
| D4 — PaaS auto-hospedado | Painel instalado no seu servidor que publica a partir do repositório (ex.: Coolify, Dokploy) | A experiência de "publicar ao dar push" no seu próprio VPS | Quando o painel vira mais uma coisa crítica sem dono | Tudo de D3, mais a segurança e a atualização do painel |
| D5 — Containers gerenciados | O provedor roda seu container e escala (ex.: Cloud Run, [Fly.io](http://Fly.io), Railway, Render) | Backend próprio sem operar servidor | Carga constante e alta, em que o custo por uso passa do custo fixo | Imagem, configuração, segredos, limites de escala e de gasto |
| D6 — Funções na borda ou serverless | Código por evento no provedor (ex.: Edge Functions do Supabase, Workers) | Webhooks, tarefas curtas, picos | Tempo de execução longo, estado em memória | Pool de conexões, idempotência, segredos por função |
| D7 — Orquestrador de containers | Cluster (ex.: Kubernetes) | Muitos serviços, vários times, alta disponibilidade exigida | Plataforma pequena; o custo de operar supera o benefício | Tudo |

As peças se combinam: frontend em D1 ou D2 com backend em D3 ou D5 é uma montagem comum, e D3 sozinho atende a plataforma inteira.

## O que direciona a escolha do deploy

| Pergunta | Aponta para |
| --- | --- |
| É só frontend com BaaS e a prioridade é publicar rápido? | D1 |
| Já existe VPS rodando n8n, banco ou workers? | D3 ou D4, para não espalhar ambientes; por exemplo, Droplet da DigitalOcean |
| O custo precisa ser fixo e previsível? | D3 ou D4 |
| Não há quem mantenha servidor? | D1, D5 ou D6 |
| O cliente exige hospedagem em infraestrutura dele ou em um país específico? | D3, D4 ou a nuvem que ele indicar |
| Há worker, fila ou processamento longo? | D3, D4 ou D5; D1 e D6 não atendem sozinhos |
| O tráfego é esporádico, com picos? | D5 ou D6 |

## O que todo deploy precisa ter, em qualquer peça

- [ ] Publicação por pipeline a partir de um commit revisado; ninguém publica da própria máquina.
- [ ] A suíte de erros e os testes rodam antes de publicar e bloqueiam em caso de reprovação.
- [ ] Ambientes separados, com segredos próprios; preview nunca aponta para dado de produção.
- [ ] HTTPS com renovação automática de certificado.
- [ ] Verificação de saúde e volta rápida para a versão anterior, testada.
- [ ] Migração de banco compatível com a versão anterior do código, para permitir a volta.
- [ ] Registro de quem publicou o quê e quando.

## DigitalOcean no framework

É o provedor de referência para VPS, e cobre quatro peças de uma vez: D3 (Droplet com containers e proxy), D5 (App Platform), D6 (Functions) e D7 (Kubernetes), mais banco gerenciado e storage. Por isso ele é o ponto de partida natural de "onde roda" quando a plataforma precisa de backend, worker ou n8n: um único painel, uma única fatura, crédito inicial para protótipo. O que direciona a saída dele: exigência do cliente por outra nuvem ou região, ou carga que justifique containers gerenciados com escala automática.

Montagem típica em DigitalOcean para uma plataforma: frontend em D1 (ou no próprio Droplet atrás do proxy); Droplet com Docker, proxy reverso, backend, workers e n8n (D3); Postgres gerenciado ou Supabase (dados); Spaces para arquivos; backups do Droplet ligados; firewall da nuvem restringindo as portas ao proxy.

## O que foi conferido de cada produto (12/11/2026)

Preços e limites mudam; a tabela registra o que estava publicado na data e de onde veio. Antes de fechar um projeto, confira de novo na página de preços do produto.

| Produto | Peça | O que foi conferido | Fonte |
| --- | --- | --- | --- |
| Vercel | D1 | O plano gratuito (Hobby) é restrito a uso pessoal e não comercial; projeto de cliente exige o Pro, a US\$ 20 por usuário por mês, mais uso. Funções com Fluid compute: 300 s no Hobby; no Pro, 300 s por padrão, configurável para mais | Documentação da Vercel (plano Hobby e limites de funções); o teto exato do Pro varia entre as fontes (800 s ou 13 min) |
| Netlify | D1 | Cobrança por créditos: Free com 300 créditos por mês e limite rígido; Personal a US$ 9 com 1.000; Pro a US$ 20 com 3.000 e membros ilimitados desde abril de 2026. Deploy, banda, computação e requisições consomem créditos | Changelog da Netlify e resumos que citam a página de preços |
| Cloudflare Pages e Workers | D1, D6 | Arquivos estáticos no Pages sem limite de requisições; funções contam no plano de Workers. Workers Free: 100 mil requisições por dia e 10 ms de CPU por chamada. Workers Paid: mínimo de US\$ 5 por mês, com 10 milhões de requisições e 30 milhões de ms de CPU incluídos; CPU de 30 s por padrão, até 5 min; memória de 128 MB em qualquer plano | Documentação da Cloudflare (preços e limites de Workers) |
| Supabase Edge Functions | D6 | 256 MB de memória; 2 s de CPU por requisição; tempo de relógio de 150 s no plano gratuito e 400 s nos pagos; 150 s sem resposta devolve erro 504. Até 100 segredos por projeto, de até 48 KiB cada | Documentação do Supabase (limites de Edge Functions) |
| Coolify | D4 | Código aberto (Apache 2.0); a versão auto-hospedada é gratuita com todos os recursos; Coolify Cloud a US$ 5 por mês para dois servidores, mais US$ 3 por servidor extra (o painel fica com eles, as aplicações nos seus servidores). Mínimo documentado: 2 núcleos e 2 GB de RAM | Página de comparação do próprio Coolify |
| Dokploy | D4 | Auto-hospedado gratuito; usa Docker, Traefik e tem clusterização por Docker Swarm; Dokploy Cloud a partir de US\$ 4,50 por servidor por mês. A licença é descrita como Apache 2.0 com uma parte proprietária; vale ler antes de uso comercial | Página de comparação do Coolify e resumos de terceiros; não conferido no site do Dokploy |
| Railway | D5 | Plano Hobby a US$ 5 por mês com US$ 5 de uso incluídos; cobrança por uso de CPU, memória e tráfego; serviços não dormem | Comparativos de terceiros, coerentes entre si |
| DigitalOcean Droplets | D3 | Droplet básico a partir de US$ 4 por mês (1 vCPU, 512 MB, 10 GB SSD, 500 GB de transferência); US$ 6 com 1 GB de RAM; cobrança por segundo desde janeiro de 2026, mínimo de 60 s; backups a 20% do preço do droplet; sem camada gratuita permanente, crédito de US\$ 200 por 60 dias para contas novas | Comparativos de terceiros que citam a página de preços (um deles conferido em 3/11/2026); a página oficial não foi aberta |
| DigitalOcean App Platform | D5 | Três sites estáticos gratuitos; aplicação dinâmica a partir de US\$ 5 por mês por container; cobrança por tamanho de container | Comparativos de terceiros |
| DigitalOcean Managed Databases | Dados | Postgres, MySQL e Valkey a partir de US$ 15 por mês em nó único; alta disponibilidade a partir de US$ 60 | Comparativos de terceiros que citam a documentação de preços |
| DigitalOcean Spaces | Dados (storage) | Armazenamento de objetos compatível com S3, com CDN, a partir de US\$ 5 por mês (250 GB mais 1 TB de transferência) | Comparativos de terceiros; uma fonte cita US\$ 21, que parece ser outro plano |
| DigitalOcean Functions | D6 | Cobrança por uso, com cota gratuita mensal (as fontes divergem entre 25 mil e 90 mil GiB-segundos) | Comparativos de terceiros, com divergência |
| DigitalOcean Kubernetes | D7 | Plano de controle gratuito; nós a partir de US\$ 12 por mês | Comparativos de terceiros |
| Render | D5 | Serviço web gratuito dorme depois de 15 minutos sem tráfego e leva de 30 a 60 s para voltar; serviço pago a partir de US\$ 7 por mês por serviço, preço fixo | Comparativos de terceiros, coerentes entre si |
| [Fly.io](http://Fly.io) | D5 | Cobrança por uso, sem plano fixo; as fontes divergem sobre a existência de camada gratuita para contas novas | Comparativos de terceiros, com divergência |
| Cloud Run | D5 | Não conferido | — |
| Nginx, Caddy, Traefik | D3 | Não conferidos; são proxies de código aberto, sem preço de licença | — |

Duas consequências práticas do que foi conferido:

- **Vercel gratuita não serve para projeto de cliente.** Ou entra o Pro no custo do projeto, ou a peça é outra.
- **Função de borda não é lugar de processamento pesado.** Os 2 s de CPU do Supabase e os 128 MB do Cloudflare decidem cedo se a operação vai para um worker.
