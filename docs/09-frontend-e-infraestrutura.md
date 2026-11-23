# 9. Fase 7 — Frontend, navegador e headers

- [ ] Tratar todo dado do navegador como controlável pelo usuário.
- [ ] Não inserir segredo privilegiado no bundle JavaScript.
- [ ] Verificar XSS e codificação contextual; sanitizar HTML quando realmente necessário.
- [ ] Configurar Content-Security-Policy de acordo com os recursos usados.
- [ ] Configurar HSTS, X-Content-Type-Options, Referrer-Policy e Permissions-Policy.
- [ ] Proteger framing com CSP frame-ancestors e/ou X-Frame-Options quando apropriado.
- [ ] Revisar cookies, storage, source maps e mensagens de erro.
- [ ] Não colocar token ou dado sensível em query string/URL.
- [ ] Garantir HTTPS e redirect para HTTPS.
- [ ] Revisar scripts e bibliotecas de terceiros.

# 10. Fase 8 — Infraestrutura, Cloudflare, hosting e dependências

- [ ] Configurar Cloudflare WAF, rate limiting e bot protection de acordo com rotas e perfil de tráfego.
- [ ] Entender que Cloudflare não substitui autorização no backend nem RLS.
- [ ] Proteger DNS, domínios, certificados, subdomínios e regras de proxy.
- [ ] Separar variáveis e permissões de desenvolvimento, preview/staging e produção.
- [ ] Proteger com MFA e menor privilégio o painel e as contas de todo provedor usado: hospedagem, DNS e borda, banco, repositório e servidor.
- [ ] Manter serviços, runtime, imagens e dependências atualizados.
- [ ] Fazer análise de dependências (SCA), análise estática (SAST) e secret scanning no CI.
- [ ] Proteger workflows de CI/CD, tokens, artefatos e permissões de deploy.
- [ ] Em Docker, usar imagens confiáveis, usuário não-root, portas mínimas e secrets fora da imagem.
- [ ] Restringir SSH e portas administrativas; preferir chave, MFA e acesso controlado.

# 10.1 — Controles para agentes de desenvolvimento e acesso ao workspace

Codex, Claude Code e outros agentes de programação podem receber acesso a arquivos, terminal, Git, rede, Docker, SSH e ferramentas locais conforme as permissões concedidas e a configuração de sandbox. Se um arquivo .env estiver acessível ao agente, ele poderá ser lido mesmo que esteja no .gitignore.

**Princípio obrigatório:** .gitignore evita versionamento acidental; não é controle de confidencialidade e não impede leitura por agentes, extensões, processos ou ferramentas com acesso ao filesystem.

## Antes de abrir um projeto a um agente

- [ ] Inventariar capacidades habilitadas: leitura/escrita de arquivos, terminal, rede, Git, Docker, SSH, cloud, navegador e conectores.
- [ ] Revisar o modo de sandbox, escopo do workspace, regras allow/deny e exigência de aprovação do agente utilizado. Não presumir que a configuração padrão bloqueia secrets.
- [ ] Identificar arquivos e diretórios sensíveis: .env, .env.local, .env.production, .npmrc, .pypirc, credentials, .ssh, chaves privadas, dumps, backups, arquivos de configuração cloud e arquivos de estado Terraform.
- [ ] Manter credenciais reais fora do workspace que o agente pode ler sempre que possível. Preferir Secret Manager, credenciais do runtime ou mecanismo seguro do provedor.
- [ ] Se um arquivo secreto precisar existir localmente para executar a aplicação, restringir tecnicamente sua leitura pelo agente; não confiar apenas em instruções no prompt.
- [ ] Não disponibilizar .env de produção a agentes de desenvolvimento. Usar credenciais descartáveis e dados fictícios em dev/staging.
- [ ] Usar contas de teste com escopo limitado e sem acesso a dados reais.
- [ ] Não incluir valores secretos em prompts, [CLAUDE.md](http://CLAUDE.md), [AGENTS.md](http://AGENTS.md) ou equivalentes. Esses arquivos podem armazenar políticas e nomes de variáveis, nunca valores.
- [ ] Instruir o agente a não abrir, imprimir, resumir, copiar, alterar ou transmitir arquivos secretos, mantendo também bloqueio técnico no ambiente.
- [ ] Desativar ou limitar acesso à rede, shell genérico, Docker socket, SSH e diretórios fora do repositório quando não forem necessários.
- [ ] Exigir revisão humana de comandos destrutivos, mudanças de infraestrutura, deploys, migrações e operações em produção.

## Prevenir vazamento pela saída do agente

- [ ] Tratar chat, histórico, logs, traces, saída do terminal, diffs, screenshots, commits, PRs, artefatos e chamadas externas como possíveis canais de saída.
- [ ] Não permitir que ferramentas retornem valores de variáveis secretas ao modelo; devolver somente presença/ausência, nome da variável, status e resultado sanitizado.
- [ ] Mascarar credenciais em logs e erros antes de serem armazenados ou enviados ao agente.
- [ ] Executar secret scanning nos arquivos alterados, diffs, commits e artefatos antes de push, PR e deploy.
- [ ] Revisar workflows, scripts de provisionamento, Dockerfiles, CI/CD e configurações para impedir inclusão de secrets.
- [ ] Impedir uploads, transferências e requisições de rede para destinos não aprovados quando contiverem dados sensíveis.
- [ ] Se um segredo aparecer no chat, terminal, diff, log ou artefato, interromper o fluxo e iniciar rotação/revogação. Apagar a mensagem ou arquivo não torna a credencial segura novamente.

## Teste obrigatório antes de usar em projetos sensíveis

Executar em repositório descartável, com arquivo .env contendo apenas valores fictícios:

1. Configurar o agente com o menor escopo de filesystem e ferramentas necessário.
2. Confirmar que o agente não consegue ler o arquivo secreto nem diretórios de credenciais fora do workspace.
3. Confirmar que comandos, logs, diffs e respostas não revelam os valores de teste.
4. Confirmar que o agente não consegue enviá-los a destinos externos não autorizados.
5. Registrar configuração e evidências; repetir após mudanças no agente, extensão, sandbox ou permissões.

**Limite importante:** se o agente opera com acesso irrestrito ao mesmo usuário e filesystem, uma regra escrita no prompt não garante que ele não consiga ler o .env. É necessário limitar tecnicamente as permissões ou manter o segredo fora do alcance desse processo.
