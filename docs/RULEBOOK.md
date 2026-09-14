# Catalogo dei controlli MD-DRD (versione 1.0, MD-DRD-SPEC-001/1.0)

> Documento **generato** da `md_drd/data/rules.yaml` con `md-drd rules --markdown`.
> Non modificare a mano: la fonte e' il catalogo.

73 controlli implementati, 10 identificatori riservati, in 10 gate.

## G0 — Intake e normalizzazione delle fonti

### G0.1 — Documento sorgente senza hash o autorevolezza

**Entita'**: source_documents · **bloccante**

Senza hash il grafo non e' riconducibile a una versione precisa della fonte; senza livello di autorevolezza non si puo' dirimere un conflitto fra documenti.

**Come correggere** — Valorizzare `hash_sha256` con lo SHA-256 del file (64 esadecimali minuscoli) e `authority` con normative | indicative | informative.

**Esempio non conforme**

```yaml
- id: D1
  title: Analisi funzionale
```

**Esempio corretto**

```yaml
- id: D1
  title: Analisi funzionale
  hash_sha256: 3b1f...c9
  authority: normative
```

### G0.2 — Segmento sorgente non indicizzabile

**Entita'**: source_segments, source_documents · **bloccante**

Un segmento e' l'unita' minima di tracciabilita': deve essere identificabile in modo univoco e localizzabile nel documento di provenienza.

**Come correggere** — Assegnare un id univoco nella forma SRC-D<n>-<sezione>-<progressivo a 3 cifre>, valorizzare `section_path` e puntare con `document_id` a un documento esistente.

**Esempio non conforme**

```yaml
- id: SRC-D1-3.2-001
  document_id: D9   # D9 non esiste
```

**Esempio corretto**

```yaml
- id: SRC-D1-3.2-001
  document_id: D1
  section_path: '3.2 Raccolta dati'
```

### G0.3 — Segmento sorgente vuoto

**Entita'**: source_segments · **bloccante**

Un segmento senza testo non e' verificabile: nessuno puo' controllare che il requisito derivato dica davvero cio' che dice la fonte.

**Come correggere** — Riportare in `text` il testo integrale del segmento (almeno 3 caratteri), senza parafrasarlo.

**Esempio non conforme**

```yaml
  text: ''
```

**Esempio corretto**

```yaml
  text: 'Il sistema deve registrare ogni accesso.'
```

### G0.4 — Nessun segmento sorgente indicizzato

**Entita'**: source_segments · **bloccante**

L'intero metodo poggia sull'indicizzazione della fonte: senza segmenti la copertura non e' calcolabile e ogni requisito e' di fatto non tracciato.

**Come correggere** — Segmentare i documenti sorgente in `source_segments` prima di estrarre i requisiti.

**Esempio non conforme**

```yaml
source_segments: []
```

**Esempio corretto**

```yaml
source_segments:
  - id: SRC-D1-1-001
    ...
```

### G0.5 — Segmento privo di classificazione

**Entita'**: source_segments · **bloccante**

L'etichetta decide se un segmento e' prescrittivo (deve generare requisiti) o descrittivo: senza di essa la copertura non ha significato.

**Come correggere** — Valorizzare `labels` con almeno una fra prescriptive, descriptive, rationale, constraint, option, open, reference, example.

**Esempio non conforme**

```yaml
  labels: []
```

**Esempio corretto**

```yaml
  labels: [prescriptive]
```

### G0.6 — Acronimo non presente a glossario

**Entita'**: glossary, source_segments · **maggiore**

Un acronimo non definito e' ambiguo per chi implementa e per chi verifica; il glossario e' l'unico punto in cui il significato e' vincolante.

**Come correggere** — Aggiungere il termine a `glossary` con la sua definizione, oppure registrarlo come alias di un termine gia' presente.

**Esempio non conforme**

```yaml
testo del segmento: 'esporre le API REST'  # REST non a glossario
```

**Esempio corretto**

```yaml
glossary:
  - term: REST
    definition: Representational State Transfer
```

## G1 — Business / Mission Analysis

### G1.1 — Categoria di stakeholder non rappresentata

**Entita'**: stakeholders · **bloccante**

Le categorie obbligatorie coprono i punti di vista che, se ignorati, producono requisiti sbilanciati: chi usa, chi possiede, chi opera, chi fornisce, chi subisce.

**Come correggere** — Aggiungere almeno uno stakeholder per la categoria mancante, oppure motivarne l'assenza registrando un OpenPoint.

**Esempio non conforme**

```yaml
stakeholders: [{category: user}, {category: owner}]
```

**Esempio corretto**

```yaml
stakeholders: [... user, owner, operator, supplier, third_party_affected]
```

### G1.2 — Stakeholder senza concern o bisogno senza stakeholder

**Entita'**: stakeholders, needs · **bloccante**

Un concern e' cio' che rende uno stakeholder rilevante per l'architettura; un bisogno senza titolare non ha nessuno che possa accettarlo.

**Come correggere** — Valorizzare `concerns` su ogni stakeholder e far puntare `stakeholder_id` di ogni need a uno stakeholder esistente.

**Esempio non conforme**

```yaml
- id: NEED-001
  stakeholder_id: STK-999
```

**Esempio corretto**

```yaml
- id: NEED-001
  stakeholder_id: STK-001
```

### G1.3 — Obiettivo non misurabile

**Entita'**: objectives · **bloccante**

Un obiettivo senza metrica, valore obiettivo e metodo di misura non e' falsificabile: a fine progetto nessuno puo' dire se e' stato raggiunto.

**Come correggere** — Valorizzare `metric`, `target_value` e `measurement_method` (e `unit` dove ha senso).

**Esempio non conforme**

```yaml
- id: OBJ-001
  statement: Ridurre i tempi di risposta
```

**Esempio corretto**

```yaml
- id: OBJ-001
  metric: p95 latenza
  target_value: 500
  unit: ms
  measurement_method: telemetria applicativa
```

### G1.4 — Confine di sistema non dichiarato

**Entita'**: arch_elements · **bloccante**

Senza almeno un elemento esterno il confine del sistema e' implicito: e' li' che nascono le sorprese di integrazione e le responsabilita' contese.

**Come correggere** — Aggiungere in `arch_elements` almeno un elemento con `c4_level: external` per ogni sistema con cui si scambiano dati.

**Esempio non conforme**

```yaml
arch_elements: [{c4_level: container}, {c4_level: component}]
```

**Esempio corretto**

```yaml
- id: ARC-EXT-001
  c4_level: external
  name: Anagrafica clienti
```

### G1.5 — _identificatore riservato, controllo non implementato_

### G1.6 — Derivazione non giustificata

**Entita'**: stakeholders, needs · **bloccante**

Un elemento EXTRACTED deve poter essere ricondotto al testo di origine; uno DERIVED/ELICITED deve dichiarare il ragionamento che lo ha prodotto, altrimenti e' indistinguibile da un'invenzione.

**Come correggere** — Valorizzare `source_refs` sugli elementi EXTRACTED e `derivation_note` su quelli DERIVED o ELICITED.

**Esempio non conforme**

```yaml
- id: NEED-002
  derivation: DERIVED
```

**Esempio corretto**

```yaml
- id: NEED-002
  derivation: DERIVED
  derivation_note: dedotto da OBJ-001 e dal concern 'continuita' operativa'
```

## G2 — Estrazione grezza dei requisiti

### G2.1 — Requisito estratto senza fonte valida

**Entita'**: requirements, constraints, source_segments · **bloccante**

Un requisito dichiarato EXTRACTED che non cita il segmento da cui proviene rompe la catena di tracciabilita' verso il documento del committente.

**Come correggere** — Aggiungere in `source_refs` gli id dei segmenti sorgente effettivamente all'origine del requisito, verificando che esistano nel grafo.

**Esempio non conforme**

```yaml
- id: RF-001
  derivation: EXTRACTED
  source_refs: []
```

**Esempio corretto**

```yaml
- id: RF-001
  derivation: EXTRACTED
  source_refs: [SRC-D1-3.2-001]
```

### G2.2 — _identificatore riservato, controllo non implementato_

### G2.3 — Segmento prescrittivo che non ha generato nulla

**Entita'**: source_segments, requirements, constraints · **bloccante**

Un segmento prescrittivo non citato da alcun requisito e' materiale del committente perso per strada: e' il difetto piu' costoso da scoprire in collaudo.

**Come correggere** — Derivare un requisito che citi il segmento in `source_refs`, oppure motivarne l'esclusione valorizzando `excluded_reason` sul segmento.

**Esempio non conforme**

```yaml
- id: SRC-D1-4.1-003
  labels: [prescriptive]   # nessun requisito lo cita
```

**Esempio corretto**

```yaml
- id: SRC-D1-4.1-003
  labels: [prescriptive]
  excluded_reason: fuori perimetro contrattuale, cfr. OP-01
```

### G2.4 — _identificatore riservato, controllo non implementato_

### G2.5 — Segmento indeterminato senza punto aperto

**Entita'**: source_segments, open_points · **bloccante**

Un segmento etichettato `open` e' una domanda ancora senza risposta: se non diventa un OpenPoint con titolare e scadenza, sparisce dal radar.

**Come correggere** — Creare un OpenPoint che citi il segmento in `source_refs`, con `owner`, `resolution_method` e `deadline`.

**Esempio non conforme**

```yaml
- id: SRC-D1-6-002
  labels: [open]
```

**Esempio corretto**

```yaml
open_points:
  - id: OP-01
    source_refs: [SRC-D1-6-002]
    owner: PM
```

### G2.6 — _identificatore riservato, controllo non implementato_

### G2.7 — Copertura dei segmenti prescrittivi sotto soglia

**Entita'**: source_segments, requirements, constraints · **bloccante**

La copertura e' la misura sintetica di quanta parte del materiale del committente e' stata effettivamente recepita: sotto il 95% la baseline non e' difendibile.

**Come correggere** — Coprire con requisiti i segmenti prescrittivi ancora orfani (`md-drd fix-plan` li elenca) oppure escluderli motivandoli.

**Esempio non conforme**

```yaml
copertura 88% con 6 segmenti prescrittivi non citati
```

**Esempio corretto**

```yaml
copertura >= 95% dopo aver derivato i requisiti mancanti
```

## G3 — Normalizzazione e scrittura dei requisiti

### G3.1 — Enunciato non conforme al pattern EARS dichiarato

**Entita'**: requirements · **bloccante**

EARS elimina l'ambiguita' sulla condizione di attivazione: se l'enunciato non segue il pattern dichiarato, la condizione resta implicita e ognuno la interpretera' a modo suo.

**Come correggere** — Allineare l'incipit al pattern: ubiquitous 'Il/La/Lo/I/Le/L'...', event_driven 'Quando...', state_driven 'Mentre...', unwanted 'Se..., allora...', optional 'Dove...'.

**Esempio non conforme**

```yaml
ears_pattern: event_driven
statement: Il sistema deve inviare la notifica.
```

**Esempio corretto**

```yaml
ears_pattern: event_driven
statement: Quando l'ordine e' confermato, il sistema deve inviare la notifica.
```

### G3.2 — Lessico non verificabile nell'enunciato

**Entita'**: requirements, constraints · **contestuale**

Termini come «veloce» o «robusto» non sono falsificabili: producono contenziosi in collaudo perche' ogni parte li interpreta a proprio favore.

**Come correggere** — Sostituire il termine con una grandezza misurabile (metrica, statistica, soglia, unita'). Un marcatore TBD/TBC e' ammesso solo se collegato a un OpenPoint tramite `open_points`.

**Esempio non conforme**

```yaml
statement: Il sistema deve rispondere velocemente.
```

**Esempio corretto**

```yaml
statement: Il sistema deve rispondere entro 500 ms al 95° percentile.
```

### G3.3 — _identificatore riservato, controllo non implementato_

### G3.4 — Requisito non singolare

**Entita'**: requirements · **bloccante**

Un enunciato con piu' obblighi non e' verificabile atomicamente: puo' essere superato a meta', e la tracciabilita' verso test e work package diventa ambigua.

**Come correggere** — Spezzare l'enunciato in requisiti distinti, uno per ciascun «deve»; eliminare congiunzioni come «e deve», «nonche'», «inoltre», «oltre a».

**Esempio non conforme**

```yaml
Il sistema deve registrare l'accesso e deve inviare una notifica.
```

**Esempio corretto**

```yaml
RF-001: Il sistema deve registrare l'accesso.
RF-002: Il sistema deve inviare una notifica di accesso.
```

### G3.5 — Requisito senza motivazione

**Entita'**: requirements · **bloccante**

Senza rationale nessuno puo' decidere, mesi dopo, se il requisito e' ancora valido: e' l'informazione che si perde per prima e serve di piu' in fase di modifica.

**Come correggere** — Valorizzare `rationale` spiegando perche' il requisito esiste (almeno 10 caratteri), non ripetendo l'enunciato.

**Esempio non conforme**

```yaml
- id: RF-003
  rationale: ''
```

**Esempio corretto**

```yaml
- id: RF-003
  rationale: obbligo di audit trail imposto dal vincolo RV-001
```

### G3.6 — Requisito di qualita' non classificato secondo ISO/IEC 25010:2023

**Entita'**: requirements · **bloccante**

La classificazione e' cio' che rende confrontabili i requisiti di qualita' fra progetti e permette di accorgersi delle caratteristiche scoperte.

**Come correggere** — Valorizzare `quality_characteristic` con una delle 9 caratteristiche ISO/IEC 25010:2023 e `quality_subcharacteristic` con la sottocaratteristica corrispondente.

**Esempio non conforme**

```yaml
type: quality
quality_characteristic: velocita'
```

**Esempio corretto**

```yaml
type: quality
quality_characteristic: performance_efficiency
quality_subcharacteristic: time_behaviour
```

### G3.7 — Caratteristica di qualita' scoperta

**Entita'**: requirements, meta · **maggiore**

Una caratteristica ISO senza alcun requisito e' quasi sempre una dimenticanza, non una scelta: dichiararla non applicabile costringe a prendere posizione.

**Come correggere** — Aggiungere almeno un requisito per la caratteristica, oppure dichiararla non applicabile in `meta.quality_not_applicable` con la motivazione.

**Esempio non conforme**

```yaml
nessun requisito con quality_characteristic: safety
```

**Esempio corretto**

```yaml
meta:
  quality_not_applicable:
    safety: sistema privo di impatti su incolumita' fisica
```

### G3.8 — Conflitto dichiarato senza sede di risoluzione

**Entita'**: requirements, open_points · **bloccante**

Un conflitto noto e non gestito viene risolto implicitamente da chi implementa, cioe' nel posto sbagliato e senza tracciabilita'.

**Come correggere** — Aprire un OpenPoint con `resolution_method` e collegarlo al requisito tramite `open_points`.

**Esempio non conforme**

```yaml
- id: RF-004
  conflicts_with: [RF-007]
```

**Esempio corretto**

```yaml
- id: RF-004
  conflicts_with: [RF-007]
  open_points: [OP-03]
```

### G3.9 — _identificatore riservato, controllo non implementato_

### G3.10 — Requisito derivato senza nota di derivazione

**Entita'**: requirements · **bloccante**

Un requisito DERIVED o ELICITED non ha una fonte testuale: la nota di derivazione e' l'unica prova che non sia gold plating.

**Come correggere** — Valorizzare `derivation_note` spiegando da quale bisogno, vincolo o analisi discende il requisito.

**Esempio non conforme**

```yaml
- id: RQ-002
  derivation: DERIVED
```

**Esempio corretto**

```yaml
- id: RQ-002
  derivation: DERIVED
  derivation_note: discende da NEED-003 e dal vincolo RV-002
```

## G4 — Verificabilita' e criteri di accettazione

### G4.1 — Metodo di verifica assente

**Entita'**: requirements, constraints · **bloccante**

Se non e' dichiarato come si verifica un requisito, la verifica viene improvvisata a fine progetto — o non avviene.

**Come correggere** — Valorizzare `verification_method` con inspection | analysis | demonstration | test.

**Esempio non conforme**

```yaml
- id: RF-005
  verification_method: null
```

**Esempio corretto**

```yaml
- id: RF-005
  verification_method: test
```

### G4.2 — Requisito senza criteri di accettazione

**Entita'**: requirements, constraints · **bloccante**

I criteri di accettazione sono il contratto operativo fra chi commissiona e chi implementa: senza di essi «fatto» non ha definizione.

**Come correggere** — Aggiungere almeno un criterio in `acceptance_criteria` in forma dato/quando/allora.

**Esempio non conforme**

```yaml
- id: RF-006
  acceptance_criteria: []
```

**Esempio corretto**

```yaml
- id: RF-006
  acceptance_criteria:
    - id: AC-RF-006-01
      kind: nominal
      given: utente autenticato
      when: invia il modulo
      then: il sistema registra la richiesta
```

### G4.3 — Soglia numerica senza caso limite

**Entita'**: requirements, constraints · **bloccante**

Una soglia senza caso limite non e' mai verificata dove conta: i difetti si annidano esattamente al valore di confine.

**Come correggere** — Aggiungere un criterio con `kind: boundary` che eserciti il valore esattamente al limite dichiarato.

**Esempio non conforme**

```yaml
statement con '500 ms', criteri solo di tipo nominal
```

**Esempio corretto**

```yaml
- id: AC-RF-007-02
  kind: boundary
  then: a 500 ms esatti la risposta e' ancora accettata
```

### G4.4 — Nessun caso negativo fra i criteri

**Entita'**: requirements, constraints · **maggiore**

Senza casi negativi si specifica solo il percorso felice: il comportamento in caso di errore resta a discrezione di chi implementa.

**Come correggere** — Aggiungere almeno un criterio con `kind: negative` che descriva il comportamento atteso all'ingresso non valido o al fallimento.

**Esempio non conforme**

```yaml
criteri: [nominal, boundary]
```

**Esempio corretto**

```yaml
- id: AC-RF-007-03
  kind: negative
  then: il sistema rifiuta la richiesta e registra l'errore
```

### G4.5 — Requisito di qualita' senza scheda di misura completa

**Entita'**: requirements · **bloccante**

Un requisito di qualita' senza definizione della metrica, statistica, condizioni e minimo accettabile non e' misurabile in modo riproducibile: due misure danno due risultati.

**Come correggere** — Compilare `quality_measure` con metric_name, metric_definition, measurement_method, statistic, target_value, unit e minimum_acceptable.

**Esempio non conforme**

```yaml
type: quality
quality_measure: null
```

**Esempio corretto**

```yaml
quality_measure:
  metric_name: latenza end-to-end
  statistic: p95
  target_value: 500
  unit: ms
  minimum_acceptable: 800
```

### G4.6 — Lessico vietato nel criterio di accettazione

**Entita'**: requirements, constraints · **bloccante**

Un enunciato misurabile con criteri vaghi resta invendicabile: l'ambiguita' si sposta semplicemente dal requisito al collaudo.

**Come correggere** — Riformulare given/when/then con grandezze osservabili; se serve una soglia, usare il blocco `measurable`.

**Esempio non conforme**

```yaml
then: il sistema risponde rapidamente
```

**Esempio corretto**

```yaml
then: il sistema risponde entro 500 ms
measurable: {metric: latenza, operator: '<=', value: 500, unit: ms}
```

### G4.7 — Criterio numerico senza unita' o valore

**Entita'**: requirements, constraints · **bloccante**

Un numero senza unita' e' un'ambiguita' pericolosa (500 cosa? ms, s, richieste?): e' fra le cause piu' banali e piu' frequenti di rilavorazione.

**Come correggere** — Valorizzare `measurable.value` e `measurable.unit` (oltre a `metric` e `operator`).

**Esempio non conforme**

```yaml
measurable: {metric: latenza, operator: '<=', value: 500}
```

**Esempio corretto**

```yaml
measurable: {metric: latenza, operator: '<=', value: 500, unit: ms}
```

## G5 — Descrizione architetturale

### G5.1 — Viewpoint che non indirizza alcun concern reale

**Entita'**: viewpoints, stakeholders · **bloccante**

Un punto di vista architetturale che non risponde a preoccupazioni di stakeholder esistenti produce diagrammi che nessuno usa.

**Come correggere** — Allineare `concerns_addressed` ai concern effettivamente dichiarati dagli stakeholder, oppure eliminare il viewpoint.

**Esempio non conforme**

```yaml
- id: VP-01
  concerns_addressed: [estetica]
```

**Esempio corretto**

```yaml
- id: VP-01
  concerns_addressed: [continuita' operativa]
```

### G5.2 — Concern non indirizzato da alcun viewpoint

**Entita'**: stakeholders, viewpoints · **bloccante**

Un concern dichiarato e mai indirizzato e' una promessa implicita non mantenuta: ISO/IEC/IEEE 42010 richiede che ogni concern trovi una vista.

**Come correggere** — Aggiungere il concern a `concerns_addressed` di un viewpoint esistente o crearne uno dedicato.

**Esempio non conforme**

```yaml
STK-002 dichiara 'auditabilita'', nessun viewpoint la indirizza
```

**Esempio corretto**

```yaml
- id: VP-03
  name: Vista di conformita'
  concerns_addressed: [auditabilita']
```

### G5.3 — Allocazione requisito → architettura mancante o pendente

**Entita'**: arch_elements, requirements, constraints · **bloccante**

Un requisito non allocato non ha un luogo dove essere realizzato; un'allocazione verso un id inesistente e' una tracciabilita' solo apparente.

**Come correggere** — Elencare il requisito in `satisfies` dell'elemento architetturale che lo realizza, verificando che l'id esista.

**Esempio non conforme**

```yaml
- id: ARC-CMP-002
  satisfies: [RF-999]
```

**Esempio corretto**

```yaml
- id: ARC-CMP-002
  satisfies: [RF-002]
```

### G5.4 — Elemento architetturale senza requisiti allocati

**Entita'**: arch_elements · **bloccante**

Un componente che non serve alcun requisito e' architettura non giustificata: costa e non risponde a nessuno.

**Come correggere** — Allocare almeno un requisito in `satisfies`, oppure rimuovere l'elemento (gli elementi `external` sono esclusi da questo controllo).

**Esempio non conforme**

```yaml
- id: ARC-CMP-005
  c4_level: component
  satisfies: []
```

**Esempio corretto**

```yaml
- id: ARC-CMP-005
  satisfies: [RQ-001]
```

### G5.5 — _identificatore riservato, controllo non implementato_

### G5.6 — Dipendenza architetturale pendente

**Entita'**: arch_elements · **bloccante**

Una dipendenza verso un elemento inesistente nasconde un pezzo di architettura non modellato: emergera' in integrazione.

**Come correggere** — Correggere l'id in `depends_on` oppure modellare l'elemento mancante (spesso e' un sistema `external`).

**Esempio non conforme**

```yaml
- id: ARC-CNT-001
  depends_on: [ARC-EXT-009]
```

**Esempio corretto**

```yaml
- id: ARC-CNT-001
  depends_on: [ARC-EXT-001]
```

### G5.7 — ADR senza condizione di innesco

**Entita'**: arch_decisions · **maggiore**

Una decisione senza condizione di riesame non viene mai riaperta: resta valida per inerzia anche quando il contesto e' cambiato.

**Come correggere** — Valorizzare `trigger_conditions` con gli eventi o le soglie che impongono di rivedere la decisione.

**Esempio non conforme**

```yaml
- id: ADR-002
  trigger_conditions: []
```

**Esempio corretto**

```yaml
- id: ADR-002
  trigger_conditions: [3]   # rivedere oltre 3 integrazioni esterne
```

### G5.8 — ADR incompleta

**Entita'**: arch_decisions · **bloccante**

Una decisione senza alternative scartate e senza conseguenze negative accettate non e' una decisione: e' una razionalizzazione a posteriori.

**Come correggere** — Registrare almeno due alternative con `reason_rejected` valorizzato e almeno una voce in `consequences_negative`.

**Esempio non conforme**

```yaml
- id: ADR-003
  alternatives_considered: [{option: A}]
```

**Esempio corretto**

```yaml
- id: ADR-003
  alternatives_considered:
    - {option: A, reason_rejected: costo di licenza}
    - {option: B, reason_rejected: nessun supporto LTS}
  consequences_negative: [lock-in sul fornitore X]
```

### G5.9 — Sistema esterno senza requisito di interfaccia

**Entita'**: arch_elements, requirements · **bloccante**

Ogni confine attraversato e' un contratto: senza requisito di interfaccia il contratto e' verbale e verra' scoperto in integrazione.

**Come correggere** — Aggiungere un requisito con `type: interface` allocato all'elemento esterno, oppure dichiarare la dipendenza in `depends_on` di un elemento interno.

**Esempio non conforme**

```yaml
- id: ARC-EXT-002
  c4_level: external   # nessun requisito lo interfaccia
```

**Esempio corretto**

```yaml
- id: RI-001
  type: interface
  allocated_to: [ARC-EXT-002]
```

## G6 — Decomposizione del lavoro

### G6.1 — Violazione della regola del 100% sulla WBS

**Entita'**: deliverables, work_packages · **bloccante**

Se un deliverable foglia non ha work package, quel lavoro non e' pianificato: la somma dei figli non ricostruisce il padre e la stima e' sistematicamente ottimista.

**Come correggere** — Creare almeno un work package per ogni deliverable foglia e verificare che `parent_id` e `deliverable_id` puntino a id esistenti.

**Esempio non conforme**

```yaml
- id: DLV-004   # foglia, nessun WP la realizza
```

**Esempio corretto**

```yaml
- id: WP-012
  deliverable_id: DLV-004
```

### G6.2 — Nodo WBS denominato con un verbo

**Entita'**: deliverables · **bloccante**

La WBS elenca risultati, non attivita': un nodo denominato con un verbo confonde il «cosa si consegna» con il «cosa si fa» e rende impossibile l'accettazione.

**Come correggere** — Rinominare il deliverable come sostantivo che descrive il prodotto («Servizio di ingestione»), non l'azione («Sviluppare l'ingestione»).

**Esempio non conforme**

```yaml
- id: DLV-002
  name: Sviluppare il modulo di ingestione
```

**Esempio corretto**

```yaml
- id: DLV-002
  name: Modulo di ingestione
```

### G6.3 — Effort del work package fuori dall'intervallo 8-80 ore

**Entita'**: work_packages · **bloccante**

Sotto le 8 ore il tracciamento costa piu' del lavoro; sopra le 80 lo stato di avanzamento non e' osservabile e lo scivolamento si scopre tardi.

**Come correggere** — Scomporre il work package, accorparlo, oppure motivare l'eccezione valorizzando `exception_8_80`.

**Esempio non conforme**

```yaml
- id: WP-004
  effort_optimistic_h: 100
  effort_most_likely_h: 140
  effort_pessimistic_h: 200
```

**Esempio corretto**

```yaml
- id: WP-004
  exception_8_80: migrazione atomica non scomponibile, cfr. ADR-004
```

### G6.4 — Requisito non realizzato da alcun work package

**Entita'**: work_packages, requirements, constraints · **bloccante**

Un requisito che nessun work package realizza non verra' implementato: e' l'argomento di vendita piu' immediato di una RTM, e il difetto piu' silenzioso.

**Come correggere** — Elencare il requisito in `requirements` del work package che lo realizza, verificando che l'id esista.

**Esempio non conforme**

```yaml
- id: WP-003
  requirements: [RF-404]
```

**Esempio corretto**

```yaml
- id: WP-003
  requirements: [RF-004]
```

### G6.5 — Work package realizzativo senza requisiti

**Entita'**: work_packages · **bloccante**

Un work package di tipo build o integration senza requisiti costruisce qualcosa che nessuno ha chiesto: e' gold plating pianificato.

**Come correggere** — Collegare i requisiti realizzati in `requirements`, oppure riclassificare il work package (spike, governance, documentation, verification, compliance).

**Esempio non conforme**

```yaml
- id: WP-007
  kind: build
  requirements: []
```

**Esempio corretto**

```yaml
- id: WP-007
  kind: build
  requirements: [RF-005]
```

### G6.6 — Perimetro escluso non dichiarato

**Entita'**: work_packages · **maggiore**

Cio' che un work package NON fa e' l'informazione che previene il maggior numero di malintesi: senza di essa lo scope si allarga senza che nessuno decida.

**Come correggere** — Valorizzare `scope_excluded` con cio' che resta fuori e dove viene trattato.

**Esempio non conforme**

```yaml
- id: WP-006
  scope_excluded: []
```

**Esempio corretto**

```yaml
- id: WP-006
  scope_excluded: [migrazione dati storici (WP-011)]
```

### G6.7 — Definition of Done troppo povera

**Entita'**: work_packages · **bloccante**

Una DoD con meno di tre voci non copre codice, verifica e documentazione: «fatto» resta un'opinione.

**Come correggere** — Portare `definition_of_done` ad almeno tre voci osservabili e verificabili da terzi.

**Esempio non conforme**

```yaml
definition_of_done: [codice scritto]
```

**Esempio corretto**

```yaml
definition_of_done: [codice in main, TC-004 verde in CI, brief aggiornato]
```

### G6.8 — Classe di deliverable assente e non esclusa

**Entita'**: deliverables, meta · **bloccante**

Le classi mancanti sono quasi sempre lavoro dimenticato (dati, sicurezza, esercizio, transizione): il piano sembra completo e non lo e'.

**Come correggere** — Aggiungere un deliverable della classe mancante, oppure dichiararla non applicabile in `meta.deliverable_classes_not_applicable`.

**Esempio non conforme**

```yaml
nessun deliverable con deliverable_class: transition
```

**Esempio corretto**

```yaml
meta:
  deliverable_classes_not_applicable: [transition]
```

### G6.9 — Nessun walking skeleton sull'intera spina dorsale

**Entita'**: stories · **bloccante**

Senza una fetta verticale che attraversi tutte le posizioni della spina dorsale non si valida l'architettura end-to-end prima di aver speso il grosso del budget.

**Come correggere** — Marcare con `is_walking_skeleton: true` un insieme di story che copra tutte le `backbone_position` presenti.

**Esempio non conforme**

```yaml
backbone: {1,2,3}; walking skeleton solo su {1,2}
```

**Esempio corretto**

```yaml
aggiungere una story con backbone_position 3 e is_walking_skeleton: true
```

### G6.10 — Criteri INVEST non soddisfatti

**Entita'**: stories · **maggiore**

Una story che viola INVEST (indipendente, negoziabile, di valore, stimabile, piccola, testabile) non e' pianificabile in modo affidabile.

**Come correggere** — Correggere la story sui criteri segnalati (tipicamente spezzarla se non e' `small`) e aggiornare `invest_check`.

**Esempio non conforme**

```yaml
invest_check: {small: false}
```

**Esempio corretto**

```yaml
spezzare la story in due e riportare invest_check: {small: true}
```

## G7 — Rischio, dipendenze e sequenziamento

### G7.1 — Grafo delle dipendenze non valido

**Entita'**: work_packages · **bloccante**

Un ciclo o un predecessore inesistente rendono il cammino critico incalcolabile: la pianificazione diventa un elenco di desideri.

**Come correggere** — Rompere il ciclo indicato o correggere l'id del predecessore in `predecessors`.

**Esempio non conforme**

```yaml
WP-002 → WP-003 → WP-002
```

**Esempio corretto**

```yaml
rimuovere la dipendenza ridondante WP-003 → WP-002
```

### G7.2 — Punto aperto senza rischio associato

**Entita'**: open_points, risks · **bloccante**

Un punto aperto e' incertezza dichiarata: se non e' anche un rischio con titolare e risposta, nessuno ne governa le conseguenze.

**Come correggere** — Registrare un rischio con `linked_open_point` valorizzato sull'id del punto aperto.

**Esempio non conforme**

```yaml
- id: OP-02
  status: open   # nessun rischio lo cita
```

**Esempio corretto**

```yaml
- id: RSK-004
  linked_open_point: OP-02
```

### G7.3 — Assunzione non validata senza rischio associato

**Entita'**: assumptions, risks · **bloccante**

Un'assunzione aperta e' una scommessa: se non e' tracciata come rischio, il suo fallimento non ha ne' indicatore ne' piano di risposta.

**Come correggere** — Registrare un rischio con `linked_assumption` valorizzato sull'id dell'assunzione.

**Esempio non conforme**

```yaml
- id: ASM-002
  status: open   # nessun rischio la cita
```

**Esempio corretto**

```yaml
- id: RSK-005
  linked_assumption: ASM-002
```

### G7.4 — Rischio critico senza risposta pianificata

**Entita'**: risks, work_packages · **bloccante**

Accettare tacitamente un rischio a esposizione >= 15 significa scoprirlo quando si materializza, senza budget ne' tempo per reagire.

**Come correggere** — Pianificare uno spike o un work package che lo mitighi (`mitigates_risks`), oppure scegliere una `response_strategy` diversa da `accept`.

**Esempio non conforme**

```yaml
- id: RSK-001
  probability: high
  impact: severe
  response_strategy: accept
```

**Esempio corretto**

```yaml
- id: WP-002
  kind: spike
  mitigates_risks: [RSK-001]
```

### G7.5 — Spike senza timebox, prodotto atteso o decision gate

**Entita'**: work_packages · **bloccante**

Uno spike senza confini si trasforma in sviluppo mascherato: consuma budget senza produrre la decisione per cui esisteva.

**Come correggere** — Valorizzare `timebox_days`, `spike_output`, `spike_decision_gate` e almeno due `spike_outcomes` tipizzati.

**Esempio non conforme**

```yaml
- id: WP-002
  kind: spike
  timebox_days: null
```

**Esempio corretto**

```yaml
- id: WP-002
  kind: spike
  timebox_days: 3
  spike_output: prototipo di misura
  spike_decision_gate: ADR-002
  spike_outcomes: [{outcome: latenza < 500 ms, consequence: si procede}, {outcome: latenza >= 500 ms, consequence: si valuta la cache}]
```

### G7.6 — Work package avviato prima della conclusione dello spike

**Entita'**: work_packages · **bloccante**

Se il lavoro che dipende da uno spike parte prima che lo spike finisca, la decisione che lo spike doveva produrre e' gia' stata presa implicitamente.

**Come correggere** — Aggiungere un legame FS verso lo spike, oppure posticipare l'inizio del work package.

**Esempio non conforme**

```yaml
WP-005 inizia al giorno 2, WP-002 (spike) finisce al giorno 4
```

**Esempio corretto**

```yaml
- id: WP-005
  predecessors: [{id: WP-002, type: FS, dependency_class: mandatory}]
```

### G7.7 — Rischio senza titolare, indicatore o strategia

**Entita'**: risks · **bloccante**

Un rischio senza owner non viene sorvegliato, senza indicatore di innesco non viene visto arrivare, senza strategia non viene gestito.

**Come correggere** — Valorizzare `owner`, `trigger_indicator` e `response_strategy` su ogni rischio.

**Esempio non conforme**

```yaml
- id: RSK-003
  owner: ''
```

**Esempio corretto**

```yaml
- id: RSK-003
  owner: Tech lead
  trigger_indicator: tasso di errore > 2%
  response_strategy: mitigate
```

### G7.8 — Incertezza di stima non gestibile

**Entita'**: work_packages · **bloccante**

Un rapporto pessimistico/ottimistico superiore a 4 dice che non si sta stimando ma indovinando: quel lavoro va prima indagato.

**Come correggere** — Convertire il work package in spike, oppure ridurre l'incertezza scomponendolo e restringendo il perimetro.

**Esempio non conforme**

```yaml
effort_optimistic_h: 4, effort_pessimistic_h: 40
```

**Esempio corretto**

```yaml
- id: WP-002
  kind: spike
  timebox_days: 3
```

### G7.9 — Dipendenza non classificata

**Entita'**: work_packages · **maggiore**

Solo le dipendenze `mandatory` sono immutabili: senza classificazione non si sa quali si possono negoziare per accorciare il cammino critico.

**Come correggere** — Valorizzare `dependency_class` con mandatory | discretionary | external | resource.

**Esempio non conforme**

```yaml
predecessors: [{id: WP-001, type: FS}]
```

**Esempio corretto**

```yaml
predecessors: [{id: WP-001, type: FS, dependency_class: mandatory}]
```

### G7.10 — Cammino critico non determinato

**Entita'**: work_packages · **bloccante**

Senza cammino critico non si sa quale ritardo sposta la data di fine: ogni impegno di consegna e' arbitrario.

**Come correggere** — Verificare che esista almeno un work package e che le stime di effort siano valorizzate.

**Esempio non conforme**

```yaml
work_packages: []
```

**Esempio corretto**

```yaml
definire i work package con effort ottimistico/probabile/pessimistico
```

## G8 — Prioritizzazione e piano di rilascio

### G8.1 — Priorita' MoSCoW assente

**Entita'**: requirements, constraints · **bloccante**

Senza priorita' esplicita, in caso di ritardo si taglia sotto pressione e a caso: la priorita' va decisa a mente fredda.

**Come correggere** — Valorizzare `priority_moscow` con must | should | could | wont.

**Esempio non conforme**

```yaml
- id: RF-008
  priority_moscow: null
```

**Esempio corretto**

```yaml
- id: RF-008
  priority_moscow: should
```

### G8.2 — Quota Must eccessiva sull'effort di release

**Entita'**: releases, work_packages · **maggiore**

Se piu' del 60% dell'effort e' Must non resta margine di manovra: alla prima difficolta' la release slitta invece di ridursi.

**Come correggere** — Ridiscutere la priorita' di parte dei requisiti Must, oppure motivare l'eccezione con `must_quota_exception` sulla release.

**Esempio non conforme**

```yaml
effort Must 78% dell'effort di REL-1
```

**Esempio corretto**

```yaml
- id: REL-1
  must_quota_exception: release regolatoria a perimetro imposto
```

### G8.3 — Vincolo legale o regolatorio non classificato Must

**Entita'**: constraints · **bloccante**

Un obbligo di legge non e' negoziabile: classificarlo diversamente da Must significa mettere in agenda la possibilita' di violarlo.

**Come correggere** — Portare `priority_moscow` a `must` sui vincoli di categoria legal, regulatory o ip.

**Esempio non conforme**

```yaml
- id: RV-002
  category: legal
  priority_moscow: should
```

**Esempio corretto**

```yaml
- id: RV-002
  category: legal
  priority_moscow: must
```

### G8.4 — Obiettivo di release espresso come componente

**Entita'**: releases · **bloccante**

«Modulo X completo» non e' un risultato per l'utente: una release deve dichiarare quale ipotesi valida o quale beneficio consegna.

**Come correggere** — Riformulare `goal` come esito osservabile per lo stakeholder, spostando i componenti in `included_requirements`.

**Esempio non conforme**

```yaml
goal: modulo di ingestione completo
```

**Esempio corretto**

```yaml
goal: l'operatore ottiene il primo report giornaliero senza intervento manuale
```

### G8.5 — MVP assente o non passante

**Entita'**: releases, stories · **bloccante**

Senza una release marcata MVP non c'e' un punto in cui si misura l'ipotesi di valore; un MVP che non attraversa la spina dorsale non e' rilasciabile.

**Come correggere** — Marcare `is_mvp: true` sulla release minima e assegnarle story che coprano tutte le `backbone_position`.

**Esempio non conforme**

```yaml
nessuna release con is_mvp: true
```

**Esempio corretto**

```yaml
- id: REL-1
  is_mvp: true
```

### G8.6 — Release che non dichiara le esclusioni

**Entita'**: releases · **bloccante**

Le aspettative si formano su cio' che non e' scritto: dichiarare cosa la release NON contiene e' l'unico modo per prevenirle.

**Come correggere** — Valorizzare `excludes` con cio' che resta fuori e in quale release e' previsto.

**Esempio non conforme**

```yaml
- id: REL-1
  excludes: []
```

**Esempio corretto**

```yaml
- id: REL-1
  excludes: [esportazione PDF (REL-2)]
```

### G8.7 — Fase senza criteri di ingresso o di uscita

**Entita'**: phases · **bloccante**

Una fase senza criteri e' una scatola temporale vuota: si entra e si esce per calendario, non per stato del lavoro.

**Come correggere** — Valorizzare `entry_criteria` ed `exit_criteria` con condizioni osservabili.

**Esempio non conforme**

```yaml
- id: PH-1
  exit_criteria: []
```

**Esempio corretto**

```yaml
- id: PH-1
  exit_criteria: [gate G4 superato a zero bloccanti]
```

### G8.8 — Predecessore pianificato in una release successiva

**Entita'**: work_packages, releases · **bloccante**

Un lavoro che dipende da qualcosa pianificato dopo non e' eseguibile: la release non chiudera' mai nei tempi previsti.

**Come correggere** — Anticipare il predecessore alla release corrente o posticipare il work package dipendente.

**Esempio non conforme**

```yaml
WP-005 in REL-1 dipende da WP-009 in REL-2
```

**Esempio corretto**

```yaml
spostare WP-009 in REL-1 oppure WP-005 in REL-2
```

## G9 — Verifica di conformita' finale

### G9.1 — _identificatore riservato, controllo non implementato_

### G9.2 — Orfano in direzione discendente (copertura)

**Entita'**: needs, requirements, constraints, test_cases, risks · **bloccante**

Ogni elemento deve essere coperto a valle: un bisogno senza requisito, un requisito senza test, un vincolo senza applicazione o un rischio senza azioni sono lavoro promesso e mai pianificato.

**Come correggere** — Collegare l'elemento al livello successivo: `parent_refs` sul requisito, `verifies` sul caso di test, `affects` sul vincolo, `response_actions` sul rischio.

**Esempio non conforme**

```yaml
- id: RF-009   # nessun TC lo verifica
```

**Esempio corretto**

```yaml
- id: TC-007
  verifies: [RF-009]
```

### G9.3 — Orfano in direzione ascendente (gold plating)

**Entita'**: requirements, constraints, test_cases, stories · **bloccante**

Un requisito senza bisogno ne' segmento sorgente e' lavoro che nessuno ha chiesto: e' la definizione operativa di gold plating.

**Come correggere** — Collegare l'elemento alla sua giustificazione (`parent_refs` o `source_refs`), oppure rimuoverlo dalla baseline.

**Esempio non conforme**

```yaml
- id: RF-010
  parent_refs: []
  source_refs: []
```

**Esempio corretto**

```yaml
- id: RF-010
  parent_refs: [NEED-002]
```

### G9.4 — Indeterminatezza non registrata

**Entita'**: requirements, constraints, needs, open_points · **bloccante**

Un TBD dentro un enunciato baselinato e' un buco contrattuale invisibile: se non e' un OpenPoint, nessuno ha la responsabilita' di chiuderlo.

**Come correggere** — Aprire un OpenPoint e collegarlo tramite `open_points`, oppure eliminare il marcatore risolvendo l'indeterminatezza.

**Esempio non conforme**

```yaml
statement: ... entro TBD secondi
```

**Esempio corretto**

```yaml
statement: ... entro TBD secondi
open_points: [OP-04]
```

### G9.5 — Conflitto irrisolto alla baseline

**Entita'**: requirements, open_points · **bloccante**

Baselinare due requisiti in conflitto senza una sede di risoluzione significa consegnare la contraddizione a chi implementa.

**Come correggere** — Collegare al requisito un OpenPoint in stato open, in_progress o resolved che tratti il conflitto.

**Esempio non conforme**

```yaml
- id: RF-004
  conflicts_with: [RF-007]
  open_points: []
```

**Esempio corretto**

```yaml
- id: RF-004
  open_points: [OP-03]
```

### G9.6 — Identificatore duplicato o non canonico

**Entita'**: — · **bloccante**

Gli id sono le chiavi di tutta la tracciabilita': un duplicato fa collidere due elementi negli indici, una forma non canonica rompe la ricerca delle citazioni nel codice.

**Come correggere** — Rendere univoco l'identificatore e allinearlo alla forma canonica del suo tipo (cfr. `docs/SCHEMA.md`, sezione sugli identificatori).

**Esempio non conforme**

```yaml
- id: RF-1
```

**Esempio corretto**

```yaml
- id: RF-001
```

### G9.7 — Riferimento pendente

**Entita'**: — · **bloccante**

Un riferimento verso un id inesistente e' tracciabilita' apparente: i proiettori la mostrano come se fosse valida e nessuno se ne accorge.

**Come correggere** — Correggere l'id citato o creare l'elemento mancante.

**Esempio non conforme**

```yaml
realized_by: [WP-404]
```

**Esempio corretto**

```yaml
realized_by: [WP-004]
```

### G9.8 — _identificatore riservato, controllo non implementato_

### G9.9 — _identificatore riservato, controllo non implementato_

### G9.10 — Baseline non identificata o fonti non congelate

**Entita'**: meta, source_documents · **bloccante**

Una baseline senza identificatore e senza hash delle fonti non e' riproducibile: non si puo' dimostrare su quale versione dei documenti si e' lavorato.

**Come correggere** — Valorizzare `meta.baseline_id` nella forma BASELINE-<CODICE>-<AAAAMMGG>-<NN> e l'hash SHA-256 di ogni documento sorgente.

**Esempio non conforme**

```yaml
meta: {baseline_id: null}
```

**Esempio corretto**

```yaml
meta: {baseline_id: BASELINE-ACME-20260210-01}
```

