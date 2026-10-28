# Esempi

Esempi di codice reali che illustrano i quattro principi. Ogni esempio mostra quali sono gli errori più comuni commessi dai modelli di linguaggio di grandi dimensioni (LLM) e come correggerli.

---

## 1. Pensare prima di scrivere il codice

### Esempio 1: Presupposti nascosti

**Richiesta dell'utente:** "Aggiungere una funzionalità per esportare i dati degli utenti"

**❌ Cosa fanno gli LLM (supposizioni errate)**

```python
def export_users(format='json'):
    """Esporta tutti gli utenti in un file."""
    users = User.query.all()
 
 if format == 'json':
 with open('users.json', 'w') as f:
 json.dump([u.to_dict() for u in users], f)
    elif format == 'csv':
        with open('users.csv', 'w') as f:
 writer = csv.DictWriter(f, fieldnames=['id', 'email', 'name'])
 writer.writeheader()
 writer.writerows([u.to_dict() for u in users])
    
    return f"{len(users)} utenti esportati"
```

**Problemi:**
- Si è dato per scontato che dovessero essere esportati TUTTI gli utenti (e la paginazione? E la privacy?)
- Si è dato per scontato il percorso del file senza verificarlo
- Si è dato per scontato quali campi includere
- Ha dato per scontati i nomi dei campi CSV senza verificare la struttura effettiva dei dati

**✅ Cosa dovrebbe succedere (individuare le supposizioni)**

```
Prima di implementare, devo chiarire:

1. **Ambito**: esportare tutti gli utenti o un sottoinsieme filtrato? (implicazioni relative alla privacy)
2. **Formato**: Hai menzionato "esportazione" - intendevi:
   - Scaricare il file nel browser?
   - Processo in background con notifica via e-mail?
   - Endpoint API che restituisce i dati?
3. **Campi**: quali campi utente? (alcuni potrebbero essere sensibili)
4. **Volume**: quanti utenti in genere? (influisce sull’approccio)

Approccio più semplice: aggiungere un endpoint API che restituisca JSON impaginato.
Servirebbero maggiori informazioni per le esportazioni basate su file.

Qual è la tua preferenza?
```

### Esempio 2: Interpretazioni multiple

**Richiesta dell’utente:** “Rendi la ricerca più veloce”

**❌ Cosa fanno i modelli di linguaggio di grandi dimensioni (LLM) (scelta silenziosa)**

```python
# Aggiunge cache, indici del database ed elaborazione asincrona
# senza chiedere quale "più veloce" sia rilevante
@lru_cache(maxsize=1000)
async def search(query: str) -> List[Result]:
    # ... 200 righe di codice di ottimizzazione
```

**✅ Cosa dovrebbe succedere**

```
"Rendere la ricerca più veloce" potrebbe significare:

1. **Tempo di risposta più veloce** (< 100 ms invece di 500 ms)
   - Aggiungere indici al database
   - Memorizzare nella cache le query frequenti
   - Tempo stimato: 2 ore

2. **Gestire più ricerche simultanee** (throughput)
   - Aggiungere async/await
   - Pooling delle connessioni
   - Tempo stimato: 4 ore

3. **Maggiore percezione di velocità** (UX)
   - Mostrare immediatamente i risultati parziali
   - Caricamento progressivo
   - Tempo stimato: 3 ore

La ricerca attuale impiega circa 500 ms per le query tipiche. 
Quale aspetto è più importante per il tuo caso d’uso?
```

---

## 2. La semplicità prima di tutto

### Esempio 1: Eccessiva astrazione

**Richiesta dell’utente:** “Aggiungere una funzione per calcolare lo sconto”

**❌ Cosa fanno i modelli di linguaggio di grandi dimensioni (LLM) (soluzione troppo complessa)**

```python
from abc import ABC, abstractmethod
from enum import Enum
from typing import Protocol, Union
from dataclasses import dataclass

class DiscountStrategy(ABC):
    @abstractmethod
    def calculate(self, amount: float) -> float:
 pass

class PercentageDiscount(DiscountStrategy):
    def __init__(self, percentage: float):
    self.percentage = percentage
 
    def calculate(self, amount: float) -> float:
        return amount * (self.percentage / 100)

class ScontoFisso(StrategiaSconto):
    def __init__(self, importo_fisso: float):
 self.importo_fisso = importo_fisso
    
    def calculate(self, amount: float) -> float:
 return min(self.fixed_amount, amount)

@dataclass
class DiscountConfig:
    strategy: DiscountStrategy
    min_purchase: float = 0.0
    max_discount: float = float('inf')

class DiscountCalculator:
    def __init__(self, config: DiscountConfig):
 self.config = config
 
 def apply_discount(self, amount: float) -> float:
 if amount < self.config.min_purchase:
            return 0
 discount = self.config.strategy.calculate(amount)
 return min(discount, self.config.max_discount)

# L'utilizzo richiede oltre 30 righe di configurazione per un semplice calcolo
```

**✅ Cosa dovrebbe succedere (semplice)**

```python
def calculate_discount(amount: float, percent: float) -> float:
    """Calcola l'importo dello sconto. percent deve essere compreso tra 0 e 100."""
    return amount * (percent / 100)

# Utilizzo
sconto = calculate_discount(100.0, 10.0)  # 10 $ di sconto
```

**Quando aggiungere complessità:** Solo quando si hanno effettivamente bisogno di più tipi di sconto. Se tale esigenza dovesse presentarsi in seguito, rifattorizzare in quel momento.

### Esempio 2: Funzionalità speculative

**Richiesta dell’utente:** "Salvare le preferenze dell’utente nel database"

**❌ Cosa fanno i modelli di linguaggio grandi (funzionalità non necessarie)**

```python
class PreferenceManager:
    def __init__(self, db, cache=None, validator=None):
 self.db = db
        self.cache = cache or InMemoryCache()
 self.validator = validator or DefaultValidator()
 
 def save(self, user_id: int, prefs: dict, 
 merge: bool = True, 
 validate: bool = True,
             notify: bool = False) -> bool:
 """Salva le preferenze con unione, convalida e notifiche opzionali."""
 
 if validate:
 errors = self.validator.validate(prefs)
 if errors:
 raise ValidationError(errors)
        
        existing = self.get(user_id) if merge else {}
 
 if merge:
 existing.update(prefs)
 prefs = existing
        
        self.db.execute(
 "UPDATE users SET preferences = ? WHERE id = ?",
 (json.dumps(prefs), user_id)
 )
 
 if self.cache:
 self.cache.set(f"prefs:{user_id}", prefs)
        
        if notify:
 self.notify_preference_change(user_id, prefs)
 
 return True
 
 def notify_preference_change(self, user_id: int, prefs: dict):
 # Altre 30 righe per una funzionalità che nessuno ha richiesto
 pass
```

**✅ Cosa dovrebbe succedere (esattamente ciò che è stato richiesto)**

```python
def save_preferences(db, user_id: int, preferences: dict):
    """Salva le preferenze dell’utente nel database."""
    db.execute(
 "UPDATE users SET preferences = ? WHERE id = ?",
 (json.dumps(preferences), user_id)
    )
```

**Da aggiungere in seguito se necessario:** Caching (quando le prestazioni contano), validazione (quando compaiono dati errati), unione (quando emerge una nuova esigenza).

---

## 3. Modifiche chirurgiche

### Esempio 1: Refactoring al volo

**Richiesta dell’utente:** "Correggi il bug per cui le email vuote causano il crash del validatore"

**❌ Cosa fanno (eccessivamente) i modelli di linguaggio di grandi dimensioni (LLM)**

```diff
  def validate_user(user_data):
-     # Controlla il formato dell’email
-     if not user_data.get('email'):
+     """Convalida i dati dell’utente."""
+     email = user_data.get('email', '').strip()
+ 
+     # Convalida l’email
+     if not email:
 raise ValueError("Indirizzo email obbligatorio")
- 
-     # Convalida di base dell'indirizzo email
-     if '@' not in user_data['email']:
+     if '@' not in email or '.' not in email.split('@')[1]:
 raise ValueError("Indirizzo email non valido")
      
-     # Controllo del nome utente
-     if not user_data.get('username'):
+     # Convalida del nome utente
+     username = user_data.get('username', '').strip()
+     if not username:
          raise ValueError("Nome utente obbligatorio")
+     se len(username) < 3:
+ raise ValueError("Nome utente troppo breve")
+     se non username.isalnum():
+ raise ValueError("Il nome utente deve essere alfanumerico")
 
 return True
```

**Problemi:**
- Convalida dell’indirizzo e-mail “migliorata” oltre alla correzione del bug
- Aggiunta la convalida del nome utente che nessuno aveva richiesto
- Modificati i commenti
- Aggiunta la stringa di documentazione

**✅ Cosa dovrebbe succedere (soluzione chirurgica)**

```diff
  def validate_user(user_data):
      # Controlla il formato dell'email
-     if not user_data.get('email'):
+     email = user_data.get('email', '')
+     if not email or not email.strip():
 raise ValueError("Email obbligatoria")
 
 # Convalida di base dell'email
-     if '@' not in user_data['email']:
+     if '@' not in email:
 raise ValueError("Indirizzo email non valido")
 
 # Controllo del nome utente
 if not user_data.get('username'):
 raise ValueError("Nome utente obbligatorio")
      
      return True
```

**Modificato solo:** Le righe specifiche che risolvono la gestione dell'indirizzo email vuoto.

### Esempio 2: Variazione di stile

**Richiesta dell'utente:** "Aggiungi la registrazione a livello di log alla funzione di upload"

**❌ Cosa fanno i modelli di linguaggio (riformattano tutto)**

```diff
- def upload_file(file_path, destination):
+ def upload_file(file_path: str, destination: str) -> bool:
+     """Carica il file nella destinazione con registrazione."""
+     logger.info(f"Caricamento di {file_path} su {destination}")
+ 
 try:
- with open(file_path, 'rb') as f:
- data = f.read()
-         
- response = requests.post(destination, files={'file': data})
- 
- if response.status_code == 200:
- return True
- else:
- return False
+ with open(file_path, "rb") as f:
+ data = f.read()
+ response = requests.post(destination, files={"file": data})
+ success = response.status_code == 200
+ 
+ if success:
+ logger.info(f"Caricamento riuscito: {file_path}")
+ else:
+ logger.error(f"Caricamento fallito: {file_path}, stato={response.status_code}")
+ 
+ return success
 except Exception as e:
- print(f"Errore: {e}")
+ logger.exception(f"Errore di caricamento per {file_path}: {e}")
 return False
```

**Problemi:**
- Modificato lo stile delle virgolette (da '' a "")
- Aggiunti suggerimenti sul tipo che nessuno aveva richiesto
- Aggiunta stringa di documentazione
- Riformattati gli spazi bianchi
- Modificata la logica di ritorno booleano

**✅ Cosa dovrebbe succedere (rispettare lo stile esistente)**

```diff
+ import logging
+ 
+ logger = logging.getLogger(__name__)
+ 
  def upload_file(file_path, destination):
+     logger.info(f'Avvio del caricamento: {file_path}')
 try:
 with open(file_path, 'rb') as f:
 data = f.read()
          
          response = requests.post(destination, files={'file': data})
 
 if response.status_code == 200:
+ logger.info(f'Caricamento riuscito: {file_path}')
 return True
 else:
+ logger.error(f'Caricamento fallito: {file_path}, stato={response.status_code}')
 return False
 except Exception as e:
- print(f"Errore: {e}")
+ logger.exception(f'Errore di caricamento: {file_path}')
 return False
```

**Corrispondenze:** virgolette singole, nessun suggerimento di tipo, pattern booleano esistente, stile di spaziatura.

---

## 4. Esecuzione orientata agli obiettivi

### Esempio 1: Vago vs. Verificabile

**Richiesta dell'utente:** "Correggi il sistema di autenticazione"

**❌ Cosa fanno i modelli di linguaggio grandi (approccio vago)**

```
Correggerò il sistema di autenticazione:
1. Esaminando il codice
2. Identificando i problemi
3. Apportando miglioramenti
4. Testando le modifiche

[Proceda ad apportare modifiche senza criteri di successo chiari]
```

**✅ Cosa dovrebbe accadere (obiettivi verificabili)**

```
Per “risolvere il problema dell’autenticazione”, devo definire dei criteri di successo. 
Quale problema specifico stiamo risolvendo?

Se il problema è “gli utenti rimangono connessi dopo la modifica della password”:

Piano:
1. Scrivere un test: Modifica password → verificare che la vecchia sessione sia invalidata
   Verifica: il test fallisce (riproduce il bug)

2. Implementare: Invalidare le sessioni al momento della modifica della password
   Verifica: il test supera il test

3. Verificare i casi limite: sessioni attive multiple, modifiche simultanee
   Verifica: i test aggiuntivi superati

4. Verificare l’assenza di regressioni: i test di autenticazione esistenti continuano a superare i test
   Verifica: suite di test completa con esito positivo

Copertura attuale dei test per l’autenticazione: [mostra la copertura]
Qual è il problema specifico di autenticazione che stai riscontrando?
```

### Esempio 2: Procedura in più fasi con verifica

**Richiesta dell’utente:** “Aggiungere la limitazione di velocità all’API”

**❌ Cosa fanno gli LLM (tutto in una volta)**

```python
# Implementa la limitazione della frequenza completa con Redis, strategie multiple,
# sistema di configurazione e monitoraggio in un unico commit di 300 righe
# Nessuna fase di verifica chiara
```

**✅ Cosa dovrebbe accadere (approccio incrementale con verifica)**

```
Piano per la limitazione della frequenza:

1. Aggiungere una limitazione di velocità di base in memoria (singolo endpoint)
   Verifica: 
   - Test: 100 richieste → le prime 10 vanno a buon fine, le restanti ricevono un errore 429
   - Manuale: eseguire `curl` sull’endpoint 11 volte, verificare l’errore di limitazione di velocità

2. Estrarre nel middleware (applicare a tutti gli endpoint)
   Verifica:
   - Test: i limiti di velocità si applicano a /users e /posts
   - I test sugli endpoint esistenti continuano a superare i controlli

3. Aggiungere il backend Redis (per multi-server)
   Verifica:
   - Test: il limite di velocità persiste anche dopo il riavvio dell’app
   - Test: due istanze dell’app condividono il contatore del limite di frequenza

4. Aggiungere la configurazione (frequenze per endpoint)
   Verifica:
   - Test: /search consente 10 richieste al minuto, /users ne consente 100 al minuto
   - Il file di configurazione viene analizzato correttamente

Ogni passaggio è verificabile e implementabile in modo indipendente.
Iniziare dal passaggio 1?
```

### Esempio 3: Verifica "Test-First"

**Richiesta dell’utente:** "L’ordinamento non funziona correttamente in presenza di punteggi duplicati"

**❌ Cosa fanno i modelli di linguaggio di grandi dimensioni (correzione senza riproduzione)**

```python
# Modifica immediatamente la logica di ordinamento senza confermare il bug
def sort_scores(scores):
    return sorted(scores, key=lambda x: (-x['score'], x['name']))
```

**✅ Cosa dovrebbe succedere (riprodurre prima il problema)**

```python
# 1. Per prima cosa, scrivere un test che riproduca il problema
def test_sort_with_duplicate_scores():
    """Verifica l'ordinamento quando più elementi hanno lo stesso punteggio."""
    scores = [
 {'name': 'Alice', 'score': 100},
 {'name': 'Bob', 'score': 100},
        {'name': 'Charlie', 'score': 90},
    ]
 
 result = sort_scores(scores)
 
 # Il bug: l'ordine non è deterministico per i duplicati
    # Esegui questo test più volte, dovrebbe dare risultati coerenti
    assert result[0]['score'] == 100
    assert result[1]['score'] == 100
    assert result[2]['score'] == 90

# Verifica: esegui il test 10 volte → fallisce a causa di un ordinamento incoerente

# 2. Ora correggiamo con un ordinamento stabile
def sort_scores(scores):
    """Ordina per punteggio in ordine decrescente, poi per nome in ordine crescente in caso di parità."""
    return sorted(scores, key=lambda x: (-x['score'], x['name']))

# Verifica: il test viene superato in modo coerente
```

---

## Riepilogo degli anti-pattern

| Principio | Anti-pattern | Soluzione |
|-----------|-------------|-----|
| Pensare prima di programmare | Presuppone tacitamente il formato dei file, i campi e l’ambito | Elencare esplicitamente i presupposti, chiedere chiarimenti |
| Semplicità prima di tutto | Pattern strategico per il calcolo di un singolo sconto | Una sola funzione finché la complessità non è effettivamente necessaria |
| Modifiche chirurgiche | Riformatta le virgolette, aggiunge suggerimenti sui tipi mentre corregge il bug | Modificare solo le righe che risolvono il problema segnalato |
| Approccio orientato all’obiettivo | “Rivedrò e migliorerò il codice” | “Scrivere un test per il bug X → farlo passare → verificare che non ci siano regressioni” |

## Approfondimento chiave

Gli esempi “eccessivamente complicati” non sono palesemente sbagliati: seguono i modelli di progettazione e le migliori pratiche. Il problema è il **tempismo**: aggiungono complessità prima che sia necessaria, il che:

- Rende il codice più difficile da comprendere
- Introduce più bug
- Richiede più tempo per l’implementazione
- Rende più difficile il test

Le versioni “semplici” sono:
- Più facili da comprendere
- Più veloci da implementare
- Più facili da testare
- Possono essere rifattorizzate in un secondo momento, quando la complessità sarà effettivamente necessaria

**Un buon codice è un codice che risolve in modo semplice il problema di oggi, non quello di domani in modo prematuro.**
