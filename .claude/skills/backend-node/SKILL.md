---
name: backend-node
description: Implementação backend em Node.js + TypeScript com Clean Architecture e DDD modular (diretório backend/). Use ao criar ou alterar entidades, value objects, use-cases, repositórios, controllers, rotas, middlewares, providers, migrations ou testes da API.
---

# Backend — Node.js (sênior)

**Antes de tudo, aplique `.claude/skills/engineering-standards/SKILL.md`.** Estrutura: `backend/README.md`.

## Stack padrão
Node LTS · TypeScript strict · Fastify (ou Express) · Prisma (ou Drizzle) · Zod · JWT · Pino (logger) · Vitest + Supertest.

## Regra de dependência
`infra → application → domain`. O `domain` não importa ORM, framework HTTP, env nem libs de infraestrutura.

## Responsabilidade de cada peça
| Peça            | Faz                                                        | Não faz                                  |
|-----------------|------------------------------------------------------------|------------------------------------------|
| Route           | Mapeia verbo + caminho → controller, aplica middlewares    | Lógica                                   |
| Validator       | Valida `body/params/query` com Zod na borda                | Regra de negócio                         |
| Controller      | Extrai input validado, chama **um** use-case, devolve via presenter | Acessar banco, regra de negócio |
| Use-case        | Orquestra uma ação de negócio (`execute(input): Promise<Output>`) | Conhecer HTTP (`req`, `res`, status) |
| Entity / VO     | Regras e invariantes do domínio                            | Persistir-se                             |
| Repository (interface) | Contrato de persistência no `domain`                | —                                        |
| Repository (impl) | Implementa com ORM em `infra/database`                   | Regra de negócio                         |
| Mapper          | Converte persistência ⇄ domínio ⇄ DTO                      | Validação                                |
| Presenter       | Formata a resposta HTTP (oculta campos sensíveis)          | —                                        |
| Provider        | Adapter de serviço externo atrás de uma `port`             | Vazar tipos da lib externa               |

## Convenções de nome
| Artefato        | Classe                       | Arquivo                          |
|-----------------|------------------------------|----------------------------------|
| Use-case        | `CreateUserUseCase`          | `create-user.use-case.ts`        |
| Repository      | `UsersRepository` (interface) / `PrismaUsersRepository` | `users.repository.ts` / `prisma-users.repository.ts` |
| Repo em memória | `InMemoryUsersRepository`    | `tests/mocks/in-memory-users.repository.ts` |
| Controller      | `CreateUserController`       | `create-user.controller.ts`      |
| DTO             | `CreateUserInputDTO` / `CreateUserOutputDTO` | `create-user.dto.ts` |
| Erro            | `UserAlreadyExistsError`     | `user-already-exists.error.ts`   |
| Factory         | `makeCreateUserUseCase`      | `make-create-user.use-case.ts`   |

## Métodos de repositório
Nomes previsíveis e por intenção: `findById`, `findByEmail`, `findMany(params)`, `create`, `save`, `delete`.
Retorno `Promise<Entity | null>` para busca única (sem lançar erro); a decisão de “não encontrado” é do use-case.

## Erros
- Erros de domínio estendem uma classe base (`AppError` / `DomainError`) com `code`.
- Um **error handler global** traduz erro → status HTTP (`NotFound → 404`, `Validation → 400`, `Conflict → 409`, desconhecido → 500 + log).
- Controllers não têm `try/catch` repetido.
- Opcional: use-cases retornam `Either<Error, Output>` quando o fluxo de erro é parte do negócio.

## Dependências
- Use-cases recebem dependências pelo **construtor** (interfaces), montadas em `main/factories` ou `main/container`.
- Nunca `new PrismaClient()` ou `process.env` dentro de use-case/domínio.
- Variáveis de ambiente validadas com Zod **uma vez** em `main/config`; o app não sobe com env inválida.

## Banco de dados
- Toda alteração de schema via migration versionada.
- Operações que precisam ser atômicas em transação (Unit of Work).
- Paginação obrigatória em listagens; selecione só os campos necessários; evite N+1.

## Segurança
Hash de senha (bcrypt/argon2), nunca retornar `password_hash`; rate-limit em auth; CORS restrito;
autorização (RBAC) em middleware; nunca logar tokens ou dados sensíveis.

## HTTP
REST com recursos no plural (`/users/:userId`), verbos corretos (GET lê, POST cria, PUT substitui, PATCH altera, DELETE remove)
e status corretos (`201` criado, `204` sem conteúdo, `400/401/403/404/409/422`).

## Testes
- Unit: use-cases com repositórios em memória (sem banco), padrão SUT (`sut = new CreateUserUseCase(...)`).
- E2E: rotas com banco de teste isolado.
- Factories de entidades em `tests/factories` (`makeUser(override)`).
