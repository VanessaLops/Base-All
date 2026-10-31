# Backend — Clean Architecture + DDD modular

## Camadas (regra de dependência: de fora para dentro)
```
infra  ──▶  application  ──▶  domain
(frameworks)  (casos de uso)    (regras puras)
```
- `domain` não conhece nada externo (sem ORM, sem HTTP).
- `application` orquestra o domínio e depende só de **interfaces** (ports).
- `infra` implementa as interfaces (banco, HTTP, e-mail, filas).
- `main` é o **Composition Root**: monta as dependências e sobe o servidor.

## Estrutura
```
src/
├── main/                 Composition Root
│   ├── config/           Variáveis de ambiente, configs
│   ├── server/           Bootstrap do servidor HTTP
│   ├── routes/           Agregador de rotas dos módulos
│   ├── factories/        Factory Pattern: monta controllers/use-cases
│   └── container/        Injeção de dependência (DI)
├── modules/              Um módulo por Bounded Context
│   └── _template/
│       ├── domain/
│       │   ├── entities/        Entidades (identidade + regras)
│       │   ├── value-objects/   Email, CPF, Money… (imutáveis)
│       │   ├── repositories/    Interfaces (Repository Pattern)
│       │   ├── events/          Domain Events (Observer)
│       │   └── errors/          Erros de domínio
│       ├── application/
│       │   ├── use-cases/       Um caso de uso por arquivo (Command)
│       │   ├── dtos/            Entrada/saída dos casos de uso
│       │   ├── mappers/         Domain ⇄ DTO ⇄ Persistência
│       │   └── ports/           Interfaces de serviços externos
│       └── infra/
│           ├── database/        Implementação dos repositórios + models ORM
│           ├── http/            controllers, routes, validators, presenters
│           └── providers/       Adapters (gateway pagamento, storage…)
├── shared/               Código transversal entre módulos
│   ├── domain/           Base Entity, AggregateRoot, Result/Either, erros base
│   ├── application/      Contratos genéricos (UseCase, UnitOfWork)
│   ├── infra/            database, middlewares, cache, queue, logger, mail, storage, events
│   ├── utils/
│   └── constants/
database/                 migrations / seeds
tests/                    unit / integration / e2e / factories / mocks
scripts/                  Scripts utilitários (seed, deploy, etc.)
```

## Design Patterns aplicados
| Pattern                 | Onde                                   |
|-------------------------|----------------------------------------|
| Repository              | `domain/repositories` + `infra/database/repositories` |
| Factory                 | `main/factories`                       |
| Dependency Injection    | `main/container`                       |
| Adapter                 | `infra/providers`, `shared/infra/*`    |
| Strategy                | Providers intercambiáveis (ex.: S3 ou disco local) |
| Observer / Domain Event | `domain/events`, `shared/infra/events` |
| Mapper / DTO            | `application/mappers`, `application/dtos` |
| Presenter               | `infra/http/presenters` (formata resposta) |
| Chain of Responsibility | `shared/infra/http/middlewares`        |
| Result / Either         | `shared/domain/base` (erros sem exceção) |
| Unit of Work            | `shared/application/contracts`         |

## Convenções
- Arquivos: `kebab-case` — `create-user.use-case`, `user.repository`, `user.controller`.
- Um módulo **não** importa o `infra` de outro módulo; comunicação via use-case ou eventos.
- Testes unitários cobrem `domain` e `application` com repositórios em memória (`tests/mocks`).
