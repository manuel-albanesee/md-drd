# Riferimento CLI

> Documento **generato** da `scripts/generate_docs.py` a partire dal parser `argparse` di `themis/cli.py` (`--help` di ogni comando). Non modificare a mano: la fonte e' il codice.

```text
usage: themis [-h] [--version]
              {validate,fix-plan,rules,project,all,init,ci-init,hooks-init,verify-sources,import-tex,coverage-diff,brief,trace-code,license}
              ...

Themis — assistente di sviluppo guidato da specifica: valida il grafo canonico dei requisiti e ne
proietta i documenti.

positional arguments:
  {validate,fix-plan,rules,project,all,init,ci-init,hooks-init,verify-sources,import-tex,coverage-diff,brief,trace-code,license}
    validate            valida schema e gate deterministici
    fix-plan            difetti raggruppati per controllo, con remediation
    rules               catalogo dei controlli deterministici
    project             genera i documenti proiettati
    all                 valida e poi proietta
    init                crea un progetto Themis minimo gia' conforme
    ci-init             genera un workflow GitHub Actions autonomo per CI su runner self-hosted
                        del cliente
    hooks-init          scrive hook git nativi (pre-push blocca, post-merge avvisa) sulla macchina
                        di sviluppo
    verify-sources      confronta gli hash dei documenti sorgente
    import-tex          estrae source_documents/source_segments/open_points dal template .tex
                        FSD/TSD (non un parser LaTeX generico: solo quel template specifico)
    coverage-diff       confronta la copertura dei segmenti prescrittivi fra due baseline
    brief               brief autosufficiente per agente da un work package
    trace-code          tracciabilita' inversa: citazioni di id del grafo nel codice
    license             fingerprint macchina, stato della licenza installata o attivazione
                        automatica (solo build compilata)

options:
  -h, --help            show this help message and exit
  --version             show program's version number and exit

Codici di uscita: 0 conforme · 1 non conforme · 2 schema invalido · 3 uso errato · 4 input
invalido · 5 errore interno.
```

## `themis validate`

```text
usage: themis validate [-h] [--format {text,json}] [--lang {it,en}] [--schema SCHEMA]
                       [--no-schema] [-v]
                       graph

positional arguments:
  graph

options:
  -h, --help            show this help message and exit
  --format {text,json}  text per una persona, json per un agente o una pipeline
  --lang {it,en}        lingua dei messaggi (default: $THEMIS_LANG, locale di sistema, it)
  --schema SCHEMA       JSON Schema alternativo (default: themis-graph.schema.json del pacchetto)
  --no-schema           esegui solo i gate
  -v, --verbose         elenca i singoli difetti
```

## `themis fix-plan`

```text
usage: themis fix-plan [-h] [--format {text,json}] [--lang {it,en}] [--schema SCHEMA]
                       [--blocking-only] [--watch] [--interval INTERVAL]
                       graph

positional arguments:
  graph

options:
  -h, --help            show this help message and exit
  --format {text,json}  text per una persona, json per un agente o una pipeline
  --lang {it,en}        lingua dei messaggi (default: $THEMIS_LANG, locale di sistema, it)
  --schema SCHEMA
  --blocking-only       ignora i difetti maggiori
  --watch               rivalida a ogni modifica del grafo
  --interval INTERVAL   intervallo di polling con --watch
```

## `themis rules`

```text
usage: themis rules [-h] [--format {text,json}] [--lang {it,en}] [--check CHECK] [--gate GATE]
                    [--detailed] [--markdown]

options:
  -h, --help            show this help message and exit
  --format {text,json}  text per una persona, json per un agente o una pipeline
  --lang {it,en}        lingua dei messaggi (default: $THEMIS_LANG, locale di sistema, it)
  --check CHECK         un singolo controllo, es. G3.2
  --gate GATE           tutti i controlli di un gate, es. G3
  --detailed            motivazione, remediation ed esempi
  --markdown            emetti docs/RULEBOOK.md su stdout
```

## `themis project`

```text
usage: themis project [-h] [--format {text,json}] [--lang {it,en}] [--out OUT] [--schema SCHEMA]
                      [--no-schema] [-v]
                      graph

positional arguments:
  graph

options:
  -h, --help            show this help message and exit
  --format {text,json}  text per una persona, json per un agente o una pipeline
  --lang {it,en}        lingua dei messaggi (default: $THEMIS_LANG, locale di sistema, it)
  --out OUT
  --schema SCHEMA
  --no-schema
  -v, --verbose
```

## `themis all`

```text
usage: themis all [-h] [--format {text,json}] [--lang {it,en}] [--out OUT] [--schema SCHEMA]
                  [--no-schema] [-v]
                  graph

positional arguments:
  graph

options:
  -h, --help            show this help message and exit
  --format {text,json}  text per una persona, json per un agente o una pipeline
  --lang {it,en}        lingua dei messaggi (default: $THEMIS_LANG, locale di sistema, it)
  --out OUT
  --schema SCHEMA
  --no-schema
  -v, --verbose
```

## `themis init`

```text
usage: themis init [-h] [--format {text,json}] [--lang {it,en}] [--project PROJECT] [--code CODE]
                   [--profile {minimal,full}] [--baseline-id BASELINE_ID] [--with-hooks] [--force]
                   [directory]

positional arguments:
  directory

options:
  -h, --help            show this help message and exit
  --format {text,json}  text per una persona, json per un agente o una pipeline
  --lang {it,en}        lingua dei messaggi (default: $THEMIS_LANG, locale di sistema, it)
  --project PROJECT     nome del progetto
  --code CODE           codice breve del progetto, es. ACME
  --profile {minimal,full}
                        minimal: scheletro essenziale; full: tutte le 22 collezioni popolate
  --baseline-id BASELINE_ID
                        sovrascrive l'id di baseline generato (pattern BASELINE-<CODICE>-AAAAMMGG-
                        NN)
  --with-hooks          scrivi anche gli hook git nativi di 'hooks-init' (pre-push blocca, post-
                        merge avvisa) subito dopo lo scaffold
  --force               sovrascrivi i file esistenti
```

## `themis ci-init`

```text
usage: themis ci-init [-h] [--format {text,json}] [--lang {it,en}] [--graph GRAPH]
                      [--runner {self-hosted}] [--workflow-filename WORKFLOW_FILENAME] [--force]
                      [directory]

positional arguments:
  directory

options:
  -h, --help            show this help message and exit
  --format {text,json}  text per una persona, json per un agente o una pipeline
  --lang {it,en}        lingua dei messaggi (default: $THEMIS_LANG, locale di sistema, it)
  --graph GRAPH         percorso del grafo canonico nel repository del cliente, embedito nello
                        step 'validate' del workflow generato
  --runner {self-hosted}
                        oggi solo self-hosted: un runner GitHub-hosted effimero non puo' essere
                        autorizzato in anticipo dalla licenza offline
  --workflow-filename WORKFLOW_FILENAME
                        nome del file dentro .github/workflows/
  --force               sovrascrivi il workflow se esiste gia'
```

## `themis hooks-init`

```text
usage: themis hooks-init [-h] [--format {text,json}] [--lang {it,en}] [--graph GRAPH] [--force]
                         [directory]

positional arguments:
  directory

options:
  -h, --help            show this help message and exit
  --format {text,json}  text per una persona, json per un agente o una pipeline
  --lang {it,en}        lingua dei messaggi (default: $THEMIS_LANG, locale di sistema, it)
  --graph GRAPH         percorso del grafo canonico validato dagli hook generati
  --force               sovrascrivi gli hook se esistono gia'
```

## `themis verify-sources`

```text
usage: themis verify-sources [-h] [--format {text,json}] [--lang {it,en}] --docs-dir DOCS_DIR
                             [--map MAP]
                             graph

positional arguments:
  graph

options:
  -h, --help            show this help message and exit
  --format {text,json}  text per una persona, json per un agente o una pipeline
  --lang {it,en}        lingua dei messaggi (default: $THEMIS_LANG, locale di sistema, it)
  --docs-dir DOCS_DIR
  --map MAP             ID=percorso relativo a --docs-dir (es. D1=Guida.docx), ripetibile
```

## `themis import-tex`

```text
usage: themis import-tex [-h] [--format {text,json}] [--lang {it,en}] --doc DOC [--title TITLE]
                         [--authority AUTHORITY] [--out OUT]

options:
  -h, --help            show this help message and exit
  --format {text,json}  text per una persona, json per un agente o una pipeline
  --lang {it,en}        lingua dei messaggi (default: $THEMIS_LANG, locale di sistema, it)
  --doc DOC             ID=percorso.tex (es. D1=FSD/Analisi.tex), ripetibile; l'ordine passato
                        conta per la deduplicazione dei punti aperti fra documenti
  --title TITLE         ID=titolo esplicito (default: dal nome file), ripetibile
  --authority AUTHORITY
                        ID=normative|indicative|informative (default: normative), ripetibile
  --out OUT             scrive il frammento YAML (source_documents/source_segments/open_points) su
                        questo file, da rivedere e fondere a mano nel grafo
```

## `themis coverage-diff`

```text
usage: themis coverage-diff [-h] [--format {text,json}] [--lang {it,en}]
                            [--min-coverage MIN_COVERAGE]
                            graph_before graph_after

positional arguments:
  graph_before
  graph_after

options:
  -h, --help            show this help message and exit
  --format {text,json}  text per una persona, json per un agente o una pipeline
  --lang {it,en}        lingua dei messaggi (default: $THEMIS_LANG, locale di sistema, it)
  --min-coverage MIN_COVERAGE
                        soglia minima richiesta sulla nuova baseline (default 0.98)
```

## `themis brief`

```text
usage: themis brief [-h] [--format {text,json}] [--lang {it,en}] [--out OUT] [--all]
                    [--out-dir OUT_DIR]
                    graph [wp]

positional arguments:
  graph
  wp                    ID del work package (es. WP-005)

options:
  -h, --help            show this help message and exit
  --format {text,json}  text per una persona, json per un agente o una pipeline
  --lang {it,en}        lingua dei messaggi (default: $THEMIS_LANG, locale di sistema, it)
  --out OUT             file di output (default: stdout)
  --all                 un brief per ogni work package
  --out-dir OUT_DIR     cartella di output con --all
```

## `themis trace-code`

```text
usage: themis trace-code [-h] [--format {text,json}] [--lang {it,en}] --code-dir CODE_DIR
                         [--ext EXT] [--files FILES]
                         graph

positional arguments:
  graph

options:
  -h, --help            show this help message and exit
  --format {text,json}  text per una persona, json per un agente o una pipeline
  --lang {it,en}        lingua dei messaggi (default: $THEMIS_LANG, locale di sistema, it)
  --code-dir CODE_DIR
  --ext EXT             estensioni separate da virgola (default: .py,.ts,.tsx,.js,.jsx,.sql,.md)
  --files FILES         limita lo scan ai file elencati in questo file (uno per riga, relativi a
                        --code-dir) invece dell'intero albero — tipicamente l'output di 'git diff
                        --name-only' su una PR
```

## `themis license`

```text
usage: themis license [-h] [--format {text,json}] [--lang {it,en}] [--token TOKEN] [--url URL]
                      [{fingerprint,info,activate}]

positional arguments:
  {fingerprint,info,activate}
                        fingerprint: identificatore da comunicare al venditore (default); info:
                        verifica la licenza installata; activate: scarica e installa la licenza da
                        un activation token (richiede --token)

options:
  -h, --help            show this help message and exit
  --format {text,json}  text per una persona, json per un agente o una pipeline
  --lang {it,en}        lingua dei messaggi (default: $THEMIS_LANG, locale di sistema, it)
  --token TOKEN         activation token ricevuto dopo l'acquisto (richiesto con 'activate')
  --url URL             endpoint di attivazione, per test/staging (default: variabile
                        THEMIS_ACTIVATION_URL o l'endpoint di produzione)
```
