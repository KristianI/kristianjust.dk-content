---
ID: 57
post_title: >
  Sådan optimerer du hastigheden på
  MySQL med MEMORY tabeller
author: kristian@justiversen.dk
post_date: 2016-09-09 20:03:59
post_excerpt: ""
layout: post
permalink: >
  http://kristianjust.dk/saadan-optimerer-du-hastigheden-paa-mysql-med-memory-tabeller/
published: true
---
PHP og MySQL hænger tæt sammen. Ønsker man at optimere hastigheden i sin PHP applikation, vil det være oplagt at forsøge at minimere kald til MySQL databasen eller at gøre eksekveringen af MySQL forespørgslerne hurtigere.

De mest almindelige MySQL tabeltyper er <em>MyISAM</em> og <em>InnoDB</em>. Men MySQL tilbyder også andre mindre populære tabeltyper - heriblandt <strong><em>MEMORY </em></strong>tabeltypen.

Med MEMORY gemmes tabeldataene i hukommelsen i stedet for på et drev. At hente data direkte fra hukommelsen giver åbenlyse hastighedsfordele, men MEMORY tabellerne er ikke egnede i alle situationer - faktisk meget få.

Herunder kan du finde de væsentligste fordele, ulemper og egnede situationer for MEMORY tabellerne.
<h3>Fordele ved MEMORY tabeller</h3>
<ul>
 	<li>Data gemmes og hentes direkte fra hukommelsen - der er næsten intet drev trafik. <strong>Læsning går derfor meget hurtig!</strong></li>
 	<li>Du kan oprette MEMORY tabeller på en almindelig MySQL server (det er ikke nødvendigt at installere MySQL Cluster).</li>
</ul>
<h3>Ulemper ved MEMORY tabeller</h3>
<ul>
 	<li>Dataene er ikke vedholdende. Data vil forsvinde ved genstart af server eller nedbrud på MySQL servicen.</li>
 	<li>Der skal være plads til dataene i hukommelsen, ellers vil den benytte et drev som virtuel hukommelse, hvilket er langsommere.</li>
 	<li>Tilbyder kun <em>locking </em>af hele tabellen, og ikke en enkelt række. Det betyder, at ved hvert INSERT/UPDATE/DELETE af data i tabellen, skal andre processer vente på, at den pågældende process er færdig med at opdatere, inden der er adgang til tabellen igen. Dette tilfører en del ventetid, hvis der er flere processer om samme tabel.</li>
 	<li>Understøtter ikke MySQL <a href="https://dev.mysql.com/doc/refman/5.7/en/sql-syntax-transactions.html"><em>transactions</em></a>.</li>
</ul>
<h3>Use-cases</h3>
De mange ulemper gør situationerne, hvor MEMORY tabellerne kan bruges, temmeligt begrænset.

I følgende tilfælde vil MEMORY tabeller være <span style="text-decoration: underline;">ubrugelige</span>:
<ul>
 	<li>Din tabel indeholder meget data.</li>
 	<li>Din server har meget lidt hukommelse tilgængeligt.</li>
 	<li>Du både skriver (INSERT/UPDATE/DELETE) og læser (SELECT) meget fra tabellen.</li>
 	<li>Flere processer læser og skriver engang imellem lidt til tabellen.</li>
 	<li>Dataene er vigtige, og må ikke gå tabt ved genstart af MySQL.</li>
</ul>
Tilbage er følgende egnede use-cases:
<ul>
 	<li>Tabel med data, der nemt kan genopbygges (fx statistik eller cache data)</li>
 	<li>Tabel med data, der skal bruges som en mellemregning i en analyse, og hvor det kun er det endelige resultat, der er vigtigt.</li>
</ul>
Selvom use-cases og fordele er få, vil der være udtrykkelige hastighedsfordele ved at benytte MEMORY tabeller i de få situationer, hvor de faktisk kan bruges.
<h3>Indexes</h3>
MEMORY tabeller understøtter både B-Tree og Hash indexes.

Hash indexes er meget hurtige ved eksakt sammenligning (fx <em>WHERE columnA = 'ABC'</em>), men understøtter ikke range søgninger (fx <em>WHERE columnA LIKE 'AB%'</em>)

B-Tree understøtter ranges, men er langsommere.

MEMORY tabeller bruger som standard Hash indexes. Ønsker du at benytte B-Tree, skal du angive dette, når du opretter tabellen med <em>CREATE TABLE</em>.
<h3>InnoDB er populært og godt alternativ</h3>
På grund af de begrænsede use-cases, vælger mange i stedet at benytte en InnoDB database med stor buffer pool eller et MySQL Cluster, der også giver mulighed for skalering over flere servere.

InnoDB følger rigtigt godt med på hastigheden (se nedenstående afsnit "Benchmarks") i forhold til MEMORY tabeller på grund af deres buffer pool, men tilbyder derudover locking på enkelte rækker i stedet for hele tabellen, vedholdende data og understøtter transactions.

Rammer du ikke en MEMORY use-case meget klart, vil du få færrest problemer ud af at vælge InnoDB i stedet.
<h3>Benchmarks - MEMORY versus InnoDB</h3>
<h4>Værktøj</h4>
Det oplagte valg af værktøj for benchmarking ville være det etablerede <em>sysbench</em>.<em> </em>Under indledende tests viser det dog grundlæggende fejl i udførelsen af MEMORY benchmarks på store mængder testdata; tests der kører for evigt etc. Fejl i forbindelse med MEMORY benchmark i sysbench bekræftes af flere tredjemænd på fora.

For at kunne bekræfte/afkræfte resultaterne af sysbench har jeg udviklet et PHP benchmark værktøj, der ved hjælp af simple MySQL forespørgsler med MySQLi driveren, kan køre en række forskellige typer af tests på InnoDB og MEMORY tabeltyperne.

Værktøjet er tilgængeligt på https://github.com/KristianI/innodb-memory-benchmarks og licenseret under MIT.

Efter fuldført benchmarking kan det bekræftes, at sysbench udviste utilregnelige resultater for MEMORY tabellen.

I det følgende er resultaterne fra innodb-memory-benchmarks værktøjet benyttet. Benchmarks er kørt på min personlige bærbare computer på en MySQL database optimeret i forhold til buffer størrelse, maksimum tabelstørrelse med mere.

Benchmarks kan køres på en hvilken som helst server, der har PHP og MySQL installeret, hvis du ønsker at dobbelttjekke nogle af testene i dit eget miljø.

Værktøjet tester ikke søgning i tabellerne, og tester derfor ikke indeksering. Alene simpel udtræk og opdatering af data testes.

<img src="http://kristianjust.dk/wp-content/uploads/2016/09/innodb-memory-benchmark-example.gif" alt="innodb-memory-benchmark-example" width="833" height="177" class="alignnone size-full wp-image-77" />
<h4>Benchmark 1: Kun læsning</h4>
For læsning er der kørt 2 slags teste:
<ul>
 	<li>Test med en tabel med 10.000 rækker data.</li>
 	<li>Test med en tabel med 100.000 rækker data.</li>
</ul>
Der er foretaget læsning af henholdsvis 1 række og alle rækker i tilfældigt skiftende rækkefølge.

<strong>Test 1A</strong>
<table>
<thead>
<tr>
<th>Rækker i tabel</th>
<th>Gentagelser</th>
<th>Tabeltype</th>
<th>Tid brugt</th>
<th>Resultat</th>
</tr>
</thead>
<tbody>
<tr>
<td>10.000</td>
<td>10.000</td>
<td>InnoDB</td>
<td>231 sekunder</td>
<td></td>
</tr>
<tr>
<td>10.000</td>
<td>10.000</td>
<td>MEMORY</td>
<td>138 sekunder</td>
<td>40 % hurtigere</td>
</tr>
</tbody>
</table>
<strong>Test 1B</strong>
<table>
<thead>
<tr>
<th>Rækker i tabel</th>
<th>Gentagelser</th>
<th>Tabeltype</th>
<th>Tid brugt</th>
<th>Resultat</th>
</tr>
</thead>
<tbody>
<tr>
<td>100.000</td>
<td>1.000</td>
<td>InnoDB</td>
<td>245 sekunder</td>
<td></td>
</tr>
<tr>
<td>100.000</td>
<td>1.000</td>
<td>MEMORY</td>
<td>138 sekunder</td>
<td>44 % hurtigere</td>
</tr>
</tbody>
</table>
&nbsp;
<h4>Benchmark 2: Læsning og skrivning, 1 process</h4>
For læsning og skrivning er der kørt 2 slags teste:
<ul>
 	<li>Test med en tabel med 10.000 rækker data.</li>
 	<li>Test med en tabel med 100.000 rækker data.</li>
</ul>
Der er tilfældigt skiftet imellem læsning og skrivning igennem alle iterationer.

Når der tilfældigt er valgt læsning, er der tilfældigt skiftet imellem <span>læsning af henholdsvis 1 række og alle rækker.</span>

Når der tilfældigt er valgt skrivning, er der tilfældigt skiftet imellem <em>UPDATE</em> og <em>INSERT.</em>

Kun 1 process har tilgået databasen under benchmarking.

<strong>Test 2A</strong>
<table>
<thead>
<tr>
<th>Rækker i tabel</th>
<th>Gentagelser</th>
<th>Tabeltype</th>
<th>Tid brugt</th>
<th>Resultat</th>
</tr>
</thead>
<tbody>
<tr>
<td>10.000</td>
<td>10.000</td>
<td>InnoDB</td>
<td>180 sekunder</td>
<td></td>
</tr>
<tr>
<td>10.000</td>
<td>10.000</td>
<td>MEMORY</td>
<td>93 sekunder</td>
<td>48 % hurtigere</td>
</tr>
</tbody>
</table>
<strong>Test 2B</strong>
<table>
<thead>
<tr>
<th>Rækker i tabel</th>
<th>Gentagelser</th>
<th>Tabeltype</th>
<th>Tid brugt</th>
<th>Resultat</th>
</tr>
</thead>
<tbody>
<tr>
<td>100.000</td>
<td>1.000</td>
<td>InnoDB</td>
<td>197 sekunder</td>
<td></td>
</tr>
<tr>
<td>100.000</td>
<td>1.000</td>
<td>MEMORY</td>
<td>71 sekunder</td>
<td>64 % hurtigere</td>
</tr>
</tbody>
</table>
&nbsp;
<h4>Benchmark 3: Læsning og skrivning, flere processer</h4>
Benchmark 3 udføres som benchmark 2 pånær, at der vil være tilsat en forstyrrende proces, som har til formål at tilgå databasen samtidig med benchmarken.

Den forstyrrende proces vil påvirke locking af tabel og rækker, som er en af de store forskelle mellem InnoDB og MEMORY, og som oftest forekommer i produktionsmiljøer med mange brugere.

<strong>Test 3A</strong>
<table>
<thead>
<tr>
<th>Rækker i tabel</th>
<th>Gentagelser</th>
<th>Tabeltype</th>
<th>Tid brugt</th>
<th>Resultat</th>
</tr>
</thead>
<tbody>
<tr>
<td>10.000</td>
<td>10.000</td>
<td>InnoDB</td>
<td>248 sekunder</td>
<td></td>
</tr>
<tr>
<td>10.000</td>
<td>10.000</td>
<td>MEMORY</td>
<td>133 sekunder</td>
<td>46 % hurtigere</td>
</tr>
</tbody>
</table>
<strong>Test 3B</strong>
<table>
<thead>
<tr>
<th>Rækker i tabel</th>
<th>Gentagelser</th>
<th>Tabeltype</th>
<th>Tid brugt</th>
<th>Resultat</th>
</tr>
</thead>
<tbody>
<tr>
<td>100.000</td>
<td>1.000</td>
<td>InnoDB</td>
<td>236 sekunder</td>
<td></td>
</tr>
<tr>
<td>100.000</td>
<td>1.000</td>
<td>MEMORY</td>
<td>107 sekunder</td>
<td>55 % hurtigere</td>
</tr>
</tbody>
</table>
<h4>Resultat af benchmarks</h4>
Benchmarkene viser, at MEMORY tabellerne vinder i alle tests.

Overraskende var det, at testene med både læsning og skrivning var hurtigere end testene, hvor der kun blev læst data.

MEMORY tabellerne tog i alle testene kun cirka halvdelen af den tid, som de tilsvarende InnoDB tests brugte, selv under påvirkning af en ekstern databaseproces, der havde til formål at påvirke locking af tabellen/rækkerne.

Under påvirkning af en forstyrrende proces blev føringen til MEMORY tabellen reduceret en smule. Ved påvirkning af flere kørende processer vil der kunne opstå yderligere reduktion. Det kan ikke udelukkes, at mange INSERT/UPDATE/DELETE vil kunne påvirke locking i så høj grad, at MEMORY ikke længere er gunstig.
<h3>Konklusion</h3>
De udførte benchmarks viser en meget stor hastighedsgevinst ved at benytte MEMORY tabeller.

Det er klart værd at undersøge, om man har nogle oplagte use-cases for MEMORY tabellen. Om ikke andet bør det tænkes ind i fremtidige projekter.