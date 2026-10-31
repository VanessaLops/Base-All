# MCP Server — Model Context Protocol

Expõe capacidades do sistema (dados e ações) para agentes de IA (Claude, IDEs, etc.).

## Os 3 primitivos MCP
| Primitivo     | O que é                                  | Exemplo freelancer                    |
|---------------|------------------------------------------|---------------------------------------|
| **Tools**     | Ações que o agente pode executar         | `criar_orcamento`, `listar_clientes`  |
| **Resources** | Dados somente leitura (contexto)         | `projeto://{id}`, `docs://contrato`   |
| **Prompts**   | Templates de prompt reutilizáveis        | `gerar_proposta`, `resumo_semanal`    |

## Estrutura
```
src/
├── server/
│   ├── transports/
│   │   ├── stdio/        Uso local (Claude Desktop / Claude Code)
│   │   └── http/         Uso remoto (Streamable HTTP)
├── registry/             Registro central de tools/resources/prompts
├── tools/
│   └── _template/        definição (nome, descrição) + schema de entrada + handler
├── resources/
│   └── _template/        URI + leitor do recurso
├── prompts/
│   └── _template/        argumentos + template
├── services/             Regras de orquestração (usadas pelos handlers)
├── adapters/
│   ├── backend-api/      Cliente da API do backend (fonte principal)
│   ├── database/         Acesso direto (apenas leitura, se necessário)
│   └── external/         APIs de terceiros
├── shared/
│   ├── errors/           Erros padronizados para o protocolo
│   ├── logger/           Log em stderr (stdout é reservado ao protocolo stdio)
│   ├── validation/       Validação de schemas de entrada
│   └── security/         Auth, escopos, rate-limit, confirmação de ações destrutivas
└── config/               Env, nome/versão do servidor, flags de tools habilitadas
tests/                    unit / integration / mocks
```

## Fluxo de uma chamada
```
Agente IA ─▶ transport ─▶ registry ─▶ tool handler ─▶ validation ─▶ service ─▶ adapter ─▶ Backend API
```

## Design Patterns aplicados
| Pattern            | Onde                                           |
|--------------------|------------------------------------------------|
| Registry           | `registry/` — auto-registro de cada tool       |
| Command            | Cada tool = um comando isolado                 |
| Ports & Adapters   | `services/` dependem de interfaces; `adapters/` implementam |
| Strategy           | `transports/` (stdio ou http intercambiáveis)  |
| Facade             | `services/` simplificam várias chamadas ao backend |
| Decorator          | `shared/security` envolvendo handlers (auth, log, rate-limit) |

## Regras
- Tools com nomes `snake_case` e descrições claras — o agente decide com base nelas.
- O MCP **não** duplica regra de negócio: chama o backend.
- Ações destrutivas devem ser explícitas na descrição e exigir confirmação.
