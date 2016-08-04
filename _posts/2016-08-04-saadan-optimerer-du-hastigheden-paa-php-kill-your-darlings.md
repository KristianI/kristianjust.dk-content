---
ID: 54
post_title: 'Sådan optimerer du hastigheden på PHP &#8211; kill your darlings'
author: kristian@justiversen.dk
post_date: 2016-08-04 09:00:32
post_excerpt: ""
layout: post
permalink: >
  http://kristianjust.dk/saadan-optimerer-du-hastigheden-paa-php-kill-your-darlings/
published: true
---
[contentblock id=hastighedsoptimering_artikelserie]

Hastighedsproblemer kan komme fra suboptimal kode, ventetid på 3. parts API osv.

Men en ofte overset årsag til hastighedsproblemer er funktioner, som man vurderer som "nødvendige" for sit program eller site. En oprydning i disse, kan hjælpe både på <strong>performance</strong> og på <strong>omkostninger til vedligeholdelse</strong> af kode.

Jeg oplever ofte, at funktioner bliver udnævnt som kernefunktioner - uundværlige funktioner - på baggrund af mavefornemmelser eller uafprøvede teser.

Du har måske selv stødt på sådanne funktioner, hvor det ikke er programmets brugere, der har udnævnt det som uundværligt, men nærmere programmets forfatter?

Eller måske det er funktioner, som brugere har efterspurgt, men hvor det alligevel ikke vil have negativ indflydelse af fjerne.

Det kan også være funktioner, som var aktuelle engang, men som på grund af andre ændringer, ikke længere er vigtige.
<h3>Simpel metode til at identificere funktioner, der kan smides ud</h3>
Gør dig selv en tjeneste at lave en liste over de 10 mest tunge funktioner i dit program eller på dit site. Stil til hver funktion følgende spørgsmål
<ul>
 	<li><strong>Mister vi brugere</strong> ved at fjerne denne funktion?</li>
 	<li>Er dette min egen <strong>kæphest</strong>, eller har den vist synlige resultater for vores forretning?</li>
 	<li>Er den stadig <strong>aktuel og vigtig</strong> i dag, som vores program ser ud nu?</li>
</ul>
Jeg er sikker på, at du kan finde vigtige performanceforbedringer med disse simple spørgsmål.
<h3>Overtal chefen</h3>
Er du i et arbejdstager forhold kan det blive en udfordring at overtale din chef. Din chef får penge for at træffe beslutningerne, og det er ikke altid nemt at komme fra gulvet og påvirke hans eller hendes arbejde.

Især i de tilfælde, hvor den funktion, du ønsker at droppe, har været din chefs ide oprindeligt, kan det give problemer.

Her er 4 gode argumenter, du kan benytte i kampen:
<ul>
 	<li>Hvis funktionen har været din chefs ide eller kæphest, så sørg for at <strong>ros</strong> funktionens virke tidligere. Fortæl hvor meget funktionen har gjort for sitet og brugerne, men at den ikke længere er vigtig, og at det fornuftige nu må være at lade funktionen fjerne.</li>
 	<li><strong>Prik til chefens økonomiske side</strong>: en funktion mindre er en mindre ting at vedligeholde. Forklar chefen, at "som han/hun jo ved" koster alle funktioner penge at holde kørende, da der vil være rettelser ved opdatering af framework, opdatering af PHP osv.</li>
 	<li><strong>Lad chefen få ideen</strong>: <strong></strong>fjernelse af funktionen giver eventuelt plads til nye funktioner, og fjernelse af netop denne funktion underbygger den vision/strategi, som chefen har for programmets eller sitets hastighed (du kender din chef og hvad denne står for, så find en af chefens visioner og forbind din beslutning om fjernelse af funktionen til denne). Du er nu den lydige medarbejder, og din chef kan gå til sin chef og fortælle, hvordan strategien fortsat udføres.</li>
 	<li><strong>Slå på stoltheden</strong>: fortæl din chef henkastet af denne funktion ikke er vildt moderne, og at den i øvrigt trækker hastigheden ned for hele sitet. Du vil snart modtage en ordre om at få den fjernet.</li>
</ul>
Alle chefer er forskellige, og du ved, hvad der virker for din. De gode chefer vil allerede være lydhøre overfor, hvad du har at sige, og du slipper helt for at skulle tale i et andet sprog, end du plejer.
<h3>Jagten kan gå i gang</h3>
Ovenstående giver dig forhåbentligt en idé til at finde hastighedsforbedringer, som ligger udenfor kategorien "tekniske forbedringer".

God jagt på undværlige "vigtige" funktioner!

[contentblock id=2 img=gcb.png]