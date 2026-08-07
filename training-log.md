# Training log

*English version below, [Italian original further down](#registro-degli-allenamenti). Unfamiliar terms are defined in [`GLOSSARY.md`](GLOSSARY.md).*

> **What this file is for.** Every time I train the model, I add a row.
> The point is to make each experiment *repeatable* and *comparable*: without these notes,
> when one training run goes better than another I won't be able to say why, and I won't be
> able to reproduce it.
> Rule: I fill it in IMMEDIATELY after each run, not "later". Better spartan but always current.

---

## How to read the table

- **Conditions (input):** the choices I made, which determine the result.
- **Results (output):** the numbers the model gives back.
- **Note / conclusion:** the most important column — why I ran this experiment and what I conclude.

---

## Table

DATASET V1 — 196 images

| # | Date | Architecture | Epochs | Image size | Augmentation | Seed | valid_pct | error_rate | train_loss | valid_loss | Note / conclusion |
|---|------|--------------|--------|------------|--------------|------|-----------|------------|------------|------------|-------------------|
| 1 | 03/08 | resnet34 | 3 | 224 | aug_transforms | 42 | 0.2 | 0.384615 | 2.028400 | 0.987559 | First run, to get an overall picture. Room to improve. |
| 2 | 03/08 | resnet34 | 5 | 224 | aug_transforms | 42 | 0.2 | 0.153846 | 1.392782 | 0.535854 | error_rate was dropping sharply — 3 epochs were too few. |
| 3 | 03/08 | resnet34 | 8 | 224 | aug_transforms | 42 | 0.2 | 0.076923 | 1.031007 | 0.246219 | valid_loss hadn't flattened yet, so there was likely still room to improve. |

DATASET V2 — 206 images

| # | Date | Architecture | Epochs | Image size | Augmentation | Seed | valid_pct | error_rate | train_loss | valid_loss | Note / conclusion |
|---|------|--------------|--------|------------|--------------|------|-----------|------------|------------|------------|-------------------|
| 4 | 06/08 | resnet34 | 8 | 224 | aug_transforms | 42 | 0.2 | 0.121951 | 1.036307 | 0.381629 | The dataset was rebuilt following the observations from field test 1. |

DATASET V3 — 174 images

| # | Date | Architecture | Epochs | Image size | Augmentation | Seed | valid_pct | error_rate | train_loss | valid_loss | Note / conclusion |
|---|------|--------------|--------|------------|--------------|------|-----------|------------|------------|------------|-------------------|
| 5 | 06/08 | resnet34 | 8 | 224 | aug_transforms | 42 | 0.2 | 0.205882 | 0.930928 | 0.375073 | This is the model without `palmera cola de pescado ramificada`. |

---

## Main confusions (`most_confused`) per run

> Here I paste the `most_confused` output of each run and write my reading of it alongside.

### Run #1

Confusions:

```
[('palmera_cola_pescado_ramificada', 'drago', 3),
 ('alcalifa', 'altro', 2),
 ('olivo', 'drago', 2),
 ('palmera_canaria', 'drago', 2),
 ('altro', 'drago', 1),
 ('altro', 'olivo', 1),
 ('drago', 'palmera_canaria', 1),
 ('mejorana', 'olivo', 1),
 ('olivo', 'palmera_cola_pescado_ramificada', 1),
 ('palmera_canaria', 'altro', 1)]
```

**My reading:** `drago` shows up almost everywhere. It acts as an attractor, above all for the
other trees — probably the trunk pulls the olive tree towards it, and the leaves pull the palms.

### Run #2

Confusions:

```
[('alcalifa', 'mejorana', 1),
 ('altro', 'drago', 1),
 ('altro', 'palmera_cola_pescado_ramificada', 1),
 ('drago', 'palmera_canaria', 1),
 ('olivo', 'palmera_cola_pescado_ramificada', 1),
 ('palmera_cola_pescado_ramificada', 'drago', 1)]
```

**My reading:** there's still some confusion among the trees, but the diagonal is far heavier
than before — very little gets mixed up now. The model is learning; I can try a few more epochs.

### Run #3

Confusions:

```
[('altro', 'drago', 1),
 ('drago', 'palmera_canaria', 1),
 ('mejorana', 'olivo', 1)]
```

**My reading:** the "trees vs drago" confusions cleared up with more training, but
mejorana/olivo persists. That suggests a more structural difficulty (similar green leaves?)
that epochs alone won't cure.

The confusions look acceptable now: accuracy is up to roughly 92% and valid_loss has almost
flattened. I'll stop here.

The validation set is about 39 photos, so the final error_rate (0.076, roughly 3 wrong photos)
is indicative but noisy. Improvements below that threshold wouldn't be statistically reliable.

### Run #4

Confusions:

```
[('altro', 'alcalifa', 1),
 ('altro', 'drago', 1),
 ('altro', 'mejorana', 1),
 ('olivo', 'palmera_cola_pescado_ramificada', 1),
 ('palmera_canaria', 'drago', 1)]
```

**My reading:** the error rate rose to about 12%. I don't think this is a real problem, since
the near-duplicate images had probably inflated the earlier figure. I'll run a second field
test on the new model and draw my conclusions from that.

8 epochs seems enough: the error rate had settled and valid_loss had almost flattened.

### Run #5

Confusions:

```
[('alcalifa', 'altro', 3),
 ('altro', 'alcalifa', 1),
 ('olivo', 'drago', 1),
 ('palmera_canaria', 'altro', 1),
 ('palmera_canaria', 'drago', 1)]
```

**My reading:** accuracy dropped to about 79% and the confusions look worse. At this point what
I'll do is simply test the model in the field and analyse the data I collect.

---

## Column reminder (what to write in each)

- **Architecture:** which pre-trained network (e.g. resnet34, resnet18).
- **Epochs:** how many complete passes (the number passed to `fine_tune`).
- **Image size:** the `Resize` dimension (e.g. 224).
- **Augmentation:** yes/no and which (e.g. `aug_transforms()`), or "none".
- **Seed:** the fixed number that makes the train/validation split repeatable.
- **valid_pct:** the fraction held back for validation (e.g. 0.2).
- **error_rate / train_loss / valid_loss:** the values from the LAST epoch.
- **Note:** the reason for the run and the conclusion. Without this, the row says nothing.

---
---

# 🇮🇹 Versione originale in italiano

# Registro degli allenamenti

> **A cosa serve questo file.** Ogni volta che alleno il modello, aggiungo una riga.
> Serve a rendere ogni esperimento *ripetibile* e *confrontabile*: senza queste note,
> quando un allenamento va meglio di un altro non saprò dire perché, e non potrò rifarlo.
> Regola: lo compilo SUBITO dopo ogni allenamento, non "dopo". Meglio spartano ma sempre aggiornato.

---

## Come si legge questa tabella

- **Condizioni (input):** le scelte che ho fatto io e che determinano il risultato.
- **Risultati (output):** i numeri che il modello mi restituisce.
- **Nota / conclusione:** la colonna più importante — perché ho fatto questa prova e cosa ne concludo.

---

## Tabella

DATASET V1 - 196 immagini

| # | Data | Architettura | Epoche | Image size | Augmentation | Seed | valid_pct | error_rate | train_loss | valid_loss | Nota / conclusione (perché questa prova, cosa concludo) |
|---|------|--------------|--------|------------|--------------|------|-----------|------------|------------|------------|----------------------------------------------------------|
| 1 |03/08 |   resnet34   |   3    |     224    |aug_transforms|  42  |    0.2    |  0.384615  |  2.028400  |  0.987559  |prima prova, per avere una visione generale, da migliorare|
| 2 |03/08 |   resnet34   |   5    |     224    |aug_transforms|  42  |    0.2    |  0.153846  |  1.392782  |  0.535854  |  error_rate scendeva notevolmente, 3 epoche erano poche  |
| 3 |03/08 |   resnet34   |   8    |     224    |aug_transforms|  42  |    0.2    |  0.076923  |  1.031007  |  0.246219  |valid_loss non si era ancora fermato, probabile margine di miglioramento|

DATASET V2 - 206 immagini

| # | Data | Architettura | Epoche | Image size | Augmentation | Seed | valid_pct | error_rate | train_loss | valid_loss | Nota / conclusione (perché questa prova, cosa concludo) |
|---|------|--------------|--------|------------|--------------|------|-----------|------------|------------|------------|----------------------------------------------------------|
| 4 |06/08 |   resnet34   |   8    |     224    |aug_transforms|  42  |    0.2    |  0.121951  |  1.036307  |  0.381629  |il dataset è stato aggiornato in seguito a osservazioni derivanti dal 1' test|

DATASET V3 - 174 immagini

| # | Data | Architettura | Epoche | Image size | Augmentation | Seed | valid_pct | error_rate | train_loss | valid_loss | Nota / conclusione (perché questa prova, cosa concludo) |
|---|------|--------------|--------|------------|--------------|------|-----------|------------|------------|------------|----------------------------------------------------------|
| 5 |06/08 |   resnet34   |   8    |     224    |aug_transforms|  42  |    0.2    |  0.205882  |  0.930928  |  0.375073  |questo è il modello senza il palmera cola de pescado ramificada|

---

## Confusioni principali (most_confused) per ogni prova

> Qui incollo l'output di `most_confused` di ogni allenamento e ci scrivo accanto la mia lettura.

### Prova #1
- Confusioni:
[('palmera_cola_pescado_ramificada', 'drago', np.int64(3)),
 ('alcalifa', 'altro', np.int64(2)),
 ('olivo', 'drago', np.int64(2)),
 ('palmera_canaria', 'drago', np.int64(2)),
 ('altro', 'drago', np.int64(1)),
 ('altro', 'olivo', np.int64(1)),
 ('drago', 'palmera_canaria', np.int64(1)),
 ('mejorana', 'olivo', np.int64(1)),
 ('olivo', 'palmera_cola_pescado_ramificada', np.int64(1)),
 ('palmera_canaria', 'altro', np.int64(1))]

- La mia lettura:
Il "drago" compare quasi ovunque, fa da attrattore soprattutto con gli altri alberi, probabilmente per via del fusto con l'ulivo e delle foglie con le palme.

### Prova #2
- Confusioni:
[('alcalifa', 'mejorana', np.int64(1)),
 ('altro', 'drago', np.int64(1)),
 ('altro', 'palmera_cola_pescado_ramificada', np.int64(1)),
 ('drago', 'palmera_canaria', np.int64(1)),
 ('olivo', 'palmera_cola_pescado_ramificada', np.int64(1)),
 ('palmera_cola_pescado_ramificada', 'drago', np.int64(1))]

- La mia lettura:
C'è ancora un po' di confusione tra gli alberi, ma la diagonale è estremamente più carica rispetto a prima, ci si confonde veramente poco, il modello sta imparando, posso ancora provare con qualche epoca in più.

### Prova #3
- Confusioni:
[('altro', 'drago', np.int64(1)),
 ('drago', 'palmera_canaria', np.int64(1)),
 ('mejorana', 'olivo', np.int64(1))]

- La mia lettura:
Le confusioni "alberi vs drago" si sono risolte con più allenamento, ma mejorana/olivo resiste. Segno che quella è una difficoltà più strutturale (forse foglie verdi simili?) che le epoche da sole non curano.
Le cunfusioni mi sembrano ormai accettabili, si è saliti ad un'accuratezza del circa 92% e la valid_loss si è quasi fermata. Mi fermo qui.
Il validation set è di circa 39 foto, quindi error_rate finale (0.076 circa 3 foto sbagliate) è indicativo ma rumoroso; miglioramenti sotto questa soglia non sarebbero statisticamente affidabili.

### Prova #4
- Confusioni:
[('altro', 'alcalifa', np.int64(1)),
 ('altro', 'drago', np.int64(1)),
 ('altro', 'mejorana', np.int64(1)),
 ('olivo', 'palmera_cola_pescado_ramificada', np.int64(1)),
 ('palmera_canaria', 'drago', np.int64(1))]

 - La mia lettura:
 L'error rate si è alzato a circa 12%. Credo che questo non sia un vero problema in quanto le immagini "gemelle" avevano probabilmente alterato le prestazioni. Effettuerò un secondo test sul nuovo modello e trarrò le mie conclusioni su questa prova.
 8 epoche mi sembrano abbastanza, l'error rate si era fermato e la valid_loss si è quasi appiattita.

 ### Prova #5
- Confusioni:
[('alcalifa', 'altro', np.int64(3)),
 ('altro', 'alcalifa', np.int64(1)),
 ('olivo', 'drago', np.int64(1)),
 ('palmera_canaria', 'altro', np.int64(1)),
 ('palmera_canaria', 'drago', np.int64(1))]

 - La mia lettura:
 L'error rate è passato a circa 80 e le confusioni sembrano maggiori. A questo punto quello che farò sarà semplicemente testare il modello sul campo e analizzare i dati raccolti.

---

## Promemoria delle colonne (cosa scrivere in ognuna)

- **Architettura:** quale rete pre-allenata (es. resnet34, resnet18).
- **Epoche:** quanti giri completi (il numero passato a `fine_tune`).
- **Image size:** la dimensione del `Resize` (es. 224).
- **Augmentation:** sì/no e quale (es. `aug_transforms()`), oppure "nessuna".
- **Seed:** il numero fissato per rendere ripetibile la divisione train/validation.
- **valid_pct:** percentuale tenuta da parte per la validazione (es. 0.2).
- **error_rate / train_loss / valid_loss:** i valori dell'ULTIMA epoca.
- **Nota:** la ragione della prova e la conclusione. Senza questa, la riga è muta.
