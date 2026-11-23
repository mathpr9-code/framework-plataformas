# Como funciona

O framework leva do problema do cliente a uma plataforma publicada, em cinco movimentos:

1. **Descobrir** o que a plataforma faz, para quem, com quais dados e o que não pode dar errado (Fase 0).
2. **Desenhar** antes de construir: estilo de arquitetura, comunicação, filas, dados, linguagem e onde roda, cada escolha com por quê, para quê e o que a direciona (Fase 1 e seções 3.1 a 3.11).
3. **Construir por fases**, com segurança em cada uma, não no fim (Fases 2 a 8).
4. **Provar** com testes e com a Matriz de Segurança; todo erro encontrado vira teste (Fase 9).
5. **Operar** com observabilidade, backup testado e plano de incidente (Fase 10).

Nenhuma peça de arquitetura é regra. Monolito, monolito modular, microsserviços, fila ou chamada direta são escolhas por plataforma, registradas em ADR, e a mais simples que atende vence.

# 1. Princípios obrigatórios

1. **Secure by design:** levantar riscos e requisitos de segurança antes de implementar.
2. **Menor privilégio:** cada usuário, serviço, função, chave e integração recebe apenas o acesso necessário.
3. **Deny by default:** acesso negado até existir uma regra explícita que permita.
4. **Server-side enforcement:** frontend melhora a experiência; backend e banco aplicam as regras de segurança.
5. **Defense in depth:** autenticação, autorização, banco, infraestrutura e monitoramento são camadas complementares.
6. **Minimização de dados:** coletar, armazenar, transmitir e retornar apenas o necessário.
7. **Segredos fora do código e do navegador:** nenhum segredo privilegiado em frontend, Git, logs ou respostas.
8. **Evidência antes de declarar seguro:** cada controle precisa de teste ou evidência verificável.
9. **A arquitetura mais simples que atende:** complexidade só entra com motivo nomeado e registrado em ADR.
10. **Falhar fechado:** em erro, timeout ou estado inesperado, o sistema nega e registra; nunca libera por padrão.
11. **Todo erro vira teste:** falha encontrada em produção, revisão ou auditoria entra na suíte e roda antes de toda publicação.
