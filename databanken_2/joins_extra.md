# Joins extra

## opgave

Hoeveel kilometers heeft iedereen in totaal gevlogen tot nu toe en hoeveel hebben ze hier in totaal voor betaald. Vermits we de posities van de planeten niet kennen, mag je de afstanden van de hemelobjecten direct gebruiken. Geef het totaal gespendeerde bedrag, de afgelegde kilometers, de prijs per kilometer en datum van hun laatste vlucht van al hun persoonlijke reizen. In het geval dat iemand niet op reis is geweest of geen kilometers gedaan heeft, toon je de boodschap 'veel geld voor niks of niet op reis geweest’ in de kolom prijs_per_kilometer.
Sorteer van voor naar achter.

## oplossing

```sql
select distinct k.klantnr, k.naam || ' ' || k.vnaam as naam, 
	(select sum(reizen.prijs) from klanten
	left outer join deelnames on(klanten.klantnr = deelnames.klantnr)
	inner join reizen on(reizen.reisnr = deelnames.reisnr)
	where k.klantnr = klanten.klantnr) as tot_bedrag,
	
	sum(h.afstand) as tot_afstand,
	
	case when 	(select sum(reizen.prijs) from klanten
				left outer join deelnames on(klanten.klantnr = deelnames.klantnr)
				inner join reizen on(reizen.reisnr = deelnames.reisnr)
				where k.klantnr = klanten.klantnr) = 0
				
				or
				
				sum(h.afstand) = 0
		then	'veel geld voor niks of niet op reis geweest'
		else
				
		((select sum(reizen.prijs) from klanten
		left outer join deelnames on(klanten.klantnr = deelnames.klantnr)
		inner join reizen on(reizen.reisnr = deelnames.reisnr)
		where k.klantnr = klanten.klantnr) / sum(h.afstand))::varchar 
		end
		
		as prijs_per_kilometer,
	
	(select max(reizen.vertrekdatum) from reizen
	 inner join deelnames on(deelnames.reisnr = reizen.reisnr)
	 where deelnames.klantnr = k.klantnr) as laatste_reis_datum
	
from klanten k
left outer join deelnames d on(k.klantnr = d.klantnr)
left outer join reizen r on(r.reisnr = d.reisnr)
left outer join bezoeken b on(b.reisnr = r.reisnr)
left outer join hemelobjecten h on(h.objectnaam = b.objectnaam)
group by 1,2, 3
order by 1,2,3,4,5
```

## opgave

Geef de volledige frequentietabel voor de diameters van de hemelobjecten (frequentie: hoeveel ojecten zijn er met de gegeven diameter, cumulatieve Frequentie, relatieve frequentie, Relatieve cumulatieve frequentie). Let op de datatypes en de precisie, gebruik CAST, rond niet af. Sorteer op diameter.

## oplossing

```sql
select diameter, 
       count(diameter) as f, sum(count(diameter)) over w1 as cf,
       to_char(float8 (count(diameter)*100::float / (select count(diameter)  from hemelobjecten)), 'FM99.00') as rf,
       to_char( float8 (sum(count(diameter)) over(order by diameter)*100::float / (select count(diameter)  from hemelobjecten)),
       'FM990.90')  as crf
from hemelobjecten
group by diameter
window w1 as ( order by diameter)
order by 1
```

## opgave

Geef voor elke reis het aantal klanten waarvan de naam niet met een 'G' begint en waarvan de periode van de geboortedatum van de klant tot de vertrekdatum van de reis overlapt met de huidige datum en 50 jaar verder (gebruik hiervoor de gepaste operator: OVERLAPS).
Indien er op de reis hemelobjecten worden bezocht waarvan de tweede letter van het hemelobject voorkomt in de naam van het hemelobject waarvan dit bezocht hemelobject een satelliet is, dan wordt deze reis genegeerd.
Sorteer op reisnr.

# oplossing

```sql
select r.reisnr, count(k.klantnr)
from klanten k 
inner join deelnames d on(k.klantnr = d.klantnr)
right outer join reizen r on (r.reisnr = d.reisnr)
where 
	(k.naam not like 'G%' or k.naam is null) 
	and ((k.geboortedatum, r.vertrekdatum)
	overlaps (now(), now()+ interval '50 years'))
	
	and r.reisnr not in (select reizen.reisnr
					from reizen 
					inner join bezoeken on(reizen.reisnr = bezoeken.reisnr)
					inner join hemelobjecten sat on (sat.objectnaam = bezoeken.objectnaam)
					inner join hemelobjecten main  on(sat.satellietvan = main.objectnaam)
					where main.objectnaam LIKE '%' || substring(sat.objectnaam, 2, 1) || '%'  )
	
	
group by r.reisnr
```

(Ik heb hier zo te zien niet gesorteerd, maar ze vragen het wel. Als deze oplossing fout rekent kan je misschien dat proberen)
