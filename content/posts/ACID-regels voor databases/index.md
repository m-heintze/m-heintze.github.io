+++
title = 'ACID-regels voor databases'
date = 2021-01-09
draft = false
description = 'Ontdek de cruciale ACID-regels voor betrouwbare databases. Leer over Atomicity, Consistency, Isolation en Durability voor foutbestendige transacties. Begrijp hun impact en implementeer ze voor robuuste systemen.'
summary = 'De ACID-regels zijn een viertal regels voor databases die er voor zorgen dat transacties betrouwbaar verlopen.'
onderwerpen = [
    'databases'
]
thumbnailAlt = 'Photo by Panumas Nikhomkhai'
+++

De ACID-regels zijn een viertal regels voor databases die er voor zorgen dat transacties betrouwbaar verlopen. Om deze regels te begrijpen moet je eerst weten wat een transactie is.

## Transacties

Een transactie bestaat uit een of meerdere queries. Een transactie is definitief als deze gecommit wordt, tot die tijd kunnen alle queries in een transactie teruggedraaid worden.

## ACID-regels

- Atomair (**A**tomicity)
- Consistent (**C**onsistency)
- Geïsoleerd (**I**solation)
- Duurzaam (**D**urability)

### Atomair (Atomicity)

Een atomaire transactie is een transactie die altijd wordt teruggedraaid als ook maar 1 van de queries faalt. Ofwel een transactie wordt alleen gecommit als alle queries succesvol waren.

### Consistent (**C**onsistency)

Consistentie zorgt ervoor dat de database altijd in een geldige staat is. Wanneer een staat geldig is wordt bepaald door de gebruiker.

**Voorbeeld**
Een restaurantketen heeft meerdere locaties. De keten houd bij hoe vaak een gerecht in totaal verkocht is en hoe vaak een gerecht per locatie verkocht is.

De onderstaande situatie is geldig omdat het totaal aantal verkochte pasta gerechten gelijk is aan de som van het aantal verkochte pasta gerechten per locatie. (10 + 15 = 25)

| locatie   | gerecht | aantal |
| --------- | ------- | ------ |
| Eindhoven | pasta   | 10     |
| Tilburg   | pasta   | 15     |

| gerecht | totaal |
| ------- | ------ |
| pasta   | 25     |

De onderstaande situatie zou ongeldig zijn. Het totaal aantal verkochte pasta gerechten is niet gelijk aan de som van het aantal verkochte pasta gerechten per locatie. (10 + 15 &NotEqual; 30)

| locatie   | gerecht | aantal |
| --------- | ------- | ------ |
| Eindhoven | pasta   | 10     |
| Tilburg   | pasta   | 15     |

| gerecht | totaal |
| ------- | ------ |
| pasta   | 30     |

#### Read inconsistentie

Wat hierboven beschreven is noem ik data consistentie. Als je een database horizontaal gaat schalen kan je te maken krijgen met inconsistenties in queries die waarde lezen. Als je hier meer over wilt weten raad ik je aan de term 'eventual consistency' op te zoeken.

### Geïsoleerd (Isolation)

Als er geen isolatie is voor transacties kunnen er bepaalde fenomenen optreden, zogeheten 'read phenomena'. In bepaalde situaties is de kans op zo een fenomeen onacceptabel.

#### Dirty reads

We spreken van een dirty read als een query een waarde leest die nog niet gecommit is.

**Voorbeeld**
Een restaurant wilt de omzet en de omzet per gerecht weten:

| GID   | aantal | prijs |
| ----- | ------ | ----- |
| pizza | 20     | €10   |
| pasta | 10     | €15   |

| Transactie + Query                                           | Antwoord                   |
| ------------------------------------------------------------ | -------------------------- |
| T1: ```SELECT GID, aantal*prijs FROM verkoop```              | pizza, 200<br />pasta, 150 |
| T2: ```UPDATE verkoop SET aantal = aantal+5 WHERE GID='pasta'``` |                            |
| T1: ```SELECT SUM(aantal*prijs) FROM verkoop```              | 425                        |

Omdat een query van transactie 2 de waarde 'aantal' in de rij van 'pasta' heeft aangepast leest de tweede query in transactie 1 een andere waarde dan de eerste query in transactie 1. Hierdoor is de uitkomst van de laatste query geen 350 maar 425.

#### Non-repeatable reads

Een non-repeatable read komt voor wanneer je een waarde voor de tweede keer leest en het veranderd is door een andere voltooide transactie. Het verschil met een dirty read is dus dat de andere transactie in het geval van een non-repeatable read wel gecommit is.

**Voorbeeld**
Een restaurant wilt de omzet en de omzet per gerecht weten:

| GID   | aantal | prijs |
| ----- | ------ | ----- |
| pizza | 20     | €10   |
| pasta | 10     | €15   |

| Transactie + Query                                           | Antwoord                   |
| ------------------------------------------------------------ | -------------------------- |
| T1: ```SELECT GID, aantal*prijs FROM verkoop```              | pizza, 200<br />pasta, 150 |
| T2: ```UPDATE verkoop SET aantal = aantal+5 WHERE GID='pasta'``` |                            |
| T2: ```COMMIT```                                             |                            |
| T1: ```SELECT SUM(aantal*prijs) FROM verkoop```              | 425                        |

Omdat een query van transactie 2 de waarde 'aantal' in de rij van 'pasta' heeft aangepast leest de tweede query in transactie 1 een andere waarde dan de eerste query in transactie 1. Hierdoor is de uitkomst van de laatste query geen 350 maar 425.

Dit is geen dirty read maar een non-repeatable read omdat transactie 2 is gecommit voordat de fout optreed.

#### Phantom reads

We spreken van een phantom read wanneer een andere transactie rijen toevoegd of verwijderd en de huidige transactie dit leest.

**Voorbeeld**
Een restaurant wilt de omzet en de omzet per gerecht weten:

| GID   | aantal | prijs |
| ----- | ------ | ----- |
| pizza | 20     | €10   |
| pasta | 10     | €15   |

| Transactie + Query                                       | Antwoord                   |
| -------------------------------------------------------- | -------------------------- |
| T1: ```SELECT GID, aantal*prijs FROM verkoop```          | pizza, 200<br />pasta, 150 |
| T2: ```INSERT INTO verkoop VALUES ('pannekoek', 8, 1)``` |                            |
| T2: ```COMMIT```                                         |                            |
| T1: ```SELECT SUM(aantal*prijs) FROM verkoop```          | 358                        |

Omdat transactie 2 een rij heeft toegevoegd leest de tweede query van transactie 1 een extra rij. Hierdoor is de uitkomst van de laatste query geen 350 maar 358.

#### Lost updates

Een lost update komt voor wanneer verschillende transacties tegelijkertijd dezelfde waarde proberen te veranderen.

Lost updates zijn eigenlijk geen read phenomena, omdat er niks gelezen wordt. Desondanks is wel belangrijk om te weten wat een lost update is.

**Voorbeeld**
Barry (T1) en Harry (T2) werken bij hetzelfde restaurant. Barry heeft 2 pizza's verkocht en Harry 1 pizza. Ze lezen beide hoeveel pizza's er al waren verkocht, rekenen uit hoeveel er zijn verkocht met hun eigen bijdrage en zetten het nieuwe aantal in de database.

| GID   | aantal | prijs |
| ----- | ------ | ----- |
| pizza | 20     | €10   |
| pasta | 10     | €15   |

| Transactie + Query                                         | Antwoord |
| ---------------------------------------------------------- | -------- |
| T1: ```SELECT aantal FROM verkoop WHERE GID='pizza'```     | 20       |
| T2: ```SELECT aantal FROM verkoop WHERE GID='pizza'```     | 20       |
| T1: ```UPDATE verkoop SET aantal = 22 WHERE GID='pizza'``` |          |
| T2: ```UPDATE verkoop SET aantal = 21 WHERE GID='pizza'``` |          |
| T1: ```COMMIT```                                           |          |
| T2: ```COMMIT```                                           |          |

Omdat Harry (T2) als laatste zijn transactie afrond eindigt het aantal pizza's op 21. De pizza's die Barry (T1) heeft verkocht raken verloren.

#### Isolatie niveaus

Er zijn verschillende niveaus van isolatie. Hoe hoger het niveau hoe meer read phenomena tegen worden gegaan. Een hoger niveau komt helaas ook met slechtere prestaties.

1. read uncommitted
2. read committed
3. repeatable reads
4. serializable

In de volgende tabel kun je zien welke read phenomena worden tegengegaan op welk niveau. ('-' = kan voorkomen, '+' = wordt tegengegaan)

| isolatie niveau  | dirty reads | lost updates | non-repeatable | phantom |
| ---------------- | :---------: | :----------: | :------------: | :-----: |
| read uncommitted |      -      |      -       |       -        |    -    |
| read committed   |      +      |      -       |       -        |    -    |
| repeatable reads |      +      |      +       |       +        |    -    |
| serializable     |      +      |      +       |       +        |    +    |

### Duurzaam (**D**urability)

Duurzaamheid zorgt er voor dat een voltooide transactie niet ongeldig kan worden. Zo is de data van een voltooide transactie er nog nadat de database herstart is.