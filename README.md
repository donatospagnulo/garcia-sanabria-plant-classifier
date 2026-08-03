# garcia-sanabria-plant-classifier

Classificatore di immagini che riconosce 6 specie di piante del **Parque García Sanabria**
(Santa Cruz de Tenerife), più una classe "altro" per le piante fuori collezione.

Progetto didattico realizzato come prima esperienza pratica di deep learning applicato,
seguendo l'approccio del corso [fast.ai](https://course.fast.ai/) (Practical Deep Learning
for Coders). Costruito con [fastai](https://docs.fast.ai/) tramite transfer learning.

---

## Cosa fa

Data una foto di una pianta, il modello predice a quale delle 7 categorie appartiene:

| Categoria | Nome scientifico | Foto |
|-----------|------------------|------|
| Drago | *Dracaena draco* | 27 |
| Mejorana | *Origanum majorana* | 33 |
| Olivo | *Olea europaea* | 27 |
| Palmera canaria | *Phoenix canariensis* | 30 |
| Alcalifa | *Acalypha* | 23 |
| Palmera cola de pescado ramificada | *Caryota* | 27 |
| Altro | (piante fuori collezione) | 29 |

Totale: **196 immagini**, scattate personalmente nel parco.

---

## Metodo

- **Transfer learning** a partire da **ResNet34** pre-allenata.
- **Data augmentation** (`aug_transforms()`) per compensare il dataset ridotto.
- Immagini ridimensionate a **224×224**.
- Divisione train/validation: **80/20** (`valid_pct=0.2`, `seed=42`).
- Allenamento: **8 epoche** (`fine_tune`), scelto sperimentalmente.

## Risultato

Accuratezza sul validation set: **~92%** (error_rate ≈ 0.077).

Il numero è stato raggiunto in modo iterativo, aumentando le epoche (3 → 5 → 8) e
osservando che la `valid_loss` continuava a scendere. Il percorso completo degli
esperimenti è documentato in [`registro_allenamenti.md`](registro_allenamenti.md).

### Limiti noti (importante)

Questo è un progetto didattico con un dataset piccolo, e i risultati vanno letti con
onestà:

- **Validation set ridotto** (~39 foto): l'accuratezza è indicativa ma statisticamente
  rumorosa — poche foto in più o in meno spostano sensibilmente il numero.
- **Scatti simili tra train e validation**: il dataset contiene gruppi di foto (anche se piccoli) dello
  stesso esemplare. La divisione casuale può assegnarne alcuni al training e altri alla
  validation, il che può aver **leggermente gonfiato** l'accuratezza rilevata. Un
  miglioramento futuro sarebbe raggruppare per esemplare e dividere per gruppo.
- **Poche immagini per classe** (23–33): margine di miglioramento raccogliendo più dati,
  soprattutto per le classi più deboli.
- Le confusioni residue (es. `mejorana`/`olivo`) sono coerenti con la somiglianza visiva
  delle specie, non con errori di etichettatura.

---

## Struttura del repository

- `notebook.ipynb` — il notebook di allenamento (preparazione dati, training, valutazione).
- `registro_allenamenti.md` — tabella di tutti gli esperimenti con letture della matrice di confusione.
- `diario_decisioni.md` — le scelte metodologiche del progetto e il loro perché.
- `README.md` — questo file.

**Non** incluso nel repository:

- Le immagini del dataset → *(dataset non distribuito pubblicamente)*
- Il modello allenato (`.pkl`) → sarà pubblicato con la demo su Hugging Face.

---

## Nota sulla trasparenza

Gli script Python usati per rinominare in blocco i file del dataset sono stati sviluppati
con l'assistenza di un tutor AI. Sono stati usati comprendendone lo scopo, e per questo
motivo non sono inclusi nel repository. La preparazione concettuale del dataset (selezione,
verifica delle etichette, criteri di scarto) e l'analisi dei risultati sono farina del mio sacco.

---

## Licenza

Codice rilasciato sotto licenza **MIT** (vedi [`LICENSE`](LICENSE)).
Le immagini del dataset, quando pubblicate, avranno una licenza propria indicata nella loro pagina.
