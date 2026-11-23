# 8. Fase 6 — Segredos e criptografia

- [ ] Mapear API keys, tokens, senhas de banco, certificados, chaves privadas e chaves de criptografia.
- [ ] Manter segredos fora do código, frontend, Git, imagens Docker, URLs, respostas e logs.
- [ ] Usar secret manager/vault conforme ambiente; restringir leitura por serviço e pessoa.
- [ ] Separar credenciais por ambiente e por serviço.
- [ ] Implementar rotação, expiração e revogação; registrar procedimento para vazamento.
- [ ] Ativar secret scanning e push protection no repositório, quando disponível.
- [ ] Usar HTTPS/TLS em trânsito e criptografia em repouso do provedor.
- [ ] Usar criptografia por campo apenas quando a classificação e o risco justificarem.
- [ ] Não criptografar senha para depois descriptografar: armazenar hash de senha resistente (preferir Argon2id; usar o mecanismo do provedor quando ele gerencia autenticação).
- [ ] Separar chaves dos dados criptografados e definir rotação/backup das chaves.
- [ ] Não inventar algoritmos ou criptografia própria.

# 8.1 Cofres de segredos e API keys

Uma API key é identidade e, em serviços pagos como modelos de IA, também é dinheiro. O problema raramente é uma chave mal guardada; é a mesma chave espalhada em vários lugares, sem dono, sem validade e sem registro de quem leu.

## O que a OWASP recomenda

Do [Secrets Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html), lido por trechos:

- **Centralizar e padronizar** armazenamento, provisionamento, auditoria, rotação e gestão. Padronizar pode significar mais de um cofre, desde que o ciclo de vida seja o mesmo em todos.
- **Menor privilégio por segredo:** controle de acesso em cada objeto; engenheiros não têm acesso a todos os segredos do cofre.
- **Automatizar a rotação,** porque a manual gera erro. O padrão tem quatro passos: criar o novo segredo, definir, testar, finalizar.
- **Segredos dinâmicos sempre que possível:** credencial gerada por sessão, com validade curta, que expira sozinha se for roubada.
- **O segredo que abre o cofre fica em outro lugar.** Sempre existem pelo menos dois sistemas.
- **Serviços não compartilham o mesmo segredo,** ou não dá para saber de onde veio um vazamento.

## Peças de cofre

| Peça | O que é | Para que serve | Limite |
| --- | --- | --- | --- |
| V1 — Variáveis do provedor de hospedagem | Segredos cadastrados no painel do host e injetados como variável de ambiente | Aplicação em plataforma gerenciada, poucos segredos | Sem rotação automática; auditoria de leitura limitada; uma cópia por ambiente e por projeto |
| V2 — Cofre dentro do banco | Segredos cifrados no Postgres, lidos por funções, triggers e webhooks (ex.: Supabase Vault) | Segredo que o próprio banco precisa usar | Não é cofre de infraestrutura; quem tem privilégio no banco lê |
| V3 — Gerenciador de segredos como serviço | Serviço central que guarda, versiona, audita e sincroniza segredos para os ambientes | VPS com vários serviços, vários ambientes, mais de uma pessoa | Mais um fornecedor com acesso aos segredos; custo |
| V4 — Secret manager da nuvem, com KMS | Cofre do provedor de nuvem, acessado pela identidade da carga de trabalho | Plataforma que roda nessa nuvem; dispensa chave estática para ler o cofre | Prende ao provedor; exige configurar identidade e permissão |
| V5 — Cofre próprio | Servidor de cofre operado por você, com segredos dinâmicos | Muitos serviços, exigência de credencial de curta duração | Operar o cofre é um trabalho; ele vira ponto único de falha |
| V6 — Credenciais do orquestrador | Gerenciador de credenciais do n8n, protegido pela chave de criptografia da instância | Credenciais usadas dentro dos fluxos | Só serve aos fluxos; a chave de criptografia precisa de cofre próprio e backup |
| V7 — Cofre de pessoas | Gerenciador de senhas do time | Logins de painéis, códigos de recuperação, o segredo raiz dos outros cofres | Nunca é lido por aplicação |

As peças se combinam. Uma plataforma típica usa V1 para o frontend e as funções, V2 para o que o banco chama, V6 se houver n8n e V7 para as pessoas. V3, V4 ou V5 entram quando há backend próprio em servidor.

## O que direciona a escolha

| Pergunta | Aponta para |
| --- | --- |
| A aplicação roda só em plataforma gerenciada, com poucos segredos? | V1 |
| O banco precisa chamar um serviço externo? | V2 para esse segredo |
| Há VPS com vários serviços e ambientes? | V3 |
| Roda em uma nuvem com identidade de workload? | V4 |
| O cliente exige credencial de curta duração ou auditoria de cada leitura? | V4 ou V5 |
| Mais de uma pessoa precisa de acesso, com níveis diferentes? | V3 em diante; V1 não separa bem |
| Agente de programação trabalha no repositório? | Qualquer peça que entregue o segredo ao processo sem arquivo legível no workspace (seção 10.1) |

## Produtos de cofre conferidos (12/11/2026)

| Produto | Peça | O que foi conferido | Fonte |
| --- | --- | --- | --- |
| Segredos de Edge Functions do Supabase | V1 | Até 100 segredos por projeto, de até 48 KiB; nomes não podem começar com `SUPABASE_` | Documentação do Supabase |
| Infisical | V3 | Núcleo de código aberto, auto-hospedável sem custo de licença; nuvem com camada gratuita; rotação, controle por papel e retenção de auditoria nos planos pagos; segredos dinâmicos em plano superior | Blog do Infisical e resumos de terceiros; os preços divergem entre as fontes |
| Doppler | V3 | Serviço proprietário; camada gratuita para poucos usuários; rotação automática e SSO em plano pago; auto-hospedagem e segredos dinâmicos só no plano Enterprise | Resumos de terceiros |
| AWS Secrets Manager | V4 | Cobrança por segredo por mês (US\$ 0,40) mais chamadas de API | Resumos de terceiros, coerentes entre si |
| Bitwarden Secrets Manager | V3 | Código aberto; plano gratuito com segredos ilimitados, até 2 usuários, 3 projetos e 3 machine accounts; Teams a US$ 6 por usuário por mês (20 machine accounts) e Enterprise a US$ 12 (50, com opção de auto-hospedar); US\$ 1 por machine account extra; acesso por CLI (`bws`) e integrações de CI | Página de planos da Bitwarden |
| Bitwarden (gerenciador de senhas) | V7 | Código aberto. Uso individual: plano gratuito; Premium a US\$ 19,80 por ano. Uso de empresa: Teams a US\$ 4 e Enterprise a US\$ 6 por usuário por mês, cobrança anual, com teste grátis e sem plano gratuito de empresa; Enterprise permite auto-hospedar. Existe uma organização gratuita de compartilhamento limitada a 2 pessoas e 2 coleções | Página de preços e central de ajuda da Bitwarden; Premium por comparativo de terceiros (5/11/2026) |
| 1Password | V7 e V3 | Gerenciador de senhas pago, com CLI (`op`): o arquivo `.env` guarda referências `op://cofre/item/campo` em vez de valores, e `op run` resolve as referências e entrega os segredos só ao processo que ele inicia; service accounts para automação; Environments em beta | Documentação de desenvolvedor da 1Password |
| dotenvx | Formato, não cofre | Cifra os valores do `.env` com chave pública (secp256k1, ECIES com AES-256-GCM); o `.env` cifrado pode ir para o Git; a chave privada fica em `.env.keys` (cabeçalho `!DOTENV_PRIVATE_KEYS!`), que não pode ser versionado; uma chave por ambiente (`DOTENV_PRIVATE_KEY_PRODUCTION`, `DOTENV_PRIVATE_KEY_CI`) | Documentação do pacote e tutoriais de terceiros |
| Google Secret Manager, Azure Key Vault, HashiCorp Vault, gerenciador de credenciais do n8n | V4, V5, V6 | Não conferidos | — |

## Bitwarden, Bitwarden Secrets Manager, Infisical e Doppler: qual é qual

| Produto | Guarda o quê | Quem lê | Código aberto e auto-hospedagem | Peça |
| --- | --- | --- | --- | --- |
| Bitwarden (gerenciador de senhas) | Logins, senhas, códigos de recuperação, notas seguras | Pessoas, pelo app e pela extensão do navegador | Sim | V7 |
| Bitwarden Secrets Manager | Chaves de API, strings de conexão, certificados de aplicação | Máquinas, por machine accounts e CLI (`bws`) | Sim; auto-hospedagem no plano Enterprise | V3 |
| Infisical | Segredos de aplicação, por projeto e ambiente | Máquinas e desenvolvedores, por CLI, SDK e integrações | Núcleo aberto, auto-hospedável sem custo de licença | V3 |
| Doppler | Segredos de aplicação, por projeto e ambiente | Máquinas e desenvolvedores, por CLI e integrações | Proprietário; auto-hospedagem só no Enterprise | V3 |

Os dois produtos da Bitwarden são serviços diferentes na mesma conta: o gerenciador de senhas é para gente; o Secrets Manager é para aplicações e agentes, dentro de uma organização. Infisical e Doppler fazem o mesmo papel do Secrets Manager e não substituem o gerenciador de senhas. A diferença entre eles: o Infisical pode rodar no seu Droplet sem pagar licença; o Doppler é só serviço, mais polido, e cobra pelo que o Infisical entrega aberto.

## Arquivo .env cifrado (dotenvx e similares)

O dotenvx é do mesmo autor do dotenv, a biblioteca de `.env` mais usada no ecossistema Node. Ele não é um cofre: é um jeito de guardar o `.env` cifrado dentro do repositório. O problema só muda de lugar, para a chave privada, e o framework trata assim:

| Arquivo | Pode ir para o Git? | Onde fica |
| --- | --- | --- |
| `.env`, `.env.production` com valores `encrypted:` | Sim | Repositório |
| `.env.keys` com `DOTENV_PRIVATE_KEY` | Nunca | Fora da pasta do projeto, em um cofre (V3 ou V7); em produção e CI, como variável de ambiente do provedor (V1) |

O ponto que mais importa com agentes de código: o agente lê o `.env` cifrado sem problema, mas se o `.env.keys` estiver dentro da pasta do projeto, ele lê a chave também e o ganho some, porque `.gitignore` não impede leitura. Regra: `.env.keys` nunca fica no workspace; a chave entra no processo na hora de rodar, vinda do cofre (por exemplo `op run -- dotenvx run -- npm start`, com `DOTENV_PRIVATE_KEY` como referência no 1Password, ou o equivalente com `bws run`).

Quando usar: projeto pequeno, uma ou duas pessoas, que quer o `.env` versionado e sem serviço externo. Quando não usar sozinho: vários serviços, rotação frequente ou exigência de auditoria de quem leu cada segredo; aí entra V3.

## Configuração por projeto (Bitwarden Secrets Manager e bws)

Feita uma vez por computador e uma vez por projeto. O passo a passo completo, com telas e comandos, fica no guia de senhas e segredos do projeto.

| Quando | O quê | Quem |
| --- | --- | --- |
| Uma vez por computador | Instalar o `bws` (releases oficiais da Bitwarden); conta de máquina `dev-local` criada; token gerado e guardado no Password Manager | Responsável |
| Uma vez por projeto | Criar o projeto no Secrets Manager; dar "Pode ler" para `dev-local`; cifrar o `.env` com dotenvx; copiar as linhas do `.env.keys` para segredos de mesmo nome; anotar o id do projeto no `CLAUDE.md` e no script `dev:seguro`; tirar o `.env.keys` da pasta | Responsável nos passos do Bitwarden e do `.env.keys`; agente de código no script e no `CLAUDE.md` |
| Toda vez que for rodar | Definir `BWS_ACCESS_TOKEN` só no terminal que roda o projeto e rodar `npm run dev:seguro` | Responsável |

## Regra obrigatória no [CLAUDE.md](http://CLAUDE.md)

Todo projeto, de agente ou de plataforma, leva este bloco no `CLAUDE.md` (ou `AGENTS.md`) desde a Etapa 0. Ele é a instrução; a barreira técnica é o `.env.keys` estar fora da pasta e o token só existir no terminal do responsável. As duas coisas juntas.

```markdown
## Segredos — regra obrigatória

Este projeto usa .env cifrado com dotenvx. As chaves privadas ficam no Bitwarden Secrets Manager (projeto: <nome-do-projeto>, id: <id-do-projeto>). O id não é segredo.

O agente:
- pode ler e versionar o .env e o .env.production cifrados (valores "encrypted:");
- nunca abre, lê, imprime, copia ou resume .env.keys, nem qualquer arquivo com chave privada, token ou senha;
- nunca roda comandos que exibem segredos: dotenvx decrypt, dotenvx get, bws secret get, bws secret list, printenv, env, Get-ChildItem Env:, echo de variáveis de ambiente;
- nunca roda bws run nem npm run dev:seguro; quem roda a aplicação com segredos é o responsável, no terminal dele;
- quando precisar de uma variável secreta nova, informa o nome e pede para o responsável rodar: npx dotenvx set NOME_DA_VARIAVEL
- se encontrar um segredo em texto (código, log, documento, commit), para, avisa o responsável e não repete o valor.
```

Para reforçar no Claude Code, acrescente regras de negação de leitura em `.claude/settings.json`:

```json
{
  "permissions": {
    "deny": ["Read(./.env.keys)", "Read(./**/.env.keys)"]
  }
}
```

Confira as regras com o comando `/permissions` dentro do Claude Code; a sintaxe de permissões pode mudar entre versões.

## Escolha padrão — proposta

| Para quê | Escolha | Alternativa | Por quê |
| --- | --- | --- | --- |
| Senhas e chaves das pessoas (painéis, provedores, bancos, códigos de recuperação) | Bitwarden (V7) | 1Password | Código aberto, plano gratuito, um dos dois mais usados no mercado; 1Password se quiser a melhor integração de desenvolvedor e aceitar pagar |
| Segredos das aplicações e dos agentes | Bitwarden Secrets Manager, plano gratuito (V3) | 1Password com `op run` e service account; Infisical auto-hospedado no Droplet | Mesmo fornecedor das senhas; o plano gratuito cobre um operador sozinho (3 projetos, 3 machine accounts); passa para Teams quando houver mais projetos |
| `.env` versionado no repositório | dotenvx, com a chave privada no cofre | Sem `.env` no repositório, só referências ao cofre | Mantém a configuração junto do código sem expor valores |
| Produção e CI | Variáveis do provedor (V1), alimentadas a partir do cofre | Integração nativa do cofre com o CI | O provedor injeta no processo; ninguém copia valor à mão |

**Notion, documentos, planilhas e chat não são cofre.** Eles não cifram o conteúdo de ponta a ponta, não registram quem leu cada valor, e entram em busca, histórico de versões, compartilhamentos e integrações, incluindo conectores de IA. O que pode ficar no Notion é o inventário de segredos (abaixo): nome, serviço, cofre, dono e validade, nunca o valor. Ao migrar, cada chave que esteve no Notion deve ser trocada (rotacionada) depois de ir para o cofre, porque o valor antigo continua no histórico da página.

## Regras para API keys

- [ ] Uma chave por ambiente e por serviço. Quando o custo é do cliente, uma chave por cliente.
- [ ] Escopo mínimo e restrição de uso (origem, domínio, modelo, endpoint) sempre que o provedor oferecer.
- [ ] Teto de gasto e alerta configurados no provedor, para toda chave de serviço pago.
- [ ] Chave de terceiro nunca vai ao navegador: o frontend chama o backend, e o backend chama o terceiro.
- [ ] Chave fornecida pelo cliente é cifrada ao guardar, nunca devolvida inteira (mostrar só os últimos caracteres), nunca registrada em log e apagável pelo cliente.
- [ ] Rotação com sobreposição: criar a nova, publicar, testar, revogar a antiga.
- [ ] Nenhuma chave em URL, mensagem de erro, trace, screenshot ou prompt.
- [ ] Chave sem uso é revogada; o último uso é conferido na revisão periódica.
- [ ] Vazou ou apareceu onde não devia: revogar primeiro, investigar depois.

## Inventário de segredos

Uma linha por segredo, sem o valor:

| Campo | Conteúdo |
| --- | --- |
| Nome | Nome da variável ou do segredo |
| Serviço e escopo | De qual provedor, com que permissão |
| Ambiente | Desenvolvimento, staging ou produção |
| Cofre | Em qual peça (V1 a V7) está guardado |
| Consumidores | Quais serviços e quais pessoas leem |
| Dono | Quem responde por ele |
| Criado em e validade | Quando nasceu e quando deve ser trocado |
| Rotação | Automática ou manual, com o procedimento |
| Último uso | Conferido na revisão |
