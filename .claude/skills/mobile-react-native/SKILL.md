---
name: mobile-react-native
description: Implementação mobile em React Native + Expo + TypeScript no padrão de estrutura da Rocketseat (diretório mobile/). Use ao criar ou alterar telas, componentes, rotas, contexts, hooks, services, storage ou tema do app mobile.
---

# Mobile — React Native + Expo (sênior)

**Antes de tudo, aplique `.claude/skills/engineering-standards/SKILL.md`.** Estrutura: `mobile/README.md`.
As regras de componentes e hooks de `.claude/skills/frontend-react/SKILL.md` (seções Componentes e Hooks) também valem aqui.

## Stack padrão
Expo (SDK atual) · TypeScript strict · React Navigation (native-stack + bottom-tabs) · Context API (auth/sessão)
· TanStack Query (dados de servidor) · React Hook Form + Zod · Axios · AsyncStorage (ou SecureStore para tokens)
· gluestack-ui / NativeWind ou StyleSheet com `@theme` · Jest + React Native Testing Library.
Instale libs nativas com `npx expo install` para manter versões compatíveis com o SDK.

## Onde cada coisa vive (padrão Rocketseat)
| Artefato                         | Local                         | Exemplo de nome                    |
|----------------------------------|-------------------------------|------------------------------------|
| Tela                             | `src/screens`                 | `SignIn.tsx`                       |
| Componente reutilizável          | `src/components`              | `Button.tsx`, `ExerciseCard.tsx`   |
| Rotas                            | `src/routes`                  | `index.tsx`, `auth.routes.tsx`, `app.routes.tsx` |
| Estado global                    | `src/contexts`                | `AuthContext.tsx`                  |
| Hook de consumo do context       | `src/hooks`                   | `useAuth.ts`                       |
| Contrato da API                  | `src/dtos`                    | `UserDTO.ts`                       |
| Cliente HTTP                     | `src/services`                | `api.ts`                           |
| Persistência local               | `src/storage`                 | `storageConfig.ts`, `storageUser.ts`, `storageAuthToken.ts` |
| Tokens visuais                   | `src/theme`                   | `index.ts`                         |
| Erro da aplicação                | `src/utils`                   | `AppError.ts`                      |
| Declarações globais              | `src/@types`                  | `png.d.ts`, `navigation.d.ts`      |
- Importe sempre por alias (`@components/Button`), nunca `../../..`.

## Navegação
- `routes/index.tsx` decide entre `AuthRoutes` e `AppRoutes` com base no `user` do `useAuth`.
- Tipar rotas: `type AppRoutes = { home: undefined; exercise: { exerciseId: string } }` e
  `AppNavigatorRoutesProps = BottomTabNavigationProp<AppRoutes>` exportados do arquivo de rotas.
- Parâmetros de rota levam só IDs; a tela busca os dados.

## Context + Storage
- `AuthContext` expõe estado e ações tipados (`user`, `signIn`, `signOut`, `isLoadingUserStorageData`).
- O context **orquestra**; quem lê/escreve no AsyncStorage são as funções de `storage/` (uma por responsabilidade).
- Chaves de storage centralizadas em `storageConfig.ts` (constantes, com prefixo do app).
- Token de autenticação: prefira `expo-secure-store`.
- Context com valor memoizado para evitar re-render em cascata.

## API
- `services/api.ts`: instância única com `baseURL` via env (`EXPO_PUBLIC_API_URL`), interceptor que converte erros em `AppError`
  e fila de refresh token.
- Telas tratam erro com `error instanceof AppError ? error.message : 'mensagem genérica'`.

## Componentes nativos — uso correto
| Objetivo                    | Use                                  | Não use                              |
|-----------------------------|--------------------------------------|--------------------------------------|
| Lista longa / dinâmica      | `FlatList` / `FlashList` com `keyExtractor` | `ScrollView` + `map`          |
| Lista com seções            | `SectionList`                        | várias `FlatList` aninhadas          |
| Toque                       | `Pressable` / `TouchableOpacity`     | `onPress` em `View`/`Text`           |
| Área segura                 | `react-native-safe-area-context`     | padding fixo para notch              |
| Teclado sobre inputs        | `KeyboardAvoidingView` / `ScrollView` com `keyboardShouldPersistTaps` | ignorar teclado |
| Imagem                      | `expo-image`                         | `Image` sem tamanho definido         |
| Diferença por plataforma    | `Platform.select` / arquivos `.ios.tsx` `.android.tsx` | `if` espalhado     |
| Estilos                     | `StyleSheet.create` ou classes do tema | objetos inline recriados a cada render |

## Estilo e tema
- Cores, fontes, tamanhos e espaçamentos **só** de `@theme`. Nenhum hex hardcoded em tela.
- Fontes carregadas no `App.tsx` (`useFonts`) com `Loading` até ficarem prontas.

## Performance
- `FlatList`: `keyExtractor` com id, `renderItem` estável, `getItemLayout` quando altura fixa.
- `React.memo` em itens de lista pesados; evite funções anônimas como props de itens memoizados.
- Animações com `react-native-reanimated` (thread de UI), não `Animated` com `useNativeDriver: false`.

## Testes
RN Testing Library pelo comportamento (`getByText`, `getByRole`); mock de `@services/api` e do storage em `tests/mocks`.
