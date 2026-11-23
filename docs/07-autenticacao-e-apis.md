# 6. Fase 4 — Autenticação, autorização e sessões

## Autenticação

- [ ] Preferir provedor maduro (por exemplo, Supabase Auth) em vez de implementar armazenamento de senha e emissão de sessão do zero.
- [ ] Implementar login, cadastro, verificação de e-mail, recuperação e redefinição de senha.
- [ ] Definir MFA/passkeys conforme risco e perfil da plataforma.
- [ ] Evitar mensagens que permitam enumeração de contas.
- [ ] Aplicar rate limit em login, cadastro, recuperação e confirmação.
- [ ] Proteger fluxos de troca de e-mail, senha e fator de autenticação.
- [ ] Exigir reautenticação em operações de alto risco quando apropriado.

## Autorização

- [ ] Separar autenticação (quem é) de autorização (o que pode fazer).
- [ ] Validar identidade e permissão no servidor em cada operação protegida.
- [ ] Implementar RBAC/ABAC e permissões por recurso/ação.
- [ ] Verificar ownership e tenant em toda leitura ou alteração por ID.
- [ ] Não confiar em papel, user\_id, tenant\_id ou flags enviados pelo frontend.
- [ ] Aplicar deny by default e registrar mudanças de privilégio.

## Sessão e “lembrar-me”

- [ ] Definir duração idle e duração máxima da sessão.
- [ ] Rotacionar identificador/token após login e elevação de privilégio.
- [ ] Invalidar sessão no logout e permitir revogação de outras sessões.
- [ ] Configurar cookies Secure, HttpOnly e SameSite de acordo com o fluxo.
- [ ] Avaliar proteção CSRF quando autenticação usa cookies.
- [ ] Não guardar senha nem credenciais de sessão em localStorage/sessionStorage sem uma arquitetura explicitamente justificada.
- [ ] “Lembrar-me” deve usar persistência controlada pelo provedor, expiração e revogação — nunca salvar a senha para facilitar o próximo login.

**Gate:** não basta esconder menus. Tentar a mesma operação pela API com um usuário sem permissão deve resultar em bloqueio.

# 7. Fase 5 — Backend e APIs

- [ ] Inventariar endpoints, métodos, papéis exigidos e dados retornados.
- [ ] Autenticar endpoints privados e autorizar cada operação.
- [ ] Validar schema, tipo, tamanho, formato, limites e allowlists.
- [ ] Implementar DTOs/allowlists contra mass assignment: não aceitar propriedades arbitrárias do JSON.
- [ ] Usar queries parametrizadas e evitar SQL dinâmico inseguro.
- [ ] Minimizar e sanitizar/serializar respostas: retornar somente campos necessários.
- [ ] Não expor stack trace, SQL, caminhos internos, segredos ou PII desnecessária.
- [ ] Aplicar rate limit por IP, usuário, tenant, rota e custo.
- [ ] Aplicar quotas, timeouts, limites de payload, paginação e limites de exportação.
- [ ] Implementar idempotência em ações repetíveis/críticas.
- [ ] Proteger webhooks com assinatura, timestamp, segredo e prevenção de replay.
- [ ] Configurar CORS por origem, método e header necessários. CORS não substitui autenticação/autorização.
- [ ] Proteger operações de alto impacto com confirmação e/ou reautenticação.
