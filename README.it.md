---
uid: 723a959b-c6ae-445d-be2d-024840d787f9
modificato: 25/10/2026 15:07
---
# Linee guida per il codice Claude ispirate a Karpathy

> Date un'occhiata al mio nuovo progetto [Multica](https://github.com/multica-ai/multica) — una piattaforma open source per l'esecuzione e la gestione di agenti di programmazione con competenze riutilizzabili.
>
> Seguitemi su X: [https://x.com/jiayuan_jy](https://x.com/jiayuan_jy)

Un unico file `CLAUDE.md` per migliorare il comportamento di Claude Code, derivato dalle [osservazioni di Andrej Karpathy](https://x.com/karpathy/status/2015883857489522876) sulle insidie della programmazione con i modelli di linguaggio di grandi dimensioni (LLM).

Inglese | [Cinese semplificato](./README.zh.md)

## I problemi

Dal post di Andrej:

> "I modelli formulano ipotesi errate per conto tuo e le seguono senza verificarle. Non gestiscono la loro confusione, non cercano chiarimenti, non evidenziano le incongruenze, non presentano i compromessi, non oppongono resistenza quando dovrebbero."

> «Amano davvero complicare eccessivamente il codice e le API, gonfiare le astrazioni, non ripulire il codice morto... implementano una struttura gonfiata di oltre 1000 righe quando ne basterebbero 100.»

> "A volte modificano o rimuovono ancora commenti e codice che non comprendono a sufficienza come effetti collaterali, anche se ortogonali al compito."

## La soluzione

Quattro principi in un unico file che affrontano direttamente questi problemi:

| Principio | Affronta |
|-----------|-----------|
| **Pensare prima di programmare** | Ipotesi errate, confusione nascosta, compromessi mancanti |
| **Semplicità prima di tutto** | Eccessiva complicazione, astrazioni gonfiate |
| **Modifiche chirurgiche** | Modifiche ortogonali, interventi su codice che non si dovrebbe toccare |
| **Esecuzione orientata agli obiettivi** | Sfruttare il "test-first" e criteri di successo verificabili |

## I quattro principi in dettaglio

### 1. Pensare prima di programmare

**Non dare nulla per scontato. Non nascondere la confusione. Mettere in luce i compromessi.**

I modelli di linguaggio di grandi dimensioni (LLM) spesso scelgono un’interpretazione in modo silenzioso e la seguono. Questo principio impone un ragionamento esplicito:

- **Esprimere esplicitamente le ipotesi** — In caso di incertezza, chiedere piuttosto che tirare a indovinare
- **Presentare più interpretazioni** — Non scegliere in silenzio quando c’è ambiguità
- **Opporsi quando giustificato** — Se esiste un approccio più semplice, segnalarlo
- **Fermarsi quando si è confusi** — Indicare ciò che non è chiaro e chiedere chiarimenti

### 2. La semplicità prima di tutto

**Il codice minimo necessario a risolvere il problema. Niente di speculativo.**

Combattere la tendenza all’eccesso di ingegnerizzazione:

- Nessuna funzionalità oltre a quanto richiesto
- Nessuna astrazione per codice monouso
- Nessuna “flessibilità” o “configurabilità” non richiesta
- Nessuna gestione degli errori per scenari impossibili
- Se 200 righe potrebbero essere 50, riscrivilo

**Il test:** un ingegnere senior direbbe che è troppo complicato? Se sì, semplifica.

### 3. Modifiche chirurgiche

**Modifica solo ciò che è necessario. Ripulisci solo il tuo disordine.**

Quando modifichi il codice esistente:

- Non “migliorare” il codice adiacente, i commenti o la formattazione
- Non rifattorizzare ciò che funziona correttamente
- Adattati allo stile esistente, anche se lo faresti diversamente
- Se noti codice morto non correlato, segnalalo — non cancellarlo

Quando le tue modifiche creano elementi orfani:

- Rimuovi le importazioni/variabili/funzioni rese inutilizzate dalle TUE modifiche
- Non rimuovere codice morto preesistente a meno che non ti venga richiesto

**Il test:** ogni riga modificata dovrebbe ricondurre direttamente alla richiesta dell’utente.

### 4. Esecuzione orientata agli obiettivi

**Definisci i criteri di successo. Ripeti il ciclo finché non sono verificati.**

Trasforma le attività imperative in obiettivi verificabili:

| Invece di... | Trasforma in... |
|--------------|-----------------|
| "Aggiungi validazione" | "Scrivi test per input non validi, poi fai in modo che superino i test" |
| "Correggi il bug" | "Scrivi un test che lo riproduca, poi fai in modo che superi il test" |
| "Rifattorizza X" | "Assicurati che i test superino il test prima e dopo" |

Per le attività in più fasi, indica un breve piano:

```
1. [Fase] → verifica: [controllo]
2. [Fase] → verifica: [controllo]
3. [Fase] → verifica: [controllo]
```

Criteri di successo ben definiti consentono all’LLM di eseguire il ciclo in modo autonomo. Criteri deboli (“farlo funzionare”) richiedono continui chiarimenti.

## Installazione

**Opzione A: Plugin Claude Code (consigliata)**

Da Claude Code, aggiungi innanzitutto il marketplace:
```
/plugin marketplace add forrestchang/andrej-karpathy-skills
```

Quindi installa il plugin:
```
/plugin install andrej-karpathy-skills@karpathy-skills
```

In questo modo le linee guida vengono installate come plugin di Claude Code, rendendo la skill disponibile in tutti i tuoi progetti.

**Opzione B: CLAUDE.md (per singolo progetto)**

Nuovo progetto:
```bash
curl -o CLAUDE.md https://raw.githubusercontent.com/forrestchang/andrej-karpathy-skills/main/CLAUDE.md
```

Progetto esistente (aggiunta):
```bash
echo "" >> CLAUDE.md
curl https://raw.githubusercontent.com/forrestchang/andrej-karpathy-skills/main/CLAUDE.md >> CLAUDE.md
```

## Utilizzo con Cursor

Questo repository include una regola di progetto Cursor sottoposta a commit ([`.cursor/rules/karpathy-guidelines.mdc`](.cursor/rules/karpathy-guidelines.mdc)), quindi le stesse linee guida si applicano quando si apre il progetto in Cursor. Consultare **[CURSOR.md](CURSOR.md)** per la configurazione, l’utilizzo della regola in altri progetti e per capire come ciò si colleghi a Claude Code.

## Approfondimento chiave

Da Andrej:

> "I modelli di linguaggio di grandi dimensioni (LLM) sono eccezionalmente bravi a ripetere cicli fino a raggiungere obiettivi specifici... Non dirgli cosa fare, forniscigli i criteri di successo e guardalo all’opera.”

Il principio dell’“esecuzione orientata agli obiettivi” sintetizza questo concetto: trasformare le istruzioni imperative in obiettivi dichiarativi con cicli di verifica.

## Come capire se funziona

Queste linee guida funzionano se noti:

- **Meno modifiche superflue nei diff** — Appaiono solo le modifiche richieste
- **Meno riscritture dovute a eccessiva complessità** — Il codice è semplice fin da subito
- **Le domande chiarificatrici vengono prima dell’implementazione** — Non dopo gli errori
- **Pull request pulite e minimaliste** — Nessun refactoring improvvisato o "miglioramenti"

## Personalizzazione

Queste linee guida sono pensate per essere integrate con le istruzioni specifiche del progetto. Aggiungile al tuo file `CLAUDE.md` esistente o creane uno nuovo.

Per le regole specifiche del progetto, aggiungi sezioni come:

```markdown
## Linee guida specifiche del progetto

- Usa la modalità strict di TypeScript
- Tutti gli endpoint API devono avere dei test
- Segui i modelli esistenti di gestione degli errori in `src/utils/errors.ts`
```

## Nota sui compromessi

Queste linee guida privilegiano la **cautela rispetto alla velocità**. Per le attività banali (semplici correzioni di errori di battitura, modifiche ovvie di una sola riga), usa il buon senso: non tutte le modifiche richiedono il massimo rigore.

L’obiettivo è ridurre gli errori costosi nel lavoro non banale, non rallentare le attività semplici.

## Licenza

MIT
