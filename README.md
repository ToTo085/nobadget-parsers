# 🏦 NoBadget - Dynamic Parser Engine (OTA)

[](https://www.google.com/search?q=https://opensource.org/licenses/MIT)
[](https://www.google.com/search?q=%23)
[](https://www.google.com/search?q=%23)
[](https://www.google.com/search?q=%23)

Benvenuto nel repository ufficiale dei **Profili di Parsing OTA (Over-The-Air)** di NoBadget.

NoBadget è un'applicazione di finanza personale costruita su un'architettura **Zero-Knowledge**: i tuoi dati finanziari non lasciano mai il tuo dispositivo. Questo repository gestisce il layer di integrazione bancaria, fornendo le regole di decodifica (tramite file JSON distribuiti in modo sicuro) per leggere e normalizzare i file CSV, XLS e XLSX esportati dagli istituti di credito.

## 🚀 Architettura Over-The-Air (OTA)

A partire dalla versione **2.1.0** dell'app (Flutter Client), NoBadget integra un motore di parsing unificato (`DynamicCsvParser`).
L'app esegue il download crittograficamente sicuro dei file di configurazione per aggiornare le proprie capacità di lettura.

I vantaggi di questa architettura enterprise:

  * **Zero App Updates:** Nuove banche e correzioni di formato sono disponibili istantaneamente, senza dipendere dai tempi di rilascio degli Store (App Store / Google Play).
  * **Massima Sicurezza:** I parsing avvengono **solo in locale** (nella RAM dello smartphone). Nessun server esterno processa le tue transazioni.
  * **Release Controllate:** Ogni payload JSON è validato manualmente dal nostro team di Architetti e firmato digitalmente per prevenire attacchi Man-In-The-Middle.

## 🤝 Linee Guida per la Contribuzione (Strict Governance)

Vuoi aggiungere il supporto alla tua banca? In NoBadget adottiamo un approccio *Human-in-the-Loop*: ogni singola regola di parsing viene revisionata e testata manualmente per garantire l'integrità del database degli utenti.

Per contribuire, segui rigorosamente questi passaggi:

1.  **Fork del Progetto:** Crea un Fork di questo repository.
2.  **Creazione Profilo:** Crea un nuovo file JSON nella directory `src/banks/` (es. `it_banca_sella_cc_v1.json`) seguendo il nostro Data Contract.
3.  **Aggiornamento Indice:** Aggiungi il riferimento al tuo file nel manifesto `src/index.json`.
4.  **Preparazione Test:** Prepara un file CSV/XLS di esportazione della banca **completamente anonimizzato** (modifica nomi, importi e date, ma mantieni intatta la struttura e le intestazioni).
5.  **Apertura PR:** Apri una Pull Request compilando il template fornito. **Devi** allegare il file anonimizzato e uno screenshot dell'esito del parsing per permetterci la riproduzione del test.
6.  🕵️‍♂️ **Review Manuale:** Un *Validatore Senior* del team NoBadget scaricherà il tuo codice, verificherà la sicurezza delle Regex e testerà il mapping. Se i test passano, la PR verrà unita (Merged) e pubblicata globalmente tramite la nostra CDN.

-----

## 📄 Specifiche Tecniche del Data Contract (`ParserProfile`)

La struttura del file di configurazione è il contratto dati stipulato tra il formato dell'istituto bancario e il motore di analisi interno di NoBadget.

| Campo | Tipo | Requisito | Descrizione Architetturale |
| :--- | :--- | :--- | :--- |
| `id` | `string` | **Richiesto** | Chiave Primaria (PK). Deve seguire il pattern `[iso_stato]_[banca]_[tipo]_[versione]` (es. `it_revolut_v1`). |
| `bankName` | `string` | **Richiesto** | Label esposto nella UI dell'utente (es. "Revolut"). |
| `fileExtension` | `string` | **Richiesto** | Enum supportato: `csv`, `xls`, `xlsx`. Definisce il reader nativo da attivare. |
| `skipRows` | `int` | **Richiesto** | Offset per bypassare intestazioni testuali o disclaimer legali prima dei dati. |
| `delimiter` | `string` | Opzionale | Separatore di colonna (es. `,`, `;`, `\t`). Default: `,` |
| `headerMode` | `bool` | **Richiesto** | Se `true`, il mapping si basa sulle label esatte delle colonne. Se `false`, utilizza gli indici (0, 1, 2...). |
| `columnMapping` | `object` | **Richiesto** | Dizionario di associazione tra la logica di NoBadget (`date`, `amount`, `description`, `status`) e le colonne del file. |
| `dateFormat` | `string` | **Richiesto** | Pattern cronologico. Supporta sintassi standard (es. `dd/MM/yyyy`, `yyyy-MM-dd`) o la key `auto` per parser ISO 8601. |
| `dateLocale` | `string` | Opzionale | Formato lingua per date testuali (es. `it` per parsing di "27 febbraio 2026"). |
| `decimalSeparator` | `string` | **Richiesto** | Carattere delimitatore dei decimali. Enum: `.` (US/UK) oppure `,` (EU). |
| `invertAmountSign` | `bool` | **Richiesto** | Switch per invertire la cassa (necessario per le banche che esportano le uscite in positivo). |
| `amountMode` | `string` | **Richiesto** | Enum: `single` (unica colonna con +/-), `split` (colonne separate `amountIn`/`amountOut`), `allExpense` (tratta tutto come uscita). |
| `statusFilter` | `object` | Opzionale | Middleware di filtraggio transazioni: `rejectValues` (ignora righe specifiche) o `allowContains` (lista bianca). |
| `descriptionFilter`| `object` | Opzionale | Sanitizzazione riga tramite Regex per ignorare movimenti spazzatura della banca. |
| `descriptionTemplate`| `string` | Opzionale | String interpolation engine. Es. `{nome} - {causale}` unisce più colonne in un'unica stringa descrittiva. |
| `version` | `int` | **Richiesto** | Versioning incrementale per forzare la risincronizzazione OTA sui dispositivi. |

-----

## 🛠️ Esempi di Implementazione

### Esempio 1: CSV Standard (Revolut - Singola Colonna)

Struttura ideale per banche moderne con formato CSV pulito.

```json
{
  "id": "it_revolut_v1",
  "bankName": "Revolut",
  "fileExtension": "csv",
  "skipRows": 0,
  "delimiter": ",",
  "headerMode": true,
  "columnMapping": {
    "date": "Data di inizio",
    "description": "Descrizione",
    "amount": "Importo",
    "status": "State"
  },
  "dateFormat": "M/d/yy",
  "decimalSeparator": ",",
  "invertAmountSign": false,
  "amountMode": "single",
  "statusFilter": {
    "column": "status",
    "rejectValues": ["OPERAZIONE ANNULLATA"],
    "mode": "exact"
  },
  "version": 1
}
```

### Esempio 2: Excel Complesso (Satispay - Template & AllowList)

Utilizzo avanzato di `descriptionTemplate` per aggregare dati su file Excel.

```json
{
  "id": "it_satispay_v1",
  "bankName": "Satispay",
  "fileExtension": "xlsx",
  "skipRows": 0,
  "delimiter": ",",
  "headerMode": true,
  "columnMapping": {
    "date": "Data",
    "amount": "Importo",
    "name": "Nome",
    "description": "Descrizione",
    "status": "Stato"
  },
  "dateFormat": "auto",
  "decimalSeparator": ",",
  "invertAmountSign": false,
  "amountMode": "single",
  "statusFilter": {
    "column": "status",
    "allowContains": ["✓", "approvato", "completato"]
  },
  "descriptionTemplate": "{name} - {description}",
  "version": 1
}
```

-----

*© 2026 NoBadget Team. Validated Code. Secure By Design. Private By Default.*
