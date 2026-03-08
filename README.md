# Mac Nettoyeur

Application multi-plateformes pour nettoyer votre clavier et trackpad en toute sécurité.
Inspirée de [CleanupBuddy](https://cleanupbuddy.app), avec blocage total des entrées pendant le nettoyage.

| Plateforme | Binaire             | Langage         | Documentation                                                  |
|-----------|---------------------|-----------------|---------------------------------------------------------------|
| macOS     | `MacCleaner`        | Swift / SwiftUI | Ce fichier                                                    |
| Linux     | `menagelinux`       | Swift / Xlib    | [LinuxApp/README_LINUX.md](LinuxApp/README_LINUX.md)          |
| Windows   | `menagewindows.exe` | C# / WPF        | [WindowsApp/README_WINDOWS.md](WindowsApp/README_WINDOWS.md) |

---

## Fonctionnalités

- **Blocage complet** du clavier et du trackpad/souris sur les 3 plateformes
- **Mode Automatique** : déverrouillage après 10 secondes (compte à rebours animé)
- **Mode Manuel** : déverrouillage par combinaison de touches maintenue 5 secondes
  - macOS : ⌘ gauche + ⌘ droite
  - Linux : Ctrl gauche + Ctrl droite
  - Windows : ⊞ gauche + ⊞ droite
- Interface sombre et épurée, overlay plein écran pendant le nettoyage

---

## Sommaire (macOS)

1. [Prérequis](#1-prérequis)
2. [Installation de Xcode](#2-installation-de-xcode)
3. [Compilation](#3-compilation)
4. [Distribuer l'application](#4-distribuer-lapplication)
5. [Permission d'accessibilité](#5-permission-daccessibilité)
6. [Utilisation](#6-utilisation)
7. [Architecture du code](#7-architecture-du-code)
8. [Dépannage](#8-dépannage)

---

## 1. Prérequis

| Élément        | Version minimale | Détail                                      |
|---------------|-----------------|---------------------------------------------|
| macOS         | 14.0 (Sonoma)   | Version cible de l'application              |
| Xcode         | 15.0            | IDE Apple, inclut Swift et les outils build |
| Swift         | 5.9             | Inclus dans Xcode 15                        |
| Apple Silicon | M1 / Intel x64  | Les deux architectures sont supportées      |

---

## 2. Installation de Xcode

### Via le Mac App Store (recommandé)

1. Ouvrir le **Mac App Store**
2. Rechercher **Xcode**
3. Cliquer **Obtenir** → **Installer**
4. Attendre (Xcode pèse ~10 Go)
5. Lancer Xcode une première fois pour accepter la licence et installer les outils supplémentaires

### Via xcode-select (ligne de commande uniquement)

Pour compiler sans l'IDE complet :

```bash
xcode-select --install
# Installe les Command Line Tools (~500 Mo), suffisants pour xcodebuild
```

### Vérifier l'installation

```bash
xcodebuild -version
# Xcode 15.x
# Build version 15Xxx

swift --version
# swift-driver version: 1.x  Swift version 5.9
```

---

## 3. Compilation

### Méthode A — Via Xcode (recommandée)

**Étape 1 — Ouvrir le projet**

```bash
open MacCleaner.xcodeproj
```

Ou double-cliquer sur `MacCleaner.xcodeproj` dans le Finder.

**Étape 2 — Configurer la signature de code**

Dans Xcode :
- Sélectionner la cible **MacCleaner** dans le panneau de gauche
- Onglet **Signing & Capabilities**
- Dans **Team**, choisir votre Apple ID

> Pour un usage personnel sans compte développeur payant (99 $/an), sélectionner
> votre Apple ID personnel. La signature "Development" suffit pour tourner
> sur votre propre machine.

**Étape 3 — Compiler et lancer**

```
⌘ + R   →  Compiler et lancer en Debug
⌘ + B   →  Compiler seulement (sans lancer)
```

Le binaire est produit dans le dossier de dérivés Xcode :
```
~/Library/Developer/Xcode/DerivedData/MacCleaner-xxx/Build/Products/Debug/MacCleaner.app
```

### Méthode B — Via xcodebuild (ligne de commande)

```bash
# Compiler en Debug
xcodebuild -project MacCleaner.xcodeproj \
           -scheme MacCleaner \
           -configuration Debug \
           build

# Compiler en Release
xcodebuild -project MacCleaner.xcodeproj \
           -scheme MacCleaner \
           -configuration Release \
           build

# Localiser le .app produit
find ~/Library/Developer/Xcode/DerivedData -name "MacCleaner.app" -type d 2>/dev/null
```

**Lancer depuis le terminal après compilation :**

```bash
APP=$(find ~/Library/Developer/Xcode/DerivedData -name "MacCleaner.app" -type d | head -1)
open "$APP"
```

---

## 4. Distribuer l'application

### Exporter un .app signé (pour partager)

Dans Xcode :

1. **Product** → **Archive** (⌘ + Shift + A)
2. Dans l'Organizer → **Distribute App**
3. Choisir **Developer ID** (pour distribuer hors App Store) ou **Copy App**
4. Suivre l'assistant

> Pour signer avec **Developer ID**, un compte Apple Developer payant est requis (99 $/an).
> Pour un usage personnel et local, un **Direct Distribution** sans signature complète
> suffit (Gatekeeper devra être contourné sur la machine cible avec clic droit → Ouvrir).

### Créer un DMG manuellement

```bash
# 1. Compiler en Release
xcodebuild -project MacCleaner.xcodeproj -scheme MacCleaner -configuration Release build

# 2. Localiser le .app
APP=$(find ~/Library/Developer/Xcode/DerivedData -name "MacCleaner.app" -type d | head -1)

# 3. Créer un dossier de staging
mkdir -p /tmp/MacCleaner-DMG
cp -r "$APP" /tmp/MacCleaner-DMG/
ln -s /Applications /tmp/MacCleaner-DMG/Applications

# 4. Créer le DMG
hdiutil create -volname "Mac Nettoyeur" \
               -srcfolder /tmp/MacCleaner-DMG \
               -ov -format UDZO \
               MacNettoyeur.dmg

# 5. Nettoyer
rm -rf /tmp/MacCleaner-DMG

echo "DMG créé : MacNettoyeur.dmg"
```

---

## 5. Permission d'accessibilité

`CGEventTap` nécessite la permission **Accessibilité** pour intercepter les événements système.

**Au premier lancement**, l'app affiche un avertissement si la permission manque.

**Pour l'accorder manuellement :**

1. Ouvrir **Réglages Système**
2. Aller dans **Confidentialité & Sécurité → Accessibilité**
3. Cliquer sur **+** ou activer **Mac Nettoyeur** dans la liste

> Cliquer sur **"Ouvrir les réglages"** dans l'application déclenche cette demande
> automatiquement. L'avertissement disparaît dans les 2 secondes suivant l'accord.

---

## 6. Utilisation

### Démarrer le nettoyage

1. Sélectionner un mode de déverrouillage
2. Cliquer sur **"Commencer le nettoyage"**
3. Un overlay noir plein écran apparaît — clavier et trackpad sont désactivés

### Modes de déverrouillage

#### Mode Automatique (10 secondes)

Un cercle de progression indique le temps restant.
L'overlay se ferme automatiquement à 0.

#### Mode Manuel (⌘ + ⌘)

Maintenir simultanément **⌘ gauche** et **⌘ droite** pendant **5 secondes**.
Une barre de progression indique l'avancement. Relâcher remet la progression à zéro.

---

## 7. Architecture du code

```
MacCleaner/
├── MacCleanerApp.swift        → Point d'entrée (@main), AppDelegate
├── ContentView.swift          → Interface principale (sélection du mode, bouton)
├── CleaningOverlayView.swift  → Overlay plein écran affiché pendant le nettoyage
├── InputBlocker.swift         → CGEventTap, timers, fenêtre overlay NSWindow
├── Info.plist                 → Métadonnées de l'application
├── MacCleaner.entitlements    → Sandbox désactivé (requis pour CGEventTap)
└── Assets.xcassets/           → Ressources (icône app)
```

### Flux de données

```
ContentView
    └── [bouton] → InputBlocker.startBlocking()
                        ├── setupEventTap()     → CGEventTap intercepte tous les événements
                        ├── showOverlayWindow() → NSWindow au niveau screenSaver
                        └── startAutoTimer()    → Timer 10s (mode Auto)
                            ou startProgressTimer() → Timer 50ms (mode Manuel)
```

### Blocage des événements (`InputBlocker.swift`)

| Type intercepté                     | Description               |
|------------------------------------|---------------------------|
| `keyDown` / `keyUp`                | Touches du clavier        |
| `flagsChanged`                     | Modificateurs (⌘, ⇧, etc.)|
| `leftMouseDown/Up`                 | Clic principal            |
| `rightMouseDown/Up`                | Clic secondaire           |
| `otherMouseDown/Up`                | Clic milieu               |
| `mouseMoved`                       | Mouvement du pointeur     |
| `leftMouseDragged/rightMouseDragged` | Glisser                 |
| `scrollWheel`                      | Défilement trackpad       |

Le callback retourne `nil` → l'OS ignore l'événement.

### Pourquoi pas de sandbox ?

`CGEventTap` avec `.headInsertEventTap` n'est pas compatible avec le sandbox macOS.
Le fichier `.entitlements` le désactive explicitement :

```xml
<key>com.apple.security.app-sandbox</key>
<false/>
```

---

## 8. Dépannage

**Le blocage ne fonctionne pas (clavier toujours actif)**
→ Vérifier la permission Accessibilité dans les Réglages Système.
→ Si Mac Nettoyeur vient d'être ajouté à la liste, le relancer complètement.

**La fenêtre overlay n'apparaît pas en plein écran**
→ L'overlay cible `NSScreen.main`. Sur un setup multi-écrans, il couvre l'écran principal.

**Xcode ne trouve pas le DEVELOPMENT_TEAM**
→ Aller dans **Signing & Capabilities** et sélectionner manuellement votre Apple ID.
→ Ou vider le champ Team et cocher **Automatically manage signing**.

**`xcodebuild: error: 'MacCleaner.xcodeproj' does not exist`**
→ Vérifier d'être dans le bon répertoire :
```bash
ls MacCleaner.xcodeproj   # doit lister le contenu du projet
```

---

## Compilation sur les autres plateformes

### Linux (`menagelinux`)

Voir **[LinuxApp/README_LINUX.md](LinuxApp/README_LINUX.md)** — instructions complètes incluant :
- Installation de Swift sur Linux
- Compilation du binaire
- Packaging **AppImage** (fichier `.AppImage` autonome)
- Packaging **Flatpak** (distribué via Flathub)

```bash
# Compilation rapide
cd LinuxApp && swift build -c release

# AppImage (script tout-en-un)
bash packaging/appimage/build-appimage.sh

# Flatpak (script tout-en-un)
bash packaging/flatpak/build-flatpak.sh
```

### Windows (`menagewindows.exe`)

Voir **[WindowsApp/README_WINDOWS.md](WindowsApp/README_WINDOWS.md)** — instructions complètes incluant :
- Installation du SDK .NET 8
- Compilation Debug / Release
- Publication en `.exe` autonome (sans .NET requis sur la machine cible)
- Création d'un installateur avec Inno Setup

```powershell
# Compilation Release
cd WindowsApp
dotnet build -c Release

# Exécutable autonome (sans .NET requis)
dotnet publish -c Release -r win-x64 --self-contained true -p:PublishSingleFile=true
```

---

## Licence

Usage personnel. Projet créé avec Claude Code.
