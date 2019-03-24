# Subqueries 1 verbreding

## opgave
Geef een lijst van alle hemelobjecten die meer keer bezocht gaan worden dan Jupiter. (onafhankelijk van het aantal deelnames)
Sorteer op objectnaam.
## oplossing
```sql
select b.objectnaam, diameter
from hemelobjecten inner join bezoeken b on hemelobjecten.objectnaam = b.objectnaam
group by b.objectnaam, diameter
having count(b.objectnaam) > (select count(*) from bezoeken bez where bez.objectnaam like 'Jupiter')
order by b.objectnaam
```

## opgave
Geef het hemellichaam dat het laatst bezocht is. 
Gebruik hiervoor de laatste vertrekdatum van de reis en laatste volgnummer van bezoek. Tip: gebruik hiervoor een rij-subquery.
Gebruik geen limit of top.
## oplossing
```sql
SELECT objectnaam FROM bezoeken
WHERE
(reisnr, volgnr) = (
	SELECT reisnr, MAX(volgnr)
	FROM bezoeken
	WHERE reisnr = (
		SELECT reisnr
		FROM reizen
		WHERE vertrekdatum = (
			SELECT MAX(vertrekdatum) AS vertrekdatum
			FROM reizen
			)
		)
	GROUP BY reisnr
	)
```

## opgave
Geef het reisnr, de prijs en vertrekdatum van de reis met de hoogste gemiddelde verblijfsduur op een hemelobject (=som van de verblijfsduur / aantal bezoeken per reis).
## oplossing
```sql
SELECT r.reisnr, r.prijs, r.vertrekdatum 
FROM	(
	SELECT reisnr, SUM(verblijfsduur)/COUNT(volgnr) AS gem_duur
	FROM bezoeken
	GROUP BY reisnr
	) AS gemiddelde_duur
INNER JOIN reizen AS r ON r.reisnr = gemiddelde_duur.reisnr
WHERE gem_duur = (
	SELECT MAX(gem_duur) 
	FROM (
		SELECT reisnr, SUM(verblijfsduur)/COUNT(volgnr) AS gem_duur
		FROM bezoeken
		GROUP BY reisnr
		) AS gemiddelde_duur
	)
```

## opgave
Geef de planeet (draait dus rond de zon) met de meeste satellieten.
Sorteer op objectnaam.
## oplossing
```sql
SELECT objectnaam 
FROM (
	SELECT h.satellietvan AS objectnaam, COUNT(h.objectnaam) AS aantal_manen
	FROM hemelobjecten AS h
	WHERE h.satellietvan <> 'Zon'
	GROUP BY h.satellietvan
     	) AS aantal_manen
WHERE aantal_manen = (
	SELECT MAX(aantal_manen) 
	FROM 	(
		SELECT h.satellietvan AS objectnaam, COUNT(h.objectnaam) AS aantal_manen
		FROM hemelobjecten AS h
		WHERE h.satellietvan <> 'Zon'
		GROUP BY h.satellietvan
		) AS aantal_manen
	)
ORDER BY objectnaam
```

## opgave
Geef het op één na kleinste hemellichaam.
Je kan dit vinden door handig gebruik te maken van expliciete joins en een doorsnedevoorwaarde. Tip: probeer eerst een lijst te krijgen van alle hemelobjecten en het aantal hemellichaam dat kleiner is dan dat hemelobject.
## oplossing
```sql
SELECT objectnaam, aantalkleiner
FROM 	(
	SELECT hg.objectnaam, COUNT(hk.objectnaam) AS aantalkleiner
	FROM hemelobjecten AS hg
	INNER JOIN hemelobjecten AS hk ON hg.diameter > hk.diameter
	GROUP BY hg.objectnaam
	) AS h
WHERE aantalkleiner = 1
```
