# Mobile — React Native + Expo (padrão Rocketseat)

Estrutura no modelo usado nos projetos do Ignite (Rocketseat): `src/` plana por tipo de artefato,
React Navigation, Context API para estado global e path aliases (`@components`, `@screens`…).

## Estrutura
```
mobile/
├── App.tsx                  Ponto de entrada: fontes, providers, <Routes />
├── app.json                 Configuração do Expo
├── babel.config.js          module-resolver com os aliases
├── tsconfig.json            paths com os mesmos aliases
└── src/
    ├── @types/              Declarações globais (png.d.ts, svg.d.ts, navigation.d.ts)
    ├── assets/              images / icons / fonts
    ├── components/          Componentes reutilizáveis (Button, Input, Header, Loading)
    ├── contexts/            Context API (AuthContext) — estado global
    ├── dtos/                Contratos de dados da API (UserDTO, ExerciseDTO)
    ├── hooks/               Hooks que consomem contexts (useAuth)
    ├── libs/                Configuração de libs externas (dayjs, react-query, onesignal)
    ├── routes/              index.tsx, auth.routes.tsx, app.routes.tsx
    ├── screens/             Telas (SignIn, SignUp, Home, Profile)
    ├── services/            api.ts — instância HTTP + interceptors (token / refresh)
    ├── storage/             storageConfig, storageUser, storageAuthToken (AsyncStorage)
    ├── theme/               Tokens: colors, fonts, fontSizes, sizes
    └── utils/               AppError e helpers puros
tests/                       unit / integration / mocks
```

## Path aliases
`@assets @components @contexts @dtos @hooks @libs @routes @screens @services @storage @theme @utils`

## Fluxo
```
Screen ─▶ hook (useAuth) ─▶ context ─▶ services/api ─▶ Backend
                              └──────▶ storage (persistência local)
```
Skill de implementação: `.claude/skills/mobile-react-native`.
