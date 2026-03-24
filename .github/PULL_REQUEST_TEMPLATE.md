## Riepilogo della Contribuzione

<!-- Descrivi brevemente la banca e il formato del file aggiunto o modificato. -->
<!-- Esempio: "Aggiunge il profilo CSV per Banca Sella - conto corrente (formato BSEL-Export-CC)." -->

**Banca:** <!-- es. Banca Sella -->
**ID Profilo:** <!-- es. it_banca_sella_cc_v1 -->
**Formato File:** <!-- csv / xls / xlsx -->
**Tipo di Modifica:** <!-- [ ] Nuovo Profilo &nbsp; [ ] Fix Profilo Esistente &nbsp; [ ] Aggiornamento Versione -->

---

## Dichiarazione Zero-Knowledge (Obbligatoria)

> NoBadget è costruito su un'architettura **Zero-Knowledge**: nessun dato finanziario reale deve
> mai entrare in questo repository. Aprendo questa Pull Request, il sottoscritto dichiara
> sotto sua piena responsabilità che:

- [ ] Il file di test allegato **non contiene** transazioni reali, nomi di persone fisiche reali,
      IBAN reali, o qualsiasi dato finanziario riconducibile a individui identificabili.
- [ ] I dati nel file di test sono stati **completamente sostituiti** con valori fittizi, nel
      rispetto delle istruzioni di anonimizzazione riportate in `docs/CONTRIBUTING.md`.
- [ ] Confermo di aver letto e compreso la sezione **§5 Anonimizzazione dei Dati di Test** della
      guida alla contribuzione.

---

## Checklist Tecnica

> Tutti i punti devono essere spuntati prima che la PR possa essere considerata per il merge.
> PR con checklist incompleta verranno chiuse senza review.

### Validazione Schema

- [ ] Il file JSON è stato validato formalmente contro `schemas/parser_profile_v1.schema.json`
      e la validazione ha prodotto **zero errori**.
      _(Strumento consigliato: [jsonschemavalidator.net](https://www.jsonschemavalidator.net/)
      oppure `ajv-cli` in locale:_
      `ajv validate -s schemas/parser_profile_v1.schema.json -d src/banks/[iso]/[id].json`)

### Posizionamento File

- [ ] Il file JSON del profilo è stato caricato nel percorso corretto:
      `src/banks/[iso]/[id_profilo].json`
      _(Esempio: `src/banks/it/it_banca_sella_cc_v1.json`)_
- [ ] Il nome del file corrisponde **esattamente** al campo `id` dichiarato all'interno del JSON.

### Aggiornamento Manifesto

- [ ] Il file `src/index.json` è stato aggiornato aggiungendo il riferimento al nuovo profilo
      nell'array `parsers`.
- [ ] Il campo `lastUpdated` in `src/index.json` è stato aggiornato alla data odierna in
      formato ISO 8601 (`YYYY-MM-DD`).

### File di Test Anonimizzato

- [ ] Un file di test anonimizzato (CSV/XLS/XLSX) è stato caricato nella directory
      `tests/anonymized_samples/` con il nome `[id_profilo]_sample.[estensione]`.
      _(Esempio: `tests/anonymized_samples/it_banca_sella_cc_v1_sample.csv`)_
- [ ] Il file di test contiene **almeno 5 righe** di transazioni fittizie per una copertura
      ragionevole dei casi d'uso.
- [ ] Se il profilo usa `statusFilter` o `descriptionFilter`, il file include almeno una riga
      che deve essere scartata, per dimostrare il corretto funzionamento del filtro.

### Verifica Parsing nell'App

- [ ] È allegato a questa PR uno **screenshot** che mostra il risultato del parsing nell'app
      NoBadget (versione >= 2.1.0) o nel simulatore, con il **100% delle transazioni del file
      di test importate correttamente** e senza errori.
- [ ] Lo screenshot mostra chiaramente il numero di transazioni importate e l'assenza di
      messaggi di errore o warning.

---

## Note per il Validatore

<!-- Sezione opzionale. Inserisci qui eventuali quirk del formato della banca, casi limite    -->
<!-- riscontrati, o motivazioni per scelte di configurazione non ovvie.                      -->
<!-- Esempio: "Il file export include una riga di saldo finale che viene scartata tramite    -->
<!-- descriptionFilter.rejectContains. Testato su export generato il 2026-03-24."            -->
