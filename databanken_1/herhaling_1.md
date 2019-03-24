# Herhaling 1 
## opgave
Geef voor elke wedstrijd het wedstrijdnummer en de volledige naam van de aanvoerder van het team waarvoor de wedstrijd werd gespeeld. Sorteer je resultaat volgens het wedstrijdnummer in oplopende volgorde.
### oplossing
```sql
select wedstrijdnr, naam, voorletters
from wedstrijden inner join teams using(teamnr) left outer join spelers on(teams.spelersnr = spelers.spelersnr)
```

## opgave
Geef voor alle huidige bestuurleden hun functie en de lijst van boetes die voor hen werd betaald.
Omdat je dit wil vergelijken met de boetebedragen die betaald werden voor leden die niet in het bestuur zitten, wil je deze boetebedragen ook opnemen in de tweede kolom van je resultaat. Sorteer je antwoord eerst op functie en daarna op het boetebedrag.
### oplossing
```sql
SELECT	functie, bedrag
FROM	bestuursleden FULL OUTER JOIN boetes
USING	(spelersnr)
WHERE	eind_datum is null
ORDER BY functie, bedrag
```

## opgave
Geef per team de verloren wedstrijden. Zorg dat teams zonder verloren wedstrijden ook in de output verschijnen.
Duid per wedstrijd aan of het om een actief bestuurslid gaat.
Sorteer op divisie en wedstrijdnummer.
### oplossing
```sql
select w.teamnr, divisie, wedstrijdnr, w.spelersnr, case when functie is null then '-' else 'actief' end as bestuurslid
from (teams t left outer join wedstrijden w on t.teamnr = w.teamnr and verloren > gewonnen) left outer join bestuursleden b on w.spelersnr = b.spelersnr and eind_datum is null
order by divisie, wedstrijdnr
```

## opgave
Geef een alfabetisch gesorteerde lijst van de namen van alle leden van de tennisvereniging die nog geen wedstrijden gespeeld hebben
### oplossing
```sql
select naam
from spelers left outer join wedstrijden w2 on spelers.spelersnr = w2.spelersnr
where w2.spelersnr is null
order by naam;
```

## opgave
Geef het spelersnummer en bondsnummer van alle spelers die jonger zijn dan de speler met bondsnummer 8467.
gebruik een INNER JOIN. Sorteer 1,2
### oplossing
```sql
select distinct s2.spelersnr, s2.bondsnr
from spelers s1 inner join spelers s2 on (s1.bondsnr = '8467')
where s1.geb_datum < s2.geb_datum
order by 1, 2
```

## opgave
Geef een lijst met het spelersnummer, de naam van de speler, de datum van de boete en het bedrag van de boete van al de spelers die een boete gekregen hebben met een bedrag groter dan 45,50 euro en in Rijswijk wonen. Geef expliciet aan welke join je gebruikt.
### oplossing
```sql
select b.spelersnr, naam, datum, bedrag
from spelers inner join boetes b on spelers.spelersnr = b.spelersnr
where plaats ilike 'rijswijk' and bedrag > 45.50
```

## opgave
Maak een lijst van alle mannelijke aanvoerders van een team en hun gespeelde wedstrijden.
Hierbij toon je voor deze spelers het spelersnummer en de volledige naam, voor het team de divisie en voor de wedstrijd het wedstrijdnummer.
Sorteer aflopend op het wedstrijdnummer.
### oplossing
```sql
select spelersnr, naam, voorletters, divisie, wedstrijdnr
from spelers inner join teams using(spelersnr) inner join wedstrijden w2 using(spelersnr)
where spelers.geslacht = 'M'
order by wedstrijdnr desc
```

## opgave
Geef voor de actieve bestuursleden zonder boete hun laatste gespeelde wedstrijd (die met het hoogste wedstrijdnummer).
Sorteer aflopend op spelersnr.
### oplossing
```sql
select bestuursleden.spelersnr, max(wedstrijdnr) as laatstewedstrijd
from bestuursleden left outer join boetes on bestuursleden.spelersnr = boetes.spelersnr and bestuursleden.eind_datum is null
inner join wedstrijden on bestuursleden.spelersnr = wedstrijden.spelersnr
where bedrag is null and eind_datum is null
group by bestuursleden.spelersnr
order by bestuursleden.spelersnr desc
```

## opgave
Geef per team de leeftijd van de aanvoerder (tip: postgresql heeft een AGE() functie) en het aantal verschillende spelers dat voor dit team gespeeld heeft.
Alleen teams waarvoor wedstrijden zijn gespeeld en die een aanvoerder hebben, moeten vermeld worden.
Sorteer op leeftijd en daarna op aantal verschillende spelers en daarna op teamnr.
### oplossing
```sql
select distinct w2.teamnr, extract(year from age(geb_datum)) || ' jaar' as leeftijd ,count(distinct w2.spelersnr) as aantalspelers
from teams inner join wedstrijden w2 on teams.teamnr = w2.teamnr inner join spelers s2 on teams.spelersnr = s2.spelersnr
group by w2.teamnr, s2.geb_datum
order by leeftijd, aantalspelers, w2.teamnr
```

## opgave
Geef per team het hoogste wedstrijdnummer van een wedstrijd, gespeeld door een bestuurslid (actief en niet meer actief) die geen boete heeft gekregen.
Sorteer op teamnr.
### oplossing
```sql
select teamnr, max(wedstrijdnr) as laatstewedstrijd
from wedstrijden inner join bestuursleden b1 on b1.spelersnr = wedstrijden.spelersnr left outer join boetes on b1.spelersnr = boetes.spelersnr
where bedrag is null
group by wedstrijden.teamnr
order by teamnr
```
