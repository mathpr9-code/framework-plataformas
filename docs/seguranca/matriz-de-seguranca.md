# Matriz de Segurança de Plataformas

> 🛡️ Matriz padrão para avaliar a segurança de cada plataforma. Marcar um controle como concluído exige evidência verificável: configuração, teste, captura sanitizada, log ou revisão de código. "Está implementado" sem teste não é evidência.

## Como utilizar

**Status por controle:** Não avaliado · Em andamento · Conforme · Não conforme · Não aplicável.

**Prioridade:** P0 = bloqueia publicação se não houver mitigação; P1 = alto risco, corrigir antes do release sempre que possível; P2 = importante; P3 = melhoria contextual.

Cada linha deve receber status, responsável, evidência, data da revisão e ação corretiva. Não guardar valores de tokens, cookies, senhas ou chaves na matriz.

## Quais seções se aplicam

As seções A a J valem para toda plataforma. As demais são ligadas pela arquitetura escolhida ([Framework, seção 3.11](../05-canvas-e-seguranca-por-decisao.md)):

| Se a plataforma tem | Seção obrigatória |
| --- | --- |
| IA | K |
| Fila, jobs ou agendador | M |
| Mais de um serviço, ou módulo com dado sensível | N |
| Pool de conexões, cache, importação ou exportação | O |
| Dado pessoal | P |
| Painel administrativo ou acesso de suporte | Q |

## Cobertura por referência

Onde cada risco das listas da OWASP é tratado nesta matriz. Serve para conferir que nada da lista ficou sem controle.

| OWASP Top 10:2025 | Seções |
| --- | --- |
| A01 Broken Access Control (inclui SSRF) | C, D, E, F (SSRF), Q |
| A02 Security Misconfiguration | G, I, O |
| A03 Software Supply Chain Failures | I, B.1, K |
| A04 Cryptographic Failures | H, B |
| A05 Injection | F, G, E |
| A06 Insecure Design | A, e o canvas do Framework |
| A07 Authentication Failures | C |
| A08 Software or Data Integrity Failures | I, M, O |
| A09 Security Logging and Alerting Failures | J |
| A10 Mishandling of Exceptional Conditions | F (falha fechada), M, e a seção 3.6 do Framework |

| OWASP API Security Top 10:2023 | Seções |
| --- | --- |
| API1 Broken Object Level Authorization | D |
| API2 Broken Authentication | C |
| API3 Broken Object Property Level Authorization | D, F |
| API4 Unrestricted Resource Consumption | F, K, M |
| API5 Broken Function Level Authorization | D, Q |
| API6 Unrestricted Access to Sensitive Business Flows | F |
| API7 Server Side Request Forgery | F |
| API8 Security Misconfiguration | G, I |
| API9 Improper Inventory Management | F |
| API10 Unsafe Consumption of APIs | F |

| OWASP Top 10 para aplicações com LLM (edição 2026, publicada em 3/8/2026) | Controle na seção K |
| --- | --- |
| LLM01 Prompt Injection | Prompt Injection |
| LLM02 Sensitive Information Disclosure | Data leakage |
| LLM03 Excessive Agency (subiu de 6 para 3) | Excessive agency; Tool authorization; Menor agência |
| LLM04 Supply Chain | Cadeia de suprimentos de modelos |
| LLM05 Data and Model Poisoning | Memória e base de conhecimento |
| LLM06 Unbounded Consumption (subiu de 10 para 6) | Cost/DoS |
| LLM07 Misinformation (subiu de 9 para 7) | Resposta sem suporte |
| LLM08 Hidden Context Exposure (era System Prompt Leakage; escopo ampliado para tudo que entra no contexto e o usuário não vê) | Vazamento de prompt de sistema |
| LLM09 Vector and Embedding Weaknesses | Vetores e embeddings; RAG access control |
| LLM10 Improper Output Handling (caiu de 5 para 10) | Output handling |

A edição 2026 substitui a 2025 que esta matriz mapeava até 11/11/2026. A ordem nova vem da combinação de voto de praticantes com dados de incidentes reais; o que mais sobe é agência excessiva, consumo sem limite e desinformação, exatamente os riscos de agente que age.

| OWASP MCP Top 10 (beta, 2025) | Seções |
| --- | --- |
| MCP01 Token Mismanagement and Secret Exposure | B, K (cadeia de suprimentos de modelos) |
| MCP02 Privilege Escalation via Scope Creep | D, K |
| MCP03 Tool Poisoning | K (tool authorization), I |
| MCP04 Software Supply Chain Attacks and Dependency Tampering | I, K |
| MCP05 Command Injection and Execution | K (execução de código gerado), F |
| MCP06 Prompt Injection via Contextual Payloads | K (prompt injection) |
| MCP07 Insufficient Authentication and Authorization | C, D, N |
| MCP08 Lack of Audit and Telemetry | J |
| MCP09 Shadow MCP Servers | K (cadeia de suprimentos), A (inventário) |
| MCP10 Context Injection and Over-Sharing | K (memória e base de conhecimento), O (cache por tenant) |

| OWASP Top 10 para aplicações agênticas (2026) | Controle na seção K |
| --- | --- |
| ASI01 Agent Goal Hijack | Prompt Injection; Agentes com ação |
| ASI02 Tool Misuse and Exploitation | Tool authorization; Excessive agency |
| ASI03 Identity and Privilege Abuse | Identidade e credencial do agente |
| ASI04 Agentic Supply Chain Vulnerabilities | Cadeia de suprimentos de modelos |
| ASI05 Unexpected Code Execution | Execução de código gerado |
| ASI06 Memory and Context Poisoning | Memória e base de conhecimento; Vetores e embeddings |
| ASI07 Insecure Inter-Agent Communication | Comunicação entre agentes |
| ASI08 Cascading Failures | Falha em cascata |
| ASI09 Human-Agent Trust Exploitation | Confiança humana na saída do agente |
| ASI10 Rogue Agents | Agente fora do objetivo |

Estado da verificação em 12/11/2026:

| Lista | Fonte oficial aberta | Conteúdo dos itens |
| --- | --- | --- |
| OWASP Top 10:2025 | Sim, página da OWASP | Confirmado na página |
| OWASP API Security Top 10:2023 | Sim, índice e tabela de riscos em owasp.org/API-Security (edição 2023) | Confirmado |
| OWASP LLM Top 10:2026 | Sim, página do recurso em genai.owasp.org (PDF bloqueia acesso automatizado) | Dez itens por resumos coerentes entre si (CSA, HackerDNA, Aembit) |
| OWASP Agentic Top 10:2026 | Sim, página do recurso em genai.owasp.org (PDF bloqueia acesso automatizado) | Dez itens por resumos coerentes entre si |
| OWASP MCP Top 10 (beta) | Sim, página oficial do projeto, lida na íntegra | Confirmado na página |
| Resolução CD/ANPD 15/2024 | O DOU bloqueia acesso automatizado; confirmado em apresentação oficial do gov.br que cita os artigos, e em seis escritórios coerentes entre si | Art. 6: três dias úteis para a ANPD; §3: complemento em vinte dias úteis; art. 9: titulares (uma fonte diz "sem demora injustificada", as demais dizem três dias úteis); art. 10: registro por cinco anos; pequeno porte em dobro |

Para auditoria formal, os PDFs da OWASP GenAI e o texto do DOU devem ser abertos manualmente; os links estão nas referências.

## Quem revisa

A matriz só vale com um nome e uma data. Em cada projeto:

| Papel | Quem | Quando | O que registra |
| --- | --- | --- | --- |
| Revisor de segurança | Responsável, em sessão separada da construção, sem o agente de código que escreveu o projeto | Gate da Etapa 0 (desenho), gate da Fase 9 (antes de produção), e a cada mudança de permissão, conector, fonte ou operação de escrita | Data, versão do quadro, cada controle P0 conferido com evidência, lista de P1 pendentes com prazo |
| Dono do projeto | Responsável pelo projeto | Sempre | Aceita ou recusa o resultado da revisão; não pode dispensar um P0 |
| Cliente | Responsável nomeado no contrato | Fase 0 e antes de produção | Papéis LGPD, autoridade das fontes, aprovações humanas exigidas |

Em time de uma pessoa, revisor e dono são a mesma pessoa em momentos separados, e o registro escrito é o que impede a revisão de virar formalidade. Quando houver uma segunda pessoa, o revisor passa a ser quem não construiu.

## A. Governança e desenho seguro

| Controle | Como verificar / evidência | Prioridade |
| --- | --- | --- |
| Escopo e ativos | Lista de sistemas, domínios, ambientes, serviços, dados e integrações. | P1 |
| Modelo de ameaças | Atores, ativos, fronteiras de confiança, caminhos de ataque e mitigação documentados. | P1 |
| Classificação de dados | Dados pessoais, sensíveis, financeiros, credenciais e retenção classificados. | P1 |
| Menor privilégio | Usuários, serviços, tokens e roles têm apenas permissões necessárias. | P0 |
| Separação de ambientes | Dev, staging e produção têm credenciais e dados isolados. | P1 |
| Secure by design | Riscos tratados no desenho, antes de implementar fluxos críticos. | P1 |
| Decisões de arquitetura | Estilo, filas, modelo multi-tenant e onde roda registrados em ADR, com as seções desta matriz que cada escolha ativa. | P1 |
| Fronteiras de confiança | Diagrama marca onde entra dado não confiável e onde cada autorização é aplicada, por seta. | P1 |
| Suíte de erros | Toda falha de segurança corrigida tem teste automatizado que roda antes de cada publicação. | P1 |
| Revisão assinada | Cada gate tem revisão de segurança registrada com nome, data e controles conferidos; gate sem revisão não fecha. | P0 |

## B. Chaves, credenciais e secrets

| Controle | Como verificar / evidência | Prioridade |
| --- | --- | --- |
| API keys no frontend | Bundle, DevTools e source maps não contêm segredos privilegiados. | P0 |
| Segredos no Git | Secret scanning, push protection e revisão do histórico; .env real fora do repositório. | P0 |
| Rotação após vazamento | Procedimento testado para revogar/substituir a credencial exposta. | P0 |
| Vault/Secret Manager | Segredos em serviço apropriado, com acesso restrito e auditoria. | P1 |
| Separação de chaves | Chaves de criptografia separadas dos dados e protegidas por KMS/Vault/HSM conforme risco. | P1 |
| Ambientes e escopos | Credenciais distintas por ambiente, aplicação e serviço. | P1 |
| Credenciais de CI/CD | Tokens com permissões mínimas e sem exposição em logs/artefatos. | P1 |
| Inventário e expiração | Dono, finalidade, validade, rotação e revogação documentados. | P2 |
| Cofre por componente | Para cada componente (frontend, funções, backend, workers, banco, n8n, CI) está definido em qual cofre ficam seus segredos; nenhum segredo vive só em arquivo local ou em mensagem. | P1 |
| Segredo raiz do cofre | A credencial que abre o cofre fica em outro sistema, com MFA, acesso de poucas pessoas e procedimento de recuperação testado. | P0 |
| Acesso por segredo | Permissão concedida por segredo ou por projeto, não ao cofre inteiro; ninguém lê todos os segredos de produção por padrão. | P1 |
| Auditoria de leitura | O cofre registra quem leu qual segredo e quando; leituras fora do padrão geram alerta quando o cofre permitir. | P1 |
| Rotação | Rotação automatizada, ou procedimento escrito e já executado uma vez: criar o novo, publicar, testar, revogar o antigo. | P1 |
| Uma chave por uso | Chave distinta por ambiente e por serviço; por cliente quando o custo ou o dado é do cliente. Serviços não compartilham a mesma chave. | P1 |
| Escopo e restrição da chave | Chave de terceiro criada com a menor permissão e com restrição de origem, domínio, modelo ou endpoint quando o provedor oferece. | P1 |
| Teto de gasto | Toda chave de serviço pago (modelos de IA, e-mail, SMS, mapas) tem limite de gasto e alerta configurados no provedor. | P1 |
| Chamada a terceiro pelo backend | Nenhuma chamada do navegador vai direto a API paga com chave privada; teste de rede confirma que a chave não trafega no cliente. | P0 |
| Chave fornecida pelo cliente | Cifrada ao guardar, nunca devolvida inteira pela API, ausente de logs e traces, apagável pelo cliente; teste de leitura pela API retorna só os últimos caracteres. | P0 |
| Chaves órfãs | Chaves sem uso ou sem dono são revogadas na revisão periódica; o último uso é conferido no provedor. | P2 |
| Chave de criptografia do orquestrador | A chave de criptografia do n8n está fora do banco e dos fluxos, em cofre, com backup e recuperação testada. | P0 |
| Segredo fora de ferramenta de notas | Nenhuma senha, chave de API, token ou string de conexão em Notion, documento, planilha ou chat; o que fica nessas ferramentas é só o inventário (nome, serviço, cofre, dono, validade). Segredo encontrado nesses lugares é movido para o cofre e rotacionado. | P0 |
| .env cifrado | Quando o `.env` vai cifrado para o repositório (dotenvx ou similar), a chave privada (`.env.keys`) não está no repositório nem na pasta do projeto acessível ao agente de código; entra no processo a partir do cofre. | P0 |

**Regra Supabase:** publishable/anon pode ser visível no cliente e só é segura com RLS/grants corretos. Secret/service_role é privilegiada e nunca deve ir ao frontend. "Public key" não equivale a permissão pública irrestrita; verificar sempre as políticas.

## B.1. Agentes de desenvolvimento e acesso a arquivos locais

Abrange Codex, Claude Code e qualquer agente/extensão que tenha acesso ao workspace, terminal, Git, rede, Docker, SSH ou ferramentas locais.

| Controle | Como verificar / evidência | Prioridade |
| --- | --- | --- |
| Inventário de capacidades | Registrar ferramentas habilitadas, escopo de arquivos, shell, rede, Git, Docker, SSH e conectores. | P1 |
| Sandbox e permissões | Revisar configuração real de sandbox, allow/deny rules, aprovações e limites do agente; não confiar apenas no prompt. | P0 |
| Acesso a .env | Agente não consegue ler .env, .env.local, .env.production e arquivos equivalentes com credenciais reais. | P0 |
| Diretórios sensíveis | Acesso a .ssh, home, pastas de credenciais, backups, dumps e cloud configs bloqueado salvo necessidade justificada. | P0 |
| Segredos fora do workspace | Credenciais reais ficam fora do diretório acessível ao agente sempre que possível; runtime/Vault fornece o segredo sem expor o valor ao modelo. | P1 |
| Credenciais de dev/staging | Agentes usam credenciais fictícias, descartáveis ou de escopo mínimo; nunca recebem .env de produção por padrão. | P0 |
| Instruções do projeto | CLAUDE.md, AGENTS.md e arquivos equivalentes contêm regras e nomes de variáveis, nunca valores secretos. | P1 |
| Saída e exfiltração | Chat, terminal, logs, diffs, screenshots, commits, PRs, artefatos e chamadas externas não revelam secrets. | P0 |
| Secret scanning pós-agente | Scanner executado nos arquivos alterados, diffs e artefatos antes de commit/push/deploy. | P1 |
| Aprovação de ações | Comandos destrutivos, alterações de infraestrutura, produção e operações privilegiadas exigem revisão/aprovação. | P1 |
| Teste de bloqueio | Teste com .env fictício prova que o agente não consegue ler nem transmitir o valor para destino externo. | P0 |
| Resposta a exposição | Procedimento de rotação/revogação iniciado quando segredo aparece em chat, saída, log, diff ou artefato. | P0 |

**Critério de aprovação:** não basta o agente declarar que não leu o arquivo. O controle precisa ser demonstrado por sandbox/permissões efetivas e teste em repositório descartável com valores fictícios.

**Atenção:** .gitignore protege contra inclusão acidental no Git; não impede o agente de ler um arquivo que esteja acessível no filesystem. Se o agente puder executar comandos irrestritos como o mesmo usuário, instruções no prompt não constituem barreira de segurança confiável.

**Evidência mínima:** configuração de permissões/sandbox, lista de caminhos bloqueados, resultado do teste com .env fictício, resultado de secret scanning e registro das credenciais de teste utilizadas (sem registrar seus valores).

## C. Identidade, autenticação e sessão

| Controle | Como verificar / evidência | Prioridade |
| --- | --- | --- |
| Autenticação server-side | Backend valida identidade/sessão/token em cada operação protegida. | P0 |
| Autorização server-side | Backend verifica papel, recurso, ação, ownership e tenant. | P0 |
| Login e recuperação | Fluxos de login, confirmação, reset e troca de e-mail testados. | P1 |
| MFA/passkeys | Disponível/aplicado conforme risco dos perfis privilegiados. | P1 |
| Hash de senha | Provedor ou algoritmo adequado (Argon2id/bcrypt/PBKDF2); nunca texto puro ou SHA-256 simples. | P0 |
| Cookies | Secure, HttpOnly e SameSite definidos conforme fluxo. | P1 |
| Ciclo de sessão | Rotação após login/privilégio, expiração, logout e revogação testados. | P1 |
| "Lembrar-me" | Persistência controlada, expiração e revogação; nunca salvar senha. | P1 |
| Proteção de tentativas | Rate limit, detecção de brute force/credential stuffing e enumeração mitigada. | P1 |
| Reautenticação | Operações de risco exigem confirmação adicional quando apropriado. | P2 |

## D. Autorização, papéis e multi-tenancy

| Controle | Como verificar / evidência | Prioridade |
| --- | --- | --- |
| RBAC/ABAC | Matriz de papel × recurso × ação documentada. | P1 |
| Isolamento por tenant | Dois tenants de teste não conseguem consultar nem alterar dados entre si. | P0 |
| BOLA/IDOR | Modificar IDs/referências em teste não revela objetos de outros usuários. | P0 |
| BFLA | Usuário comum não executa funções administrativas pela API. | P0 |
| BOPLA / Mass Assignment | Campos privilegiados não podem ser alterados enviando JSON extra; DTO/allowlist aplicada. | P0 |
| Negação por padrão | Acesso não explicitamente permitido é negado. | P0 |
| Revogação de acesso | Usuário desativado/removido perde acesso no prazo esperado. | P1 |

## E. Banco de dados e Supabase

| Controle | Como verificar / evidência | Prioridade |
| --- | --- | --- |
| RLS habilitada | Todas as tabelas expostas com dados privados estão cobertas. | P0 |
| Policies por operação | SELECT, INSERT, UPDATE e DELETE têm regras específicas. | P0 |
| USING / WITH CHECK | Leitura e valores novos/alterados são limitados corretamente. | P0 |
| Roles e grants | Roles sem privilégios excessivos; sem superuser para aplicação. | P1 |
| RPC, views e funções | SECURITY DEFINER, grants e validações revisados; nenhum caminho alternativo ignora policy. | P0 |
| Storage | Buckets públicos/privados e policies revisados; URLs assinadas com expiração adequada. | P1 |
| Consultas | Parametrização, limites, paginação e proteção contra SQL injection. | P0 |
| Backup e restore | Backup protegido e restauração testada em ambiente controlado. | P1 |
| Contexto de tenant com pool | Em pool de modo transação, o contexto usado pelas policies é definido dentro da transação; teste com duas requisições de tenants diferentes em sequência não mistura dados. | P0 |
| Usuário de banco da aplicação | Aplicação e workers conectam com usuário sem privilégio de administrador nem de ignorar RLS; migrações usam usuário separado. | P0 |
| Limites de consulta | Tempo máximo por consulta e por transação parada configurados; pool dimensionado abaixo do teto do banco. | P1 |
| Migrações | Mudanças de schema versionadas e aplicadas por pipeline; nenhuma alteração manual em produção sem registro. | P1 |
| Recuperação em ponto no tempo | Janela de recuperação conhecida e compatível com a perda de dados aceitável. | P2 |

## F. APIs, entradas e respostas

| Controle | Como verificar / evidência | Prioridade |
| --- | --- | --- |
| Inventário de endpoints | Endpoints, métodos, versões, público/privado e responsáveis documentados. | P1 |
| Validação de entrada | Schema, tipo, tamanho, formato, allowlist e limites testados. | P0 |
| Mass assignment | Apenas campos permitidos podem ser criados/alterados. | P0 |
| Resposta minimizada | API não retorna campos internos, segredos ou PII desnecessária. | P1 |
| SQL/command injection | Queries parametrizadas e ausência de execução insegura de comandos. | P0 |
| Rate limit e quotas | Limites por IP/usuário/tenant/rota/custo; endpoints caros protegidos. | P1 |
| CORS | Origens, métodos e headers estritos; credenciais apenas quando necessárias. | P1 |
| Webhooks | Assinatura, timestamp, prevenção contra replay e rotação de segredo. | P1 |
| Erros | Cliente recebe mensagem genérica; logs internos preservam detalhe sem segredos. | P1 |
| Timeout e payload | Limites para corpo, upload, execução, paginação e consumo. | P1 |
| Idempotência | Repetição de operações críticas não duplica efeito indevidamente. | P2 |
| SSRF | Endpoint que busca URL informada pelo usuário valida destino por lista permitida e bloqueia rede interna, loopback e metadata de nuvem. | P0 |
| Fluxos de negócio sensíveis | Cadastro, cupom, reserva, compra e outros fluxos abusáveis por automação têm limite por conta e por dispositivo, e detecção de uso anômalo. | P1 |
| Consumo de APIs de terceiros | Resposta de API externa é validada como entrada não confiável; redirecionamentos não são seguidos cegamente; timeout e limite de tamanho definidos. | P1 |
| Versões antigas e endpoints esquecidos | Versões desativadas, rotas de teste e ambientes de preview não ficam expostos com dados reais. | P1 |
| Falha fechada | Erro, timeout ou estado inesperado resulta em negação e registro; teste força a falha e confirma que nada é liberado. | P0 |
| Desserialização e caminho de arquivo | Dados serializados de fonte não confiável não são desserializados em objetos executáveis; nomes de arquivo não compõem caminho sem normalização. | P1 |

## G. Frontend, navegador e cookies

| Controle | Como verificar / evidência | Prioridade |
| --- | --- | --- |
| DevTools / Network | Requisições e respostas revisadas; tokens e PII não aparecem sem necessidade. | P1 |
| XSS | Conteúdo não confiável codificado/sanitizado; sinks perigosos revisados. | P0 |
| CSRF | Proteção coerente com autenticação por cookie e ações mutáveis. | P1 |
| Local/session storage | Não contém senhas, refresh tokens ou credenciais sem arquitetura justificada. | P1 |
| Source maps | Exposição avaliada; nenhum segredo ou informação sensível embutida. | P2 |
| Security headers | CSP, HSTS, X-Content-Type-Options, Referrer-Policy, Permissions-Policy e frame protection revisados. | P1 |
| HTTPS | Toda navegação e API usa TLS; redirecionamento para HTTPS validado. | P0 |
| Uploads | Tipo, tamanho, extensão, nome, armazenamento e download validados. | P1 |

## H. Criptografia e privacidade

| Controle | Como verificar / evidência | Prioridade |
| --- | --- | --- |
| TLS em trânsito | Browser, API, banco e integrações usam transporte criptografado. | P0 |
| Criptografia em repouso | Banco, storage, backups e dispositivos protegidos conforme provedor/risco. | P1 |
| Hash de senha | Algoritmo próprio para senha com salt e custo adequado; não reversível. | P0 |
| Dados sensíveis | Campos de alto impacto recebem proteção adicional quando necessário. | P1 |
| Gestão de chaves | Geração, acesso, rotação, revogação e recuperação definidos. | P1 |
| Pseudonimização/anonimização | Uso de dados de teste minimizados; reidentificação avaliada. | P1 |
| Logs e PII | Dados pessoais e segredos mascarados/minimizados e com retenção definida. | P1 |
| Retenção e descarte | Regras de retenção e exclusão aplicadas e verificadas. | P2 |

## I. Infraestrutura, Cloudflare e cadeia de suprimentos

| Controle | Como verificar / evidência | Prioridade |
| --- | --- | --- |
| WAF | Regras Cloudflare adequadas e revisadas sem depender delas como único controle. | P1 |
| Bot protection | Proteção de rotas sensíveis ajustada e testada contra falsos positivos. | P2 |
| Rate limiting na borda | Limites aplicados para rotas de login, APIs e operações caras. | P1 |
| Firewall e portas | Apenas serviços necessários expostos; painéis administrativos protegidos. | P1 |
| Administração | MFA e menor privilégio em todo provedor usado: hospedagem, DNS e borda, banco, repositório, servidor e painel de deploy. | P0 |
| Servidor próprio de deploy | Quando o deploy é em VPS: sistema atualizado, só as portas do proxy expostas, TLS com renovação automática, containers sem root, painel de deploy fora da internet pública ou com MFA. | P0 |
| Volta de versão | Retorno à versão anterior testado no ambiente em uso, incluindo a compatibilidade da migração de banco. | P1 |
| Ambientes de preview | Previews e ambientes temporários exigem autenticação e não usam segredo nem dado de produção. | P1 |
| Dependências | SCA/alertas de vulnerabilidade, lockfiles e atualizações revisadas. | P1 |
| SAST e secret scanning | Pipeline verifica código e segredos antes do merge/deploy. | P1 |
| CI/CD | Tokens, workflows, ambientes e artefatos protegidos. | P1 |
| Docker/servidor | Imagens confiáveis, usuário não-root, SSH e permissões revisados. | P2 |
| Versões fixadas e inventário | Lockfile versionado; dependências e imagens base com versão fixa; lista de componentes (SBOM) gerada no build. | P1 |
| Origem do que é publicado | O artefato em produção vem do pipeline, a partir de um commit revisado; ninguém publica da própria máquina. | P1 |
| Pacotes novos | Dependência nova passa por revisão de origem, manutenção e permissões de instalação antes de entrar. | P1 |
| Configuração como código | Infraestrutura e configuração de serviços versionadas e verificadas; divergência entre o repositório e o ambiente é detectada. | P2 |

## J. Operação, monitoramento e recuperação

| Controle | Como verificar / evidência | Prioridade |
| --- | --- | --- |
| Logs de segurança | Login, falha de autorização, alteração de papel e ação administrativa registrados. | P1 |
| Alertas | Alertas acionáveis para anomalias, abuso e falhas repetidas. | P1 |
| Integridade dos logs | Acesso restrito, retenção e proteção contra alteração indevida. | P2 |
| Plano de incidentes | Contenção, revogação, investigação, correção, restauração e comunicação documentados. | P1 |
| Backup/restore | Restauração testada; RPO/RTO compatíveis com a operação. | P1 |
| Atualizações | Runtime, framework, sistema, containers e dependências mantidos. | P1 |
| Revisão periódica | Segurança reavaliada a cada grande mudança e em cadência definida. | P2 |
| Exercício de incidente | Simulação do runbook feita ao menos uma vez, com tempo de detecção e de contenção medidos. | P2 |
| Saúde e dependências | Verificação de saúde cobre banco, fila e integrações; alerta quando a fila cresce ou a fila de mortas recebe mensagem. | P1 |

## K. AI Security (quando houver IA)

| Controle | Como verificar / evidência | Prioridade |
| --- | --- | --- |
| Prompt Injection | Entradas e conteúdo recuperado tratados como não confiáveis. | P1 |
| Data leakage | Testes mostram isolamento de dados e contexto entre usuários/tenants. | P0 |
| Tool authorization | Backend autoriza cada ferramenta; modelo não decide a própria permissão. | P0 |
| Excessive agency | Ferramentas, escopos, limites e confirmações proporcionais ao risco. | P1 |
| Output handling | Saída do modelo validada antes de renderização ou execução. | P1 |
| RAG access control | ACL e filtro de tenant aplicados no servidor na recuperação. | P0 |
| Cost/DoS | Rate limit, quotas, timeout e teto de custo por usuário/tenant. | P1 |
| Evals de segurança | Dataset de regressão para injection, leakage, abuso de ferramentas e saída indevida. | P1 |
| Vazamento de prompt de sistema | O prompt não contém segredo, credencial nem regra cuja exposição cause dano; controle de acesso não depende de o prompt ficar oculto. | P1 |
| Vetores e embeddings | Índice vetorial isolado por tenant com filtro no servidor; conteúdo indexado tem origem conhecida; exclusão de documento remove também os vetores. | P0 |
| Memória e base de conhecimento | Instrução vinda de conversa, e-mail ou documento não é promovida a memória nem a base sem validação de origem; fatos persistidos têm proveniência e validade. | P1 |
| Cadeia de suprimentos de modelos | Modelo, versão e provedor fixados e registrados; conectores, servidores MCP e plugins aprovados, com versão fixa e revisão a cada mudança de definição. | P1 |
| Resposta sem suporte | Em fluxo factual, a resposta só usa fatos de fonte autorizada; verificação antes do envio; teste com pergunta sem resposta na base. | P1 |
| Agentes com ação | Ação proposta pelo modelo é autorizada e executada por código, com aprovação vinculada aos parâmetros exatos quando o risco exigir. | P0 |
| Menor agência | O agente tem só as ferramentas, os dados e a autonomia que a tarefa exige; caminho conhecido é workflow em código, não decisão do modelo. | P1 |
| Identidade e credencial do agente | O agente age com identidade própria e escopo mínimo, não com a credencial do usuário nem de administrador; credencial de curta duração; subagente recebe só o escopo delegado. | P0 |
| Execução de código gerado | Código ou comando gerado pelo modelo não é executado; quando a função exigir, roda em sandbox sem root, sem rede livre e sem acesso a segredos, com validação entre gerar e executar. | P0 |
| Comunicação entre agentes | Mensagem de outro agente é entrada não confiável: autenticada, validada por contrato e sem poder de alterar objetivo ou permissão de quem recebe. | P1 |
| Falha em cascata | Teto de passos, de chamadas e de custo por tarefa; disjuntor entre agentes e ferramentas; erro de um passo não é repassado como fato ao seguinte. | P1 |
| Confiança humana na saída do agente | Pedido de aprovação mostra a ação exata, os parâmetros e a evidência, não um resumo persuasivo; ação sensível exige confirmação explícita. | P1 |
| Agente fora do objetivo | Trace de cada passo e de cada chamada de ferramenta; alerta para sequência fora do padrão; forma de desligar o agente e passar tudo para humano, testada. | P1 |
| Objetivo e política versionados | Prompt de sistema, lista de ferramentas e política do agente mudam só por revisão, com versão e registro, como código. | P1 |

## M. Arquitetura assíncrona, filas e jobs

Obrigatória quando a plataforma tem fila, worker, agendador ou webhook processado depois.

| Controle | Como verificar / evidência | Prioridade |
| --- | --- | --- |
| Contexto de autorização no job | Toda mensagem leva tenant e usuário de origem; o worker reaplica a autorização e recusa mensagem sem contexto. | P0 |
| Worker sem privilégio total | Worker não usa credencial que ignora RLS, salvo necessidade documentada e com filtro explícito de tenant no código. | P0 |
| Quem pode publicar | Só componentes autorizados publicam na fila; a fila não é alcançável da internet. | P0 |
| Mensagem como entrada não confiável | O worker valida schema e limites da mensagem antes de processar. | P1 |
| Idempotência | Mensagem duplicada não duplica efeito; teste envia a mesma mensagem duas vezes. | P0 |
| Retry e fila de mortas | Tentativas limitadas, com espera crescente; mensagem que falha vai para a fila de mortas e gera alerta. | P1 |
| Dados na fila e na fila de mortas | Mensagens não carregam segredo; dado pessoal é minimizado (id em vez do registro); retenção definida. | P1 |
| Outbox | Evento gravado na mesma transação do dado; teste derruba o processo entre a gravação e a publicação e confirma que nada se perde. | P1 |
| Replay e ordem | Mensagem antiga reenviada não reverte estado mais novo; controle por versão ou carimbo. | P1 |
| Agendador | Tarefas agendadas rodam com identidade própria e escopo mínimo; execução e falha registradas. | P1 |
| Limites por tenant | Um cliente não ocupa todos os workers; cota ou fila por tenant quando o risco existir. | P2 |
| Painel da fila | Interface de administração da fila protegida por autenticação e fora da internet pública. | P0 |

## N. Comunicação entre serviços e módulos

Obrigatória com mais de um serviço, ou em monolito modular com módulo que trata dado sensível.

| Controle | Como verificar / evidência | Prioridade |
| --- | --- | --- |
| Autenticação entre serviços | Cada chamada interna é autenticada (token de serviço de curta duração ou TLS mútuo); estar na mesma rede não autentica. | P0 |
| Autorização entre serviços | O serviço chamado verifica o que o chamador pode pedir; o usuário e o tenant de origem viajam na chamada e são revalidados. | P0 |
| Uma credencial por serviço | Segredos não são compartilhados entre serviços; o vazamento de um não abre os outros. | P1 |
| Banco por serviço ou schema por módulo | Um serviço ou módulo não lê tabela de outro; grants impedem, não só a convenção. | P1 |
| Rede interna | Serviços internos não têm porta pública; só o gateway ou a borda recebe tráfego externo. | P0 |
| Gateway | Autenticação, limite de taxa e validação básica na entrada, sem substituir a autorização de cada serviço. | P1 |
| Timeout e disjuntor | Toda chamada interna tem timeout; falha de um serviço não esgota os recursos dos outros. | P1 |
| Rastreamento | Id de correlação atravessa todos os serviços, para reconstruir uma operação em incidente. | P1 |
| Fronteira de módulo | Teste automatizado quebra o build quando um módulo acessa o interior de outro. | P2 |

## O. Cache, pool de conexões e tratamento de dados

| Controle | Como verificar / evidência | Prioridade |
| --- | --- | --- |
| Chave de cache por tenant | Teste mostra que um tenant não recebe resposta cacheada de outro. | P0 |
| Cache na borda | Resposta autenticada não é cacheada como pública; cabeçalhos de cache revisados nas rotas privadas. | P0 |
| Autorização em cache | Permissão em cache tem prazo curto e é invalidada na revogação. | P1 |
| Dado sensível em cache | Segredo e dado de alto impacto fora de cache compartilhado; o cache exige autenticação e não é alcançável da internet. | P1 |
| Importação de arquivo | Tipo, tamanho e estrutura validados; conteúdo tratado como não confiável (fórmulas de planilha, caminhos, scripts); processamento fora da requisição. | P1 |
| Exportação | Exportação exige permissão própria, tem limite de volume, é registrada e entregue por link assinado com expiração. | P0 |
| Paginação e limite | Toda listagem tem limite máximo no servidor; não é possível pedir a base inteira em uma chamada. | P1 |
| Integração em lote | Carga registra origem, volume e resultado; reconciliação detecta divergência. | P2 |
| Dados de produção fora de produção | Cópias para teste ou análise são anonimizadas e têm dono e prazo de descarte. | P1 |

## P. LGPD e direitos do titular

Obrigatória quando a plataforma trata dado pessoal. Esta seção organiza controles técnicos; não substitui orientação jurídica.

| Controle | Como verificar / evidência | Prioridade |
| --- | --- | --- |
| Papéis definidos | Para cada plataforma, registrado quem é controlador e quem é operador, e o que o contrato com o cliente diz sobre o tratamento. | P1 |
| Base legal e finalidade | Cada conjunto de dados pessoais tem finalidade e base legal registradas; dado sensível e de criança ou adolescente identificados à parte. | P1 |
| Registro das operações de tratamento | Inventário de quais dados, para quê, onde ficam, com quem são compartilhados e por quanto tempo. | P1 |
| Direitos do titular | Existe caminho para atender pedido de acesso, correção, eliminação e portabilidade, com prazo e responsável; teste de eliminação confirma remoção em banco, storage, cache, índice de busca e vetores. | P1 |
| Encarregado | Contato do encarregado definido e publicado quando exigido. | P2 |
| Relatório de impacto | Tratamento de alto risco tem relatório de impacto elaborado antes da publicação. | P2 |
| Operadores e suboperadores | Lista de fornecedores que recebem dado pessoal (hospedagem, banco, e-mail, modelos de IA), com contrato e país de tratamento. | P1 |
| Transferência internacional | Dado pessoal enviado a serviço fora do país está mapeado e tem o mecanismo de transferência definido. | P1 |
| Comunicação de incidente | O runbook prevê a avaliação de risco aos titulares e a comunicação à ANPD em três dias úteis a partir do conhecimento de que o incidente afetou dados pessoais (art. 6º), com complemento em até vinte dias úteis (art. 6º, § 3º); comunicação aos titulares no mesmo prazo (art. 9º); prazos em dobro para agente de pequeno porte. | P0 |
| Registro de incidentes | Todo incidente com dado pessoal fica registrado, comunicado ou não, por no mínimo cinco anos (art. 10). | P1 |
| Dado pessoal em IA | Dado pessoal enviado a modelo ou a ferramenta de terceiros é minimizado, e o fornecedor consta na lista de operadores. | P1 |

Os prazos de incidente vêm da Resolução CD/ANPD nº 15/2024. O texto no DOU bloqueia acesso automatizado; os artigos foram confirmados em apresentação oficial do gov.br (Centro de Excelência em Privacidade e Segurança, 30/4/2025) e em resumos de seis escritórios coerentes entre si. Link oficial: [Resolução CD/ANPD nº 15/2024 no DOU](https://www.in.gov.br/en/web/dou/-/resolucao-cd/anpd-n-15-de-24-de-abril-de-2024-556243024).

## Q. Acesso administrativo, suporte e operação privilegiada

| Controle | Como verificar / evidência | Prioridade |
| --- | --- | --- |
| Painel administrativo | Rota separada, com MFA obrigatório e, quando possível, restrição de origem. | P0 |
| Acesso de suporte a dados de cliente | Entrar na conta ou ver dados de um cliente exige motivo registrado, tem prazo e fica visível em log de auditoria. | P0 |
| Operação direta no banco de produção | Acesso nominal, não compartilhado; consulta e alteração manuais registradas; alteração por script revisado. | P1 |
| Acesso de emergência | Credencial de emergência guardada à parte, uso com alerta e rotação depois de usada. | P1 |
| Saída de pessoa do time | Lista de acessos a revogar por pessoa (provedores, repositório, banco, painéis, cofre) e evidência da revogação. | P1 |
| Contas de serviço | Cada conta de serviço tem dono, finalidade e escopo; contas sem uso são removidas. | P2 |
| Mudança de permissão | Concessão e elevação de papel registradas com quem concedeu; revisão periódica dos administradores. | P1 |

## L. Resultado da avaliação

Ao final, registrar:

- Plataforma e ambiente avaliados.
- Versão/commit/deploy testado.
- Data da avaliação e pessoa responsável.
- Controles conformes, não conformes e não aplicáveis.
- Achados ordenados por risco e impacto.
- Evidência sanitizada (sem tokens, cookies, senhas, chaves ou PII real).
- Correção, responsável, prazo e resultado do reteste.
- Riscos residuais aceitos e quem aprovou a exceção.

**Regra de aprovação:** P0 não conforme bloqueia release. P1 exige correção antes do release ou mitigação documentada e aceita. P2/P3 entram no plano de melhoria com responsável e prazo.

## Referências

- [OWASP Top 10:2025](https://top10.owasp.org/2025/pt-BR/0x00_2025-Introduction/)
- [OWASP API Security Top 10:2023](https://api-security.owasp.org/editions/2023/en/0x11-t10/)
- [OWASP ASVS](https://owasp.org/www-project-application-security-verification-standard/)
- [OWASP Web Security Testing Guide](https://wstg.owasp.org/)
- [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/)
- [Supabase API Keys](https://supabase.com/docs/guides/getting-started/api-keys)
- [Supabase Vault](https://supabase.com/docs/guides/database/vault)
- [OWASP Secrets Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html) — cofres, rotação, segredos dinâmicos, menor privilégio por segredo
- [Supabase — Connect to your database](https://supabase.com/docs/guides/database/connecting-to-postgres) — modos de pool e prepared statements
- [OWASP Top 10 for LLM Applications](https://genai.owasp.org/llm-top-10/)
- OWASP Top 10 for Agentic Applications 2026 (OWASP GenAI Security Project, 9/12/2025) — [página oficial](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/); PDF bloqueado a acesso automatizado
- [OWASP GenAI LLM Top 10 2026](https://genai.owasp.org/resource/owasp-genai-llm-top-10-2026/) — página oficial, edição de 3/8/2026
- [OWASP MCP Top 10](https://owasp.org/projects/mcp-top-10) — página oficial, beta
- [OWASP API Security Top 10 2023](https://owasp.org/API-Security/editions/2023/fr/0x00-toc/) — edição oficial
- [Azure Architecture Center — Cloud design patterns](https://learn.microsoft.com/en-us/azure/architecture/patterns/) — filas, retry, disjuntor, saga
- Resolução CD/ANPD nº 15/2024 — Regulamento de Comunicação de Incidente de Segurança (consultada por resumos; ver nota na seção P)
- Lei nº 13.709/2018 (LGPD)

Revisão de 12/11/2026: acrescentadas as seções M, N, O, P e Q, o mapa de cobertura e linhas novas em A, E, F, I, J e K. Os controles das seções M, N, O e Q são critério de desenho derivado das referências acima, não transcrição de uma norma.
