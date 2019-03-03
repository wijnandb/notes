---
layout: post
title: Django website wielerspel
description: "opzetten van een Django website om wielerspel scores automatisch te verwerken."
modified: 2014-12-24
tags: [django, wielerspel]
image:
  path: /images/abstract-3.jpg
  feature: abstract-3.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
---

# Wielerspel website #

Op Dell Ubuntu laptop, directory sites, subdirectory Wielerspel (ook virtualenv wielerspel) is wielerspel de Django app.

Activate virtualenv in ~/sites/wielerspel

```
$ source bin/activate

$ cd wielerspel

$ python manage.py runserver
```

## Django
Django kent 
- models
- URLs and views
- templates



## Minimal Viable Product
MVP: ploegen bevatten renners, renners hebben punten behaald.

Lijst met renners en behaalde punten en ploegleider (of niet verkocht)

Rennerpagina: alle uitslagen die een renner heeft behaald

Uitslagenpagina: pagina met uitslagen per koers

## Gegevens importeren
Dit werkt natuurlijk het mooiste als ik uitslagen kan importeren waarna de berekening van de stand automatisch gebeurt. Er zijn packages beschikbaar om CSV's te importeren, dus een eerste stap is om die werkend te krijgen.
Laat ik echter vooruit kijken naar de werkwijze die ik voor ogen heb, want het is ook een bneetje jammer als ik dit allemaal optuig en het blijkt niet onderhoudbaar te zijn.

- Hoe 'verzamel' ik uitslagen?
- Hoe koppel ik uitslagen aan renners in de database?
- Hoe koppel ik uitslagen aan koersen in de database?

Het antwoord op de eerste vraag bepaalt in hoge mate de antwoorden op de volgende twee vragen, in de zin dat renners alleen gevonden kunnen worden als ze hetzelfde geschreven zijn. Ditzelfde geldt voor de koersen.

Er zijn verschillende bronnen waar ik naar kan kijken als het gaat om het scrapen van uitslagen:
- [UCI](https://dataride.uci.ch/Results/iframe/Results/10/)
- [Procyclingstats](https://www.procyclingstats.com/)
- [CQ ranking]()

### [UCI](https://dataride.uci.ch/Results/iframe/Results/10/)

Op de pagina van de [UCI](https://dataride.uci.ch/Results/iframe/Results/10/) staat een export button, om uitslagen in XLSX te downloaden.
Nadeel is dat de naam van de koers of het onderdeel er niet bijstaat in de download, maar wellicht kunnen we een koppeling maken met het koersID, wat ook te vinden is in de kalender.

### [Procyclingstats](https://www.procyclingstats.com/)
Procyclingstats heeft vele mogelijkheden om resultaten binnen te halen: 
- per ploeg
    Nadeel: alleen top 100 resultaten
- per renner
- per koers

Het gaat er om een regel te krijgen zoals die in het uitslagenoverzicht van de ploegen staat:
date, result, race, rider, class, points

Bij de andere uitslagen ontbreekt de naam van de koers.

Als in de database een koppeling wordt gemaakt tussen koers en renner, dan kun je dat wellicht op een andere manier 'regelen'.
Tijd om aan de slag te gaan met CSV import.

Nog even op een rijtje:
- lijst met koersen en bijbehorende categorie&euml;n
- lijst met renners
- uitslagenlijst is koppeling van koersen, renners en posities.

Hier zit nog wel 'aandachtspunt' voor wat betreft de jaargang.

Met bepaalde CSV-imports is het mogelijk om te [mappen naar een foreign key](https://github.com/edcrewe/django-csvimport#more-complex-relations). Hiervoor is het wel nofig dat er een unieke waarde is waar naar verwezen kan worden, zoals bijvoorbeeld volledige naam van een renner.

Dan blijft de mapping naar de foreign_key voor de koers nog over als uitdaging.

Wat heel mooi is aan de uitslagen op CQranking, is dat er onderscheid wordt gemaakt tussen verschillende klassementen. Zo is 2.1s de etappe-uitslag van een 2.1 koers, en 2.1 met daarbij een getal is de uitslag in het eindklasemnet en 2.1s met daarbij "leader" betekent dat de renner de leiderstrui droeg na afloop van de etappe.

Ander interessant punt: op de [rennerspagina](https://cqranking.com/men/asp/gen/rider_palm.asp?riderid=4377&year=2019&all=1&current=0) met de uitslagen, staat ook het UCi-nummer van de renner. In de URL staat het CQ riderID, wat PB en Jan ook gebruiken.

OK, [dit is de mooiste en interessantste pagina](https://cqranking.com/men/asp/gen/rider_palm.asp?riderid=5) die er is. Het bevat alle uitslagen van een renner in z'n carriere. Ik moet zien hoe ik die data kan scrapen en dan het id van de renner bij de uitslag op kan slaan. Daarnaast moet ik een renner-tabel maken waarbij het CQ-id het id is in mijn database
 

## Pagina's

Homepage: stand. 
Lijst met ploegleiders, geordend op behaald aantal punten van hoog naar laag.

/jaartal/
Bepaald naar welk jaar we kijken. Als er geen jaartal in URL staat kijken we naar huidige jaargang.

/jaartal/naamploegleider
Ploeg van een ploegleider in gegeven jaar


## Tabellen
ploegen: jaartal, ploegleider, renner

teams: id, jaartal, naam, niveau (dit zijn de 'echte' wielerploegen, soms krijgen ze nieuwe sponsornaam, vandaar 

```
ploegen/models.py¶
from django.db import models
from django_countries.fields import CountryField

class Ploegleiders(models.Model):
    voornaam = models.CharField(max_length=30)
    achternaam = models.Charfiels(max_length=30)
    birthdate = models.DateTimeField('geboortedatum')


class Ploeg(models.Model):
    ploegleider = models.ForeignKey(Ploegleiders, on_delete=models.CASCADE)
    renner = models.ForeignKey(Renners, on_delete=models.CASCADE)
    jaargang = models.IntegerField(default=0)

class Renners(models.Model):
    voornaam = models.CharField(max_length=50)
    achternaam = models.Charfiels(max_length=50)
    birthdate = models.DateTimeField('geboortedatum')
    nationaliteit = CountryField()

class Uitslagen(models.Model):
    race_naam = models.CharField(max_length=50)
    land = CountryField()
    

    race_category = (
        ('1.1'),
        ('1.HC'),
        ('NK1),
        ('NK2'),
        ('NK0.5'),
        ('WK'),
        ('WMON'),
        ('W1.KL'),
        ('W1.SK'),
        ('TdF'),
        ('GT'),
        ('W2.KL'),
        ('W2.SK'),
        ('2.HC'),
        ('2.1'),
    )
```
Hoe werkt dat?
Er is een race, een renner wordt eerste, dat betekent een aantal punten, en een aantal jackpotpunten (je kunt er ook voor kiezen om dat te negeren). Zet je dat aantal behaalde punten dan achter de uitslag? Dat is wel makkelijk, dat kun je makkelijk tonen.

Ik wil (denk ik) van alle races de top 10 opslaan en kunnen tonen, dus dat wordt een tabel met race, positie, renner, jaargang, punten. Eventueel nog zaken als tijd et cetrea. Per race zijn er meerdere klassementen, dus dat moet er ook in verwerkt worden: etappes, bergklassement, puntenklassement, algemeen klassement, jongerenklassement. Bij de grote rondes heb je ook nog eens de truidrager na elka etappe: is dat een apart soort klassement? Dan kun je krijgen: etappe 1, 1 t/m 10. Etappe 1, bergtrui. Etappe 1, jongerenklassement. Op die manier kun je ook nummers 2 t/m infinity een plek geven. Wat dan nodig is, zijn soorten klassementen: etappe-uitslag, KOM, punten, jongeren, ploegen, combinatie. Hoe maak je onderscheid tussen 


## Apps
Een app is een onderdeel van een Django site en is herbruikbaar. Niet druk over maken, komt vanzelf wel <strike>of over nadenken?</strike>












## Create database
Use PostgreSQL
- [x] Create database
- [x] Create database user
- [x] Put connection settings in wielerspel/settings.py

``` 
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'USER': 'ploegleider',
        'NAME': 'wielerspel',
    },
}

```
### Creating user
```
$ sudo -u postgres createuser <username>
```
### Creating Database
```
$ sudo -u postgres createdb <dbname>
```
### Giving the user a password
```
$ sudo -u postgres psql
psql=# alter user <username> with encrypted password '<password>';
```
### Granting privileges on database
```
psql=# grant all privileges on database <dbname> to <username> ;
```
### From within Postgresql
```
CREATE DATABASE wielerspel;
CREATE USER ploegleider WITH ENCRYPTED PASSWORD 'wielerspel1994';
GRANT ALL PRIVILEGES ON DATABASE wielerspel TO ploegleider;
```


