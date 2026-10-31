# Ideia: Gerenciador de Mods

> Status: **ideia / proposta** — nada implementado ainda.

## Objetivo
App que detecta os jogos instalados no PC do usuário (The Sims 4, GTA V…), lista os mods instalados
e notifica quando:

1. **Mod desatualizado**: o jogo recebeu um patch e o mod ficou incompatível.
2. **Mod com atualização**: existe uma versão mais nova do mod na fonte.
3. **Mod novo**: um autor, categoria ou jogo que o usuário segue publicou algo novo.

## Restrição principal: precisa ser desktop
Navegador e celular não conseguem ler as pastas do jogo. O gerenciador será um **app desktop em Electron**
(interface em React + processo principal em Node), mantendo a stack do Base-All.

## Funcionalidades

### 1. Detecção de jogos
- Ler bibliotecas de launchers: Steam (`libraryfolders.vdf`), Epic, EA App e o registro do Windows.
- Cada jogo suportado é um **perfil** com: pasta de mods, como ler a versão do jogo e tipos de arquivo de mod.

| Jogo        | Pasta de mods                                    | Versão do jogo          | Arquivos de mod            |
|-------------|--------------------------------------------------|-------------------------|----------------------------|
| The Sims 4  | `Documents/Electronic Arts/The Sims 4/Mods`      | `GameVersion.txt`       | `.package`, `.ts4script`   |
| GTA V       | pasta do jogo + `mods/`                          | versão do executável    | `.asi`, `.dll`, `.rpf`     |

### 2. Identificação dos mods
É a parte mais difícil: os mods quase nunca carregam metadados. A identificação segue esta ordem:
1. Mods instalados pelo próprio gerenciador já ficam registrados (100% de precisão).
2. Hash do arquivo comparado com o catálogo da fonte.
3. Heurística por nome e metadados do arquivo.
4. Associação manual pelo usuário, que o app memoriza.

### 3. Verificação e notificações
| Situação           | Como detectar                                                              |
|--------------------|----------------------------------------------------------------------------|
| Mod desatualizado  | Versão do jogo × versão suportada pelo mod (ou listas de mods quebrados mantidas pela comunidade) |
| Mod tem atualização| Versão instalada × versão mais recente na fonte                            |
| Mod novo           | Feed de autores, categorias ou jogos seguidos                              |

O app verifica periodicamente em segundo plano e mostra **notificações nativas** do sistema operacional.
Opcionalmente, também envia push para o app mobile.

### 4. Fontes de dados
| Fonte        | Acesso                         | Observação                                              |
|--------------|--------------------------------|---------------------------------------------------------|
| Nexus Mods   | API oficial                    | App público precisa ser registrado e seguir os termos de uso |
| CurseForge   | API oficial com chave          | Inclui The Sims 4                                       |
| Sites sem API (ModTheSims, Patreon) | —       | **Não raspar** (termos de uso e fragilidade); tratar como fonte manual |

## Arquitetura no Base-All
```
desktop/     (Electron)  scanner de jogos e mods, agendador de verificação, notificações, UI React
backend/     (Node)      catálogo de mods, cache das APIs, mapa de compatibilidade, contas
mobile/      (opcional)  recebe push "seu mod atualizou"
mcp-server/  (opcional)  "quais mods meus estão quebrados?" via agente de IA
```

### Pontos de extensão (Strategy + Adapter)
- `GameProfile`: um adapter por jogo (detectar, ler versão, listar arquivos de mod).
- `ModSource`: um adapter por fonte (buscar mod, versões, novidades).
- Adicionar um jogo ou uma fonte não altera o núcleo do app.

### Fluxo
```
Scanner ─▶ GameProfile ─▶ arquivos de mod ─▶ Identificador ─▶ ModSource ─▶ Comparador de versões ─▶ Notificação
```

## Riscos
- Identificar mods sem metadados (mitigado com hash + associação manual).
- Limites de requisição e termos de uso das APIs (mitigado com cache no backend).
- Cada jogo organiza os mods de um jeito diferente (mitigado com perfis por jogo).

## MVP proposto
1. **The Sims 4 + CurseForge**: detectar o jogo, listar mods, verificar atualização e notificar.
2. Depois: GTA V + Nexus Mods.
3. Depois: feed de mods novos, push no mobile e integração MCP.

## Decisões em aberto
- Lista de jogos suportados na primeira versão.
- Se o app vai instalar e atualizar mods ou apenas verificar e notificar.
- Se terá contas de usuário (sincronizar entre PCs) ou funcionará só localmente.
