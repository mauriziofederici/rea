# REA — configurazione del fork mauriziofederici

Questo repository è un **fork** del progetto originale [morluto/rea](https://github.com/morluto/rea), distribuito con licenza MIT. Il fork salva il codice su GitHub, ma **salvare il codice non equivale a installare REA** su un computer o a collegarlo automaticamente a un agente AI.

La guida ufficiale in italiano è [docs/readme/README_it.md](docs/readme/README_it.md); le istruzioni tecniche aggiornate sono in [docs/installation.md](docs/installation.md).

## 1. Utilizzo da GitHub Codespaces (compilazione del fork)

La directory [`.devcontainer`](.devcontainer/devcontainer.json) prepara un ambiente Linux con Node.js 24. L'ambiente viene creato solo su tua iniziativa.

1. Nel repository GitHub apri **Code → Codespaces → Create codespace on main**.
2. Attendi che il contenitore si avvii. La configurazione usa l'immagine Microsoft **`24-bookworm`** e non esegue automaticamente `npm ci`, per evitare che un eventuale errore di compilazione blocchi Codespaces.
3. Apri il terminale e installa/compila **il tuo fork**:
   ```bash
   node --version
   npm ci
   npm run build:cached
   node scripts/rea.mjs --help
   ```
4. Se un Codespace esistente è in **recovery mode**, nel terminale esegui `git pull --ff-only`. Poi apri la palette con `Ctrl+Shift+P`, scegli **Codespaces: Rebuild Container** e attendi. Se il problema continua, clicca **View Creation Log** e leggi l'errore esatto.
5. Per analizzare un progetto per il quale disponi delle autorizzazioni necessarie, carica i file di prova nel Codespace ed esegui un comando documentato da REA. Esempio per una directory JavaScript/Electron:
   ```bash
   node scripts/rea.mjs analyze-javascript-application /percorso/assoluto/app --json
   ```

**Limiti:** Codespaces può comportare costi secondo il piano GitHub e non include necessariamente Hopper, Ghidra, IDA, un browser interattivo o un'applicazione Windows da analizzare. Il server MCP non si collega automaticamente al ChatGPT di questa conversazione. Per l'integrazione va configurato un client AI compatibile nell'ambiente dove REA viene eseguito.

## 2. Utilizzo dal tuo PC Windows (pacchetto ufficiale npm)

1. Installa Node.js compatibile con REA: **22.19+**, **24.11+** oppure **26+**. Non tutte le versioni intermedie di Node.js sono supportate.
2. Apri **PowerShell** e verifica:
   ```powershell
   node --version
   npm --version
   ```
3. Esegui il setup ufficiale (può scaricare il pacchetto npm):
   ```powershell
   npx.cmd rea-agents@latest setup
   ```
4. Seleziona soltanto gli agenti AI che intendi usare (per esempio Codex, Claude Code o Cursor), controlla le modifiche proposte e approva quando richiesto.
5. Riavvia il client AI selezionato e controlla lo stato:
   ```powershell
   npx.cmd -y rea-agents@latest doctor --json
   ```

Il comando `doctor` può segnalare strumenti di analisi nativa assenti senza che questo impedisca l'analisi statica JavaScript. Per analizzare eseguibili nativi occorrono inoltre gli strumenti indicati nella documentazione ufficiale.

**Attenzione:** questa modalità installa il pacchetto **REA ufficiale da npm**, non il codice personalizzato contenuto nel tuo fork.

## 3. Compilare esattamente il tuo fork sul computer

Se desideri utilizzare il sorgente salvato qui, invece del pacchetto npm pubblicato:

```powershell
git clone https://github.com/mauriziofederici/rea.git
cd rea
npm ci
npm run build:cached
node scripts/rea.mjs --help
```

Questi comandi scaricano le dipendenze ed eseguono la build **in locale**. Non registrano automaticamente il server MCP nel tuo client AI: per la configurazione di un eseguibile sorgente consulta [docs/installation.md](docs/installation.md) e [docs/client-configuration.md](docs/client-configuration.md).

## 4. Verifica automatizzata su GitHub

Nel menu **Actions**, scegli **REA — Verifica installazione npm** e premi **Run workflow**. È un test manuale: installa temporaneamente il pacchetto ufficiale in un runner GitHub e verifica che il comando `rea` si avvii. **Non installa REA sul tuo PC né crea un servizio sempre attivo.**

## 5. Sicurezza e manutenzione

- Usa solo file e applicazioni che hai diritto di analizzare; evita copie di codice proprietario non autorizzate.
- Non inserire password, token, dati clienti o altri segreti nei repository pubblici, nei log di Actions o negli artefatti.
- Per file sospetti usa un ambiente isolato. REA non equivale a una sandbox di sicurezza.
- Il fork è distinto dal progetto originale: per incorporare gli aggiornamenti ufficiali usa **Sync fork** su GitHub, poi verifica eventuali conflitti.
- Il repository originale contiene numerosi workflow CI; prima di attivare Actions o Codespaces controlla limiti, permessi e costi del tuo account GitHub.

**Stato iniziale:** questa configurazione aggiunge un ambiente predisposto e un controllo manuale. Non attesta che l'installazione sia già stata eseguita o testata sul tuo computer.

## 6. MCP in VS Code / GitHub Codespaces: usare il fork compilato

La configurazione [`.vscode/mcp.json`](.vscode/mcp.json) registra un server MCP locale di nome **rea** per il workspace VS Code. Esegue direttamente il codice del fork già compilato tramite `node scripts/rea.mjs --mcp` nella directory del progetto; **non scarica una seconda copia da npm**.

1. Dopo aver eseguito `npm ci`, `npm run build:cached` e `node scripts/rea.mjs --help`, aggiorna il workspace con `git pull --ff-only origin main`.
2. In VS Code apri la palette `Ctrl+Shift+P` e cerca **MCP: List Servers**.
3. Seleziona **rea** e, se proposto, **Start Server**; controlla che lo stato sia **Running**. Per problemi usa **Show Output**.
4. Apri la chat di VS Code con un agente dotato di accesso MCP (per esempio GitHub Copilot in modalità agente), abilita gli strumenti REA e prova la richiesta: «Usa il server MCP rea per elencare le sue capacità e indicare quali analisi statiche sono disponibili in questo Codespace. Non analizzare file esterni e non modificare il repository».

Se REA non parte, controlla i log del server e ricompila con `npm ci && npm run build:cached`. Dopo una ricostruzione del container potrebbero mancare gli artefatti compilati. Se il client segnala **più di 128 strumenti abilitati**, limita gli strumenti REA disponibili per la singola chat oppure abilita la gestione dei gruppi virtuali di VS Code (se disponibile).

**Sicurezza:** il server MCP dispone dei permessi del processo nell'ambiente Codespaces. Non assegnargli accesso a segreti, dati clienti o file non autorizzati. La configurazione MCP del workspace non collega automaticamente REA a ChatGPT in questa conversazione e non garantisce l'accesso a motori nativi come Ghidra.
