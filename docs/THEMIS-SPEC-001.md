# THEMIS-SPEC-001 — Specifica del grafo canonico di requisiti

**Versione:** 1.0
**Stato:** normativa, in vigore per `themis` >= 2.0.0

> **Nota sulla provenienza di questo documento.** Il toolkit `themis` cita questa specifica in
> tutto il codice (`§2`, `§3.1`, `§5..§14`, `§16.2`, ...) fin dalla sua prima versione interna, ma
> il testo non era mai stato pubblicato: chi comprava il prodotto ne vedeva l'applicazione
> (`docs/RULEBOOK.md`, i messaggi di `themis validate`) senza poterne leggere la fonte. Questo
> documento è la sua pubblicazione: ricostruisce, in forma normativa (clausole "DEVE"), esattamente
> le regole che i 10 gate implementano — la numerazione delle sezioni è quella già citata nel
> codice, non una rinumerazione successiva. `docs/RULEBOOK.md` (motivazione, remediation, esempi
> per ognuna delle 73 regole) e `docs/SCHEMA.md` (struttura campo per campo) restano i documenti di
> riferimento per il dettaglio operativo; questo documento è la fonte normativa a cui entrambi
> rispondono.

## 1. Scopo e ambito

Themis definisce un **grafo canonico** di requisiti — bisogno → requisito → architettura → work
package → test — come unica fonte di verità di un progetto software, in sostituzione di una
specifica scritta in prosa.
Il grafo è conforme a questa specifica se e solo se supera tutti i controlli **bloccanti** dei 10
gate deterministici `G0`..`G9` definiti alle sezioni 5-14. Un'implementazione conforme (come
`themis`) verifica meccanicamente ogni clausola qui sotto; nessuna clausola è lasciata al giudizio
di chi legge.

Questa specifica è agnostica rispetto al dominio applicativo: si applica a qualunque progetto che
adotti il grafo canonico, indipendentemente dal settore.

## 2. Il grafo canonico come fonte di verità

Il grafo è un documento strutturato (oggi: YAML conforme allo schema pubblico, `docs/SCHEMA.md`)
composto dalle entità elencate al §4. Ogni entità del grafo che afferma un fatto derivato da un
documento esterno (requisito, bisogno, decisione) **DEVE** poter essere ricondotta a una fonte
tracciata (§5, `source_documents`/`source_segments`).

**Integrità delle fonti.** Ogni documento sorgente indicizzato nel grafo **DEVE** restare
verificabile contro il proprio contenuto originale: il grafo ne congela un hash al momento
dell'indicizzazione: (`source_documents[].hash`), e un'implementazione conforme **DEVE** offrire un
modo di verificare che il file sorgente non sia stato alterato dopo l'indicizzazione senza
aggiornare il grafo di conseguenza. Questo controllo è indipendente dai gate `G0`..`G9`, che
operano solo sul grafo: verifica il grafo *contro* le fonti, non le fonti fra loro.

## 3. Identificatori

Ogni entità del grafo ha un identificatore univoco e una forma canonica fissa per tipo (prefisso +
separatori + cifre), non libera.

### 3.1 Forme canoniche

Un identificatore **DEVE** rispettare la forma canonica del proprio tipo:

| Tipo | Forma | Esempio |
|---|---|---|
| Documento sorgente | `SRC-D<n>-<tag>-<seq3>` | `SRC-D1-contratto-001` |
| Stakeholder | `STK-<seq3>` | `STK-001` |
| Bisogno | `NEED-<seq3>` | `NEED-001` |
| Obiettivo | `OBJ-<seq3>` | `OBJ-001` |
| Requisito funzionale | `RF-<seq3>` | `RF-001` |
| Requisito di qualità | `RQ-<seq3>` | `RQ-001` |
| Vincolo | `RV-<seq3>` | `RV-001` |
| Requisito d'interfaccia | `RI-<seq3>` | `RI-001` |
| Requisito derivato | `RD-<seq3>` | `RD-001` |
| Regola di transizione | `RT-<seq3>` | `RT-001` |
| Elemento architetturale | `ARC-{CTX\|CNT\|CMP\|EXT}-<seq3>` | `ARC-CMP-001` |
| Decisione architetturale | `ADR-<seq3>` | `ADR-001` |
| Deliverable | `DLV-<seq3>` | `DLV-001` |
| Work package | `WP-<seq3>` | `WP-001` |
| User story | `US-<seq3>` | `US-001` |
| Test case | `TC-<seq3>` | `TC-001` |
| Rischio | `RSK-<seq3>` | `RSK-001` |
| Assunzione | `ASM-<seq3>` | `ASM-001` |
| Punto aperto | `OP-<seq2>` | `OP-01` |
| Fase | `PH-<n>` | `PH-1` |
| Release | `REL-<n>` | `REL-1` |
| Viewpoint | `VP-<seq2>` | `VP-01` |
| Tradeoff | `TO-<seq3>` | `TO-001` |

Un identificatore che non rispetta la propria forma canonica, o che compare più di una volta nel
grafo, è un difetto bloccante (§14, `G9.6`). La stessa tabella è la fonte unica sia per la
conformità di schema degli id sia per la tracciabilità inversa codice→grafo (citazione degli id nel
codice sorgente del progetto che implementa il grafo).

## 4. Entità del grafo

Il grafo è composto dalle entità elencate qui sotto, ciascuna con i propri campi obbligatori —
struttura completa, campo per campo, in `docs/SCHEMA.md`: documenti e segmenti sorgente,
stakeholder, obiettivi, bisogni, requisiti (funzionali, di qualità, vincoli, interfaccia, derivati),
viewpoint, elementi architetturali, decisioni architetturali (ADR), tradeoff, deliverable, work
package, user story, test case, rischi, assunzioni, punti aperti, fasi, release, richieste di
modifica, glossario.

## 5. G0 — Intake e normalizzazione delle fonti

Prima di estrarre alcunché, ogni documento e segmento sorgente **DEVE** essere censito e
qualificato:

| ID | Il grafo DEVE garantire che |
|---|---|
| G0.1 | ogni documento sorgente dichiari un hash di integrità e un livello di autorevolezza |
| G0.2 | ogni segmento sorgente sia indicizzabile (riferimento univoco a documento e posizione) |
| G0.3 | nessun segmento sorgente sia vuoto |
| G0.4 | almeno un segmento sorgente sia indicizzato prima di procedere all'estrazione |
| G0.5 | ogni segmento sorgente sia classificato (prescrittivo, informativo, indeterminato, ...) |
| G0.6 | ogni acronimo usato nelle fonti compaia nel glossario del grafo |

## 6. G1 — Business / Mission Analysis

| ID | Il grafo DEVE garantire che |
|---|---|
| G1.1 | ogni categoria di stakeholder prevista sia rappresentata da almeno uno stakeholder |
| G1.2 | ogni stakeholder abbia almeno un concern e ogni bisogno sia riconducibile a uno stakeholder |
| G1.3 | ogni obiettivo (`objective`) sia misurabile |
| G1.4 | il confine del sistema (in/out of scope) sia dichiarato esplicitamente |
| G1.6 | ogni bisogno o obiettivo derivato riporti una giustificazione della derivazione |

*(`G1.5` riservato, nessun controllo attivo in questa versione.)*

## 7. G2 — Estrazione grezza dei requisiti

| ID | Il grafo DEVE garantire che |
|---|---|
| G2.1 | ogni requisito estratto citi una fonte valida (un segmento sorgente indicizzato, §5) |
| G2.3 | ogni segmento sorgente prescrittivo generi almeno un requisito o un punto aperto |
| G2.5 | ogni segmento indeterminato generi un punto aperto tracciato |
| G2.7 | la copertura dei segmenti prescrittivi (trasformati in requisiti/punti aperti) superi la soglia minima configurata |

*(`G2.2`, `G2.4`, `G2.6` riservati.)*

## 8. G3 — Normalizzazione e scrittura dei requisiti

| ID | Il grafo DEVE garantire che |
|---|---|
| G3.1 | ogni requisito sia scritto secondo il pattern EARS dichiarato |
| G3.2 | l'enunciato di un requisito non contenga lessico non verificabile (vago/ambiguo) |
| G3.4 | ogni requisito esprima un solo vincolo/comportamento (atomicità) |
| G3.5 | ogni requisito riporti una motivazione (rationale) |
| G3.6 | ogni requisito di qualità sia classificato secondo una caratteristica ISO/IEC 25010:2023 |
| G3.7 | ogni caratteristica di qualità applicabile al progetto sia coperta da almeno un requisito |
| G3.8 | ogni conflitto dichiarato fra requisiti indichi la sede/il processo di risoluzione |
| G3.10 | ogni requisito derivato riporti una nota di derivazione |

*(`G3.3`, `G3.9` riservati.)*

## 9. G4 — Verificabilità e criteri di accettazione

| ID | Il grafo DEVE garantire che |
|---|---|
| G4.1 | ogni requisito dichiari un metodo di verifica (test, ispezione, analisi, dimostrazione) |
| G4.2 | ogni requisito abbia almeno un criterio di accettazione |
| G4.3 | ogni criterio con soglia numerica includa un caso limite (boundary) |
| G4.4 | ogni requisito abbia almeno un criterio di accettazione negativo |
| G4.5 | ogni requisito di qualità abbia una scheda di misura completa (metrica, metodo, soglia) |
| G4.6 | nessun criterio di accettazione contenga lessico vietato/non verificabile |
| G4.7 | ogni criterio numerico dichiari unità di misura e valore |

## 10. G5 — Descrizione architetturale

| ID | Il grafo DEVE garantire che |
|---|---|
| G5.1 | ogni viewpoint dichiarato indirizzi almeno un concern reale |
| G5.2 | ogni concern sia indirizzato da almeno un viewpoint |
| G5.3 | ogni requisito sia allocato a un elemento architetturale, senza allocazioni pendenti alla baseline |
| G5.4 | ogni elemento architetturale abbia almeno un requisito allocato |
| G5.6 | nessuna dipendenza fra elementi architetturali resti pendente alla baseline |
| G5.7 | ogni ADR dichiari la condizione che l'ha resa necessaria (trigger) |
| G5.8 | ogni ADR sia completa (contesto, opzioni valutate, decisione, conseguenze) |
| G5.9 | ogni sistema esterno abbia almeno un requisito di interfaccia associato |

*(`G5.5` riservato.)*

## 11. G6 — Decomposizione del lavoro

| ID | Il grafo DEVE garantire che |
|---|---|
| G6.1 | la WBS rispetti la regola del 100% (i nodi figli coprono interamente il nodo padre, senza sovrapposizioni) |
| G6.2 | ogni nodo WBS (deliverable) sia denominato con un sostantivo, non un verbo |
| G6.3 | ogni work package abbia un effort stimato fra 8 e 80 ore |
| G6.4 | ogni requisito sia realizzato da almeno un work package |
| G6.5 | ogni work package realizzativo realizzi almeno un requisito |
| G6.6 | ogni esclusione di perimetro sia dichiarata esplicitamente |
| G6.7 | ogni work package abbia una Definition of Done sufficientemente specifica |
| G6.8 | ogni classe di deliverable prevista sia presente o esplicitamente esclusa |
| G6.9 | il piano preveda un walking skeleton end-to-end sull'intera spina dorsale del sistema |
| G6.10 | ogni user story soddisfi i criteri INVEST |

## 12. G7 — Rischio, dipendenze e sequenziamento

| ID | Il grafo DEVE garantire che |
|---|---|
| G7.1 | il grafo delle dipendenze fra work package sia valido (aciclico, riferimenti risolti) |
| G7.2 | ogni punto aperto abbia un rischio associato |
| G7.3 | ogni assunzione non ancora validata abbia un rischio associato |
| G7.4 | ogni rischio critico abbia una risposta pianificata |
| G7.5 | ogni spike dichiari timebox, prodotto atteso e decision gate |
| G7.6 | nessun work package dipendente sia avviato prima della conclusione dello spike da cui dipende |
| G7.7 | ogni rischio abbia un titolare, un indicatore di innesco e una strategia di risposta |
| G7.8 | l'incertezza di stima (PERT, §12.6) di ogni work package resti entro un intervallo gestibile |
| G7.9 | ogni dipendenza fra work package sia classificata per tipo |
| G7.10 | il cammino critico (CPM, §12.7) del piano sia determinabile in modo univoco |

**§12.6 — Stima PERT.** L'effort di ogni work package è stimato con la formula PERT a tre punti
(ottimistico, più probabile, pessimistico); la deviazione standard risultante è la misura di
incertezza di stima usata da `G7.8`.

**§12.7 — Cammino critico (CPM).** Il cammino critico è calcolato sul grafo delle dipendenze fra
work package (§7.1) con il metodo del cammino critico standard (forward/backward pass); usato da
`G7.10` e nelle proiezioni di piano (§14).

## 13. G8 — Prioritizzazione e piano di rilascio

| ID | Il grafo DEVE garantire che |
|---|---|
| G8.1 | ogni requisito abbia una priorità MoSCoW assegnata |
| G8.2 | l'effort complessivo dei requisiti Must di una release non superi la quota massima configurata |
| G8.3 | ogni requisito derivato da un vincolo legale o regolatorio sia classificato Must |
| G8.4 | l'obiettivo di una release sia espresso in termini di valore, non come elenco di componenti tecnici |
| G8.5 | il piano definisca un MVP che soddisfi i propri criteri di validazione |
| G8.6 | ogni release dichiari esplicitamente cosa NON include |
| G8.7 | ogni fase del piano abbia criteri di ingresso e di uscita |
| G8.8 | nessun work package dipenda da un predecessore pianificato in una release successiva alla propria |

## 14. G9 — Verifica di conformità finale

| ID | Il grafo DEVE garantire che |
|---|---|
| G9.2 | ogni requisito a monte sia coperto da almeno un elemento a valle (nessun orfano discendente) |
| G9.3 | ogni elemento a valle sia giustificato da almeno un elemento a monte (nessun gold plating) |
| G9.4 | ogni indeterminatezza residua sia registrata come punto aperto tracciato |
| G9.5 | nessun conflitto dichiarato resti irrisolto al momento della baseline |
| G9.6 | ogni identificatore sia univoco e conforme alla propria forma canonica (§3.1) |
| G9.7 | nessun riferimento fra entità del grafo resti pendente (puntatore a un id inesistente) |
| G9.10 | una baseline sia identificata univocamente e le fonti che la compongono siano congelate (hash bloccato, §2) |

*(`G9.1`, `G9.8`, `G9.9` riservati.)*

**Documenti proiettati.** Un grafo conforme a tutti i gate `G0`..`G9` **DEVE** poter generare,
senza intervento manuale, i seguenti documenti derivati — mai editati a mano, sempre rigenerati dal
grafo:

- **§14.1 — Matrice di tracciabilità (RTM)**: bisogno → requisito → architettura → work package →
  test, in entrambe le direzioni (copertura discendente, `G9.2`; giustificazione ascendente,
  `G9.3`).
- **§14.2 — Report di conformità e metriche**: esito dei 73 controlli, stima PERT (§12.6), cammino
  critico CPM (§12.7), esposizione al rischio.
- **§14.5 — Interscambio (ReqIF)**: esportazione in formato ReqIF per strumenti terzi di gestione
  requisiti.

Oltre a questi: registro di governance e backlog prioritizzato, entrambi derivati dal grafo con lo
stesso vincolo di non editabilità manuale.

## 15. Localizzazione e messaggistica

Ogni messaggio rivolto a chi legge l'esito della validazione (difetti, remediation, help dei
comandi) **DEVE** essere risolto da un catalogo versionato per lingua, non generato inline: un
difetto è identificato da un codice macchina stabile (severità: `blocking`, `major`, `minor`,
`info`), una chiave di catalogo, e un puntatore JSON (RFC 6901) alla posizione esatta nel grafo.
Questo separa il contenuto (stabile, testabile) dalla sua traduzione (estendibile senza toccare la
logica dei gate).

## 16. Principi di validazione

**§16.1 — Determinismo.** La conformità di un grafo a questa specifica è decisa esclusivamente da
regole deterministiche (espressioni regolari sul lessico, calcolo di grafo per gli orfani,
PERT/CPM sul piano, la regola del 100% sulla WBS) — mai da un giudizio linguistico probabilistico.
Uno stesso grafo produce sempre lo stesso esito, a parità di versione della specifica.

**§16.2 — I gate sono codice, non prompt.** Un'entità (umana o un modello linguistico) che genera
il grafo non può essere anche l'unico giudice della propria conformità: la verifica è un processo
separato, eseguibile in modo riproducibile da chiunque, indipendente da chi ha scritto il
contenuto. Un agente che genera contenuto per il grafo resta soggetto agli stessi 10 gate di
chiunque altro.
