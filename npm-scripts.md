## Come usare npm scripts con Heft in SPFx

Gli **npm scripts** sono comandi personalizzati definiti nel file `package.json` del tuo progetto. Ti permettono di eseguire istruzioni complesse (come i comandi Heft) con semplici alias come `npm start`, senza dover ricordare flag o percorsi.

### 1. Struttura di base in `package.json`

All'interno del tuo progetto SPFx (generato con v1.22+), troverai già una sezione `"scripts"` simile a questa:

```json
{
  "scripts": {
    "start": "heft start",
    "build": "heft build --production",
    "test": "heft test",
    "clean": "heft clean"
  }
}
```

### 2. Come eseguirli

Apri un terminale nella cartella del progetto e digita:

```bash
npm start            # equivale a "heft start"
npm run build        # equivale a "heft build --production"
npm run test         # equivale a "heft test"
npm run clean        # equivale a "heft clean"
```

> **Nota**: `npm start` e `npm test` sono abbreviati, non richiedono la parola `run`. Per tutti gli altri script (`build`, `clean`, `package` ecc.) devi usare `npm run <nome-script>`.

### 3. Creare script personalizzati (esempi pratici)

Aggiungi nuove righe nella sezione `scripts` del tuo `package.json` per automatizzare operazioni frequenti.

#### Esempio 1 – Package della soluzione SPFx

```json
"package": "heft package-solution --production"
```

Poi esegui:  
```bash
npm run package
```

#### Esempio 2 – Pulizia + rebuild completo

```json
"rebuild": "npm run clean && npm run build"
```

Esegui con:  
```bash
npm run rebuild
```

#### Esempio 3 – Avvio con pulizia preventiva

```json
"start:clean": "heft start --clean"
```

#### Esempio 4 – Build di sviluppo (senza produzione) per debug

```json
"build:dev": "heft build"
```

### 4. Passare argomenti aggiuntivi a Heft tramite npm script

Se il tuo script richiede flag aggiuntivi, puoi passarli usando `--` dopo il comando npm:

```json
"package": "heft package-solution"
```

Eseguendo:  
```bash
npm run package -- --production
```

Il `--` separa i parametri per npm da quelli per lo script. In pratica, viene eseguito `heft package-solution --production`.

### 5. Script predefiniti nello scaffold SPFx v1.22

Quando crei un nuovo progetto con Yeoman, ottieni questi script pronti all'uso:

| Script | Comando Heft | Uso tipico |
|--------|--------------|-------------|
| `start` | `heft start` | Sviluppo con workbench locale |
| `build` | `heft build --production` | Build pronta per il deployment |
| `test` | `heft test` | Esecuzione test unitari |
| `clean` | `heft clean` | Rimozione cartelle `lib/`, `temp/`, `dist/` |

### 6. Vantaggi dell'uso di npm scripts

| Vantaggio | Spiegazione |
|-----------|-------------|
| **Semplicità** | Non ricordi flag complessi (`--production --clean --serve`) |
| **Condivisione** | Tutti i membri del team usano gli stessi comandi |
| **CI/CD** | Facile integrazione con pipeline (Azure DevOps, GitHub Actions) |
| **Composizione** | Puoi concatenare più comandi (`&&`, `||`, `npm run ...`) |

### 7. Esempio avanzato: script per più ambienti

```json
{
  "scripts": {
    "build:dev": "heft build",
    "build:prod": "heft build --production",
    "package:dev": "heft package-solution",
    "package:prod": "heft package-solution --production",
    "deploy:dev": "npm run build:dev && npm run package:dev && echo 'Deploy su tenant di test'",
    "deploy:prod": "npm run build:prod && npm run package:prod && echo 'Deploy su tenant di produzione'"
  }
}
```

Esegui:  
```bash
npm run deploy:dev
```

### 8. Differenza tra eseguire Heft direttamente o tramite npm script

| Esecuzione | Vantaggi | Svantaggi |
|------------|----------|------------|
| `heft start` | Veloce se Heft è installato globalmente | Devi ricordare flag; non standardizzato nel team |
| `npm start` | Standard, documentato, uguale per tutti | Richiede che `heft` sia nel `node_modules` (già incluso da SPFx) |

**Consiglio**: usa sempre `npm start` / `npm run build` nei tutorial e nella documentazione del team. Rendi così il progetto accessibile anche a chi non conosce Heft.

### 9. Verificare gli script disponibili

Per vedere tutti gli script definiti in un progetto:

```bash
npm run
```

(Su alcuni sistemi) oppure guarda direttamente nel `package.json`.

---
