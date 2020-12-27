# Subq 1
## opgave
Geef voor elke aanvoerder het spelersnr, de naam en het aantal boetes dat voor hem of haar betaald is en het aantal teams dat hij of zij aanvoert. Aanvoerders zonder boetes mogen niet getoond worden. Sorteer, beginnend bij de eerste kolom, eindigend bij de laatste kolom.
## oplossing
```sql
select spelersnr, naam, voorletters, aantalboetes, aantalteams
from spelers
       inner join (select count(*) as aantalboetes, spelersnr from boetes group by spelersnr) as t using (spelersnr)
       inner join (select count(*) as aantalteams, spelersnr from teams group by spelersnr) as u using(spelersnr)
order by 1,2,3,4
```

## opgave
Geef van elke speler het spelersnr, de naam en het verschil tussen zijn of haar jaar van toetreding en het gemiddeld jaar van toetreding van de spelers die in dezelfde plaats wonen. Sorteer op spelersnr. Zet het berekende verschil om naar het datatype numeric met precisie 5 en schaal 3.
## oplossing
```sql
select spelersnr, naam, voorletters, cast(jaartoe - gem as numeric(5,3))
from spelers inner join (select avg(jaartoe) as gem, plaats from spelers group by plaats) as t using(plaats)
order by spelersnr
```

## opgave
Geef de spelersgegevens van de speler(s) met het hoogste bedrag (voor één boete, niet het totaalbedrag). Als twee spelers een even hoge boete gehad hebben, moeten beide spelers getoond worden (LIMIT is dus geen optie).
Sorteer alfabetisch op naam en voorletters.
## oplossing
```sql
select spelersnr, voorletters, naam
from spelers
where spelersnr in (select spelersnr from boetes where bedrag = (select max(bedrag) from boetes))
order by naam, voorletters
```

## opgave
We willen een statistiek van hoeveel wedstrijden de spelers winnen.
Geef een lijst met aantal gewonnen wedstrijden en het aantal spelers dat dit aantal wedstrijden gewonnen heeft.
Dus bv als er vier spelers zijn die elk drie wedstrijden hebben gewonnen, dan is de output: aantal_gewonnen: 3, aantal_spelers: 4. Dit graag voor alle aantallen gewonnen wedstrijden en alle spelers.
Sorteer op aantal gewonnen wedstrijden.
## oplossing
```sql
select aantal_gewonnen, count(*) as aantal_spelers
from (select count(wedstrijdnr) as aantal_gewonnen from  wedstrijden where gewonnen > verloren group by spelersnr) as gewonnen
group by aantal_gewonnen
order by aantal_gewonnen
```

## opgave
Geef voor elke speler die een wedstrijd heeft gespeeld het spelersnr en het totaal aantal boetes. Spelers die een wedstrijd gespeeld hebben, maar geen boetes hebben, moeten ook getoond worden.
Sorteer op het aantal boetes en op spelersnr;
## oplossing
```sql
select distinct spelersnr, aantalboetes
from wedstrijden left outer join (
  select spelersnr, count(betalingsnr) as aantalboetes
  from boetes
  group by spelersnr) boetes using(spelersnr)
order by aantalboetes, spelersnr
```

## opgave
Geef voor alle spelers die geen penningmeester zijn of zijn geweest alle gewonnen wedstrijden, gesorteerd op wedstrijdnummer.
## oplossing
```sql
SELECT spelersnr, wedstrijdnr
FROM wedstrijden
WHERE spelersnr NOT IN (
	SELECT spelersnr
	FROM bestuursleden
	WHERE functie = 'Penningmeester')
	AND gewonnen > verloren
ORDER BY wedstrijdnr
```

## opgave
Je kan per speler berekenen hoeveel boetes die speler heeft gehad en wat het totaalbedrag per speler is. Pas nu deze querie aan zodat per verschillend aantal boetes wordt getoond hoe vaak dit aantal boetes voorkwam.Sorteer eerst op de eerste kolom en daarna op de tweede kolom.
## oplossing
```sql
select aantalboetes as a, count(aantalboetes)
from (select count(*) as aantalboetes from boetes
  group by spelersnr) as t
group by aantalboetes
order by 1,2
```

## opgave
Geef van alle spelers het verschil tussen het jaar van toetreding en het geboortejaar, maar geef alleen die spelers waarvan dat verschil groter is dan 20. Sorteer deze gegevens beginnend bij de eerste kolom, eindigend bij de laatste kolom.
## oplossing
```sql
select spelersnr, naam, voorletters, toetredingsleeftijd
from spelers inner join (select (jaartoe - extract(year from geb_datum)) as toetredingsleeftijd, spelersnr
  from spelers ) as t using(spelersnr)
where toetredingsleeftijd > 20
order by 1,2,3,4
```

## opgave
Geef een lijst van alle spelers (spelersnr en woonplaats) die met minstens twee in dezelfde plaats wonen. Sorteer aflopend op woonplaats, daarna op spelersnr.
## oplossing
```sql
SELECT spelersnr, plaats
FROM spelers
WHERE plaats in (SELECT plaats
	FROM spelers
	GROUP BY plaats
	HAVING count(plaats) >= 2)
ORDER BY plaats DESC, spelersnr ASC
```
