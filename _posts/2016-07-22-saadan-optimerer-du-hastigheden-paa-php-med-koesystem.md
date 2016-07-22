---
ID: 45
post_title: >
  Sådan optimerer du hastigheden på PHP
  med køsystem
author: kristian@justiversen.dk
post_date: 2016-07-22 08:06:31
post_excerpt: ""
layout: post
permalink: >
  http://kristianjust.dk/saadan-optimerer-du-hastigheden-paa-php-med-koesystem/
published: true
---
Mange opgaver kan udføres, uden brugeren af din webapplikation behøver vente imens. Låser du brugeren, mens der fx afsendes e-mail, så giver jeg dig nu opskriften på, hvordan <strong>du kan gøre det smartere</strong>.

Med det rigtige setup, kan du give en sømløs brugeroplevelse - uden at forsinke udførelsen af opgaver.
<h3>Køsystem</h3>
Et køsystem er et system, der kan sætte en række opgaver i kø til senere udførelse.

Grundprincippet i forbindelse med hastighedsoptimering er, at alle opgaver, der ikke nødvendigvis <em>skal</em> udføres med det samme, lægges i køen.

En forespørgsel til fx en SMTP server kan hurtigt have en svartid på 50 ms + tiden til processering. Dette sparer du stortset brugerne helt for (der er dog en lille tidsomkostning forbundet med at lægge i kø, men den er minimal).

Din server vil selvfølgelig skulle bruge tiden på at udføre opgaven efterfølgende, men din bruger bliver ikke belastet.
<h3>Konceptet: simpelt køsystem</h3>
Du kan lave dit eget simple køsystem blot med en databasetabel og en cronjob, der løbende lytter på, om der er nye ikke-udførte opgaver i køen:
<pre>id     | queued_at           | executed_at         | function          | data
5      | 20XX-07-21 09:59:00 | null                | send_email        | {recipient: 'receiver@justiversen.dk', headline: '..', body: '..'}
4      | 20XX-07-21 09:57:50 | 20XX-07-21 09:57:52 | send_email        | {recipient: 'receiver@justiversen.dk', headline: '..', body: '..'}
3      | 20XX-07-21 09:57:48 | 20XX-07-21 09:57:52 | send_email        | {recipient: 'receiver@justiversen.dk', headline: '..', body: '..'}
2      | 20XX-07-21 09:56:34 | 20XX-07-21 09:56:38 | send_email        | {recipient: 'receiver@justiversen.dk', headline: '..', body: '..'}
1      | 20XX-07-21 09:53:10 | 20XX-07-21 09:53:12 | send_email        | {recipient: 'receiver@justiversen.dk', headline: '..', body: '..'}
</pre>
Cronjobbet kalder blot funktionen fra feltet <em>function </em>med de givne data, der ligger i feltet <em>data</em>. Når jobbet er udført opdaterer den <em>executed_at</em> feltet.

Cronjobbet processerer kun opgaver, hvor <em>executed_at</em> har en null-værdi.

Det er i princippet, hvad der skal til.

I ovenstående er der dog ikke taget højde for
<ul>
 	<li>Hvordan og hvornår cronjobbet skal køre og lytte efter nye opgaver</li>
 	<li>Hvad der sker, hvis flere kører cronjobbet samtidig</li>
 	<li>Mulighed for at skalere over flere servere</li>
 	<li>Ingen notering af, om et cronjob har påbegyndt en opgave</li>
 	<li>Hvad der sker, hvis serveren går ned midt i operationen</li>
</ul>
Derudover skal du selv til at skrive koden til dit nye simple køsystem. Min anbefaling til dig er derfor at vælge en af de eksisterende løsninger.

<a href="http://kr.github.io/beanstalkd/">Beanstalkd</a> (<a href="https://github.com/pda/pheanstalk">pheanstalk</a> til PHP) er et modent og stabilt køsystem, med en veludbygget API og udmærket dokumentation.

<span>Som en del af Amazons populære serverløsninger findes også <a href="https://aws.amazon.com/sqs/">Amazon SQS</a>, der løfter samme opgave. Det er let at implementere med færdigskrevne PHP-pakker, ligesom der findes færdige pakker til de andre løsninger derude.</span>

<span>Løsningerne er i bund og grund ikke så forskellige, så find en der passer godt til dit setup. Jeg er sikker på, at du og dine brugere bliver glade for dit nye køsystem.</span>