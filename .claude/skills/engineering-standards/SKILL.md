---
name: engineering-standards
description: Base de engenharia sênior compartilhada por todas as stacks do projeto (React, React Native, Node, MCP). Use sempre que escrever, revisar ou refatorar qualquer código TypeScript deste repositório — define nomenclatura de variáveis, tipos e métodos, uso correto de métodos nativos, SOLID, Clean Code e quando aplicar cada design pattern.
---

# Padrões de Engenharia (base comum)

Atue como engenheiro de software sênior. Todo código é **TypeScript strict**. As skills de stack
(`frontend-react`, `mobile-react-native`, `backend-node`, `mcp-node`) estendem estas regras — nunca as contradizem.

## 1. Princípios
- **SOLID**: uma responsabilidade por arquivo/função; dependa de interfaces, não de implementações.
- **DRY com critério**: extraia na 2ª/3ª repetição real. Duplicação acidental (parecida, mas muda por motivos diferentes) **não** é extraída.
- **KISS / YAGNI**: não crie abstração, pattern ou parâmetro “para o futuro”.
- **Fail fast**: valide entradas na borda (HTTP, formulário, tool) e confie nos tipos dentro do sistema.
- Siga a estrutura de pastas do README da stack; não invente pastas novas.

## 2. Nomenclatura
| Elemento                     | Convenção          | Exemplo                                  |
|------------------------------|--------------------|------------------------------------------|
| Variável / função / método   | camelCase          | `totalPrice`, `calculateDiscount()`      |
| Booleano                     | prefixo is/has/can/should | `isLoading`, `hasPermission`      |
| Constante global imutável    | UPPER_SNAKE_CASE   | `MAX_RETRY_ATTEMPTS`                     |
| Classe / Type / Interface / Enum | PascalCase     | `CreateUserUseCase`, `UserDTO`           |
| Componente React             | PascalCase         | `ProductCard`                            |
| Hook                         | `use` + PascalCase | `useAuth`                                |
| Handler de evento            | `handle` + Ação    | `handleSubmit`; prop: `onSubmit`         |
| Arquivo (exceto componentes) | kebab-case + sufixo de papel | `create-user.use-case.ts`      |

- Funções começam com **verbo** (`get`, `create`, `update`, `remove`, `validate`, `format`, `parse`, `to`).
- Nomes revelam intenção: nada de `data`, `info`, `temp`, `aux`, `x`, `obj`, `item2`.
- Sem abreviações obscuras (`usr`, `qtd`, `btnCtrl`). Coleções no plural (`users`), item no singular (`user`).
- Código, nomes e commits em **inglês**; textos de UI podem ser em português.

## 3. Tipos
- **Proibido `any`**. Use `unknown` + narrowing quando o tipo for realmente desconhecido.
- `type` para unions, composições e props; `interface` para contratos implementáveis (repositórios, providers, ports).
- Tipe explicitamente o **retorno** de funções públicas, use-cases, services e hooks exportados.
- Prefira **union de literais** (`'pending' | 'paid'`) ou `as const` a `enum` numérico.
- Use utilitários em vez de redeclarar: `Pick`, `Omit`, `Partial`, `Readonly`, `ReturnType`, `z.infer`.
- Evite `as` (type assertion) e `!` (non-null). Se precisar, a modelagem está errada ou falta validação.
- Dados de fronteira (API, storage, env) são `unknown` até passarem por validação (zod).

## 4. Variáveis
- `const` por padrão; `let` só com reatribuição real; **nunca `var`**.
- Escopo mínimo: declare perto do uso.
- Sem números/strings mágicos — extraia para constante nomeada.
- Não mute parâmetros nem estado compartilhado; retorne novos objetos (`{ ...user, name }`).

## 5. Métodos — uso correto
| Objetivo                              | Use                     | Não use                          |
|---------------------------------------|-------------------------|----------------------------------|
| Transformar lista                      | `map`                  | `forEach` + `push`               |
| Filtrar                                | `filter`               | `map` retornando `undefined`     |
| Agregar em um valor                    | `reduce` (ou loop claro) | `map` com variável externa     |
| Buscar 1 item                          | `find` / `findIndex`   | `filter(...)[0]`                 |
| Testar existência                      | `some` / `every` / `includes` | `find` só para checar     |
| Efeito colateral por item              | `for...of`             | `map` ignorando o retorno        |
| Async sequencial                       | `for...of` + `await`   | `forEach(async ...)`             |
| Async paralelo                         | `Promise.all` / `allSettled` | `await` dentro de loop sem necessidade |
| Lookup por chave frequente             | `Map` / `Set`          | `find` repetido em array grande  |
| Valor padrão                           | `??`                   | `\|\|` (quando 0/'' são válidos) |
| Acesso opcional                        | `?.`                   | cadeias de `&&`                  |
| Copiar                                 | spread / `structuredClone` | `JSON.parse(JSON.stringify())` |

- `async/await` em vez de `.then()` encadeado. Toda Promise é aguardada ou tratada.
- Comparação sempre com `===` / `!==`.

## 6. Funções
- Pequenas, um nível de abstração, **early return** em vez de `if/else` aninhado.
- Até 3 parâmetros; acima disso, receba um objeto tipado.
- Sem flags booleanas que mudam o comportamento (`save(user, true)`) — crie duas funções.
- Funções puras sempre que possível; efeitos colaterais isolados nas bordas.

## 7. Erros
- Lance erros **tipados** (classe de erro de domínio / `AppError`), nunca `throw 'string'`.
- Nunca engula erro (`catch {}` vazio). Trate, converta ou propague.
- Mensagens para o usuário separadas de detalhes técnicos (log).

## 8. Design patterns — quando usar
| Pattern       | Use quando…                                                     |
|---------------|-----------------------------------------------------------------|
| Repository    | Isolar persistência do domínio                                  |
| Adapter       | Integrar lib/API externa sem vazar o formato dela               |
| Factory       | Montar objetos com dependências (composição)                    |
| Strategy      | Há 2+ algoritmos intercambiáveis (substitui `switch` crescente) |
| Observer/Event| Reagir a algo sem acoplar emissor e receptor                    |
| Decorator     | Adicionar logging/cache/auth sem alterar a função original     |
| Facade        | Simplificar um subsistema complexo                              |
Não aplique pattern sem o problema que ele resolve.

## 9. Comentários e limpeza
- O código explica o **quê**; comentário só para o **porquê** não óbvio.
- Sem código comentado, `console.log` esquecido, imports ou variáveis não usados.
- Sem `TODO` sem contexto.

## 10. Checklist antes de entregar
- [ ] Zero `any`, `as` desnecessário e `!`
- [ ] Nomes seguem a tabela da seção 2
- [ ] Métodos de array/async usados conforme seção 5
- [ ] Nenhuma duplicação real; nenhuma abstração prematura
- [ ] Erros tipados e tratados
- [ ] Arquivo na pasta correta da estrutura da stack
- [ ] Lint, typecheck e testes passando
