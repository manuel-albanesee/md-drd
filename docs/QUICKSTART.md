# Primi 15 minuti

Da zero a un grafo validato, un difetto corretto e i documenti proiettati. Copia-incolla i
comandi così come sono: usano le fixture già presenti in `examples/`, non serve preparare nulla.

## 0. Installazione

```bash
pip install -e .[dev]        # da checkout — vedi README.md § Installazione per il binario cliente
md-drd --version
```

## 1. Crea un progetto minimo

`md-drd init` genera lo scheletro più piccolo che sia già conforme: uno stakeholder per
categoria, un bisogno, un requisito, un work package, un test case.

```bash
md-drd init progetto-demo/
```

Guarda cosa è stato creato:

```bash
cat progetto-demo/graph.yaml
```

## 2. Valida

```bash
md-drd validate progetto-demo/graph.yaml -v
```

Uscita attesa: `0 bloccanti, 0 maggiori`, codice di uscita `0`. Questo è il punto di partenza:
uno scheletro minimo ma già conforme, non un file vuoto da riempire alla cieca.

## 3. Guarda i gate in azione su un grafo rotto

`examples/broken/graph.yaml` contiene 21 difetti diversi introdotti di proposito (ognuno
annotato con un commento `# DIFETTO Gx.y`). È il modo più rapido per vedere cosa intercetta il
toolkit prima che arrivi in implementazione:

```bash
md-drd fix-plan examples/broken/graph.yaml
```

L'output raggruppa i difetti per controllo, bloccanti prima, con la remediation già inclusa in
ogni gruppo — non serve consultare altro per sapere cosa correggere. Per una singola voce del
catalogo:

```bash
md-drd rules --check G3.2 --detailed
```

## 4. Proietta i documenti

Dal grafo conforme di prima (o da `examples/complete/graph.yaml`, che ha tutte le entità della
specifica: vincoli, ADR, compromessi, spike, storie, punti aperti, due release):

```bash
md-drd project examples/complete/graph.yaml --out out/
ls out/
```

Genera cinque file: matrice di tracciabilità (`05_RTM.md`), report di conformità
(`06_Conformance-Report.md`), registro di governance (`07_Governance-Log.md`), backlog
(`backlog.csv`), export ReqIF (`requirements.reqif`) — prefissati col codice progetto
(`SCHEDE-05_RTM.md`, ...) perché `examples/complete/graph.yaml` dichiara `meta.project_code`.
Sono derivati, non scritti a mano: si rigenerano a ogni run.

## 5. Genera un brief per un agente

Un brief è un documento autosufficiente per implementare un work package: requisiti da
soddisfare, vincoli, dipendenze, architettura di riferimento, definizione di fatto, casi di
test, rischi, tracciabilità alle fonti — tutto derivato deterministicamente dal grafo, niente da
indovinare.

```bash
md-drd brief examples/complete/graph.yaml WP-002 --out WP-002-brief.md
cat WP-002-brief.md
```

## 6. Con un agente o in CI, aggiungi `--format json`

Ogni comando accetta `--format json`: un solo oggetto JSON su stdout, diagnostica su stderr, così
`json.loads(stdout)` funziona sempre. Ogni difetto porta il proprio `json_pointer` verso il punto
esatto del grafo e la propria `remediation`.

```bash
md-drd validate examples/broken/graph.yaml --format json | jq '.exit_code, .summary'
```

## Prossimi passi

- **Un agente di coding scrive/consuma il grafo?** → [Guida all'integrazione con agenti di
  coding](AGENT-INTEGRATION.md) (CLI, server MCP, CI).
- **Serve il dettaglio di ogni comando/flag?** → [Riferimento CLI](CLI.md) (generato dal parser).
- **Serve capire un controllo o un campo dello schema?** → [`RULEBOOK.md`](RULEBOOK.md) (catalogo
  dei controlli) e [`SCHEMA.md`](SCHEMA.md) (struttura del grafo).
- **Un documento sorgente reale (analisi funzionale, verbale) da cui derivare il grafo?** → la
  skill di authoring in [`skills/md-drd-authoring/`](../skills/md-drd-authoring/SKILL.md) itera
  `validate`/`fix-plan` automaticamente fino a zero difetti bloccanti.
