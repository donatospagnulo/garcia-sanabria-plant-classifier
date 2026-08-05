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

---


## vocabolario:

- Oggetti: elementi artificiali, umani, animali.
- Naturale: in circostanza di piante, alberi o fiori.
