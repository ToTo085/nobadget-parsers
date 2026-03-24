# Guida alla Contribuzione — NoBadget Parser Engine

Benvenuto! Questa guida descrive il processo completo per contribuire un nuovo profilo di parsing
al repository OTA di NoBadget. Leggi attentamente ogni sezione prima di aprire una Pull Request.

---

## Indice

1. [Principi Fondamentali](#1-principi-fondamentali)
2. [Prerequisiti](#2-prerequisiti)
3. [Struttura del Repository](#3-struttura-del-repository)
4. [Creare un Nuovo Profilo](#4-creare-un-nuovo-profilo)
5. [Anonimizzazione dei Dati di Test](#5-anonimizzazione-dei-dati-di-test)
6. [Aggiornare il Manifesto](#6-aggiornare-il-manifesto)
7. [Aprire la Pull Request](#7-aprire-la-pull-request)
8. [Processo di Review](#8-processo-di-review)
9. [Versionamento e Aggiornamenti](#9-versionamento-e-aggiornamenti)

---

## 1. Principi Fondamentali

NoBadget è un'applicazione di finanza personale ad architettura **Zero-Knowledge**: i dati
finanziari degli utenti non lasciano mai il loro dispositivo. Questo repository gestisce
esclusivamente le *regole* di parsing, non i dati.

Il modello di governance è **Human-in-the-Loop**: non esiste automazione CI/CD per il merge dei
profili. Ogni singolo profilo viene revisionato, testato e approvato manualmente da un membro del
team NoBadget prima della pubblicazione sulla CDN.

**Implicazioni pratiche per i contributori:**

- Non aspettarti merge automatici: la review manuale può richiedere alcuni giorni.
- La qualità e la sicurezza del profilo sono responsabilità del contributor prima della review.
- PR incomplete o con dati reali verranno chiuse immediatamente senza review.

---

## 2. Prerequisiti

Per contribuire avrai bisogno di:

- Un account GitHub e conoscenza base di Git (fork, branch, commit, pull request).
- L'app NoBadget versione **>= 2.1.0** installata su un dispositivo reale o su un simulatore
  (iOS Simulator / Android Emulator) per testare il profilo prima di aprire la PR.
- Un file di export reale della banca target, da usare come riferimento per la struttura
  (che poi **devi anonimizzare** come descritto nella sezione 5).
- Uno strumento di validazione JSON Schema per verificare il profilo prima dell'invio (sezione 4).

---

## 3. Struttura del Repository

```
nobadget-parsers/
├── schemas/
│   └── parser_profile_v1.schema.json   # Schema di validazione (Data Contract)
├── src/
│   ├── index.json                      # Manifesto globale dei profili (OTA)
│   └── banks/
│       └── [iso_stato]/                # Cartella per codice ISO paese (es. "it", "de", "fr")
│           └── [id_profilo].json       # File del profilo di parsing
├── tests/
│   └── anonymized_samples/             # File di test completamente anonimizzati
│       └── [id_profilo]_sample.[ext]
├── docs/
│   └── CONTRIBUTING.md                 # Questa guida
└── .github/
    └── PULL_REQUEST_TEMPLATE.md        # Template obbligatorio per le PR
```

Il codice ISO del paese segue lo standard **ISO 3166-1 alpha-2**
(es. `it` per Italia, `de` per Germania, `fr` per Francia, `es` per Spagna).

---

## 4. Creare un Nuovo Profilo

### 4.1 Convenzionare l'ID

Il campo `id` è la chiave primaria del profilo e deve rispettare il pattern:

```
^[a-z]{2}_[a-z0-9_]+_v[0-9]+$
```

| ID | Significato |
|---|---|
| `it_revolut_v1` | Revolut, mercato italiano, versione 1 |
| `it_banca_sella_cc_v1` | Banca Sella conto corrente, versione 1 |
| `de_n26_v2` | N26, mercato tedesco, versione 2 |

Regole pratiche:
- Usa il codice ISO paese come prefisso (2 lettere minuscole).
- Usa un identificatore della banca senza spazi (usa `_` come separatore interno).
- Aggiungi un suffisso di tipo se la stessa banca ha formati diversi (`_cc` per conto corrente,
  `_dep` per deposito, `_prepaid` per carta prepagata, ecc.).
- Il suffisso di versione `_v1`, `_v2`, ecc. è sempre obbligatorio.

### 4.2 Scegliere l'`amountMode` Corretto

| Modalità | Quando usarla | Campi `columnMapping` obbligatori |
|---|---|---|
| `single` | Il file ha una sola colonna importo con segno (+/-) | `date`, `amount` |
| `split` | Il file ha due colonne separate: entrate e uscite | `date`, `amountIn`, `amountOut` |
| `allExpense` | Il file contiene solo uscite (es. estratto carta di credito) | `date`, `amount` |

### 4.3 Validare il Profilo

Prima di aprire una PR, valida il tuo JSON contro lo schema ufficiale.

**Opzione A — Online (consigliata):**
Vai su [jsonschemavalidator.net](https://www.jsonschemavalidator.net/), incolla il contenuto
di `schemas/parser_profile_v1.schema.json` nel pannello schema e il tuo profilo nel pannello
input. La validazione deve mostrare **zero errori**.

**Opzione B — Locale con `ajv-cli`:**

```bash
npm install -g ajv-cli
ajv validate -s schemas/parser_profile_v1.schema.json -d src/banks/it/it_mia_banca_v1.json
```

---

## 5. Anonimizzazione dei Dati di Test

> **Questa è la sezione più critica dell'intera guida. Leggerla con la massima attenzione.**

I profili di parsing vengono testati su file reali per garantirne il corretto funzionamento.
Tuttavia, caricare un estratto conto reale in un repository pubblico è una grave violazione della
privacy e potenzialmente della normativa GDPR. Per questo motivo, **ogni file di test deve essere
completamente anonimizzato** prima del caricamento.

### 5.1 Cosa Modificare (Obbligatorio)

Sostituisci sistematicamente i seguenti contenuti con valori fittizi inventati:

| Tipo di Dato | Dato Reale (esempio) | Dato Fittizio Accettabile |
|---|---|---|
| Nomi di persone o aziende | `Mario Rossi`, `Amazon EU S.a.r.l.` | `Test Persona`, `Azienda Fittizia SRL` |
| Descrizioni delle transazioni | `Pagamento affitto Gennaio` | `Descrizione transazione test` |
| Date | `15/02/2026` | `01/01/2024`, `15/03/2024` |
| Importi | `1.247,83`, `-89,50` | `100,00`, `-50,00` |
| IBAN o codici di riferimento | `IT60X0542811101000000123456` | `IT00X0000000000000000000000` |
| Note o memo liberi | Qualsiasi testo personale | `Nota test` |

### 5.2 Cosa NON Modificare (Critico per il Funzionamento)

La struttura tecnica del file è il dato che il profilo deve saper leggere. Alterarla renderebbe
il file di test inutile ai fini della verifica. **Non modificare mai:**

- **Le intestazioni delle colonne (header row):** I nomi delle colonne devono rimanere identici
  all'originale. Se la banca usa `"Data operazione"`, quella stringa deve restare invariata.

- **La struttura del formato data:** Se la banca usa `dd/MM/yyyy` con slash come separatore,
  le date fittizie devono usare la stessa struttura (`01/06/2024`, non `2024-06-01`). Il motore
  usa il `dateFormat` del profilo per interpretare questa struttura: cambiarla causa errori di
  parsing immediati o silenziosi.

- **Il separatore decimale negli importi:** Se la banca usa la virgola come separatore decimale
  (`1.234,56`), le cifre fittizie devono mantenere la stessa convenzione (`100,00`, non `100.00`).
  Invertire il separatore causa errori di parsing silenziosi e difficili da diagnosticare.

- **Il delimitatore di colonna del CSV:** Se il file originale usa `;` come separatore di campo,
  il file anonimizzato deve farlo allo stesso modo.

- **Il numero di righe di intestazione:** Se il file originale ha 2 righe di testo prima dei dati
  (`skipRows: 2`), il file anonimizzato deve mantenere la stessa struttura iniziale, anche se il
  testo può essere sostituito con segnaposto generici.

- **Le righe speciali della banca:** Righe di saldo, totali o footer che il profilo potrebbe
  scartare tramite `descriptionFilter` o `statusFilter` devono essere presenti nel campione
  (con valori fittizi) per dimostrare che il filtro funziona correttamente.

### 5.3 Numero Minimo di Righe

Il file di test deve contenere **almeno 5 transazioni** fittizie. Se il profilo usa
`statusFilter` o `descriptionFilter`, includi almeno una riga che deve essere scartata,
per dimostrare il corretto funzionamento del filtro.

### 5.4 Esempio Pratico

**File originale — NON caricare mai questo:**

```
Data,Descrizione,Importo,Stato
15/02/2026,Stipendio Febbraio ACME SRL,2500.00,COMPLETATO
18/02/2026,Amazon Prime abbonamento,-14.99,COMPLETATO
20/02/2026,Bonifico Mario Rossi affitto,-850.00,COMPLETATO
```

**File anonimizzato corretto — caricare questo:**

```
Data,Descrizione,Importo,Stato
01/01/2024,Entrata fittizia Test Azienda SRL,1000.00,COMPLETATO
05/01/2024,Uscita servizio test,-20.00,COMPLETATO
10/01/2024,Bonifico Test Persona,-500.00,COMPLETATO
15/01/2024,Transazione da scartare,-99.00,ANNULLATO
20/01/2024,Altra entrata test,200.00,COMPLETATO
```

Nota: il separatore decimale (`.`), il formato data (`dd/MM/yyyy`), le intestazioni di colonna
e il delimitatore CSV (`,`) sono **invariati**. Solo i valori di contenuto sono stati sostituiti.

---

## 6. Aggiornare il Manifesto

Dopo aver creato il file del profilo, aggiungi il riferimento in `src/index.json`.

Aggiungi un oggetto all'array `parsers`:

```json
{
  "id": "it_mia_banca_v1",
  "name": "Mia Banca",
  "format": "csv",
  "path": "banks/it/it_mia_banca_v1.json"
}
```

Aggiorna il campo `lastUpdated` con la data odierna in formato `YYYY-MM-DD`.

---

## 7. Aprire la Pull Request

1. Fai il fork di questo repository sul tuo account GitHub.
2. Crea un branch descrittivo: `git checkout -b add/it_mia_banca_v1`
3. Aggiungi i seguenti file al commit:
   - `src/banks/[iso]/[id_profilo].json`
   - `tests/anonymized_samples/[id_profilo]_sample.[ext]`
   - `src/index.json` (aggiornato)
4. Apri la Pull Request verso il branch `main` di questo repository.
5. Compila **integralmente** il template `.github/PULL_REQUEST_TEMPLATE.md`.
   PR con template incompleto o checklist non spuntata verranno chiuse senza review.
6. Allega lo screenshot del parsing riuscito nell'app o nel simulatore.

---

## 8. Processo di Review

Il processo è interamente manuale, nel rispetto del modello Human-in-the-Loop di NoBadget.

**Cosa fa il Validatore Senior durante la review:**

1. Scarica il profilo JSON e lo valida contro lo schema ufficiale.
2. Verifica che il file di test non contenga dati personali reali.
3. Importa il file di test nell'app usando il profilo, confrontando l'output con lo screenshot
   allegato.
4. Controlla la logica dei filtri (`statusFilter`, `descriptionFilter`) e la correttezza del
   `columnMapping`.
5. Verifica `dateFormat` e `decimalSeparator` rispetto alla struttura del file di test.

**Criteri per il Merge:**

- Zero errori di validazione schema.
- 100% delle transazioni attese importate correttamente (nessuna riga persa silenziosamente).
- Nessun dato personale reale presente nel file di test.
- `src/index.json` aggiornato correttamente.
- Screenshot allegato e coerente con i dati del file di test.

---

## 9. Versionamento e Aggiornamenti

Se la banca modifica il formato del suo file di export, crea un **nuovo profilo** con versione
incrementata (es. `it_mia_banca_v2`) invece di modificare quello esistente. Questo garantisce
la retrocompatibilità per gli utenti che hanno già file nel vecchio formato.

Il campo `version` nel JSON del profilo viene usato dal client Flutter per rilevare
automaticamente i profili aggiornati e forzare la risincronizzazione OTA.

---

*© 2026 NoBadget Team (ToTo085). Validated Code. Secure By Design. Private By Default.*
