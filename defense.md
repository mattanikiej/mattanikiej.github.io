# Defense Quality: A New Way of Measuring Defensive Impact in the NHL

One of the hardest things to measure in the NHL is the impact good defense has on a game. There aren't many counting stats that can directly point to good defense, rather, good defense often shows up as a lack of production from an offensive player. There's been many attempts to create solid defensive stats, and many excellent ones as well. I want to pose a new stat called Defense Quality, that measures a player's impact compared to the rest of the league. Defense Quality addresses gaps in current defense metrics by focusing on the amount of scoring chances a player gives up rather than goals, and rewarding efficiency rather than single game performance.

## Current Stats
Defense Quality is not the first stat that measures defensive impact, and not even the first one focused on scoring chances. Corsi and Fenwick do a great job of showing a player's impact, and xG models are incredible at showing tangible results. [Dom Luszczyszyn](https://www.nytimes.com/athletic/author/dom-luszczyszyn/) from The Athletic has a great [Defensive Rating](https://www.nytimes.com/athletic/4396412/2023/04/12/nhl-advanced-stats-offensive-defensive-rating/) metric that is based largely on xG and per game stats. It's measured in goals, and can be translated into a WAR metric. It is an excellent stat for measuring the results of a game, and this is where Defense Quality differs from Defensive Rating.

There are three main gaps in Defense Rating that Defense Quality aims to fill. 

### 1. Defensive Rating does not take into account how a goal was scored.
While all goals are not necessarily treated equal as it uses G - xG, it still only focuses on the outcome of a play. Defense Quality aims to fix this by focusing on the quality of scoring chances opposing teams have, while the player is on the ice.

### 2. Defensive Rating focuses on per game statistics rather than efficiency.
Per game statistics are good when focusing on positive stats such as, hits, blocks, etc. However, a top pair defenseman is going to be playing more minutes against better opposition, than a bottom pair defenseman, which will inevitably inflate their negative stats such as goals, chances, etc. This makes it difficult to correctly scale a player's "per 60" impact. However, I've found that incorporating the player's ice time and situation has pretty successfully addressed these concerns.

### 3. Defensive Rating neglects a goalie's impact on goals.
Using xG is great for analyzing the results of a game and a player's impact. However, it can unfairly punish good defense because of poor goaltending. Defense Quality does the opposite, and ignores the result in favor of focusing on the quality of the scoring chance itself.

Together, Defensive Rating and Defense Quality give a much more nuanced understanding of a player's impact on defense. I want to emphasize that Defense Quality is not meant to replace Defensive Rating. Fundamentally, they measure different things. Hockey games are won by scoring goals and not generating good opportunities, so it's important to analyze the results of a game like Defensive Rating does. Defense Quality is meant to address gaps in the information of Defensive Rating, and provide a further in depth analysis of the player's impact on defense. Since we want numbers accurate to today, we will be basing this metric on data from the 2024-25 season.


```python
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
```


```python
data = pd.read_csv("/Users/mattanikiej/nhl-stats/data/2024-25/skaters.csv")

data.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>playerId</th>
      <th>season</th>
      <th>name</th>
      <th>team</th>
      <th>position</th>
      <th>situation</th>
      <th>games_played</th>
      <th>icetime</th>
      <th>shifts</th>
      <th>gameScore</th>
      <th>...</th>
      <th>OffIce_F_xGoals</th>
      <th>OffIce_A_xGoals</th>
      <th>OffIce_F_shotAttempts</th>
      <th>OffIce_A_shotAttempts</th>
      <th>xGoalsForAfterShifts</th>
      <th>xGoalsAgainstAfterShifts</th>
      <th>corsiForAfterShifts</th>
      <th>corsiAgainstAfterShifts</th>
      <th>fenwickForAfterShifts</th>
      <th>fenwickAgainstAfterShifts</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>8478047</td>
      <td>2024</td>
      <td>Michael Bunting</td>
      <td>NSH</td>
      <td>L</td>
      <td>other</td>
      <td>76</td>
      <td>2237.0</td>
      <td>37.0</td>
      <td>26.19</td>
      <td>...</td>
      <td>7.28</td>
      <td>10.09</td>
      <td>72.0</td>
      <td>87.0</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>8478047</td>
      <td>2024</td>
      <td>Michael Bunting</td>
      <td>NSH</td>
      <td>L</td>
      <td>all</td>
      <td>76</td>
      <td>70819.0</td>
      <td>1474.0</td>
      <td>43.70</td>
      <td>...</td>
      <td>161.54</td>
      <td>187.75</td>
      <td>3221.0</td>
      <td>3522.0</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>8478047</td>
      <td>2024</td>
      <td>Michael Bunting</td>
      <td>NSH</td>
      <td>L</td>
      <td>5on5</td>
      <td>76</td>
      <td>59813.0</td>
      <td>1294.0</td>
      <td>43.70</td>
      <td>...</td>
      <td>112.73</td>
      <td>122.08</td>
      <td>2661.0</td>
      <td>2707.0</td>
      <td>0.71</td>
      <td>1.71</td>
      <td>19.0</td>
      <td>43.0</td>
      <td>16.0</td>
      <td>31.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>8478047</td>
      <td>2024</td>
      <td>Michael Bunting</td>
      <td>NSH</td>
      <td>L</td>
      <td>4on5</td>
      <td>76</td>
      <td>6.0</td>
      <td>2.0</td>
      <td>2.58</td>
      <td>...</td>
      <td>0.20</td>
      <td>0.17</td>
      <td>4.0</td>
      <td>11.0</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>8478047</td>
      <td>2024</td>
      <td>Michael Bunting</td>
      <td>NSH</td>
      <td>L</td>
      <td>5on4</td>
      <td>76</td>
      <td>8763.0</td>
      <td>141.0</td>
      <td>36.88</td>
      <td>...</td>
      <td>23.81</td>
      <td>2.60</td>
      <td>311.0</td>
      <td>54.0</td>
      <td>0.00</td>
      <td>0.01</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>1.0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 154 columns</p>
</div>



## Defense Quality Formula

So what is Defense Quality? It is a weighted average of a player's defensive stats. There are 16 total and they're separated into 6 groups. Each gro'up's total weight, is the sum of the individual statistics in that group.

### 1. Ice Time (5%)
* Ice Time - 5%
 
This is used to weight a player's ice time, so as not to over reward great shifts in small sample sizes.
 
### 2. Shot Suppression (27%)
* High Danger Shots Against - 15%
* Medium Danger Shots Against - 10%
* Low Danger Shots Against - 1%
* Shots Against - 1%

This has the highest weight and importance in Defense Quality. We want to measure the quality of offense the opposition has, and therefore Defense Quality is very heavily focused on suppressing scoring chances. Scoring chances are much more tangible than xG, and much easier to understand. Although this doesn't address the biggest issue with the blackbox of xG sometimes (expected by whom?) since there are a million and one ways to define the danger of scoring chances, it still provides good information on preventing offense. Essentially, it's a more in depth Corsi Against.

For those wondering, this dataset used [MoneyPuck's formula](https://moneypuck.com/glossary.htm) since it's very easy to understand: High Danger is a shot with >= 20% chance of a goal, 20% > Medium >= 8%, 8% > Low. [Natural Stat Trick has a danger calculation as well](https://www.naturalstattrick.com/glossary.php?teams).
 
### 3. Physical Defense (15%)
* Blocked Shots - 15%

Regardless of the quality of the chance, a blocked shot is potentially as good a save. Anytime a defenseman can stop a puck from getting to the net is good defense and must be rewarded.
 
### 4. Puck Management (20%)
* Takeaways - 5%
* Giveaways - 5%
* Defensive Zone Giveaways - 10%

A giveaway is as disastrous as a takeaway is good, which is why they're weighted the same. A giveaway in the defensive zone is a blunder that can cost games. 10% may seem low for that reason, but keep in mind it's getting punished twice (Giveaway + Defensive Zone Giveaway), and potentially a third time if it leads to a scoring chance.
 
### 5. Shift Quality (17%)
* Defensive Zone Starts - 5%
* Defensive Zone Shift Ends - 5%
* Offensive Zone Shift Ends - 5%
* Neutral Zone Shift Ends - 1%
* Fly Shift Ends - 1%

Being able to break the puck out is good defense. That's why starting in your defensive zone, and ending in the offensive zone gets rewarded. Understanding where a shift starts and ends also lets us imply some things that can't be measured. Starting in your defensive zone often implies good defense, as the coach trusts the player's defensive impact. Ending in the offensive zone, implies a good and efficient break out, as the player wasn't gassed yet and didn't come to the bench. Ending in the neutral zone or on the fly is rewarded, but might have taken a while so it's not as rewarding as a quick transition from defense to offense. Is this a perfect way of measuring a breakout? No, but the NHL Edge data does not have breakout statistics and impact, and I've found this is a good work around that fits in 90% of situations.
  
### 6. Penalty Differential (16%)
* Penalties Taken - 15%
* Penalties Drawn - 1%

Taking a penalty is often due to lazy defense or getting beat. However it happened, a penalty kill puts the player's team in an extremely dangerous situation is treated as such. Conversely, drawing penalties usually happens during good offense, and it unfairly skewed towards offensive players. For example, when weighted equally, Jack Hughes was a top 5 defensive forward. It's still rewarded, but at a much lower weight than taking a penalty is penalized.



```python
# All stats needed for Defensive Rating calculation
defensive_stats = [
    # Player Info
    'name',
    'team',
    'position',
    'games_played',
    
    # Ice time responsibility
    'icetime',

    # Shot Suppression
    'OnIce_A_highDangerShots',
    'OnIce_A_mediumDangerShots',
    'OnIce_A_lowDangerShots',
    'OnIce_A_shotAttempts',
    
    # Physical Defense
    'shotsBlockedByPlayer',
    
    # Puck Management
    'I_F_takeaways',
    'I_F_giveaways',
    'I_F_dZoneGiveaways',
    
    # Shift Quality
    'I_F_dZoneShiftStarts',
    'I_F_dZoneShiftEnds',
    'I_F_oZoneShiftEnds',
    'I_F_neutralZoneShiftEnds',
    'I_F_flyShiftEnds',
    
    # Penalty Differential
    'penalties',
    'penaltiesDrawn',
    
]

print(f"Total stats used: {len(defensive_stats) - 4}")

```

    Total stats used: 16



```python
data_5on5 = data[data['situation'] == '5on5']

data_5on5.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>playerId</th>
      <th>season</th>
      <th>name</th>
      <th>team</th>
      <th>position</th>
      <th>situation</th>
      <th>games_played</th>
      <th>icetime</th>
      <th>shifts</th>
      <th>gameScore</th>
      <th>...</th>
      <th>OffIce_F_xGoals</th>
      <th>OffIce_A_xGoals</th>
      <th>OffIce_F_shotAttempts</th>
      <th>OffIce_A_shotAttempts</th>
      <th>xGoalsForAfterShifts</th>
      <th>xGoalsAgainstAfterShifts</th>
      <th>corsiForAfterShifts</th>
      <th>corsiAgainstAfterShifts</th>
      <th>fenwickForAfterShifts</th>
      <th>fenwickAgainstAfterShifts</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2</th>
      <td>8478047</td>
      <td>2024</td>
      <td>Michael Bunting</td>
      <td>NSH</td>
      <td>L</td>
      <td>5on5</td>
      <td>76</td>
      <td>59813.0</td>
      <td>1294.0</td>
      <td>43.70</td>
      <td>...</td>
      <td>112.73</td>
      <td>122.08</td>
      <td>2661.0</td>
      <td>2707.0</td>
      <td>0.71</td>
      <td>1.71</td>
      <td>19.0</td>
      <td>43.0</td>
      <td>16.0</td>
      <td>31.0</td>
    </tr>
    <tr>
      <th>7</th>
      <td>8480950</td>
      <td>2024</td>
      <td>Ilya Lyubushkin</td>
      <td>DAL</td>
      <td>D</td>
      <td>5on5</td>
      <td>80</td>
      <td>70786.0</td>
      <td>1535.0</td>
      <td>10.38</td>
      <td>...</td>
      <td>121.70</td>
      <td>119.55</td>
      <td>2715.0</td>
      <td>2625.0</td>
      <td>8.56</td>
      <td>0.32</td>
      <td>155.0</td>
      <td>9.0</td>
      <td>123.0</td>
      <td>6.0</td>
    </tr>
    <tr>
      <th>12</th>
      <td>8477369</td>
      <td>2024</td>
      <td>Carson Soucy</td>
      <td>NYR</td>
      <td>D</td>
      <td>5on5</td>
      <td>75</td>
      <td>69895.0</td>
      <td>1609.0</td>
      <td>8.07</td>
      <td>...</td>
      <td>95.12</td>
      <td>95.00</td>
      <td>2385.0</td>
      <td>2269.0</td>
      <td>5.71</td>
      <td>1.21</td>
      <td>107.0</td>
      <td>19.0</td>
      <td>75.0</td>
      <td>15.0</td>
    </tr>
    <tr>
      <th>17</th>
      <td>8481518</td>
      <td>2024</td>
      <td>Nolan Foote</td>
      <td>NJD</td>
      <td>L</td>
      <td>5on5</td>
      <td>7</td>
      <td>4075.0</td>
      <td>100.0</td>
      <td>1.92</td>
      <td>...</td>
      <td>11.78</td>
      <td>9.15</td>
      <td>283.0</td>
      <td>217.0</td>
      <td>0.09</td>
      <td>0.02</td>
      <td>4.0</td>
      <td>2.0</td>
      <td>3.0</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>22</th>
      <td>8477964</td>
      <td>2024</td>
      <td>Ivan Barbashev</td>
      <td>VGK</td>
      <td>C</td>
      <td>5on5</td>
      <td>70</td>
      <td>64523.0</td>
      <td>1217.0</td>
      <td>49.58</td>
      <td>...</td>
      <td>108.42</td>
      <td>103.12</td>
      <td>2432.0</td>
      <td>2368.0</td>
      <td>0.90</td>
      <td>0.33</td>
      <td>23.0</td>
      <td>21.0</td>
      <td>17.0</td>
      <td>13.0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 154 columns</p>
</div>




```python
defense_5on5 = data_5on5[defensive_stats]

defense_5on5.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>team</th>
      <th>position</th>
      <th>games_played</th>
      <th>icetime</th>
      <th>OnIce_A_highDangerShots</th>
      <th>OnIce_A_mediumDangerShots</th>
      <th>OnIce_A_lowDangerShots</th>
      <th>OnIce_A_shotAttempts</th>
      <th>shotsBlockedByPlayer</th>
      <th>I_F_takeaways</th>
      <th>I_F_giveaways</th>
      <th>I_F_dZoneGiveaways</th>
      <th>I_F_dZoneShiftStarts</th>
      <th>I_F_dZoneShiftEnds</th>
      <th>I_F_oZoneShiftEnds</th>
      <th>I_F_neutralZoneShiftEnds</th>
      <th>I_F_flyShiftEnds</th>
      <th>penalties</th>
      <th>penaltiesDrawn</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2</th>
      <td>Michael Bunting</td>
      <td>NSH</td>
      <td>L</td>
      <td>76</td>
      <td>59813.0</td>
      <td>47.0</td>
      <td>127.0</td>
      <td>481.0</td>
      <td>923.0</td>
      <td>17.0</td>
      <td>11.0</td>
      <td>51.0</td>
      <td>11.0</td>
      <td>79.0</td>
      <td>191.0</td>
      <td>201.0</td>
      <td>166.0</td>
      <td>736.0</td>
      <td>26.0</td>
      <td>25.0</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Ilya Lyubushkin</td>
      <td>DAL</td>
      <td>D</td>
      <td>80</td>
      <td>70786.0</td>
      <td>49.0</td>
      <td>165.0</td>
      <td>637.0</td>
      <td>1196.0</td>
      <td>109.0</td>
      <td>17.0</td>
      <td>103.0</td>
      <td>67.0</td>
      <td>208.0</td>
      <td>193.0</td>
      <td>202.0</td>
      <td>168.0</td>
      <td>972.0</td>
      <td>15.0</td>
      <td>8.0</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Carson Soucy</td>
      <td>NYR</td>
      <td>D</td>
      <td>75</td>
      <td>69895.0</td>
      <td>48.0</td>
      <td>137.0</td>
      <td>643.0</td>
      <td>1154.0</td>
      <td>88.0</td>
      <td>17.0</td>
      <td>71.0</td>
      <td>47.0</td>
      <td>172.0</td>
      <td>198.0</td>
      <td>178.0</td>
      <td>175.0</td>
      <td>1058.0</td>
      <td>18.0</td>
      <td>4.0</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Nolan Foote</td>
      <td>NJD</td>
      <td>L</td>
      <td>7</td>
      <td>4075.0</td>
      <td>1.0</td>
      <td>8.0</td>
      <td>26.0</td>
      <td>49.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>8.0</td>
      <td>13.0</td>
      <td>15.0</td>
      <td>10.0</td>
      <td>62.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>22</th>
      <td>Ivan Barbashev</td>
      <td>VGK</td>
      <td>C</td>
      <td>70</td>
      <td>64523.0</td>
      <td>52.0</td>
      <td>127.0</td>
      <td>607.0</td>
      <td>1091.0</td>
      <td>30.0</td>
      <td>18.0</td>
      <td>73.0</td>
      <td>23.0</td>
      <td>129.0</td>
      <td>176.0</td>
      <td>202.0</td>
      <td>194.0</td>
      <td>645.0</td>
      <td>4.0</td>
      <td>12.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
defense_5on5.describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>games_played</th>
      <th>icetime</th>
      <th>OnIce_A_highDangerShots</th>
      <th>OnIce_A_mediumDangerShots</th>
      <th>OnIce_A_lowDangerShots</th>
      <th>OnIce_A_shotAttempts</th>
      <th>shotsBlockedByPlayer</th>
      <th>I_F_takeaways</th>
      <th>I_F_giveaways</th>
      <th>I_F_dZoneGiveaways</th>
      <th>I_F_dZoneShiftStarts</th>
      <th>I_F_dZoneShiftEnds</th>
      <th>I_F_oZoneShiftEnds</th>
      <th>I_F_neutralZoneShiftEnds</th>
      <th>I_F_flyShiftEnds</th>
      <th>penalties</th>
      <th>penaltiesDrawn</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>920.000000</td>
      <td>920.000000</td>
      <td>920.000000</td>
      <td>920.000000</td>
      <td>920.000000</td>
      <td>920.000000</td>
      <td>920.000000</td>
      <td>920.000000</td>
      <td>920.000000</td>
      <td>920.000000</td>
      <td>920.000000</td>
      <td>920.000000</td>
      <td>920.000000</td>
      <td>920.000000</td>
      <td>920.000000</td>
      <td>920.000000</td>
      <td>920.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>51.330435</td>
      <td>42484.634783</td>
      <td>29.608696</td>
      <td>85.815217</td>
      <td>369.086957</td>
      <td>682.005435</td>
      <td>36.035870</td>
      <td>11.857609</td>
      <td>37.316304</td>
      <td>17.343478</td>
      <td>97.647826</td>
      <td>122.763043</td>
      <td>123.059783</td>
      <td>121.967391</td>
      <td>537.643478</td>
      <td>8.369565</td>
      <td>7.692391</td>
    </tr>
    <tr>
      <th>std</th>
      <td>29.600450</td>
      <td>27703.787686</td>
      <td>20.653026</td>
      <td>57.658355</td>
      <td>243.716931</td>
      <td>448.245700</td>
      <td>32.277999</td>
      <td>9.683363</td>
      <td>27.815476</td>
      <td>17.069322</td>
      <td>71.697465</td>
      <td>77.491397</td>
      <td>78.321758</td>
      <td>83.790031</td>
      <td>347.767046</td>
      <td>7.315994</td>
      <td>6.781844</td>
    </tr>
    <tr>
      <th>min</th>
      <td>1.000000</td>
      <td>70.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>21.000000</td>
      <td>13766.500000</td>
      <td>9.000000</td>
      <td>28.000000</td>
      <td>119.000000</td>
      <td>223.750000</td>
      <td>10.000000</td>
      <td>3.000000</td>
      <td>10.000000</td>
      <td>4.000000</td>
      <td>27.000000</td>
      <td>45.000000</td>
      <td>45.000000</td>
      <td>36.500000</td>
      <td>185.750000</td>
      <td>2.000000</td>
      <td>2.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>63.000000</td>
      <td>48113.000000</td>
      <td>31.000000</td>
      <td>91.000000</td>
      <td>411.000000</td>
      <td>754.000000</td>
      <td>29.000000</td>
      <td>11.000000</td>
      <td>37.000000</td>
      <td>13.000000</td>
      <td>98.500000</td>
      <td>136.000000</td>
      <td>141.000000</td>
      <td>131.500000</td>
      <td>618.500000</td>
      <td>7.000000</td>
      <td>7.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>78.000000</td>
      <td>65117.000000</td>
      <td>46.000000</td>
      <td>132.000000</td>
      <td>563.250000</td>
      <td>1038.500000</td>
      <td>50.000000</td>
      <td>18.000000</td>
      <td>57.000000</td>
      <td>23.000000</td>
      <td>151.000000</td>
      <td>186.000000</td>
      <td>185.000000</td>
      <td>191.000000</td>
      <td>799.500000</td>
      <td>12.000000</td>
      <td>12.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>85.000000</td>
      <td>104612.000000</td>
      <td>94.000000</td>
      <td>228.000000</td>
      <td>987.000000</td>
      <td>1779.000000</td>
      <td>179.000000</td>
      <td>50.000000</td>
      <td>140.000000</td>
      <td>104.000000</td>
      <td>314.000000</td>
      <td>311.000000</td>
      <td>302.000000</td>
      <td>318.000000</td>
      <td>1471.000000</td>
      <td>45.000000</td>
      <td>35.000000</td>
    </tr>
  </tbody>
</table>
</div>



So how are these stats being compared? There's a few things we have to do first. In order to reduce the impact of small sample sizes, we need to establish a metric that will make a player qualify for Defense Quality. Think quality starts in baseball, or minimum games for leaderboards. Defense Quality requires a player to average 10 minutes a game, and have played in at least half of the games that season. This ensures we are eliminating players that had great shifts in small roles, which will heavily skew the per 60 stats.

Also, the defensive role for forwards and defenseman is much different, so each stat is normalized by position, and will also be split accordingly later. For this exercise, all forwards will be treated as a forward, regardless of actual position. I understand there are different responsibilities, but we'll see later how forwards might need to have some different weights to their formulation anyway. 

To calculate Defense Quality we will take an approach very similar to my [first blog about finding the NHL's lucky charm](https://medium.com/hockey-harmony/the-nhls-lucky-charm-ccbb5df786d9) and build a composite z-score. This will bring everything together onto the same scale, and allow us to compare to the average player much better. I wanted to use this blog to highlight various statistical techniques and how they can be applied to hockey analytics, but I believe a composite z-score is most appropriate for this case.


```python
defense_5on5.loc[:, ['icetime_minutes']] = defense_5on5['icetime'] / 60

defense_10_minutes = defense_5on5[
    (defense_5on5['icetime_minutes'] / defense_5on5['games_played'] >= 10)
    & (defense_5on5['games_played'] >= 41)
]

defense_10_minutes.describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>games_played</th>
      <th>icetime</th>
      <th>OnIce_A_highDangerShots</th>
      <th>OnIce_A_mediumDangerShots</th>
      <th>OnIce_A_lowDangerShots</th>
      <th>OnIce_A_shotAttempts</th>
      <th>shotsBlockedByPlayer</th>
      <th>I_F_takeaways</th>
      <th>I_F_giveaways</th>
      <th>I_F_dZoneGiveaways</th>
      <th>I_F_dZoneShiftStarts</th>
      <th>I_F_dZoneShiftEnds</th>
      <th>I_F_oZoneShiftEnds</th>
      <th>I_F_neutralZoneShiftEnds</th>
      <th>I_F_flyShiftEnds</th>
      <th>penalties</th>
      <th>penaltiesDrawn</th>
      <th>icetime_minutes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>560.000000</td>
      <td>560.000000</td>
      <td>560.000000</td>
      <td>560.000000</td>
      <td>560.000000</td>
      <td>560.000000</td>
      <td>560.000000</td>
      <td>560.000000</td>
      <td>560.000000</td>
      <td>560.000000</td>
      <td>560.000000</td>
      <td>560.000000</td>
      <td>560.000000</td>
      <td>560.000000</td>
      <td>560.000000</td>
      <td>560.000000</td>
      <td>560.000000</td>
      <td>560.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>72.032143</td>
      <td>61838.539286</td>
      <td>43.216071</td>
      <td>124.976786</td>
      <td>536.716071</td>
      <td>991.246429</td>
      <td>52.085714</td>
      <td>17.625000</td>
      <td>54.703571</td>
      <td>25.301786</td>
      <td>142.600000</td>
      <td>175.857143</td>
      <td>175.992857</td>
      <td>179.337500</td>
      <td>775.741071</td>
      <td>11.814286</td>
      <td>11.055357</td>
      <td>1030.642321</td>
    </tr>
    <tr>
      <th>std</th>
      <td>10.909895</td>
      <td>14681.047776</td>
      <td>13.481642</td>
      <td>34.357759</td>
      <td>138.546995</td>
      <td>250.912161</td>
      <td>30.962436</td>
      <td>7.817147</td>
      <td>20.528735</td>
      <td>17.048914</td>
      <td>50.451386</td>
      <td>42.152517</td>
      <td>42.772686</td>
      <td>49.356996</td>
      <td>192.651991</td>
      <td>6.755464</td>
      <td>6.230700</td>
      <td>244.684130</td>
    </tr>
    <tr>
      <th>min</th>
      <td>41.000000</td>
      <td>26425.000000</td>
      <td>12.000000</td>
      <td>42.000000</td>
      <td>203.000000</td>
      <td>382.000000</td>
      <td>8.000000</td>
      <td>3.000000</td>
      <td>15.000000</td>
      <td>3.000000</td>
      <td>23.000000</td>
      <td>69.000000</td>
      <td>65.000000</td>
      <td>57.000000</td>
      <td>290.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>440.416667</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>67.000000</td>
      <td>52770.250000</td>
      <td>34.000000</td>
      <td>101.750000</td>
      <td>446.000000</td>
      <td>819.000000</td>
      <td>29.000000</td>
      <td>12.000000</td>
      <td>39.000000</td>
      <td>12.000000</td>
      <td>107.000000</td>
      <td>147.000000</td>
      <td>146.000000</td>
      <td>142.750000</td>
      <td>646.750000</td>
      <td>7.000000</td>
      <td>6.750000</td>
      <td>879.504167</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>76.000000</td>
      <td>61491.000000</td>
      <td>43.000000</td>
      <td>124.000000</td>
      <td>533.000000</td>
      <td>985.000000</td>
      <td>42.500000</td>
      <td>17.000000</td>
      <td>52.000000</td>
      <td>19.000000</td>
      <td>140.500000</td>
      <td>176.500000</td>
      <td>178.000000</td>
      <td>180.000000</td>
      <td>768.000000</td>
      <td>10.000000</td>
      <td>10.000000</td>
      <td>1024.850000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>81.000000</td>
      <td>70570.500000</td>
      <td>52.000000</td>
      <td>147.000000</td>
      <td>626.250000</td>
      <td>1158.000000</td>
      <td>69.000000</td>
      <td>22.000000</td>
      <td>67.000000</td>
      <td>36.000000</td>
      <td>172.250000</td>
      <td>205.000000</td>
      <td>205.000000</td>
      <td>215.000000</td>
      <td>889.500000</td>
      <td>15.250000</td>
      <td>15.000000</td>
      <td>1176.175000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>85.000000</td>
      <td>104612.000000</td>
      <td>94.000000</td>
      <td>228.000000</td>
      <td>987.000000</td>
      <td>1779.000000</td>
      <td>179.000000</td>
      <td>50.000000</td>
      <td>140.000000</td>
      <td>104.000000</td>
      <td>314.000000</td>
      <td>311.000000</td>
      <td>302.000000</td>
      <td>318.000000</td>
      <td>1471.000000</td>
      <td>45.000000</td>
      <td>35.000000</td>
      <td>1743.533333</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Normalize all defensive stats by ice time (per 60 minutes)
defense_normalized = defense_10_minutes.copy()

# Convert ice time to hours for per-60 calculations
icetime_hours = defense_normalized['icetime'] / 3600

# Calculate ice time per game (will be normalized by position)
defense_normalized['icetime_per_game'] = defense_normalized['icetime'] / defense_normalized['games_played']

# Normalize all stats
stats_to_normalize = [stat for stat in defensive_stats if stat not in ['icetime', 'icetime_per_game', 'games_played', 'name', 'team', 'position']]

for stat in stats_to_normalize:
    defense_normalized[f'{stat}_per60'] = defense_normalized[stat] / icetime_hours

# Standardize the per60 stats BY POSITION (position-specific z-scores)
per60_stats = [f'{stat}_per60' for stat in stats_to_normalize] + ['icetime_per_game'] + ['name', 'team', 'position']

# Transform C, L, R positions to F (Forward)
defense_normalized['position'] = defense_normalized['position'].replace({'C': 'F', 'L': 'F', 'R': 'F'})

for stat in per60_stats:
    if stat not in ['name', 'team', 'position']:
        # Calculate position-specific mean and std, then standardize within position
        defense_normalized[stat] = defense_normalized.groupby('position')[stat].transform(
            lambda x: (x - x.mean()) / x.std()
        )

# Show the normalized data
defense_normalized.head()

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>team</th>
      <th>position</th>
      <th>games_played</th>
      <th>icetime</th>
      <th>OnIce_A_highDangerShots</th>
      <th>OnIce_A_mediumDangerShots</th>
      <th>OnIce_A_lowDangerShots</th>
      <th>OnIce_A_shotAttempts</th>
      <th>shotsBlockedByPlayer</th>
      <th>...</th>
      <th>I_F_takeaways_per60</th>
      <th>I_F_giveaways_per60</th>
      <th>I_F_dZoneGiveaways_per60</th>
      <th>I_F_dZoneShiftStarts_per60</th>
      <th>I_F_dZoneShiftEnds_per60</th>
      <th>I_F_oZoneShiftEnds_per60</th>
      <th>I_F_neutralZoneShiftEnds_per60</th>
      <th>I_F_flyShiftEnds_per60</th>
      <th>penalties_per60</th>
      <th>penaltiesDrawn_per60</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2</th>
      <td>Michael Bunting</td>
      <td>NSH</td>
      <td>F</td>
      <td>76</td>
      <td>59813.0</td>
      <td>47.0</td>
      <td>127.0</td>
      <td>481.0</td>
      <td>923.0</td>
      <td>17.0</td>
      <td>...</td>
      <td>-1.021651</td>
      <td>0.177342</td>
      <td>-0.889666</td>
      <td>-1.388037</td>
      <td>0.477589</td>
      <td>0.411046</td>
      <td>-0.691069</td>
      <td>-0.119593</td>
      <td>2.001072</td>
      <td>1.887581</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Ilya Lyubushkin</td>
      <td>DAL</td>
      <td>D</td>
      <td>80</td>
      <td>70786.0</td>
      <td>49.0</td>
      <td>165.0</td>
      <td>637.0</td>
      <td>1196.0</td>
      <td>109.0</td>
      <td>...</td>
      <td>-0.466159</td>
      <td>2.662645</td>
      <td>2.187086</td>
      <td>1.514781</td>
      <td>-0.022019</td>
      <td>0.425182</td>
      <td>-0.953497</td>
      <td>0.612857</td>
      <td>0.235820</td>
      <td>-0.016663</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Carson Soucy</td>
      <td>NYR</td>
      <td>D</td>
      <td>75</td>
      <td>69895.0</td>
      <td>48.0</td>
      <td>137.0</td>
      <td>643.0</td>
      <td>1154.0</td>
      <td>88.0</td>
      <td>...</td>
      <td>-0.434646</td>
      <td>0.261487</td>
      <td>0.277254</td>
      <td>0.522278</td>
      <td>0.185438</td>
      <td>-0.079843</td>
      <td>-0.510809</td>
      <td>1.573115</td>
      <td>0.682549</td>
      <td>-0.901796</td>
    </tr>
    <tr>
      <th>22</th>
      <td>Ivan Barbashev</td>
      <td>VGK</td>
      <td>F</td>
      <td>70</td>
      <td>64523.0</td>
      <td>52.0</td>
      <td>127.0</td>
      <td>607.0</td>
      <td>1091.0</td>
      <td>30.0</td>
      <td>...</td>
      <td>-0.003845</td>
      <td>1.646416</td>
      <td>1.226863</td>
      <td>-0.490188</td>
      <td>-0.524670</td>
      <td>0.040595</td>
      <td>-0.058717</td>
      <td>-1.842489</td>
      <td>-1.174090</td>
      <td>-0.354278</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Egor Zamula</td>
      <td>PHI</td>
      <td>D</td>
      <td>63</td>
      <td>56910.0</td>
      <td>28.0</td>
      <td>102.0</td>
      <td>443.0</td>
      <td>863.0</td>
      <td>82.0</td>
      <td>...</td>
      <td>-0.586896</td>
      <td>-0.392575</td>
      <td>-0.122937</td>
      <td>-0.940028</td>
      <td>-0.096327</td>
      <td>0.182298</td>
      <td>0.653050</td>
      <td>0.727374</td>
      <td>-1.322950</td>
      <td>-1.252205</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 37 columns</p>
</div>




```python
plt.figure(figsize=(10, 6))
sns.histplot(defense_normalized['OnIce_A_highDangerShots_per60'], bins=20, kde=True)
plt.title(f'Normalized Distribution of High Danger Shots Per 60')
plt.xlabel('High Danger Shots Per 60')
plt.show()
```


    
![png](defense_files/defense_12_0.png)
    



```python
per60_df = defense_normalized[per60_stats]

per60_df.describe()

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>OnIce_A_highDangerShots_per60</th>
      <th>OnIce_A_mediumDangerShots_per60</th>
      <th>OnIce_A_lowDangerShots_per60</th>
      <th>OnIce_A_shotAttempts_per60</th>
      <th>shotsBlockedByPlayer_per60</th>
      <th>I_F_takeaways_per60</th>
      <th>I_F_giveaways_per60</th>
      <th>I_F_dZoneGiveaways_per60</th>
      <th>I_F_dZoneShiftStarts_per60</th>
      <th>I_F_dZoneShiftEnds_per60</th>
      <th>I_F_oZoneShiftEnds_per60</th>
      <th>I_F_neutralZoneShiftEnds_per60</th>
      <th>I_F_flyShiftEnds_per60</th>
      <th>penalties_per60</th>
      <th>penaltiesDrawn_per60</th>
      <th>icetime_per_game</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>5.600000e+02</td>
      <td>5.600000e+02</td>
      <td>5.600000e+02</td>
      <td>5.600000e+02</td>
      <td>5.600000e+02</td>
      <td>5.600000e+02</td>
      <td>5.600000e+02</td>
      <td>5.600000e+02</td>
      <td>5.600000e+02</td>
      <td>5.600000e+02</td>
      <td>5.600000e+02</td>
      <td>5.600000e+02</td>
      <td>5.600000e+02</td>
      <td>5.600000e+02</td>
      <td>5.600000e+02</td>
      <td>5.600000e+02</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>-1.871519e-16</td>
      <td>5.202188e-16</td>
      <td>5.963484e-16</td>
      <td>-1.122911e-15</td>
      <td>-1.649474e-16</td>
      <td>-1.173664e-16</td>
      <td>-2.474211e-16</td>
      <td>-4.472613e-16</td>
      <td>-3.219647e-16</td>
      <td>3.425831e-16</td>
      <td>-4.821540e-16</td>
      <td>-1.015061e-16</td>
      <td>1.358437e-15</td>
      <td>-3.489272e-17</td>
      <td>1.205385e-16</td>
      <td>6.978545e-17</td>
    </tr>
    <tr>
      <th>std</th>
      <td>9.991051e-01</td>
      <td>9.991051e-01</td>
      <td>9.991051e-01</td>
      <td>9.991051e-01</td>
      <td>9.991051e-01</td>
      <td>9.991051e-01</td>
      <td>9.991051e-01</td>
      <td>9.991051e-01</td>
      <td>9.991051e-01</td>
      <td>9.991051e-01</td>
      <td>9.991051e-01</td>
      <td>9.991051e-01</td>
      <td>9.991051e-01</td>
      <td>9.991051e-01</td>
      <td>9.991051e-01</td>
      <td>9.991051e-01</td>
    </tr>
    <tr>
      <th>min</th>
      <td>-2.687816e+00</td>
      <td>-2.668635e+00</td>
      <td>-3.033788e+00</td>
      <td>-3.046615e+00</td>
      <td>-2.167089e+00</td>
      <td>-2.255737e+00</td>
      <td>-2.336807e+00</td>
      <td>-2.624505e+00</td>
      <td>-2.785129e+00</td>
      <td>-2.850734e+00</td>
      <td>-2.698309e+00</td>
      <td>-3.282014e+00</td>
      <td>-2.678271e+00</td>
      <td>-1.702241e+00</td>
      <td>-2.151523e+00</td>
      <td>-2.902965e+00</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>-6.947883e-01</td>
      <td>-6.852118e-01</td>
      <td>-6.871908e-01</td>
      <td>-6.537320e-01</td>
      <td>-7.168265e-01</td>
      <td>-7.020389e-01</td>
      <td>-7.273986e-01</td>
      <td>-6.928125e-01</td>
      <td>-6.018277e-01</td>
      <td>-7.048456e-01</td>
      <td>-6.451592e-01</td>
      <td>-7.043824e-01</td>
      <td>-6.867701e-01</td>
      <td>-7.070728e-01</td>
      <td>-7.048873e-01</td>
      <td>-7.599645e-01</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>-1.037711e-02</td>
      <td>-7.539434e-03</td>
      <td>7.647053e-03</td>
      <td>-2.551437e-02</td>
      <td>-6.326369e-02</td>
      <td>-1.234383e-01</td>
      <td>-3.107141e-03</td>
      <td>-2.809440e-02</td>
      <td>-6.925049e-02</td>
      <td>2.578623e-02</td>
      <td>3.835170e-02</td>
      <td>8.386918e-04</td>
      <td>-7.484519e-02</td>
      <td>-1.946170e-01</td>
      <td>-9.305307e-02</td>
      <td>4.888170e-02</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>6.706291e-01</td>
      <td>6.180114e-01</td>
      <td>6.046613e-01</td>
      <td>7.045182e-01</td>
      <td>5.889090e-01</td>
      <td>6.210524e-01</td>
      <td>6.316330e-01</td>
      <td>6.516388e-01</td>
      <td>5.267702e-01</td>
      <td>6.612892e-01</td>
      <td>7.017442e-01</td>
      <td>6.527776e-01</td>
      <td>6.423752e-01</td>
      <td>4.451587e-01</td>
      <td>5.574157e-01</td>
      <td>7.360586e-01</td>
    </tr>
    <tr>
      <th>max</th>
      <td>3.744299e+00</td>
      <td>3.772877e+00</td>
      <td>4.038967e+00</td>
      <td>3.969688e+00</td>
      <td>4.371575e+00</td>
      <td>3.501708e+00</td>
      <td>3.405730e+00</td>
      <td>4.421493e+00</td>
      <td>4.322669e+00</td>
      <td>2.840199e+00</td>
      <td>2.574751e+00</td>
      <td>3.228487e+00</td>
      <td>3.746914e+00</td>
      <td>4.415791e+00</td>
      <td>4.170701e+00</td>
      <td>3.302208e+00</td>
    </tr>
  </tbody>
</table>
</div>




```python
# All stats needed for Defensive Rating calculation
defensive_weights = {
    # Ice time responsibility (higher is better - rewards players who play more)
    'icetime_per_game': 5,
    
    # Shot Suppression (Lower is better)
    'OnIce_A_highDangerShots_per60': -15,
    'OnIce_A_mediumDangerShots_per60': -10,
    'OnIce_A_lowDangerShots_per60': -1,
    'OnIce_A_shotAttempts_per60': -1,
    
    # Physical Defense (Higher is better)
    'shotsBlockedByPlayer_per60': 15,
    
    # Puck Management (Lower is better)
    'I_F_takeaways_per60': 5,
    'I_F_giveaways_per60': -5,
    'I_F_dZoneGiveaways_per60': -10,
    
    # Shift Quality (Higher is better, except defense ends)
    'I_F_dZoneShiftStarts_per60': 5,
    'I_F_dZoneShiftEnds_per60': -5,
    'I_F_oZoneShiftEnds_per60': 5,
    'I_F_neutralZoneShiftEnds_per60': 1,
    'I_F_flyShiftEnds_per60': 1,
    
    # Penalty Differential (Lower is better)
    'penalties_per60': -15,
    'penaltiesDrawn_per60': 1,
}
```

### Mathematical Formula

Defense Quality is calculated as a weighted composite z-score:

$$
DQ = \sum_{i=1}^{n} \frac{w_i}{\sum_{j=1}^{n}|w_j|} \cdot z_i
$$

Where:
- $DQ$ = Defense Quality score
- $n$ = 16 (total number of defensive statistics)
- $w_i$ = weight for statistic $i$ (can be positive or negative)
- $z_i$ = position-specific z-score for statistic $i$ (normalized per 60 minutes)
- $\sum_{j=1}^{n}|w_j|$ = sum of absolute values of all weights (normalization factor)

The z-scores are calculated separately by position (Forward vs Defenseman):

$$
z_i = \frac{x_i - \mu_{\text{position}}}{\sigma_{\text{position}}}
$$

Where $x_i$ is the per-60 rate statistic, $\mu_{\text{position}}$ is the position-specific mean, and $\sigma_{\text{position}}$ is the position-specific standard deviation.


```python
s = sum(np.abs(defensive_weights[stat]) for stat in defensive_weights.keys())
defense_quality = sum((defensive_weights[stat] / s) * per60_df[stat] for stat in defensive_weights.keys())

per60_df.loc[:, ['defense_quality']] = defense_quality

# Add raw ice time data for reference
per60_df.loc[:, ['icetime']] = defense_normalized['icetime']
per60_df.loc[:, ['games_played']] = defense_normalized['games_played']
per60_df.loc[:, ['icetime_per_game']] = defense_normalized['icetime_per_game']

plt.figure(figsize=(10, 6))

sns.histplot(per60_df['defense_quality'], bins=20, kde=True)
plt.title('Distribution of Defense Quality')
plt.xlabel('Defense Quality')

plt.tight_layout()
plt.show()
```


    
![png](defense_files/defense_16_0.png)
    


## Results and Interpretation

We finally have our new statistic! Whew, this was an intense one but we're finally at the fun part! If we look at the distribution of forwards and defensemen, we can see they're both mostly normal with a very slight skew left. This is to be expected, most players are average defensively, some great, some awful. On a macro level, it looks good! Let's dive deep to see if it's accurate.


```python
defense_dq = per60_df[per60_df['position'] == 'D']
forward_dq = per60_df[per60_df['position'] == 'F']
```


```python
plt.figure(figsize=(10, 6))

plt.subplot(1, 2, 1)
sns.histplot(defense_dq['defense_quality'], bins=20, kde=True)
plt.title('Distribution of Defense Quality (Defensemen)')

plt.subplot(1, 2, 2)
sns.histplot(forward_dq['defense_quality'], bins=20, kde=True)
plt.title('Distribution of Defense Quality (Forwards)')

plt.tight_layout()
plt.show()

```


    
![png](defense_files/defense_19_0.png)
    


The defensive top 10 looks very accurate. Spurgeon is an absolute lock down defenseman, and right behind him is the LA King's top pairing. The only surprise to me in the top 10 is Rasmus Ristolainen. He's had an up and down career, but looking at the numbers, he had a great year last season (+3, 94 BLK, 25 TAKE) that was unfortunately derailed by injuries.


```python
display_stats = ['name', 'team', 'position', 'defense_quality']
d_top_10 = defense_dq[display_stats].sort_values(by='defense_quality', ascending=False).head(10).reset_index(drop=True)
d_top_10.index += 1

d_top_10.round(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>team</th>
      <th>position</th>
      <th>defense_quality</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>Jared Spurgeon</td>
      <td>MIN</td>
      <td>D</td>
      <td>0.970</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Vladislav Gavrikov</td>
      <td>LAK</td>
      <td>D</td>
      <td>0.865</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Mikey Anderson</td>
      <td>LAK</td>
      <td>D</td>
      <td>0.812</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Chris Tanev</td>
      <td>TOR</td>
      <td>D</td>
      <td>0.800</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Artem Zub</td>
      <td>OTT</td>
      <td>D</td>
      <td>0.797</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Rasmus Ristolainen</td>
      <td>PHI</td>
      <td>D</td>
      <td>0.744</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Colton Parayko</td>
      <td>STL</td>
      <td>D</td>
      <td>0.714</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Jaccob Slavin</td>
      <td>CAR</td>
      <td>D</td>
      <td>0.687</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Jake Sanderson</td>
      <td>OTT</td>
      <td>D</td>
      <td>0.655</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Adam Pelech</td>
      <td>NYI</td>
      <td>D</td>
      <td>0.620</td>
    </tr>
  </tbody>
</table>
</div>



Remember how I said the weights might need to be adjusted for forwards? While the defensive metric is great for defensemen, it fails to accurately assess defense for forwards in my opinion. The list is mainly third line players, generally whose role is to be good defensivley and shut down an opposing team's top line. This makes sense why players such as Logan O'Connor are at the top. In that sense, it's good even if I don't agree with the top ranking. However, I can't help but feel like Sam Reinhart (12) should not be above Aleksander Barkov (16). There's other questionable rankings in here, but overall I think it's a solid foundation that can (and should) be built upon for forwards.


```python
f_top_10 = forward_dq[display_stats].sort_values(by='defense_quality', ascending=False).head(10).reset_index(drop=True)
f_top_10.index += 1

f_top_10.round(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>team</th>
      <th>position</th>
      <th>defense_quality</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>Logan O'Connor</td>
      <td>COL</td>
      <td>F</td>
      <td>1.101</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Noel Acciari</td>
      <td>PIT</td>
      <td>F</td>
      <td>1.063</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Elias Pettersson</td>
      <td>VAN</td>
      <td>F</td>
      <td>1.018</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Colton Sissons</td>
      <td>NSH</td>
      <td>F</td>
      <td>0.972</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Parker Kelly</td>
      <td>COL</td>
      <td>F</td>
      <td>0.841</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Joel Kiviranta</td>
      <td>COL</td>
      <td>F</td>
      <td>0.802</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Anthony Cirelli</td>
      <td>TBL</td>
      <td>F</td>
      <td>0.793</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Adam Lowry</td>
      <td>WPG</td>
      <td>F</td>
      <td>0.787</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Ryan Poehling</td>
      <td>PHI</td>
      <td>F</td>
      <td>0.779</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Alexander Wennberg</td>
      <td>SJS</td>
      <td>F</td>
      <td>0.734</td>
    </tr>
  </tbody>
</table>
</div>



The biggest issue with this raw Defense Quality stat, is it's interpretability. Defense Rating is in goals, WAR is in wins, but Defense Quality is in standard deviations of the mean. That's neither sexy, nor is it meaningful without knowing the data. Baseball has had a solution for this forever, the plus statistic! We can transform Defense Quality as is, to Defense Quality Plus, so that it is now measured in "percent of the average." The numbers are nearly identical since our average was at 0, and we're just shifting it to 100. This means "Spurgeon has 97% better defense than the average player," or "Spurgeon gives up 97% less quality offense for the opposing team than average." This plus version is what I propose to be the default way of showing Defense Quality, much like how the default way of showing PDO is the transformed statistic, rather than raw sum percentage.


```python
dq_plus = 100 + defense_dq['defense_quality'] * 100
defense_dq.loc[:, ['dq_plus']] = dq_plus / np.mean(dq_plus) * 100

dq_plus = 100 + forward_dq['defense_quality'] * 100
forward_dq.loc[:, ['dq_plus']] = dq_plus / np.mean(dq_plus) * 100

top10_d = defense_dq[display_stats + ['dq_plus']].sort_values(by='dq_plus', ascending=False).head(10).reset_index(drop=True)
top10_d.index += 1

top10_d.round(3)

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>team</th>
      <th>position</th>
      <th>defense_quality</th>
      <th>dq_plus</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>Jared Spurgeon</td>
      <td>MIN</td>
      <td>D</td>
      <td>0.970</td>
      <td>197.041</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Vladislav Gavrikov</td>
      <td>LAK</td>
      <td>D</td>
      <td>0.865</td>
      <td>186.464</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Mikey Anderson</td>
      <td>LAK</td>
      <td>D</td>
      <td>0.812</td>
      <td>181.166</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Chris Tanev</td>
      <td>TOR</td>
      <td>D</td>
      <td>0.800</td>
      <td>180.031</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Artem Zub</td>
      <td>OTT</td>
      <td>D</td>
      <td>0.797</td>
      <td>179.718</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Rasmus Ristolainen</td>
      <td>PHI</td>
      <td>D</td>
      <td>0.744</td>
      <td>174.398</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Colton Parayko</td>
      <td>STL</td>
      <td>D</td>
      <td>0.714</td>
      <td>171.409</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Jaccob Slavin</td>
      <td>CAR</td>
      <td>D</td>
      <td>0.687</td>
      <td>168.713</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Jake Sanderson</td>
      <td>OTT</td>
      <td>D</td>
      <td>0.655</td>
      <td>165.462</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Adam Pelech</td>
      <td>NYI</td>
      <td>D</td>
      <td>0.620</td>
      <td>161.999</td>
    </tr>
  </tbody>
</table>
</div>



The final Defense Quality Plus (DQ+) metric transforms DQ to a percentage scale:

$$
DQ+ = \frac{100 + (DQ \times 100)}{\overline{DQ+}} \times 100
$$

Where $\overline{DQ+}$ is the mean of the transformed scores, centering the league average at 100.

When we graph DQ+ we can easily highlight elite vs above average defense. Since this is a normal distribution, we can focus on the standard deviations. Anything 2 standard deviations (around 76%) is considered elite. This limits us to only the top 5 as well in this case. Also, anything below 62% can be considered poor defense that needs improvement.


```python
# Visualize the DR+ distribution
std = defense_dq['dq_plus'].std()
mean = defense_dq['dq_plus'].mean()

plt.figure(figsize=(10, 6))
sns.histplot(defense_dq['dq_plus'], bins=20, kde=True)
plt.axvline(mean, color='red', linestyle='--', label=f'Average ({mean:.2f})')
plt.axvline(mean + std, color='orange', linestyle='--', alpha=0.5, label=f'+1 Std Dev ({mean + std:.2f})')
plt.axvline(mean + 2 * std, color='green', linestyle='--', alpha=0.5, label=f'+2 Std Dev ({mean + 2 * std:.2f})')
plt.axvline(mean - std, color='orange', linestyle='--', alpha=0.5, label=f'-1 Std Dev ({mean - std:.2f})')
plt.title('Distribution of DQ+ For Defensemen')
plt.xlabel(f'DQ+ ({mean:.2f} = League Average For Defensemen)')
plt.legend()
plt.show()
```


    
![png](defense_files/defense_28_0.png)
    


Although not the intention of the metric, it can be used to measure a team's defense pretty well. If we take the average of a team's players' DQ+, we get a pretty good understanding on a team's overall defense that does tend to line up well. It's not perfect because the forward DQ+ isn't, but it's not a bad list. Again, this isn't the intention of the stat, it just lines up well and shows more ways it can be used.


```python
team_dq = pd.concat([defense_dq, forward_dq])
team_dq = team_dq.groupby('team')['dq_plus'].mean().sort_values(ascending=False).reset_index()
top10_team_dq = team_dq.head(10)

top10_team_dq.round(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>team</th>
      <th>dq_plus</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>PHI</td>
      <td>128.909</td>
    </tr>
    <tr>
      <th>1</th>
      <td>LAK</td>
      <td>128.790</td>
    </tr>
    <tr>
      <th>2</th>
      <td>WPG</td>
      <td>123.566</td>
    </tr>
    <tr>
      <th>3</th>
      <td>COL</td>
      <td>120.687</td>
    </tr>
    <tr>
      <th>4</th>
      <td>STL</td>
      <td>120.397</td>
    </tr>
    <tr>
      <th>5</th>
      <td>VAN</td>
      <td>120.346</td>
    </tr>
    <tr>
      <th>6</th>
      <td>MIN</td>
      <td>115.182</td>
    </tr>
    <tr>
      <th>7</th>
      <td>EDM</td>
      <td>115.108</td>
    </tr>
    <tr>
      <th>8</th>
      <td>CGY</td>
      <td>112.099</td>
    </tr>
    <tr>
      <th>9</th>
      <td>VGK</td>
      <td>110.477</td>
    </tr>
  </tbody>
</table>
</div>



Overall, Defense Quality provides an excellent way to compare defensemen, and understand their defensive impact on the quality of offense an opposing team gets while they're on the ice. It is a very good stat to pair with Defensive Rating, so that one can understand that results of a player's defense, and the reasons behind it. Since Defense Quality directly compares players in the same position, it is able to be quickly understood as to what is "good" vs "bad" Defense Quality without needing a deep understanding of the statistic.

# Thanks For Reading!
If you made it this far, thanks! I want to keep this blog pretty light hearted and fun, more like my first two, but I thought it would be interesting to go deep into trying to fill a hole that exists. [I've also started an X account in case anyone is interested](https://x.com/HockeyxHarmony), but I'm not sure how much I'll be using it. Would love to hear your thoughts on where else this stat can be useful! Also, let me know if you more preferred the previous ones aimed at being more fun, or this one that aimed at providing a real solution.
