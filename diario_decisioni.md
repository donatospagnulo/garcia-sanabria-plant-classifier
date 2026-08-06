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
