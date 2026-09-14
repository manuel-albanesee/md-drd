# Guida all'integrazione con agenti di coding

MD-DRD è pensato perché un agente (Claude Code, Cursor, o qualunque altro) lo chiami durante il
proprio ciclo di lavoro, non solo perché una persona lo lanci a mano. Questa guida copre le tre
superfici — CLI, server MCP, CI — e il loop di lavoro consigliato. Per l'installazione vedi il
[README](../README.md); per il dettaglio di ogni comando il [Riferimento CLI](CLI.md).

## Perché un agente può fidarsi dell'output

> I gate sono codice, non prompt (MD-DRD-SPEC-001 §16.2).

Il toolkit non usa un modello linguistico per decidere se un requisito è ben formato: applica
regole deterministiche. Questo ha una conseguenza diretta per un agente: un grafo generato da un
agente può essere validato dallo stesso agente in modo riproducibile, perché il giudizio non
proviene dalla stessa istanza che ha scritto il contenuto — proviene da codice.

Tre proprietà rendono l'output consumabile a colpo sicuro, senza logica ad hoc lato agente:

- **Codici di uscita stabili** (`md_drd/exitcodes.py`, contratto pubblico): `0` conforme, `1`
  difetti bloccanti/regressione, `2` schema invalido, `3` uso errato, `4` input invalido, `5`
  errore interno. Un agente decide il da farsi dal codice di uscita prima ancora di leggere
  l'output.
- **`--format json` su ogni comando**: un solo oggetto JSON su stdout, diagnostica e progress su
  stderr — `json.loads(stdout)` funziona sempre, senza dover filtrare righe di log. Lo schema di
  ogni payload è versionato (`output_format_version`, cfr. `docs/VERSIONING.md`).
- **Ogni difetto porta la propria remediation** e il proprio `json_pointer` verso il punto esatto
  del grafo: un agente non deve indovinare cosa correggere né dove.

## CLI o server MCP?

| | CLI | Server MCP |
|---|---|---|
| Quando | script, CI, agenti senza supporto MCP | Claude Code, Cursor, o qualunque client MCP |
| Overhead | shell-out + parsing di stdout | tool nativo, nessun parsing di testo |
| Formato | identico (`--format json` ↔ tool MCP) | stesso `output_format_version`, stesso `exit_code` |
| Setup | nessuno (entry point `md-drd`) | `pip install md-drd-toolkit[mcp]`, un grafo di default |

Stesso contratto dati sotto: un agente che già conosce l'output JSON del CLI non deve impararne
uno nuovo per usare i tool MCP. Configurazione completa (Claude Code, Cursor, client generico,
troncamento degli elenchi grandi) in [`docs/MCP.md`](MCP.md).

## Loop di lavoro consigliato per un work package

1. **Prima di scrivere codice**, chiedi il brief del work package invece di rileggere l'intero
   grafo: `md-drd brief grafo.yaml WP-009` (CLI) o il tool MCP `brief`. Contiene requisiti da
   soddisfare, vincoli, dipendenze, architettura di riferimento, definizione di fatto, casi di
   test, rischi e tracciabilità alle fonti — derivato deterministicamente, non riassunto a mano.
2. **Implementa**, citando gli id del grafo pertinenti nel codice o nei commit (lo stesso formato
   libero già usato per gli ADR/requisiti: `WP-009`, `REQ-042`, ...).
3. **Prima di aprire la PR**, verifica la tracciabilità inversa sui soli file toccati (non
   sull'intero albero):

   ```bash
   git diff --name-only origin/main...HEAD > /tmp/cambiati.txt
   md-drd trace-code grafo.yaml --code-dir . --files /tmp/cambiati.txt
   ```

   Segnala citazioni verso id inesistenti o assunzioni invalidate (bloccante), ADR superate o
   punti aperti non risolti (avviso) — prima che li trovi un revisore umano.
4. **Se il work package tocca il grafo stesso** (nuovo requisito, WP, ADR, ...), ripeti
   `validate`/`fix-plan` fino a zero difetti bloccanti prima di committare:

   ```bash
   md-drd fix-plan grafo.yaml --format json --watch --interval 2
   ```

   `--watch` rivalida a ogni salvataggio: utile mentre l'agente itera sul file invece di
   rilanciare il comando ad ogni modifica.

## Derivare un grafo da zero

Se il punto di partenza è un documento di analisi (funzionale, verbale, backlog) e non un grafo
già esistente, non scrivere lo YAML a mano: la skill
[`skills/md-drd-authoring/`](../skills/md-drd-authoring/SKILL.md) guida l'estrazione e itera
`validate`/`fix-plan` automaticamente fino a zero difetti bloccanti. Non è pensata per giudicare
la conformità di un grafo già scritto da altri — per quello basta `md-drd validate` da solo.

## CI

Il principio è lo stesso della verifica locale: il codice di uscita decide l'esito della build,
senza bisogno di fare parsing dell'output testuale. Due scenari diversi, non confonderli:

**Uso interno su un proprio progetto** (questo repository e chi lo usa come dipendenza sui
propri grafi): `.github/workflows/validate-graph.yml` in questo stesso repository è un
[workflow riusabile](https://docs.github.com/actions/using-workflows/reusing-workflows) —
richiamabile da qualunque altro repository con due righe:

```yaml
name: MD-DRD
on: [push, pull_request]
jobs:
  validate:
    uses: manuel-albanesee/md-drd-toolkit/.github/workflows/validate-graph.yml@master
    with:
      graph-path: requisiti/graph.yaml
      # opzionale: fallisce la build se la copertura scende sotto la soglia
      # baseline-graph-path: requisiti/graph.yaml   # es. checkout di un tag precedente
      # min-coverage: "0.98"
    secrets:
      toolkit-token: ${{ secrets.MD_DRD_TOOLKIT_TOKEN }}
```

Installa il toolkit da sorgente (non il binario compilato) ed esegue `validate` — e, se richiesto,
`coverage-diff` — col codice di uscita che decide l'esito della build. Non serve alcun file di
licenza: `license_.enforce()` è un no-op fuori dalla build compilata
(`md_drd/_build_marker.py`), esattamente come per la CI di questo stesso repo
(`.github/workflows/ci.yml`).

Passo manuale, non automatizzabile da qui: il repository è privato, quindi `pip install` da un
repo diverso da quello chiamante richiede un token con accesso in lettura — creare un [personal
access token fine-grained](https://github.com/settings/personal-access-tokens) con permesso
"Contents: read" limitato a `md-drd-toolkit`, e salvarlo come secret `MD_DRD_TOOLKIT_TOKEN` nel
repository chiamante (o come secret d'organizzazione, per non ripetere il passo per ogni
progetto). `ref: master` nel workflow riusabile insegue sempre l'ultimo commit del toolkit; per
una pipeline che non deve rompersi a un rilascio nuovo, pinnare un tag (`@v2.0.0`, ecc.) sia nel
riferimento al workflow (`uses: ...@v2.0.0`) sia nell'input `ref`.

**Consegna a un cliente esterno**: scenario diverso, non coperto dal workflow riusabile sopra —
un cliente non ha (e non deve avere) accesso al sorgente di questo repo, quindi non può mai usare
`toolkit-token`: userebbe quel token per clonare il prodotto stesso, non solo per validarlo. Usa
invece il binario compilato + un file di licenza, vedi [Installazione](../README.md#installazione).
Questo impone un vincolo reale sulla CI del cliente, non solo di distribuzione: la licenza
autorizza un elenco **fisso** di fingerprint macchina (`machine_fingerprints`,
`md_drd/license.py`), calcolate da `uuid.getnode()` — stabile su una macchina reale, non
garantita stabile su una VM ricreata ad ogni run. Quindi:

- **Runner self-hosted dal cliente** (macchina/VM persistente che lui controlla): funziona come
  un suo workstation qualunque — `md-drd license fingerprint` una volta sul runner, la fingerprint
  autorizzata nella sua licenza (dentro i suoi `max_seats`), il binario e il file di licenza
  installati sul runner (o iniettati da un secret), poi `md-drd validate grafo.yaml --format
  json` come qualunque altro step CI. `md-drd ci-init --graph grafo.yaml` genera in locale il
  file `.github/workflows/md-drd.yml` con questo stesso step già pronto (`runs-on: self-hosted`,
  nessun riferimento al sorgente del toolkit, a differenza del workflow riusabile sopra) — il
  cliente lo commette così com'è, senza scriverlo a mano.
- **Runner effimero (GitHub-hosted, ecc.)**: non supportato oggi — nessun modo di autorizzare in
  anticipo una macchina che non esiste ancora al momento dell'emissione della licenza, e nessun
  server di attivazione per farlo a runtime (scelta voluta, air-gapped). Gap tracciato in
  `ROADMAP.md`, F7.T4.5.
- **Nessun runner CI del cliente (né self-hosted né altro)**: `md-drd hooks-init [--graph
  grafo.yaml]` scrive due hook git nativi (`.githooks/pre-push`, `.githooks/post-merge`) sulla
  macchina di sviluppo stessa — zero problema di fingerprint, perché è già la macchina del
  cliente, autorizzata come un suo workstation qualunque. `pre-push` blocca il push se il grafo
  non è conforme; `post-merge` può solo avvisare dopo un pull (git non offre un hook "prima" di
  un pull: fetch e merge devono completarsi prima di sapere cosa sta arrivando). Attivazione con
  un solo comando manuale, mai eseguito da noi: `git config core.hooksPath .githooks` (stampato
  da `hooks-init` stesso; anche disponibile come `md-drd init --with-hooks` per chi parte da
  zero, invece di ricordarsi un comando separato). **Non sostituisce un runner CI**: un hook
  locale è una convenzione bypassabile (`git push --no-verify`, o semplicemente non
  installandolo) e non può bloccare il merge di una PR, che è un concetto lato server — solo
  `ci-init` offre un gate imposto. Cfr. `ROADMAP.md`, F7.T4.7.

Il canale di consegna del binario al cliente (F0.T4.1) resta comunque manuale, non ancora
attivato.

> Il commento automatico sulla PR col delta di conformità/copertura (F6.T3.2) non è ancora
> costruito — oggi la copertura del workflow riusabile si ferma al fallire/passare della build.

## Riferimenti

- [Riferimento CLI](CLI.md) — ogni comando e flag, generato dal parser.
- [`docs/MCP.md`](MCP.md) — server MCP: installazione, configurazione client, tool e risorse.
- [`docs/RULEBOOK.md`](RULEBOOK.md) — catalogo dei controlli deterministici.
- [`docs/SCHEMA.md`](SCHEMA.md) — struttura del grafo canonico.
- [`docs/VERSIONING.md`](VERSIONING.md) — compatibilità fra versione del toolkit, formato di
  output JSON e schema del grafo.
