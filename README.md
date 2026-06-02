## 📄 README.md (SongoGame):


# SongoGame

Jeu de société traditionnel (Songo) sur mobile – React Native CLI.  
Deux joueurs, plateau 2×7 cases, semailles, prises, règles de solidarité.

---
#Aris Workflow 

## 🚀 Installation et premier lancement

```bash
git clone https://github.com/ton-compte/SongoGame.git
cd SongoGame
npm install
npx react-native start --reset-cache
```

**Second terminal :**
```bash
npx react-native run-android
```

> Appareil USB : `adb reverse tcp:8081 tcp:8081` avant `run-android`

---
## 📁 Structure et rôle des modules

### 🧠 `gameEngine`
**Rôle** : Logique métier complète du Songo.  
**Exporte** : `getBoard()`, `getScores()`, `playMove()`, `isValidMove()`, `isGameOver()`, `getWinner()`

### 🎨 `board`
**Rôle** : Affiche le plateau (2 lignes × 7 cases) avec le nombre de graines par case.  
**Exporte** : `BoardComponent` (props : `board`, `currentPlayer`, `onHousePress`)

### 🔄 `turnManager`
**Rôle** : Gère l'alternance et valide les coups avant exécution.  
**Exporte** : `getCurrentPlayer()`, `switchTurn()`, `validateAndExecuteMove()`

### 🖥️ `ui`
**Rôle** : Affiche les scores, l'écran principal, l'écran de fin de partie, et le bouton "Nouvelle partie".  
**Exporte** : `MainScreen` (props : `board`, `currentPlayer`, `scores`, `onHousePress`, `gameOver`, `winner`, `onNewGame`)

### 🔗 `integration`
**Rôle** : Assemble tous les modules dans `App.js`.  
**Exporte** : Rien – point d'entrée final.

---

## 🌿 Branches

| Module | Branche |
|--------|---------|
| `gameEngine` | `feat/game-engine` |
| `board` | `feat/board` |
| `turnManager` | `feat/turn-manager` |
| `ui` | `feat/ui` |
| `integration` | `feat/integration` |

```bash
git checkout feat/votre-module
```

---

## 🧪 Tester son module seul

Créer un mock dans `src/mocks/` pour simuler un autre module.

Exemple pour `board` qui a besoin de `gameEngine` :

```javascript
// src/mocks/mockGameEngine.js
export const mockGetBoard = () => [
  [5,5,5,5,5,5,5], // Sud
  [5,5,5,5,5,5,5]  // Nord
];
export const mockGetCurrentPlayer = () => 0;
```

---

## ⚠️ Règles d'or

1. Ne toucher qu'à `src/modules/son-module/`
2. Ne pas modifier `src/shared/interfaces.js` sans validation
3. Commits fréquents sur sa branche
4. Pas de merge sur `main` sans le lead

---

## 🔧 Dépendances (aucune pour l'instant – jeu 100% logique)

Aucune librairie externe nécessaire. Tout est en JavaScript pur.

---

## 📞 Problèmes fréquents

| Problème | Solution |
|----------|----------|
| Metro ne démarre pas | `npx react-native start --reset-cache` |
| `@react-native/metro-config` manquant | `npm install --save-dev @react-native/metro-config` |
| L'app plante | `npx react-native log-android` |
