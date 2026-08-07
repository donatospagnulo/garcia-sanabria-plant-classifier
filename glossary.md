# Glossary · Glossario

Definitions of the terms used across [`README.md`](README.md),
[`diario_decisioni.md`](diario_decisioni.md) and [`registro_allenamenti.md`](registro_allenamenti.md).

*Definizioni dei termini usati nei tre documenti. Versione italiana più sotto.*

---

## English

### Project-specific terms

These are my own words, used consistently across the documents.

- **Object** — an artificial, human or animal element: anything that isn't a plant. Used in the
  field tests to check that the model rejects non-plants.
- **Natural** — a context of plants, trees or flowers.
- **Whole-plant shot** (*scatto d'insieme*) — a photo framing the entire plant: trunk, branches
  and foliage together. The opposite of a detail or close-up shot.
- **Generic trunk** — a photo of a trunk belonging to no target species, used to test whether
  the model wrongly assigns it to one of them.
- **NOT A MATCH** — the outcome when the model predicts `altro`, or when its confidence falls
  below the threshold. In the game this means "this isn't one of the six".
- **Incoming false positives** (*"da altro", "da drago"*) — in the test breakdowns, photos of
  *another* category that the model wrongly assigned to the class being listed. Reading them
  alongside the correct predictions shows which classes act as magnets.
- **Attractor** — a class the model over-predicts, pulling in photos that belong elsewhere. In
  v1, `drago` was an attractor.
- **Field test** — testing the model by photographing live in the park, as opposed to measuring
  it on the validation set. It is the only way the sky/`drago` problem became visible.

### Machine learning terms

- **Transfer learning** — starting from a network already trained on millions of generic images
  and adapting it to a new, much smaller task. It's what makes ~200 photos enough: the network
  already knows edges, textures and shapes, and only has to learn what distinguishes these six
  plants.
- **ResNet34** — the pre-trained network used as a starting point. The 34 refers to its depth
  in layers. Deeper networks (ResNet50, ResNet101) hold more capacity but overfit more easily
  on small datasets.
- **fastai** — the library used for training, built on top of PyTorch.
- **`fine_tune`** — the fastai command that carries out the transfer learning: it first adapts
  the final layer to the new classes, then adjusts the rest of the network more gently.
- **Epoch** — one complete pass of the training set through the network. Eight epochs means the
  model has seen every photo eight times.
- **Data augmentation** (`aug_transforms()`) — automatically generating variants of each
  training photo (rotations, crops, brightness and contrast changes) so the model sees more
  variety than the photos alone provide. Standard practice when data is scarce.
- **Training set / validation set** — the photos the model learns from, versus the photos held
  back to measure it. The model never trains on the validation set: that's what makes the
  measurement meaningful.
- **`valid_pct=0.2`** — the fraction held back for validation, here 20%.
- **Seed** — a fixed number that makes the random train/validation split reproducible. With the
  same seed you get the same split, so two runs are comparable.
- **`error_rate`** — the share of validation photos the model gets wrong. 0.122 means about 12%
  wrong, i.e. roughly 88% accuracy.
- **`train_loss` / `valid_loss`** — how badly the model is wrong on the training photos and on
  the validation photos respectively. Loss is more informative than error_rate because it
  accounts for confidence: a confidently wrong answer costs more than a hesitant one.
  `valid_loss` flattening out is the signal that further epochs would add little.
- **Overfitting** — when a model memorises the training photos instead of learning the general
  features, and so performs well in training and poorly on anything new. The main risk with
  small datasets.
- **Confusion matrix** — a table cross-referencing true class against predicted class. The
  diagonal holds the correct predictions; everything off the diagonal is an error, and where it
  sits tells you *which* classes get mixed up with which.
- **`most_confused`** — the fastai command that lists the off-diagonal cells of the confusion
  matrix in descending order, i.e. the most frequent confusions.
- **Near-duplicates** (*quasi-gemelli*) — several photos of the same individual specimen taken
  moments apart. If a random split puts some in training and some in validation, the model is
  effectively being tested on photos it has almost already seen, which inflates the score.
- **Shortcut learning** — when a model latches onto an incidental correlation rather than the
  relevant feature: for example, learning "sky = drago" because half the drago photos have sky
  behind them. It's the pathology this project spent most of its diagnostic effort on.
- **Hallucinating** (used informally here) — the model confidently assigning a known class to
  something that isn't one of them, e.g. calling a shoe a `drago`.
- **Threshold** — the minimum confidence required before the game accepts a prediction as a
  find. Set to 0.55 on the site: below that, the answer is treated as "no match".
- **Softmax** — the operation converting the model's raw outputs into percentages that add up to
  100%. It's what produces the "98% confident" figure shown in the game.
- **Class** / **category** — one of the labels the model can predict. Here there are 7: six
  plants plus `altro`.
- **ONNX** — an open format for saving a trained model so it can run outside the library that
  produced it. It's what allows the model to run inside a browser rather than on a server.
- **onnxruntime-web** — the JavaScript library that executes the ONNX model in the browser.
- **Normalisation** (mean/std) — recentring the pixel values before feeding them to the model.
  It must be done identically at training time and at prediction time, otherwise the model
  receives inputs unlike anything it was trained on.

---

## Italiano

### Termini specifici del progetto

Parole mie, usate in modo coerente in tutti i documenti.

- **Oggetti** — elementi artificiali, umani, animali: tutto ciò che non è una pianta. Usati nei
  test sul campo per verificare che il modello rifiuti ciò che non è pianta.
- **Naturale** — in circostanza di piante, alberi o fiori.
- **Scatto d'insieme** — foto che inquadra la pianta intera: tronco, rami e chioma insieme. Il
  contrario del dettaglio o del primo piano.
- **Tronco generico** — foto di un tronco che non appartiene a nessuna specie nota, usata per
  verificare se il modello lo assegna erroneamente a una di esse.
- **NOT A MATCH** — l'esito quando il modello predice `altro`, oppure quando la sua confidenza
  scende sotto la soglia. Nel gioco significa "non è nessuna delle sei".
- **"Da altro", "da drago"** — nei conteggi dei test, le foto di *un'altra* categoria che il
  modello ha assegnato per errore alla classe elencata. Lette accanto alle predizioni corrette
  mostrano quali classi fanno da calamita.
- **Attrattore** — una classe che il modello predice troppo spesso, tirando a sé foto che
  appartengono altrove. Nella v1, `drago` era un attrattore.
- **Test sul campo** — provare il modello fotografando dal vivo nel parco, invece di misurarlo
  sul validation set. È l'unico modo in cui il problema cielo/`drago` è diventato visibile.

### Termini di machine learning

- **Transfer learning** — partire da una rete già allenata su milioni di immagini generiche e
  adattarla a un compito nuovo e molto più piccolo. È ciò che rende sufficienti ~200 foto: la
  rete conosce già bordi, texture e forme, e deve solo imparare cosa distingue queste sei piante.
- **ResNet34** — la rete pre-allenata usata come punto di partenza. Il 34 indica la profondità
  in strati. Reti più profonde (ResNet50, ResNet101) hanno più capacità ma vanno più facilmente
  in overfitting su dataset piccoli.
- **fastai** — la libreria usata per l'allenamento, costruita sopra PyTorch.
- **`fine_tune`** — il comando fastai che esegue il transfer learning: prima adatta l'ultimo
  strato alle nuove classi, poi ritocca più delicatamente il resto della rete.
- **Epoca** — un passaggio completo del training set attraverso la rete. Otto epoche significa
  che il modello ha visto ogni foto otto volte.
- **Data augmentation** (`aug_transforms()`) — generare automaticamente varianti di ogni foto di
  training (rotazioni, ritagli, variazioni di luminosità e contrasto) perché il modello veda più
  varietà di quanta le foto da sole ne offrano. Prassi standard quando i dati sono pochi.
- **Training set / validation set** — le foto da cui il modello impara, contro le foto tenute da
  parte per misurarlo. Il modello non si allena mai sul validation set: è questo che rende la
  misura significativa.
- **`valid_pct=0.2`** — la frazione tenuta da parte per la validazione, qui il 20%.
- **Seed** — numero fissato che rende riproducibile la divisione casuale train/validation. Con
  lo stesso seed si ottiene la stessa divisione, quindi due prove sono confrontabili.
- **`error_rate`** — la quota di foto di validation che il modello sbaglia. 0.122 significa
  circa 12% di errori, cioè circa 88% di accuratezza.
- **`train_loss` / `valid_loss`** — quanto gravemente il modello sbaglia sulle foto di training
  e su quelle di validation. La loss è più informativa dell'error_rate perché tiene conto della
  confidenza: sbagliare con sicurezza costa più che sbagliare con esitazione. L'appiattirsi
  della `valid_loss` è il segnale che altre epoche aggiungerebbero poco.
- **Overfitting** — quando un modello memorizza le foto di training invece di imparare le
  caratteristiche generali, e quindi va bene in allenamento e male su tutto ciò che è nuovo. È
  il rischio principale con dataset piccoli.
- **Matrice di confusione** — tabella che incrocia classe vera e classe predetta. Sulla diagonale
  stanno le predizioni corrette; tutto ciò che è fuori diagonale è un errore, e la sua posizione
  dice *quali* classi vengono scambiate con quali.
- **`most_confused`** — il comando fastai che elenca le celle fuori diagonale in ordine
  decrescente, cioè le confusioni più frequenti.
- **Quasi-gemelli** — più foto dello stesso esemplare scattate a pochi istanti di distanza. Se
  una divisione casuale ne mette alcune in training e altre in validation, il modello viene di
  fatto testato su foto che ha quasi già visto, e il punteggio risulta gonfiato.
- **Scorciatoia visiva** (*shortcut learning*) — quando il modello si aggancia a una correlazione
  accidentale invece che alla caratteristica rilevante: per esempio imparare "cielo = drago"
  perché metà delle foto del drago hanno il cielo dietro. È la patologia a cui questo progetto ha
  dedicato la maggior parte del lavoro diagnostico.
- **Allucinare** (uso informale) — il modello che assegna con sicurezza una classe nota a
  qualcosa che non lo è, per esempio chiamare `drago` una scarpa.
- **Soglia** — la confidenza minima richiesta perché il gioco accetti una predizione come
  ritrovamento. Sul sito è 0.55: sotto quel valore la risposta diventa "nessuna corrispondenza".
- **Softmax** — l'operazione che converte le uscite grezze del modello in percentuali che
  sommano a 100%. È ciò che produce il "98% di confidenza" mostrato nel gioco.
- **Classe** / **categoria** — una delle etichette che il modello può predire. Qui sono 7: sei
  piante più `altro`.
- **ONNX** — formato aperto per salvare un modello allenato in modo che funzioni fuori dalla
  libreria che l'ha prodotto. È ciò che permette al modello di girare dentro un browser invece
  che su un server.
- **onnxruntime-web** — la libreria JavaScript che esegue il modello ONNX nel browser.
- **Normalizzazione** (mean/std) — ricentrare i valori dei pixel prima di darli al modello. Va
  fatta in modo identico in allenamento e in predizione, altrimenti il modello riceve input
  diversi da tutto ciò su cui è stato allenato.
