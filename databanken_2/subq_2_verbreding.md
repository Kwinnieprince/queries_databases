# Subqueries 2 verbreding

## opgave
Geef de diameter van het grootste hemellichaam dat bezocht is op de vroegste reis waar klantnr 126 niet op meegegaan is.
## oplossing
```sql
SELECT MAX(diameter) AS grootste
FROM hemelobjecten h1 JOIN bezoeken USING(objectnaam) JOIN reizen USING(reisnr)
WHERE vertrekdatum = (
        SELECT MIN(vertrekdatum)
        FROM reizen
        WHERE reisnr NOT IN (
                SELECT klantnr
                FROM deelnames
                WHERE klantnr = 126
        )
)
```
## opgave
Geef de deelnemers waarbij hun aantal reizen die ze ondernemen groter is dan alle hemelobjecten (die niet beginnen met de letter 'M') hun aantal keren dat ze bezocht zijn. Of anders geformuleerd:
Geef de deelnemers met meer deelnames dan het grootste aantal bezoeken aan een hemelobject dat niet met de letter 'M' begint (:deze deelnemer meer deelnames heeft dan de "grootste" .. = deze deelnemer heeft meer deelnames dan "alle" ..)
Sorteer op klantnr.
## oplossing
```sql
SELECT klantnr, vnaam, naam, COUNT(reisnr) aantaldeelnames
FROM klanten INNER JOIN deelnames USING(klantnr)
GROUP BY klantnr, vnaam, naam
HAVING COUNT(reisnr) > ALL (
        SELECT COUNT(*)
        FROM bezoeken
        WHERE NOT EXISTS (
                SELECT objectnaam
                FROM hemelobjecten
                WHERE objectnaam LIKE 'M%'
                AND bezoeken.objectnaam = hemelobjecten.objectnaam
        )
        GROUP BY objectnaam
)
```
## opgave
Geef alle niet-bezochte hemelobjecten, buiten het grootste hemellichaam.
Sorteer op diameter en objectnaam.
## oplossing
```sql
SELECT objectnaam, afstand, diameter
FROM hemelobjecten
WHERE NOT EXISTS (
        SELECT reisnr
        FROM bezoeken b
        WHERE b.objectnaam = hemelobjecten.objectnaam
)
AND diameter < ANY (
        SELECT max(diameter)
        FROM hemelobjecten
)
ORDER BY diameter, objectnaam
```
## opgave
Maak een lijst van klanten die meer dan 2 keer een reis gemaakt hebben waarbij er geen bezoek was aan Jupiter.
## oplossing
```sql
SELECT klantnr, naam || ' ' || vnaam as klantnaam, COUNT(reisnr) aantalreizen
FROM deelnames INNER JOIN klanten USING(klantnr)
WHERE NOT EXISTS (
        SELECT reisnr
        FROM bezoeken
        WHERE objectnaam = 'Jupiter'
        AND deelnames.reisnr = bezoeken.reisnr
)
GROUP BY klantnr, klantnaam
HAVING COUNT(reisnr) > 2
```
## opgave
Geef de klantnr voor de klant met het meeste bezoeken aan de maan. Geef ook het aantal bezoeken.
Gebruik geen limit of top.
## oplossing
```sql
SELECT klantnr, COUNT(aantalbezoeken) AS count
FROM deelnames JOIN (
        SELECT reisnr, volgnr
        FROM bezoeken
        WHERE objectnaam = 'Maan'
        GROUP BY reisnr, volgnr) AS aantalbezoeken USING (reisnr)
GROUP BY klantnr
HAVING COUNT(reisnr) > 3
```
