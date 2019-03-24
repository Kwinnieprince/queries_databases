# group by

## opgave
Geef voor elke geboortejaar van de klanten, het aantal klanten, het kleinste klantennummer en het grootste klantennummer. Geef ook het totaal aantal klanten en het kleinste en grootste klantennummer.
Sorteer van voor naar achter.
## oplossing
```sql
select extract(year from geboortedatum), count(*), min(klanten.klantnr), max(klanten.klantnr)
from klanten
group by (extract(year from geboortedatum))
UNION
(select null , count(*), min(klantnr), max(klantnr) from klanten )
order by 1
```

## opgave
We willen telkens het aantal reizen en de totale prijs van de volgende situaties. Voor elke maand van vertrek, voor elk tiental van de reisduur, voor de combinatie van maand van vertrek en tiental van de reisduur en het het totale plaatje.
Sorteer van voor naar achter.
## oplossing
```sql
select extract(month from vertrekdatum) as date_part, round((reisduur/10), 0), count(vertrekdatum), sum(prijs)
from reizen
group by rollup(date_part, round((reisduur/10), 0))
union all
select null, round((reisduur/10),0), count(vertrekdatum), sum(prijs)
from reizen
group by round((reisduur/10),0)
order by 1,2,3,4
```

## opgave
Geef voor de hemelobjecten een diameter groter dan 1000 het aantal hemelobjecten en de gemiddelde diameter per groep die aan het volgende voldoet. We zien per object zien hoeveel satellieten hieraan voldoen, daarnaast in combinatie met hetzelfde object en afstand per 100tal. Alsook een algemeen overzicht. Sorteer van voor naar achter.
## oplossing
```sql
select objectnaam, round(afstand/100),satellietvan, count(*) as count, avg(diameter)
from hemelobjecten
where diameter>1000
group by objectnaam
union
select null as objectnaam, null as count,satellietvan, count(*) as count, avg(diameter)
from hemelobjecten
where diameter>1000
group by satellietvan
union
select null as objectnaam, null as count,null as satellietvan, count(*) as count, avg(diameter)
from hemelobjecten
where diameter>1000
order by 3,1,4

--met rollup
select objectnaam, round(afstand/100), satellietvan, count(*) as count, avg(diameter)
from hemelobjecten
where diameter>1000
group by rollup ((satellietvan),(objectnaam,afstand))
order by 3,1,4
```
