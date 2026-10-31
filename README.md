# Base-All — Boilerplate estrutural para projetos freelancer

Estrutura base (sem código) para iniciar projetos fullstack com padrões consistentes.

| Pasta         | Responsabilidade                                      | Padrão principal                         |
|---------------|-------------------------------------------------------|------------------------------------------|
| `backend/`    | API, regras de negócio, persistência                  | Clean Architecture + DDD modular         |
| `frontend/`   | Interface, estado de UI, consumo da API               | Feature-based + Atomic Design            |
| `mcp-server/` | Expor capacidades do sistema para agentes de IA (MCP) | Ports & Adapters + Registry              |
| `infra/`      | Docker, CI/CD, proxy                                  | —                                        |
| `docs/`       | Arquitetura, ADRs, convenções                         | —                                        |

## Como iniciar um projeto novo
1. Copie a pasta `Base-All` e renomeie para o cliente/projeto.
2. Para cada domínio do negócio, duplique `backend/src/modules/_template` e `frontend/src/features/_template`.
3. Para cada capacidade exposta à IA, duplique `mcp-server/src/tools/_template`.
4. Registre decisões importantes em `docs/adr/` (ex.: `0001-escolha-do-banco.md`).

## Fluxo entre camadas
```
Frontend (feature) ──HTTP──▶ Backend (controller → use-case → domain → repository)
                                          ▲
MCP Server (tool → service → adapter) ────┘  (consome a API do backend, nunca o domínio direto)
```
