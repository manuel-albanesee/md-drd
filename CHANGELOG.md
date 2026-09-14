# Changelog

Le modifiche rilevanti di questo progetto sono documentate in questo file.

Il formato segue [Keep a Changelog](https://keepachangelog.com/it/1.0.0/). Il numero di versione
segue [Semantic Versioning](https://semver.org/lang/it/) ed è quello del **toolkit**: non va
confuso con la versione dello schema del grafo (`meta.spec_version`) né con il contratto del
formato di output JSON (`OUTPUT_FORMAT_VERSION`), versionati separatamente — vedi
`docs/VERSIONING.md`.

## [Unreleased]

### Aggiunto
- `md-drd license activate --token <token> [--url endpoint]` (F7.T4): attivazione automatica
  della licenza contattando il servizio di provisioning (repo separata `md-drd-licensing` su
  Vercel — Stripe + Supabase + firma Ed25519 remota). Unico punto di rete di tutto il toolkit, e
  solo per questo provisioning una tantum: la verifica a runtime (`enforce()`) resta sempre
  offline. Le macchine CI restano fuori da questo flusso (fingerprint instabile sui runner
  effimeri, F7.T4.5): per loro l'attivazione resta manuale via email. *(Voce mancante da questo
  file al momento del merge, aggiunta ora insieme al resto di questa sessione — CHANGELOG.md era
  disallineato dallo stato reale esattamente come `docs/ONBOARDING.md`, vedi sotto.)*
- Modello di prezzo (F7.T4.2): **10€/mese per postazione**, abbonamento — non un acquisto una
  tantum. `md-drd-licensing` aggiornato di conseguenza: Stripe Checkout in modalità
  sottoscrizione, `expires_at` non più calcolato una volta da `metadata.months` ma riesteso al
  `current_period_end` a ogni fattura pagata (`invoice.paid`). Nessuna revoca attiva prevista o
  necessaria: se l'abbonamento è cancellato o un pagamento fallisce, la licenza smette di
  allungarsi e scade da sola alla fine dell'ultimo periodo pagato, coerente con la verifica
  sempre offline di `license.py`.
- Consegna automatica del binario dopo il pagamento (F7.T4.8): `public/success.html` (repo
  `md-drd-licensing`) mostra ora, oltre al token di attivazione, i link di download per la
  piattaforma rilevata verso l'ultima release del nuovo repo pubblico `md-drd` — chiude la parte
  ancora manuale del flusso di acquisto (fino a questa sessione il cliente riceveva solo il
  comando di attivazione, non il programma).
- Repo pubblico [`md-drd`](https://github.com/manuel-albanesee/md-drd) (F7.T1.5): landing page di
  prodotto, l'intera documentazione cliente (`docs/`, `EULA.md`, `CHANGELOG.md`) e le release
  Nuitka scaricabili pubblicamente — prima erano solo su una GitHub Release del repo privato,
  raggiungibile solo da chi aveva accesso a quel repo. `pyproject.toml`: aggiunto `Documentation`
  a `[project.urls]`, deliberatamente assente finché questo repo non esisteva.
- `docs/MD-DRD-SPEC-001.md` (F7.T1.6): ricostruzione ad-hoc, in forma normativa, della specifica
  che il toolkit implementa e cita ovunque nel codice (`§2`, `§3.1`, `§5..§14`, `§16.2`, ...) —
  non la fonte originale (mai esistita nel repo), ma coerente con ogni citazione di sezione già
  presente. Pubblicata nel repo `md-drd` insieme a `docs/RULEBOOK.md`/`docs/SCHEMA.md`.
- `.github/workflows/validate-graph.yml` (F6.T3.1): workflow riusabile (`workflow_call`) che
  qualunque altro repository può richiamare per validare il proprio grafo — installa il toolkit
  da sorgente (nessun file di licenza richiesto) ed esegue `validate`, opzionalmente
  `coverage-diff` contro una baseline. Esempio di workflow chiamante e passo manuale richiesto
  (token di accesso al repo privato) in `docs/AGENT-INTEGRATION.md`.
- `md-drd ci-init [dir] --graph grafo.yaml [--runner self-hosted]` (F7.T4.6): nuovo sottocomando
  che scrive un workflow GitHub Actions autonomo (`.github/workflows/md-drd.yml`) per la CI di un
  cliente su un proprio runner self-hosted — nessun riferimento al sorgente privato del toolkit,
  nessuna chiamata di rete. Copre il percorso CI descritto in F7.T4.5, senza risolvere il gap dei
  runner effimeri (GitHub-hosted) ancora aperto in quel task.
- `md-drd hooks-init [dir] [--graph grafo.yaml]`, più `--with-hooks` su `md-drd init` (F7.T4.7):
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
  CLI di `md-drd` (che risponde `--help` con exit `0`) anche per `md-drd-mcp`, che non ha
  sottocomandi — 3 dei 6 job del primo run reale su `v2.0.0` fallivano scambiando l'assenza di
  `--help` per una licenza mancante.
- Pipeline di release Nuitka su Windows: un'interpolazione diretta di `$RUNNER_TEMP` in uno
  script Python leggeva il backslash del percorso Windows come sequenza di escape.

## [2.0.0] - 2026-09-08

Prima versione del prodotto pensata per essere distribuita e venduta pubblicamente. Il numero
2.0.0 riflette il lavoro svolto durante lo sviluppo interno (il file `md_drd/_version.py` è
partito da questo numero fin dalla sua introduzione), non un incremento rispetto a una 1.0.0 mai
distribuita a un cliente esterno: prima di F0, nessuna versione è mai stata rilasciata
pubblicamente.

### Aggiunto

**Motore di validazione e proiettori**
- Validazione del grafo canonico dei requisiti (MD-DRD-SPEC-001) contro schema JSON e 73
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
  `md_drd/data/rules.yaml`, non scritto a mano).

**Ciclo di authoring (F2)**
- `md-drd init` per generare uno scaffold di progetto da zero.
- Skill di authoring per generare un grafo MD-DRD a partire da un documento sorgente.
- Ciclo `validate` → `fix-plan` → correzione, pensato per un agente di coding, non solo per un
  umano.

**Server MCP (F3)**
- `md-drd-mcp`: `validate`, `fix-plan`, `brief`, `trace-code`, `coverage-diff`, `verify-sources`
  esposti come tool MCP, per l'uso diretto da un agente compatibile con il protocollo.

**Internazionalizzazione (F4)**
- Interfaccia agente e documenti proiettati (`project`, `brief`) bilingue via `--lang`,
  `MD_DRD_LANG`, `meta.language`.

**Distribuzione e licenza (F0, in chiusura)**
- Verifica offline della licenza (Ed25519, `md_drd/license.py`) applicata solo nella build
  compilata Nuitka — no-op nel checkout di sviluppo e in tutta la suite di test.
- `scripts/issue_license.py`/`scripts/generate_signing_keypair.py` per l'emissione delle licenze
  lato venditore; chiave privata mai versionata né presente in CI.
- `scripts/build_nuitka.py` e pipeline CI di smoke test (`build-nuitka.yml`) con verifica
  anti-bypass del binario compilato (nessun sorgente Python in chiaro accanto al binario, rifiuto
  verificato sia per licenza assente sia per firma non valida).
- `THIRD-PARTY-LICENSES.md`, generato deterministicamente su ogni OS di sviluppo
  (`scripts/generate_third_party_notices.py`).
- `EULA.md` (bozza).

[Unreleased]: https://github.com/manuel-albanesee/md-drd-toolkit/compare/v2.0.0...HEAD
[2.0.0]: https://github.com/manuel-albanesee/md-drd-toolkit/releases/tag/v2.0.0
