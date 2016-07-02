---
ID: 10
post_title: 'Beskyttet: Sådan optimerer du hastigheden på PHP med Cachegrind'
author: kristian@justiversen.dk
post_date: 2016-06-30 09:50:09
post_excerpt: ""
layout: post
permalink: >
  http://kristianjust.dk/hastighedsoptimering-af-php-med-cachegrind/
published: true
---
[contentblock id=hastighedsoptimering_artikelserie]

Den bedste hastighed får du ved at skrive god kode, når du skriver den. Alligevel kan det være nødvendigt at skulle optimere på det efterfølgende - det hænder i hvert fald for mig :-) Koden vokser, brugeraktiviteten vokser og der er pludselig en side, der er langsom, selvom man synes, man tog højde for det hele.

Men hvordan ved du, hvad der gør siden langsom?

Ved at benytte en <em>profiler</em>, kan du få det <strong>komplette overblik</strong> over ydelsen af din kode - hvor lang tid hver funktion tager om eksekvering, hvor mange gange en funktion kaldes, <em>callstack</em> for hver funktion med mere.

Du kan også benytte et meget omfattende <em>var_dump(microtime(true));</em> setup, men det vil jeg ikke anbefale dig ;-)

Jeg vil i det følgende vise, hvordan jeg bruger <a href="https://xdebug.org/docs/profiler"><em>Xdebugs Profiler</em></a> og et <em>Cachegrind-</em>analyseprogram til at finde ud af, hvad der skaber flaskehalse i et PHP script.
<h3>Sådan køres Xdebugs Profiler</h3>
<em>Xdebugs Profiler </em>køres samtidig med eksekvering af dit PHP script og outputter dens rapport i en <em>Cachegrind</em>-fil.

Aktiver Xdebug i din PHP installation og sæt PHP-variablen <em>xdebug.profiler_enable_trigger </em>til 1. Angiv hvor <em>Cachegrind</em>-filerne skal oprettes med PHP-variablen <em>xdebug.profiler_output_dir.</em>

Start nu dit script med GET-parametret XDEBUG_PROFILE. Fx http://localhost/ditProjekt/index.php?XDEBUG_PROFILE.

Der ligger efterfølgende en <em>Cachegrind-</em>fil klar i den valgte sti, som er klar til at blive importeret i dit yndlings <em>Cachegrind</em>-analyseprogram.
<h3>Analyse i WinCacheGrind</h3>
Der er mange forskellige analyseprogrammer at vælge imellem. Da jeg er på Windows faldt jeg over <a href="https://github.com/ceefour/wincachegrind">WinCacheGrind</a>, som jeg er meget tilfreds med. Andre er glade for <a href="https://kcachegrind.github.io/html/Home.html">KCacheGrind</a>, <a href="https://sourceforge.net/projects/qcachegrindwin/">QCacheGrind</a> eller <a href="https://github.com/jokkedk/webgrind">Webgrind</a>.

Hvis jeg åbner min <em>Cachegrind</em>-fil i <em>WinCacheGrind</em>, vælger <em>"{main}"</em> i venstre-menuen og tabben <em>"Overall" </em>ser jeg en komplet liste over alle funktionskald, hvor lang tid de gennemsnitligt har taget om at blive eksekveret, og en samlet eksekveringstid for alle de gange, den pågældende funktion er blevet kaldt.

<img src="http://kristianjust.dk/wp-content/uploads/2016/06/cachegrind-overall-1024x536.png" alt="WinCacheGrind Overall - Xdebug profiler" width="750" height="393" class="alignnone size-large wp-image-14" />

Brug 1 minut på at kigge på listen over funktioner, inden du læser videre.

Klar igen?

Listen viser alle funktionskald. Alle. Det vil også sige funktioner, der er kaldt af andre funktioner. De øverste linjer vil af samme grund oftest ikke være særligt interessante, da det blot er overordnede funktioner, der initialiserer dit projekt. For at finde de rigtige syndere, skal der dykkes lidt ned på listen.

Trævisningen i venstre side giver mulighed for kun at vise funktionskald under et andet givent funktionskald. Min erfaring siger mig, at det dog kan være en del sværere at bevare overblikket - men det kan være, det fungerer for dig.

<em>WinCacheGrind</em> viser som standard en eksekveringstid i millisekunder. Mange andre <em>Cachegrind</em>-analyseprogrammer har valgt at vise eksekveringstiden som en procent af den samlede eksekveringstid for PHP scriptet. Dette skyldes, at man ikke kan regne med millisekunderne i rapporten. Dels kører du højest sandsynligt <em>profileren</em> på din udviklingsmaskine fremfor produktionsserver, dels gør <em>Xdebug Profileringen</em> at eksekveringstiden er højere end normalt.

Indstil WinCacheGrind til at benytte procent fremfor millisekunder ved at trykke på "sum"-ikonet i <em>toolbaren</em>.

Den samlede eksekveringstid for scriptet står under <em>"Cumulative time"</em> i <em>"{main}"</em>-vinduet.
<h3>Find de suboptimale funktioner</h3>
Led efter de (selvstændige) funktioner på listen, der optager en betydelig procent af den samlede eksekveringstid.

Se også i kolonnen <em>"Calls"</em> om det giver mening, at funktionen skulle være kaldt, så mange gange, som den er. Måske kaldes funktionen rekursivt ved en fejl.

Med din viden om projektet og kendskab til koden kan du nemt vurdere, hvor lang eksekveringstid der er rimelig for en given funktion.
<h3>Filtrer på includes</h3>
<img src="http://kristianjust.dk/wp-content/uploads/2016/06/cachegrind-includes.png" alt="WinCacheGrind - includes" width="531" height="410" class="alignnone size-full wp-image-15" />

En anden fremgangsmåde er at filtrere på <em>includes/requires</em>.

I ovenstående eksempel ses et række standard WordPress filer, samt nogle af lidt mere interessant karakter: <em>dashboard-top-view.php, activity-loop.php </em>og<em> entry.php.</em>

Jeg ved at<em> activity-loop.php</em> kalder <em>entry.php</em> - så måske kan loopet begrænses til færre iterationer, eller jeg kan undersøge om koden i <em>entry.php</em> kan optimeres.

Nu fandt jeg i hvert fald på under 3 minutter ud af, at <em>entry.php </em>skal undersøges nærmere, da den kan være en mulig flaskehals.
<h3>Udfordringer ved call_user_func</h3>
Har du mangler funktioner, der kaldes igennem <em>call_user_func</em> mister du overblikket i <em>WinCacheGrind</em>.

Har du lavet meget tynde funktioner, er det dog som regel ikke et reelt problem, da du i stedet kan se på en relateret funktion.

Ellers må du markere <em>call_user_func-</em>linjen i <em>WinCacheGrind</em> og gennemtrævle <em>callstacken.</em>
<h3>Jeg synes det hele ser fint ud?!</h3>
Hvis ingen funktioner ser ud til at have en urimelig lang eksekveringstid, kan din langsomme eksekveringstid for PHP scriptet skyldes følgende
<ol>
 	<li>Mange bække små: alle dine funktioner tager lidt for lang tid :-) Måske du er flittig til at lave MySQL kald (dette vil i så fald også fremgå i <span style="text-decoration: underline;"><em>WinCacheGrind</em></span>, da du kan se <em>query</em>-funktionens samlede eksekveringstid og samlede antal kald)</li>
 	<li>Det er ikke dit PHP scripts skyld: din server er langsom, din PHP konfiguration er ikke optimal el.lign.</li>
 	<li>Dine "nødvendige kernefunktioner" er simpelthen bare tunge. Formålet med dit PHP script kan jo være tung processering af et eller andet. Der er grænser for, hvor meget der kan optimeres (ellers ville alle sider kunne køre på en one.com server - det kan de ikke..).</li>
</ol>
<h3>Var det det?</h3>
<strong></strong>Ja!

Overblikket over dine funktionskald i <em>WinCacheGrind</em>, lidt sund fornuft og eftertanke gør, at du meget nemt kan overtage kontrollen over dine store PHP projekter igen.