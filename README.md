# World Happiness 2015-2019 — Analisi e strategia narrativa in Tableau

> Workbook interattivo che ricostruisce la dinamica della felicità globale su 5 anni e la traduce in una narrazione a quattro dashboard (dove, come, perché, cosa fare), usando LOD expressions, table calculations e parametri.

![Tableau](https://img.shields.io/badge/Tableau-Public-E97627?logo=tableau&logoColor=white)
![Dataset](https://img.shields.io/badge/Dataset-World%20Happiness%20Report-blue)
![Status](https://img.shields.io/badge/status-completed-success)

**Dashboard live su Tableau Public:** [World Happiness 2015-2019 — Analisi e Strategia](https://public.tableau.com/views/WorldHappinessReport2015-2019AnalisieStrategia/Dove)

<!-- TODO: inserire qui 3-4 screenshot delle dashboard (Dove? / Perché? / Com'è cambiato? / Strategia) -->

---

## Contesto

Il [World Happiness Report](https://worldhappiness.report/) misura ogni anno il benessere percepito in oltre 150 Paesi, combinando un punteggio di felicità auto-riportato con sei driver socio-economici (PIL pro capite, supporto sociale, aspettativa di vita sana, libertà, generosità, percezione della corruzione).

Questo progetto prende l'estratto 2015-2019 e lo tratta come avrebbe bisogno un analista a supporto di un'organizzazione internazionale o di una NGO: non "quanto è felice il mondo", ma **dove si concentrano i divari, come si muovono nel tempo e quali leve li spiegano** — per orientare dove conviene intervenire.

- **Periodo:** 2015-2019 (5 annualità)
- **Copertura:** 164 Paesi, 10 macro-regioni
- **Osservazioni:** 782 righe Paese-anno (ricomposte a partire dal sorgente 2015-2019)
- **Granularità:** Paese × anno × sei fattori

## Obiettivo

Strutturare i dati in un percorso narrativo che risponda in sequenza a quattro domande, in modo che chi guarda passi dal quadro generale alla raccomandazione:

1. **Dove?** — Come si distribuisce geograficamente la felicità.
2. **Com'è cambiato?** — Chi è salito e chi è sceso nel quinquennio.
3. **Perché?** — Quali fattori sono effettivamente correlati con il punteggio.
4. **Strategia** — Su quali leve conviene concentrare le risorse.

## Workflow e modellazione

I dati arrivano in formato Paese-anno-fattore. Il modello è volutamente semplice (tabella unica con pivot dei sei fattori in coppie nome/valore), così da poter calcolare correlazioni e profili fattoriali con un solo set di campi parametrici invece di sei misure separate.

Le quattro dashboard sono costruite su 11 worksheet:

| Dashboard | Worksheet che la compongono | Cosa risponde |
|---|---|---|
| **Dove?** | Mappa della felicità, Punteggio medio globale, Paese più/meno felice | Distribuzione geografica e benchmark globale |
| **Com'è cambiato?** | Heatmap Region/Year, Vincitori/Perdenti, Gap top-bottom | Dinamica temporale e mobilità dei Paesi |
| **Perché?** | Heatmap correlazioni, Felicità vs fattori, Radar fattori | Driver del punteggio |
| **Strategia** | Top e bottom 10, sintesi fattoriale | Sintesi orientata alla decisione |

## Scelte di design e metodologia

**Pivot dei fattori invece di sei misure.** Portare i sei driver in formato lungo (un campo "Nomi fattore", un campo "Valori fattore") permette di calcolare la correlazione con un'unica espressione e di guidare il radar con un parametro, anziché duplicare la logica sei volte. È la decisione che tiene insieme tutto il workbook.

**Correlazione calcolata in-viz, non importata.** I coefficienti tra ogni fattore e lo score sono ricalcolati direttamente in Tableau con LOD e table calculation, così restano coerenti con qualunque filtro applicato dall'utente, invece di essere numeri fissi calcolati altrove.

**LOD per i record annuali.** Leader e fanalino di coda di ogni anno sono estratti con espressioni FIXED, in modo da restare stabili indipendentemente dal livello di dettaglio della vista in cui vengono mostrati.

**Soglie esplicite per la classificazione.** Vincitori e perdenti non sono semplicemente ordinati: sono classificati con soglie dichiarate (variazione ≥ +0,5 o ≤ −0,5 punti tra 2015 e 2019), per separare il movimento reale dal rumore.

## Calcoli significativi

Leader globale per anno (LOD):

```
{ FIXED [Year] : MIN(IF [Rank] = 1 THEN [Country] END) }
```

Correlazione fattore-score, ricalcolata per anno e per fattore:

```
{ FIXED [Year], [Nomi fattore] : CORR([Score], [Valori fattore]) }
```

Variazione 2015→2019 per Paese, base della classifica vincitori/perdenti:

```
{ FIXED [Country] : MAX(IF [Year] = 2019 THEN [Score] END) }
-
{ FIXED [Country] : MAX(IF [Year] = 2015 THEN [Score] END) }
```

Radar pilotato da parametro (l'utente sceglie quale fattore isolare):

```
IF [Nomi fattore] = [Parametro fattore] THEN [Valori fattore] END
```

## Insight principali

| Tema | Risultato |
|---|---|
| **Trend globale** | Lo score medio resta stabile in una fascia stretta (5,35–5,41); minimo nel 2017, massimo nel 2019. |
| **Vertice** | La leadership annuale è sempre dell'Europa occidentale (Svizzera → Danimarca → Norvegia → Finlandia), ma ruota tra Paesi: cluster omogeneo, non monopolio. |
| **Polarizzazione 2019** | Tra Finlandia (7,769) e Sud Sudan (2,853) ci sono oltre 4,9 punti di divario. |
| **Driver** | Aspettativa di vita sana (0,738), PIL pro capite (0,712) e supporto sociale (0,624) sono i più correlati; generosità (0,078) e corruzione (0,188) quasi irrilevanti. |
| **Mobilità** | I movimenti più forti non sono in cima: Benin (+1,54), Costa d'Avorio (+1,29) salgono; Venezuela (−2,10) e Lesotho (−1,10) crollano. |

Il dato meno scontato è l'ultimo: la classifica si muove ai margini, non al vertice. I Paesi che cambiano davvero la propria posizione sono quelli a basso e medio reddito, dove uno shock economico o politico (Venezuela) o un miglioramento strutturale (Africa occidentale) sposta il punteggio molto più che nei Paesi già consolidati in testa.

## Conclusione orientata alla decisione

Se l'obiettivo fosse allocare risorse per migliorare il benessere percepito, i dati indicano che le leve **strutturali** (salute, reddito, reti di supporto sociale) spiegano il punteggio molto più di quelle civiche o comportamentali. Per un'organizzazione con budget limitato, questo significa che investire su sanità di base e tenuta del reddito ha un ritorno atteso sul benessere superiore rispetto a campagne su generosità o trasparenza — almeno in base alle correlazioni osservabili in questa finestra temporale. I Paesi su cui l'effetto marginale è plausibilmente più alto sono quelli in fascia medio-bassa e in movimento, non quelli già in coda da anni.

## Limiti e prossimi passi

- **Correlazione, non causalità.** I coefficienti mostrano associazione, non effetto. La conclusione strategica è un'ipotesi di priorità, non una prova.
- **Dataset noto e limitato.** Il World Happiness Report 2015-2019 è un dataset molto diffuso; il valore qui è nella costruzione analitica, non nella scoperta. Un'estensione naturale è agganciare gli anni 2020-2023 (shock pandemico) per testare la tenuta dei driver.
- **Nessun controllo per confondenti.** Reddito, salute e supporto sociale sono fortemente intercorrelati: un modello di regressione multipla (o un'analisi di importanza dei fattori) separerebbe i contributi meglio della correlazione semplice.
- **Prossimo passo tecnico:** portare la classificazione vincitori/perdenti su un breakdown per regione, per capire se i movimenti sono fenomeni nazionali o regionali.

## Stack tecnico

| Strumento | Uso |
|---|---|
| **Tableau Desktop / Public** | Modellazione, calcoli, dashboard |
| **LOD expressions (FIXED)** | Record annuali, gap, variazioni temporali |
| **Table calculations** | `CORR`, `WINDOW_CORR`, `RANK` |
| **Parametri** | Selezione interattiva del fattore nel radar |
| **Excel** | Sorgente dati (estratto 2015-2019) |

## File del repository

```
world-happiness-tableau/
├── README.md
├── world_happiness_2015_2019.twbx   # workbook Tableau (dati inclusi)
├── presentazione.pdf                # deck di sintesi dei risultati
└── screenshots/                     # (da aggiungere) immagini delle 4 dashboard
```

## Come aprirlo

1. **Online:** apri il [viz su Tableau Public](https://public.tableau.com/views/WorldHappinessReport2015-2019AnalisieStrategia/Dove) — nessuna installazione.
2. **In locale:** scarica `world_happiness_2015_2019.twbx` e aprilo con [Tableau Public](https://www.tableau.com/products/public/download) (gratuito) o Tableau Desktop. Il `.twbx` include già i dati.

## Cosa dimostra questo progetto

- Padronanza delle **LOD expressions** e delle table calculation di Tableau, non solo del drag-and-drop di base.
- Capacità di **modellare i dati per l'analisi** (pivot dei fattori) invece di adattarsi alla forma grezza.
- Costruzione di una **narrazione a dashboard** con un percorso logico (dove → come → perché → strategia), non una collezione di grafici scollegati.
- Calcolo di **correlazioni in-viz** che restano coerenti con i filtri dell'utente.
- Traduzione di un'analisi descrittiva in una **raccomandazione operativa**, con i limiti dichiarati onestamente.

---

*Progetto personale di data visualization sul dataset pubblico World Happiness Report (2015-2019). Realizzato in Tableau per esercitare modellazione dei dati, calcoli avanzati e data storytelling.*
