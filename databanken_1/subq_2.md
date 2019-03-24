# subq 2

## opgave
Geef het totaal aantal boetes, het totale boetebedrag, het minimum en het maximum boetebedrag dat door onze club betaald werd. Let er hierbij op dat er gehele getallen worden getoond (rond af indien nodig). Sorteer van voor naar achter, oplopend.
## oplossing
```sql
select round(count(*), 0) as aantal_boetes, round(sum(bedrag), 0) as totaal_bedrag, round(min(bedrag), 0) as minimum, round(max(bedrag)) as maximum
from boetes
order by 1,2,3,4
```

## opgave
Geef voor elke aanvoerder het spelersnr, de naam en het aantal boetes dat voor hem of haar betaald is en het aantal teams dat hij of zij aanvoert. Toon enkel aanvoerders die boetes gekregen hebben. Sorteer van voor naar achter, oplopend.
## oplossing
```sql
select spelersnr, naam, voorletters, aantalboetes, aantalteams
from spelers inner join (select spelersnr, count(betalingsnr) as aantalboetes from boetes group by spelersnr) as boetes using(spelersnr)
inner join (select spelersnr, count(teamnr) as aantalteams from teams group by spelersnr) as teams using (spelersnr)
order by 1,2,3,4,5
```

## opgave
Geef een lijst met het spelersnummer en de naam van de spelers die in Rijswijk wonen en die in 1980 een boete gekregen hebben van 25 euro (meerdere voorwaarden dus). Gebruik hiervoor geen exists operator maar wel zijn tegenhanger die meestal bij niet-gecorreleerde subquery's wordt gebruikt. Sorteer van voor naar achter, oplopend.
## oplossing
```sql
select spelersnr, naam
from spelers
where plaats = 'Rijswijk' and spelersnr in (
  select spelersnr from boetes where extract(year from datum) = 1980 and bedrag  = 25
  )
```

## opgave
Geef alle spelers voor wie meer boetes zijn betaald dan dat ze wedstrijden hebben gespeeld. Zorg dat spelers zonder wedstrijd ook getoond worden.
Sorteer van voor naar achter, oplopend.
## oplossing
```sql
select naam, voorletters, geb_datum
from spelers
where (select count(betalingsnr) from boetes where boetes.spelersnr = spelers.spelersnr)
        >
      (select count(wedstrijdnr) from wedstrijden where spelers.spelersnr = wedstrijden.spelersnr)
order by 1,2,3
```

## opgave
Geef alle spelers die geen wedstrijd voor team 1 heeft gespeeld. Sorteer op naam, daarna op nr.
## oplossing
```sql
select spelersnr, naam
from spelers
where spelersnr not in (select spelersnr from wedstrijden where teamnr = 1)
order by naam, spelersnr
```

## opgave
Geef voor elke speler die ooit een boete heeft betaald, de hoogste boete weer en hoelang het geleden is dat deze boete werd betaald. Sorteer van groot naar klein op bedrag en daarna omgekeerd op “leeftijd..” van de boete.
## oplossing
```sql
select spelersnr, bedrag, age(datum)
from boetes
where bedrag in (select max(bedrag) from boetes b where b.spelersnr = boetes.spelersnr )
order by  bedrag desc , 3
```

## opgave
Welke spelers hebben voor alle teams gespeeld uit de teamstabel ?
(= voor welke speler bestaat er geen enkel team waar de betreffende speler nooit voor gespeeld heeft). Sorteer op spelers nummer. Gebruik de exists operator.
## oplossing
```sql
select spelersnr
from spelers
where not exists(
    select * from teams where not exists(
        select * from wedstrijden where teams.teamnr = wedstrijden.teamnr and spelers.spelersnr = wedstrijden.spelersnr
      )
  )
order by spelersnr
```

## opgave
Maak een lijst met de spelers (naam van de speler, voorletter en woonplaats) die ooit gespeeld hebben voor een team dat nu in de tweede divisie speelt en waarvoor geen enkele boete betaald werd voor 1 januari 1981. Sorteer van voor naar achter, oplopend. Zorg dat er geen dubbels worden getoond.
## oplossing
```sql
select naam, voorletters, plaats
from spelers
where spelersnr in (select w.spelersnr from wedstrijden w inner join teams using(teamnr) where divisie = 'tweede')
and spelersnr in (select spelersnr from boetes right outer join spelers using(spelersnr) where datum > '1981-01-01'::date or datum is null)
order by 1,2,3
```

## opgave
Geef de twee laagste bondnrs terug. (tip: dwz er zijn dus minder dan 2 bondsnr die kleiner zijn). Sorteer op bondnr. Zonder het gebruik van LIMIT.
## oplossing
```sql
select bondsnr
from spelers
where 2 > (select count(*) from spelers s2 where spelers.bondsnr > s2.bondsnr) and bondsnr is not null
order by bondsnr
```
