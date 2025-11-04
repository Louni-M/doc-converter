# Doc Converter - APIs Utilisés

## Vue d'ensemble

Doc Converter utilise plusieurs APIs et bibliothèques pour accomplir ses fonctionnalités de conversion de documents. Voici une documentation complète de tous les APIs utilisés.

---

## 📦 APIs et Bibliothèques Principales

### 1. Electron APIs

#### **Main Process APIs**

**app (Electron.app)**
- **Usage** : Gestion du cycle de vie de l'application
- **Méthodes utilisées** :
  ```javascript
  app.on('ready', () => {})           // Démarrage de l'application
  app.on('window-all-closed', () => {}) // Fermeture de toutes les fenêtres
  app.on('activate', () => {})        // Réactivation sur macOS
  app.quit()                          // Quitter l'application
  app.getPath('userData')             // Chemin des données utilisateur
  app.getPath('temp')                 // Chemin des fichiers temporaires
  ```

**BrowserWindow (Electron.BrowserWindow)**
- **Usage** : Création et gestion des fenêtres
- **Configuration** :
  ```javascript
  const mainWindow = new BrowserWindow({
    width: 1200,
    height: 800,
    webPreferences: {
      sandbox: true,              // Sandboxing activé
      contextIsolation: true,     // Isolation du contexte
      nodeIntegration: false,     // Node.js désactivé dans renderer
      preload: path.join(__dirname, 'preload.js')
    }
  })
  ```

**ipcMain (Electron.ipcMain)**
- **Usage** : Communication inter-processus (Main ← Renderer)
- **Handlers** :
  ```javascript
  ipcMain.handle('select-file', async () => {})
  ipcMain.handle('convert-file', async (event, { inputPath, outputFormat }) => {})
  ipcMain.on('cancel-conversion', (event) => {})
  ```

**dialog (Electron.dialog)**
- **Usage** : Dialogues système (sélection de fichiers, alertes)
- **Méthodes** :
  ```javascript
  dialog.showOpenDialog({
    properties: ['openFile'],
    filters: [
      { name: 'Documents', extensions: ['pdf', 'docx', 'doc', 'xlsx'] }
    ]
  })

  dialog.showSaveDialog({
    defaultPath: 'converted-document.pdf',
    filters: [{ name: 'PDF', extensions: ['pdf'] }]
  })

  dialog.showMessageBox({
    type: 'error',
    title: 'Conversion Error',
    message: 'Failed to convert document'
  })
  ```

#### **Renderer Process APIs**

**contextBridge (Electron.contextBridge)**
- **Usage** : Pont sécurisé entre Main et Renderer
- **Implementation** :
  ```javascript
  contextBridge.exposeInMainWorld('electronAPI', {
    selectFile: () => ipcRenderer.invoke('select-file'),
    convertFile: (inputPath, outputFormat) =>
      ipcRenderer.invoke('convert-file', { inputPath, outputFormat }),
    onProgress: (callback) =>
      ipcRenderer.on('conversion-progress', callback),
    onComplete: (callback) =>
      ipcRenderer.on('conversion-complete', callback),
    onError: (callback) =>
      ipcRenderer.on('conversion-error', callback)
  })
  ```

**ipcRenderer (Electron.ipcRenderer)**
- **Usage** : Communication inter-processus (Renderer → Main)
- **Méthodes** :
  ```javascript
  ipcRenderer.invoke('channel', data)  // Appel asynchrone
  ipcRenderer.on('channel', callback)  // Écouter des événements
  ipcRenderer.send('channel', data)    // Envoi one-way
  ```

---

### 2. Node.js Core APIs

#### **fs (File System)**
- **Usage** : Opérations sur le système de fichiers
- **Méthodes utilisées** :
  ```javascript
  const fs = require('fs').promises;

  // Lecture de fichiers
  await fs.readFile(path, 'utf8')
  await fs.readFile(path)  // Buffer binaire

  // Écriture de fichiers
  await fs.writeFile(path, content)

  // Informations sur les fichiers
  await fs.stat(path)
  await fs.access(path, fs.constants.R_OK)

  // Gestion des répertoires
  await fs.mkdir(path, { recursive: true })
  await fs.rm(path, { recursive: true })
  await fs.readdir(path)

  // Suppression
  await fs.unlink(path)  // Supprimer un fichier
  ```

#### **path**
- **Usage** : Manipulation des chemins de fichiers
- **Méthodes** :
  ```javascript
  const path = require('path');

  path.join(__dirname, 'file.txt')      // Joindre des chemins
  path.resolve('relative/path')         // Résoudre en absolu
  path.normalize('/path/../file')       // Normaliser
  path.basename('/path/file.txt')       // Nom du fichier
  path.dirname('/path/file.txt')        // Répertoire
  path.extname('file.txt')              // Extension (.txt)
  path.isAbsolute('/path')              // Vérifier si absolu
  ```

#### **os**
- **Usage** : Informations système
- **Méthodes** :
  ```javascript
  const os = require('os');

  os.homedir()          // Répertoire home de l'utilisateur
  os.tmpdir()           // Répertoire temporaire
  os.platform()         // Plateforme (darwin, win32, linux)
  os.arch()             // Architecture (x64, arm64)
  ```

#### **child_process**
- **Usage** : Exécution de processus externes (LibreOffice)
- **Méthodes** :
  ```javascript
  const { spawn } = require('child_process');

  const process = spawn(command, args, {
    timeout: 60000,
    shell: false,  // Important pour la sécurité
    env: { HOME: os.homedir() }
  });

  process.stdout.on('data', (data) => {});
  process.stderr.on('data', (data) => {});
  process.on('close', (code) => {});
  process.on('error', (error) => {});
  ```

#### **crypto**
- **Usage** : Génération de noms de fichiers aléatoires
- **Méthodes** :
  ```javascript
  const crypto = require('crypto');

  crypto.randomBytes(16).toString('hex')  // Génère une chaîne aléatoire
  ```

---

### 3. JSZip Library (v3.10.1)

**Description** : Bibliothèque JavaScript pure pour créer, lire et modifier des fichiers ZIP.

**Usage** : Manipulation de documents Office (DOCX, XLSX) qui sont des archives ZIP contenant du XML.

#### **API Principale**

```javascript
const JSZip = require('jszip');

// Charger un fichier ZIP existant
const data = await fs.readFile('document.docx');
const zip = await JSZip.loadAsync(data);

// Lire le contenu d'un fichier
const content = await zip.file('word/document.xml').async('string');
const buffer = await zip.file('word/media/image1.png').async('nodebuffer');

// Créer un nouveau ZIP
const newZip = new JSZip();
newZip.file('document.xml', xmlContent);
newZip.folder('subfolder');
newZip.file('subfolder/file.txt', 'content');

// Générer l'archive
const buffer = await newZip.generateAsync({
  type: 'nodebuffer',
  compression: 'DEFLATE',
  compressionOptions: { level: 9 }
});

await fs.writeFile('output.zip', buffer);

// Parcourir tous les fichiers
zip.forEach((relativePath, file) => {
  console.log(relativePath, file.name, file.dir);
});

// Vérifier l'existence d'un fichier
const exists = zip.file('path/to/file.xml') !== null;
```

#### **Méthodes JSZip**

| Méthode | Description | Retour |
|---------|-------------|--------|
| `loadAsync(data)` | Charge un ZIP depuis un Buffer/Blob | Promise<JSZip> |
| `file(name)` | Accède à un fichier dans le ZIP | ZipObject |
| `file(name, data)` | Ajoute/modifie un fichier | JSZip |
| `folder(name)` | Crée un dossier | JSZip |
| `forEach(callback)` | Itère sur tous les fichiers | void |
| `generateAsync(options)` | Génère l'archive ZIP | Promise<Buffer> |
| `remove(name)` | Supprime un fichier/dossier | JSZip |

#### **ZipObject Methods**

```javascript
const file = zip.file('word/document.xml');

await file.async('string')      // Contenu en string
await file.async('nodebuffer')  // Contenu en Buffer
await file.async('uint8array')  // Contenu en Uint8Array
await file.async('arraybuffer') // Contenu en ArrayBuffer

// Propriétés
file.name                       // Nom du fichier
file.dir                        // true si c'est un dossier
file._data.uncompressedSize    // Taille décompressée
file._data.compressedSize      // Taille compressée
```

---

### 4. Dépendances de JSZip

#### **pako (v1.0.2)**
- **Usage** : Compression/décompression (implémentation de zlib en JavaScript)
- **API** :
  ```javascript
  const pako = require('pako');

  const compressed = pako.deflate(data);
  const decompressed = pako.inflate(compressed);
  ```

#### **readable-stream (v2.3.6)**
- **Usage** : Polyfill de streams Node.js pour compatibilité
- **API** : Compatible avec l'API standard de Node.js streams

#### **lie (v3.3.0)**
- **Usage** : Polyfill de Promises pour anciens environnements
- **Note** : Utilisé en interne par JSZip

#### **setimmediate (v1.0.5)**
- **Usage** : Polyfill de setImmediate pour navigateurs
- **Note** : Utilisé en interne par JSZip

---

## 🔧 APIs Externes (Optionnels)

### LibreOffice CLI

**Commande** : `/Applications/LibreOffice.app/Contents/MacOS/soffice`

**Usage** : Conversion avancée de documents via ligne de commande

#### **Syntaxe**

```bash
soffice --headless --convert-to <format> --outdir <dir> <input-file>
```

#### **Exemples**

```bash
# PDF → DOCX
soffice --headless --convert-to docx --outdir /tmp input.pdf

# DOCX → PDF
soffice --headless --convert-to pdf --outdir /tmp document.docx

# XLSX → CSV
soffice --headless --convert-to csv --outdir /tmp spreadsheet.xlsx

# HTML → PDF
soffice --headless --convert-to pdf --outdir /tmp webpage.html
```

#### **Formats Supportés**

**Input** : PDF, DOC, DOCX, XLS, XLSX, PPT, PPTX, ODT, ODS, ODP, RTF, TXT, HTML, CSV

**Output** : PDF, DOCX, XLSX, HTML, TXT, CSV, ODP, ODT, ODS

#### **Options Avancées**

```bash
# Filtres de conversion
--infilter="filter_name"
--convert-to "format:filter_name"

# Format PDF avec options
--convert-to "pdf:writer_pdf_Export"

# Timeout
timeout 60s soffice --headless --convert-to pdf input.docx
```

---

## 🌐 Web APIs (Renderer Process)

### DOM APIs

```javascript
// Sélection d'éléments
document.getElementById('element-id')
document.querySelector('.class-name')
document.querySelectorAll('div')

// Manipulation
element.addEventListener('click', callback)
element.classList.add('active')
element.textContent = 'Hello'
element.innerHTML = '<b>Bold</b>'

// Création
const div = document.createElement('div')
document.body.appendChild(div)
```

### Fetch API (si activé)
```javascript
// Note: Actuellement désactivé (pas d'accès réseau)
// Pourrait être utilisé pour futures fonctionnalités
fetch('https://api.example.com/convert')
  .then(response => response.json())
  .then(data => console.log(data))
```

### File API (Drag & Drop)

```javascript
// Zone de drop
dropZone.addEventListener('drop', (event) => {
  event.preventDefault();
  const files = event.dataTransfer.files;
  // Envoyer le chemin à Main Process via IPC
});

dropZone.addEventListener('dragover', (event) => {
  event.preventDefault();
});
```

---

## 📡 IPC (Inter-Process Communication)

### Channels Utilisés

#### **Main → Renderer**

| Channel | Payload | Description |
|---------|---------|-------------|
| `conversion-progress` | `{ percent: number, status: string }` | Progression de la conversion |
| `conversion-complete` | `{ outputPath: string, duration: number }` | Conversion terminée |
| `conversion-error` | `{ error: string, code: string }` | Erreur de conversion |
| `file-selected` | `{ path: string, name: string }` | Fichier sélectionné |

#### **Renderer → Main**

| Channel | Payload | Retour | Description |
|---------|---------|--------|-------------|
| `select-file` | - | `string` (path) | Ouvre le dialogue de sélection |
| `convert-file` | `{ inputPath, outputFormat }` | `Promise<void>` | Lance la conversion |
| `cancel-conversion` | - | `void` | Annule la conversion en cours |
| `get-supported-formats` | - | `string[]` | Liste des formats supportés |

### Exemple d'utilisation

**Renderer Process**:
```javascript
// Sélectionner un fichier
const filePath = await window.electronAPI.selectFile();

// Lancer la conversion
window.electronAPI.onProgress((data) => {
  console.log(`Progress: ${data.percent}%`);
});

window.electronAPI.onComplete((data) => {
  console.log(`Conversion complete: ${data.outputPath}`);
});

window.electronAPI.onError((error) => {
  console.error(`Error: ${error.error}`);
});

await window.electronAPI.convertFile(filePath, 'pdf');
```

---

## 🔒 Security Context

### Permissions Déclarées (Info.plist)

```xml
<key>NSCameraUsageDescription</key>
<string>This app needs access to the camera</string>

<key>NSMicrophoneUsageDescription</key>
<string>This app needs access to the microphone</string>

<key>NSBluetoothAlwaysUsageDescription</key>
<string>This app needs access to Bluetooth</string>
```

**Note** : Ces permissions sont déclarées mais **non utilisées** dans la version actuelle.

### APIs Désactivés pour Sécurité

- ❌ **Node Integration** dans Renderer (désactivé)
- ❌ **Remote Module** (désactivé)
- ❌ **eval()** et code dynamique (non utilisé)
- ❌ **Accès réseau** dans Renderer (bloqué par CSP)

---

## 📊 Résumé des APIs

### APIs Critiques
1. **Electron** (app, BrowserWindow, ipcMain/Renderer, dialog)
2. **Node.js fs** (lecture/écriture de fichiers)
3. **JSZip** (manipulation des archives DOCX/XLSX)
4. **child_process** (exécution de LibreOffice)

### APIs Secondaires
- path, os, crypto (utilitaires Node.js)
- pako (compression via JSZip)
- readable-stream (streams)

### APIs Externes
- LibreOffice CLI (optionnel)
- macOS system APIs (via Electron)

---

**Dernière Mise à Jour** : 2025-11-04
**Version** : 1.0.0