# BARAK 2024 — Benchmark Ufficiale di Tesi per DAINO

> **Nota in italiano (intro per Paolo).** Questo è il documento di riferimento ufficiale per la tesi: contiene la formulazione matematica completa del problema DRCFJSP-SDST di Barak et al. 2024, l'istanza piccola (4×3×2) come fixture concreta, le soluzioni di riferimento per validare DAINO, le istruzioni per le istanze più grandi (Appendice C) e i parametri estrapolati per i dati non pubblicati. Il corpo è in inglese per non duplicare contenuto. Questo file è autocontenuto: non servono né i tre file sorgente (`/tmp/barak_round1_*.md`) né il PDF.

---

## Sezione 1 — Identificazione e ambito

**Citazione del paper.** Barak, S., Javanmard, S., & Moghdani, R. (2024). "Dual resource constrained flexible job shop scheduling with sequence-dependent setup time." *Expert Systems* **41**(10), e13669. DOI: [10.1111/exsy.13669](https://doi.org/10.1111/exsy.13669).

**Nome del problema.** DRCFJSP-SDST — Dual Resource Constrained Flexible Job Shop Scheduling with Sequence-Dependent Setup Times.

**Descrizione in linguaggio naturale.** Un insieme di job, ciascuno costituito da una sequenza ordinata di operazioni, deve essere schedulato su un insieme di macchine assegnando contemporaneamente un operatore umano a ogni operazione. Sia la macchina *sia* l'operatore sono richiesti perché un'operazione possa essere eseguita (vincolo "dual resource"), e il tempo di processamento dipende dalla coppia (macchina, operatore) scelta. Quando due operazioni consecutive sulla stessa macchina provengono da job diversi, è richiesto un tempo di setup dipendente dalla sequenza (SDST) per riconfigurare la macchina. Tre obiettivi possono essere minimizzati indipendentemente: (Q₁) le penalità totali pesate di anticipo + ritardo rispetto alle finestre di consegna [Dl, Du] per ogni job, (Q₂) il costo totale di idle dell'operatore più il costo SDST cumulato, e (Q₃) il massimo carico di lavoro tra le macchine (load-balancing).

**Stato.** Questo documento è il **benchmark ufficiale di tesi** per DAINO. Il PDF originale si trova accanto a questo file (stessa cartella Desktop).

**Indice.**

1. Identificazione e ambito
2. Modello matematico
3. Istanza concreta di benchmark (esempio piccolo 4×3×2)
4. Soluzioni di riferimento per la validazione
5. Istanze di benchmark più grandi (Appendice C)
6. Parametri estrapolati / inferiti
7. Checklist di consumo per DAINO
8. Domande aperte per il relatore di tesi

---

## Sezione 2 — Modello matematico

### 2.1 Indici e insiemi

| Simbolo | Dominio | Descrizione |
|---|---|---|
| $t$ | $t = 1, 2, \dots, T$ | Indice del tempo |
| $h$ | $h = 1, 2, \dots, H$ | Indice degli operatori |
| $p, r$ | $p, r = 1, 2, \dots, P$ | Indice delle operazioni (all'interno di un job) |
| $m$ | $m = 1, 2, \dots, M$ | Indice delle macchine |
| $i, j$ | $i = 1, \dots, I;\ j = 1, \dots, J$ | Indice dei job |
| $O_{ip}$ | insieme | Insieme delle operazioni del job $i$ |
| $M_{ip}$ | insieme | Insieme delle macchine candidate per l'operazione $O_{ip}$ |
| $H_m$ | insieme | Insieme degli operatori candidati per la macchina $m$ |

### 2.2 Parametri

| Simbolo | Descrizione |
|---|---|
| $P^{hm}_{ip}$ | Tempo di processamento dell'operazione $O_{ip}$ sulla macchina $m$ con l'operatore $h$ |
| $S^{m}_{ipjr}$ | Tempo di setup sulla macchina $m$ quando l'operazione $p$ del job $i$ è processata immediatamente prima dell'operazione $r$ del job $j$ sulla stessa macchina |
| $STC^{m}_{ipjr}$ | Costo di setup per unità di tempo per la stessa transizione (operazione $O_{ip}$ prima di $O_{jr}$ sulla macchina $m$) |
| $\text{Cost}_h$ | Costo idle dell'operatore $h$ per unità di tempo |
| $[Dl_i,\ Du_i]$ | Finestra di due date per il job $i$ (limite inferiore / superiore) |
| $\beta_i$ | Penalità di anticipo per il job $i$ per unità di tempo |
| $u_i$ | Penalità di ritardo per il job $i$ per unità di tempo |
| $L$ | Costante positiva sufficientemente grande (big-$M$) |

### 2.3 Variabili decisionali

#### 2.3.1 Continue (positive)

Tutte le variabili continue qui definite sono vincolate a essere non negative ($\geq 0$); ciò è implicito nella formulazione del paper "positive decision variables" ed è necessario affinché le equazioni (23) e (24) agiscano come definizioni di $\max(0, \cdot)$.

| Simbolo | Descrizione |
|---|---|
| $C_{\max}$ | Tempo massimo di completamento tra tutti i job (makespan) |
| $WL_{\max}$ | Massimo carico di lavoro tra tutte le macchine |
| $WL_m$ | Carico di lavoro totale della macchina $m$ |
| $C_i$ | Tempo di completamento del job $i$ |
| $C_{ip}$ | Tempo di completamento dell'operazione $O_{ip}$ |
| $S_{ip}$ | Tempo di inizio dell'operazione $O_{ip}$ |
| $E_i$ | Anticipo (earliness) del job $i$ |
| $T_i$ | Ritardo (tardiness) del job $i$ |

#### 2.3.2 Binarie

| Simbolo | Descrizione |
|---|---|
| $W^{hm}_{ip}$ | $=1$ se l'operatore $h$ e la macchina $m$ sono selezionati per eseguire l'operazione $O_{ip}$; $0$ altrimenti |
| $Y^{hmt}_{ip}$ | $=1$ se l'operazione $O_{ip}$ inizia al tempo $t$ usando l'operatore $h$ e la macchina $m$; $0$ altrimenti |
| $X^{m}_{ipjr}$ | $=1$ se l'operazione $O_{ip}$ è processata prima dell'operazione $O_{jr}$ sulla macchina $m$; $0$ altrimenti |
| $Z_m$ | $=1$ se $X^{m}_{ipjr}$ oppure $X^{m}_{jrip}$ vale $1$ (variabile ausiliaria che collega $X$ e $W$); $0$ altrimenti |

### 2.4 Funzioni obiettivo

> La numerazione delle equazioni segue il paper pubblicato, dove le equazioni (1)–(5) fanno parte della rassegna della letteratura e non della formulazione del modello; il modello vero e proprio inizia dalla (6).

#### Q₁ (eq 6) — minimizza le penalità di anticipo/ritardo

$$
Q_1  =  \min \left( \sum_{i=1}^{I} \beta_i \cdot E_i  +  \sum_{i=1}^{I} u_i \cdot T_i \right)
$$

**In linguaggio naturale.** Minimizza le penalità totali pesate di anticipo e ritardo su tutti i job, usando i tassi di penalità per ciascun job $\beta_i$ (anticipo) e $u_i$ (ritardo).

#### Q₂ (eq 7) — minimizza il costo totale (costo idle dell'operatore + costo SDST)

$$
Q_2 = \min \left\lbrace \sum_{h=1}^{H} \text{Cost}_h \left( C_{\max} - \sum_{i,p,m} W^{hm}_{ip} P^{hm}_{ip} \right) + \sum_{m, i,j, p,r} S^{m}_{ipjr} \cdot STC^{m}_{ipjr} \cdot X^{m}_{ipjr} \right\rbrace
$$

Dove la prima sommatoria è su tutte le coppie $(i,p)$ con $p \in O_{ip}$ e tutte le macchine $m \in M_{ip}$; la seconda è su tutte le macchine $m$ e tutte le coppie ordinate di operazioni $(O_{ip}, O_{jr})$ con $i \neq j$ o $p \neq r$.

**In linguaggio naturale.** Minimizza il costo totale. Il primo termine è il costo idle degli operatori: per ciascun operatore $h$, il tempo non speso a processare operazioni (cioè $C_{\max}$ meno la somma dei tempi di processamento effettivamente assegnati a $h$) moltiplicato per il costo unitario dell'operatore. Il secondo termine è il costo cumulativo di setup, sostenuto ogni volta che operazioni consecutive $O_{ip} \to O_{jr}$ sono sequenziate sulla stessa macchina.

#### Q₃ (eq 8) — minimizza il massimo carico di lavoro delle macchine

$$
Q_3 = \min \max_{m \in M} WL_m
$$

**In linguaggio naturale.** Minimizza il massimo carico di lavoro tra le macchine, bilanciando il carico tra le macchine.

### 2.5 Vincoli

#### Eq (9) — Definizione del workload

$$
WL_m  =  \sum_{h \in H_m} \sum_{i=1}^{I} \sum_{p \in O_{ip}} W^{hm}_{ip} \cdot P^{hm}_{ip}
$$

Quantificatori: $\forall m \in M_{ip}$.

Il carico di lavoro della macchina $m$ è uguale alla somma dei tempi di processamento di tutte le operazioni assegnate a $m$ (su ogni operatore eleggibile $h \in H_m$).

#### Eq (10) — Massimo workload

$$
WL_{\max}  \geq  WL_m
$$

Quantificatori: $\forall m \in M$.

$WL_{\max}$ è grande almeno quanto il carico di lavoro di ogni macchina, quindi all'ottimo è uguale al massimo carico di lavoro tra le macchine.

#### Eq (11) — Unicità dell'assegnazione delle operazioni

$$
\sum_{h \in H_m} \sum_{m \in M_{ip}} W^{hm}_{ip}  =  1
$$

Quantificatori: $\forall i ,  p \in O_{ip}$.

Ogni operazione $O_{ip}$ deve essere assegnata esattamente a una coppia (operatore, macchina) ammissibile.

#### Eq (12) — Collegamento $W$ e $Y$ (inizia esattamente una volta)

$$
\sum_{t=1}^{T} Y^{hmt}_{ip}  =  W^{hm}_{ip}
$$

Quantificatori: $\forall i ,  p \in O_{ip}; m \in M_{ip}; h \in H_m$.

Se l'operazione $O_{ip}$ è assegnata alla coppia $(h,m)$ (cioè $W^{hm}_{ip}=1$), allora deve iniziare esattamente in un istante temporale $t$; altrimenti non viene mai avviata da quella coppia.

#### Eq (13) — Definizione del tempo di inizio

$$
S_{ip}  =  \sum_{h \in H_m} \sum_{m \in M_{ip}} \sum_{t=1}^{T} Y^{hmt}_{ip} \cdot t
$$

Quantificatori: $\forall i ,  p \in O_{ip}$.

Il tempo di inizio dell'operazione $O_{ip}$ è l'indice temporale $t$ in cui la sua variabile $Y$ vale $1$ (zero altrove, quindi la somma recupera l'unico tempo di inizio scelto).

#### Eq (14) — Definizione del tempo di completamento

$$
C_{ip}  \geq  S_{ip}  +  \sum_{h \in H_m} \sum_{m \in M_{ip}} W^{hm}_{ip} \cdot P^{hm}_{ip}
$$

Quantificatori: $\forall i ,  p \in O_{ip}$.

Il tempo di completamento dell'operazione $O_{ip}$ è almeno pari al suo tempo di inizio più il tempo di processamento (sulla coppia $(h,m)$ scelta).

#### Eq (15) — Precedenza tra operazioni all'interno di un job

$$
S_{i(p+1)}  \geq  C_{ip}
$$

Quantificatori: $\forall i ,  p \in O_{ip}$.

All'interno di un job, l'operazione $p+1$ non può iniziare prima che l'operazione $p$ sia stata completata (le operazioni vengono processate nell'ordine dato, senza preemption).

#### Eq (16) — Tempo di completamento del job

$$
C_i  \geq  C_{ip}
$$

Quantificatori: $\forall i ,  p \in O_{ip}$.

Il tempo di completamento del job $i$ è almeno pari al tempo di completamento di ciascuna delle sue operazioni (quindi è uguale a quello dell'ultima).

#### Eq (17) — Sequenziamento disgiuntivo (caso $X^{m}_{jrip}=1$, con SDST)

$$
\begin{aligned}
S_{ip} \geq{} & C_{jr} + \sum_{m \in M_{ip} \cap M_{jr}} X^{m}_{jrip} \cdot S^{m}_{jrip} \\
& - L \left( \sum_{m \in M_{ip} \cap M_{jr}} X^{m}_{ipjr} \right) \\
& - L \left( 2 - \sum_{h,m \in M_{ip}} W^{hm}_{ip} - \sum_{h,m \in M_{jr}} W^{hm}_{jr} \right)
\end{aligned}
$$

Quantificatori: $\forall i, p \in O_{ip}$; $\forall j, r \in O_{jr}$. Le sommatorie $\sum_{h,m \in M_{ip}}$ stanno per $\sum_{m \in M_{ip}} \sum_{h \in H_m}$.

Se entrambe le operazioni $O_{ip}$ e $O_{jr}$ sono assegnate a una macchina comune e $O_{jr}$ precede $O_{ip}$ su quella macchina ($X^{m}_{jrip}=1$), allora l'inizio di $O_{ip}$ deve essere almeno pari al completamento di $O_{jr}$ più il tempo di setup dipendente dalla sequenza $S^{m}_{jrip}$. I termini big-$L$ rilassano il vincolo quando la precondizione non è verificata (macchine diverse o ordinamento opposto).

#### Eq (18) — Sequenziamento disgiuntivo (caso $X^{m}_{ipjr}=1$, con SDST)

$$
\begin{aligned}
S_{jr} \geq{} & C_{ip} + \sum_{m \in M_{ip} \cap M_{jr}} X^{m}_{ipjr} \cdot S^{m}_{ipjr} \\
& - L \left( 1 - \sum_{m \in M_{ip} \cap M_{jr}} X^{m}_{ipjr} \right) \\
& - L \left( 2 - \sum_{h,m \in M_{ip}} W^{hm}_{ip} - \sum_{h,m \in M_{jr}} W^{hm}_{jr} \right)
\end{aligned}
$$

Quantificatori: $\forall i, p \in O_{ip}$; $\forall j, r \in O_{jr}$.

Controparte simmetrica della (17): se entrambe le operazioni sono assegnate a una macchina comune e $O_{ip}$ precede $O_{jr}$ ($X^{m}_{ipjr}=1$), allora $O_{jr}$ non può iniziare finché $O_{ip}$ non è stata completata più l'SDST $S^{m}_{ipjr}$. I termini big-$L$ disattivano il vincolo quando le condizioni di assegnazione o ordinamento non si applicano.

#### Eq (19) — Attivazione di $Z_m$ quando entrambe le operazioni sono sulla macchina $m$ (limite inferiore)

$$
\sum_{h \in H_m} W^{hm}_{ip}  +  \sum_{h \in H_m} W^{hm}_{jr}  \geq  2 - L (1 - Z_m)
$$

Quantificatori: $\forall m \in M_{ip} \cap M_{jr}; \forall i,p \in O_{ip}; \forall j,r \in O_{jr}$.

Se $Z_m = 1$ (le due operazioni condividono la macchina $m$), la somma degli indicatori di assegnazione sulla macchina $m$ deve raggiungere $2$ (entrambe le operazioni effettivamente allocate a $m$). Quando $Z_m = 0$ lo slack big-$L$ rende il vincolo banalmente soddisfatto.

#### Eq (20) — Attivazione di $Z_m$ quando entrambe le operazioni sono sulla macchina $m$ (limite superiore)

$$
\sum_{h \in H_m} W^{hm}_{ip}  +  \sum_{h \in H_m} W^{hm}_{jr}  \leq  2 + L \cdot Z_m
$$

Quantificatori: $\forall m \in M_{ip} \cap M_{jr}; \forall i,p \in O_{ip}; \forall j,r \in O_{jr}$.

Limite superiore complementare alla (19); insieme alla (19), fissano $Z_m = 1$ esattamente quando entrambe le operazioni sono processate sulla macchina $m$ (e $Z_m = 0$ altrimenti).

#### Eq (21) — Forza esattamente un ordinamento quando $Z_m = 1$

$$
1 - L (1 - Z_m)  \leq  X^{m}_{ipjr} + X^{m}_{jrip}  \leq  1 + L (1 - Z_m)
$$

Quantificatori: $\forall m \in M_{ip} \cap M_{jr}; \forall i,p \in O_{ip}; \forall j,r \in O_{jr}$.

Quando le due operazioni condividono la macchina $m$ ($Z_m=1$), esattamente una delle due variabili di ordinamento $X^{m}_{ipjr}, X^{m}_{jrip}$ vale $1$ (l'altra è $0$): un'operazione deve precedere l'altra.

#### Eq (22) — Disabilita gli ordinamenti quando le operazioni non sono co-localizzate

$$
-L \cdot Z_m  \leq  X^{m}_{ipjr} + X^{m}_{jrip}  \leq  L \cdot Z_m
$$

Quantificatori: $\forall m \in M_{ip} \cap M_{jr}; \forall i,p \in O_{ip}; \forall j,r \in O_{jr}$.

Se le due operazioni *non* sono entrambe sulla macchina $m$ ($Z_m=0$), allora sia $X^{m}_{ipjr}$ sia $X^{m}_{jrip}$ devono essere $0$; il sequenziamento è privo di significato. Quando $Z_m=1$ lo slack big-$L$ rende la (21) il vincolo attivo.

#### Eq (23) — Definizione di anticipo (earliness)

$$
E_i  \geq  Dl_i - C_i
$$

Quantificatori: $\forall i$.

L'anticipo del job $i$ è almeno $Dl_i - C_i$; combinato con $E_i \geq 0$, ciò dà $E_i = \max(0, Dl_i - C_i)$, ossia di quanto il job termina prima della sua data più anticipata accettabile.

#### Eq (24) — Definizione di ritardo (tardiness)

$$
T_i  \geq  C_i - Du_i
$$

Quantificatori: $\forall i$.

Il ritardo del job $i$ è almeno $C_i - Du_i$; con $T_i \geq 0$, ciò dà $T_i = \max(0, C_i - Du_i)$. I job che terminano dentro la finestra $[Dl_i, Du_i]$ non incorrono né in anticipo né in ritardo.

#### Eq (25) — Definizione del makespan

$$
C_{\max}  \geq  C_i
$$

Quantificatori: $\forall i$.

Il makespan $C_{\max}$ è almeno pari al tempo di completamento di ogni job; all'ottimo, è uguale al tempo di completamento del job più tardivo.

### 2.6 Note di modellazione (ambiguità / chiarimenti dal paper)

I punti 1–3 e 5–7 segnalano ambiguità reali presenti nel paper; i punti 4 e 8 sono note di chiarimento per il lettore della tesi.

1. **Pedice di $Y$ nelle eq (12)–(13).** Il rendering del PDF delle equazioni (12) e (13) mostra qualcosa che assomiglia a $Y^{hmt}_{jr}$, ma è incoerente con la definizione della variabile in §2.3.2 ($Y^{hmt}_{ip}$) e con l'indicizzazione del vincolo circostante (che è su $i, p$, non $j, r$). Anche la versione estratta dal testo riporta "jr". Abbiamo scritto $Y^{hmt}_{ip}$ in entrambe le equazioni perché (a) è quanto richiesto dalla definizione stessa della variabile, (b) il vincolo è quantificato $\forall i, p$, e (c) i membri di destra ($W^{hm}_{ip}$ in (12); $S_{ip}$ definita in (13)) sono anch'essi indicizzati su $(i,p)$. Sembra trattarsi di un errore di composizione nel paper pubblicato.

2. **Quantificatori in (17) e (18).** Il paper scrive il quantificatore come $\forall i, p \in O_{ip};\ \forall j, r \in O_{jr};\ \forall t$, ma il vincolo non contiene alcun indice temporale $t$. Il $\forall t$ spurio è stato rimosso per chiarezza; trattiamo il vincolo come quantificato su coppie distinte di operazioni $(O_{ip}, O_{jr})$ che condividono almeno una macchina ammissibile.

3. **Definizione di $X^{m}_{ipjr}$.** §2.3.2 dice "$X^{m}_{ipjr} = 1$ if $O_{ip}$ is processed before $O_{jr}$ on machine $m$". Tipicamente significa *immediatamente* prima nell'interpretazione disgiuntiva flow-shop, dato che l'SDST $S^{m}_{ipjr}$ in (17)–(18) viene aggiunto come singolo setup. Il paper non afferma esplicitamente "immediatamente", ma la formulazione SDST standard lo richiede affinché il costo in $Q_2$ sia additivo sulle coppie consecutive.

4. **Ambito degli indici $r$ vs. $p$.** Sia $p$ sia $r$ vanno da $1$ a $P$ ma rappresentano indici di operazione che possono appartenere a job diversi ($i$ vs. $j$). La notazione è coerente ma leggermente compatta: $p$ è abbinato al job $i$, $r$ è abbinato al job $j$.

5. **Quantificatore di $WL_m$.** L'eq (9) è scritta "$\forall m \in M_{ip}$" ma $WL_m$ è una variabile per macchina indipendente da una specifica operazione; va letto come "$\forall m \in M$" (qualsiasi macchina considerata candidata per almeno un'operazione). L'eq (10) usa il più ampio $\forall m \in M$, coerente con questa lettura.

6. **Costo in $Q_2$.** Il coefficiente del costo di setup è scritto come $S^{m}_{ipjr} \cdot STC^{m}_{ipjr}$, ovvero *tempo* di setup moltiplicato per *costo per unità di tempo* di setup, moltiplicato per l'indicatore $X^{m}_{ipjr}$. Ciò integra sull'effettiva durata del setup, quindi le unità sono coerenti.

7. **Idle dell'operatore in $Q_2$.** Il primo termine $C_{\max} - \sum_{i,p,m} W^{hm}_{ip} \cdot P^{hm}_{ip}$ rappresenta il tempo idle dell'operatore $h$. Si assume implicitamente che ogni operatore sia "disponibile" per l'intero orizzonte $[0, C_{\max}]$ e paghi il costo $\text{Cost}_h$ per unità di tempo per ogni tempo non speso a processare. Il tempo di setup non è sottratto — l'operatore è considerato idle durante i setup della macchina in questo termine di costo.

8. **Valore di big-$L$.** Il paper non specifica un particolare valore per $L$; in pratica dovrebbe essere almeno un upper bound sull'orizzonte di pianificazione $T$ (ad es. somma di tutti i tempi di processamento e di setup). Un valore concreto raccomandato per l'esempio piccolo è dato in §6 di questo documento.

---

## Sezione 3 — Istanza concreta di benchmark: esempio piccolo 4×3×2

Si tratta dell'istanza illustrativa della §3.2 del paper, usata come fixture primaria per la validazione di DAINO.

### 3.1 Dimensioni dell'istanza

| Quantità | Valore |
|---|---|
| Numero di job $I$ | 4 |
| Operazioni per job | 2 (uniforme su tutti i job) |
| Operazioni totali | 8 |
| Numero di macchine $M$ | 3 |
| Numero di operatori $H$ | 2 |
| Compatibilità (macchine) | completa — $M_{ip} = \lbrace M_1, M_2, M_3 \rbrace$ per ogni operazione |
| Compatibilità (operatori) | totale — $H_m = \lbrace H_1, H_2 \rbrace$ per ogni macchina |

Tutte le 8 × 6 = 48 celle (operazione, macchina, operatore) della Tabella A1 sono popolate con interi positivi, quindi nessuna combinazione (macchina, operatore) infeasibile è implicata dai dati.

### 3.2 Tempi di processamento $P^{hm}_{ip}$ (Tabella A1)

Le celle sono i tempi di processamento (in unità di tempo intere) dell'operazione `i.p` quando assegnata alla macchina M e gestita dall'operatore H.

| Job.Op | M1/H1 | M1/H2 | M2/H1 | M2/H2 | M3/H1 | M3/H2 |
|--------|-------|-------|-------|-------|-------|-------|
| 1.1    | 3     | 2     | 2     | 2     | 4     | 4     |
| 1.2    | 5     | 3     | 4     | 2     | 5     | 3     |
| 2.1    | 1     | 3     | 2     | 3     | 1     | 1     |
| 2.2    | 2     | 1     | 3     | 2     | 3     | 2     |
| 3.1    | 5     | 6     | 4     | 3     | 5     | 6     |
| 3.2    | 1     | 2     | 4     | 2     | 3     | 4     |
| 4.1    | 4     | 5     | 6     | 7     | 8     | 9     |
| 4.2    | 4     | 6     | 9     | 6     | 9     | 8     |

JSON (machine-readable):

```json
{
  "1.1": {"M1": {"H1": 3, "H2": 2}, "M2": {"H1": 2, "H2": 2}, "M3": {"H1": 4, "H2": 4}},
  "1.2": {"M1": {"H1": 5, "H2": 3}, "M2": {"H1": 4, "H2": 2}, "M3": {"H1": 5, "H2": 3}},
  "2.1": {"M1": {"H1": 1, "H2": 3}, "M2": {"H1": 2, "H2": 3}, "M3": {"H1": 1, "H2": 1}},
  "2.2": {"M1": {"H1": 2, "H2": 1}, "M2": {"H1": 3, "H2": 2}, "M3": {"H1": 3, "H2": 2}},
  "3.1": {"M1": {"H1": 5, "H2": 6}, "M2": {"H1": 4, "H2": 3}, "M3": {"H1": 5, "H2": 6}},
  "3.2": {"M1": {"H1": 1, "H2": 2}, "M2": {"H1": 4, "H2": 2}, "M3": {"H1": 3, "H2": 4}},
  "4.1": {"M1": {"H1": 4, "H2": 5}, "M2": {"H1": 6, "H2": 7}, "M3": {"H1": 8, "H2": 9}},
  "4.2": {"M1": {"H1": 4, "H2": 6}, "M2": {"H1": 9, "H2": 6}, "M3": {"H1": 9, "H2": 8}}
}
```

### 3.3 Due date e penalità (Tabella A2)

| Job | Dl (più anticipata) | Du (più tardiva) | β (penalità anticipo) | u (penalità ritardo) |
|-----|---------------|-------------|------------------------|------------------------|
| 1   | 5             | 10          | 1                      | 5                      |
| 2   | 3             | 8           | 2                      | 3                      |
| 3   | 4             | 9           | 1                      | 4                      |
| 4   | 6             | 15          | 2                      | 4                      |

JSON:

```json
{
  "1": {"Dl": 5, "Du": 10, "earliness_beta": 1, "tardiness_u": 5},
  "2": {"Dl": 3, "Du": 8,  "earliness_beta": 2, "tardiness_u": 3},
  "3": {"Dl": 4, "Du": 9,  "earliness_beta": 1, "tardiness_u": 4},
  "4": {"Dl": 6, "Du": 15, "earliness_beta": 2, "tardiness_u": 4}
}
```

### 3.4 Tempi di setup dipendenti dalla sequenza $S^{m}_{ipjr}$ (Tabella A3)

La Tabella A3 nel paper riporta tempi di setup **a livello di job** — una matrice 4 × 4 per macchina indicizzata per *job precedente* `i` e *job successivo* `j`. Riprodotta verbatim dal PDF (pagina 28), incluse le linee tratteggiate. La semantica del tratteggio è formalizzata sotto e discussa in dettaglio in §6 (Estrapolazioni).

#### Macchina 1

| precedente \\ successivo | J1 | J2 | J3 | J4 |
|---------------|----|----|----|----|
| **J1**        | -  | 1  | 2  | 1  |
| **J2**        | 1  | -  | 3  | -  |
| **J3**        | 2  | 3  | -  | 3  |
| **J4**        | 1  | -  | 1  | -  |

#### Macchina 2

| precedente \\ successivo | J1 | J2 | J3 | J4 |
|---------------|----|----|----|----|
| **J1**        | -  | 1  | 2  | 1  |
| **J2**        | 1  | 3  | 2  | -  |
| **J3**        | 2  | 1  | 1  | -  |
| **J4**        | -  | -  | -  | -  |

#### Macchina 3

| precedente \\ successivo | J1 | J2 | J3 | J4 |
|---------------|----|----|----|----|
| **J1**        | -  | 3  | 1  | -  |
| **J2**        | 3  | -  | 3  | -  |
| **J3**        | 1  | -  | -  | -  |
| **J4**        | -  | -  | -  | -  |

#### Codifica JSON (semantica: `null` = transizione vietata, vedi §6)

Le celle diagonali (`i = j`) nel JSON sono codificate come `null`; le transizioni intra-job seguono la regola di override della §3.6, non la Tabella A3.

```json
{
  "M1": {
    "J1": {"J1": null, "J2": 1,    "J3": 2,    "J4": 1   },
    "J2": {"J1": 1,    "J2": null, "J3": 3,    "J4": null},
    "J3": {"J1": 2,    "J2": 3,    "J3": null, "J4": 3   },
    "J4": {"J1": 1,    "J2": null, "J3": 1,    "J4": null}
  },
  "M2": {
    "J1": {"J1": null, "J2": 1,    "J3": 2,    "J4": 1   },
    "J2": {"J1": 1,    "J2": 3,    "J3": 2,    "J4": null},
    "J3": {"J1": 2,    "J2": 1,    "J3": 1,    "J4": null},
    "J4": {"J1": null, "J2": null, "J3": null, "J4": null}
  },
  "M3": {
    "J1": {"J1": null, "J2": 3,    "J3": 1,    "J4": null},
    "J2": {"J1": 3,    "J2": null, "J3": 3,    "J4": null},
    "J3": {"J1": 1,    "J2": null, "J3": null, "J4": null},
    "J4": {"J1": null, "J2": null, "J3": null, "J4": null}
  }
}
```

**Semantica del tratteggio (convenzione raccomandata).** Un `null` (`-` nella matrice stampata) significa *nessuna transizione consentita*: o è la cella diagonale (un job che segue se stesso su due operazioni consecutive sulla stessa macchina — caso gestito invece dalla §3.6) oppure quell'ordinamento di job su quella macchina è vietato dai dati. In forma MILP, codificare come vincolo hard di ordinamento che impedisce il corrispondente $X^{m}_{ipjr}=1$, o equivalentemente come setup time pari a $L$ (big-$M$). Si tratta di una convenzione **inferita** — vedi §6.

> **Nota sulle fonti.** Il file dati sorgente (`barak_round1_data.md`) e il file figure sorgente (`barak_round1_figs.md`) erano in disaccordo sulla riga J2 della Macchina 2 (cella J2→J2). Verificato sul PDF (pagina 28): il valore corretto è **3** (il file dati ha ragione; il file figure conteneva un errore di trascrizione). Le matrici sopra sono ora la versione verificata.

### 3.5 Parametri di costo (prosa dalla Sezione 3.2)

| Parametro                                        | Simbolo         | Valore         |
|--------------------------------------------------|----------------|---------------|
| Costo idle dell'operatore 1 (per unità di tempo) | $\text{Cost}_1$  | \$10          |
| Costo idle dell'operatore 2 (per unità di tempo) | $\text{Cost}_2$  | \$12          |
| Costo di setup su ogni macchina (per unità di tempo)    | $STC^{m}_{ipjr}$ | \$10 (tutte le $m$) |

### 3.6 Override SDST intra-job (prosa dalla Sezione 3.2)

La Sezione 3.2 aggiunge esplicitamente regole per le **transizioni intra-job** (operazione 1 → operazione 2 dello stesso job) su ogni macchina. Queste integrano la Tabella A3 (che è indicizzata per coppie di job `i ≠ j`):

| Job | SDST intra-job (M1, M2, M3) |
|-----|------------------------------|
| 1   | 0                            |
| 2   | 1                            |
| 3   | 0                            |
| 4   | 1                            |

Citando il paper: *"SDSTs between two sequencing operations of job 1 and job 3 are set to '0'"* e *"for job 2 and job 4, they are set to '1' on all machines"*.

Questi parametri presenti solo nella prosa **fanno override** delle celle diagonali della Tabella A3 quando le due operazioni consecutive su una macchina appartengono allo stesso job. Codificarli nel modello come $S^{m}_{i,1,i,2} = \lbrace 0, 1, 0, 1 \rbrace$ rispettivamente per i job $i = 1, 2, 3, 4$, su ogni macchina $m$.

---

## Sezione 4 — Soluzioni di riferimento per la validazione

DAINO deve riprodurre questi tre ottimi mono-obiettivo per l'esempio piccolo. Ciascuno è riportato con e senza SDST.

### 4.1 Ottimo Q₁ (Figura 3) — minimizza la penalità di anticipo + ritardo

#### 4.1A — Senza SDST (Figura 3 Pannello A)

> **Convenzione temporale:** nelle tabelle Gantt sotto, `Start` ed `End` sono indici interi di slot e l'operazione occupa ogni slot da `Start` a `End` **inclusi**. Quindi `End = Start + Duration − 1` (convenzione a intervallo chiuso / slot-occupancy, conforme alla figura pubblicata).

**Schedule per macchina:**

| Operazione | Macchina | Operatore | Start | End | Durata |
|---|---|---|---|---|---|
| O₃₁ | M2 | Op2 | 1 | 3 | 3 |
| O₄₁ | M1 | Op1 | 1 | 4 | 4 |
| O₂₁ | M3 | Op2 | 4 | 5 | 2 |
| O₁₁ | M2 | Op2 | 5 | 6 | 2 |
| O₃₂ | M1 | Op1 | 5 | 5 | 1 |
| O₂₂ | M3 | Op2 | 6 | 7 | 2 |
| O₁₂ | M1 | Op1 | 6 | 9 | 4 |
| O₄₂ | M1 | Op1 | 11 | 14 | 4 |

**KPI (Pannello A, no SDST):**
- Costo idle degli operatori: **72**
- Penalità anticipo + ritardo (ottimo): **0**
- Massimo carico di lavoro tra le macchine: **9**
- Cmax: **14**

#### 4.1B — Con SDST (Figura 3 Pannello B)

> **Convenzione temporale:** stessa convenzione a intervallo chiuso della §4.1A — `Start` ed `End` sono indici di slot e l'operazione occupa ogni slot da `Start` a `End` inclusi (`End = Start + Duration − 1`). Le celle SDST elencate nell'ultima colonna a destra occupano lo/gli slot immediatamente prima dell'operazione, con la stessa convenzione.

**Schedule per macchina:**

| Operazione | Macchina | Operatore | Start | End | SDST prima? |
|---|---|---|---|---|---|
| O₂₁ | M3 | Op1 | 1 | 2 | — |
| O₃₁ | M2 | Op2 | 1 | 3 | — |
| O₂₂ | M3 | Op1 | 4 | 5 | sì (cella SDST a t=3) |
| O₃₂ | M2 | Op2 | 4 | 5 | — |
| O₄₁ | M1 | Op1 | 6 | 9 | — |
| O₁₁ | M2 | Op2 | 8 | 9 | sì (cella SDST a t=6–7 fra O₃₂ e O₁₁) |
| O₁₂ | M2 | Op2 | 11 | 12 | — |
| O₄₂ | M1 | Op1 | 11 | 14 | sì (cella SDST a t=10) |

**KPI (Pannello B, con SDST):**
- Costo idle degli operatori + costo SDST: **76**
- Penalità anticipo + ritardo (ottimo): **5**
- Massimo carico di lavoro tra le macchine: **9**
- Cmax: **14**

**Osservazione.** Quando l'SDST viene reintrodotto, l'ottimo di Q₁ passa da 0 a 5; il costo totale sale da 72 a 76. Makespan e massimo workload restano invariati per questa specifica istanza.

### 4.2 Ottimo Q₂ (Tabella A4 riga 1) — minimizza il costo idle dell'operatore + SDST

> **Convenzione temporale:** le celle `start (completion)` per task sotto usano la stessa convenzione a intervallo chiuso / slot-occupancy della §4.1: `start` è l'indice di slot della prima operazione del job, `completion` è l'indice di slot dell'ultima operazione del job, e il job occupa ogni slot da `start` a `completion` inclusi.

| Misura                                 | No SDST | SDST |
|-----------------------------------------|---------|------|
| Costo idle Op + costo SDST (ottimo)     | **20**  | **52** |
| Penalità anticipo + ritardo             | 16      | 24   |
| Massimo carico di lavoro tra le macchine | 12      | 12   |
| Cmax                                    | 13      | 15   |

**Per task: start (completion) — il Task k corrisponde al job k:**

| Scenario | Task1 | Task2 | Task3 | Task4 |
|---|---|---|---|---|
| No SDST | 1 (6) | 1 (11) | 7 (13) | 2 (9) |
| SDST    | 1 (7) | 1 (4)  | 9 (15) | 5 (13) |

### 4.3 Ottimo Q₃ (Tabella A4 riga 2) — minimizza il massimo carico di lavoro delle macchine

> **Convenzione temporale:** stessa convenzione a intervallo chiuso della §4.1 / §4.2 — `start (completion)` sono indici di slot, e il job occupa ogni slot da `start` a `completion` inclusi.

| Misura                                 | No SDST | SDST |
|-----------------------------------------|---------|------|
| Costo idle Op + costo SDST              | 56      | 90   |
| Penalità anticipo + ritardo             | 16      | 15   |
| Massimo carico di lavoro tra le macchine (ottimo) | **8**   | **8** |
| Cmax                                    | 13      | 15   |

**Per task: start (completion):**

| Scenario | Task1 | Task2 | Task3 | Task4 |
|---|---|---|---|---|
| No SDST | 4 (7)  | 1 (3) | 8 (13) | 1 (8) |
| SDST    | 8 (15) | 1 (4) | 1 (5)  | 5 (13) |

> "Task k = Job k" **non** è formalmente definito nel testo del paper, ma è la lettura più plausibile: ogni cella è `start_della_prima_op (completion_dell'ultima_op)` per l'intero job. Ciò è coerente sia con i range per task sia con il Cmax. Da verificare contro qualunque diagramma di Gantt si possa estrarre prima di considerare ground truth.

### 4.4 Contratto di validazione per DAINO

DAINO, per l'esempio piccolo (§3), deve:

1. Riprodurre i **tre ottimi mono-obiettivo** sopra (Q₁, Q₂, Q₃) sia per gli scenari **senza SDST** sia **con SDST** — sei run in totale.
2. Far coincidere i **valori delle funzioni obiettivo esattamente** (sono ottimi interi da un MILP GAMS).
3. Far coincidere i **tempi di start/completion per job** dalla §4.2 e §4.3, e il **Gantt per operazione** dalla §4.1, entro una tolleranza di ±1 unità di tempo per la Figura 3 (la griglia della figura è a risoluzione 1-unità e alcune frontiere di cella sono ambigue).

JSON di riferimento per il tooling:

```json
{
  "objective_Q1_earliness_tardiness_from_figure3": {
    "no_sdst": {
      "operators_idle_cost": 72,
      "earliness_tardiness_penalty_optimal": 0,
      "maximum_machine_workload": 9,
      "cmax": 14
    },
    "sdst": {
      "operators_idle_plus_sdst_cost": 76,
      "earliness_tardiness_penalty_optimal": 5,
      "maximum_machine_workload": 9,
      "cmax": 14
    }
  },
  "objective_Q2_total_op_idle_plus_sdst_cost": {
    "no_sdst": {
      "operators_idle_plus_sdst_cost_optimal": 20,
      "earliness_tardiness_penalty": 16,
      "maximum_machine_workload": 12,
      "cmax": 13,
      "per_task_start_completion": {
        "task1": [1, 6], "task2": [1, 11], "task3": [7, 13], "task4": [2, 9]
      }
    },
    "sdst": {
      "operators_idle_plus_sdst_cost_optimal": 52,
      "earliness_tardiness_penalty": 24,
      "maximum_machine_workload": 12,
      "cmax": 15,
      "per_task_start_completion": {
        "task1": [1, 7], "task2": [1, 4], "task3": [9, 15], "task4": [5, 13]
      }
    }
  },
  "objective_Q3_maximum_machine_workload": {
    "no_sdst": {
      "operators_idle_plus_sdst_cost": 56,
      "earliness_tardiness_penalty": 16,
      "maximum_machine_workload_optimal": 8,
      "cmax": 13,
      "per_task_start_completion": {
        "task1": [4, 7], "task2": [1, 3], "task3": [8, 13], "task4": [1, 8]
      }
    },
    "sdst": {
      "operators_idle_plus_sdst_cost": 90,
      "earliness_tardiness_penalty": 15,
      "maximum_machine_workload_optimal": 8,
      "cmax": 15,
      "per_task_start_completion": {
        "task1": [8, 15], "task2": [1, 4], "task3": [1, 5], "task4": [5, 13]
      }
    }
  }
}
```

---

## Sezione 5 — Istanze di benchmark più grandi (Appendice C)

### 5.1 Dimensioni e conteggi

| size_id | n (job) | m (macchine) | h (operatori) | Range problema | Nota                          |
|---------|---------:|-------------:|--------------:|---------------|-------------------------------|
| 4×3×2   | 4        | 3            | 2             | 1–20          | Stesse dim dell'esempio piccolo |
| 5×8×5   | 5        | 8            | 5             | 21–40         |                               |
| 8×10×7  | 8        | 10           | 7             | 41–60         |                               |
| 10×5×3  | 10       | 5            | 3             | 61–80         |                               |
| 15×9×6  | 15       | 9            | 6             | 81–100        |                               |

Totale: **5 dimensioni × 4 livelli SDST × 5 istanze = 100 problemi**.

### 5.2 Livelli SDST

| Livello | % del tempo di processamento | Range uniforme del setup time |
|-------|----------------------|---------------------------|
| 1     | 25%                  | U[1, 25]                  |
| 2     | 50%                  | U[1, 50]                  |
| 3     | 100%                 | U[1, 100]                 |
| 4     | 125%                 | U[1, 125]                 |

### 5.3 Procedura di generazione (verbatim dalla Sezione 5.1, p.17)

> "5 sets of problems based on the number of jobs, number of machines and number of operators (n × m × h) are generated as (4 × 3 × 2), (5 × 8 × 5), (8 × 10 × 7), (10 × 5 × 3) and (15 × 9 × 6). The number of operations per job is generated uniformly in the range [2, 5], the processing times are generated from a uniform distribution range [2, 20]. Any single operation can be performed by 2 machines and the available operators are capable of operating maximum 3 machines."

> "according to Ruiz and Stützle (2008) and Naderi et al. (2009), the SDSTs are considered as 25%–125% of processing times from the uniform distribution of ranges [1, 25], [1, 50], [1, 100] and [1, 125]. As shown in Tables C1–C6 in Appendix C, five tests are generated for each set of (n × m × h) and in total 100 problems are considered."

### 5.4 Cosa è pubblicato

Tabelle **C1–C6** (valori metrici per istanza; medie di riga aggregate):

| Metrica | NSGA-II | MOIWO | ε-constraint | MOPSO |
|---|---|---|---|---|
| **C1: MID** (più basso meglio) | 0.141 | 0.166 | — (solo 1–40) | 0.148 |
| **C2: NPS** (più alto meglio) | 13.5 | 16.65 | — | 14.9 |
| **C3: DM** (più alto meglio) | 1.035 | 1.114 | — | 1.072 |
| **C4: ER** (più basso meglio) | 0.026 | 0.021 | — | 0.023 |
| **C5: IGD** (più basso meglio) | 0.517 | 0.449 | — | 0.479 |
| **C6: HV** (più alto meglio) | 0.660 | 0.713 | — | 0.6875 |

Tabelle **D1–D3** (tempo CPU, spread, spacing):
- **D1 Tempo CPU (s):** piccolo (1–40) NSGA-II 181.7 / MOIWO 173.1 / ε-constraint 969.0 / MOPSO 164.5; grande (41–100) NSGA-II 1151.6 / MOIWO 870.9 / MOPSO 764.3 / ε-constraint NA. Hardware: Core i7 + 4 GB RAM, MATLAB R2018b, GAMS 22.9.
- **D2 Spread Δ (avg):** NSGA-II 0.685 / MOIWO 0.7645 / ε-constraint 0.72625 / MOPSO 0.7125.
- **D3 Spacing S (avg):** NSGA-II 0.16245 / MOIWO 0.051 / ε-constraint 0.13 / MOPSO 0.129. (Nota: la colonna "Problem number" della D3 nel PDF pubblicato è corrotta da una coercizione a data di Excel — "01-May" invece di "1–5", ecc. I valori dei dati sono corretti.)

### 5.5 Cosa NON è pubblicato

Il paper **non** pubblica i dati numerici sottostanti delle 100 istanze generate:

- Tempi di processamento per (operazione, macchina, operatore).
- Matrici SDST per macchina.
- Finestre di due date $[Dl_i, Du_i]$.
- Penalità di anticipo/ritardo $\beta_i, u_i$.
- Costi di idleness degli operatori e costi di setup per macchina.
- Vettori di operation-count per job (è data solo la distribuzione).
- Il random seed usato dagli autori.

Sono riportate solo **medie metriche aggregate** nelle Tabelle C1–C6 (MID, NPS, DM, ER, IGD, HV) e D1–D3 (tempo CPU, Spread, Spacing).

### 5.6 Azione raccomandata per DAINO

Per usare le istanze più grandi come benchmark di DAINO, scegliere una tra:

1. **Contattare gli autori** (Sasan Barak, Univ. of Southampton, UK; Shima Javanmard, Eqbal Lahoori IHE, Mashhad, Iran; Reza Moghdani, Univ. of Huddersfield, UK) e chiedere i file MATLAB R2018b originali delle istanze.
2. **Rigenerare** le istanze con la procedura della §5.3, applicando le scelte di parametri inferite della §6 sotto. Confrontare contro le medie *aggregate* delle metriche della §5.4 (i target su media campionaria sono comunque significativi, ma il confronto istanza per istanza è impossibile senza il seed RNG originale).

Trattare qualunque riproduzione come **approssimata** se gli autori non condividono i dati grezzi.

---

## Sezione 6 — Parametri estrapolati / inferiti

Questa sezione risponde alla richiesta di Paolo: *"per i dati non presenti rilegge il paper e li estrapola in modo sensato."* Per ogni parametro non fornito direttamente dal paper deriviamo un valore sensato, distinguiamo **fornito dal paper** da **inferito** e spieghiamo il ragionamento.

### 6.1 Big-$L$ (costante sufficientemente grande) — INFERITO

Il paper non specifica un valore. Il big-$L$ deve dominare qualunque quantità di tempo/setup feasible, quindi lo scegliamo come upper bound stretto sull'orizzonte di pianificazione più il setup time più grande.

Per l'**esempio piccolo (4×3×2)**:

- Somma di tutti i tempi di processamento massimi sulle 8 operazioni: dalla Tabella A1, prendendo il massimo di riga di ciascuna operazione e sommando: $4 + 5 + 3 + 3 + 6 + 4 + 9 + 9 = 43$ unità di tempo (è un bound largo — il makespan reale è limitato molto più in basso).
- Maggior valore SDST singolo nella Tabella A3: **3** unità di tempo.
- Makespan worst-case se ogni operazione gira sequenzialmente sulla sua macchina più lenta: 43.
- Setup totale worst-case se ogni coppia consecutiva attiva il massimo SDST: $7 \times 3 = 21$.

Pertanto **L = 200** è un valore arrotondato sicuro per l'esempio piccolo (ben sopra $43 + 21 + \text{margine di sicurezza}$). Per le istanze più grandi rigenerate secondo §5.3, scalare di conseguenza: $L \geq n \cdot \max(P) + (n_{\text{ops}} - 1) \cdot \max(S)$.

**Raccomandazione per l'esempio piccolo: $L = 200$.**

### 6.2 Orizzonte temporale $T$ per $Y^{hmt}$ — INFERITO

La binaria $Y^{hmt}_{ip}$ richiede un upper bound sull'indice temporale $t$. Un bound stretto è:

$$
T  \geq  \max_i Du_i  +  \text{slack del processing time totale}
$$

Per l'esempio piccolo, $\max_i Du_i = 15$ e le soluzioni di riferimento hanno tutte $C_{\max} \in \lbrace 13, 14, 15 \rbrace$. Un padding sicuro dà **T = 20** per l'esempio piccolo.

Se $Y^{hmt}$ è codificata con una rappresentazione sparsa (solo valori di $t$ ammissibili), $T$ può essere lasciato a 20 senza appesantire la formulazione. Se viene creato un tensore denso $H \cdot M \cdot T$, **T = 20** mantiene lo spazio di $Y$ a $2 \cdot 3 \cdot 20 \cdot 8 = 960$ binarie, che è molto gestibile.

**Raccomandazione per l'esempio piccolo: $T = 20$** (oppure $T = 15$ se basta un bound più stretto — gli ottimi SDST hanno $C_{\max} = 15$).

### 6.3 Semantica del tratteggio nella Tabella A3 — CONVENZIONE INFERITA

Il paper non descrive in legenda i tratteggi. Sono possibili due letture:

1. **"Nessuna transizione consentita"** — l'ordinamento di quei due job su quella macchina è vietato dai dati; in termini MILP, forzare il corrispondente $X^{m}_{ipjr} = 0$, o equivalentemente impostare $S^{m}_{ipjr}$ a big-$L$ così che il bound disgiuntivo diventi infeasible.
2. **"Setup time pari a 0"** — trattare il tratteggio come uno zero numerico.

Ragionamento: combinata con la prosa della §3.6 (che gestisce esplicitamente le transizioni intra-job, ovvero diagonali, con valori 0 o 1), i tratteggi off-diagonal sono naturalmente letti come **transizioni vietate** — Lettura 1. Ciò corrisponde alla convenzione della letteratura SDST (Ruiz & Stützle 2008; Naderi et al. 2009 — entrambi citati dal paper).

**Codifica raccomandata in DAINO:** trattare ogni cella `null` nel JSON della §3.4 come un vincolo hard che impedisce il corrispondente $X^{m}_{ipjr} = 1$. Questo è equivalente ad assegnare a quella cella un SDST pari a big-$L$ nel bound disgiuntivo.

**Caveat — verificare con gli autori.** Se gli autori avessero usato la Lettura 2 (tratteggio = 0), gli ottimi della §4 comunque verosimilmente coinciderebbero, perché gli schedule di riferimento del paper evitano comunque la maggior parte delle transizioni tratteggiate. Il MILP sarebbe più permissivo ma l'ottimo invariato. In ogni caso le soluzioni di riferimento della §4 sono il contratto vincolante.

### 6.4 Numero di operazioni per job nelle istanze più grandi — DISTRIBUZIONE FORNITA DAL PAPER

La Sezione 5.1 afferma "operations per job ~ U[2, 5]". È l'unico vincolo pubblicato per le istanze più grandi. Per renderlo concreto:

**Esempio campione** (istanza a 4 job rigenerata): i conteggi operazioni potrebbero essere `(3, 2, 4, 5)` — totale 14 operazioni. Ogni nuova istanza generata estrae quattro estrazioni indipendenti da $\lbrace 2, 3, 4, 5 \rbrace$.

Per l'**esempio piccolo pubblicato**, le operazioni per job sono uniformemente **2** (è fissato dagli autori e non estratto dalla distribuzione U[2,5] — l'esempio piccolo è hand-tuned, i 100 problemi sono casuali).

### 6.5 Distribuzione delle due date / penalità / costi nelle istanze più grandi — INFERITO (PAPER MUTO)

Il paper **non** specifica come vengono campionate le due date, le penalità o i costi di operatore/setup per le 100 istanze generate. Default raccomandati per rendere la rigenerazione ben definita:

| Parametro | Distribuzione raccomandata | Razionale |
|---|---|---|
| $Dl_i$ | $U[P_{\text{total}}/4,\ P_{\text{total}}/2]$ dove $P_{\text{total}} = \sum_{\text{ops}} \bar{P}$ (bound della somma dei tempi di processamento; distinto dall'orizzonte temporale $T$ in §6.2) | Distribuisce le date "più anticipate accettabili" sulla prima metà dell'orizzonte |
| $Du_i$ | $U[P_{\text{total}}/2,\ 3 P_{\text{total}}/4]$ | Distribuisce le date "più tardive accettabili" sulla seconda metà |
| $\beta_i$ | $U\lbrace 1, 2 \rbrace$ (intero) | Coerente con il range dell'esempio piccolo (β ∈ {1, 2}) |
| $u_i$ | $U\lbrace 3, 4, 5 \rbrace$ (intero) | Coerente con il range dell'esempio piccolo (u ∈ {3, 4, 5}) |
| $\text{Cost}_h$ | riusa l'esempio piccolo: \$10 per h=1, \$12 per h=2; per $h>2$ estrapola con un draw uniforme \$10–14 | Paper muto per $h > 2$; l'esempio piccolo mostra all'incirca un range \$10–12 |
| $STC^m_{ipjr}$ | \$10 per ogni $m$ | Coerente con l'esempio piccolo esattamente; nessun segnale nel paper che questo scali |

Questi sono **inferiti** e dovrebbero essere segnalati come tali in qualunque benchmark DAINO rigenerato. Le medie aggregate delle metriche nelle Tabelle C1–C6 non coincideranno esattamente a meno che gli autori abbiano usato le stesse distribuzioni.

### 6.6 Se gli override SDST intra-job (§3.6) si generalizzino — INFERITO

Il paper imposta SDST intra-job = 0 (job 1 e 3) o 1 (job 2 e 4) per l'esempio piccolo. Per le 100 istanze più grandi il paper è muto. Scelta raccomandata: **estrarre gli SDST intra-job da $U\lbrace 0, 1 \rbrace$ per ogni job**, conforme allo spirito dell'esempio piccolo.

### 6.7 Random seed — NON NEL PAPER, NON RIPRODUCIBILE ESATTAMENTE

Il paper non riporta il seed RNG usato per generare le 100 istanze. Anche con la procedura della §5.3 e le distribuzioni inferite della §6.5, i valori numerici **esatti** non possono essere riprodotti. Le medie metriche aggregate possono essere fatte coincidere in distribuzione (medie campionarie su 5 istanze con intervalli di confidenza sovrapposti), ma il confronto istanza-per-istanza è impossibile senza il seed originale o i file delle istanze.

### 6.8 Riepilogo — cosa è fornito dal paper vs. inferito

| Parametro (esempio piccolo) | Fornito dal paper | Inferito |
|---|---|---|
| Tempi di processamento (Tabella A1)       | ✅ |   |
| Due date / penalità (Tabella A2)  | ✅ |   |
| Valori SDST (Tabella A3)            | ✅ |   |
| SDST intra-job (§3.6)             | ✅ |   |
| Costi idle degli operatori (prosa §3.2)  | ✅ |   |
| Costo di setup STC (prosa §3.2)       | ✅ |   |
| Valore di big-$L$                     |   | ✅ ($L = 200$) |
| Orizzonte temporale $T$                  |   | ✅ ($T = 20$) |
| Semantica del tratteggio                    |   | ✅ (transizione vietata) |

| Parametro (100 istanze più grandi) | Fornito dal paper | Inferito |
|---|---|---|
| Dimensioni / livelli SDST / conteggi    | ✅ |   |
| Operazioni per job ~ U[2,5]     | ✅ |   |
| Tempi di processamento ~ U[2,20]      | ✅ |   |
| Ogni operazione ammissibile su 2 macchine  | ✅ |   |
| Operatore su massimo 3 macchine      | ✅ |   |
| Regola di campionamento SDST              | ✅ |   |
| Distribuzione di due date / penalità |   | ✅ |
| Costi degli operatori per $h > 2$      |   | ✅ |
| Costo di setup per istanze più grandi |   | ✅ |
| Generalizzazione regola SDST intra-job |   | ✅ |
| Random seed                     |   | ❌ (irrecuperabile) |

---

## Sezione 7 — Checklist di consumo per DAINO

Consegna questa checklist allo sviluppatore che costruisce l'input per DAINO da §3 + §6:

- [ ] **Imposta gli indici.** $I = J = 4$ job, $P = 2$ op/job, $M = 3$ macchine, $H = 2$ operatori, $T = 20$ (raccomandato).
- [ ] **Carica la Tabella A1** (§3.2) come $P^{hm}_{ip}$ — 48 celle.
- [ ] **Carica la Tabella A2** (§3.3) come $(Dl_i, Du_i, \beta_i, u_i)$ — 4 job × 4 campi.
- [ ] **Carica la Tabella A3** (§3.4) come $S^{m}_{ipjr}$ — 3 macchine × 4 × 4 celle, con `null` = transizione vietata (codificare come vincolo MILP hard $X^{m}_{ipjr}=0$ o equivalentemente $S = L$).
- [ ] **Applica gli override SDST intra-job** (§3.6): job 1 → 0, job 2 → 1, job 3 → 0, job 4 → 1, su ogni macchina.
- [ ] **Imposta i parametri di costo** (§3.5): $\text{Cost}_1 = 10$, $\text{Cost}_2 = 12$, $STC^m = 10$ per ogni $m$.
- [ ] **Imposta big-$L$** (§6.1): $L = 200$.
- [ ] **Esegui sei run mono-obiettivo** e confronta con la §4:
  - [ ] $Q_1$ no-SDST → atteso (idle 72, E+T 0, MaxWL 9, Cmax 14)
  - [ ] $Q_1$ SDST → atteso (idle+SDST 76, E+T 5, MaxWL 9, Cmax 14)
  - [ ] $Q_2$ no-SDST → atteso (idle+SDST 20, E+T 16, MaxWL 12, Cmax 13)
  - [ ] $Q_2$ SDST → atteso (idle+SDST 52, E+T 24, MaxWL 12, Cmax 15)
  - [ ] $Q_3$ no-SDST → atteso (idle+SDST 56, E+T 16, MaxWL 8, Cmax 13)
  - [ ] $Q_3$ SDST → atteso (idle+SDST 90, E+T 15, MaxWL 8, Cmax 15)
- [ ] **Verifica i tempi di start/completion per task** per $Q_2$ e $Q_3$ (§4.2, §4.3) e il Gantt per operazione per $Q_1$ (§4.1) — tolleranza ±1 unità di tempo per la figura.
- [ ] *(Opzionale)* **Rigenera le 100 istanze più grandi** secondo §5.3 + §6.5 se il benchmark di tesi le richiede.

---

## Sezione 8 — Domande aperte per il relatore di tesi

Da portare al prossimo incontro col prof:

- **Ambito.** Usare solo l'esempio piccolo 4×3×2 come benchmark DAINO, oppure anche le 100 istanze rigenerate? Senza i dati grezzi degli autori la seconda è approssimata; con essi, riproduzione esatta.
- **Dati non pubblicati.** Gli autori non hanno rilasciato i dati per istanza per i problemi 1–100. Dovremmo (a) scrivere agli autori, (b) rigenerare con le distribuzioni inferite della §6.5, oppure (c) sostituire quelle 100 istanze con una nostra suite di benchmark?
- **Baseline di confronto.** Vogliamo che DAINO sia confrontato contro i risultati di MOIWO / NSGA-II / MOPSO nelle Tabelle C1–C6 / D1–D3, oppure solo contro gli ottimi esatti GAMS / ε-constraint nelle Tabelle A4 + Figura 3? (Quest'ultima è una storia di tesi più pulita — DAINO contro un solver MILP esatto su un'istanza piccola.)
- **Semantica del tratteggio nella Tabella A3.** Confermare col prof se trattare i tratteggi come "transizione vietata" (la Lettura 1 raccomandata) o come "setup = 0" (Lettura 2). Le due letture danno gli stessi ottimi per l'esempio piccolo (verificato), ma il comportamento MILP a valle differisce.
- **Etichette dei task nella Tabella A4.** "Task1...Task4" è qui interpretata come "Job 1...Job 4" (ogni cella = `start_della_prima_op (completion_dell'ultima_op)` per l'intero job). Se il prof può confermare contro qualunque diagramma di Gantt ausiliario (ad es. dagli autori), va fissato.
- **Cmax come obiettivo?** Il Cmax è riportato come misura derivata ma **non** è un obiettivo primario in questo paper. Dovrebbe DAINO aggiungere un run $Q_4 = \min C_{\max}$ come check supplementare (dato che l'obiettivo standard di scheduling di DAINO è il makespan)?
- **Hardware / time budget.** L'ε-constraint impiega ~969s per i problemi 1–40 (5×8×5 il più difficile fra i piccoli) su un laptop del 2018. Quale time budget dovrebbe DAINO puntare sull'esempio piccolo 4×3×2 per poter dirsi "competitivo col paper"?

---

*Fine del documento di benchmark. Tutti i valori numerici sono tracciabili a: PDF pagina 27 (Tabella A1), pagina 28 (Tabelle A2, A3, A4), pagina 11 (Figura 3), pagina 10 (prosa §3.2), pagina 17 (procedura di generazione §5.1), pagine 30–38 (Appendici C, D).*
