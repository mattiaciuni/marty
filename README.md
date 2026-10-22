# Per te, Marty

Una lettera privata e cinematografica, costruita per essere ascoltata dal telefono. La poesia nel sito è il testo fornito da te. La registrazione che hai fornito è stata aggiunta in `public/audio/marty.m4a`.

## Avvio locale

Richiede Node.js 20.19+ o 22.12+.

```sh
npm install
npm run dev
```

Per controllo tipi e build di produzione: `npm run typecheck` e `npm run build`. I file statici generati in `dist/` sono pronti per Vercel.

## Il poema e le scene

Il testo che hai fornito è in [`src/data/poem.ts`](src/data/poem.ts), mantenuto riga per riga e senza riscritture. Ogni strofa conserva i propri a capo e ha un identificatore stabile per la sincronizzazione. I cambi di luce sono soltanto quelli esplicitamente indicati nei marcatori `everyday`, `midnight`, `vulnerable`, `future` e `final`; le strofe senza marcatore non fanno cambiare la scena.

## Aggiungere la registrazione

La registrazione fornita è in `public/audio/marty.m4a` e viene caricata direttamente nel player; non è stata convertita né alterata. Se registri una nuova versione MP3, puoi sostituire il file con `public/audio/marty.mp3` e aggiornare l'attributo `src` in `src/App.tsx`. Il file M4A attuale è incluso e compatibile con Chrome/Safari moderni. Non c'è audio dimostrativo né autoplay. Il player si abilita soltanto dopo un tocco e mostra con delicatezza quando il file non è ancora presente. Il player usa l'elemento Audio HTML, gestisce caricamento/errore, seek, velocità, replay, visualizzazione spettrale da Web Audio e Media Session (comandi cuffie/schermata bloccata quando il browser li supporta). Safari può sospendere la riproduzione in background secondo le proprie impostazioni energetiche.

## Sincronizzare parole e voce

Il testo e la registrazione presenti non bastano a dichiarare timestamp esatti: il file [`src/data/poem-timing.json`](src/data/poem-timing.json) resta vuoto finché non si allinea il testo alla voce reale. La lettera non scompare mai; finché non importi i tempi, resta tutta leggibile e il player continua a funzionare, senza fingere l'effetto di scrittura sincronizzata.

### Generare tempi locali con WhisperX

Lo script [`scripts/align_poem.py`](scripts/align_poem.py) usa il poema sorgente senza duplicarlo, non esegue riconoscimento vocale e non invia l'audio a servizi esterni. Forza l'allineamento in italiano sul file reale; produce inizi/fini per parola, riferimenti esatti a strofa/verso e indici UTF-16 del testo. I segni sono rivedibili nel JSON e importabili nell'editor di sviluppo.

Serve Python 3.10–3.12 in un ambiente locale. WhisperX e PyTorch occupano molto spazio; si può usare CPU, senza installare CUDA. Lo script usa WhisperX offline per trovare prima i segmenti vocali nella registrazione, poi allinea il testo riconosciuto alla voce e abbina in ordine solo le parole esattamente corrispondenti al poema. Installalo nell'ambiente scelto (il progetto upstream documenta Windows e l'allineatore italiano):

```powershell
py -3.11 -m venv .venv-align
.venv-align\\Scripts\\Activate.ps1
python -m pip install --upgrade pip
python -m pip install whisperx
python scripts/align_poem.py --device cpu --asr-model small
```

Il primo uso richiede internet per installare pacchetti e scaricare/cacheare il modello di allineamento italiano; la successiva elaborazione di audio e testo è locale. L'allineamento forzato presuppone che il testo corrisponda a ciò che è stato davvero detto: pause, parole omesse, ripetizioni o pronunce insolite possono generare errori. Lo script li lascia non allineati (mai inventati/interpolati) e li segnala nel campo `review`.

1. Verifica che la registrazione reale sia `public/audio/marty.m4a`; sostituisci il file se l'audio definitivo è diverso.
2. Dalla radice del repository, esegui il comando di allineamento sopra. Per un altro percorso usa `--audio percorso/file.m4a`; per non sovrascrivere il JSON versione usa `--output percorso/revisione.json`. Se la registrazione è rumorosa/prolissa, scegli un modello più grande con `--asr-model medium` (maggiore uso di RAM/tempo, sempre in locale).
3. Controlla `review.asrSegments` e `review.missingWords`. Le parole sono incluse soltanto se il testo riconosciuto corrisponde esattamente, in sequenza, a quello del poema. Una parola mancante o diversa non riceve un tempo inventato e non sposta i successivi. Ascolta e verifica tutti i confini: anche una parola apparentemente allineata può avere tempi errati. L'output resta `whisperx-local-review-required` finché non viene rivisto.
4. In `npm run dev`, apri la lettera con `?sync` nella URL locale e importa il JSON da **Importa JSON**. Usa il player, seleziona i versi/parole e correggi inizio/fine, poi esporta. Per la produzione, copia il JSON revisionato in `src/data/poem-timing.json` e riavvia la build; rimuovi dal file l'eventuale campo `review` solo se la revisione è completa.
5. `start`/`end` sono secondi audio. `startIndex`/`endIndex` sono intervalli half-open UTF-16, uguali a `String.slice()`, e `text` deve combaciare esattamente col verso. È incluso un piccolo margine di fine per scrivere la punteggiatura assieme alla parola seguente.

MFA (Montreal Forced Aligner) è una valida alternativa locale su Windows con modello acustico e dizionario italiani `italian_cv`; produce TextGrid/JSON che richiedono conversione nello schema del progetto. Non usare timestamp generati distribuendo la durata del file sulle parole: non sono sincronizzazione. Fonti: [WhisperX](https://github.com/m-bain/whisperx), [installazione MFA](https://montreal-forced-aligner.readthedocs.io/en/latest/installation.html), [modello acustico italiano MFA](https://mfa-models.readthedocs.io/en/latest/acoustic/Italian/Italian%20CV%20acoustic%20model%20v2_0_0.html).

Quando ci sono tempi parola-per-parola, la lettera ricalcola la porzione scritta dall'orologio reale `audio.currentTime`: pause e resume fermano/riprendono insieme, i seek avanti/indietro ricostruiscono il testo senza duplicarlo. L'auto-follow accompagna la riga mentre si scrive; un gesto di scroll sospende il follow e il pulsante **Torna alla scrittura** lo ripristina. Senza tempi parola-per-parola, non parte un timer di scrittura fittizio.

## Personalizzare apertura e chiusura

Modifica l'apertura in [`src/components/WelcomeScene.tsx`](src/components/WelcomeScene.tsx), i tre momenti in [`src/components/IntroductionSequence.tsx`](src/components/IntroductionSequence.tsx) e il foglio/signature in [`src/components/LoveLetter.tsx`](src/components/LoveLetter.tsx). Il poema completo resta in [`src/data/poem.ts`](src/data/poem.ts).

## Vercel

Importa il repository su Vercel e lascia rilevare a Vercel Vite, oppure configura:

- Framework preset: **Vite**
- Build command: `npm run build`
- Output directory: `dist`
- Install command: `npm install`

Non sono necessari segreti o variabili d'ambiente. Verifica il deploy con la registrazione presente e l'URL HTTPS (necessario per una compatibilità più affidabile con Web Audio e Media Session).

## Privacy e protezione dell'accesso

L'app non usa analytics, pubblicità, cookie, account, embed o chiamate a servizi applicativi di terze parti. Le sole preferenze salvate sono la velocità di riproduzione in `localStorage`. I font editoriali si caricano dai Google Fonts dichiarati in `index.html`; ciò comporta quella richiesta al servizio dei font. Per una visita completamente senza richieste esterne, scarica Instrument Serif e DM Sans, servili localmente e rimuovi i link/preconnect in `index.html`.

Il sito e i suoi asset, compresa la voce e la lettera, sono pubblici a chiunque abbia l'URL se distribuiti normalmente su Vercel. Un cancello password solo client è facilmente aggirabile e **non protegge** gli asset. Per un vero accesso privato abilita la protezione a livello hosting (Vercel Deployment Protection / password protection del progetto o accesso del team) oppure un proxy/server con autenticazione prima di servire qualsiasi risorsa. Scegli e configura il piano/controllo hosting che protegge l'URL di produzione e gli asset statici, non solo la schermata React.
#   a t e m a r t i n a  
 