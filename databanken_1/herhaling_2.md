# herhaling 2

## opgave
Welke reizigers hebben al meer dan 1 reis ondernomen waarvoor ze meer dan 2,5 miljoen euro moesten betalen?
Sorteer op naam
## oplossing
```sql
select naam, count(d2.reisnr) as aantal_reizen
from reizen inner join deelnames d2 on reizen.reisnr = d2.reisnr inner join klanten k on d2.klantnr = k.klantnr
where prijs > 2.5
group by naam
having count(d2.reisnr) > 1
order by naam
```

## opgave
Op welke planeten verblijft men gemiddeld langer dan 2 dagen?
Sorteer op objectnaam.
## oplossing
```sql
SELECT h.objectnaam, AVG(b.verblijfsduur)
FROM bezoeken AS b
INNER JOIN hemelobjecten AS h USING (objectnaam)
WHERE h.satellietvan = 'Zon'
GROUP BY h.objectnaam
HAVING AVG(b.verblijfsduur) > 2
ORDER BY h.objectnaam
```

## opgave
Geef een lijst van alle reizen met tenminste 1 bezoek of passage aan de Maan of aan Mars. De lijst moet stijgend gesorteerd zijn op basis van het vertrekdatum.
## oplossing
```sql
select distinct b.reisnr, vertrekdatum
from reizen inner join bezoeken b on reizen.reisnr = b.reisnr
where objectnaam like 'Maan' or objectnaam like 'Mars'
order by vertrekdatum asc
```

## opgave
Welke reizen hebben exact drie verschillende hemelobjecten als reisdoel?
Sorteer op reisnr.
## oplossing
```sql
select reisnr
from bezoeken
group by reisnr
HAVING count(distinct objectnaam) = 3
```

## opgave
Welke klanten (klantnaam, geboortedatum) zijn op een ruimtereis vertrokken in het jaar dat ze 45 jaar geworden zijn?
Het resultaat moet stijgend gesorteerd worden op de geboortedatum.
## oplossing
```sql
SELECT k.naam AS klantnaam, k.geboortedatum
FROM deelnames d
INNER JOIN klanten k USING (klantnr)
INNER JOIN reizen r USING (reisnr)
WHERE EXTRACT(year FROM r.vertrekdatum) - EXTRACT(year FROM k.geboortedatum) = 45
GROUP BY k.naam, k.geboortedatum
ORDER BY geboortedatum
```

## opgave
Geef de leeftijd van de jongste klant op moment van vertrek.
## oplossing
```sql
SELECT EXTRACT(year FROM min(age(r.vertrekdatum, k.geboortedatum))) AS jongsteleeftijd
FROM deelnames AS d
INNER JOIN reizen AS r USING (reisnr)
INNER JOIN klanten AS k USING (klantnr)
```

## opgave
Geef de diameter van de grootste, niet bezochte maan (satelliet van een planeet)
## oplossing
```sql
select max(diameter) as grootstemaan
from hemelobjecten left outer join bezoeken b on hemelobjecten.objectnaam = b.objectnaam
where satellietvan not ilike 'zon' and reisnr is null
```

## opgave
Geef de naam, diameter en het langste verblijf op van alle hemelobjecten met minder dan 5 hemelobjecten die rond dit object draaien.
Sorteer op objectnaam.
## oplossing
```sql
select planeet.objectnaam, planeet.diameter, max(verblijfsduur) as maximale_verblijf
from (hemelobjecten planeet left outer join hemelobjecten maan on planeet.objectnaam = maan.satellietvan) inner join bezoeken b on b.objectnaam = planeet.objectnaam
group by planeet.objectnaam
having count(distinct maan.objectnaam) < 5
```

## opgave
Geef per klant het totaal aantal reizen waaraan deze klant zal deelnemen en het langste bezoek dat deze klant zal maken aan een hemelobject, over alle reizen heen.
Klanten zonder reizen of zonder bezoeken moeten ook voorkomen in het overzicht.
Sorteer op klantnr.
## oplossing
```sql
select klantnr, count(distinct d.reisnr) as aantal, max(verblijfsduur) as langstebezoek
from klanten left outer join deelnames d using(klantnr) left outer join reizen r on d.reisnr = r.reisnr left outer join bezoeken b on r.reisnr = b.reisnr
group by klantnr
order by klantnr
```

## opgave
Geef voor elk hemelobject de minimale en maximale gemiddelde afstand tot zijn zon (de centrale ster in een sterrenstelsel) als je weet dat de kolom 'afstand' de gemiddelde afstand bevat tot het hemelobject waarrond ze draaien. Met de grootte van het hemelobject hoef je geen rekening te houden.
Sorteer op minimale afstand en op objectnaam.
Tip: bekijk de functie COALESCE om met null-waardes rekening te houden.
## oplossing
```sql
SELECT satelliet.objectnaam,
COALESCE(planeet.afstand + satelliet.afstand, satelliet.afstand, 0) AS maximale_afstand,
COALESCE(planeet.afstand - satelliet.afstand, satelliet.afstand, 0) AS minimale_afstand
FROM hemelobjecten AS satelliet
LEFT OUTER JOIN hemelobjecten AS planeet ON planeet.objectnaam = satelliet.satellietvan
ORDER BY minimale_afstand, satelliet.objectnaam
--klopt niet altijd volgens sqldropbox
```
