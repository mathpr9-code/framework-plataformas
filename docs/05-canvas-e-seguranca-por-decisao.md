# 3.10 Canvas da plataforma e ordem de montagem

O desenho é feito antes do código, em um quadro com sete colunas e as faixas de fronteiras de confiança, onde roda, Matriz ativada e segurança por coluna. O template está anexado na Etapa 0, no início deste documento; o modelo são as sete colunas: Usuários, Frontend, Borda, Backend, Assíncrono, Dados, Integrações. Nunca se parte de um quadro em branco.

## O prompt que pede o desenho

```txt
Desenhe a arquitetura desta plataforma no quadro do framework.

O quadro tem sete colunas fixas: 1 Usuários, 2 Frontend, 3 Borda, 4 Backend, 5 Assíncrono,
6 Dados, 7 Integrações. Faixas: Fronteiras de confiança, Onde roda, Matriz ativada, Segurança
por coluna, Regra. Não invente colunas.

Use docs/discovery/ e docs/sources/. Para cada coluna: uma peça do catálogo (estilo 3.1,
fila Q1-Q5, pool e multi-tenant 3.4, deploy D1-D7, cofre V1-V7), o porquê, a hipótese e
o que a confirma, os controles da Matriz que ela ativa. Para cada seta, onde a autorização
acontece. Marque as peças descartadas com o motivo.

Entregue o .excalidraw em docs/architecture/diagrams/, a tabela em docs/architecture/canvas.md
e a lista do que não conseguiu responder. Não preencha lacunas com suposição.
```

| Coluna | O que se decide | O que direciona |
| --- | --- | --- |
| Usuários e papéis | Quem usa, com que permissão, de que tenant | Fase 0 |
| Frontend | Aplicação web, o que roda no navegador | Tipo de interface; tempo real ou não |
| Borda | CDN, WAF, limite de taxa | Exposição pública; perfil de tráfego |
| Backend | Estilo (3.1), módulos, linguagem (3.8) | Complexidade da regra; time |
| Assíncrono | Fila, jobs, agendador (3.3) | Operações longas, picos, integrações instáveis |
| Deploy | Peça de deploy por componente (3.9) | Quem opera; custo fixo ou por uso; exigência do cliente |
| Dados | Banco, modelo multi-tenant, pool, storage, cache (3.4, 3.7) | Número de clientes; isolamento exigido |
| Integrações | APIs externas, webhooks, n8n, IA | Mapa de integrações da Fase 0 |

Faixas: **fronteiras de confiança**, marcando onde entra dado não confiável e onde cada autorização é aplicada; e **onde roda**, com observabilidade e deploy.

Ordem de montagem:

1. Usuários, papéis e tenants.
2. Dados: entidades principais e classificação.
3. Operações críticas e operações longas.
4. Estilo de arquitetura.
5. Para cada operação longa ou instável: síncrono ou assíncrono, e qual fila.
6. Modelo multi-tenant e modo de conexão.
7. Integrações e como cada uma falha.
8. Linguagem por fronteira.
9. Onde roda, e qual cofre guarda os segredos de cada componente (8.1).
10. Fronteiras de confiança e ponto de autorização de cada seta.
11. Seções da Matriz de Segurança que a arquitetura escolhida ativa (3.11).
12. ADR de cada escolha material e plano por fases.

# 3.11 Segurança por decisão de arquitetura

Cada escolha de arquitetura liga controles específicos da Matriz de Segurança.

| Se a arquitetura tem | Seções da Matriz que passam a ser obrigatórias |
| --- | --- |
| Frontend mais BaaS | E (banco e RLS), B (chaves no frontend), D (isolamento) |
| Backend próprio | C, D, F |
| Fila, jobs ou agendador | M (assíncrono) |
| Mais de um serviço, ou monolito modular com módulo sensível | N (comunicação entre serviços e módulos) |
| Pool de conexões, cache, importação ou exportação | O (cache, pool e tratamento de dados) |
| Dado pessoal | H e P (LGPD) |
| Painel administrativo ou acesso de suporte | Q (acesso privilegiado) |
| Qualquer segredo ou API key | B (cofres e chaves), com o inventário da seção 8.1 |
| IA | K |
| VPS, containers, CI/CD | I |
