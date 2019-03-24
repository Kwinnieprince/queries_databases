# extra 2

## opgave
Geef van elke speler het spelersnr, de naam en het verschil tussen zijn of haar jaar van toetreding en het gemiddeld jaar van toetreding van de spelers die in dezelfde plaats wonen. Sorteer van voor naar achter. Toon 3 getallen na de komma, maximaal 2 voor de komma; gebruik een cast functie.
## oplossing
```sql
select spelersnr, naam, voorletters, jaartoe - round((select avg(jaartoe) from spelers s2 where s.plaats = s2.plaats),3) as numeric
from spelers s
order by 1,2,3,4
```

## opgave
Toon alle mogelijke combinaties van de letters 'x' en 'y'. Tip zie handboek, voorbeeld met cijfers. (Wat is een cartesisch product?). Sorteer
## oplossing
```sql
select concat(t.column1, tt.column1) as "?column?"
from (values('x'), ('y')) as t cross join  (values('x'), ('y')) as tt
```

## opgave
Geef de wedstrijden die door spelers zijn gespeeld die in Leuven, Rotterdam of Leiden wonen. Sorteer van voor naar achter. Geef alle (*) informatie van wedstrijden.
## oplossing
```sql
select w.*
from spelers inner join wedstrijden w on spelers.spelersnr = w.spelersnr
where plaats like 'Rotterdam' or plaats like 'Leuven' or plaats like 'Leiden'
```
