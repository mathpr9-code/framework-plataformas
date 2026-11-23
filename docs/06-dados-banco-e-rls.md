# 4. Fase 2 — Modelagem de dados e privacidade

- [ ] Modelar entidades, relações, tenant\_id, ownership e papéis.
- [ ] Definir quem pode ler, criar, alterar, excluir e exportar cada entidade.
- [ ] Classificar dados por sensibilidade e finalidade.
- [ ] Aplicar minimização: evitar campos que não tenham finalidade clara.
- [ ] Definir retenção, exclusão, anonimização/pseudonimização e atendimento aos direitos aplicáveis da LGPD.
- [ ] Definir campos que podem aparecer no frontend, em relatórios, logs e respostas de API.
- [ ] Planejar índices, constraints, integridade referencial e transações.
- [ ] Definir backup, restauração e retenção.

**Regra:** anonimizar não é apenas esconder nome na interface. Pseudonimização ainda pode permitir reidentificação e continua exigindo proteção. Usar dados fictícios ou devidamente sanitizados em desenvolvimento e testes.

# 5. Fase 3 — Banco, roles e RLS

Para Supabase/PostgreSQL:

- [ ] Habilitar RLS em toda tabela acessível por APIs expostas.
- [ ] Criar policies específicas para SELECT, INSERT, UPDATE e DELETE.
- [ ] Usar USING e WITH CHECK corretamente.
- [ ] Aplicar isolamento por usuário e tenant no banco, não apenas com filtros do frontend.
- [ ] Revisar views, RPC/functions, triggers, Storage e Edge Functions.
- [ ] Separar roles de aplicação e privilégios administrativos.
- [ ] Revisar GRANT/REVOKE e impedir privilégios excessivos.
- [ ] Testar com usuário anônimo, usuário autenticado, usuário de outra empresa e administrador.
- [ ] Verificar consultas diretas pela API, e não apenas pela interface.
- [ ] Validar backups e teste de restauração.

**Chaves Supabase:** publishable/anon é destinada a ambientes cliente e pode ser visível; sua segurança depende de RLS e grants. Secret/service\_role é privilegiada, pode ignorar RLS e deve ficar exclusivamente em ambiente confiável no servidor. Nunca colocar essa chave no bundle do frontend.

**Gate:** nenhuma tabela com dados privados entra em produção sem RLS/policies revisadas e testes de isolamento.
