# SongoGame — documentation développeur (centrée sur le code)

Objectif: documenter **l’implémentation** et les **concepts de code** utilisés dans ce repo (React Native CLI + TypeScript). Ce document est destiné aux développeurs qui reprennent le code.

## 1) Comment lire ce repo (la “carte” minimale)

Chemins principaux à connaître:
- Point d’entrée: `App.tsx`
- Navigation: `src/navigation/RootNavigator.tsx`, `src/navigation/TabsNavigator.tsx`, `src/navigation/types.ts`
- Écrans: `src/screens/` (ex: `src/screens/GameScreen.tsx`)
- UI réutilisable: `src/components/` (ex: `src/components/Pit.tsx`, `src/components/Board.tsx`, `src/components/TopAppBar.tsx`)
- Stores (état global): `src/store/` (Zustand)
- Logique “métier” pure (règles): `src/modules/gameEngine/`
- IA: `src/modules/bot/simpleBot.ts`
- Assets: `src/assets/` (avatars, fonts)
- Tests: `__tests__/` + mocks natifs: `__mocks__/`

### 1.1 Navigation typée (pourquoi `types.ts` existe)
React Navigation passe des `params` entre écrans. Pour éviter:
- des params manquants,
- des mauvais noms de routes,
- des types “any” partout,
on centralise les types dans `src/navigation/types.ts`, puis on les réutilise dans les navigators et les screens.

## 2) Notions de programmation (appliquées au code ici)

### 2.1 Module (fichier) vs “classe”
- Un **module** = un fichier `.ts/.tsx` qui exporte des choses (`export ...`).
  - Exemple: `src/modules/gameEngine/engine.ts` exporte des fonctions de règles.
- Une **classe** (OOP) encapsule état + méthodes via `class`.
  - Dans ce repo, l’architecture est surtout **fonctionnelle**: on manipule des **objets** et des **fonctions**. Les classes sont rares (voire absentes selon les zones).

### 2.2 Fonction
Une **fonction**:
- reçoit des **paramètres** (entrées),
- calcule,
- retourne une **valeur** (sortie) ou provoque un **effet**.

Deux catégories utiles pour comprendre le code:
1) **Fonctions pures** (idéales pour les règles):
   - pas d’I/O, pas de `Date.now()`, pas de storage, pas de navigation
   - mêmes entrées → mêmes sorties
   - se testent facilement
   - Exemple: moteur dans `src/modules/gameEngine/engine.ts`
2) **Fonctions à effets** (*side effects*):
   - touchent au temps (timers), au stockage, au son/haptics, à la navigation, etc.
   - Exemple: persistance via AsyncStorage dans `src/store/useSettingsStore.ts`

### 2.3 Types (TypeScript): “décrire les données”
Un **type** (ou une `interface`) décrit la forme d’une donnée.
- Exemple typique: `EngineState`, `Player`, `GameStatus` dans `src/modules/gameEngine/types.ts`.

Pourquoi c’est important ici:
- UI ↔ store ↔ moteur échangent des objets; les types rendent ces contrats **explicites** et évitent les incohérences.

### 2.4 Composant React: “une fonction qui rend de l’UI”
Un **composant** est une fonction qui retourne du JSX (des vues RN).
- Exemple: `src/components/Pit.tsx` expose un composant clickable/pressable.

Règle mentale:
- si des **props** ou un **state** changent → React **re-render** le composant.

### 2.5 Hook: “brancher logique ↔ rendu”
Un **hook** est une fonction (souvent `useX`) qui connecte un composant à une logique:
- `useState`, `useEffect` (React)
- `useGameStore(...)`, `useSettingsStore(...)` (Zustand) dans `src/store/`

### 2.6 Props, state, immutabilité (pour éviter des bugs “fantômes”)
- **Props**: entrées d’un composant (comme des paramètres de fonction).
- **State**: mémoire interne d’un composant (ou d’un store) qui, lorsqu’elle change, déclenche un re-render.
- **Immutabilité (pratique)**: on évite de “modifier en place” des objets/tableaux partagés.
  - En React/Zustand, si tu mutates un objet sans changer sa référence, tu peux empêcher la UI de se mettre à jour.
  - Bon réflexe: produire un **nouvel objet** / **nouveau tableau** quand tu fais une mise à jour.

### 2.7 Effets (useEffect) et pièges classiques
`useEffect` exécute du code **après** un rendu, typiquement pour des effets:
- démarrer/arrêter un timer
- déclencher un son/haptics
- réagir à un changement de joueur/mode

Piège: oublier de “nettoyer” (cleanup) un interval → timers multiples.
Dans ce repo, les timers sont principalement gérés dans `src/screens/GameScreen.tsx`.

## 3) Flux d’exécution: UI → Store → Moteur → UI

Le chemin standard d’un coup joué ressemble à ça:
1) l’utilisateur interagit avec l’UI (tap/press sur un pit)
2) l’écran déclenche une **action** du store (Zustand)
3) l’action appelle le **moteur** (fonctions pures) pour calculer le nouvel état
4) le store met à jour son state → les composants qui lisent ce state se re-render

Fichiers à suivre (lecture recommandée dans cet ordre):
1) `App.tsx` (boot + providers)
2) `src/navigation/RootNavigator.tsx` et `src/navigation/TabsNavigator.tsx` (quels écrans existent)
3) `src/screens/GameScreen.tsx` (orchestration UI + timers)
4) `src/store/useGameStore.ts` (orchestration d’une partie)
5) `src/modules/gameEngine/engine.ts` (règles pures)
6) `src/components/Board.tsx` et `src/components/Pit.tsx` (interaction plateau)

## 4) Zustand (stores): “state + actions”, source de vérité

### 4.1 Pourquoi un store?
Sans store, chaque écran maintient son état “local” → duplications et incohérences. Ici les stores centralisent:
- la partie en cours: `src/store/useGameStore.ts`
- le profil & préférences: `src/store/useSettingsStore.ts`
- les stats & streak: `src/store/useUserStatsStore.ts`

### 4.2 Lire vs écrire
- **Lire**: un composant lit un sous-ensemble du state (souvent via un sélecteur).
- **Écrire**: un composant appelle une **action** (fonction) exposée par le store.

### 4.3 `useGameStore`: orchestration match
`src/store/useGameStore.ts` contient typiquement:
- configuration de match (mode, timers, ids)
- `engineState` (l’état du plateau + joueur courant + scores + statut)
- des **actions** qui font avancer le match (ex: init, coup, undo, surrender, fin de match)

Point clé:
- le store **n’invente pas les règles**: il délègue au moteur (`src/modules/gameEngine/*`) puis stocke le résultat.

### 4.4 Mini-exemple (lecture/écriture depuis un composant)
Exemple conceptuel (le vrai code varie selon les écrans):

```ts
const currentPlayer = useGameStore(s => s.engineState.currentPlayer);
const playMove = useGameStore(s => s.playMove);

return <Pit onPress={() => playMove(pitIndex)} disabled={currentPlayer !== PLAYER_1} />;
```

## 5) Moteur de jeu: code pur, testable, réutilisable

Répertoire: `src/modules/gameEngine/`

### 5.1 Principe de séparation
Le moteur doit rester:
- indépendant de React Native (pas de UI)
- indépendant du temps réel (pas de timers)
- indépendant du stockage

But: pouvoir tester et raisonner sur les règles “à froid”.

### 5.2 Données principales
- Plateau: tableau `0..13` (cf. `src/modules/gameEngine/types.ts` + `src/modules/gameEngine/constants.ts`)
- Le moteur transforme un `EngineState` en un nouvel `EngineState`.

### 5.3 Pipeline mental pour un coup
Dans `src/modules/gameEngine/engine.ts`, on peut lire le coup comme une suite:
1) **Validation**: le coup est-il légal? (`isValidMove`)
2) **Semailles**: prise + distribution (`applySowing`)
3) **Capture**: calcul des prises (`computeCapture`)
4) **Fin de partie**: status/winner (`finalizeIfGameOver`)

## 6) Timers: deux notions, deux comportements

Où: `src/screens/GameScreen.tsx` + actions dans `src/store/useGameStore.ts`.

- **Temps global du match**
  - s’écoule en continu pendant tout le match
  - ne se réinitialise pas à chaque tour
  - à zéro: fin du match (action type `endMatchOnTime()`)

- **Temps de réflexion (par tour)**
  - se réinitialise à chaque changement de joueur
  - à zéro: le joueur **perd son tour** (action type `forfeitTurn()`), le match continue

## 7) Persistance: ce qui est stocké et comment (AsyncStorage)

### 7.1 Pattern utilisé
Les stores persistés utilisent `zustand/middleware`:
- `persist(...)` sérialise en JSON
- `createJSONStorage(() => AsyncStorage)` utilise `@react-native-async-storage/async-storage`

### 7.2 Stores persistés
- `src/store/useSettingsStore.ts` (ex: pseudo, avatar, toggles) — clé: `songo.settings`
- `src/store/useUserStatsStore.ts` (ex: streak, stats cumulées) — clé: `songo.userStats`

### 7.3 Store non persisté (volontaire)
- `src/store/useGameStore.ts` n’est pas persisté: un match en cours dépend de timers/side-effects; restaurer “à moitié” crée des états incohérents si ce n’est pas pensé end-to-end.

## 8) Tests: ce qui est testé + pourquoi des mocks existent

### 8.1 Tests unitaires (logique)
- `__tests__/gameEngine.test.ts`: idéal car teste des fonctions pures du moteur
- `__tests__/bot.test.ts`: teste l’IA simple

### 8.2 Tests de rendu
- `__tests__/App.test.tsx`: smoke test du rendu racine

### 8.3 Mocks “native”
En Jest, certains modules RN natifs ne sont pas disponibles; on les remplace par des mocks:
- `__mocks__/@react-native-async-storage/async-storage.js`
- `__mocks__/react-native-haptic-feedback.js`
- `__mocks__/react-native-vector-icons/MaterialIcons.js`

## 9) Guide pratique: ajouter une logique sans “hardcoder” dans l’UI

Règle: si une donnée doit être fiable, évolutive, ou partagée entre écrans → elle doit être:
- dans un **store** (state + actions), ou
- dans le **moteur** (règles pures),
pas comme une constante locale dans un écran.

Checklist:
1) Définir/étendre le **type** de données au bon endroit (souvent `src/modules/gameEngine/types.ts` ou un type de store).
2) Mettre la logique au bon niveau:
   - règles → `src/modules/gameEngine/`
   - orchestration match → `src/store/useGameStore.ts`
   - préférences profil → `src/store/useSettingsStore.ts`
   - stats/streak → `src/store/useUserStatsStore.ts`
3) Exposer une **action** côté store, puis brancher la UI sur cette action.
4) Tester les fonctions pures (moteur/bot) dans `__tests__/` quand c’est pertinent.
