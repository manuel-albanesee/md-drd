# Themis

> **Themis** — il grafo canonico di requisiti (bisogno → requisito → architettura → work package
> → test) definito da **[THEMIS-SPEC-001](docs/THEMIS-SPEC-001.md)** e validato da regole
> deterministiche, non da un altro prompt.

**Questo repository è la vetrina pubblica del prodotto**: documentazione cliente, la specifica che
il toolkit implementa, e i binari compilati scaricabili. Il codice sorgente è proprietario e vive
in un repository privato — non è distribuito né qui né su PyPI (vedi [Licenza](#licenza)).

## Il problema

Una specifica scritta in prosa (Word, Confluence, un lungo file Markdown) non impedisce a un
requisito di restare senza test, a un work package di restare senza requisiti a monte, o a un
agente che ha appena scritto la specifica di essere anche l'unico giudice di quanto sia ben fatta.
Themis sostituisce la prosa con un **grafo esplicito** e lo verifica con **73 controlli
deterministici su 10 gate** (`G0`..`G9`): requisiti orfani, WBS incompleta, lessico non
verificabile, copertura dei test, e altro. Zero difetti bloccanti prima di implementare, non
durante il collaudo.

## A chi serve

- **Team che integrano un agente di coding** (Claude Code, Cursor, ...) e vogliono dargli una
  specifica che l'agente possa leggere, validare e citare senza ambiguità.
- **Chi scrive la specifica** (analista, PM tecnico) e vuole un ciclo genera → valida → correggi
  con remediation azionabile ad ogni difetto.
- **Chi deve dimostrare la conformità** a uno stakeholder non tecnico: matrice di tracciabilità,
  report di conformità e registro di governance si generano dal grafo, non si scrivono a mano.

## Prezzo

**10€/mese per postazione**, abbonamento — nessun vincolo di durata minima, si cancella quando si
vuole (la licenza resta valida fino alla fine dell'ultimo periodo già pagato). Il numero di
postazioni acquistate è la quantità scelta in fase di acquisto.

## Acquisto, download, attivazione

1. **Acquista** su [themis-licensing.vercel.app](https://themis-licensing.vercel.app) (Stripe
   Checkout — al momento in modalità test, vedi nota sotto). Dopo il pagamento, la pagina di
   consegna mostra il link di download per la tua piattaforma e il comando di attivazione.
2. **In alternativa**, scarica in qualunque momento il binario per la tua piattaforma dalla
   [pagina delle release](../../releases/latest):

   | Piattaforma | File |
   |---|---|
   | Windows | `themis-windows-latest.exe` |
   | macOS | `themis-macos-latest` |
   | Linux | `themis-ubuntu-latest` |

   (stesso elenco per `themis-mcp`, il server MCP — vedi [`docs/MCP.md`](docs/MCP.md)).
3. **Attiva** la licenza sulla macchina dove userai Themis:

   ```bash
   themis license activate --token <token-mostrato dopo l'acquisto>
   ```

   Il CLI calcola la fingerprint della macchina, contatta una volta sola il servizio di
   attivazione e scrive il file di licenza firmato. Da quel momento la verifica è **sempre
   offline** — nessun server contattato ad ogni uso del prodotto.

> **Nota:** il sistema di pagamento è al momento in **modalità test di Stripe** — nessun addebito
> reale finché non viene comunicata la disponibilità in produzione.

Senza un file di licenza valido per la macchina corrente, il binario rifiuta di eseguire qualunque
comando applicativo (eccetto `license fingerprint`, sempre disponibile). Dettagli contrattuali in
[`EULA.md`](EULA.md).

→ **[Guida per nuovi clienti](docs/ONBOARDING.md)**: lo stesso flusso passo per passo, più come
innestare il grafo su un progetto già avviato o farlo nascere insieme a uno nuovo, CLI vs server
MCP, e i comandi git successivi.

## Come funziona

> I gate sono codice, non prompt (THEMIS-SPEC-001 §16.2).

Nessun modello linguistico decide se un requisito è ben formato: regole deterministiche
(espressioni regolari sul lessico vietato, calcolo di grafo per gli orfani, PERT/CPM sul piano, la
regola del 100% sulla WBS). Un agente che genera il grafo può quindi essere validato in modo
riproducibile, senza che la stessa istanza che ha scritto il contenuto ne giudichi anche la
conformità.

```bash
themis init progetto/                       # scheletro minimo, già conforme
themis validate progetto/graph.yaml -v      # 73 controlli, 0 = conforme
themis project  progetto/graph.yaml --out out/   # RTM, conformità, governance, backlog, ReqIF
```

## Integrazione con agenti di coding

Codici di uscita stabili (`0` conforme, `1` non conforme, ...), `--format json` su ogni comando
con schema versionato, e ogni difetto già corredato di `json_pointer` e remediation: un agente
decide il da farsi dal codice di uscita, senza logica ad hoc per interpretare l'output.

- **CLI** — `themis <comando> --format json`, per script, CI e agenti senza supporto MCP.
- **Server MCP** — `themis-mcp` espone `validate`/`fix-plan`/`brief`/`trace-code`/`coverage-diff`/
  `verify-sources` come tool nativi per Claude Code, Cursor o qualunque client MCP.

→ **[Guida all'integrazione con agenti di coding](docs/AGENT-INTEGRATION.md)** e
[`docs/MCP.md`](docs/MCP.md) (setup del server MCP).

## Documentazione

| Documento | Contenuto |
|---|---|
| [`docs/ONBOARDING.md`](docs/ONBOARDING.md) | guida per nuovi clienti: acquisto, installazione, licenza, primo grafo, CLI vs MCP, git |
| [`docs/THEMIS-SPEC-001.md`](docs/THEMIS-SPEC-001.md) | la specifica normativa che questo toolkit implementa |
| [`docs/QUICKSTART.md`](docs/QUICKSTART.md) | primi 15 minuti, comando per comando |
| [`docs/AGENT-INTEGRATION.md`](docs/AGENT-INTEGRATION.md) | loop di lavoro per un agente di coding, CLI vs MCP, CI |
| [`docs/MCP.md`](docs/MCP.md) | server MCP: installazione, configurazione client, tool e risorse |
| [`docs/CLI.md`](docs/CLI.md) | riferimento completo di ogni comando e flag |
| [`docs/RULEBOOK.md`](docs/RULEBOOK.md) | catalogo dei 73 controlli: motivazione, remediation, esempi |
| [`docs/SCHEMA.md`](docs/SCHEMA.md) | struttura del grafo canonico (ogni entità e campo) |
| [`docs/VERSIONING.md`](docs/VERSIONING.md) | compatibilità fra versione del toolkit, formato JSON e schema |

## Licenza

Software proprietario — codice sorgente non pubblicato, nessuna distribuzione via PyPI pubblico.
L'uso del prodotto (binario compilato) è regolato da [`EULA.md`](EULA.md). Cronologia delle
versioni in [`CHANGELOG.md`](CHANGELOG.md).

Questo repository contiene solo materiale destinato al cliente (documentazione, specifica, binari
compilati): non è il repository di sviluppo, non accetta contributi di codice, non ha una issue
tracker per bug del toolkit (per assistenza, i canali sono quelli indicati dopo l'acquisto).
