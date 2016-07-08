---
ID: 24
post_title: >
  Sådan optimerer du hastigheden på PHP
  med memory caching
author: kristian@justiversen.dk
post_date: 2016-07-08 07:47:41
post_excerpt: ""
layout: post
permalink: >
  http://kristianjust.dk/saadan-optimerer-du-hastigheden-paa-php-med-memory-caching/
published: true
---
[contentblock id=hastighedsoptimering_artikelserie]
<h3>Problemet</h3>
Hver gang PHP skal lave beregninger eller vente på et tredjepartsprogram (kald til database, API kald over netværk etc.) tager det tid.

Dette problem kan blive forværret ved unødvendige <em>gentagelser</em> i koden.

Tunge beregninger, lang ventetid på tredjepartsprogrammer eller mange gentagelser kan udgøre en <em>flaskehals</em> i eksekveringen af din applikation. Du kan finde flaskehalse i din kode igennem eksempelvis <a href="http://kristianjust.dk/hastighedsoptimering-af-php-med-cachegrind/">en <em>cachegrind</em> analyse</a>.

Hvad kan der gøres ved en sådan flaskehals, spørger du? Jeg har svaret :-)
<h3>Caching på funktionsniveau to the rescue</h3>
En ofte anvendt løsning på hastighedsproblemer er <em>caching</em>.

Mange beregninger eller kald til tredjepartsprogrammer ændrer ikke resultat fra minut til minut, og at lade alle brugere foretage de samme beregninger igen og igen er håbløs spild af <span style="text-decoration: underline;">resurser på din server</span> og <span style="text-decoration: underline;">tid for brugerne</span>.

I stedet kan resultatet af en beregning gemmes ved første eksekvering, og det gemte resultat kan serveres på de efterfølgende eksekveringer direkte fra hukommelsen eller en fil.

En god fremgangsmåde er at <em>cache </em>returværdien af en eller flere funktioner. Denne struktur gør, at man bevarer overblikket over, hvor <em>caching</em> er implementeret.

Afhængig af hvilken <em>cache service</em> der benyttes, vil en <em>cached</em> værdi blive gemt med et navn, fx "tung_funktion_user_3466", og med en værdi svarende til returværdien af den cachede funktion.

<em><strong>Eksempel på funktion uden cache</strong></em>
<pre class="prettyprint">&lt;?php

/**
 * Tung funktion
 *
 * Funktion, der udfører en tung beregning for bruger $userId
 *
 * @param int $userId
 * @param array $array
 * @return int Beregnet tal for bruger
 */
public function tungFunktion($userId, $array)
{
     $calculatedValue = 0;

     foreach($array =&gt; $item) {
          // Beregning, der ændrer $calculatedValue,
          // foretages her.
     }

     return $calculatedValue;
}</pre>
<em><strong>Eksempel på funktion med cache</strong></em>
<pre class="prettyprint">&lt;?php

/**
 * Tung funktion
 *
 * Funktion, der udfører en tung beregning for bruger $userId
 *
 * @param int $userId
 * @param array $array
 * @return int Beregnet tal for bruger
 */
public function tungFunktion($userId, $array)
{
     if ($cached = $this-&gt;cache-&gt;get('tung_funktion_user_' . $userId))
          return $cached;
     $calculatedValue = 0;

     foreach($array =&gt; $item) {
          // Beregning, der ændrer $calculatedValue,
          // foretages her.
     }

     $this-&gt;cache-&gt;set('tung_funktion_user_' . $userId, $calculatedValue);

     return $calculatedValue;
}</pre>
Funktionerne <em>$this-&gt;cache-&gt;get()</em> og <em>$this-&gt;cache-&gt;set() </em>henviser til den pågældende <em>cache drivers</em> funktioner til hhv. at hente og gemme i <em>cachen</em> - men disse funktioner kan hedde hvad som helst, afhængig af valg af <em>framework</em>, <em>cache driver</em> osv.

Som det ses, gør implementering af <em>caching</em>, at funktionens resursekrævende kode slet ikke eksekveres, men simpelthen springes over.
<h3>Hvilken caching skal jeg vælge?</h3>
Først og fremmest bør du tage stilling til, om du vil bruge en <em>filcache</em> eller <em>in-memory cache</em>. Da normal <em>filcache </em>næsten altid vil tabe på hastighed til <em>in-memory cache</em>, bør man som udgangspunkt vælge <em>in-memory caching</em>, hvis du har mulighed for det.

Jeg vil anbefale at benytte <a href="http://redis.io/" style="font-style: italic;">Redis</a>, som er en solid <em>in-memory caching service, </em>der er rig på funktioner og har mange forskellige datatyper. Det er sikkert, at <em>Redis</em> vil<em> </em>overtage mere og mere marked fra det ellers etablerede <em>Memcached.</em>

Hvor <em>Memcached</em> er at finde i alle de kendte <em>frameworks</em>, er <em>Redis</em> stadig kun tilgængeligt i nogle få, men let at implementere med eksempelvis <a href="https://packagist.org/packages/predis/predis">denne composer pakke</a>.

P.S. <em>Filcaching</em> kan også opsættes til at benytte en <a href="http://www.vanemery.com/Linux/Ramdisk/ramdisk.html">RAM disk</a>, ligesom det bør bemærkes, at MySQL også vil forsøge at <em>cache </em>resultater af forespørgsler i RAM. <a href="http://we-love-php.blogspot.dk/2013/02/php-caching-shm-apc-memcache-mysql-file-cache.html">Der kan være situationer, hvor det er hurtigere ikke at benytte en </a><em>caching service, </em>selvom det må anses for at høre til sjældenhederne.
<h3>Sådan vælger du, hvilke funktioner, du skal cache</h3>
Du skal udvælge funktioner, som <strong>kaldes ofte</strong> og <strong>ændrer resultat sjældent.</strong>

Eksempler
<ul>
 	<li>Funktion, der henter basisinformation om et produkt fra database</li>
 	<li>Funktion, der henter stamdata for en bruger</li>
 	<li>Funktion, der henter noget statistik</li>
</ul>
Alle funktioner kan <em>caches</em>, men vælger du funktioner, hvor resultatet ikke får lov at ligge i <em>cachen</em> særligt lang tid af gangen, inden <em>cachen</em> skal slettes (invalideres) og værdien beregnes på ny, så giver <em>caching</em> på funktionen ikke mening, og det bliver blot en ekstra kilde til eventuelle fejl.
<h3>Fejl 1: afhængighed af cache</h3>
<em>Caching</em> skal ses som et ekstra <span style="text-decoration: underline;">lag</span>, der skal kunne fjernes, uden at funktionaliteten i din applikation går i stykker. Der skal ikke være nogen funktioner, der er afhængige af en <em>cached</em> værdi.

Ligeledes skal du ikke bruge <em>caching</em> til at komme udenom dårligt performende kode, så en sådan suboptimal kode bør rettes i stedet. Ellers ender du med at bruge <em>caching</em> som et plastrer, der holder hele din kode sammen :-)

Din applikation kan i drift alligevel være afhængig af <em>caching</em>, pga. det høje load, som din server ellers vil blive udsat for. Det er helt berettiget, da det jo er en af grundene til, at man ønsker at implementere <em>caching </em>i første omgang. Hvis <em>caching systemet</em> går ned i drift, bliver det en opgave for server administratoren at holde skibet flydende.
<h3>Fejl 2: manglende cache invalidering</h3>
<blockquote>There are only two hard things in Computer Science: cache invalidation and naming things.

- Phil Karlton</blockquote>
Det gamle citat lyver ikke.

Når gemning og hentning af cache værdier er implementeret, er du kun halvvejs færdig med implementering af <em>caching</em>.

Nu ligger den store opgave foran dig med at <em>invalidere cachen</em>, når værdierne ikke længere er gældende.

I praksis gøres dette ved at slette <em>cache</em> værdierne på given tidspunkter. Har du eksempelvis <em>cached</em> oplysningerne om en bruger, skal disse værdier slettes, når brugeren retter i sin profil.

Det kan være svært at nå hele vejen rundt, men det er ikke desto mindre meget vigtigt, at du gør en indsats for det.
<h3>Nu er du klar</h3>
Dette var de grundlæggende principper i <em>caching</em> af værdier på funktionsniveau. Nu er det tid til at undersøge mulighederne i dit yndlings <em>framework, </em>og komme i gang med at få optimeret hastigheden på din applikation :-)

[contentblock id=2 img=gcb.png]