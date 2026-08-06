# garcia-sanabria-plant-classifier

Classificatore di immagini che riconosce 6 specie di piante del **Parque García Sanabria**
(Santa Cruz de Tenerife), più una classe "altro" per le piante e gli elementi fuori collezione.

Progetto didattico realizzato come prima esperienza pratica di deep learning applicato,
seguendo l'approccio del corso [fast.ai](https://course.fast.ai/) (Practical Deep Learning
for Coders). Costruito con [fastai](https://docs.fast.ai/) tramite transfer learning, e
messo online come gioco interattivo.

> **Nota:** questo progetto è sviluppato in modo *iterativo*. Ogni versione del modello nasce
> dall'analisi degli errori della precedente, raccolti **testando sul campo**, non solo
> leggendo l'error_rate. Qui sotto sono documentate tutte le versioni, non solo quella online —
> perché il percorso di diagnosi e correzione è parte del lavoro.

**Demo giocabile:** https://planthuntsanabria.netlify.app/

---

## Cosa fa

Data una foto di una pianta, il modello predice a quale delle 7 categorie appartiene:

| Categoria | Nome scientifico |
|-----------|------------------|
| Drago | *Dracaena draco* |
| Mejorana | *Origanum majorana* |
| Olivo | *Olea europaea* |
| Palmera canaria | *Phoenix canariensis* |
| Alcalifa | *Acalypha wilkesiana* |
| Palmera cola de pescado ramificada | *Caryota* |
| Altro | (piante ed elementi fuori collezione) |

### Distribuzione delle immagini per classe

| Categoria | v1 | v2 | v3 |
|-----------|----|----|----|
| Drago | 27 | 29 | 29 |
| Mejorana | 33 | 33 | 33 |
| Olivo | 27 | 27 | 27 |
| Palmera canaria | 30 | 29 | 29 |
| Alcalifa | 23 | 26 | 26 |
| Palmera cola de pescado | 27 | 32 | — |
| Altro | 29 | 30 | 30 |
| **Totale** | **196** | **206** | **174** |

Il ribilanciamento in v2 non è casuale: le classi rinforzate (palmera cola de pescado, alcalifa)
sono proprio quelle che la diagnosi degli errori aveva individuato come più deboli. La v3 non è
un dataset nuovo, ma la v2 privata di una classe, per isolarne l'effetto.

---

## Metodo (comune a tutte le versioni)

- **Transfer learning** a partire da **ResNet34** pre-allenata.
- **Data augmentation** (`aug_transforms()`) per compensare il dataset ridotto.
- Immagini ridimensionate a **224×224**.
- Divisione train/validation: **80/20** (`valid_pct=0.2`, `seed=42`).
- Allenamento con `fine_tune`, numero di epoche scelto sperimentalmente.

---

## Versioni del modello

### v1 — baseline (03/08/2026)

- **Dataset:** 196 immagini, 7 classi (23–33 foto per classe).
- **Allenamento:** 8 epoche (scelto iterativamente: 3 → 5 → 8, osservando la `valid_loss`).
- **Risultato:** error_rate ≈ 0.077 (~92% sul validation set, ~39 foto).
- **Diagnosi post-test:** testando il modello sul campo sono emersi problemi non visibili
  nell'error_rate. In particolare il modello tendeva ad "allucinare" la classe `drago` in
  presenza di **sfondo uniforme (cielo)**, e faticava sui **tronchi** e sulla **palmera cola
  de pescado**. Il ~92% risultava inoltre gonfiato dalla presenza di scatti quasi-gemelli
  divisi tra train e validation.

### v2 — correzione basata sulla diagnosi (06/08/2026)

- **Dataset:** 206 immagini. Modifiche mirate rispetto a v1, tutte derivanti dall'analisi
  degli errori:
  - rimosse foto di `drago` e `palmera_canaria` con troppo cielo nello sfondo (ipotesi: il
    modello associava il cielo alla classe `drago`);
  - rimossi alcuni scatti quasi-gemelli da varie classi;
  - aggiunte foto di `drago` senza cielo o con cielo ridotto;
  - aggiunte foto "d'insieme" alle classi che venivano riconosciute male se inquadrate per intero;
  - arricchita la classe `altro` (piante biancastre, oggetti) e ribilanciata rimuovendo
    alcuni tronchi in eccesso.
- **Allenamento:** 8 epoche.
- **Risultato:** error_rate ≈ 0.122 (~88% sul validation set, ~41 foto).
- **È la versione attualmente online.**

> **Perché v2 ha un numero più basso di v1 pur essendo migliore.** Il validation set di v2 è
> più *pulito* (meno scatti quasi-gemelli) e più *severo* (foto più varie e difficili). Un
> numero più basso su un test più onesto non indica un modello peggiore: il ~92% di v1 era in
> parte gonfiato. La verifica reale dei miglioramenti (es. il problema del cielo/drago) si fa
> testando sul campo, non leggendo l'error_rate.

### v3 — esperimento a 6 classi, scartato (06/08/2026)

- **Dataset:** 174 immagini — la v2 **senza** la classe `palmera cola de pescado ramificada`.
- **Motivazione:** quella specie risultava la più problematica nei test sul campo
  (5 riconoscimenti su 22 tentativi). L'ipotesi era che togliendola si semplificasse il
  problema e si riducessero le confusioni sulle altre classi.
- **Risultato:** error_rate ≈ 0.206 (~79% sul validation set, ~35 foto) — il valore più basso
  delle tre versioni.
- **Cosa ha risolto:** i **tronchi generici** non venivano più attribuiti a `olivo` o
  `palmera_canaria`, ma correttamente respinti come "nessuna corrispondenza". È il problema
  che la v2 non era riuscita a chiudere.
- **Cosa ha rotto:** la `palmera_canaria` è passata da affidabile a **riconosciuta 1 volta su
  26**, a parità di condizioni e tipologia di scatto rispetto alla v2.
- **Decisione:** versione scartata. Il guadagno sui tronchi non compensava una regressione così
  netta su una delle specie principali.

> **Nota metodologica sui confronti.** I tre valori di accuratezza (92% / 88% / 79%) **non sono
> direttamente confrontabili**: ogni versione ha un dataset diverso e quindi un validation set
> diverso, per composizione e difficoltà. Servono a seguire l'andamento di ogni singola
> versione, non a stabilire una classifica tra loro.

---

## Test sul campo

L'error_rate da solo si è rivelato insufficiente a descrivere il comportamento reale del
modello. Per questo ogni versione è stata testata fotografando dal vivo, e i risultati sono
registrati per intero nel diario.

| Test | Modello | Foto | Cosa ha rivelato |
|------|---------|------|------------------|
| Test 1 | v1 | 94 | Il `drago` veniva predetto anche per oggetti (scarpe, chiavi, bottiglie) su superfici uniformi. Incrociando i dati, l'indiziato è risultato il **cielo** presente in circa metà delle foto di training della classe. Tronchi e palmera cola de pescado già problematici. |
| Test 2 | v2 | 126 | Il problema degli oggetti è **rientrato**: ora finiscono quasi sempre in `altro`. `Drago`, `olivo` e `palmera canaria` vengono riconosciuti dagli **scatti d'insieme**. Restano deboli i tronchi isolati e la palmera cola de pescado (5/22). |
| Test 3 | v3 | — | Tronchi generici correttamente respinti, ma `palmera canaria` riconosciuta 1 volta su 26. Versione scartata. |

**Totale: 220 fotografie di test**, oltre a quelle di training.

Osservazioni ricorrenti emerse dai test:

- **Gli scatti d'insieme funzionano, i dettagli no.** Fa eccezione la palmera cola de pescado,
  per cui i primi piani sulle foglie danno risultati migliori dello scatto intero.
- **I tronchi isolati sono il punto debole generale**, condiviso da più classi.
- La `mejorana` viene confusa con altre piante piccole e verdi molto simili: comportamento
  accettato, non è un errore di etichettatura.
- L'`alcalifa` viene riconosciuta male quando nell'inquadratura sono presenti piante di
  colorazione diversa dalla sua.

---

## Demo online

Il modello è pubblicato non come classificatore nudo, ma come **caccia al tesoro botanica**:
l'utente fotografa le piante del parco e il modello conferma se ha trovato una delle sei
specie. Trovandole tutte, si sblocca un'immagine riepilogativa condivisibile.

- **Link:** https://planthuntsanabria.netlify.app/
- **Piattaforma:** Netlify. Il modello gira **interamente nel browser** via **ONNX**
  (onnxruntime-web): nessun server, nessuna foto caricata o conservata altrove.
- **Modello impiegato:** v2 (7 categorie).
- **Pulsante di conferma manuale:** vista la difficoltà documentata sulla palmera cola de
  pescado, dopo un tentativo fallito l'utente può confermare manualmente di trovarsi davanti a
  quella pianta. La scelta è deliberata e limitata a quella sola specie: serve a non bloccare
  il gioco davanti a un limite noto del modello, ed è dichiarata apertamente nel sito.

La pagina dichiara che si tratta di un progetto didattico, riporta i numeri reali (206 foto di
training, 220 di test, ~88%) e spiega quali inquadrature funzionano meglio, in modo che
l'utente sia guidato invece che lasciato a indovinare.

---

## Struttura del repository

- `notebook.ipynb` — notebook di allenamento (preparazione dati, training, valutazione).
- `registro_allenamenti.md` — tabella di tutti gli esperimenti, per ogni versione, con letture
  della matrice di confusione.
- `diario_decisioni.md` — le scelte metodologiche del progetto e il loro perché (decision log),
  compresi i risultati grezzi dei test sul campo.
- `README.md` — questo file.

**Non** incluso nel repository (dati e modelli non vanno versionati su GitHub):

- Le immagini dei dataset (v1, v2, v3) → conservate a parte; tutte le versioni sono mantenute
  per riproducibilità.
- I modelli allenati (`.pkl` / `.onnx`) → conservati a parte.

---

## Limiti noti

Progetto didattico con dataset piccolo; i risultati vanno letti con onestà:

- **Validation set ridotto** (35–41 foto secondo la versione): l'accuratezza è indicativa ma
  statisticamente rumorosa, e non confrontabile tra versioni diverse.
- **Poche immagini per classe:** margine di miglioramento raccogliendo più dati, soprattutto
  per le classi più deboli (palmera cola de pescado in primis).
- **Sensibilità allo sfondo e alle inquadrature parziali** (tronchi isolati): affrontata in
  parte in v2, non ancora risolta.
- **Palmera cola de pescado:** riconosciuta in modo inaffidabile (5/22 sul campo). I fusti
  sottili si confondono con la vegetazione circostante. Mitigata nel gioco con la conferma
  manuale, non risolta nel modello.
- Alcune confusioni sono coerenti con la reale somiglianza visiva delle specie, non con errori
  di etichettatura.
- **Modifiche non isolate:** le correzioni di v2 sono state introdotte tutte insieme, quindi
  l'ipotesi "cielo → drago" resta plausibile ma non dimostrata. Un ciclo futuro più rigoroso
  cambierebbe una variabile per volta, sostituendo alla v1 la sola classe `drago` della v2.
  Tutti i dataset sono conservati proprio per rendere possibile questa verifica.

---

## Prossimi passi

- Verificare in isolamento l'ipotesi dello sfondo uniforme sulla classe `drago`.
- Raccogliere più foto di tronchi per le classi che ne hanno poche (`drago`, `olivo`) e
  riequilibrare la classe `altro`, che ne contiene in eccesso.
- Rivedere la strategia sulla palmera cola de pescado: più scatti d'insieme in contesti
  diversi, oppure ridefinire il caso d'uso sui soli primi piani fogliari.
- Sostituire la divisione casuale train/validation con una divisione **per esemplare**, per
  evitare che scatti quasi-gemelli finiscano da entrambe le parti.

---

## Nota sulla trasparenza

Gli script Python usati per rinominare in blocco i file dei dataset sono stati sviluppati con
l'assistenza di un tutor AI, così come il sito della demo. Sono stati usati comprendendone lo
scopo, e per questo motivo gli script non sono inclusi nel repository. La preparazione
concettuale dei dataset (selezione, verifica delle etichette, criteri di scarto), la
progettazione ed esecuzione dei test sul campo, la diagnosi degli errori e l'analisi dei
risultati sono farina del mio sacco.

---

## Licenza

Codice rilasciato sotto licenza **MIT** (vedi [`LICENSE`](LICENSE)).
Le immagini dei dataset, quando pubblicate, avranno una licenza propria indicata nella loro pagina.
