# Decision journal

*English version below, [Italian original further down](#diario-delle-decisioni). Unfamiliar terms are defined in [`GLOSSARY.md`](GLOSSARY.md).*

> **What this file is for.** Here I write down, in prose and in my own words, the underlying
> choices of the project and *why* I made them. Unlike the training log (a table of numbers),
> this one is discursive. Every entry I write here would become a paragraph of the methodology
> chapter in a thesis.
> Rule: I write a decision WHEN I take it, not after the fact.

---

## How an entry is written

Every entry answers three questions:

1. **What** I decided.
2. **Why** — the reasoning, and the alternatives I rejected.
3. (where relevant) **With what consequences** / what I expect to happen.

I date each entry so I can reconstruct the chronology of my own thinking.

---

## Decisions

### [03/08/2026] — Choice of architecture

- **What:** resnet34.
- **Why:** the dataset is small. Too large a model on too little data risks more overfitting.

### [03/08/2026] — Number of categories, and the "altro" class

- **What:** 7 categories (6 known plants + one "other" class).
- **Why:** I chose quality over quantity. I started from 11 candidate labelled plants and cut
  down to 6 for various reasons: plants too similar to each other, impossible shooting
  conditions for some, and plants tangled up with others. The seventh class, `altro`, was added
  to give the model exposure to plants that are similar but different. The photos in this
  category were taken under the same conditions and in the same way as the known plants.

### [03/08/2026] — How I verified the botanical labels

- **What:** the species name plates are present in Parque García Sanabria, Santa Cruz de
  Tenerife. Where in doubt, I cross-checked on iNaturalist to confirm the photographed plant
  really was the right one.
- **Why:** the park's own plates are a reliable anchor.

### [03/08/2026] — Criterion for the number of epochs

- **What:** 8 epochs.
- **Why:** at 3 and 5 there was obvious room for improvement; at 8 accuracy reached roughly 92%
  and valid_loss had almost completely flattened.

### [03/08/2026] — Whether to use data augmentation

- **What:** data augmentation was used.
- **Why:** the number of photos per species wasn't high enough without it.

### [03/08/2026]

- **What:** I only have 23 photos of alcalifa after the selection, and I decided to start anyway.
- **Why:** the goal is to experiment, not to reach perfect performance. I can top it up later.

### [03/08/2026]

- **What:** I decided to rename the photo files anyway, even though labelling is done by folder.
- **Why:** general tidiness.

### [03/08/2026]

- **What:** I decided to add an "altro" folder with generic plants other than the 6 main ones.
- **Why:** the model will learn to reject images of different plants more easily.

### [03/08/2026]

The dataset contains groups of similar shots of the same individual plant. The default
train/validation split assigns them at random, so some near-duplicates can end up on opposite
sides: this may have slightly inflated the measured accuracy. Future improvement: group by
individual specimen and split by group.

### [03/08/2026]

I excluded photos with distinctive background elements (irrigation pipes, pools, signs) to
avoid the model associating a species with a contextual element rather than with the features
of the plant itself — the risk of visual "shortcuts".

### [04/08/2026]

The model detects objects (shoes, bottles, devices, …) as known species, almost always as
`drago`. I assume I need to expand the `altro` section with images of objects as well as plants.

### [05/08/2026]

**What:** I ran tests at the park. Objects are no longer detected as known species but as
`altro`, almost always. Time to write the tests down and analyse them properly.

- I repeated the tests indoors: the model still predicts known species (above all `drago`) when
  the objects are on the floor. When I placed the objects on a blanket with a floral pattern,
  the model returned `altro`.
- I repeated the test photographing objects on dark backgrounds: the model predicts known species.
- I repeated the test photographing objects on light backgrounds: the model predicts known species.
- One test remains: photographing objects outdoors but away from nature. My guess is the model
  will still predict known species. My idea is that it isn't the indoor setting or the artificial
  light that makes the model hallucinate, but the *absence of a natural context*.

**Why:** the model seems to hallucinate only when it looks like we're in a non-natural or
non-uniform environment. Possible reasons:

1. The `altro` section of the dataset lacks objects on non-natural backgrounds.
2. Checking the species it most often confuses things with (`drago`), I noticed that out of 28
   items, roughly half have a much more uniform background — the sky. It's very likely that
   this is what creates the confusion. The sky is also visible in some palmera canaria photos,
   in far smaller amounts.

### [05/08/2026]

**What:** I tested the model on the various known and unknown plants. I collected the data;
here is the analysis.

The test covered 94 photos:

**21 objects indoors:**

- 3 on a floral blanket → 3 NOT A MATCH
- 12 on a uniform or semi-uniform surface → 12 DRAGO and 1 PALMERA CANARIA → **important data point**
- 5 on a non-uniform surface (needs a better test) → 3 NOT A MATCH and 2 DRAGO

**73 at Parque García Sanabria:**

- 4 of objects → 4 NOT A MATCH
- 69 of plants:

**ALTRO (38)**
- NOT A MATCH — 31
- MEJORANA — 3 (one very similar, one moderately, one very different at 95%)
- ALCALIFA — 1 (very different, 56%)
- DRAGO — 1 (similar leaves)
- PALMERA COLA PESCADO — 1 (very similar colour)
- OLIVO — 1

**DRAGO (6)**
- DRAGO — 3 (branches + a few leaves + very little sky, 98%); (trunk + branches + leaves, a little sky, 74%); (with sky, 65%)
- NOT A MATCH — 2 (no sky); (branches + a few leaves + a bit of sky)
- OLIVO — 1 (trunk)
- *Incoming false positives:* from ALTRO (1); from PALMERA CANARIA (1, background); from uniform/semi-uniform surfaces (12); from non-uniform surfaces (2, to be redone properly)

**ALCALIFA (3)**
- ALCALIFA — 3
- *Incoming:* from ALTRO (1)

**PALMERA COLA PESCADO (10) — VERY BAD!!!**
- PALMERA COLA PESCADO — 2 (leaves, 97%); (trunk + fruit, 61%)
- NOT A MATCH — 5 (leaves); (leaves); (trunk + fruit); (trunk + leaves + fruit); (leaves + a bit of trunk)
- PALMERA CANARIA — 2 (trunk); (trunk + fruit + leaves)
- OLIVO — 1 (trunk + fruit)
- *Incoming:* from ALTRO (1)

**MEJORANA (2)**
- MEJORANA — 2
- *Incoming:* from ALTRO (3)

**PALMERA CANARIA (6)**
- PALMERA CANARIA — 2 (trunk + leaves, upper part); (upper part (trunk + leaves) + a bit of sky)
- NOT A MATCH — 3 (trunk); (trunk); (trunk)
- DRAGO — 1 (a lot of sky in the background)
- *Incoming:* from ALTRO (0); from uniform/semi-uniform surfaces (1); from PALMERA COLA PESCADO (2, probably the trunk)

**OLIVO (4)**
- OLIVO — 1 (full shot)
- NOT A MATCH — 3 (trunk); (leaves); (trunk + branches)
- *Incoming:* from ALTRO (1); from PALMERA COLA PESCADO (1, trunk + fruit); from DRAGO (1, trunk)

**CONCLUSIONS FROM TEST 1:**

- Cross-referencing the object photos with the plant photos, it seems fairly clear that the
  `drago` problem is the uniform background — that is, the sky. I need to reshoot those photos
  differently.
- The only photo I took of a whitish plant was detected as `palmera cola pescado`. I need to add
  at least a couple of whitish-coloured plants to the `altro` class.
- Mejorana needs more investigation given the 3 false positives from unknown plants; also, 2
  samples is too few.
- Alcalifa looks fine, but 3 samples is too few.
- Palmera cola pescado has obvious problems. The main one is the trunk, but the leaves are never
  recognised either. Needs investigation.
- Palmera canaria: here too the problem is the trunk.
- The olive tree seems to be recognised as a whole, but not in parts.
- **Trunks are a general problem.**

**IDEA:** narrow the use case to the full-plant shot. That way the model should work better.

**What I need to do now:**

1. Reshoot the drago against backgrounds that aren't sky.
2. Add the cases that slip through to `altro`.
3. Verify the "promising" classes with more samples.
4. Design the product so that it guides the user.

### [05/08/2026]

Drago and olivo have very few trunk photos (plenty of branches and leaves). Those need adding too.

### [05/08/2026]

The `altro` class has too many trunks and too little colour variety.

### [05/08/2026]

I ran a second test and it emerged that alcalifa isn't recognised as often as it seemed. I took
some more photos of it too.

### [06/08/2026]

The idea was to isolate at least the background variable, to confirm that the cause of the
hallucination was the sky in the drago photos. But time is short, so I'm putting everything
into the new dataset together. When I have time I'll isolate the problem and test it properly.

### [06/08/2026]

The following changed in the new dataset:

- **Removed:**
  - photos with a lot of sky, from drago and palmera canaria;
  - some trunks from the `altro` category;
  - some near-duplicate photos from the other species.
- **Added:**
  - drago photos without sky;
  - full-plant photos for the species that needed them (drago, alcalifa, olivo, palmera cola pescado);
  - semi-full photos for palmera canaria;
  - more colourful plants in the `altro` section.

The new dataset has 206 photos.

### [06/08/2026]

I ran the second test. Results below.

Test 2 covered 126 photos:

**10 in semi-open spaces, one plant and one object each:** me, an iPad, some keys, a small water
bottle, the view from my desk (PC + door), 2 photos of rocks, an unknown plant. All were
classified as NOT A MATCH. A third photo of the rocks was detected as `mejorana` at 55%.

**116 at Parque García Sanabria:**

**7 objects:** a painting, a bin, a pigeon, a manhole cover, 2 shoes — all classified as NOT A
MATCH. A third photo of a shoe was classified as `drago`.

**109 plants:**

**altro (38)**
- 21 correct
- 3 alcalifa (colour)
- 1 palmera cola pescado (very different)
- 3 mejorana (very similar)
- 3 olivo (similar trunk + sky) — note there are no sky photos in the olivo class
- 7 palmera canaria (4 trunks + sky, 3 completely different)

**alcalifa (12)**
- 8 correct
- 3 not a match
- 1 mejorana
- *(3 incoming from altro)*

**palmera canaria (14)**
- 9 correct
- 4 not a match: (dead leaves); (leaves only); (the classic shot it almost always recognised); (lower part of the trunk)
- 1 drago
- *(7 incoming from altro)*

**drago (10)**
- 6 correct (full shots)
- 3 not a match (shot from below, only branches and leaves)
- 1 alcalifa (trunk) ????
- *(1 incoming from objects)*

**mejorana (7)**
- 6 correct
- 1 not a match, too far away
- *(1 incoming from rocks)*

**olivo (8)**
- 4 correct (full shots)
- 4 not a match: (2 full plant); (2 leaves)

**cola pescado (22)**
- 5 correct: (3 leaves); (2 leaves + trunk)
- 1 palmera canaria (full shot, there was one behind it)
- 16 not a match: (6 leaves only); (1 trunk only); (1 fruit only); (3 leaves + trunk); (5 full shots)

**Analysing the data:**

- `drago` no longer acts as an attractor for objects — those go to `altro` almost always. The
  sky diagnosis was probably right! I'll verify it more rigorously later, by swapping only the
  drago category of v2 into the v1 model.
- `drago` is now recognised when full-plant photos are taken.
- `olivo` is now recognised when full-plant photos are taken.
- `palmera canaria` is recognised more easily.
- `mejorana` is often mistaken for a very similar small green species. I accept that.
- `alcalifa` is often confused with plants of its own colouring, and moreover isn't recognised
  when plants of a different colour are in the frame. This reinforces my hypothesis further.
  I accept that.
- Generic trunks are routed automatically to `olivo` or `palmera canaria`, because of the
  abundance of trunk photos in their folders and/or the removal of others from the `altro`
  category. My first diagnosis pointed at the presence of sky in some trunk shots, but 3 of the
  7 generic trunk photos the model predicted as palmera canaria have no sky; and 4 times the
  trunk + sky combination pointed to `olivo`, which has no photos with sky at all. I'm fairly
  confident that hypothesis is wrong, but I have to decide how to act (remove some trunks from
  the olivo and palmera canaria categories, put more back into `altro`, or both?).
- `palmera cola de pescado` isn't recognised as a whole plant. Its trunks are too thin and get
  confused with the surrounding environment — it will certainly have formed a texture together
  with the background that resembles the `altro` category much more, that category being far
  more varied. Close-ups of the leaves seem to work better.

### [06/08/2026]

I decided that `palmera cola de pescado ramificada` would not be present in the online version.
The reason is not wanting to complicate the user's experience of the game.

So I'll retrain the model with 6 categories instead of 7, changing nothing else — simply not
taking the palmera cola de pescado ramificada category into account.

I'll retest it above all on generic trunks. If the trunk problem disappears and no new important
ones emerge, I'll conclude my improvements here. Otherwise I'll judge, based on the new data and
the time left, whether it's worth modifying the dataset again.

### [06/08/2026]

I need to take photos that I'll use once I've left Tenerife, to run more accurate tests on my
intuitions — above all the one about the sky in the drago photos.

### [06/08/2026]

I tested the v3 model. Training without the `palmera cola de pescado ramificada` category, I
noticed two things:

1. Photos of generic trunks ended up as NOT A MATCH, no longer as olivo or palmera canaria.
2. Palmera canaria was recognised just 1 time out of 26 (all the rest went to NOT A MATCH),
   shooting the same kind of photos under the same conditions as with v2.

For this reason I decided to stop improving further, for now: I think the problem is spreading
too far. I may investigate it in future.

For the online version I'll go back to v2 with 7 categories, and add a button that lets the
player mark the palmera cola de pescado as found if it isn't recognised. The reason, again, is
to make the game more enjoyable and avoid making it stressful.

Thank you for your attention.

---
---

# 🇮🇹 Versione originale in italiano

# Diario delle decisioni

> **A cosa serve questo file.** Qui scrivo, in prosa e con parole mie, le scelte di fondo del
> progetto e il *perché*. A differenza del registro allenamenti (una tabella di numeri), questo è
> discorsivo. Ogni voce che scrivo qui, in una tesi, diventa un paragrafo del capitolo metodologico.
> Regola: scrivo la decisione QUANDO la prendo, non a posteriori.

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

### [04/08/2026]
Il modello rileva oggetti (scarpe, bottiglie, dispisitivi, ...) come specie note, quasi sempre come "drago", suppongo debba incrementare la sezione "altro" con immagini di oggetti oltre che di piante.

### [05/08/2026]
Cosa: Ho effettuato dei test al parco, gli oggetti non vengono più rilevati come specie note ma come "altro" quasi sempre. è il caso di appuntare e analizzare i test.
    Ho rieffettuato i test in spazi chiusi, il modello continua a predire specie note (soprattutto "drago") quando gli oggetti si trovano sul pavimento; ho riposto gli oggetti sulla coperta con disegni floreali, il modello rileva "altro".
    Ho rieffettuato il test fotografando oggetti su sfondi scuri, il modello predice specie note.
    Ho rieffettuato il test fotografando oggetti su sfondi chiari, il modello predice specie note.
    Rimane un ultimo test da fare: fotografare oggetti in spazi aperti non a contatto con la natura. Secondo me il modello predirrà specie note, la mia idea è che non è l'ambiente chiuso e la luce artificiale a far allucinare il modello ma il contesto naturale assente.

Perchè: Il modello sembra allucinare solo quando sembra che ci si trovi in un ambiente non naturale/non uniforme.
    Possibili motivazioni:
        1. Mancano oggetti su sfondo non naturale nella sezione "altro" del dataset.
        2. Controllando la specie con cui avviene più spesso la confusione (drago) ho notato che su 28 elementi, circa la metà hanno uno sfondo molto più uniforme (il cielo), è molto probabile che sia questo a creare confusione.
        Il cielo è visibile anche in alcune foto della palmera canaria (in quantità estremamente minore).

### [05/08/2026]
Cosa: Ho testato il modello sulle varie piante note e non note. Ho raccolto i dati, ora li analizzo:
    Il test è stato effettuato su 94 foto, di cui:
        - 21 oggetti in spazi chiusi, di cui
            - 3 su una coperta floreale -> 3 NOT A MATCH
            - 12 su una superficie uniforme o semi-uniforme -> 12 DRAGO and 1 PALMERA CANARIA -> dato importante.
            - 5 su una superficie non uniforme (da effettuare un test migliore) -> 3 NOT A MATCH and 2 DRAGO.

        - 73 al Parque Garcia Sanabria, di cui:
            - 4 a oggetti -> 4 NOT A MATCH
            - 69 a piante:
                - ALTRO (38)
                    NOT A MATCH - 31
                    MEJORANA - 3 (una molto simile, una media, una molto diversa con 95%)
                    ALCALIFA - 1 (molto diversa 56%)
                    DRAGO - 1 (foglie simili)
                    PALMERA COLA PESCADO - 1 (colore molto simile)
                    OLIVO - 1

                - DRAGO (6)
                    DRAGO - 3 (rami + qualche foglia + pochissimo il cielo con 98%) ; (tronco + rami + foglie - un poco di cielo con 74%) ; (con cielo 65%)
                    NOT A MATCH - 2 (no cielo) ; (rami + qualche foglia + un po' di cielo)
                    OLIVO - 1 (tronco)

                        DA ALTRO (1)
                        DA PALMERA CANARIA (1) (sfondo)
                        DA SUPERFICIE UNIFORME / SEMI-UNIFORME (12)
                        DA SUPERFICIE NON UNIFORME (DA RIFARE MEGLIO) (2)

                - ALCALIFA (3)
                    ALCALIFA - 3

                        DA ALTRO (1)

                - PALMERA COLA PESCADO (10) - MOLTO MALE!!!
                    PALMERA COLA PESCADO - 2 (foglie 97%) ; (tronco + frutti con 61%)
                    NOT A MATCH - 5 (foglie) ; (foglie) ; (tronco + frutti) ; (tronco + foglie + frutti) ; (foglie + un po' di tronco)
                    PALMERA CANARIA - 2 (tronco) ; (tronco + frutti + foglie)
                    OLIVO - 1 (tronco + frutti)

                        DA ALTRO (1)

                - MEJORANA (2)
                    MEJORANA - 2

                        DA ALTRO (3)

                - PALMERA CANARIA (6)
                    PALMERA CANARIA - 2 (tronco + foglie | parte alta) ; (parte alta (tronco + foglie) + un po' di cielo)
                    NOT A MATCH - 3 (tronco) ; (tronco) ; (tronco)
                    DRAGO - 1 (molto cielo nello sfondo)

                    DA ALTRO (0)
                    DA SUPERFICIE UNIFORME / SEMI-UNIFORME (1)
                    DA PALMERA COLA PESCADO (2) (probabilmente tronco)

                - OLIVO (4)
                    OLIVO - 1 (FULL)
                    NOT A MATCH - 3 (tronco) ; (foglie) ; (tronco + rami)

                    DA ALTRO (1)
                    DA PALMERA COLA PESCADO (1) (tronco + frutti)
                    DA DRAGO (1) (tronco)


    CONCLUSIONI PRIMO TEST:
        - incrociando i dati delle foto agli oggetti e le foto alle piante, mi sembra abbastanza evidente che il problema del drago sia lo sfondo uniforme, cioè il cielo.
            Devo rifare le foto in maniera differente
        - L'unica foto di una pianta biancastra che ho scattato è stata rilevata come "palmera cola pescado", devo inserire nella classe altro almeno un paio di piante di colore biancastro.
        - da indagare meglio sulla mejorana per i 3 accoppiamenti da piante non note, inoltre 2 campioni sono pochi.
        - l'alcalifa sembra tutto apposto ma 3 campioni sono pochi
        - palmera cola pescado ha evidenti problemi. Il principale è il tronco, ma anche le foglie non vengono mai riconosciute. Da indagare.
        - palmera canaria, anche qui il problema è il tronco.
        - l'olivo, preso d'insieme sembra essere riconosciuto, preso a pezzi no.
        - I tronchi sono un problema generale.

        IDEA: restringo il caso d'uso allo scatto d'insieme, in questo modo il modello dovrebbe funzionare meglio.

        Cosa devo fare ora:
            1. Rifotografa il drago con sfondi che non sono cielo.
            2. Aggiungi ad altro i casi che sfuggono.
            3. Verifica con più campioni le classi "promettenti"
            4. Progetta il prodotto perchè guidi l'utente.


### [05/08/2026]
Il drago e l'olivo hanno pochissime foto di tronchi (abbastanza di rami e foglie). Vanno implementati anche questi.

### [05/08/2026]
La classe altro ha troppi tronchi e poca varietà di colore.

### [05/08/2026]
Ho effettuato un secondo test ed è sorto che l'alcalifa non viene riconosciuta così spesso come sembrava, ho scattato un po' di foto anche per essa.

### [06/08/2026]
L'idea era di isolare almeno l'idea dello sfondo per confermare che la causa dell'allucinazione fosse il cielo nelle foto del drago, ma il tempo stringe, quindi metto tutto insieme nel nuovo dataset.
Quando avrò tempo isolerò il problema e lo testerò seriamente.

### [06/08/2026]
Nel nuovo dataset sono cambiate le seguenti cose:
    - Rimosso:
        - Foto con tanto cielo rimosse dal drago e dalla palmera canaria
        - Alcuni tronchi dalla categoria altro
        - Alcune foto clone dalle altre specie

    - Aggiunto:
        - Foto del drago senza cielo
        - Foto di specie per intero dove servivano (drago, alcalifa, olivo, palmera cola pescado)
        - Foto di specie semi-intere per la palmera canaria
        - Foto di piante più colorate nella sezione altro

    Il nuovo dataset ha 206 foto.

### [06/08/2026]
Ho effettuato il secondo test, di seguito i risultati:

Il test 2 è stato effettuato su 126 foto, di cui:

    - 10 in spazi semi-aperti di cui una pianta e un oggetto:
    me, un ipad, delle chiavi, una bottiglietta d'acqua, la vista dalla mia postazione (pc + porta), 2 foto di rocce, una pianta non nota. Sono stati tutti classificato come "NOT A MATCH".
    Una terza foto delle rocce è stata rilevata come "mejorana" al 55%.

    - 116 foto al parque garcia sanabria, di cui:
        - 7 oggetti: 
            - un dipinto, un bidone dell'immondizia, un piccione, un tombino, 2 scarpe. Sono stati tutti classificati come "NOT A MATCH".
            - una terza foto di una scarpa è stata classificata come "drago".

        - 109 piante:

            altro (38)
                - 21 corrette
                - 3 alcalifa (colore)
                - 1 palmera cola pescado (molto diversa)
                - 3 mejorana (molto simili)
                - 3 olivo (tronco simile + cielo) (in olivo non sono presenti foto del cielo)
                - 7 palmera canaria (4 tronchi + cielo, 3 completamente diversi)

            alcalifa (12)
                - 8 corrette
                - 3 not a match
                - 1 mejorana

                - (3 da altro)

            palmera canaria (14)
                - 9 corrette
                - 4 not a match (foglie morte); (solo foglie); (foto classica che ha riconosciuto quasi sempre); (tronco parte bassa)
                - 1 drago

                - (7 da altro)

            drago (10)
                - 6 corrette (foto full)
                - 3 not a match (foto da sotto verso solo rami e foglie)
                - 1 alcalifa (tronco) ????

                - (1 da oggetti)
                
            mejorana (7)
                - 6 corrette
                - 1 not a match, troppo lontano

                - (1 da rocce)

            olivo (8)
                - 4 corrette (foto full)
                - 4 not a match (2 per intero); (2 foglie)

            cola pescado (22)
                - 5 corrette (3 foglie); (2 foglie + tronco)
                - 1 palmera canaria (foto full, era presente dietro)
                - 16 not a match (6 solo foglie) ; (1 solo tronco); (1 solo frutti); (3 foglie + tronco); (5 foto full)
                 
Analizzando i dati:

    - Il "drago" non fa più da attrattore per gli oggetti, questi vanno in altro quasi sempre. La diagnosi del cielo era probabilmente giusta! Verificherò con più sicurezza in seguito, sostituiendo al modello v1 solamente la categoria drago del v2.
    - Il "drago" ora viene riconosciuto se vengono fatte foto per intero.
    - L' "olivo" ora viene riconosciuto se vengono fatte foto per intero.
    - La "palmera canaria" viene riconosciuta con più facilità.

    - La "mejorana" viene scambiata spesso con specie verde e piccola molto simile. Lo accetto.
    - L' "alcalifa" viene spesso confusa con piante della sua stessa colorazione, inoltre non viene riconosciuta se ci sono piante di colorazione differente nell'inquadratura, questo alimenta ancora di più la mia idea. Lo accetto.
    - Tronchi generici vengono mandati automaticamente in olivo o palmera canaria per l' abbondanza di foto di tronchi nelle loro cartelle e/o l'eliminazione di altri dalla categoria altro. Una prima diagnosi mi faceva pensare alla presenza del cielo in alcuni scatti dei tronchi, ma 3 su 7 delle foto di tronchi generici che il modello ha predetto come palmera canaria non hanno il cielo; inoltre 4 volte, la combo tronco + cielo ha rimandato a olivo, che non ha foto con cielo. Sono abbastanza sicuro che quest'ipotesi non sia giusta, ma devo decidere come agire (elimino un po' di tronchi dalle categorie olivo e palmera canaria, li implemento nuovamente in altro, o entrambi?).
    - la palmera cola de pescado non viene riconosciuta per intero, i tronchi sono troppo sottili e vengono confusi con l'ambiente circostante, si sarà sicuramente creata una texture con lo sfondo che somiglia molto di più alla categoria altro essendo molto più varia.
    I close up sulle foglie sembrano funzionare meglio. Ho deciso che per la versione online, questa pianta non sarà presente per non rendere l'esperienza dell'utente inutilmente difficile e noiosa.
    
### [06/08/2026]
Ho deciso che nella versione utilizzabile online la "palmera cola de pescado ramificada" non sarà presente. Questa scelta è dovuta alla volontà di non complicare l'esperienza dell'utente nel gioco.
Per tanto allenerò nuovamente il modello con 6 categorie al posto di 7, non cambierò nulla se non che la categoria della palmera cola de pescado ramificada non verrà presa in considerazione.
Lo ritesterò soprattutto sui tronchi generici, se il problema dei tronchi scompare e non ne escono di nuovi, importanti, concluderò qui i miei miglioramenti.
In caso contrario valuterò in base ai nuovi dati e al tempo rimasto se converrebbe modificare il dataset nuovamente il dataset oppure no.

### [06/08/2026]
Devo scattare delle foto che utilizzerò una volta andato via da Tenerife per effettuare test più accurati sulle mie intuizioni, soprattutto su quella del cielo nelle foto del drago.

### [06/08/2026]
Ho testato il modello v3, allenando il modello senza la categoria "palmera cola de pescado ramificada" mi sono accorto di due cose:
    1. Le fotografie di tronchi generici finivano su NOT A MATCH e non più su olivo o su palmera canaria
    2. La palmera canaria è stata riconosciuta ben 1 volta su 26 (andavano tutte in NOT A MATCH) scattando foto dello stesso genere e nelle stesse condizioni della v2.

Ho deciso per questo motivo di smettere di migliorare ulteriormente, per ora, in quanto credo che il problema si stia allargando troppo. Magari ci indagherò in futuro.

Per la versione online tornerò alla v2 con 7 categorie, inserirò un pulsante per dare la palmera cola de pescado per buona qualora non venga riconosciuta.
La motivazione è sempre per rendere l'esperienza di gioco più divertente, evitando di renderla stressante.

Ti ringrazio per l'attenzione

---

## vocabolario:

- Oggetti: elementi artificiali, umani, animali.
- Naturale: in circostanza di piante, alberi o fiori.
