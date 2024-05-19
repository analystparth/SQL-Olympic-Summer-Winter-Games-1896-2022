# Olympic Summer Winter Games (1896-2022)

**1.** **which team has won the maximum gold medals over the years.**
```
select top 1 a.team, COUNT(medal) medal_cnt
from athletes a left join athlete_events ae
on a.id = ae.athlete_id
where medal = 'Gold'
group by a.team
order by medal_cnt desc
```
**2.** **for each team print total silver medals and year in which they won maximum silver medal..output 3 columns
-- team,total_silver_medals, year_of_max_silver**
```
with cte as(
select a.team, COUNT(medal) total_silver_medals
from athletes a left join athlete_events ae
on a.id = ae.athlete_id
where medal = 'Silver'
group by a.team 
),
cte2 as(
select a.team, ae.year, COUNT(ae.medal) medal_silver_yearly
from athletes a left join athlete_events ae
on a.id = ae.athlete_id
where medal = 'Silver'
group by a.team , ae.year

), cte3 as(
select team,year from(
select *, rank()over(partition by team order by medal_silver_yearly desc, year asc) rnk
from cte2) as a
where rnk=1)

select cte.team, cte.total_silver_medals, cte3.year from cte join cte3 on cte.team = cte3.team ;
```
**3.** **which player has won maximum gold medals  amongst the players**
**which have won only gold medal (never won silver or bronze) over the years**
```
with cte as(
select name,medal 
from athletes a join athlete_events ae
on a.id = ae.athlete_id)
select top 1 name, count(medal) cnt
from cte 
where name not in (select distinct name from cte where medal in ('Silver','Bronze'))
and medal ='Gold'
group by name 
order by cnt desc;
```
**4.** **in each year which player has won maximum gold medal . Write a query to print year,player name 
--and no of golds won in that year . In case of a tie print comma separated player names.**
```
WITH CTE AS(
select ae.year,a.name, count(ae.medal) cnt
from athletes a join athlete_events ae
on a.id = ae.athlete_id 
where medal='Gold'
group by year,name
)
select year, STRING_AGG(name,' , ') name,cnt from 
(select *, RANK()over(partition by year order by cnt desc) rnk from CTE) a
where rnk =1
group by year, cnt;
```
**5.** **in which event and year India has won its first gold medal,first silver medal and first bronze medal
--print 3 columns medal,year,sport**
```
with cte as(
select ae.year,ae.medal, ae.sport
from athletes a join athlete_events ae
on a.id = ae.athlete_id 
where team = 'India' and medal ! = 'NA'
)

select medal, case when medal = 'Gold' then min(year)
				   when medal ='Silver' then min(year)
				   when medal ='Bronze' then min(year)
				   end as yr
from cte
group by medal;
```
 **6.** **find players who won gold medal in summer and winter olympics both.**
```
select a.name  
from athlete_events ae
inner join athletes a on ae.athlete_id=a.id
where medal='Gold'
group by a.name having count(distinct season)=2;

```

**7.** **find players who won gold, silver and bronze medal in a single olympics. print player name along with year.**
```
select name,year 
from athlete_events ae
inner join athletes a on ae.athlete_id=a.id
where medal != 'NA'
group by name,year 
having  count(distinct medal)=3 and count(distinct year) =1
```
**8.** **find players who have won gold medals in consecutive 3 summer olympics in the same event . Consider only olympics 2000 onwards. 
--Assume summer olympics happens every 4 year starting 2000. print player name and event name.**
```
with cte as(
select name, medal,event,year,season
from athlete_events ae
inner join athletes a on ae.athlete_id=a.id
where medal = 'Gold' and season = 'Summer' and year >=2000), cte2 as(
select name, event , lead(event,1)over(partition by name,event order by year) lead_event,lead(event,2)over(partition by name,event order by year) lead_event_2,
year, lead(year,2)over(partition by name,event order by year)lead_year
from cte)

select name, event from cte2 
where lead_year-year = 8 and event = lead_event and lead_event=lead_event_2
```
