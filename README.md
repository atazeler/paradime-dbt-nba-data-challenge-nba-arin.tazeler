# NBA Draft Analyses
## Introduction
Hello, thank you for being interested in my analyses of some NBA related data, more specifically on NBA draft. I won't be going into what NBA is but for those who are not very familiar with it, NBA draft is a yearly event where teams are selecting young, up and coming talents that declare their intentions to play in NBA. As a big fan of the league, I think the draft system is one of the main reasons why NBA is such a popular league not only in North America but also globally. 

While its not that simple basic system of the draft is like following: 
In the modern NBA by default every team gets 2 draft picks, one for the first round, another for the second round. The worse a team's record is in the year, more likely they will land a higher draft pick in that year's NBA draft. This way the league tries to have some sort of balance in the league so no single team can dominate the league for year after year since that team will be unlikely to find new star talent in the draft as they will be drafting players at lower spots while their current roster ages hence team starts to be less and less successful. 

For general managers (GM) and front offices (FO) of the teams draft is obviously very important and its not only for scouting new talent reasons, because you can trade draft picks with other picks as well as players. Ideally a GM tries to build a team where all of its players' peaks are more or less aligned so your window of championship is as long as possible. For example if you have a young promising core but some older star players, you may wanna rebuild your team by trading your star players for some high draft picks. But doing that you should also consider that a young player probably won't be contributing positively to the team immedately.

In my analyses below im mostly taking the role of a GM building a team and try to come up with some draft strategies.

### [Github Repo](https://github.com/atazeler/paradime-dbt-nba-data-challenge-nba-arin.tazeler/tree/master)
## Data Sources and DAG
![image](https://github.com/atazeler/paradime-dbt-nba-data-challenge-nba-arin.tazeler/assets/107131288/17edb9b6-7ae9-4b1b-b5ea-1e4db6fae474)
- I treated the dbt project as if i'm part of an organisation so i tried to build models so that intermediate models are more modular and can be used as singe source of truth for other presentation tables as well. 
- I didn't change the folder structure as the standard `stg`, `int`, `fact`/`mart`/`presentation` format is sufficient here.
- I didn't update the `.yml` files with new columns or tests since im the only one working on this.
- Existing `stg` models were sufficient so didn't touch them.

#### Intermediate models:
- `int_player_common_info`: This model has the static player info that doesn't change over seasons such as name, birth date, draft position, debut age, etc. Unique on `player_id`.
- `int_player_common_info_by_season`: This model has player information that changes from season to season (besides stats) such as age, nba tenure, salary and booleans on whether its the player's rookie season or last season. Unique on `player_id`, -`season` combination.
- `int_player_stats_by_season`: This model has player stats split by regular season and playoffs. Unique on `player_id`, `season` combination.
#### Presentation model:
- `presentation_players_by_season`: This model joins the 3 above intermediate models. Unique on `player_id`, `season` combination.

#### Tools Used
- **[Paradime](https://www.paradime.io/)** for SQL, dbt™.
- **[Snowflake](https://www.snowflake.com/)** for data storage and computing.
- **Sigma** for data visualization.

#### Macros used
- `convert_inch_to_cm`: A simple macro that converts heights of players from inch to cm since in cm it can be a numeric value. In the end i didn't use any height as filter or field in my analyses.
- `draft_segmentation`: A simple macro that maps the draft rounds and positions of players into 4. Lottery pick: 1st round 1-15 picks, Late first round: 1st round 16-30 picks, Second round: Second round from 31 to 60, Other/Undrafted: Rest.

## Analyses
### Performance per draft pick number
First things first i let my inner fan to take over and check the performance of players by their draft pick number. Obviously teams would want the highest pick possible but its still interesting to check. We should expect something like higher the pick higher the performance, which is proxied by the avg. points per game and avg. plus minus (a metric that measures impact of the player, read more [here](https://en.wikipedia.org/wiki/Plus%E2%80%93minus)).

![image](https://github.com/atazeler/paradime-dbt-nba-data-challenge-nba-arin.tazeler/assets/107131288/5c7e63ca-b412-45ec-99fb-ba514306aaeb)

**Insights:**

The avg. PPG (blue bars) is gradually lower as the draft pick goes higher from 1 to 30, for plus minus (yellow bars) that's not the case really but average 1st pick plus minus is still the highest. What is quite interesting here is in both avg. PPG and plus minus 2nd pick is considerably lower than the 3rd pick. Does it mean that its more likely to be a bust (a player that is far below expectations) if you are a second pick? Maybe. But it might be due to some outliers as well. Michael Jordan who is arguably the greatest player of all time was a 3rd pick in 1984 draft while 2nd pick in the same draft was Sam Bowie who is considered a bust. Pretty much same thing for 2018 draft where 3rd pick is Luka Doncic who is one of the best players in today's NBA while 2nd pick was Marvin Bagley Jr., another bust. **So if you are a GM and you land the 2nd pick in the next draft, be very very careful on your scouting.**

### When do players reach their peak
Players don't start performing at their peak level as soon as they enter the league. GMs should take into account of some years until player reaches his full potential, but what year is that? We can use the `nba_tenure` field to check player's stats over their tenure.

![image](https://github.com/atazeler/paradime-dbt-nba-data-challenge-nba-arin.tazeler/assets/107131288/429414b2-7457-465b-b46d-d016318d9018)

![image](https://github.com/atazeler/paradime-dbt-nba-data-challenge-nba-arin.tazeler/assets/107131288/257bec6f-693d-4780-a316-73777114b8f6)


**Insights:**
The above graphs show the average Points (green), Rebounds (blue) and Assists (yellow) per game over year 1 to 20 as well as the % difference between 2 years. It looks like on all 3 main statistics players reach their peaks around year 6. Players increase their stats from year 1 to year 2 the most which then their growth in terms of stats diminishes. By the way the fact that stats went up from year 18 to 19 and 19 to 20 might be solely because of Lebron James :eyes:. Since there are not that many players who played that long Lebron himself is very likely skewing those tenures.

### Does it differ per draft segment?

So players reach their peak around year 6, but is there significant difference between their draft position, do higher picks reach their peaks earlier? We can split the above graph and check it.
![image](https://github.com/atazeler/paradime-dbt-nba-data-challenge-nba-arin.tazeler/assets/107131288/0b923730-56b1-4ed0-8cfe-d03744b2675c)
![image](https://github.com/atazeler/paradime-dbt-nba-data-challenge-nba-arin.tazeler/assets/107131288/6c160136-c6dc-47a3-bd32-bd563aa5b176)
**Insights:**
It looks like when we split the players by their draft segmentation every segment is still reaching their peak on average on their 6th year in NBA. There are couple interesting things though:

- For lottery picks: they don't seem to improve their stats as much as late 1st round or 2nd round picks. 1st year growth for them is around 16% in points and rebounds while its more around 20-25% level for late 1st round and 2nd round picks.
- Again for lottery picks: there is this interesting increase in assist numbers in year 6, it might indicate that star players selected at high picks learning to share the ball.

First point could imply that lottery picks are more NBA ready so their growth is less compared to other draft picks. It could also mean that from the year 1 they are the primary option in their team so they are put in a position to score more while late 1st round picks and 2nd round picks have to prove themselves by taking a smaller role and then growing to a solid player. 

### Does it differ per debut age?

Another split that might be interesting to check is whether players that spend more years in college basketball (or in other leagues) have any difference in their growth profiles or not. Usually in recent years players tend to stay in college only 1 year (one and done) before declaring for NBA draft. The ones that stay longer in college lose couple years of NBA experience (and NBA money) but in the media they are portrayed as more NBA ready compared to more raw talent of one and done players. Now we don't have data on how many years they spent in college but we can use the `debut_age` as a proxy for that.
![image](https://github.com/atazeler/paradime-dbt-nba-data-challenge-nba-arin.tazeler/assets/107131288/50dba359-fc34-4ebc-82ca-05759e43d01f)
![image](https://github.com/atazeler/paradime-dbt-nba-data-challenge-nba-arin.tazeler/assets/107131288/36580320-7fa1-4da2-83cf-af566fbdc98a)
![image](https://github.com/atazeler/paradime-dbt-nba-data-challenge-nba-arin.tazeler/assets/107131288/0c6b9478-9c32-4455-80ff-4f61aaeb1dcb)
![image](https://github.com/atazeler/paradime-dbt-nba-data-challenge-nba-arin.tazeler/assets/107131288/8578f123-0613-47c4-a29b-00b889b7dd61)
![image](https://github.com/atazeler/paradime-dbt-nba-data-challenge-nba-arin.tazeler/assets/107131288/24b9a64a-d867-4d4b-93d3-5f0839b6147e)
**Insights:**
Above graphs show the % difference in points, rebounds and assists over season split by the debut age as a proxy for years in college. There is no real trend like bigger the debut age smaller the growth over years but there might be some valuable insights here.

- While we've seen that on average players reach their peak at year 6 but for players that debuted at age 19 (and maybe even for age 20) that moves a bit towards year 7 where on average all 3 stats see an increase at year 7 too.
- Common narrative is that one and done (debut age 19) players that come to the league have more room to grow, but actually in their first year their stats increase is less than or equal to other debut ages.
- Players with debut age 23 (considered old to debut) actually on average grow the most compared to other debut ages in their first year.

### Stats per Draft Segment and Debut Age
Finally we can have a look at the table below that shows the avg. points, rebound and assists per draft segment and debut age. The GMs have of course way more information and a lot of scouting reports over players they consider but this could help them decide on a player over another if they are really close in reports.
![image](https://github.com/atazeler/paradime-dbt-nba-data-challenge-nba-arin.tazeler/assets/107131288/59fed850-21e8-450d-93dc-6cdbfdadfeb3)
**Insights:**
Basically this table shows that for all segments younger players (debut age 19 and 20) tend to put on better stats than others.

## Conclusion
I've sliced and diced the data in different ways to see if there are clear differences between player's growth profile over things like draft segment or debut age. While there is no glaring differences we can still come up with some conclusion.

- If you have the 2nd draft pick in the draft be very very careful as it looks like its a tricky place to draft. Players selected 2nd overall on average have worse stats than players selected 3rd overall. (Jordan, Doncic, Tatum are clear examples)
- Lottery picks tend to grow less to their 2nd year compared to others, if your lottery pick is not performing that well in the first year it's more likely that he's a bust.
- Contrary to common narrative players that debut in an older age do grow as much as younger ones, if not more, over their first couple years. If you are trying to maximize your success/championship window you may want your player that debuted at age 23 to be in his 3rd, 4th year to contribute.
- If you are just looking to draft the best available player, younger players tend to perform better, at least in terms of stats.
