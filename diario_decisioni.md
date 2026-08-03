# Diario delle decisioni

> **A cosa serve questo file.** Qui scrivo, in prosa e con parole mie, le scelte di fondo del
> progetto e il *perché*. A differenza del registro allenamenti (una tabella di numeri), questo è
> discorsivo. Ogni voce che scrivo qui, in una tesi, diventa un paragrafo del capitolo metodologico.
> Regola: scrivo la decisione QUANDO la prendo, non a posteriori. Ricostruirla dopo è dieci volte più difficile.

---

## Come si scrive una voce

Ogni voce risponde a tre domande:
1. **Cosa** ho deciso.
2. **Perché** — il ragionamento, le alternative che ho scartato.
3. (se rilevante) **Con quali conseguenze** / cosa mi aspetto.

Metto la data così ricostruisco la cronologia del pensiero.

---

## Decisioni

### [03/08/2026] — Scelta dell'architettura
- Cosa: resnet34
- Perché:Il dataset è piccolo, un modello troppo grande su pochi dati rischia più overfitting.

### [03/08/2026] — Numero di categorie e classe "altro"
- Cosa: 7 categorie (6 note + una classe altro)
- Perché: si è preferito puntare alla qualità piuttosto che alla quantità, si è partiti da 11 piante candidate note, si è passati a 6 per varie motivazioni quali: piante troppo simili, condizioni impossibili per determinate fotografie, piante intrecciate con altre. La settima classe "altro", è stata inserita per poter dare al modello la conoscenza di piante simili, ma diverse, le foto inserite in questa categoria sono state scattate nelle stesse condizioni e modalità delle piante note.

### [03/08/2026] — Come ho verificato le etichette botaniche
- Cosa: Le etichette con i nomi delle specie sono presenti nel parco di Garcia Sanabria, Santa Cruz de Tenerife. Nel dubbio ho effettuato un check su iNaturalist per confermare che la pianta fotografata fosse effettivamente quella giusta.
- Perché: Le etichette presenti nel parco rappresentano una sicurezza.

### [03/08/2026] — Criterio per il numero di epoche
- Cosa: 8 epoche
- Perché:con 3 e 5 c'era evidente margine di miglioramento, con 8 si è arrivati a circa 92% di accuratezza e valid_loss si era quasi fermata del tutto.

### [03/08/2026] — Uso (o meno) della data augmentation
- Cosa: la data augmentation è stata utilizzata.
- Perché:si è sostenuto che il numero delle foto per specie non fosse abbastanza alto.

### [03/08/2026]
- Cosa: Ho solo 23 foto dell'alcalifa dopo la selezione, ho deciso di partire lo stesso
- Perché: L'obbiettivo è sperimentare, non fare raggiungere performance perfette, eventualmente integro dopo.

### [03/08/2026]
- Cosa: Ho deciso di modificare lo stesso il nome delle foto anche se le chiamo per cartelle
- Perché: Per pulizia generale

### [03/08/2026]
- Cosa: Ho deciso di inserire una cartella "altro" con piante generiche diverse dalle 6 principali
- Perché: Il modello imparerà a rifiutare immagini di piante diverse più facilmente.

### [03/08/2026]
Il dataset contiene gruppi di scatti simili dello stesso esemplare. La divisione train/validation di default li assegna casualmente, quindi alcuni quasi-gemelli possono trovarsi divisi tra i due insiemi: questo può aver gonfiato leggermente l'accuratezza rilevata. Miglioramento futuro: raggruppare per esemplare e dividere per gruppo.

### [03/08/2026]
Ho escluso foto con elementi di sfondo distintivi (tubi di irrigazione, piscine, insegne) per evitare che il modello associasse una specie a un elemento contestuale anziché alle caratteristiche della pianta stessa (rischio di 'scorciatoie' visive)."

---