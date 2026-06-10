In un progetto **SPFx v1.22.2** (che utilizza il nuovo motore di build Heft), il file `config/rig.json` è il punto di collegamento a un pacchetto NPM esterno, chiamato "rig", che contiene tutte le regole di build predefinite per SharePoint Framework.

## 🤔 A Cosa Serve: Il Concetto di "Rig"

Un "Rig" (dall'inglese "impalcatura") è un pacchetto NPM che fornisce una configurazione di build standardizzata, riutilizzabile e manutenuta centralmente. In precedenza, questa logica era contenuta nel file `gulpfile.js`, il che portava a duplicazione di codice, potenziali conflitti e maggiore complessità negli aggiornamenti. Con il nuovo approccio, il progetto "eredita" l'intera pipeline di build dalla rig, eliminando la necessità di configurazioni manuali complesse e rendendo gli aggiornamenti delle dipendenze più puliti e sicuri.

## ⚙️ Cosa Posso Fare: Contenuto e Operatività

Il file `rig.json` è tipicamente molto semplice, con pochi campi chiave che puntano alla configurazione esterna.

### Contenuto Tipico di `rig.json`
Ecco l'aspetto standard di questo file in un progetto SPFx v1.22.2:

```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/rig-package/rig.schema.json",
  "rigPackageName": "@microsoft/spfx-web-build-rig"
}
```
*   **`rigPackageName` (Obbligatorio)**: Specifica il nome del pacchetto NPM che contiene la configurazione di build da ereditare. Per SPFx, il pacchetto è `@microsoft/spfx-web-build-rig`.
*   **`rigProfile` (Opzionale)**: Permette di selezionare un profilo di configurazione specifico all'interno del pacchetto. Se omesso, viene utilizzato il profilo `"default"`.

Il file `typescript.json`, spesso presente nella stessa cartella `config/`, lavora in sinergia con il `rig.json`, estendendo la configurazione di TypeScript fornita dalla rig e aggiungendo eventuali personalizzazioni locali (come la copia di asset statici).

### Come Opera: Heft in Azione
Quando esegui un comando come `npm run build`, il processo che si attiva è il seguente:

1.  **Lettura della Configurazione**: Heft, il motore di build, inizia leggendo il file `rig.json` del tuo progetto.
2.  **Caricamento della "Rig"**: Individua e carica il pacchetto NPM specificato, che in SPFx v1.22.2 è `@microsoft/spfx-web-build-rig`.
3.  **Esecuzione della Pipeline**: Utilizza le definizioni contenute nella rig per eseguire la pipeline di build completa: compilazione TypeScript, bundling con Webpack, compilazione SCSS, elaborazione delle localizzazioni, generazione dei manifest e creazione del pacchetto `.sppkg`.

## 🛠️ Cosa Posso Fare: Personalizzare il Comportamento di Build

Anche se si eredita una configurazione standard, ci sono diversi modi per personalizzarla:

*   **Configurazione Locale**: Il progetto mantiene comunque i propri file di configurazione (ad esempio `config/serve.json`, `config/package-solution.json`). Le impostazioni locali possono sovrascrivere quelle ereditate dalla rig, offrendo un primo livello di personalizzazione.
*   **Uso di Plugin Heft**: Heft include plugin integrati per eseguire compiti comuni come copiare file, cancellare directory o impostare variabili d'ambiente. Per abilitarli, è sufficiente creare un file `heft.json` locale e configurarli secondo le proprie esigenze.
*   **Script Personalizzati**: Quando le funzionalità dei plugin integrati non sono sufficienti, si può utilizzare il "Run Script Plugin" per eseguire script Node.js arbitrari durante la pipeline di build.
*   **Controllo Avanzato su Webpack (Eject)**: Per scenari che richiedono il massimo controllo, è possibile "eiettare" la configurazione di Webpack, ottenendo un file di configurazione completo da modificare a piacimento.
