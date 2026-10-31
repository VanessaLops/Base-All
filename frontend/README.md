# Frontend — Feature-based + Atomic Design

## Princípios
- **Feature-first**: tudo de um domínio vive junto em `features/<nome>`.
- **Shared** só recebe o que é usado por 2+ features.
- Componentes de UI não chamam API: dados chegam via **hooks** (Container/Presentational).
- Dependência permitida: `app → features → shared` (nunca o inverso; features não importam outras features).

## Estrutura
```
public/assets/            images, icons, fonts
src/
├── app/                  Camada de aplicação
│   ├── routes/           Definição de rotas/páginas
│   ├── layouts/          Layouts (público, autenticado, admin)
│   ├── providers/        Providers globais (tema, query client, auth)
│   └── guards/           Proteção de rotas (auth, roles)
├── features/
│   └── _template/
│       ├── components/   Componentes específicos da feature
│       ├── hooks/        Lógica + estado (Custom Hooks)
│       ├── services/     Chamadas à API da feature
│       ├── store/        Estado local da feature
│       ├── schemas/      Validação de formulários
│       ├── types/        Tipos/interfaces
│       └── pages/        Telas da feature (Containers)
├── shared/
│   ├── ui/               Design System (Atomic Design)
│   │   ├── atoms/        Button, Input, Icon
│   │   ├── molecules/    FormField, SearchBar
│   │   ├── organisms/    Header, DataTable, Modal
│   │   └── templates/    Esqueletos de página
│   ├── hooks/            Hooks genéricos (useDebounce, useMediaQuery)
│   ├── lib/              Wrappers de libs de terceiros
│   ├── api/
│   │   ├── http/         Cliente HTTP único + interceptors (auth, erros)
│   │   └── adapters/     Converte resposta da API ⇄ modelo do front
│   ├── store/            Estado global (sessão, tema)
│   ├── styles/           tokens (cores, espaçamento, tipografia) + themes
│   ├── types/ utils/ constants/ i18n/
└── config/               Env, feature flags
tests/                    unit / integration / e2e / mocks
```

## Design Patterns aplicados
| Pattern                     | Onde                                  |
|-----------------------------|---------------------------------------|
| Atomic Design               | `shared/ui`                           |
| Container / Presentational  | `pages` (container) × `components`    |
| Custom Hooks                | `features/*/hooks`, `shared/hooks`    |
| Adapter                     | `shared/api/adapters`                 |
| Singleton                   | Cliente HTTP em `shared/api/http`     |
| Interceptor / Middleware    | `shared/api/http` (token, refresh, erros) |
| Provider (Context)          | `app/providers`                       |
| Guard                       | `app/guards`                          |
| Design Tokens               | `shared/styles/tokens`                |

## Convenções
- Componentes em `PascalCase`; hooks com prefixo `use`; demais em `kebab-case`.
- Cada pasta de feature expõe um arquivo índice público — importe só por ele.
