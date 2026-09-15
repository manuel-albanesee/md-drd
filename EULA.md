# Contratto di licenza d'uso finale (EULA) — themis-toolkit

> **BOZZA — richiede revisione legale prima dell'uso commerciale.** Questo testo è stato
> redatto per coprire i punti minimi necessari (F0.T1.2, `ROADMAP.md`) ed è coerente con il
> meccanismo tecnico realmente implementato (`themis/license.py`, `scripts/issue_license.py`),
> ma non è stato validato da un legale. Non distribuire il prodotto a clienti paganti sulla base
> di questo testo senza revisione professionale, in particolare per la sezione 10 (foro
> competente/legge applicabile), oggi un segnaposto esplicito.

**Licenziante:** Manuel Albanese, persona fisica (di seguito "il Licenziante").
**Prodotto:** themis-toolkit — comprende gli eseguibili compilati `themis` e `themis-mcp`, la
relativa documentazione e ogni aggiornamento fornito dal Licenziante (di seguito "il Software").

Installando, copiando o eseguendo il Software, il Licenziatario ("Cliente" o "Utente") accetta
integralmente i termini seguenti. Se il Cliente non accetta questi termini, non deve installare
né eseguire il Software.

## 1. Oggetto della licenza

Il Licenziante concede al Cliente una licenza d'uso non esclusiva, non trasferibile (salvo
quanto previsto alla sezione 6) e limitata al numero di postazioni indicato nel file di licenza
emesso al momento dell'acquisto (di seguito "File di Licenza"), per installare ed eseguire il
Software esclusivamente in forma di eseguibile compilato fornito dal Licenziante. Questa licenza
non concede alcun diritto sul codice sorgente del Software, che non viene distribuito e resta di
esclusiva proprietà del Licenziante.

## 2. Meccanismo di attivazione

Il Software richiede, per funzionare, un File di Licenza valido presente sulla macchina del
Cliente (di norma in `~/.themis/license.json`, sovrascrivibile tramite la variabile d'ambiente
`THEMIS_LICENSE_FILE`). Il File di Licenza è firmato digitalmente dal Licenziante e verificato
offline dal Software: **non esiste alcun server di attivazione**, nessuna connessione di rete è
richiesta o effettuata per l'attivazione o per l'uso ordinario del Software.

Il File di Licenza autorizza un elenco fisso di identificativi macchina ("fingerprint"), fino al
numero massimo di postazioni acquistate ("seat"). Il Cliente ottiene l'identificativo della
propria macchina eseguendo il comando `themis license fingerprint`, fornito dallo stesso
eseguibile, e lo comunica al Licenziante prima dell'emissione o dell'estensione del File di
Licenza. L'aggiunta di una postazione richiede la riemissione del File di Licenza da parte del
Licenziante: il Cliente non può modificare autonomamente l'elenco delle macchine autorizzate né
il numero di postazioni.

Il File di Licenza ha una data di scadenza. **Non è previsto alcun periodo di tolleranza
("grace period"): allo scadere della licenza, o su una macchina non presente nell'elenco
autorizzato, il Software rifiuta di eseguire qualunque comando** (fatta eccezione per il comando
diagnostico `themis license fingerprint`, sempre disponibile per consentire l'attivazione o il
rinnovo).

## 3. Restrizioni d'uso

Il Cliente si impegna a **non**:

a. decompilare, disassemblare, eseguire reverse engineering o comunque tentare di derivare il
   codice sorgente del Software, salvo nei limiti in cui tale divieto sia inderogabile per legge
   applicabile;
b. rimuovere, aggirare o tentare di aggirare il meccanismo di verifica della licenza descritto
   alla sezione 2, né manomettere il File di Licenza o alterarne il contenuto;
c. ridistribuire, vendere, noleggiare, concedere in sublicenza o mettere altrimenti a
   disposizione di terzi il Software o il File di Licenza, per intero o in parte, salvo previo
   consenso scritto del Licenziante;
d. utilizzare il Software su un numero di postazioni superiore a quello autorizzato dal File di
   Licenza in vigore;
e. utilizzare il Software per costruire un prodotto o servizio concorrente.

## 4. Proprietà intellettuale

Il Software, in ogni sua componente (codice sorgente, eseguibili, documentazione, marchi), resta
di proprietà esclusiva del Licenziante. Nessuna disposizione del presente EULA trasferisce al
Cliente alcun diritto di proprietà intellettuale sul Software: la licenza concessa è
esclusivamente un diritto d'uso, nei limiti descritti alla sezione 1.

Il Software incorpora componenti software di terze parti distribuite con licenze open source
compatibili, elencate con i rispettivi testi di licenza in `THIRD-PARTY-LICENSES.md`, allegato al
Software. Nulla nel presente EULA limita i diritti concessi da tali licenze di terze parti sui
rispettivi componenti.

## 5. Dati elaborati dal Cliente

Il Software elabora esclusivamente file forniti in locale dal Cliente (grafi di requisiti,
documenti sorgente, configurazioni) e non trasmette tali dati al Licenziante né a terzi: non
esiste alcuna funzionalità di telemetria, raccolta dati o comunicazione di rete nel normale
funzionamento del Software (cfr. sezione 2 — nessun server di attivazione). Il Cliente resta
l'unico responsabile dei dati che sceglie di elaborare con il Software.

## 6. Trasferimento e cessazione

Il Cliente non può cedere o trasferire la presente licenza a terzi senza il previo consenso
scritto del Licenziante.

La licenza cessa automaticamente, senza necessità di comunicazione, in caso di violazione di uno
qualunque dei termini del presente EULA, in particolare delle restrizioni di cui alla sezione 3.
Alla cessazione, il Cliente deve cessare ogni uso del Software e distruggere ogni copia in suo
possesso.

## 7. Garanzie ("as-is")

**Il Software è fornito "così com'è" ("as is"), senza alcuna garanzia di alcun tipo, espressa o
implicita**, incluse a titolo esemplificativo e non esaustivo le garanzie implicite di
commerciabilità, idoneità per uno scopo particolare e non violazione di diritti di terzi. Il
Licenziante non garantisce che il Software sia privo di errori, che funzioni ininterrottamente, o
che i risultati prodotti (inclusi gli esiti di validazione e i proiettori di documenti) siano
esenti da difetti o adeguati a un particolare contesto normativo o contrattuale del Cliente. Il
Cliente resta responsabile della verifica dell'idoneità del Software al proprio caso d'uso prima
di farvi affidamento in contesti critici.

## 8. Limitazione di responsabilità

Nella misura massima consentita dalla legge applicabile, il Licenziante non sarà responsabile per
alcun danno indiretto, incidentale, speciale, consequenziale o punitivo (inclusi, a titolo
esemplificativo, perdita di profitti, di dati o di avviamento) derivante dall'uso o
dall'impossibilità di uso del Software, anche qualora il Licenziante fosse stato informato della
possibilità di tali danni.

In ogni caso, la responsabilità complessiva del Licenziante verso il Cliente per qualunque
pretesa derivante dal presente EULA o dall'uso del Software non potrà eccedere l'importo
effettivamente corrisposto dal Cliente per la licenza oggetto della pretesa.

## 9. Aggiornamenti

Il Licenziante non è tenuto a fornire aggiornamenti, correzioni o nuove versioni del Software.
Quando forniti, gli aggiornamenti sono soggetti ai termini del presente EULA salvo diversa
indicazione scritta del Licenziante al momento della consegna.

## 10. Legge applicabile e foro competente

*(Segnaposto — da definire in sede di revisione legale, non un'omissione accidentale.)*

Il presente EULA è regolato dalla legge **[FORO DA DEFINIRE]**. Per qualunque controversia
relativa alla validità, interpretazione o esecuzione del presente EULA sarà competente in via
esclusiva il foro di **[FORO DA DEFINIRE]**.

## 11. Disposizioni finali

Se una qualunque clausola del presente EULA fosse ritenuta invalida o inapplicabile da un
tribunale competente, le restanti clausole rimarranno pienamente valide ed efficaci. Il presente
EULA costituisce l'intero accordo tra le parti relativamente all'uso del Software e sostituisce
ogni intesa precedente, scritta o orale, sul medesimo oggetto.

---

*Ultimo aggiornamento: 2026-09-08. Versione del prodotto a cui si riferisce: 2.0.0.*
