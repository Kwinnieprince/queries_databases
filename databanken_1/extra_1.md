# extra 1

## opgave
Hoeveel sets zijn er in totaal gewonnen, hoeveel sets werden in totaal verloren en welk is het uiteindelijke saldo?
## oplossing
```sql
select sum(gewonnen) as totaal_gewonnen, sum(verloren) as totaal_verloren, (sum(gewonnen) - sum(verloren)) as saldo
from wedstrijden
```

## opgave
Geef een totaal overzicht van alle spelers, hun boetes en de functies die ooit vervuld hebben. Elke speler moet getoond worden, als ie een eventuele boete heeft gekregen en/of een eventuele functie als bestuurslid heeft gehad dan moet dit ook getoond worden. Toon de volledige naam, het bedrag en datum van de eventuele boete en de eventuele bestuursfuncties. Sorteer van voor naar achter.
## oplossing
```sql
select naam, voorletters, functie, bedrag, datum
from boetes right outer join spelers using(spelersnr) left outer join bestuursleden using(spelersnr)
order by 1,2,3,4,5
```

## opgave
Geef een overzicht van de spelers die een boete hebben gekregen, indien deze boete meer van 90 euro is toon je informatie “pijnlijk” anders “te doen”. Toen de volledige naam en de categorie van de bijhorende boete. Sorteer van voor naar achter.
## oplossing
```sql
select naam, voorletters, case when bedrag > 90 then 'pijnlijk' else 'te doen' end as comment
from boetes inner join spelers s on boetes.spelersnr = s.spelersnr
order by 1,2,3
```
