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
