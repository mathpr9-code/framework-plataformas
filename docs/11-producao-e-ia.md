# 12. Fase 10 — Produção, observabilidade e incidentes

- [ ] Monitorar autenticação, falhas de autorização, mudanças de papel, exportações e operações administrativas.
- [ ] Criar alertas para padrões anômalos, falhas repetidas e consumo inesperado.
- [ ] Não registrar tokens, senhas, cookies ou dados pessoais completos.
- [ ] Definir retenção, acesso e integridade dos logs.
- [ ] Manter inventário de serviços, responsáveis e dependências.
- [ ] Ter runbook de incidente: conter, revogar segredos/sessões, preservar evidências, avaliar impacto, corrigir, restaurar e comunicar.
- [ ] Testar recuperação e restauração periodicamente.
- [ ] Reavaliar riscos após mudanças relevantes, novas integrações ou funcionalidades.

# 13. Regras específicas para agentes de IA

Quando a plataforma possuir IA:

- [ ] Tratar prompt, arquivos, páginas e documentos recuperados como entradas não confiáveis.
- [ ] Separar instruções de sistema de conteúdo recuperado.
- [ ] Autorizar cada ferramenta no backend; o modelo não concede a si próprio permissões.
- [ ] Aplicar identidade, tenant, escopo e validação em toda ferramenta.
- [ ] Exigir confirmação humana para ações de alto impacto quando necessário.
- [ ] Limitar ferramentas, volume, frequência e custo.
- [ ] Filtrar RAG por permissões/tenant no servidor.
- [ ] Evitar exposição de dados sensíveis em prompt, contexto, logs e resposta.
- [ ] Validar saída do modelo antes de executar comandos, SQL, HTML ou ações.
- [ ] Quando houver agente que planeja e age: aplicar o OWASP Top 10 para aplicações agênticas (ASI01 a ASI10), mapeado na seção K da Matriz de Segurança e no framework de agentes.
- [ ] Criar testes de prompt injection, vazamento e abuso de ferramentas.
