# MARINA — nessun modello risolve tutto da solo. Ma decomposto, la flotta ci arriva.

> Un esperimento di **AI distribuita in locale** su una flotta di PC eterogenei.
> Modelli piccoli (0.5–3B) che da soli falliscono, un cervello grande (12B) che da solo
> a volte allucina — e un metodo, la **decomposizione su lavagna condivisa**, con cui
> insieme risolvono meglio di ciascuno preso singolarmente.
>
> _SudoWAI · Livorno · tutto self-hosted, zero cloud._

---

## Il punto di partenza (un fallimento onesto)

Problema, a parole:

> *20 banane pagate 4 € in tutto; 7 marciscono; le altre le vendo a 30 € l'una, ma 2 di
> esse col 20% di sconto. Qual è il guadagno netto?*

Risposta corretta: **374 €**.

Provato sui nostri nodi locali, a colpo singolo:

| chi | modello | risposta |
|---|---|---|
| nodo piccolo A | 3B | 306,60 € — sconto applicato a tutte, non a 2 |
| nodo piccolo B | 3B | *timeout* |
| **cervello grande** | **12B** | **ha allucinato: ha creduto che le banane fossero *persone* ("9 donne"), risposta priva di senso** |

Il dato scomodo: **anche il modello grande, da solo, su una domanda mal formulata è andato
a sbattere.** Riformulata in modo pulito la risolveva; ma "a naso", no. Nessun modello,
piccolo o grande, è affidabile a colpo singolo.

---

## Il metodo: decomporre, e far fare a ciascuno un pezzo

Invece di chiedere a un modello di risolvere **tutto**, il problema viene **spezzato in
passi atomici** — una sola operazione ciascuno — e distribuito. Ogni passo è banale, dentro
la portata anche di un modello minuscolo.

```
20 − 7        = 13     ✓
13 − 2        = 11     ✓
13 − 11       = 2      ✓
30 × 0.20     = 6      ✓
30 − 6        = 24     ✓
11 × 30       = 330    ✓
2 × 24        = 48     ✓
330 + 48      = 378    ✓
378 − 4       = 374    ✓   ← risultato
```

Nove passi, calcolati un pezzo per volta dai nodi piccoli che si alternano, con lo stato
tenuto su una **lavagna condivisa** (una memoria comune, in rete, che tutti leggono e
scrivono). Ogni passo passa da un **verificatore esterno** (l'aritmetica ricontrollata in
modo deterministico): se un nodo sbaglia un conto, si corregge e la catena non si rompe.

**Risultato: 374 €.** Gli stessi modelli che sul problema intero fallivano, decomposti lo
azzeccano — e battono anche il modello grande che da solo aveva allucinato.

---

## I ruoli (un'architettura "a lavagna")

- **Il decompositore** (modello grande, 12B) — non risolve: **spezza**. Il suo valore è
  capire *come* dividere il problema. È la cosa che i piccoli non sanno fare.
- **Gli esecutori** (modelli piccoli, 0.5–3B) — fanno **un pezzo ciascuno**. Un conto, una
  riga di codice: cose dentro la loro portata. Sempre accesi, costano quasi niente.
- **La lavagna condivisa** — la memoria di lavoro comune. Tiene lo stato che ai piccoli, da
  soli, mancherebbe. Persistente, accessibile in rete da ogni nodo.
- **Il verificatore esterno** — l'aritmetica ricontrollata, oppure il **codice eseguito**.
  Non è un'opinione di un modello: o il conto torna, o il codice gira. È la rete di
  sicurezza che rende il sistema affidabile pur essendo fatto di pezzi inaffidabili.

Un principio operativo: **niente tetti di tempo o di token. Conta la risposta giusta, non la
fretta.**

---

## Funziona anche sul codice

Stesso metodo su un compito di programmazione (una funzione con dei test da superare):

1. Il decompositore progetta la funzione come una **sequenza di righe**, ognuna una sola
   istruzione — descritte, non scritte.
2. Ogni nodo piccolo scrive **una riga**.
3. Si assembla e si **esegue** contro i test.
4. Se una riga è rotta (è successo: un nodo ha prodotto un ternario invalido), l'esecuzione
   la smaschera e il decompositore **ripara**.

La decomposizione sul codice è più fragile del calcolo — una riga sbagliata rompe la sintassi
dell'intero — ma la coppia **esecuzione + riparazione** regge, perché il verificatore è
esterno: il codice gira o non gira, non si discute.

---

## Cosa NON è (onestà sui limiti)

- **I modelli piccoli, da soli, non "pensano".** Sul problema intero producono rumore. Il
  loro valore non è risolvere: è fare un pezzo, e non allontanarsi dal problema mentre il
  grande a volte sbanda.
- **Non è magia emergente.** È ingegneria: decomposizione, memoria condivisa, verifica
  esterna. Ogni singolo pezzo resta inaffidabile; è il *sistema* a essere affidabile.
- **Serve un decompositore capace.** Se il piano di divisione è sbagliato, la catena
  produce con precisione la risposta sbagliata. Il modello grande resta necessario — per
  spezzare, non per risolvere.

---

## Perché in locale

Tutto gira su una flotta di PC di casa — nessuna chiamata a servizi esterni, nessun dato che
esce. La stessa logica di SudoWAI: **l'AI resta a casa tua.** Una flotta di macchine modeste,
di cui nessuna da sola basterebbe, che insieme fanno un lavoro serio.

Filoni collegati:
- [local-llm-benchmark](https://github.com/alessiom18/local-llm-benchmark) — "non esiste il
  modello migliore": 40 LLM locali su oltre 2000 compiti reali.
- [agent-scaffolding-benchmark](https://github.com/alessiom18/agent-scaffolding-benchmark) —
  l'impalcatura non è neutra: lo stesso modello dentro agenti diversi vince o crolla.

MARINA è il passo successivo: **non il modello migliore, non l'agente migliore — il
sistema.** Tanti pezzi deboli, un metodo che li fa lavorare insieme, e la verità tenuta da
un verificatore che non discute.

---

_SudoWAI — Livorno. Software e AI in locale, senza Big Tech._
