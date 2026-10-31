# Base-All — Boilerplate estrutural para projetos freelancer

Estrutura base (sem código) para iniciar projetos fullstack com padrões consistentes.

| Pasta         | Responsabilidade                                      | Padrão principal                         |
|---------------|-------------------------------------------------------|------------------------------------------|
| `backend/`    | API em Node.js                                        | Clean Architecture + DDD modular         |
| `frontend/`   | Web em React.js                                       | Feature-based + Atomic Design            |
| `mobile/`     | App em React Native + Expo                            | Estrutura Rocketseat (src/ + aliases)    |
| `mcp-server/` | Servidor MCP em Node.js                               | Ports & Adapters + Registry              |
| `infra/`      | Docker, CI/CD, proxy                                  | —                                        |
| `docs/`       | Arquitetura, ADRs, convenções                         | —                                        |

## Como iniciar um projeto novo
1. Copie a pasta `Base-All` e renomeie para o cliente/projeto.
2. Para cada domínio do negócio, duplique `backend/src/modules/_template` e `frontend/src/features/_template`.
3. Para cada capacidade exposta à IA, duplique `mcp-server/src/tools/_template`.
4. Registre decisões importantes em `docs/adr/` (ex.: `0001-escolha-do-banco.md`).

## Fluxo entre camadas
```
Frontend / Mobile ──HTTP──▶ Backend (controller → use-case → domain → repository)
                                          ▲
MCP Server (tool → service → adapter) ────┘  (consome a API do backend, nunca o domínio direto)
```

## Skills (Claude Code) — `.claude/skills/`
| Skill                   | Aplica-se a                         |
|-------------------------|-------------------------------------|
| `engineering-standards` | Base comum: nomes, tipos, métodos, SOLID, patterns |
| `frontend-react`        | `frontend/`                         |
| `mobile-react-native`   | `mobile/`                           |
| `backend-node`          | `backend/`                          |
| `mcp-node`              | `mcp-server/`                       |
