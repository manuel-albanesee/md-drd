# MD-DRD — Schema del grafo canonico

> Documento **generato** da `md_drd/data/schema/md-drd-graph.schema.json`.
> Non modificare a mano.

## Modello relazionale (chi referenzia chi)

La tracciabilita' del grafo e' verticale: parte dagli interessati e dal bisogno, scende fino al codice tramite requisito → elemento architetturale → work package → caso di test, e risale al testo sorgente tramite i riferimenti di provenienza. Ogni riferimento e' un identificatore del grafo, mai un indice posizionale: il gate `G9.7` rifiuta un grafo che ne contenga uno verso un identificatore inesistente ("riferimento pendente").

| Collezione | Campo | Punta a | Significato |
|---|---|---|---|
| `needs` | `stakeholder_id` | `stakeholders` | ogni bisogno ha un interessato che lo esprime |
| `objectives` | `stakeholder_id` | `stakeholders` | ogni obiettivo ha un interessato che lo esprime |
| `requirements` | `parent_refs` | `needs` | ogni requisito discende da almeno un bisogno (o da un requisito/obiettivo superiore) |
| `arch_elements` | `satisfies` | `requirements` | l'elemento architetturale dichiara quali requisiti soddisfa |
| `deliverables` | `satisfies` | `requirements` | il deliverable dichiara quali requisiti soddisfa |
| `requirements` | `allocated_to` | `arch_elements` | il requisito indica dove e' allocato nell'architettura |
| `requirements` | `realized_by` | `work_packages` | il requisito indica quale work package lo realizza |
| `work_packages` | `requirements` | `requirements` | simmetricamente, il work package elenca i requisiti che realizza |
| `requirements` | `verified_by` | `test_cases` | chiude la catena verso la verifica |
| `work_packages` | `deliverable_id` | `deliverables` | il work package appartiene a un deliverable |
| `phases` | `deliverables` | `deliverables` | la fase raggruppa i deliverable che la compongono |
| `work_packages` | `mitigates_risks` | `risks` | il work package che tratta un rischio lo dichiara esplicitamente |
| `arch_decisions` | `affected_requirements, affected_elements` | `requirements, arch_elements` | l'impatto di una decisione architetturale e' tracciato verso cio' che tocca |
| `change_requests` | `affected_elements` | `arch_elements` | l'impatto di una richiesta di modifica e' tracciato verso cio' che tocca |
| `constraints` | `affects` | `qualunque collezione` | riferimento libero verso l'identificatore che il vincolo limita |
| `open_points` | `blocking_for` | `qualunque collezione` | riferimento libero verso l'identificatore che il punto aperto blocca |
| `stories` | `requirements` | `requirements` | la storia utente si appoggia ai requisiti che copre |
| `releases` | `included_requirements, included_stories` | `requirements, stories` | il rilascio elenca cosa contiene |
| `source_segments` | `document_id` | `source_documents` | ogni segmento appartiene al documento sorgente da cui e' stato estratto |
| `needs, objectives, requirements, constraints, glossary` | `source_refs` | `source_segments` | provenienza testuale: verificata dal gate G0/G2 (copertura) e da `verify-sources` (hash) |

## Collezioni di primo livello

| Collezione | Obbligatoria | Entita' | Gate che la presidiano | Controlli |
|---|---|---|---|---|
| `arch_decisions` | no | [`ArchDecision`](#archdecision) | G5 | 2 |
| `arch_elements` | no | [`ArchElement`](#archelement) | G1, G5 | 5 |
| `assumptions` | no | [`Assumption`](#assumption) | G7 | 1 |
| `change_requests` | no | [`ChangeRequest`](#changerequest) | — | — |
| `constraints` | no | [`Constraint`](#constraint) | G2, G3, G4, G5, G6, G8, G9 | 17 |
| `deliverables` | no | [`Deliverable`](#deliverable) | G6 | 3 |
| `glossary` | no | [`GlossaryTerm`](#glossaryterm) | G0 | 1 |
| `meta` | sì | [`Meta`](#meta) | G3, G6, G9 | 3 |
| `needs` | sì | [`Need`](#need) | G1, G9 | 4 |
| `objectives` | no | [`Objective`](#objective) | G1 | 1 |
| `open_points` | no | [`OpenPoint`](#openpoint) | G2, G3, G7, G9 | 5 |
| `phases` | no | [`Phase`](#phase) | G8 | 1 |
| `releases` | no | [`Release`](#release) | G8 | 5 |
| `requirements` | sì | [`Requirement`](#requirement) | G2, G3, G4, G5, G6, G8, G9 | 26 |
| `risks` | no | [`Risk`](#risk) | G7, G9 | 5 |
| `source_documents` | sì | [`SourceDocument`](#sourcedocument) | G0, G9 | 3 |
| `source_segments` | sì | [`SourceSegment`](#sourcesegment) | G0, G2 | 9 |
| `stakeholders` | sì | [`Stakeholder`](#stakeholder) | G1, G5 | 5 |
| `stories` | no | [`Story`](#story) | G6, G8, G9 | 4 |
| `test_cases` | no | [`TestCase`](#testcase) | G9 | 2 |
| `tradeoffs` | no | [`TradeOff`](#tradeoff) | — | — |
| `viewpoints` | no | [`Viewpoint`](#viewpoint) | G5 | 2 |
| `work_packages` | no | [`WorkPackage`](#workpackage) | G6, G7, G8 | 15 |

## Entita'

### `AcceptanceCriterion`

| Campo | Tipo | Obbligatorio |
|---|---|---|
| `id` | string | sì |
| `kind` | enum(nominal \| boundary \| negative) | sì |
| `given` | string | sì |
| `when` | string | sì |
| `then` | string | sì |
| `measurable` | object \| null | no |

### `ArchDecision`

| Campo | Tipo | Obbligatorio |
|---|---|---|
| `id` | string | sì |
| `title` | string | sì |
| `status` | enum(proposed \| accepted \| deprecated \| superseded) | sì |
| `superseded_by` | string \| null | no |
| `context` | string | sì |
| `decision` | string | sì |
| `trigger_conditions` | `array<integer>` | no |
| `alternatives_considered` | `array<object>` | sì |
| `consequences_positive` | `array<string>` | no |
| `consequences_negative` | `array<string>` | sì |
| `affected_requirements` | `IdList` | no |
| `affected_elements` | `IdList` | no |
| `date` | string | no |
| `decided_by` | string | no |

### `ArchElement`

| Campo | Tipo | Obbligatorio |
|---|---|---|
| `id` | string | sì |
| `name` | string | sì |
| `c4_level` | enum(context \| container \| component \| external) | sì |
| `responsibility` | string | sì |
| `technology` | string \| null | no |
| `interfaces_exposed` | `IdList` | no |
| `interfaces_consumed` | `IdList` | no |
| `satisfies` | `array<string>` | sì |
| `depends_on` | `IdList` | no |
| `view_refs` | `IdList` | no |
| `deployment_unit` | string \| null | no |

### `Assumption`

| Campo | Tipo | Obbligatorio |
|---|---|---|
| `id` | string | sì |
| `statement` | string | sì |
| `rationale` | string | sì |
| `validation_method` | string | sì |
| `validation_deadline` | string | no |
| `owner` | string | sì |
| `impact_if_false` | string | sì |
| `linked_risk` | string \| null | no |
| `source_refs` | `IdList` | no |
| `status` | enum(open \| validated \| invalidated) | sì |

### `ChangeRequest`

| Campo | Tipo | Obbligatorio |
|---|---|---|
| `id` | string | sì |
| `origin` | enum(stakeholder \| defect \| external_constraint \| spike_outcome \| …) | sì |
| `description` | string | sì |
| `affected_elements` | `IdList` | no |
| `impact_analysis` | object | sì |
| `decision` | enum(approved \| rejected \| deferred) | sì |
| `decided_by` | string | no |
| `decision_date` | string | no |
| `new_baseline` | string \| null | no |

### `Constraint`

| Campo | Tipo | Obbligatorio |
|---|---|---|
| `id` | string | sì |
| `statement` | string | sì |
| `category` | enum(legal \| regulatory \| contractual \| platform_tos \| …) | sì |
| `legal_basis` | string \| null | no |
| `imposed_by` | string | sì |
| `consequence_of_violation` | string | sì |
| `affects` | `IdList` | no |
| `source_refs` | `IdList` | no |
| `derivation` | `Derivation` | no |
| `derivation_note` | string \| null | no |
| `priority_moscow` | `Moscow` | no |
| `verification_method` | `VerificationMethod` | sì |
| `open_points` | `IdList` | no |
| `status` | `Status` | sì |

### `Deliverable`

| Campo | Tipo | Obbligatorio |
|---|---|---|
| `id` | string | sì |
| `name` | string | sì |
| `description` | string | no |
| `acceptance_definition` | string | sì |
| `wbs_code` | string | sì |
| `parent_id` | string \| null | no |
| `deliverable_class` | enum(product \| data \| integration \| quality \| …) | sì |
| `satisfies` | `IdList` | no |
| `phase` | string | no |

### `GlossaryTerm`

| Campo | Tipo | Obbligatorio |
|---|---|---|
| `term` | string | sì |
| `definition` | string | sì |
| `aliases` | `IdList` | no |
| `source_refs` | `IdList` | no |

### `Meta`

| Campo | Tipo | Obbligatorio |
|---|---|---|
| `project` | string | sì |
| `project_code` | string | no |
| `spec_version` | string | sì |
| `baseline_id` | string | sì |
| `graph_version` | string | no |
| `generated_at` | string | sì |
| `language` | string | no |
| `derivation_threshold` | number | no |

### `Need`

| Campo | Tipo | Obbligatorio |
|---|---|---|
| `id` | string | sì |
| `statement` | string | sì |
| `stakeholder_id` | string | sì |
| `rationale` | string | sì |
| `priority_raw` | enum(essential \| important \| desirable) | no |
| `source_refs` | `IdList` | no |
| `derivation` | `Derivation` | sì |
| `derivation_note` | string \| null | no |
| `status` | `Status` | sì |

### `Objective`

| Campo | Tipo | Obbligatorio |
|---|---|---|
| `id` | string | sì |
| `statement` | string | sì |
| `stakeholder_id` | string | no |
| `metric` | string | sì |
| `baseline_value` | string \| number \| null | no |
| `target_value` | string \| number | sì |
| `unit` | string \| null | no |
| `horizon` | string \| null | no |
| `measurement_method` | string | sì |
| `source_refs` | `IdList` | no |
| `derivation` | `Derivation` | no |

### `OpenPoint`

| Campo | Tipo | Obbligatorio |
|---|---|---|
| `id` | string | sì |
| `question` | string | sì |
| `blocking_for` | `IdList` | no |
| `resolution_criterion` | string | sì |
| `owner` | string | sì |
| `deadline` | string | no |
| `resolution_method` | enum(spike \| stakeholder_decision \| external_dependency \| measurement \| …) | sì |
| `resolution` | string \| null | no |
| `source_refs` | `IdList` | no |
| `status` | enum(open \| in_progress \| resolved \| escalated) | sì |

### `Phase`

| Campo | Tipo | Obbligatorio |
|---|---|---|
| `id` | string | sì |
| `name` | string | sì |
| `objective` | string | sì |
| `entry_criteria` | `array<string>` | sì |
| `exit_criteria` | `array<string>` | sì |
| `deliverables` | `IdList` | no |
| `gate_review` | string | no |

### `QualityMeasure`

| Campo | Tipo | Obbligatorio |
|---|---|---|
| `metric_name` | string | sì |
| `metric_definition` | string | sì |
| `measurement_method` | string | sì |
| `measurement_conditions` | object | no |
| `statistic` | enum(mean \| median \| p90 \| p95 \| …) | sì |
| `target_value` | number \| string | sì |
| `unit` | string | sì |
| `tolerance` | string \| null | no |
| `minimum_acceptable` | number \| string | sì |
| `measurement_frequency` | string | no |

### `Release`

| Campo | Tipo | Obbligatorio |
|---|---|---|
| `id` | string | sì |
| `name` | string | sì |
| `goal` | string | sì |
| `hypothesis_validated` | string | no |
| `included_stories` | `IdList` | no |
| `included_requirements` | `IdList` | no |
| `is_mvp` | boolean | sì |
| `excludes` | `array<string>` | sì |
| `exit_criteria` | `array<string>` | sì |
| `target_date` | string \| null | no |

### `Requirement`

| Campo | Tipo | Obbligatorio |
|---|---|---|
| `id` | string | sì |
| `version` | string | no |
| `statement` | string | sì |
| `ears_pattern` | enum(ubiquitous \| event_driven \| state_driven \| unwanted \| …) | sì |
| `type` | enum(functional \| quality \| constraint \| interface \| …) | sì |
| `quality_characteristic` | string \| null | no |
| `quality_subcharacteristic` | string \| null | no |
| `quality_measure` | `QualityMeasure` \| null | no |
| `rationale` | string | sì |
| `source_refs` | `IdList` | no |
| `parent_refs` | `IdList` | no |
| `derivation` | `Derivation` | sì |
| `derivation_note` | string \| null | no |
| `priority_moscow` | `Moscow` | sì |
| `wsjf` | object \| null | no |
| `risk_level` | `Scale3` | no |
| `difficulty` | `Scale3` | no |
| `stability` | enum(stable \| likely_to_change \| volatile) | no |
| `verification_method` | `VerificationMethod` | sì |
| `acceptance_criteria` | `array<AcceptanceCriterion>` | sì |
| `allocated_to` | `IdList` | no |
| `realized_by` | `IdList` | no |
| `verified_by` | `IdList` | no |
| `conflicts_with` | `IdList` | no |
| `open_points` | `IdList` | no |
| `status` | `Status` | sì |
| `owner` | string | no |

### `Risk`

| Campo | Tipo | Obbligatorio |
|---|---|---|
| `id` | string | sì |
| `statement` | string | sì |
| `category` | enum(technical \| external \| organizational \| project_management \| …) | sì |
| `probability` | enum(very_low \| low \| medium \| high \| …) | sì |
| `impact` | enum(negligible \| minor \| moderate \| major \| …) | sì |
| `response_strategy` | enum(avoid \| mitigate \| transfer \| accept \| …) | sì |
| `response_actions` | `array<string>` | no |
| `trigger_indicator` | string | sì |
| `owner` | string | sì |
| `source_of_risk` | string | no |
| `linked_open_point` | string \| null | no |
| `linked_assumption` | string \| null | no |
| `linked_requirements` | `IdList` | no |
| `residual_probability` | enum(very_low \| low \| medium \| high \| …) | no |
| `residual_impact` | enum(negligible \| minor \| moderate \| major \| …) | no |
| `status` | enum(identified \| analysed \| responded \| closed \| …) | sì |

### `SourceDocument`

| Campo | Tipo | Obbligatorio |
|---|---|---|
| `id` | string | sì |
| `title` | string | sì |
| `version` | string | no |
| `date` | string | no |
| `hash_sha256` | string | sì |
| `authority` | enum(normative \| indicative \| informative) | sì |
| `language` | string | no |
| `doc_type` | string | no |
| `filename` | string | no |

### `SourceSegment`

| Campo | Tipo | Obbligatorio |
|---|---|---|
| `id` | string | sì |
| `document_id` | string | sì |
| `section_path` | string | sì |
| `page` | integer \| null | no |
| `segment_type` | enum(paragraph \| list_item \| table_cell \| table_row \| …) | sì |
| `text` | string | sì |
| `labels` | `array<enum(prescriptive \| descriptive \| rationale \| constraint \| …)>` | sì |
| `excluded_reason` | string \| null | no |

### `Stakeholder`

| Campo | Tipo | Obbligatorio |
|---|---|---|
| `id` | string | sì |
| `name` | string | sì |
| `category` | enum(user \| operator \| owner \| regulator \| …) | sì |
| `description` | string | no |
| `concerns` | `array<string>` | sì |
| `authority` | enum(decision_maker \| consulted \| informed) | no |
| `source_refs` | `IdList` | no |
| `derivation` | `Derivation` | sì |
| `derivation_note` | string \| null | no |

### `Story`

| Campo | Tipo | Obbligatorio |
|---|---|---|
| `id` | string | sì |
| `epic_id` | string | no |
| `narrative` | object | sì |
| `requirements` | `array<string>` | sì |
| `invest_check` | object | no |
| `story_points` | integer \| null | no |
| `release` | string | no |
| `backbone_position` | integer | sì |
| `is_walking_skeleton` | boolean | no |

### `TestCase`

| Campo | Tipo | Obbligatorio |
|---|---|---|
| `id` | string | sì |
| `verifies` | `array<string>` | sì |
| `level` | enum(unit \| integration \| system \| acceptance \| …) | sì |
| `method` | `VerificationMethod` | sì |
| `preconditions` | `array<string>` | no |
| `steps` | `array<string>` | no |
| `expected_result` | string | sì |
| `pass_criteria` | string | sì |
| `automation` | enum(manual \| automatable \| automated) | no |

### `TradeOff`

| Campo | Tipo | Obbligatorio |
|---|---|---|
| `id` | string | sì |
| `quality_a` | string | sì |
| `quality_b` | string | sì |
| `requirements_involved` | `IdList` | no |
| `tension_description` | string | sì |
| `options` | `array<object>` | no |
| `chosen_option` | string | sì |
| `decision_ref` | string | sì |
| `monitoring_indicator` | string | sì |

### `Viewpoint`

| Campo | Tipo | Obbligatorio |
|---|---|---|
| `id` | string | sì |
| `name` | string | sì |
| `concerns_addressed` | `array<string>` | sì |
| `stakeholders` | `array<string>` | sì |
| `model_kinds` | `array<string>` | sì |
| `notation` | string | no |

### `WorkPackage`

| Campo | Tipo | Obbligatorio |
|---|---|---|
| `id` | string | sì |
| `wbs_code` | string | sì |
| `name` | string | sì |
| `deliverable_id` | string | sì |
| `kind` | enum(build \| spike \| integration \| migration \| …) | sì |
| `description` | string | no |
| `scope_included` | `array<string>` | sì |
| `scope_excluded` | `array<string>` | sì |
| `requirements` | `IdList` | no |
| `definition_of_done` | `array<string>` | sì |
| `effort_optimistic_h` | number | sì |
| `effort_most_likely_h` | number | sì |
| `effort_pessimistic_h` | number | sì |
| `predecessors` | `array<object>` | no |
| `skills_required` | `array<string>` | no |
| `owner_role` | string | no |
| `timebox_days` | number \| null | no |
| `spike_output` | string \| null | no |
| `spike_outcomes` | array \| null | no |
| `spike_decision_gate` | string \| null | no |
| `mitigates_risks` | `IdList` | no |
| `phase` | string | sì |
| `release` | string \| null | no |
| `exception_8_80` | string \| null | no |

## Forme canoniche degli identificatori

| Prefisso | Espressione regolare |
|---|---|
| `SRC-` | `^SRC-D\d+-[^-]+-\d{3}$` |
| `STK-` | `^STK-\d{3}$` |
| `NEED-` | `^NEED-\d{3}$` |
| `OBJ-` | `^OBJ-\d{3}$` |
| `RF-` | `^RF-\d{3}$` |
| `RQ-` | `^RQ-\d{3}$` |
| `RV-` | `^RV-\d{3}$` |
| `RI-` | `^RI-\d{3}$` |
| `RD-` | `^RD-\d{3}$` |
| `RT-` | `^RT-\d{3}$` |
| `ARC-` | `^ARC-(?:CTX|CNT|CMP|EXT)-\d{3}$` |
| `ADR-` | `^ADR-\d{3}$` |
| `DLV-` | `^DLV-\d{3}$` |
| `WP-` | `^WP-\d{3}$` |
| `US-` | `^US-\d{3}$` |
| `TC-` | `^TC-\d{3}$` |
| `RSK-` | `^RSK-\d{3}$` |
| `ASM-` | `^ASM-\d{3}$` |
| `OP-` | `^OP-\d{2}$` |
| `PH-` | `^PH-\d$` |
| `REL-` | `^REL-\d$` |
| `VP-` | `^VP-\d{2}$` |
| `TO-` | `^TO-\d{3}$` |
| `CR-` | `^CR-\d{3}$` |
