# extra 3

## opgave
Maak een lijst met de reizen, gevolgd door de hemelobjecten wiens objectnaam met een A begint. Sorteer van voor naar achter. TIP: Gebruik CAST AS CHAR(10) voor conversies het reisnr en vertrekdatum.
## oplossing
```sql
SELECT reisnr::char(10) as rnr_en_naam, vertrekdatum::char(10) as dtm_en_naam, reisduur, prijs
from reizen
union
select objectnaam, satellietvan, afstand, diameter
FROM hemelobjecten
where objectnaam like 'A%'
order by rnr_en_naam
```

## opgave
Welke reizen hebben vijf hemelobjecten als reisdoel? Hou er rekening mee dat een reis op verschillende keren hetzelfde hemelobject kan bezoeken. Verduidelijking: hoe vaak is de reis uit de uitvoer bezocht? Sorteer.
## oplossing
```sql
select reisnr
from reizen
inner join bezoeken using (reisnr) inner join hemelobjecten using (objectnaam)
group by reisnr
having count(reisnr) = 5
```

## opgave
Maak een lijst met klantgegevens van de personen die nog nooit op Phobos op bezoek geweest zijn. Maak expliciet gebruik van de 'uitgezonderd' set operator. (Is dit de meest efficiënte oplossing?) Sorteer van voor naar achter.
## oplossing
```sql

```
