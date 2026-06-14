# StudyScheduler

## **Idea di base**

Il sistema costruisce un piano di studio che sfrutta la **ripetizione spaziata**: 
ogni argomento studiato il giorno $n$ viene ripassato a intervalli crescenti ($n+1$, $n+3$, $n+7$), sfruttando il fatto che rivedere materiale già visto è molto più rapido che studiarlo la prima volta.

La caratteristica principale di questo software è che il piano è **dinamico**: 
se un giorno si studia più o meno del previsto, tutto il piano si aggiorna automaticamente a cascata.

---

## **Struttura del piano**

### Giorni di studio e giorni di ripasso

Il piano alterna in modo rigido due tipi di giorno:

- **Giorni dispari** ($1°, 3°, 5°, ...$): 
	- giorni di **studio**, in cui si affrontano argomenti nuovi (o si continua un argomento iniziato il giorno di studio precedente).
- **Giorni pari** ($2°, 4°, 6°, ...$): 
	- giorni di **ripasso**, in cui si rivedono gli argomenti che "scadono" quel giorno secondo gli intervalli di ripetizione spaziata.

Questi due tipi non si mescolano mai. 
Un giorno è quindi sempre esclusivamente dell'uno o dell'altro tipo.

### Sequenza vs. calendario

I numeri "dispari/pari" si riferiscono alla **sequenza dei giorni disponibili**, non al calendario. 
Il 3° giorno disponibile è sempre un giorno di studio, il 4° sempre un giorno di ripasso, indipendentemente da quando cadono sul calendario.  
I weekend e i festivi vengono semplicemente saltati, senza che questo cambi la logica dell'alternanza.  


---

## **Costruzione del piano iniziale**

L'input del sistema è:

- **Data di inizio** e **data dell'esame**
- **Lista degli argomenti** con il numero di pagine di ciascuno
- **Configurazione del calendario**: di default si studia solo dal lunedì al venerdì. Ma può essere impostato in qualsiasi modo. 

Il sistema distribuisce automaticamente gli argomenti nei giorni di studio disponibili, in ordine, uno per giorno. 
Nei giorni di ripasso inserisce automaticamente tutti gli argomenti che "scadono" quel giorno secondo gli intervalli.

I giorni che rimangono liberi dopo aver distribuito tutti gli argomenti sono destinati a esercizi.

---

## **Ripassi**

### Intervalli standard

Ogni argomento viene ripassato nei giorni in cui scadono questi intervalli calcolati dalla data di studio effettiva:

- **+1 giorno**: ripasso immediato
- **+3 giorni**: ripasso a breve termine
- **+7 giorni**: ripasso a medio termine
- **+15 giorni**: ripasso a lungo termine (opzionale) 

Poiché i ripassi devono cadere su giorni pari, se un intervallo cade su un giorno dispari (di studio), il ripasso viene automaticamente spostato al giorno pari successivo.

## Quarto ripasso

Esiste un quarto intervallo a **+15 giorni**, disabilitato di default.  
Può essere abilitato in qualsiasi momento, e tutti i ripassi già pianificati vengono aggiornati.

## I ripassi sono "best effort"

I giorni di ripasso non sono critici: se un giorno di ripasso non viene completato, non è necessario recuperarlo. 
Non esiste un meccanismo di recupero per i ripassi mancati e non verrà implementato

## **Aggiornamento del progresso**

Al termine di ogni giorno di studio, si aggiorna il piano comunicando la pagina effettivamente raggiunta. 
Il sistema gestisce automaticamente tutti i casi.

### Caso normale

Si arriva esattamente alla pagina pianificata. Non cambia nulla.

### Underperformance (si studia meno del previsto)

Se si pianificava di arrivare a pagina 40 ma ci si ferma a pagina 37, l'argomento viene spezzato in due blocchi indipendenti:

- Il primo blocco (pp. 1–37) resta al giorno corrente e ottiene i suoi ripassi calcolati a partire da oggi.
- Il secondo blocco (pp. 38–40) diventa un blocco autonomo che viene inserito in testa al giorno di studio successivo.
- Tutti gli argomenti successivi slittano in avanti di un giorno.
- I due blocchi vengono ripassati **indipendentemente**, ognuno con i propri intervalli calcolati dalla propria data di studio. Come se fossero due argomenti diversi.

### Overperformance (si studia più del previsto)

Se si pianificava di arrivare a pagina 40 ma si arriva a pagina 43, le pagine "in più" vengono **assorbite dal blocco successivo**, che inizierà da pagina 44 invece che da 41.  
Se le pagine extra sono sufficienti ad assorbire completamente il blocco successivo, quel blocco scompare e si passa al seguente.

### Zero pagine (giorno saltato completamente)

Se in un giorno di studio non si riesce a fare nulla, l'intero argomento pianificato per quel giorno viene spostato al giorno di studio successivo.  
Il giorno corrente diventa libero, e il giorno di ripasso immediatamente successivo rimane vuoto (non c'è nulla da ripassare, perché nulla è stato studiato). 
La sequenza di ripassi per quell'argomento partirà dalla nuova data di studio effettiva.

### Cascata a tutti i livelli

In tutti i casi di rescheduling (under, over, zero pagine), la modifica si propaga automaticamente a cascata su tutti i giorni successivi.

---

## **Gestione del calendario**

### Configurazione di default

Di default il sistema usa solo i giorni lavorativi (lunedì–venerdì). 
Sabato e domenica vengono ignorati.

### Aggiunta di giorni durante il piano

In qualunque momento è possibile abilitare il sabato, la domenica, entrambi o addirittura impostare un calendario di studio personalizzato.
Ad esempio, si può decidere che si ha a disposizione solo il lunedì, il martedì e il venerdì. 
Il sistema distingue due situazioni:

- **Nessun giorno ancora studiato**: il calendario viene ricostruito da zero con le nuove impostazioni.
- **Alcuni giorni già studiati**: i giorni passati vengono congelati esattamente com'erano (i loro numeri di sequenza non cambiano, le date di ripasso restano valide). Solo i giorni futuri vengono rigenerati con le nuove impostazioni.

### Giorni off specifici (festivi, impegni)

È possibile in qualsiasi momento aggiungere o rimuovere singoli giorni.  
- L'aggiunta di un giorno libero rimuove quel giorno dalla sequenza e riorganizza i giorni futuri. 
- La rimozione di un festivo reinserisce quel giorno e compatta il piano.

---

## **Overflow**

Se gli argomenti sono troppi per entrare tutti nel periodo disponibile prima dell'esame, quelli in eccesso vengono messi in una coda di overflow.  
Se in seguito si aggiungono giorni al calendario (ad esempio abilitando il sabato), il sistema tenta automaticamente di recuperare questi argomenti.

---



## **Riepilogo dei casi particolari**

| **Situazione**                                     | **Comportamento**                                                                           |
| -------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| Si finisce esattamente le pagine                   | Nessuna modifica                                                                            |
| Si studia meno del previsto                        | L'argomento viene spezzato.Il residuo va al giorno di studio successivo                 |
| Si studia più del previsto                         | Le pagine extra erodono l'inizio del blocco successivo                                      |
| Si studia 0 pagine (giorno saltato)                | L'intero blocco va al giorno di studio successivo.Il ripasso del giorno saltato è vuoto |
| Blocco spezzato                                    | Ogni parte ha ripassi indipendenti calcolati dalla propria data di studio                   |
| Il blocco successivo viene assorbito completamente | Scompare dal piano e si passa al seguente                                                   |
| Si aggiunge il sabato/domenica a piano in corso    | Il passato è congelato e solo il futuro viene rigenerato                                    |
| Si aggiunge un giorno libero                       | Quel giorno scompare dalla sequenza e il futuro si riorganizza                              |
| Si rimuove un festivo                              | Quel giorno rientra e il futuro si compatta                                                 |
| Quarto ripasso abilitato                           | Tutti i ripassi già pianificati vengono aggiornati                                          |
| Argomenti in eccesso (overflow)                    | Coda separata. Si recuperano se si aggiungono giorni                                        |
| Ripasso non completato                             | Ignorato. Nessun recupero previsto                                                          |

---

## **Punti deboli del sistema**

Il sistema presuppone che l'utente conosca la difficoltà degli argomenti che deve studiare. Questo è ovviamente impossibile.


## Problema
Se gli ultimi giorni di teoria sono molto vicini al giorno dell'esame e gli argomenti da studiare hanno una complessità elevata, si rischia di farli slittare oltre la data dell'esame perché i ritardi si accumulano ma non ci sono giorni sufficienti per recuperarli. 

### Soluzione
Bisogna finire la teoria almeno 7 giorni prima dell'esame, quindi quando necessario, è **obbligatorio** introdurre dei giorni di studio aggiuntivi (di default sabato e domenica) in modo da recuperare. 
Il sistema deve notificare il problema e suggerire l'introduzione di giorni di studio aggiuntivi. 
L'utente deve scegliere se accettare la modifica oppure no.
