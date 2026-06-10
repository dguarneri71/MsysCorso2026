## Cos'è Heft e perché Microsoft lo ha scelto

Heft (il cui nome completo è **Rush Stack Heft**) è un orchestratore di build moderno, configurabile a file JSON, sviluppato da Microsoft all'interno dell'ecosistema **Rush Stack**. Con il rilascio di **SPFx v1.22 (dicembre 2025)**, Microsoft lo ha reso il sistema di build predefinito per tutti i nuovi progetti al posto del precedente toolchain basato su Gulp.

La transizione risponde a diverse esigenze accumulate nel tempo:

*   **Debito tecnico** – la vecchia toolchain (gulp-core-build) non riceveva manutenzione da anni, accumulando vulnerabilità e dipendenze obsolete segnalate da `npm audit`.
*   **"Scatola nera"** – molti passaggi del build erano oscurati, rendendo difficile la personalizzazione e la manutenzione su progetti enterprise di grandi dimensioni.
*   **Allineamento interno** – Microsoft stessa utilizzava Heft internamente da tempo; il passaggio unifica l'esperienza di sviluppo tra Microsoft, ISV e sviluppatori indipendenti, accelerando il delivery di nuove funzionalità.

> **Nota importante**: Il bundling sottostante rimane **Webpack**. Quello che cambia è l'orchestratore che gestisce la sequenza delle operazioni di build (compilazione TypeScript, linting, test, bundle e packaging).

---

## Come Heft può esserti utile nei tuoi progetti SPFx

### 1. Performance e scalabilità

Heft è stato progettato per gestire progetti enterprise su larga scala. A differenza di Gulp, che esegue i task prevalentemente in sequenza, Heft sfrutta:

*   **Esecuzione parallela** dei task dove possibile
*   **Cache intelligente** e build incrementali
*   **Compilazione in watch mode** che rileva selettivamente i file modificati

**Risultato**: feedback loop più rapidi e produttività aumentata.

### 2. Manutenibilità del progetto

Con Heft, gran parte della configurazione è spostata in **file JSON**, non più in `gulpfile.js`. I nuovi progetti SPFx utilizzano un **rig** (un pacchetto npm chiamato `@microsoft/spfx-web-build-rig`) che contiene la configurazione predefinita di tutte le fasi di build. Questo riduce drasticamente il boilerplate e rende gli upgrade meno dolorosi.

### 3. Sicurezza migliorata

Microsoft ha risolto tutte le vulnerabilità note segnalate da `npm audit` sia nello Yeoman generator che nei progetti scaffoldati con v1.22. Ora, dopo uno scaffolding, `npm audit` non riporta più vulnerabilità ad alta gravità.

### 4. Flessibilità configurabile

Heft offre un'architettura **basata su plugin** che rende le personalizzazioni molto più pulite rispetto alle vecchie `gulpfile.js`:

*   **Plugin built-in**: copy-files, delete-files, run-script, set-env-var
*   **Run Script Plugin**: per eseguire codice JavaScript/TypeScript arbitrario durante la build
*   **Webpack Patch Plugin**: per modificare la configurazione di Webpack senza dover "ejectare" completamente la configurazione

Ecco un pratico esempio per copiare un file di licenza nella cartella della soluzione SharePoint:

```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/heft/v0/heft.schema.json",
  "extends": "@microsoft/spfx-web-build-rig/profiles/default/config/heft.json",
  "phases": {
    "package-solution": {
      "taskPlugins": [
        {
          "pluginName": "copy-files",
          "options": {
            "copyOperations": [
              {
                "sourcePath": "./sharepoint/assets/LICENSE.md",
                "destinationFolders": ["./sharepoint/solution"]
              }
            ]
          }
        }
      ]
    }
  }
}
```


---

## Trucchi e best practice utili da sapere

### 1. Usa npm scripts per non memorizzare i flag di Heft

Heft ha diversi flag (`--clean`, `--production`, `--serve`). Invece di memorizzarli, utilizza npm scripts personalizzati nel tuo `package.json`:

```json
{
  "scripts": {
    "package": "npm run clean && heft package-solution",
    "package:prod": "npm run clean && heft test --production && heft package-solution --production",
    "clean": "heft clean",
    "start:clean": "heft start --clean"
  }
}
```

**Vantaggi**: comandi brevi, autodocumentati, e facilmente integrabili in pipeline CI/CD.

### 2. Crea una `heft.json` locale sovrascrivendo il rig

Non devi modificare il pacchetto `@microsoft/spfx-web-build-rig`. Crea un file `./config/heft.json` locale che **estende** la configurazione del rig:

```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/heft/v0/heft.schema.json",
  "extends": "@microsoft/spfx-web-build-rig/profiles/default/config/heft.json"
}
```

**Importante**: non eliminare il file `rig.json` perché altre parti di Heft ne hanno ancora bisogno.

### 3. Installa Heft globalmente (opzionale ma comodo)

Puoi installare il pacchetto `@rushstack/heft` globalmente sul tuo sistema:

```bash
npm install @rushstack/heft --global
```

A differenza di Gulp, che richiedeva il CLI globale, con Heft puoi scegliere. Se installato globalmente, puoi eseguire `heft start` da qualsiasi progetto senza usare `npx` o script npm. Tuttavia, lo strumento funziona comunque correttamente se eseguito tramite npm scripts.

### 4. Utilizza patch Webpack invece di ejectare

Se hai bisogno di modificare la configurazione di Webpack (es. aggiungere un plugin come Bundle Analyzer), utilizza il **Webpack Patch Plugin** incluso in SPFx. Crea:

*   `./config/webpack-patch/webpack-bundle-analyzer.js` – il tuo script di modifica
*   `./config/webpack-patch.json` – per registrare lo script

**Questo approccio preserva la compatibilità con i futuri aggiornamenti di SPFx**, mentre "ejectare" Webpack ti lascerebbe con una configurazione statica e disallineata.

### 5. Attenzione alla modalità watch: il linting non è supportato

Un comportamento importante da conoscere: **il linting non è attualmente supportato in watch mode**. Quando esegui `heft start` o `heft build-watch`, il task di linting non viene eseguito. Per testare il linting, devi eseguire un build esplicito senza watch.

**heft start vs heft build: Quale usare per testare il linting?**

La scelta tra i due comandi dipende dall'effetto desiderato:

heft start (Watch Mode): Avvia il server di sviluppo e rimane in esecuzione, eseguendo il linting in modo incrementale sui file che modifichi. Ideale per lo sviluppo attivo, perché ricevi feedback immediato.

heft build (Build Esplicita): Esegue una build completa e singola, includendo il linting su tutti i file del progetto. Utile prima di un commit per una verifica totale, o in ambienti CI.

### 6. Gestisci le dipendenze mancanti con package manager moderni

Se utilizzi **pnpm** (o altri package manager con dependency hoisting limitato), potresti incontrare un errore: `Cannot find type definition file for 'heft-jest'`. La soluzione è semplice: aggiungi esplicitamente il pacchetto mancante:

```bash
pnpm add @types/heft-jest -D
```

Heft si aspetta che tutte le dipendenze siano esplicitamente dichiarate per prevenire il fenomeno delle "phantom dependencies".

### 7. Build delle librerie: usa `--production` per il packaging

Quando lavori con **library components** SPFx, non basta `heft build`. Devi eseguire:

```bash
heft build --production
```

Inoltre, dopo aver creato un link simbolico con `npm link`, ricordati di eseguire almeno **una volta** `heft build` prima di poter importare la libreria in un altro progetto (perché il campo `"main": "lib/index.js"` nel `package.json` deve puntare a un file esistente).

---

## Tabella riassuntiva: Gulp vs Heft

| Caratteristica | Gulp (SPFx v1.0–v1.21) | Heft (SPFx v1.22+) |
|----------------|--------------------------|---------------------|
| **Approccio** | Task scripted | Orchestrazione a fasi |
| **Configurazione** | `gulpfile.js` (JavaScript) | File JSON + rig |
| **Performance** | Lenta su larga scala | Veloce con cache e parallelismo |
| **Type Safety** | Limitata | Forte |
| **Supporto monorepo** | Debole | Built-in |
| **Debugging** | Difficile, errori oscuri | Log chiari e tracciabili |
| **Plugin** | Ad-hoc | Architettura plugin nativa |



---

## Comandi principali di Heft che devi conoscere

| Azione | Comando Heft | Equivalente npm script |
|--------|--------------|------------------------|
| Avvia server locale | `heft start` | `npm start` |
| Build di produzione | `heft build --production` | `npm run build` |
| Package .sppkg | `heft package-solution --production` | – |
| Pulizia build | `heft clean` | – |
| Build incrementale + watch | `heft build-watch --serve` | – |

Nota: gli npm script `npm start` e `npm run build` sono preconfigurati nello scaffold SPFx, quindi il tuo flusso di lavoro quotidiano rimane invariato.

---

## Problemi comuni e soluzioni rapide

| Problema | Soluzione |
|----------|-----------|
| Web part non compare nello Workbench | Verifica di aver incluso la **debug query string** nei parametri URL (es. `?debugManifestsFile=...`). Heft la stampa nel terminale dopo l'avvio. |
| Errori di runtime dopo aver riaperto il property pane | Pubblica la pagina o chiudi il property pane prima di ricaricare – è un problema noto della v1.22 legato al nuovo toolchain. |
| Build fallisce con `Cannot find module` | Esegui `npm install` e verifica che tutte le dipendenze siano esplicitamente dichiarate nel `package.json`. |
| `Cannot find type definition file for 'heft-jest'` | Installa `@types/heft-jest` come dev dependency (`npm install @types/heft-jest --save-dev`). |



---

## Riferimenti utili per approfondire

*   [Documentazione ufficiale della toolchain Heft in SPFx](https://learn.microsoft.com/sharepoint/dev/spfx/toolchain/sharepoint-framework-toolchain-rushstack-heft)
*   [Migrazione da Gulp a Heft – guida passo-passo](https://learn.microsoft.com/sharepoint/dev/spfx/toolchain/migrate-gulptoolchain-hefttoolchain)
*   [Personalizzazione della toolchain con plugin Heft](https://learn.microsoft.com/sharepoint/dev/spfx/toolchain/customize-heft-toolchain-heft-included-plugins)
*   [Personalizzazione di Webpack con Webpack Patch Plugin](https://learn.microsoft.com/sharepoint/dev/spfx/toolchain/customize-heft-toolchain-customize-webpack-config)
*   [Heft official documentation](https://heft.rushstack.io/)
*   [Esempio: copiare file con Copy Files Plugin](https://heft.rushstack.io/pages/plugins/copy-files/)
*   [Esempio: eseguire script con Run Script Plugin](https://heft.rushstack.io/pages/plugins/run-script/)
*   [Heft CLI reference](https://heft.rushstack.io/pages/intro/cli/)

---

Heft rappresenta un significativo passo avanti per l'ecosistema SPFx. Sebbene richieda un piccolo investimento iniziale per familiarizzare con i nuovi concetti (`rig`, fasi, plugin), i benefici in termini di **manutenibilità, performance e flessibilità** lo rendono un upgrade altamente consigliato per tutti i nuovi progetti e per quelli esistenti che intendono modernizzarsi.


Il comando `heft eject-webpack` fa parte del toolchain di build **Heft**, usato in SharePoint Framework (SPFx) e, più in generale, nei progetti che si basano su **Rush Stack**. Il suo scopo principale è estrarre la configurazione di **webpack** dal controllo centralizzato del toolchain, copiandola integralmente all'interno del tuo progetto per darti la possibilità di personalizzarla in modo completo.

Ecco una panoramica dettagliata di cosa fa e delle sue implicazioni.

### Cosa significa "eject" la configurazione?

Per capire il comando, è utile sapere che, in un progetto SPFx standard, la configurazione del webpack è astratta e gestita internamente da Microsoft. Il comando `heft eject-webpack` rompe questa astrazione:

1.  **Copia file e dipendenze**: Trasferisce tutti i file di configurazione nascosti di webpack e le relative dipendenze all'interno della directory del tuo progetto.
2.  **Rimuove le configurazioni standard**: Elimina le configurazioni dei plugin Heft predefiniti dal tuo progetto.
3.  **Aggiunge nuovi file di configurazione**: Crea i file di configurazione di webpack sia per lo sviluppo (`development`) che per la produzione (`production`).
4.  **Aggiunge configurazioni delle task**: Include la configurazione delle task di Heft (con tutte le opzioni) direttamente nel progetto.

In sostanza, l'intera responsabilità della configurazione di webpack passa dal toolchain centralizzato a te, lo sviluppatore. È un'operazione **unidirezionale**: una volta eseguita, non è possibile tornare indietro facilmente senza ripristinare una versione precedente del progetto tramite un sistema di controllo versione (es. Git).

### Cosa comporta e quando usarlo

*   **Implicazioni importanti**: Questa operazione ti dà il **controllo totale** sulla configurazione, ma ciò significa che tu diventi l'unico responsabile della sua manutenzione e del suo corretto funzionamento. Inoltre, Microsoft avverte che **non fornisce supporto ufficiale** per i progetti SPFx che hanno eseguito l'ejection.
*   **Quando è utile**: L'ejection è consigliata solo quando le opzioni di personalizzazione standard (come i plugin integrati) risultano insufficienti per le tue esigenze specifiche.

### Alternative all'ejection

Prima di scegliere questa strada, è sempre consigliabile valutare alternative meno invasive e più supportate.

*   **Plugin integrati di Heft**: Heft dispone di plugin ufficiali per le operazioni più comuni, come la copia di file (`copy-files-plugin`) o la cancellazione di file. Questi plugin possono essere configurati semplicemente modificando un file (`config/heft.json`).
*   **Script personalizzati**: Attraverso il **Run Script Plugin**, puoi eseguire script o codice arbitrario in punti specifici del processo di build.
*   **Webpack Patch Plugin**: Questo plugin consente di fornire uno script che può "mutare" (modificare) la configurazione di webpack prima che venga eseguita, senza dover estrarre completamente la configurazione.

Per farti un'idea, la tabella seguente riassume le differenze tra queste opzioni:

| Approccio | Complessità | Flessibilità | Manutenzione |
| :--- | :--- | :--- | :--- |
| Plugin integrati | Bassa | Limitata | Nessuna (gestita da Microsoft) |
| Script personalizzati | Media | Media | Media (gestita dallo sviluppatore) |
| **Eject** | **Alta** | **Completa** | **Alta (gestita dallo sviluppatore)** |

In conclusione, `heft eject-webpack` è uno strumento molto potente, da usare con cautela, riservandolo ai casi in cui hai bisogno di una personalizzazione estrema che le alternative più semplici non sono in grado di offrire.

Se hai altri dubbi o vuoi approfondire una delle alternative, chiedi pure.
