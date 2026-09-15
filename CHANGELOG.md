# Changelog

Le modifiche rilevanti di questo progetto sono documentate in questo file.

Il formato segue [Keep a Changelog](https://keepachangelog.com/it/1.0.0/). Il numero di versione
segue [Semantic Versioning](https://semver.org/lang/it/) ed è quello del **toolkit**: non va
confuso con la versione dello schema del grafo (`meta.spec_version`) né con il contratto del
formato di output JSON (`OUTPUT_FORMAT_VERSION`), versionati separatamente — vedi
`docs/VERSIONING.md`.

## [3.0.0] - 2026-09-15

### Modificato
- **Rebranding completo del prodotto: MD-DRD → Themis.** Cambio **MAJOR** per la policy di
  `docs/VERSIONING.md` (rottura dell'interfaccia CLI): rinominati il pacchetto Python, i comandi,
  le variabili d'ambiente, lo scheme delle risorse MCP, il namespace degli `$id` degli schema
  JSON e l'identificatore di spec. Nessuna modifica al comportamento dei gate di validazione né al
  contratto `OUTPUT_FORMAT_VERSION` (resta `"1.0"`): un grafo valido resta valido dopo aver
  aggiornato `meta.spec_version`. Tabella di migrazione per chi aggiorna da `2.0.0`:

  | Prima | Ora |
  |---|---|
  | comando `md-drd` | comando `themis` |
  | comando `md-drd-mcp` | comando `themis-mcp` |
  | `MD_DRD_LANG` | `THEMIS_LANG` |
  | `MD_DRD_GRAPH_PATH` | `THEMIS_GRAPH_PATH` |
  | `MD_DRD_LICENSE_FILE` | `THEMIS_LICENSE_FILE` |
  | `MD_DRD_ACTIVATION_URL` | `THEMIS_ACTIVATION_URL` |
  | `MD_DRD_TOOLKIT_TOKEN` | `THEMIS_TOOLKIT_TOKEN` |
  | risorse MCP `md-drd://...` | `themis://...` |
  | `meta.spec_version: "MD-DRD-SPEC-001/1.0"` | `"THEMIS-SPEC-001/1.0"` |
  | licenza in `~/.md-drd/license.json` | `~/.themis/license.json` |
  | repo pubblico `md-drd` | `themis` |
  | servizio `md-drd-licensing` | `themis-licensing` |

### Aggiunto
- `themis license activate --token <token> [--url endpoint]` (F7.T4): attivazione automatica
  della licenza contattando il servizio di provisioning (repo separata `themis-licensing` su
  Vercel — Stripe + Supabase + firma Ed25519 remota). Unico punto di rete di tutto il toolkit, e
  solo per questo provisioning una tantum: la verifica a runtime (`enforce()`) resta sempre
  offline. Le macchine CI restano fuori da questo flusso (fingerprint instabile sui runner
  effimeri, F7.T4.5): per loro l'attivazione resta manuale via email. *(Voce mancante da questo
  file al momento del merge, aggiunta ora insieme al resto di questa sessione — CHANGELOG.md era
  disallineato dallo stato reale esattamente come `docs/ONBOARDING.md`, vedi sotto.)*
- Modello di prezzo (F7.T4.2): **10€/mese per postazione**, abbonamento — non un acquisto una
  tantum. `themis-licensing` aggiornato di conseguenza: Stripe Checkout in modalità
  sottoscrizione, `expires_at` non più calcolato una volta da `metadata.months` ma riesteso al
  `current_period_end` a ogni fattura pagata (`invoice.paid`). Nessuna revoca attiva prevista o
  necessaria: se l'abbonamento è cancellato o un pagamento fallisce, la licenza smette di
  allungarsi e scade da sola alla fine dell'ultimo periodo pagato, coerente con la verifica
  sempre offline di `license.py`.
- Consegna automatica del binario dopo il pagamento (F7.T4.8): `public/success.html` (repo
  `themis-licensing`) mostra ora, oltre al token di attivazione, i link di download per la
  piattaforma rilevata verso l'ultima release del nuovo repo pubblico `themis` — chiude la parte
  ancora manuale del flusso di acquisto (fino a questa sessione il cliente riceveva solo il
  comando di attivazione, non il programma).
- Repo pubblico [`themis`](https://github.com/manuel-albanesee/themis) (F7.T1.5): landing page di
  prodotto, l'intera documentazione cliente (`docs/`, `EULA.md`, `CHANGELOG.md`) e le release
  Nuitka scaricabili pubblicamente — prima erano solo su una GitHub Release del repo privato,
  raggiungibile solo da chi aveva accesso a quel repo. `pyproject.toml`: aggiunto `Documentation`
  a `[project.urls]`, deliberatamente assente finché questo repo non esisteva.
- `docs/THEMIS-SPEC-001.md` (F7.T1.6): ricostruzione ad-hoc, in forma normativa, della specifica
  che il toolkit implementa e cita ovunque nel codice (`§2`, `§3.1`, `§5..§14`, `§16.2`, ...) —
  non la fonte originale (mai esistita nel repo), ma coerente con ogni citazione di sezione già
  presente. Pubblicata nel repo `themis` insieme a `docs/RULEBOOK.md`/`docs/SCHEMA.md`.
- `.github/workflows/validate-graph.yml` (F6.T3.1): workflow riusabile (`workflow_call`) che
  qualunque altro repository può richiamare per validare il proprio grafo — installa il toolkit
  da sorgente (nessun file di licenza richiesto) ed esegue `validate`, opzionalmente
  `coverage-diff` contro una baseline. Esempio di workflow chiamante e passo manuale richiesto
  (token di accesso al repo privato) in `docs/AGENT-INTEGRATION.md`.
- `themis ci-init [dir] --graph grafo.yaml [--runner self-hosted]` (F7.T4.6): nuovo sottocomando
  che scrive un workflow GitHub Actions autonomo (`.github/workflows/themis.yml`) per la CI di un
  cliente su un proprio runner self-hosted — nessun riferimento al sorgente privato del toolkit,
  nessuna chiamata di rete. Copre il percorso CI descritto in F7.T4.5, senza risolvere il gap dei
  runner effimeri (GitHub-hosted) ancora aperto in quel task.
- `themis hooks-init [dir] [--graph grafo.yaml]`, più `--with-hooks` su `themis init` (F7.T4.7):
  scrive hook git nativi (`.githooks/pre-push` bloccante, `.githooks/post-merge` solo di avviso)
  sulla macchina di sviluppo del cliente — nessun problema di fingerprint della licenza (è già la
  sua macchina), utile come rete di sicurezza anche quando non esiste alcun runner CI. Non un
  sostituto di `ci-init`: un hook locale è bypassabile e non può bloccare il merge di una PR.
- `docs/ONBOARDING.md`: guida unica per un nuovo cliente, dal download del binario alla licenza,
  allo scaffolding del grafo — sia su un progetto già avviato (innesto in un repo esistente,
  import da `.tex`/skill di authoring) sia su uno ancora da avviare — fino all'uso quotidiano
  (CLI vs server MCP) e ai comandi git successivi (citazione degli id, `trace-code` pre-PR,
  `ci-init`/`hooks-init`, workflow CI riusabile). Collegata dal README (§ Documentazione e §
  Installazione).

### Corretto
- `docs/ONBOARDING.md`: la sezione di attivazione della licenza descriveva solo il flusso manuale
  (fingerprint → email → file firmato a mano), disallineata da quando `license activate --token`
  è stato introdotto (F7.T4, sopra) senza aggiornare questo documento. Riscritta: §1 copre
  acquisto+download dal nuovo repo pubblico, §2a il flusso automatico (caso normale), §2b il
  flusso manuale via email (ora esplicitamente limitato alla CI su runner effimeri, l'unico caso
  in cui serve ancora). Stesso allineamento nel README (sezione "Installazione").
- Pipeline di release Nuitka (`_nuitka-build-and-verify.yml`): il check anti-licenza assumeva il
  CLI di `themis` (che risponde `--help` con exit `0`) anche per `themis-mcp`, che non ha
  sottocomandi — 3 dei 6 job del primo run reale su `v2.0.0` fallivano scambiando l'assenza di
  `--help` per una licenza mancante.
- Pipeline di release Nuitka su Windows: un'interpolazione diretta di `$RUNNER_TEMP` in uno
  script Python leggeva il backslash del percorso Windows come sequenza di escape.

## [2.0.0] - 2026-09-08

Prima versione del prodotto pensata per essere distribuita e venduta pubblicamente. Il numero
2.0.0 riflette il lavoro svolto durante lo sviluppo interno (il file `themis/_version.py` è
partito da questo numero fin dalla sua introduzione), non un incremento rispetto a una 1.0.0 mai
distribuita a un cliente esterno: prima di F0, nessuna versione è mai stata rilasciata
pubblicamente.

### Aggiunto

**Motore di validazione e proiettori**
- Validazione del grafo canonico dei requisiti (THEMIS-SPEC-001) contro schema JSON e 73
  controlli automatizzati attivi su 10 gate deterministici (`G0`..`G9`), 10 identificatori
  riservati per compatibilità con la numerazione della spec.
- Proiettori: matrice di tracciabilità (RTM), report di conformità, registro di governance,
  backlog in CSV, export ReqIF, brief di lavoro per agente.
- `coverage-diff` (copertura fra due versioni del grafo), `verify-sources` (hash delle fonti),
  `trace-code` (tracciabilità inversa codice → grafo, incluso indice di riferimento inverso e
  filtro `--files`), `import-tex` (importazione da documenti sorgente `.tex` FSD/TSD).

**Interfaccia agent-native (F1)**
- Output JSON strutturato per ogni comando (`--format json`), envelope di errore uniforme,
  `OUTPUT_FORMAT_VERSION` come contratto esplicito verso i consumatori dell'output.
- Rulebook macchina-leggibile del catalogo controlli (`docs/RULEBOOK.md`, generato da
  `themis/data/rules.yaml`, non scritto a mano).

**Ciclo di authoring (F2)**
- `themis init` per generare uno scaffold di progetto da zero.
- Skill di authoring per generare un grafo Themis a partire da un documento sorgente.
- Ciclo `validate` → `fix-plan` → correzione, pensato per un agente di coding, non solo per un
  umano.

**Server MCP (F3)**
- `themis-mcp`: `validate`, `fix-plan`, `brief`, `trace-code`, `coverage-diff`, `verify-sources`
  esposti come tool MCP, per l'uso diretto da un agente compatibile con il protocollo.

**Internazionalizzazione (F4)**
- Interfaccia agente e documenti proiettati (`project`, `brief`) bilingue via `--lang`,
  `THEMIS_LANG`, `meta.language`.

**Distribuzione e licenza (F0, in chiusura)**
- Verifica offline della licenza (Ed25519, `themis/license.py`) applicata solo nella build
  compilata Nuitka — no-op nel checkout di sviluppo e in tutta la suite di test.
- `scripts/issue_license.py`/`scripts/generate_signing_keypair.py` per l'emissione delle licenze
  lato venditore; chiave privata mai versionata né presente in CI.
- `scripts/build_nuitka.py` e pipeline CI di smoke test (`build-nuitka.yml`) con verifica
  anti-bypass del binario compilato (nessun sorgente Python in chiaro accanto al binario, rifiuto
  verificato sia per licenza assente sia per firma non valida).
- `THIRD-PARTY-LICENSES.md`, generato deterministicamente su ogni OS di sviluppo
  (`scripts/generate_third_party_notices.py`).
- `EULA.md` (bozza).

[3.0.0]: https://github.com/manuel-albanesee/themis-toolkit/compare/v2.0.0...v3.0.0
[2.0.0]: https://github.com/manuel-albanesee/themis-toolkit/releases/tag/v2.0.0
