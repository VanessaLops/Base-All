---
name: frontend-react
description: Implementação frontend em React.js + TypeScript seguindo a arquitetura feature-based + Atomic Design do diretório frontend/. Use ao criar ou alterar componentes, páginas, hooks, services, stores, formulários ou estilos no frontend web.
---

# Frontend — React.js (sênior)

**Antes de tudo, aplique `.claude/skills/engineering-standards/SKILL.md`.** Estrutura: `frontend/README.md`.

## Stack padrão
React + TypeScript (strict) · Vite (ou Next.js se SSR/SEO exigir) · React Router · TanStack Query (estado de servidor)
· Zustand (estado global de cliente) · React Hook Form + Zod · Axios · Tailwind ou CSS Modules com tokens · Vitest + Testing Library · Playwright.

## Onde cada coisa vive
| Artefato                    | Local                                  |
|-----------------------------|----------------------------------------|
| Tela (container)            | `features/<feature>/pages`             |
| Componente da feature       | `features/<feature>/components`        |
| Componente reutilizável     | `shared/ui/{atoms,molecules,organisms}`|
| Chamada HTTP                | `features/<feature>/services` usando `shared/api/http` |
| Conversão API ⇄ front       | `shared/api/adapters`                  |
| Schema de formulário        | `features/<feature>/schemas`           |
- Feature não importa outra feature; o que for comum sobe para `shared`.
- Cada feature exporta só pelo `index.ts` público.

## Componentes
- Componentes de **função**, export nomeado (sem `export default`, exceto onde o framework exige).
- Props tipadas com `type <Componente>Props`; estenda props nativas quando for wrapper (`ComponentProps<'button'>`).
- **Presentational** não busca dados nem conhece API — recebe props. **Container** (page) usa hooks e repassa.
- Um componente por arquivo. Passou de ~150 linhas ou tem 2 responsabilidades → divida.
- Composição (`children`, slots) em vez de dezenas de props booleanas.
- Listas: `key` estável e única (id), **nunca o índice** quando a lista muda.
- Renderização condicional: `cond ? <A/> : null` ou early return; evite `&&` com números (`0 &&` renderiza `0`).

## Hooks — uso correto
| Hook          | Use para                                     | Não use para                                   |
|---------------|----------------------------------------------|------------------------------------------------|
| `useState`    | Estado local de UI                           | Valores deriváveis de props/estado (calcule no render) |
| `useEffect`   | Sincronizar com sistema externo (listener, socket, DOM) | Buscar dados (use TanStack Query), derivar estado, reagir a evento do usuário |
| `useMemo`     | Cálculo caro medido                          | Toda variável “por garantia”                   |
| `useCallback` | Função passada a filho memoizado / dependência de hook | Todo handler                       |
| `useRef`      | Referência a DOM ou valor mutável sem re-render | Substituir estado que aparece na tela       |
| `useReducer`  | Estado com várias transições relacionadas    | Um único booleano                              |
- Regras dos hooks: só no topo, nunca em condição/loop. Array de dependências **completo** (sem desabilitar o lint).
- Lógica reutilizável → custom hook `useAlgo` que retorna objeto tipado.

## Estado
- **Servidor** (dados da API) → TanStack Query, com query keys centralizadas por feature.
- **Global de cliente** (sessão, tema) → Zustand em `shared/store`.
- **Local** → `useState`. **URL** (filtros, paginação) → search params.
- Nunca duplique estado de servidor em `useState`/store.

## Dados e API
- Um único cliente HTTP (Singleton) com interceptors de token e erro em `shared/api/http`.
- Services retornam **modelos do front** já adaptados, nunca a resposta crua.
- Valide respostas críticas com Zod; tipos derivados via `z.infer`.
- Trate sempre os 3 estados: carregando, erro, vazio.

## Formulários
React Hook Form + `zodResolver`; o schema é a fonte única dos tipos do form. Mensagens de erro vindas do schema.

## Estilo
Sem cores/tamanhos hardcoded: use tokens de `shared/styles/tokens`. Sem estilo inline, exceto valores dinâmicos.

## Acessibilidade e performance
- HTML semântico (`button` para ação, `a` para navegação), `label` em inputs, `alt` em imagens.
- Lazy load de rotas; otimize só após medir (React DevTools Profiler).

## Testes
Testes pelo comportamento do usuário (`getByRole`), não por detalhes de implementação. Mock de API com MSW.
