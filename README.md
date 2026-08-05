# garcia-sanabria-plant-classifier

Classificatore di immagini che riconosce 6 specie di piante del **Parque García Sanabria**
(Santa Cruz de Tenerife), più una classe "altro" per le piante fuori collezione.

Progetto didattico realizzato come prima esperienza pratica di deep learning applicato,
seguendo l'approccio del corso [fast.ai](https://course.fast.ai/) (Practical Deep Learning
for Coders). Costruito con [fastai](https://docs.fast.ai/) tramite transfer learning, e
messo online come demo interattiva.

> **Nota:** questo progetto è sviluppato in modo *iterativo*. Ogni versione del modello nasce
> dall'analisi degli errori della precedente. Qui sotto sono documentate tutte le versioni,
> non solo l'ultima — perché il percorso di diagnosi e correzione è parte del lavoro.

---

## Cosa fa

Data una foto di una pianta, il modello predice a quale delle 7 categorie appartiene:

| Categoria | Nome scientifico |
|-----------|------------------|
| Drago | *Dracaena draco* |
| Mejorana | *Origanum majorana* |
| Olivo | *Olea europaea* |
| Palmera canaria | *Phoenix canariensis* |
| Alcalifa | *Acalypha* |
| Palmera cola de pescado ramificada | *Caryota* |
| Altro | (piante ed elementi fuori collezione) |

### Distribuzione delle immagini per classe

| Categoria | v1 | v2 |
|-----------|----|----|
| Drago | 27 | 29 |
| Mejorana | 33 | 33 |
| Olivo | 27 | 27 |
| Palmera canaria | 30 | 29 |
| Alcalifa | 23 | 26 |
| Palmera cola de pescado | 27 | 32 |
| Altro | 29 | 30 |
| **Totale** | **196** | **206** |

Il ribilanciamento in v2 non è casuale: le classi rinforzate (palmera cola de pescado, alcalifa)
sono proprio quelle che la diagnosi degli errori aveva individuato come più deboli.

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

- **Dataset:** 196 immagini, 7 classi (23-33 foto per classe).
- **Allenamento:** 8 epoche (scelto iterativamente: 3 -> 5 -> 8, osservando la `valid_loss`).
- **Risultato:** error_rate ~ 0.077 (~92% di accuratezza sul validation set).
- **Diagnosi post-test:** testando il modello sul campo sono emersi problemi non visibili
  nell'error_rate. In particolare il modello tendeva ad "allucinare" la classe `drago` in
  presenza di **sfondo uniforme (cielo)**, e faticava sui **tronchi** e sulla **palmera cola
  de pescado**. Il ~92% risultava inoltre gonfiato dalla presenza di scatti simili divisi tra
  train e validation.

### v2 — correzione basata sulla diagnosi (05/08/2026)

- **Dataset:** 206 immagini. Modifiche mirate rispetto a v1, tutte derivanti dall'analisi
  degli errori:
  - rimosse foto di `drago` e `palmera_canaria` con troppo cielo nello sfondo (ipotesi: il
    modello associava il cielo alla classe `drago`);
  - rimossi alcuni scatti quasi-gemelli da varie classi;
  - aggiunte foto di `drago` senza cielo o con cielo ridotto;
  - aggiunte foto "d'insieme" alle classi che venivano riconosciute male se inquadrate per intero;
  - arricchita la classe `altro` (piante biancastre, oggetti) e ribilanciata rimuovendo
    alcuni alberi in eccesso.
- **Allenamento:** 8 epoche.
- **Risultato:** error_rate ~ 0.122 (~88% sul validation set).

> **Perché v2 ha un numero più basso di v1 pur essendo migliore.** Il validation set di v2 è
> più *pulito* (meno scatti quasi-gemelli) e più *severo* (foto più varie e difficili). Un
> numero più basso su un test più onesto non indica un modello peggiore: il ~92% di v1 era in
> parte gonfiato. La verifica reale dei miglioramenti (es. il problema del cielo/drago) si fa
> testando sul campo, non leggendo l'error_rate.

---

## Demo online

Il modello è consultabile come demo interattiva: l'utente carica o scatta una foto di una
pianta e riceve la predizione con le probabilità per classe.

- Piattaforma: **Netlify** (il modello gira nel browser via **ONNX** / onnxruntime-web).
- Link: (https://planthuntsanabria.netlify.app/)

La demo è pensata come progetto didattico, resa disponibile per scopo dimostrativo.

---

## Struttura del repository

- `notebook.ipynb` — notebook di allenamento (preparazione dati, training, valutazione).
- `registro_allenamenti.md` — tabella di tutti gli esperimenti, per ogni versione, con letture della matrice di confusione.
- `diario_decisioni.md` — le scelte metodologiche del progetto e il loro perché (decision log).
- `README.md` — questo file.

**Non** incluso nel repository (dati e modelli non vanno versionati su GitHub):

- Le immagini dei dataset (v1 e v2) -> conservate a parte; entrambe le versioni sono mantenute per riproducibilità.
- I modelli allenati (`.pkl` / `.onnx`) -> conservati a parte.

---

## Limiti noti

Progetto didattico con dataset piccolo; i risultati vanno letti con onestà:

- **Validation set ridotto:** l'accuratezza è indicativa ma statisticamente rumorosa.
- **Poche immagini per classe:** margine di miglioramento raccogliendo più dati, soprattutto
  per le classi più deboli (es. palmera cola de pescado).
- **Sensibilità allo sfondo e alle inquadrature parziali** (tronchi isolati): affrontata in
  parte in v2, ancora in corso di verifica sul campo.
- Alcune confusioni sono coerenti con la reale somiglianza visiva delle specie, non con errori
  di etichettatura.
- **Miglioramenti sperimentali pianificati:** le modifiche di v2 sono state introdotte tutte
  insieme; un ciclo futuro più rigoroso isolerebbe una variabile per volta (es. solo lo sfondo
  del drago) per attribuire con certezza i miglioramenti. Entrambi i dataset sono conservati
  proprio per rendere possibile questa verifica.

---

## Nota sulla trasparenza

Gli script Python usati per rinominare in blocco i file dei dataset sono stati sviluppati con
l'assistenza di un tutor AI. Sono stati usati comprendendone lo scopo, e per questo motivo non
sono inclusi nel repository. La preparazione concettuale dei dataset (selezione, verifica delle
etichette, criteri di scarto), la diagnosi degli errori e l'analisi dei risultati sono farina
del mio sacco.

---

## Licenza

Codice rilasciato sotto licenza **MIT** (vedi [`LICENSE`](LICENSE)).
Le immagini dei dataset, quando pubblicate, avranno una licenza propria indicata nella loro pagina.
