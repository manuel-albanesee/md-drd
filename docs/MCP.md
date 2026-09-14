# Server MCP

Il pacchetto include un server [Model Context Protocol](https://modelcontextprotocol.io) che
espone `validate`, `fix-plan`, `brief`, `trace-code`, `coverage-diff` e `verify-sources` come
tool invocabili direttamente da un coding agent (Claude Code, Cursor, o qualunque client MCP),
più il catalogo dei controlli e il grafo come risorse consultabili — senza che l'agente debba
fare shell-out al CLI e fare parsing di stdout.

È una dipendenza **opzionale**: il resto del toolkit (CLI, libreria) non la richiede e non la
importa mai.

## Installazione

```bash
pip install "md-drd-toolkit[mcp]"
```

Questo registra anche il comando `md-drd-mcp`, l'eseguibile del server.

## Grafo di default

Il server è pensato per essere legato a **un solo progetto per istanza**, come i server MCP
filesystem/git sono legati a una sola radice: passa `--graph` all'avvio (o imposta la variabile
d'ambiente `MD_DRD_GRAPH_PATH`) e ogni tool che non riceve esplicitamente `graph_path` opera su
quel grafo. Ogni tool accetta comunque `graph_path` per operare su un grafo diverso senza
riavviare il server — necessario per `coverage_diff`, che ne confronta sempre due.

Se non configuri un grafo di default, ogni chiamata deve passare `graph_path` esplicitamente
(le risorse `md-drd://graph/...`, che non hanno un parametro equivalente, restano inutilizzabili
senza un grafo di default).

## Configurazione client

### Claude Code

```bash
claude mcp add md-drd -- md-drd-mcp --graph /percorso/assoluto/al/grafo.yaml
```

oppure aggiungendo a `.mcp.json` nella root del progetto:

```json
{
  "mcpServers": {
    "md-drd": {
      "command": "md-drd-mcp",
      "args": ["--graph", "${workspaceFolder}/graph.yaml"]
    }
  }
}
```

### Cursor

`.cursor/mcp.json` (progetto) o `~/.cursor/mcp.json` (globale):

```json
{
  "mcpServers": {
    "md-drd": {
      "command": "md-drd-mcp",
      "env": {
        "MD_DRD_GRAPH_PATH": "/percorso/assoluto/al/grafo.yaml"
      }
    }
  }
}
```

### Client MCP generico

Qualunque client che parla il trasporto `stdio` funziona: avvia `md-drd-mcp` (opzionalmente con
`--graph <path>` e `--lang {it,en}`) e comunica su stdin/stdout col protocollo MCP standard.
`--transport {stdio,sse,streamable-http}` è disponibile per client che preferiscono HTTP, ma
`stdio` è quello atteso da Claude Code e Cursor ed è il default.

## Tool esposti

| Tool | Equivalente CLI | Note |
|---|---|---|
| `validate` | `md-drd validate` | schema + i 10 gate; primo tool da chiamare su qualunque grafo |
| `fix_plan` | `md-drd fix-plan` | difetti raggruppati per controllo, bloccanti prima, con remediation |
| `brief` | `md-drd brief` | brief autosufficiente per un work package (o tutti, con `all_work_packages`) |
| `trace_code` | `md-drd trace-code` | citazioni di id del grafo nel codice sorgente, id inesistenti/assunzioni invalidate |
| `coverage_diff` | `md-drd coverage-diff` | copertura dei segmenti prescrittivi fra due baseline |
| `verify_sources` | `md-drd verify-sources` | hash dei documenti sorgente rispetto al grafo |

Ogni tool restituisce esattamente lo stesso oggetto JSON del corrispondente comando CLI con
`--format json` (stesso `output_format_version`, stesso `exit_code`), così un agente che già
conosce il contratto dati del CLI non deve impararne uno nuovo.

### Troncamento degli elenchi grandi

Un grafo di grandi dimensioni può produrre centinaia di difetti o occorrenze: per non saturare
la finestra di contesto dell'agente, gli elenchi (`defects`, `groups[].occurrences`, `briefs`,
`findings.*`) sono troncati a un limite configurabile per chiamata (`max_defects`,
`max_occurrences_per_group`, `max_briefs`, `max_findings`; default 200, `max_briefs` 50). Il
troncamento non è mai silenzioso: ogni elenco troncato porta accanto un blocco `*_page` con
`returned`/`total`/`truncated`.

### Errori come risultati strutturati

Un grafo assente, un percorso invalido o un uso scorretto **non** fanno fallire la chiamata MCP
con un'eccezione di protocollo: il tool restituisce lo stesso payload di errore che il CLI
emette con `--format json` (`status: "error"`, `exit_code`, `error.code`/`error.message`). Un
agente gestisce l'errore leggendo lo stesso payload che già sa interpretare, senza dover
intercettare un'eccezione MCP diversa per ogni tool.

## Risorse esposte

| URI | Contenuto |
|---|---|
| `md-drd://rules` | catalogo completo dei controlli (stesso payload di `md-drd rules`) |
| `md-drd://rules/{id}` | un controllo singolo (`G3.2`) o tutti i controlli di un gate (`G3`) |
| `md-drd://graph` | meta-dati e id di tutti gli elementi del grafo di default |
| `md-drd://graph/{id}` | un elemento del grafo di default per id (es. `WP-005`), col suo JSON Pointer |

Le risorse `md-drd://graph/...` richiedono un grafo di default configurato all'avvio
(`--graph`/`MD_DRD_GRAPH_PATH`): a differenza dei tool non hanno un parametro `graph_path` per
riceverlo a ogni chiamata.

## Verifica manuale

```bash
# elenco dei tool/risorse esposti, via l'ispettore ufficiale del protocollo
npx @modelcontextprotocol/inspector md-drd-mcp --graph examples/complete/graph.yaml
```

La suite del toolkit include anche un test di integrazione che avvia il server come sottoprocesso
e lo interroga con l'SDK client MCP reale, su stdio (`tests/test_mcp_server.py`) — si salta
automaticamente se la dipendenza opzionale `mcp` non è installata.

## Pubblicazione nelle directory MCP

Da valutare quando il pacchetto sarà pubblicato su PyPI (F0.T4): le directory pubbliche di
server MCP (il registro ufficiale su `modelcontextprotocol.io`, quello di Anthropic per Claude
Code/Desktop, quelli di terze parti come Smithery/Glama) sono un canale di scoperta a costo
pressoché nullo per chi cerca "requirements MCP" o "spec-driven MCP" — ma richiedono un pacchetto
installabile pubblicamente, non solo questo repository.
