# set oper

## opgave
Geef een overzicht van alle spelers, gevolgd door alle bestuursleden, gesorteerd op jaar van toetreding of beginjaar van hun functie en vervolgens op spelersnr.
Geen dubbels tonen.
## oplossing
```sql
select spelersnr as veld1, naam as veld2, jaartoe as veld3
from spelers
union
select spelersnr, functie, extract(YEAR from date(begin_datum))
from bestuursleden
order by veld3, veld1
```

## opgave
Geef een lijst met alle spelersnrs, naam en het aantalwedstrijden ze gespeeld hebben en op een nieuwe lijn het aantal bestuursfuncties die ze hebben/hadden.
Spelers die zowel wedstrijden gespeeld hebben als bestuurslid zijn, komen dus twee keer voor in het resultaat.
Sorteer op spelersnr en aantal. Geen dubbels tonen.
## oplossing
```sql
select spelersnr as nr, naam, aantal
from spelers inner join (
  select spelersnr, count(wedstrijdnr) as aantal
  from wedstrijden
  group by spelersnr
  ) t using(spelersnr)
union
select s.spelersnr, naam, count(*)
from bestuursleden inner join spelers s on bestuursleden.spelersnr = s.spelersnr
group by s.spelersnr
order by nr, aantal
```

## opgave
Geef een overzicht van het aantal records in de vijf tabellen uit de database tennis. Je mag hiervoor elke tabel manueel tellen.
Sorteer op label.
## oplossing
```sql
select 'bestuursleden' as label,count(*) as aantal from bestuursleden
union all
select 'boetes' as label, count(*) as aantal from boetes
union all
select 'spelers' as label, count(*) as aantal from spelers
union all
select 'teams' as label, count(*) as aantal from teams
union all
select 'wedstrijden' as label, count(*) as aantal from wedstrijden
--Dit is de juiste oplossing hierboven maar onder vind ik zelf een betere, performantere en makkelijk bruikbaar op andere tabellen
SELECT relname as label,n_live_tup as aantal
FROM pg_stat_user_tables
where schemaname = 'tennis' and relname not ilike '%_%l'
ORDER BY label;
--sql dropbox rekent deze niet juist maar het geeft wel de juiste uitkomst
```

## opgave
Geef een overzicht van de boetebedragen, aantal gewonnen en verloren sets en aantal verschillende functies. Bekijk de output voor de manier hoe het getoond moet worden.
Sorteer van links naar rechts.
Tip: Het lijkt onlogisch, maar zelfs NULL krijgt een datatype en kan niet impliciet wijzigen van datatype.
## oplossing
```sql
select sum(bedrag)::text as boetebedrag, null as aantalgewonnen, null as aantalverloren, null as aantalfuncties
from boetes
union
select null, sum(gewonnen)::text, null, null
from wedstrijden
union
select null, null, sum(verloren)::text, null
from wedstrijden
union
select null, null, null, count(distinct functie)::text
from bestuursleden
order by boetebedrag, aantalgewonnen, aantalverloren, aantalfuncties
```

## opgave
Geef de spelersnummers die minstens één keer bestuurslid zijn geweest.
Gebruik hiervoor geen JOIN, DISTINCT, GROUP BY, IN, ANY, ALL of EXISTS.
Sorteer op spelersnr
## oplossing
```sql
select spelers.spelersnr
from spelers, bestuursleden
where spelers.spelersnr = bestuursleden.spelersnr
intersect
select spelers.spelersnr
from spelers
order by spelersnr
```

## opgave
Geef de spelersnummers die geen wedstrijd gespeeld hebben.
Gebruik hiervoor geen JOIN, DISTINCT, GROUP BY, IN, ANY, ALL of EXISTS.
Sorteer op spelersnr
## oplossing
```sql
select spelers.spelersnr
from spelers
except
select spelers.spelersnr
from spelers, wedstrijden
where spelers.spelersnr = wedstrijden.spelersnr
order by spelersnr
```

## opgave
Geef voor de spelers die bestuurslid en/of teamkapitein zijn hun naam en een oplijsting van hun functienamen (huidig of verleden) en hun divisies waarvoor ze kapitein zijn.
Sorteer op spelersnaam en naam. 
Gebruik geen OUTER JOIN of WHERE.
## oplossing
```sql
select naam as spelersnaam, functie as naam
from spelers s inner join bestuursleden b using (spelersnr)
union
select naam, divisie
from spelers inner join teams t on spelers.spelersnr = t.spelersnr
order by spelersnaam, naam
```

## opgave
Geef een lijst van alle spelers die bestuurslid geweest zijn (of nu nog zijn) en/of een boete hebben gehad hun aantal boetes en hun laatst begonnen bestuursfunctie. Zorg dat spelers die boetes gehad hebben én bestuurder zijn (geweest) twee keer voorkomen in de kolom 'data' twee verschillende waardes.
Sorteer op naam, voorletters en data.
Gebruik de to_char functie voor het formaat van de geboortedatum (bv 12/12/1900).
## oplossing
```sql
select naam, voorletters, to_char(geb_datum, 'DD/MM/YYYY') as geboortedatum, 'Boetes: ' || count(betalingsnr) as data
from spelers inner join boetes b using(spelersnr)
group by spelersnr
union
select naam, voorletters, to_char(geb_datum, 'DD/MM/YYYY'), functie
from spelers inner join bestuursleden b using (spelersnr)
where begin_datum = (select max(begin_datum) from bestuursleden where b.spelersnr = spelersnr)
order by naam, voorletters, data
```

## opgave
(Een variant op een vorige opgave)
We zijn wat betreft boetes alleen geïnteresseerd als de speler geboren is voor 1963.

Verder uitleg:
Geef een lijst van ALLE spelers die bestuurslid geweest zijn en/of een boete hebben gehad, (alleen als ze geboren zijn voor 1963) en hun laatst begonnen bestuursfunctie. 
Zorg ervoor dat spelers die een bestuursfunctie hebben/hadden, geboren zijn voor 1963, maar geen boete hebben gekregen, toch in het resultaat voorkomen met een extra lijn 'Geen boetes'.
Spelers die geen bestuurslid zijn geweest, maar een boete hebben gehad, komen één keer voor in het resultaat met hun aantal boetes.
Spelers die geen bestuurslid zijn geweest en geboren zijn na 1962, moeten ook één keer in het resultaat voorkomen met als data 'Gewone speler' (het kan zelfs zijn dat geen boete gekregen hebben, alle spelers moeten sowieso getoond worden).
Sorteer op naam, voorletters en data (let op de hoofdletters in het veld data).

(ps: in deze oefening zit bijna alles wat je de voorbije twee jaar gezien hebt van SQL)
## oplossing
```sql
select naam, voorletters, to_char(geb_datum, 'DD/MM/YYYY') as geboortedatum, case when begin_datum is null then 'Gewone speler' else functie end as data
from spelers left outer join bestuursleden b using(spelersnr)
where begin_datum = (select max(begin_datum) from bestuursleden where b.spelersnr = bestuursleden.spelersnr)
   or (begin_datum is null and extract(year from geb_datum) > '1962')
union
select naam, voorletters, to_char(geb_datum, 'DD/MM/YYYY') as geboortedatum, case when betalingsnr is null then 'Geen boetes' else 'Boetes: ' || count(betalingsnr) end
from spelers left outer join boetes using (spelersnr)
where extract(year from geb_datum) < '1963'
group by betalingsnr, spelersnr
order by naam, voorletters, data
```
