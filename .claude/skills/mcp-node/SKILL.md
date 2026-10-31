---
name: mcp-node
description: Implementação de servidores MCP (Model Context Protocol) em Node.js + TypeScript com o SDK oficial (diretório mcp-server/). Use ao criar ou alterar tools, resources, prompts, transports, registry, services ou adapters do servidor MCP.
---

# MCP Server — Node.js (sênior)

**Antes de tudo, aplique `.claude/skills/engineering-standards/SKILL.md`.** Estrutura: `mcp-server/README.md`.

## Stack padrão
Node LTS · TypeScript strict · `@modelcontextprotocol/sdk` (`McpServer`) · Zod (schemas de entrada) · transport stdio (local)
e Streamable HTTP (remoto) · Vitest.

## Escolha do primitivo
| O agente precisa…                    | Primitivo   |
|--------------------------------------|-------------|
| Executar uma ação / consultar com parâmetros | **Tool** |
| Ler um dado identificável por URI (contexto) | **Resource** |
| Um template de instrução reutilizável        | **Prompt** |

## Tools
- Uma tool por pasta em `src/tools/<nome>/`: definição (nome, título, descrição), schema Zod e handler.
- Nome `snake_case`, verbo + objeto: `create_invoice`, `list_clients`, `get_project_status`.
- **Descrição é a interface com o modelo**: diga o que faz, quando usar, o que retorna e efeitos colaterais.
- Cada campo do schema com `.describe()`; tipos e limites precisos (`z.string().uuid()`, `z.number().int().min(1).max(100)`).
- Handler fino: valida → chama `service` → formata resultado. Sem regra de negócio, sem HTTP direto.
- Erros de negócio retornam resultado com `isError: true` e mensagem útil para o modelo corrigir; não derrube o servidor.
- Saída concisa e estruturada; pagine listas (`limit`, `cursor`) — contexto do modelo é caro.
- Ações destrutivas: nome e descrição explícitos, `annotations` (`destructiveHint`, `idempotentHint`, `readOnlyHint`) corretas.
- Prefira poucas tools com propósito claro a muitas tools quase iguais.

## Resources e Prompts
- Resources com URI estável (`project://{projectId}`), somente leitura, `mimeType` correto.
- Prompts com argumentos tipados e descritos; o texto do template fica no arquivo do prompt, não espalhado.

## Registry
`src/registry` registra todas as tools/resources/prompts num único ponto; cada módulo exporta uma função
`register<Nome>(server)`. Habilitar/desabilitar via config, sem `if` espalhado.

## Services e Adapters
- `services/` contém a orquestração e depende de **interfaces**; `adapters/backend-api` implementa chamando a API do backend.
- O MCP **não** duplica regra de negócio nem acessa o banco para escrita: o backend é a fonte da verdade.
- Respostas de APIs externas validadas com Zod antes de chegar ao handler.

## Transport e logs
- **stdio: nunca escreva em `stdout`** (é o canal do protocolo). Logs vão para `stderr` via `shared/logger`.
- HTTP remoto exige autenticação (OAuth/token), validação de origem e rate-limit (`shared/security`, Decorator sobre handlers).
- Configuração (URL do backend, tokens) validada com Zod em `src/config`; segredos só por env.

## Testes
- Unit: handlers com service mockado; verifique schema (entradas inválidas rejeitadas) e formato do resultado.
- Integração: cliente MCP em memória contra o servidor, listando e chamando tools.
- Teste manual com MCP Inspector.
