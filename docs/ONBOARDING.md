# Guida per nuovi clienti

Percorso completo per chi riceve il prodotto per la prima volta: dal download alla licenza,
dallo scaffolding di un grafo (su un progetto già avviato o su uno ancora da iniziare) all'uso
quotidiano da CLI o da un agente via MCP. Ogni sezione è autosufficiente — se conosci già il
prodotto, salta a quella che ti serve.

> Questa guida presuppone il **binario compilato**, scaricato dopo l'acquisto (il caso di ogni
> cliente). Se invece hai accesso al repository sorgente, vedi [`CONTRIBUTING.md`](../CONTRIBUTING.md)
> — percorso diverso, non per i clienti del prodotto.

## Mappa della guida

```text
 1. Acquisto/installazione ──▶ 2. Licenza ──▶ 3. Hai già un progetto?
                                              │
                          ┌───────────────────┴───────────────────┐
                          │ Sì, esiste già codice/documenti        │ No, parto da zero
                          ▼                                        ▼
                4. Innesta il grafo                        4. Crea il grafo da zero
                   nel progetto esistente                     e fai partire il progetto
                          │                                        │
                          └───────────────────┬────────────────────┘
                                               ▼
                                   5. Uso quotidiano (CLI/MCP)
                                               ▼
                                   6. Comandi git successivi
```

## 1. Acquisto e installazione

Il prodotto **non** si installa via `pip`/PyPI pubblico: il codice non è pubblicato, licenza
proprietaria regolata da [`EULA.md`](../EULA.md). Prezzo: **10€/mese per postazione** (per
macchina attivata), abbonamento — non una licenza perpetua.

1. **Acquista** su [md-drd-licensing.vercel.app](https://md-drd-licensing.vercel.app) (Stripe
   Checkout, abbonamento mensile; imposta la quantità sul numero di postazioni che ti servono).
   Al completamento del pagamento vieni reindirizzato a una pagina di consegna che mostra, per la
   tua piattaforma rilevata automaticamente, il link di download e il comando di attivazione — non
   serve altro contatto con il venditore.
2. **Scarica** il binario per la tua piattaforma (Windows/macOS/Linux) dalla stessa pagina, oppure
   direttamente dalle [release del repo pubblico `md-drd`](https://github.com/manuel-albanesee/md-drd/releases/latest)
   in qualunque momento (pubbliche, nessun account richiesto):

   | Eseguibile | A cosa serve |
   |---|---|
   | `md-drd` | CLI — comandi da terminale, script, CI |
   | `md-drd-mcp` | Server MCP — tool nativi per Claude Code, Cursor o un client MCP |

3. Rendili eseguibili e raggiungibili da terminale (su Linux/macOS: `chmod +x md-drd` e spostali
   in una cartella nel tuo `PATH`, es. `/usr/local/bin`; su Windows: aggiungi la cartella al
   `PATH` o richiama l'eseguibile col percorso completo).
4. Verifica:

   ```bash
   md-drd --version
   ```

Senza una licenza valida per la macchina corrente, `md-drd --version` funziona ma **ogni altro
comando viene rifiutato** — è normale, il prossimo passo è attivare la licenza.

## 2. Attivazione della licenza

La verifica a runtime resta **sempre interamente offline** (`md-drd validate`/`init`/... non
contattano mai alcun server): quello che è cambiato è solo *come* arriva il file di licenza sulla
tua macchina la prima volta. Due percorsi:

### 2a. Attivazione automatica (il caso normale)

```text
┌──────────┐  activation token  ┌──────────────────┐  md-drd license activate   ┌──────────────┐
│ Pagamento│ ──────────────────▶│  Pagina di        │ ───────────token+fpr─────▶│ md-drd-       │
│ Stripe   │                    │  consegna          │                            │ licensing     │
└──────────┘                    └──────────────────┘                            └──────┬───────┘
                                                                        license.json firmato
                                                                                          │
                                                               scritto in ~/.md-drd/  ◀───┘
                                                               license.json, riverificato subito
```

1. Dopo il pagamento (§1), la pagina di consegna mostra un **activation token** e il comando da
   lanciare:

   ```bash
   md-drd license activate --token <token>
   ```

2. Il CLI calcola la fingerprint di questa macchina, contatta l'unico endpoint di rete di tutto il
   toolkit (`md-drd-licensing`, provisioning una tantum — non un controllo a ogni uso), riceve il
   file di licenza firmato (Ed25519), lo scrive in `~/.md-drd/license.json` e lo riverifica subito
   offline. Da questo momento in poi nessuna rete è più coinvolta.
3. Conserva il token: lo stesso comando su una macchina diversa attiva una nuova postazione, fino
   al numero acquistato (`--url` esiste solo per puntare a un endpoint di test/staging, non serve
   in uso normale).
4. Verifica lo stato della licenza installata in qualunque momento:

   ```bash
   md-drd license info
   ```

Essendo un abbonamento, la scadenza si allunga da sola a ogni rinnovo mensile pagato — non serve
riattivare ogni mese. Se l'abbonamento viene cancellato o un pagamento fallisce, la licenza non
viene revocata attivamente: smette semplicemente di allungarsi e scade alla fine dell'ultimo
periodo pagato (nessuna sorpresa a metà mese).

### 2b. Attivazione manuale (solo CI su runner effimeri)

Il percorso automatico richiede una fingerprint macchina stabile: non funziona su runner CI
effimeri (GitHub-hosted, CircleCI cloud, una VM ricreata a ogni job). Per quel caso — e solo per
quello, cfr. `docs/AGENT-INTEGRATION.md` §CI — resta il flusso manuale via email:

1. Ottieni l'identificativo della macchina (o del runner):

   ```bash
   md-drd license fingerprint
   ```

2. Invia la stringa restituita al venditore via email. È l'unico comando applicativo disponibile
   **senza** licenza.
3. Il venditore emette a mano un file di licenza firmato (`scripts/issue_license.py`, lato
   venditore) che autorizza quella fingerprint, e te lo restituisce.
4. Posizionalo come `~/.md-drd/license.json`, oppure indicane il percorso con la variabile
   d'ambiente `MD_DRD_LICENSE_FILE` (utile per non toccare la home, o in CI):

   ```bash
   export MD_DRD_LICENSE_FILE=/percorso/al/license.json
   ```

Nota bene, in entrambi i percorsi: **nessun periodo di tolleranza**. Alla scadenza, o su una
macchina non in elenco, il binario torna a rifiutare tutto tranne `license fingerprint`. Dettagli
contrattuali in [`EULA.md`](../EULA.md).

## 3. Hai già un progetto avviato, o parti da zero?

Il prodotto funziona allo stesso modo in entrambi i casi — cambia solo **da dove nasce il
grafo**. Scegli il percorso:

- **Progetto già avviato** (codice esistente, magari con requisiti sparsi in Word/Confluence/un
  backlog) → [§4a](#4a-progetto-già-avviato-innesta-il-grafo).
- **Progetto ancora da avviare** (nessun codice, nessun documento formale) → [§4b](#4b-progetto-ancora-da-avviare-crea-il-grafo-da-zero).

In entrambi i casi il grafo che nasce è lo stesso oggetto: un file `graph.yaml` conforme a
`MD-DRD-SPEC-001` (bisogno → requisito → architettura → work package → test), validato dagli
stessi 73 controlli.

### 4a. Progetto già avviato: innesta il grafo

1. **Scegli dove vive il grafo** dentro il repository esistente — una cartella dedicata, non
   mescolata al codice applicativo, es. `requisiti/` o `md-drd/`:

   ```bash
   cd /percorso/del/tuo/progetto        # repo git già esistente
   md-drd init requisiti/ --project "Nome Progetto" --code ACME
   ```

   `--profile minimal` (default) genera lo scheletro essenziale, già conforme; `--profile full`
   popola tutte le 22 collezioni della specifica se vuoi partire da un esempio più ricco da
   potare. `--force` sovrascrive file già presenti — utile solo se stai rigenerando da zero.

2. **Se hai già requisiti in un documento** (analisi funzionale, verbale, backlog) non
   trascriverli a mano nello YAML:
   - un template `.tex` FSD/TSD → `md-drd import-tex --doc D1=Analisi.tex --out frammento.yaml`
     estrae documenti/segmenti/punti aperti da fondere nel grafo;
   - qualunque altro formato (Word, Confluence, backlog libero) → se il tuo venditore ti ha
     fornito anche la skill di authoring per un agente di coding, quella guida l'estrazione e
     itera `validate`/`fix-plan` automaticamente fino a zero difetti bloccanti (vedi
     [Guida all'integrazione con agenti di coding](AGENT-INTEGRATION.md));
   - una volta congelato un documento sorgente nel grafo, `md-drd verify-sources` ti avvisa se
     qualcuno lo modifica senza rifare la derivazione — non aspettare che sia un revisore a
     accorgersene.

3. **Valida e correggi** finché non esce pulito (vedi [§5](#5-uso-quotidiano)).

### 4b. Progetto ancora da avviare: crea il grafo da zero

Qui il grafo **è** il punto di partenza del progetto: prima ancora del codice.

```bash
mkdir progetto-nuovo && cd progetto-nuovo
md-drd init . --project "Nome Progetto" --code ACME --with-hooks
md-drd validate graph.yaml -v
```

`md-drd init` genera lo scheletro più piccolo già conforme (uno stakeholder per categoria, un
bisogno, un requisito, un work package, un test case) — un punto di partenza reale, non un file
vuoto da riempire alla cieca. `--with-hooks` scrive in più gli hook git nativi che tengono il
grafo conforme fin dal primo commit (vedi [§6](#6-comandi-git-successivi)); se preferisci
aggiungerli in un secondo momento, `md-drd hooks-init` fa la stessa cosa su un progetto già
inizializzato. Da lì:

1. Arricchisci il grafo (bisogni → requisiti → architettura → work package → test), validando a
   ogni passo — vedi la [guida "primi 15 minuti"](QUICKSTART.md) per il giro completo comando per
   comando.
2. Quando un work package è pronto per essere implementato, genera il suo brief invece di far
   leggere l'intero grafo a chi scrive il codice:

   ```bash
   md-drd brief graph.yaml WP-001 --out WP-001-brief.md
   ```

3. Solo a questo punto **inizializza il repository di codice** (`git init`) e comincia a
   implementare citando gli id del grafo pertinenti — vedi [§6](#6-comandi-git-successivi).

## 5. Uso quotidiano

Cinque comandi coprono la quasi totalità del lavoro giornaliero:

| Comando | A cosa serve |
|---|---|
| `md-drd validate graph.yaml -v` | schema + 73 controlli su 10 gate; primo comando da lanciare su qualunque grafo |
| `md-drd fix-plan graph.yaml` | difetti raggruppati per controllo, bloccanti prima, con remediation già inclusa |
| `md-drd brief graph.yaml WP-005` | brief autosufficiente per implementare un work package |
| `md-drd project graph.yaml --out out/` | proietta RTM, report di conformità, registro di governance, backlog, ReqIF |
| `md-drd rules --check G3.2 --detailed` | motivazione, remediation ed esempio di un singolo controllo |

Ogni comando accetta `--format json` (un solo oggetto JSON su stdout) e restituisce un codice di
uscita stabile: `0` conforme, `1` non conforme, `2` schema invalido, `3` uso errato, `4` input
invalido, `5` errore interno. Utile per script e CI, non solo per un agente. Riferimento completo
di ogni comando e flag: [`docs/CLI.md`](CLI.md).

### CLI o server MCP?

Stesso motore, stesso contratto dati, due modi di chiamarlo:

| | CLI (`md-drd`) | Server MCP (`md-drd-mcp`) |
|---|---|---|
| Quando | terminale, script, CI | Claude Code, Cursor, o qualunque client MCP |
| Come chiama i comandi | shell-out, un comando per invocazione | tool nativi (`validate`, `fix_plan`, `brief`, `trace_code`, `coverage_diff`, `verify_sources`) |
| Output | identico: `--format json` ↔ risposta del tool | stesso `output_format_version`, stesso `exit_code` |
| Setup | nessuno, solo l'eseguibile | un grafo di default all'avvio |

Configura il server MCP passandogli il grafo di default all'avvio:

```bash
# Claude Code
claude mcp add md-drd -- md-drd-mcp --graph /percorso/assoluto/a/graph.yaml

# Cursor: .cursor/mcp.json
{
  "mcpServers": {
    "md-drd": {
      "command": "md-drd-mcp",
      "env": { "MD_DRD_GRAPH_PATH": "/percorso/assoluto/a/graph.yaml" }
    }
  }
}
```

Se `md-drd-mcp` non è nel `PATH`, usa il percorso completo dell'eseguibile scaricato al punto 1.
Dettagli (tool esposti, risorse, troncamento di elenchi grandi): [`docs/MCP.md`](MCP.md).

**Non devi scegliere una volta per tutte**: una persona può lanciare `md-drd fix-plan` a mano dal
terminale mentre lo stesso grafo, nello stesso momento, è servito via MCP a un agente in un altro
progetto — sono due letture dello stesso file, non due prodotti diversi.

## 6. Comandi git successivi

Il grafo è testo (YAML): vive nel controllo di versione come qualunque altro file sorgente.

```bash
git add requisiti/graph.yaml          # o dove l'hai messo (§4a/§4b)
git commit -m "requisiti: baseline iniziale MD-DRD"
```

Convenzioni utili, non obbligatorie ma coerenti con come il toolkit è pensato per essere usato
da un agente:

- **Cita gli id del grafo** nei commit e nel codice (`WP-009`, `REQ-042`, ...): è lo stesso
  formato libero già usato dentro il grafo per ADR e requisiti, e `md-drd trace-code` lo legge
  per la tracciabilità inversa.
- **Prima di aprire una PR**, verifica che le citazioni nel codice puntino a id reali e non ad
  assunzioni superate:

  ```bash
  git diff --name-only origin/main...HEAD > cambiati.txt
  md-drd trace-code graph.yaml --code-dir . --files cambiati.txt
  ```

- **I documenti proiettati** (`out/`, generati da `md-drd project`) sono derivati, non scritti a
  mano: si rigenerano identici a ogni run dallo stesso grafo. Nella maggior parte dei casi
  conviene **escluderli dal repository** (`.gitignore`) e rigenerarli in CI o al bisogno; versionali
  solo se un pubblico esterno (stakeholder non tecnico) deve poterli sfogliare da git senza
  rilanciare `project`.
- **Hook git nativi sulla tua macchina**, se non hai (ancora) una CI:

  ```bash
  md-drd hooks-init --graph requisiti/graph.yaml
  git config core.hooksPath .githooks       # comando stampato da hooks-init stesso
  ```

  Scrive `.githooks/pre-push` (blocca il push se il grafo non è conforme) e
  `.githooks/post-merge` (avvisa dopo un pull — git non offre un hook "prima" di un pull). Un
  hook locale resta bypassabile (`git push --no-verify`) e non sostituisce un gate lato server:
  è una rete di sicurezza, non un controllo imposto. Fatto in un colpo solo con `md-drd init
  --with-hooks` in fase di scaffold (§4b).
- **In CI**, il codice di uscita decide l'esito della build senza bisogno di fare parsing
  dell'output. Due percorsi diversi:
  - **Validi il grafo con il toolkit installato da sorgente** (nessuna licenza richiesta): un
    workflow riusabile per GitHub Actions è già pronto,

    ```yaml
    jobs:
      validate:
        uses: manuel-albanesee/md-drd-toolkit/.github/workflows/validate-graph.yml@master
        with:
          graph-path: requisiti/graph.yaml
        secrets:
          toolkit-token: ${{ secrets.MD_DRD_TOOLKIT_TOKEN }}
    ```

  - **Fai girare il binario licenziato sulla tua CI** (un runner self-hosted persistente, non
    effimero — la licenza autorizza fingerprint macchina fisse): non serve scriverlo a mano,

    ```bash
    md-drd ci-init --graph requisiti/graph.yaml
    ```

    genera `.github/workflows/md-drd.yml` già pronto per `runs-on: self-hosted`, con lo step
    `md-drd validate` incluso e nessun riferimento al sorgente privato del toolkit. Commettilo
    così com'è.

  Dettagli su entrambi i percorsi (incluso il gap noto sui runner effimeri) in
  [`docs/AGENT-INTEGRATION.md`](AGENT-INTEGRATION.md#ci).

## 7. Domande frequenti e problemi comuni

- **"Comando rifiutato" su tutto tranne `license fingerprint`** → nessuna licenza installata, o
  installata nel posto sbagliato. Verifica `md-drd license info` e la variabile
  `MD_DRD_LICENSE_FILE` se non usi il percorso di default.
- **Licenza scaduta** → per design nessun periodo di tolleranza (verifica offline, niente server
  da contattare per un'estensione temporanea). Se l'abbonamento è ancora attivo ma la licenza
  locale risulta scaduta (rinnovo non ancora propagato), rilancia `md-drd license activate
  --token <lo-stesso-token>`: è idempotente, non consuma una postazione aggiuntiva. Se hai
  attivato via email (§2b, solo CI), serve invece una riemissione manuale del venditore.
- **Macchina non autorizzata / ho spostato il progetto su un'altra macchina** → la fingerprint
  cambia, la vecchia licenza non è valida sulla nuova macchina. Rilancia `md-drd license activate
  --token <token>` sulla nuova macchina: attiva una nuova postazione nei limiti di quelle
  acquistate (oltre quel limite l'endpoint rifiuta con "postazioni esaurite" — serve aumentare la
  quantità dell'abbonamento). Per licenze emesse via email (§2b) serve invece una riemissione
  manuale.
- **`md-drd validate` non trova lo schema** → non capita col binario compilato (schema
  incorporato); se vedi questo errore controlla di non aver puntato `--schema` a un percorso
  sbagliato.
- **Voglio usare il prodotto anche da un agente MCP e da terminale sullo stesso grafo** → nessun
  conflitto, vedi la nota a fondo [§5](#5-uso-quotidiano).

## Prossimi passi

- **Comando per comando, su fixture pronte** → [`docs/QUICKSTART.md`](QUICKSTART.md).
- **Un agente di coding scrive/consuma il grafo** → [`docs/AGENT-INTEGRATION.md`](AGENT-INTEGRATION.md).
- **Configurazione completa del server MCP** → [`docs/MCP.md`](MCP.md).
- **Ogni comando e flag della CLI** → [`docs/CLI.md`](CLI.md).
- **Perché un controllo scatta, e come correggerlo** → [`docs/RULEBOOK.md`](RULEBOOK.md).
- **Struttura del grafo, entità e campi** → [`docs/SCHEMA.md`](SCHEMA.md).
- **Termini contrattuali della licenza** → [`EULA.md`](../EULA.md).
