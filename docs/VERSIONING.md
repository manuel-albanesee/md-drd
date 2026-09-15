# Versioning

Questo progetto ha **tre numeri di versione indipendenti**, ciascuno con una propria politica di
compatibilità. Confonderli è l'errore più comune per chi integra il toolkit in un agente o in una
pipeline CI: qui sotto sono documentati separatamente, con dove trovarli nel codice.

## 1. Versione del toolkit

**Dove:** `themis._version.__version__` (unica fonte; `pyproject.toml` la legge dinamicamente via
`[tool.setuptools.dynamic]`, non dichiara un numero proprio).

**Politica:** [Semantic Versioning](https://semver.org/lang/it/) — `MAJOR.MINOR.PATCH`.

- `MAJOR`: cambi incompatibili nell'interfaccia a riga di comando, nel comportamento dei gate di
  validazione (nuovo difetto bloccante su grafi prima validi), o nel meccanismo di licenza.
- `MINOR`: nuove funzionalità retrocompatibili (nuovo comando, nuovo proiettore, nuova opzione).
- `PATCH`: correzioni di bug che non cambiano il comportamento osservabile atteso.

Cronologia in `CHANGELOG.md` (formato [Keep a Changelog](https://keepachangelog.com/it/1.0.0/)).

## 2. Versione del formato di output JSON

**Dove:** `themis._version.OUTPUT_FORMAT_VERSION` (oggi `"1.0"`), incluso nell'envelope di ogni
risposta `--format json`/tool MCP.

**Politica:** cambia **solo** per modifiche incompatibili della *forma* dei payload (rinominare o
rimuovere un campo, cambiare il tipo di un campo esistente, cambiare la semantica di un exit
code). Aggiungere un campo facoltativo a un payload esistente **non** la incrementa — un
consumatore che legge solo i campi che conosce non deve rompersi ad ogni release del toolkit.

Chi integra il toolkit (CI, editor, altro agente) dovrebbe verificare `OUTPUT_FORMAT_VERSION`
prima di fare affidamento sulla struttura esatta della risposta, non sulla versione del toolkit:
le due possono avanzare a ritmi diversi (un `PATCH`/`MINOR` del toolkit può non toccare affatto il
formato di output).

## 3. Versione dello schema del grafo (`meta.spec_version`)

**Dove:** campo obbligatorio `meta.spec_version` di ogni grafo, vincolato dallo schema JSON
(`themis/data/schema/themis-graph.schema.json`) a `const: "THEMIS-SPEC-001/1.0"`.

**Politica attuale: nessuna tolleranza di versione minore.** Lo schema oggi accetta **solo**
`"THEMIS-SPEC-001/1.0"` letterale — qualunque altro valore (incluso un ipotetico `.../1.1`) è
rifiutato dallo schema stesso, prima ancora che i gate `G0`..`G9` entrino in gioco. Non esiste
oggi un meccanismo di negoziazione, warning "versione più vecchia" o migrazione automatica: un
grafo con `spec_version` diverso da quello atteso semplicemente non valida.

**Intenzione futura (non ancora implementata):** quando uscirà una `THEMIS-SPEC-001/1.1` (o
superiore), la scelta fra *rifiuto secco*, *warning con validazione permissiva* o *migrazione
automatica del grafo* è una decisione di prodotto da prendere a quel punto, guidata da cosa
rompe realmente la compatibilità fra le due versioni dello schema — non un impegno implementativo
preso ora. Questa sezione va aggiornata quando quella decisione verrà presa.

## Perché tre versioni e non una sola

Cambiano per ragioni indipendenti: il motore può guadagnare un nuovo comando (`MINOR` del
toolkit) senza toccare né la forma dell'output JSON né lo schema del grafo; il formato di output
può stabilizzarsi per anni mentre il toolkit continua a ricevere patch; lo schema del grafo (la
metodologia THEMIS-SPEC-001 in sé) evolve secondo un ciclo editoriale suo, non secondo il ciclo di
rilascio del software che la implementa. Legarle a un solo numero costringerebbe un consumatore
a ri-verificare tutto ad ogni release, anche quando nulla di ciò che gli interessa è cambiato.
