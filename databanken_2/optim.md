# Optim

## opgave
Geef een lijst met het spelersnummer en de naam van de speler die in Rijswijk wonen en die in 1980 een boete gekregen hebben van 25 euro. Sorteer van voor naar achter.
Probeer gelijk of beter te doen dan "(cost=2.37..2.37 rows=1 width=68)".
## oplossing
```sql
select spelersnr, naam
from spelers inner join boetes using (spelersnr)
where plaats = 'Rijswijk' and extract(year from datum) = '1980' and bedrag = 25
order by 1, 2
```

## opgave
Geef de naam en het spelersnummer van de spelers die ooit penningmeester geweest zijn van de club, die bovendien ooit een boete betaald hebben van meer dan 75 euro, en die ooit een wedstrijd gewonnen hebben met meer dan 2 sets verschil. Sorteer van voor naar achter. 
Probeer gelijk of beter te doen dan "Unique (cost=100.38..100.54 rows=21 width=68)".
## oplossing
```sql
select naam, spelersnr
from spelers inner join bestuursleden using(spelersnr) inner join boetes using(spelersnr) inner join wedstrijden using(spelersnr)
where functie = 'Penningmeester' and bedrag > 75 and (gewonnen - verloren) > 2
```


## opgave
Geef van elke speler het spelersnr, de naam en het verschil tussen zijn of haar jaar van toetreding en het gemiddeld jaar van toetreding. Sorteer van voor naar achter. 
Probeer gelijk of beter te doen dan "Sort (cost=33.16..33.66 rows=200 width=86)"
## oplossing
```sql
select spelersnr, naam, voorletters, jaartoe - (select avg(jaartoe) from spelers) as verschil
from spelers
order by 1,2,3,4
```

## opgave
Je kan per speler berekenen hoeveel boetes die speler heeft gehad en wat het totaalbedrag per speler is. Pas nu deze querie aan zodat per verschillend aantal boetes wordt getoond hoe vaak dit aantal boetes voorkwam. Sorteer van voor naar achter. 
Probeer gelijk of beter te doen dan "Sort (cost=46.39..46.89 rows=200 width=8)".
## oplossing
```sql
select a, count(a) count
from (select count(*) a
from boetes
group by spelersnr) aantallen
group by a
order by 1,2
```

## opgave
Geef van alle spelers het verschil tussen het jaar van toetreding en het geboortejaar, maar geef alleen die spelers waarvan dat verschil groter is dan 20. Sorteer van voor naar achter. 
Probeer zo goed of beter te doen dan "Sort (cost=17.20..17.37 rows=67 width=90)"
## oplossing
```sql
select spelersnr, naam, voorletters, (jaartoe - extract(year from geb_datum)) as toetredingsleeftijd
from spelers
where (jaartoe - extract(year from geb_datum)) > 20
order by 1,2,3,4
```

## opgave

## oplossing
```sql

```
