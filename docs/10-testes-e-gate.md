# 11. Fase 9 — Testes de segurança e revisão pré-release

- [ ] Executar a [Matriz de Segurança](seguranca/matriz-de-seguranca.md).
- [ ] Testar papéis distintos e isolamento entre tenants.
- [ ] Testar endpoints diretamente, sem depender da interface.
- [ ] Revisar login, recuperação, sessão, logout, cookies e “lembrar-me”.
- [ ] Testar validação, mass assignment, exposição excessiva e erros.
- [ ] Verificar RLS, Storage, funções e webhooks.
- [ ] Conferir segredos no bundle, Git, logs e artefatos.
- [ ] Revisar o acesso dos agentes de programação (Codex, Claude Code ou equivalentes) ao workspace, .env, terminal, rede, Git, Docker e SSH.
- [ ] Executar teste com .env fictício para comprovar bloqueio de leitura e exfiltração pelo agente.
- [ ] Verificar secret scanning nos diffs, commits, PRs e artefatos gerados por agentes.
- [ ] Verificar TLS, CORS, headers, WAF e rate limits.
- [ ] Executar scans SAST/SCA/DAST compatíveis com o projeto.
- [ ] Rodar os testes ofensivos internos (11.2): Gobuster, TestSprite e Burp Suite ou ZAP, contra staging.
- [ ] Fazer backup e validar restauração antes de mudanças críticas.
- [ ] Registrar achados com evidência sanitizada, severidade, correção e reteste.

**Gate de release:** falhas críticas/altas sem mitigação aprovada bloqueiam publicação. Exceções precisam de risco documentado, responsável e prazo de correção.

# 11.1 Suíte de erros e testes de arquitetura

**Suíte de erros.** Toda falha encontrada (pelo cliente, em revisão, em auditoria ou em produção) vira um teste automatizado antes de ser corrigida. A suíte inteira roda antes de toda publicação e uma reprovação bloqueia o deploy. Falha de segurança entra na suíte como qualquer outra: um vazamento entre tenants corrigido sem teste volta na próxima mudança de policy.

**Testes que a arquitetura escolhida exige:**

- [ ] Monolito modular: teste de dependência entre módulos quebra o build quando a fronteira é furada.
- [ ] Fila: mensagem duplicada não duplica efeito; mensagem defeituosa vai para a fila de mortas; worker derrubado no meio não perde o job.
- [ ] Assíncrono: job roda com o tenant e o usuário de origem e é recusado sem eles.
- [ ] Pool em modo transação: duas requisições de tenants diferentes em sequência não compartilham contexto.
- [ ] Cache: usuário de um tenant não recebe resposta cacheada de outro.
- [ ] Importação: arquivo inválido é rejeitado inteiro ou por linha, conforme definido; reenvio não duplica.
- [ ] Dependência externa fora do ar: timeout respeitado, disjuntor abre, o resto da plataforma responde.
- [ ] Carga: o pico esperado não esgota conexões de banco.
- [ ] Restauração: backup restaurado em ambiente de teste.

# 11.2 Testes ofensivos internos

Antes de alguém de fora testar a plataforma, ela é atacada por dentro, com as mesmas ferramentas que um atacante usaria. O objetivo é achar primeiro o que está aberto: rota esquecida, arquivo exposto, subdomínio de teste, sessão que não expira, ID trocado que devolve dado de outro cliente.

**Regras que valem para todas as ferramentas:**

- Só contra o que é seu ou o que o cliente autorizou por escrito, com escopo (domínios, ambientes, janela de horário). Testar sistema de terceiro sem autorização é crime, não teste.
- Primeiro em staging, com dado fictício. Em produção, só o reconhecimento passivo e com aviso a quem opera.
- Taxa e concorrência limitadas: o teste não pode derrubar o ambiente nem estourar a cota de serviço pago.
- Credenciais de teste próprias, nunca as de um usuário real; contas de teste apagadas no fim.
- Todo achado vira caso na suíte de erros (11.1) antes da correção, com evidência sanitizada e reteste.

| Ferramenta | O que é | Para que serve aqui | Quando roda | Cuidado |
| --- | --- | --- | --- | --- |
| [TestSprite](https://www.testsprite.com/) | Agente de testes com IA, ligado ao editor por MCP ou pela CLI open source. Lê o código e o PRD, gera e roda testes de UI, API e ponta a ponta e devolve a falha com a causa provável | Cobrir os fluxos funcionais e de API que o agente de código escreveu, incluindo papéis diferentes e entradas inválidas | A cada entrega, antes do gate | Os testes rodam na nuvem do fornecedor: só contra staging, com dado fictício, e o fornecedor entra na lista de operadores se houver dado pessoal (Matriz P) |
| [Gobuster](https://github.com/OJ/gobuster) | Ferramenta open source (Apache 2.0) em Go que testa por força bruta, com listas de palavras, diretórios e arquivos, subdomínios (DNS), virtual hosts e buckets | Achar o que está publicado sem querer: painel admin, `.env`, backup, `.git`, rota de teste, ambiente de preview, bucket aberto | Antes do primeiro deploy público e a cada mudança de domínio, proxy ou storage | Listas do SecLists; limite de threads; resultado 200 em caminho sensível é achado P0 (Matriz B, I, Q) |
| [Burp Suite](https://portswigger.net/burp) | Proxy de interceptação da PortSwigger. A Community é gratuita (código fechado) e traz Proxy, Repeater, Decoder, Sequencer e Comparer; a Professional é paga e acrescenta o scanner automático | Teste manual do que a automação não vê: trocar IDs (BOLA/IDOR), repetir requisição com outro token, mexer em campos privilegiados, testar sessão, CSRF e injeção | Gate da Fase 9 e a cada mudança de autenticação, autorização ou API | Rodar com usuários de teste de dois tenants; nada de sessão real |
| [ZAP](https://www.zaproxy.org/) | Alternativa open source ao Burp, com scanner ativo e passivo | Varredura DAST automatizada no CI e o mesmo teste manual sem licença paga | No pipeline, contra staging | Scanner ativo só em staging |

Ordem de uso: Gobuster mapeia o que está exposto; TestSprite cobre os fluxos funcionais; Burp Suite ou ZAP testam autorização e sessão em cima do mapa. O resultado entra na Matriz (seção R) com data, versão e evidência.

# 14. Gate final

Quem assina: o responsável como dono e o revisor de segurança (a mesma pessoa em sessão separada, enquanto não houver segunda), com data, versão e lista de controles P0 conferidos, como define a seção "Quem revisa" da Matriz. Sem essa assinatura a plataforma não entra em produção.<br />— checklist de publicação

- [ ] Arquitetura, dados e ameaças documentados.
- [ ] Autenticação e autorização server-side testadas.
- [ ] RLS e isolamento multi-tenant comprovados.
- [ ] Cookies, sessões, recuperação e “lembrar-me” revisados.
- [ ] Segredos fora de frontend, Git, logs e artefatos.
- [ ] Vault/secret manager, rotação e acesso mínimo definidos.
- [ ] Criptografia e classificação de dados revisadas.
- [ ] APIs com validação, rate limit, CORS e respostas minimizadas.
- [ ] Headers, HTTPS, WAF e proteção contra bots revisados.
- [ ] Dependências e pipeline verificados.
- [ ] Logs, alertas, backup, restore e runbook definidos.
- [ ] Testes de segurança executados e evidências sanitizadas anexadas.
- [ ] Acesso de agentes de desenvolvimento ao workspace e a arquivos secretos foi revisado e testado.
- [ ] Riscos residuais documentados e aceitos explicitamente.
