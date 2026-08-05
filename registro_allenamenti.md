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
