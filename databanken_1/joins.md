# joins 1

## opgave
Maak een lijst met alle spelers die ooit een boete gekregen hebben die hoger is dan 50 euro. Geen dubbels. Sorteer van voor naar achter.
## oplossing
```sql
select distinct naam, voorletters, plaats
from spelers inner join boetes b on spelers.spelersnr = b.spelersnr
where bedrag > 50
order by naam
```

## opgave
Geef van elke boete het betalingsnr, het boetebedrag en het percentage dat het bedrag uitmaakt van de som van alle bedragen. Sorteer deze data op het betalingsnr. Zorg dat er maar twee getallen na de komma getoond worden (rond af). Sorteer van voor naar achter.
## oplossing
```sql
select betalingsnr, bedrag, round((bedrag/(select sum(bedrag) from boetes)*100),2)
from boetes
group by betalingsnr
order by betalingsnr
```

## opgave
Geef chronologisch de spelersnummers van de bestuursleden die voorzitter zijn of geweest zijn (chronologisch op begindatum van het voorzitterschap) met vermelding van deze begindatum, alsook hun naam en huidig adres.
Als het vollegie adres niet gekend is dan moet “adres ongekend” weergegeven worden. Sorteer van voor naar achter.
## oplossing
```sql
select begin_datum, naam, case
  when straat is null or huisnr is null or postcode is null or plaats is null
    then 'adres ongekend'
  else straat || ' ' || huisnr || ' ' || plaats || ' ' || postcode
end as adres
from bestuursleden inner join spelers s on bestuursleden.spelersnr = s.spelersnr
where functie = 'Voorzitter'
order by begin_datum
```

## opgave
Geef alle wedstrijden van het team waarvan speler 6 aanvoerder is. Sorteer
## oplossing
```sql
select wedstrijdnr
from wedstrijden inner join teams using (teamnr)
where teams.spelersnr = '6'
order by wedstrijdnr
```

## opgave
Geef alle spelers die geen enkele wedstrijd voor team 1 hebben gespeeld. Sorteer op naam, daarna op spelersnr.
## oplossing
```sql
select s.spelersnr, naam
from spelers s 
where spelersnr not in (select spelersnr from wedstrijden where teamnr = 1)
order by naam, spelersnr
```

## opgave
Maak een lijst met de spelers (naam van de speler, voorletter en woonplaats) die ooit gespeeld hebben voor een team dat nu in de tweede divisie speelt en waarvoor geen enkele boete betaald werd voor 1 januari 1981. Geen dubbels, sorteer van voor naar achter.
## oplossing
```sql
select naam, voorletters, plaats
from spelers
where spelersnr in (select w.spelersnr from wedstrijden w inner join teams using(teamnr) where divisie = 'tweede')
and spelersnr in (select spelersnr from boetes right outer join spelers using(spelersnr) where datum > '1981-01-01'::date or datum is null)
order by 1,2,3
```
