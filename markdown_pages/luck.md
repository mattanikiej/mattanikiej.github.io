---
layout: none
---

# The NHL's Lucky Charm

I've been wanting to explore which player is a "lucky charm" for their team. Not necessarily which player is getting lucky, but which player is causing his **team** to get lucky. Just how great players make their teammates better by setting them up beautifully, or scoring goals off any pass, lucky charm players make their teammates better by just being on the same team as them! "Puck luck" and "lucky bounces" are always talked about in hockey and it's an intangible part of the game, but let's try to quantify it anyway!


```python
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
```

## Exploring The Data

The data that I used for this is from [MoneyPuck](https://moneypuck.com/) which is an incredible resource for any hockey stats nerd. This used the 2024-25 season's data, but it should run with any season that you'd like to check.




```python
season = "2024-25"

file_prefix = f"../data/{season}/"

skaters_file = file_prefix + "skaters.csv"
```

The data shows different statistics for different situations. This is a very important distinction to make in hockey, as the game is played completely differently in man-up vs even strength situations.


```python
data = pd.read_csv(skaters_file)

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



Overall there were 920 unique players in the NHL in the 2024-25 season. However, we want to make sure to find someone who was consistently lucky, rather than someone who got lucky being lucky. There's nothing more lucky than lucky number 7, so to make sure we have a good sample size, we're going to only look at players who have played at least 77 games.


```python
unique_players = len(data['playerId'].unique())

print(f"Number of players in the {season} NHL Season: {unique_players}")
```

    Number of players in the 2024-25 NHL Season: 920



```python
# get all players who have played at least 70 games
players_77 = data[data["games_played"] >= 77].copy()

unique_players_77 = len(players_77['playerId'].unique())

print(f"Number of players in the {season} NHL Season with at least 77 games played: {unique_players_77}")
```

    Number of players in the 2024-25 NHL Season with at least 77 games played: 280


## Current Puck Luck Stats

Right now, there's already an advanced metric that we can use to sort of define "puck luck", and that's PDO (Personal Discouragement Outcomes). All PDO is, is the sum of a team's shooting percentage plus a team's save percentage. This averages out to 100.

With a big sample size, PDO can help see if a team is playing above or below expectations, and can help aid in understanding how a team might regress back to the mean. In small sample sizes, it can help show who is really hot or really cold. It is important to note that since PDO is measuring luck, and is very influenced by luck, it is not a good stat in isolation. It is extremely valuable in conjunction with other stats and showing a quick overview, but is not meant to be a sole stat used to inform a team's or player's performance.

So let's use it for exactly that! I want to find the On Ice PDO of each player, rather than the team PDO or player's individual PDO. This will help us see who's teammates are getting lucky during their shifts as well, while ignoring the team's performance while on the player is on the bench. 


```python
# calculate pdo based on shots and goals
shots_for = players_77["OnIce_F_shotsOnGoal"]
shots_against = players_77["OnIce_A_shotsOnGoal"]

players_77["OnIce_F_ShotPercentage"] = np.where(shots_for > 0,
                                    players_77["OnIce_F_goals"] / shots_for,
                                    np.nan)

players_77["OnIce_F_SavePercentage"] = np.where(shots_against > 0,
                                    1 - (players_77["OnIce_A_goals"] / shots_against),
                                    np.nan)

# PDO on 1.000 scale (typical analytics usage)
players_77["OnIce_PDO"] = players_77["OnIce_F_ShotPercentage"] + players_77["OnIce_F_SavePercentage"]

# PDO on ~100 scale (common media display)
players_77["OnIce_PDO_100"] = 100 * players_77["OnIce_PDO"]

pdo = players_77[["playerId", "name", "team", "position", "situation", "OnIce_F_ShotPercentage", "OnIce_F_SavePercentage", "OnIce_PDO", "OnIce_PDO_100"]]
```


```python
pdo.head()
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
      <th>name</th>
      <th>team</th>
      <th>position</th>
      <th>situation</th>
      <th>OnIce_F_ShotPercentage</th>
      <th>OnIce_F_SavePercentage</th>
      <th>OnIce_PDO</th>
      <th>OnIce_PDO_100</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>5</th>
      <td>8480950</td>
      <td>Ilya Lyubushkin</td>
      <td>DAL</td>
      <td>D</td>
      <td>other</td>
      <td>1.000000</td>
      <td>0.800000</td>
      <td>1.800000</td>
      <td>180.000000</td>
    </tr>
    <tr>
      <th>6</th>
      <td>8480950</td>
      <td>Ilya Lyubushkin</td>
      <td>DAL</td>
      <td>D</td>
      <td>all</td>
      <td>0.100000</td>
      <td>0.902760</td>
      <td>1.002760</td>
      <td>100.275953</td>
    </tr>
    <tr>
      <th>7</th>
      <td>8480950</td>
      <td>Ilya Lyubushkin</td>
      <td>DAL</td>
      <td>D</td>
      <td>5on5</td>
      <td>0.093306</td>
      <td>0.920354</td>
      <td>1.013660</td>
      <td>101.366027</td>
    </tr>
    <tr>
      <th>8</th>
      <td>8480950</td>
      <td>Ilya Lyubushkin</td>
      <td>DAL</td>
      <td>D</td>
      <td>4on5</td>
      <td>0.093750</td>
      <td>0.866667</td>
      <td>0.960417</td>
      <td>96.041667</td>
    </tr>
    <tr>
      <th>9</th>
      <td>8480950</td>
      <td>Ilya Lyubushkin</td>
      <td>DAL</td>
      <td>D</td>
      <td>5on4</td>
      <td>0.000000</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



If we look at the On Ice PDO in all situations, we see why it's important that we're going to make a distinction at only looking at 5 on 5 stats. In the 2024-25 season, the top 3 players are all on the Winnipeg Jets. That season, the Jets had the best power play in the entire NHL, and their goalie won both the Vezina and Hart trophies, and had the second highest save percentage (0.925) in the entire league. Only Anthony Stolarz had a higher save percentage with 0.926, but had 29 less games played (63 vs 34).


```python
pdo_all = pdo[pdo["situation"] == "all"]

pdo_all.sort_values(by="OnIce_PDO_100", ascending=False).head(5)
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
      <th>name</th>
      <th>team</th>
      <th>position</th>
      <th>situation</th>
      <th>OnIce_F_ShotPercentage</th>
      <th>OnIce_F_SavePercentage</th>
      <th>OnIce_PDO</th>
      <th>OnIce_PDO_100</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2466</th>
      <td>8478398</td>
      <td>Kyle Connor</td>
      <td>WPG</td>
      <td>L</td>
      <td>all</td>
      <td>0.147651</td>
      <td>0.904762</td>
      <td>1.052413</td>
      <td>105.241291</td>
    </tr>
    <tr>
      <th>956</th>
      <td>8476460</td>
      <td>Mark Scheifele</td>
      <td>WPG</td>
      <td>C</td>
      <td>all</td>
      <td>0.145717</td>
      <td>0.905055</td>
      <td>1.050773</td>
      <td>105.077295</td>
    </tr>
    <tr>
      <th>1726</th>
      <td>8475799</td>
      <td>Nino Niederreiter</td>
      <td>WPG</td>
      <td>R</td>
      <td>all</td>
      <td>0.112769</td>
      <td>0.936508</td>
      <td>1.049277</td>
      <td>104.927742</td>
    </tr>
    <tr>
      <th>2366</th>
      <td>8474151</td>
      <td>Ryan McDonagh</td>
      <td>TBL</td>
      <td>D</td>
      <td>all</td>
      <td>0.138889</td>
      <td>0.909624</td>
      <td>1.048513</td>
      <td>104.851330</td>
    </tr>
    <tr>
      <th>2391</th>
      <td>8478010</td>
      <td>Brayden Point</td>
      <td>TBL</td>
      <td>C</td>
      <td>all</td>
      <td>0.149286</td>
      <td>0.898860</td>
      <td>1.048147</td>
      <td>104.814690</td>
    </tr>
  </tbody>
</table>
</div>



Elite special teams and goaltending is not lucky. If we change this to look at 5 on 5 only, we'll get a much clearer picture on each player's puck luck. We have more reasonable results like Kirill Marchenko who had an amazing breakout year, but a very high save percentage while he's on the ice. This is no fault of his, in fact he's directly influencing it by being great on the defensive end of the puck too! His goalie, Elvis Merzļikins, only had a 0.892 save percentage, which was the 41st ranked SV% and a far cry from the 0.931 with Kirill Marchenko on the ice.

Again, we run into similar issues as before, but at least our results are more reasonable. Adrian Kempe was great year, and Darcy Kuemper was phenomenal which isn't lucky. Then we see Brayden Point's entire line in the top 5, which makes sense since they're backed by another Vezina winner, and have the 2024-25 Ted Lindsay winner on the wing, who's known for his creative playmaking.

This shows the faults of isolating PDO. Especially since we know these players aren't getting lucky, as they've been putting up good performances their entire careers. The 2024-25 season wasn't even Adrian Kempe's best year. Also, we don't want to know which player is getting lucky, but who is making their **team** lucky. This On Ice PDO is going to help us understand at a glance who is getting slightly lucky, but we're going to have to dive a little bit deeper.


```python
pdo_5v5 = pdo[pdo["situation"] == "5on5"]

pdo_5v5.sort_values(by="OnIce_PDO_100", ascending=False).head(10)
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
      <th>name</th>
      <th>team</th>
      <th>position</th>
      <th>situation</th>
      <th>OnIce_F_ShotPercentage</th>
      <th>OnIce_F_SavePercentage</th>
      <th>OnIce_PDO</th>
      <th>OnIce_PDO_100</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1332</th>
      <td>8477960</td>
      <td>Adrian Kempe</td>
      <td>LAK</td>
      <td>R</td>
      <td>5on5</td>
      <td>0.117761</td>
      <td>0.932790</td>
      <td>1.050551</td>
      <td>105.055084</td>
    </tr>
    <tr>
      <th>2392</th>
      <td>8478010</td>
      <td>Brayden Point</td>
      <td>TBL</td>
      <td>C</td>
      <td>5on5</td>
      <td>0.124567</td>
      <td>0.925566</td>
      <td>1.050134</td>
      <td>105.013382</td>
    </tr>
    <tr>
      <th>767</th>
      <td>8477404</td>
      <td>Jake Guentzel</td>
      <td>TBL</td>
      <td>C</td>
      <td>5on5</td>
      <td>0.116694</td>
      <td>0.931148</td>
      <td>1.047841</td>
      <td>104.784122</td>
    </tr>
    <tr>
      <th>4537</th>
      <td>8480893</td>
      <td>Kirill Marchenko</td>
      <td>CBJ</td>
      <td>R</td>
      <td>5on5</td>
      <td>0.116641</td>
      <td>0.930530</td>
      <td>1.047171</td>
      <td>104.717091</td>
    </tr>
    <tr>
      <th>872</th>
      <td>8476453</td>
      <td>Nikita Kucherov</td>
      <td>TBL</td>
      <td>R</td>
      <td>5on5</td>
      <td>0.122754</td>
      <td>0.922835</td>
      <td>1.045589</td>
      <td>104.558914</td>
    </tr>
    <tr>
      <th>2012</th>
      <td>8476902</td>
      <td>Esa Lindell</td>
      <td>DAL</td>
      <td>D</td>
      <td>5on5</td>
      <td>0.108911</td>
      <td>0.935438</td>
      <td>1.044348</td>
      <td>104.434848</td>
    </tr>
    <tr>
      <th>1837</th>
      <td>8479385</td>
      <td>Jordan Kyrou</td>
      <td>STL</td>
      <td>R</td>
      <td>5on5</td>
      <td>0.114338</td>
      <td>0.929577</td>
      <td>1.043915</td>
      <td>104.391503</td>
    </tr>
    <tr>
      <th>4152</th>
      <td>8478874</td>
      <td>Adam Gaudette</td>
      <td>OTT</td>
      <td>R</td>
      <td>5on5</td>
      <td>0.101587</td>
      <td>0.941358</td>
      <td>1.042945</td>
      <td>104.294533</td>
    </tr>
    <tr>
      <th>2467</th>
      <td>8478398</td>
      <td>Kyle Connor</td>
      <td>WPG</td>
      <td>L</td>
      <td>5on5</td>
      <td>0.111675</td>
      <td>0.930876</td>
      <td>1.042551</td>
      <td>104.255070</td>
    </tr>
    <tr>
      <th>2367</th>
      <td>8474151</td>
      <td>Ryan McDonagh</td>
      <td>TBL</td>
      <td>D</td>
      <td>5on5</td>
      <td>0.111667</td>
      <td>0.930769</td>
      <td>1.042436</td>
      <td>104.243590</td>
    </tr>
  </tbody>
</table>
</div>



## What's Luck?

To create a stat that is going to quantify luck, first we are going to have to define what it means to be lucky. We are going to incorporate 5 stats:

These metrics will be:
1. On Ice PDO (defined above)
2. Individual Point Percentage
3. G - xG Difference
4. Linemate G - Linemate xG Difference
5. Low Danger Goals - Low Danger xG

Then we will build a composite z-score metric that will define luck. Using a composite z-score will allow us to combine these different stats on the same scale, and give us a clear picture of not only who is the luckiest charm, but how much more luck they bring than other players.

### IPP (Individual Point Percentage)

IPP will help us see whose teammates are scoring goals without them needing to do anything. This doesn't mean they're freeloaders. They could be drawing defenders to them, screening the goalie, or otherwise letting their linemates get lucky and get to high danger areas more easily than they would've otherwise. This could even be something as lucky as a defender tripping over the blue line, and allowing a free lane for a player to have a free 1 on 1 with the opposing goalie.


```python
# Calculate IPP (Individual Points Percentage)
# IPP = (Player's points on on-ice goals) / (Team goals while player is on ice)

on_ice_goals_for = players_77["OnIce_F_goals"]

players_77["IPP"] = np.where(on_ice_goals_for > 0,
                              players_77["I_F_points"] / on_ice_goals_for,
                              np.nan)

# IPP as percentage (0-100 scale)
players_77["IPP_percentage"] = 100 * players_77["IPP"]

# View IPP by situation
ipp_view = players_77[["playerId", "name", "team", "position", "situation", "I_F_points", "OnIce_F_goals", "IPP", "IPP_percentage"]]

```

There's just one problem... Defenders are naturally going to have a significantly lower IPP as compared to forwards, since they don't have the puck on their sticks as often in the offensive zone.


```python
ipp_5v5 = ipp_view[ipp_view["situation"] == "5on5"]

ipp_5v5.sort_values(by="IPP_percentage", ascending=True).head(10)
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
      <th>name</th>
      <th>team</th>
      <th>position</th>
      <th>situation</th>
      <th>I_F_points</th>
      <th>OnIce_F_goals</th>
      <th>IPP</th>
      <th>IPP_percentage</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>3522</th>
      <td>8476885</td>
      <td>Jacob Trouba</td>
      <td>ANA</td>
      <td>D</td>
      <td>5on5</td>
      <td>10.0</td>
      <td>52.0</td>
      <td>0.192308</td>
      <td>19.230769</td>
    </tr>
    <tr>
      <th>3562</th>
      <td>8481122</td>
      <td>Simon Benoit</td>
      <td>TOR</td>
      <td>D</td>
      <td>5on5</td>
      <td>9.0</td>
      <td>44.0</td>
      <td>0.204545</td>
      <td>20.454545</td>
    </tr>
    <tr>
      <th>1917</th>
      <td>8470600</td>
      <td>Ryan Suter</td>
      <td>STL</td>
      <td>D</td>
      <td>5on5</td>
      <td>13.0</td>
      <td>58.0</td>
      <td>0.224138</td>
      <td>22.413793</td>
    </tr>
    <tr>
      <th>1187</th>
      <td>8475188</td>
      <td>Brayden McNabb</td>
      <td>VGK</td>
      <td>D</td>
      <td>5on5</td>
      <td>16.0</td>
      <td>68.0</td>
      <td>0.235294</td>
      <td>23.529412</td>
    </tr>
    <tr>
      <th>942</th>
      <td>8475279</td>
      <td>Ben Chiarot</td>
      <td>DET</td>
      <td>D</td>
      <td>5on5</td>
      <td>12.0</td>
      <td>51.0</td>
      <td>0.235294</td>
      <td>23.529412</td>
    </tr>
    <tr>
      <th>2507</th>
      <td>8478443</td>
      <td>Brandon Carlo</td>
      <td>TOR</td>
      <td>D</td>
      <td>5on5</td>
      <td>11.0</td>
      <td>46.0</td>
      <td>0.239130</td>
      <td>23.913043</td>
    </tr>
    <tr>
      <th>3672</th>
      <td>8477948</td>
      <td>Travis Sanheim</td>
      <td>PHI</td>
      <td>D</td>
      <td>5on5</td>
      <td>17.0</td>
      <td>69.0</td>
      <td>0.246377</td>
      <td>24.637681</td>
    </tr>
    <tr>
      <th>3437</th>
      <td>8475455</td>
      <td>Brenden Dillon</td>
      <td>NJD</td>
      <td>D</td>
      <td>5on5</td>
      <td>14.0</td>
      <td>56.0</td>
      <td>0.250000</td>
      <td>25.000000</td>
    </tr>
    <tr>
      <th>2982</th>
      <td>8476331</td>
      <td>Dylan DeMelo</td>
      <td>WPG</td>
      <td>D</td>
      <td>5on5</td>
      <td>18.0</td>
      <td>70.0</td>
      <td>0.257143</td>
      <td>25.714286</td>
    </tr>
    <tr>
      <th>562</th>
      <td>8481568</td>
      <td>Alex Vlasic</td>
      <td>CHI</td>
      <td>D</td>
      <td>5on5</td>
      <td>16.0</td>
      <td>61.0</td>
      <td>0.262295</td>
      <td>26.229508</td>
    </tr>
  </tbody>
</table>
</div>



Also, we'd expect IPP to be normally distributed, but there are clearly two humps in the histogram when we play the frequency of IPP.


```python
plt.figure(figsize=(10, 6))
sns.histplot(ipp_5v5["IPP_percentage"], bins=30, kde=True)
plt.title("Historgram IPP 5on5")
plt.xlabel("IPP")
plt.show()

```


    
![png](luck_files/luck_medium_24_0.png)
    


Not to worry though! Two humps most likely means that one of the distributions is for forwards, and the other for defense. If we look at the stats seperately for the positions, we see this to be true.


```python
ipp_d = ipp_5v5[ipp_5v5["position"] == "D"]

ipp_d.describe()
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
      <th>I_F_points</th>
      <th>OnIce_F_goals</th>
      <th>IPP</th>
      <th>IPP_percentage</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>7.400000e+01</td>
      <td>74.000000</td>
      <td>74.000000</td>
      <td>74.000000</td>
      <td>74.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>8.478267e+06</td>
      <td>20.500000</td>
      <td>57.864865</td>
      <td>0.350878</td>
      <td>35.087824</td>
    </tr>
    <tr>
      <th>std</th>
      <td>2.814328e+03</td>
      <td>7.360111</td>
      <td>11.622279</td>
      <td>0.082019</td>
      <td>8.201860</td>
    </tr>
    <tr>
      <th>min</th>
      <td>8.470600e+06</td>
      <td>9.000000</td>
      <td>29.000000</td>
      <td>0.192308</td>
      <td>19.230769</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>8.476564e+06</td>
      <td>15.250000</td>
      <td>49.500000</td>
      <td>0.289263</td>
      <td>28.926282</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>8.478020e+06</td>
      <td>18.500000</td>
      <td>57.500000</td>
      <td>0.347319</td>
      <td>34.731935</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>8.480801e+06</td>
      <td>24.000000</td>
      <td>66.000000</td>
      <td>0.411096</td>
      <td>41.109626</td>
    </tr>
    <tr>
      <th>max</th>
      <td>8.483457e+06</td>
      <td>48.000000</td>
      <td>86.000000</td>
      <td>0.575342</td>
      <td>57.534247</td>
    </tr>
  </tbody>
</table>
</div>




```python
ipp_f = ipp_5v5[ipp_5v5["position"] != "D"]

ipp_f.describe()
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
      <th>I_F_points</th>
      <th>OnIce_F_goals</th>
      <th>IPP</th>
      <th>IPP_percentage</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>2.060000e+02</td>
      <td>206.000000</td>
      <td>206.000000</td>
      <td>206.000000</td>
      <td>206.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>8.478728e+06</td>
      <td>29.742718</td>
      <td>44.660194</td>
      <td>0.656910</td>
      <td>65.690969</td>
    </tr>
    <tr>
      <th>std</th>
      <td>2.670904e+03</td>
      <td>10.865332</td>
      <td>13.580871</td>
      <td>0.094020</td>
      <td>9.401985</td>
    </tr>
    <tr>
      <th>min</th>
      <td>8.470621e+06</td>
      <td>5.000000</td>
      <td>12.000000</td>
      <td>0.400000</td>
      <td>40.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>8.476834e+06</td>
      <td>22.250000</td>
      <td>36.000000</td>
      <td>0.600000</td>
      <td>60.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>8.478460e+06</td>
      <td>29.000000</td>
      <td>44.000000</td>
      <td>0.659545</td>
      <td>65.954545</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>8.480823e+06</td>
      <td>36.000000</td>
      <td>53.750000</td>
      <td>0.713134</td>
      <td>71.313364</td>
    </tr>
    <tr>
      <th>max</th>
      <td>8.484958e+06</td>
      <td>66.000000</td>
      <td>82.000000</td>
      <td>0.916667</td>
      <td>91.666667</td>
    </tr>
  </tbody>
</table>
</div>




```python
plt.figure(figsize=(10, 6))
sns.histplot(ipp_f["IPP_percentage"], bins=50, kde=True, label="Forwards")
sns.histplot(ipp_d["IPP_percentage"], bins=50, kde=True, label="Defensemen")
plt.title("Historgram Forward vs Defense IPP 5on5")
plt.xlabel("IPP")
plt.legend()
plt.show()


```


    
![png](luck_files/luck_medium_28_0.png)
    


We can normalize the IPP by position, rather than as a whole. This way we get a mcuh clearer picture of who has the lowest IPP, since it's going to be relative to the player's position. When we do that, we can see that Jacob Trouba went from 1st place, to 9th in the lowest IPP.


```python
# Filter to 5v5 only for clean analysis
luck_df = players_77[players_77['situation'] == '5on5'].copy()

# IPP Z-Score
# Calculate position-specific IPP means 
position_ipp_means = luck_df.groupby('position')['IPP'].transform('mean')
position_ipp_stds = luck_df.groupby('position')['IPP'].transform('std')

luck_df['ipp_deviation'] = (luck_df['IPP'] - position_ipp_means) / position_ipp_stds

# Z-score normalize the deviation (so it's on same scale as other components)
mean_ipp_dev = luck_df['ipp_deviation'].mean()
std_ipp_dev = luck_df['ipp_deviation'].std()
luck_df['component_ipp'] = (luck_df['ipp_deviation'] - mean_ipp_dev) / std_ipp_dev

print(f"\nTop 10 smallest IPP (relative to position):")
luck_df.nsmallest(10, 'component_ipp')[['name', 'position', 'team', 'IPP_percentage', 'I_F_points', 'OnIce_F_goals', 'component_ipp']]

```

    
    Top 10 smallest IPP (relative to position):





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
      <th>position</th>
      <th>team</th>
      <th>IPP_percentage</th>
      <th>I_F_points</th>
      <th>OnIce_F_goals</th>
      <th>component_ipp</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1272</th>
      <td>John Beecher</td>
      <td>C</td>
      <td>BOS</td>
      <td>40.000000</td>
      <td>8.0</td>
      <td>20.0</td>
      <td>-2.403228</td>
    </tr>
    <tr>
      <th>3582</th>
      <td>Luke Glendening</td>
      <td>C</td>
      <td>TBL</td>
      <td>41.666667</td>
      <td>5.0</td>
      <td>12.0</td>
      <td>-2.231466</td>
    </tr>
    <tr>
      <th>2617</th>
      <td>Isac Lundestrom</td>
      <td>C</td>
      <td>ANA</td>
      <td>42.105263</td>
      <td>8.0</td>
      <td>19.0</td>
      <td>-2.186266</td>
    </tr>
    <tr>
      <th>3032</th>
      <td>Alex Iafallo</td>
      <td>L</td>
      <td>WPG</td>
      <td>50.000000</td>
      <td>18.0</td>
      <td>36.0</td>
      <td>-2.144604</td>
    </tr>
    <tr>
      <th>77</th>
      <td>Barclay Goodrow</td>
      <td>C</td>
      <td>SJS</td>
      <td>42.857143</td>
      <td>6.0</td>
      <td>14.0</td>
      <td>-2.108779</td>
    </tr>
    <tr>
      <th>597</th>
      <td>Zemgus Girgensons</td>
      <td>C</td>
      <td>TBL</td>
      <td>42.857143</td>
      <td>6.0</td>
      <td>14.0</td>
      <td>-2.108779</td>
    </tr>
    <tr>
      <th>2347</th>
      <td>Martin Pospisil</td>
      <td>C</td>
      <td>CGY</td>
      <td>43.181818</td>
      <td>19.0</td>
      <td>44.0</td>
      <td>-2.075319</td>
    </tr>
    <tr>
      <th>1727</th>
      <td>Nino Niederreiter</td>
      <td>R</td>
      <td>WPG</td>
      <td>51.162791</td>
      <td>22.0</td>
      <td>43.0</td>
      <td>-1.988713</td>
    </tr>
    <tr>
      <th>3522</th>
      <td>Jacob Trouba</td>
      <td>D</td>
      <td>ANA</td>
      <td>19.230769</td>
      <td>10.0</td>
      <td>52.0</td>
      <td>-1.943827</td>
    </tr>
    <tr>
      <th>2752</th>
      <td>Lawson Crouse</td>
      <td>L</td>
      <td>UTA</td>
      <td>51.612903</td>
      <td>16.0</td>
      <td>31.0</td>
      <td>-1.942781</td>
    </tr>
  </tbody>
</table>
</div>



We already calculated the On Ice PDO earlier, so let's get the z-score for it as well, and confirm it's a normal distribution.


```python
# PDO Z-Score 
# Calculate z-score normalization for OnIce_PDO_100 by situation
mean_pdo = luck_df.groupby('situation')['OnIce_PDO_100'].transform('mean')
std_pdo = luck_df.groupby('situation')['OnIce_PDO_100'].transform('std')
luck_df['OnIce_PDO_100_normalized'] = (luck_df['OnIce_PDO_100'] - mean_pdo) / std_pdo

# Use the normalized PDO for component calculation
luck_df['component_pdo'] = luck_df['OnIce_PDO_100_normalized']

plt.figure(figsize=(10, 6))
sns.histplot(luck_df['component_pdo'], bins=50, kde=True)
plt.title('PDO Z-Score Distribution')
plt.xlabel('Z-Score')
plt.ylabel('Frequency')
plt.show()
```


    
![png](luck_files/luck_medium_32_0.png)
    


Another luck based metric we can use the difference in goals minus expected goals. This gives us  an understanding of who is scoring at a higher rate than other players from the same location. We want to make sure we get the rate metric of this though, so we will get this difference per 60 minutes. Like I mentioned earlier, we want players who are consistently lucky, rather than lucky just once.


```python
# Convert ice time to hours for per-60 calculations
luck_df['icetime_hours'] = luck_df['icetime'] / 3600
```


```python
# Individual Goals vs xGoals per 60 minutes
luck_df['goals_minus_xgoals'] = luck_df['I_F_goals'] - luck_df['I_F_xGoals']
luck_df['goals_minus_xgoals_per60'] = (luck_df['goals_minus_xgoals'] / luck_df['icetime_hours']) * (60/60)

# Z-score normalize
mean_g_xg = luck_df['goals_minus_xgoals_per60'].mean()
std_g_xg = luck_df['goals_minus_xgoals_per60'].std()
luck_df['component_goals_xgoals'] = (luck_df['goals_minus_xgoals_per60'] - mean_g_xg) / std_g_xg

print(f"\nTop 5 luckiest finishers (goals above expected per 60):")
luck_df.nlargest(5, 'goals_minus_xgoals_per60')[['name', 'I_F_goals', 'I_F_xGoals', 'goals_minus_xgoals_per60']]

```

    
    Top 5 luckiest finishers (goals above expected per 60):





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
      <th>I_F_goals</th>
      <th>I_F_xGoals</th>
      <th>goals_minus_xgoals_per60</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>667</th>
      <td>Adam Fantilli</td>
      <td>25.0</td>
      <td>11.23</td>
      <td>0.673002</td>
    </tr>
    <tr>
      <th>4152</th>
      <td>Adam Gaudette</td>
      <td>16.0</td>
      <td>8.34</td>
      <td>0.639607</td>
    </tr>
    <tr>
      <th>1317</th>
      <td>Artemi Panarin</td>
      <td>25.0</td>
      <td>12.75</td>
      <td>0.588024</td>
    </tr>
    <tr>
      <th>4537</th>
      <td>Kirill Marchenko</td>
      <td>24.0</td>
      <td>12.78</td>
      <td>0.558549</td>
    </tr>
    <tr>
      <th>4177</th>
      <td>Ryan Donato</td>
      <td>23.0</td>
      <td>13.07</td>
      <td>0.539927</td>
    </tr>
  </tbody>
</table>
</div>



More importantly, we want to know who is getting their teammates lucky! We can use the same metric, but define it by the goals their teammates scored and their expected goals, without the current player's goals and expected goals. This will give us a good sense of whose linemmates are scoring more than other players from the same location.


```python
# Teammate Goals vs xGoals per 60 (linemate luck, excluding player's own goals)
# IMPORTANT: Subtract individual goals to avoid double-counting
# This isolates teammate/linemate luck separate from personal finishing luck
luck_df['teammate_goals'] = luck_df['OnIce_F_goals'] - luck_df['I_F_goals']
luck_df['teammate_xgoals'] = luck_df['OnIce_F_xGoals'] - luck_df['I_F_xGoals']
luck_df['teammate_goals_minus_xgoals'] = luck_df['teammate_goals'] - luck_df['teammate_xgoals']
luck_df['teammate_goals_minus_xgoals_per60'] = (luck_df['teammate_goals_minus_xgoals'] / luck_df['icetime_hours']) * (60/60)

# Z-score normalize
mean_tg_xg = luck_df['teammate_goals_minus_xgoals_per60'].mean()
std_tg_xg = luck_df['teammate_goals_minus_xgoals_per60'].std()
luck_df['component_teammate_goals_xgoals'] = (luck_df['teammate_goals_minus_xgoals_per60'] - mean_tg_xg) / std_tg_xg

print(f"\nTop 5 luckiest linemate performers (teammates scoring above expected):")
luck_df.nlargest(5, 'teammate_goals_minus_xgoals_per60')[['name', 'position', 'team', 'teammate_goals', 'teammate_xgoals', 'teammate_goals_minus_xgoals_per60']]
```

    
    Top 5 luckiest linemate performers (teammates scoring above expected):





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
      <th>position</th>
      <th>team</th>
      <th>teammate_goals</th>
      <th>teammate_xgoals</th>
      <th>teammate_goals_minus_xgoals_per60</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>4472</th>
      <td>JJ Peterka</td>
      <td>R</td>
      <td>BUF</td>
      <td>53.0</td>
      <td>33.91</td>
      <td>1.052580</td>
    </tr>
    <tr>
      <th>1047</th>
      <td>Pierre-Luc Dubois</td>
      <td>L</td>
      <td>WSH</td>
      <td>60.0</td>
      <td>44.03</td>
      <td>0.819663</td>
    </tr>
    <tr>
      <th>2347</th>
      <td>Martin Pospisil</td>
      <td>C</td>
      <td>CGY</td>
      <td>41.0</td>
      <td>27.94</td>
      <td>0.767997</td>
    </tr>
    <tr>
      <th>4292</th>
      <td>Jared McCann</td>
      <td>L</td>
      <td>SEA</td>
      <td>40.0</td>
      <td>27.33</td>
      <td>0.763381</td>
    </tr>
    <tr>
      <th>2132</th>
      <td>Ryan McLeod</td>
      <td>C</td>
      <td>BUF</td>
      <td>42.0</td>
      <td>29.43</td>
      <td>0.724357</td>
    </tr>
  </tbody>
</table>
</div>



Finally, scoring from low danger situations is a little bit lucky sometimes. While we want to know who is a lucky charm for their team, some of that luck has to rub off on yourself too right?


```python
# Low Danger Goals Above Expected (shot quality luck)
luck_df['lowdanger_goals_minus_xgoals'] = luck_df['I_F_lowDangerGoals'] - luck_df['I_F_lowDangerxGoals']
luck_df['lowdanger_goals_minus_xgoals_per60'] = (luck_df['lowdanger_goals_minus_xgoals'] / luck_df['icetime_hours']) * (60/60)

# Z-score normalize
mean_ld = luck_df['lowdanger_goals_minus_xgoals_per60'].mean()
std_ld = luck_df['lowdanger_goals_minus_xgoals_per60'].std()
luck_df['component_lowdanger'] = (luck_df['lowdanger_goals_minus_xgoals_per60'] - mean_ld) / std_ld

print(f"\nTop 5 luckiest low danger scorers:")
luck_df.nlargest(5, 'lowdanger_goals_minus_xgoals_per60')[['name', 'position', 'team', 'I_F_lowDangerGoals', 'I_F_lowDangerxGoals', 'lowdanger_goals_minus_xgoals_per60']]
```

    
    Top 5 luckiest low danger scorers:





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
      <th>position</th>
      <th>team</th>
      <th>I_F_lowDangerGoals</th>
      <th>I_F_lowDangerxGoals</th>
      <th>lowdanger_goals_minus_xgoals_per60</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>4307</th>
      <td>Kiefer Sherwood</td>
      <td>L</td>
      <td>VAN</td>
      <td>11.0</td>
      <td>4.16</td>
      <td>0.428452</td>
    </tr>
    <tr>
      <th>1737</th>
      <td>Alex Tuch</td>
      <td>R</td>
      <td>BUF</td>
      <td>12.0</td>
      <td>4.20</td>
      <td>0.421059</td>
    </tr>
    <tr>
      <th>4537</th>
      <td>Kirill Marchenko</td>
      <td>R</td>
      <td>CBJ</td>
      <td>14.0</td>
      <td>5.62</td>
      <td>0.417169</td>
    </tr>
    <tr>
      <th>1317</th>
      <td>Artemi Panarin</td>
      <td>L</td>
      <td>NYR</td>
      <td>15.0</td>
      <td>6.88</td>
      <td>0.389776</td>
    </tr>
    <tr>
      <th>1837</th>
      <td>Jordan Kyrou</td>
      <td>R</td>
      <td>STL</td>
      <td>13.0</td>
      <td>6.25</td>
      <td>0.342896</td>
    </tr>
  </tbody>
</table>
</div>



Alright we have all our components! Now it's time to bring them together. Not every stat should be weighted equally. Also, combining z-scores let's us interpret and analyze new metrics as if they were normal z-scores. However, they are no longer in units of std, since the range can vary drasitcally. There are ways to bring it back, such as taking the average of all the metrics (same as having the same weight for each component), or making sure that our weights add up to 1. 

Weights:
* 30% PDO
    * Despite PDO's faults, it is the gold standard for measuring puck luck
* 30% IPP
    * "Why is it negative?" Well we want the lowest IPP to be more important
    * "But that doesn't add up to 1? You lied!" No, I did not lie. It's the same as if we inverted the column first by multiplying by -1, and then multiplied by 0.30.
* 20% Teammate G - xG
    * Teammates need to be scoring for a player to be a lucky charm. No likes "almost" goals. Some of this is captured in IPP, so we'll give it a lower weight than IPP and PDO.
* 10% G - xG and Low Danger G - Low Danger xG
    * Who doesn't want to be lucky themselves?


```python
# Weighted combination of all components
luck_df['Luck_Score'] = (
    0.30 * luck_df['component_pdo'] +                    # PDO (team on-ice luck)
    -0.30 * luck_df['component_ipp'] +                   # Involvement luck
    0.20 * luck_df['component_teammate_goals_xgoals'] +  # Linemate/team luck
    0.10 * luck_df['component_goals_xgoals'] +           # Personal finishing luck
    0.10 * luck_df['component_lowdanger']                # Shot quality luck
)

plt.figure(figsize=(10, 6))
sns.histplot(luck_df['Luck_Score'], bins=30, kde=True)
plt.title('Luck Score Distribution')
plt.xlabel('Luck Score')
plt.ylabel('Frequency')
plt.show()

```


    
![png](luck_files/luck_medium_41_0.png)
    


It worked! We have a normal distribution, so our math was correct! Our luckiest charm of the 2024-25 season was... (imagine there's a corny drum roll here) JJ Peterka! JJ Peterka had an incredible breakout season last year, and was a bright spot on an otherwise dissappointing Buffalo Sabres Team. What really set him apart from other players was his huge Teammate Goals minus Expected Goals difference. Last year he became a much better playmaker and nearly doubled his career high assists total.


```python
print("TOP 10 LUCKIEST PLAYERS IN THE NHL (2024 Season, 5v5)")

luck_df.nlargest(10, 'Luck_Score')[
    ['name', 'team', 'position', 'Luck_Score',
     'component_pdo', 'component_goals_xgoals', 'component_teammate_goals_xgoals',
     'component_ipp', 'component_lowdanger']
]

```

    TOP 10 LUCKIEST PLAYERS IN THE NHL (2024 Season, 5v5)





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
      <th>Luck_Score</th>
      <th>component_pdo</th>
      <th>component_goals_xgoals</th>
      <th>component_teammate_goals_xgoals</th>
      <th>component_ipp</th>
      <th>component_lowdanger</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>4472</th>
      <td>JJ Peterka</td>
      <td>BUF</td>
      <td>R</td>
      <td>1.754177</td>
      <td>1.420772</td>
      <td>1.816297</td>
      <td>3.126413</td>
      <td>-1.083948</td>
      <td>1.958489</td>
    </tr>
    <tr>
      <th>4537</th>
      <td>Kirill Marchenko</td>
      <td>CBJ</td>
      <td>R</td>
      <td>1.612333</td>
      <td>2.127610</td>
      <td>2.675489</td>
      <td>0.771595</td>
      <td>-0.837337</td>
      <td>3.009810</td>
    </tr>
    <tr>
      <th>2392</th>
      <td>Brayden Point</td>
      <td>TBL</td>
      <td>C</td>
      <td>1.448174</td>
      <td>2.267434</td>
      <td>1.661380</td>
      <td>1.818725</td>
      <td>-0.227574</td>
      <td>1.697885</td>
    </tr>
    <tr>
      <th>767</th>
      <td>Jake Guentzel</td>
      <td>TBL</td>
      <td>C</td>
      <td>1.269989</td>
      <td>2.159243</td>
      <td>0.371581</td>
      <td>1.940786</td>
      <td>-0.656979</td>
      <td>-0.001933</td>
    </tr>
    <tr>
      <th>3032</th>
      <td>Alex Iafallo</td>
      <td>WPG</td>
      <td>L</td>
      <td>1.242725</td>
      <td>1.565437</td>
      <td>-0.675543</td>
      <td>0.942813</td>
      <td>-2.144604</td>
      <td>0.087045</td>
    </tr>
    <tr>
      <th>1332</th>
      <td>Adrian Kempe</td>
      <td>LAK</td>
      <td>R</td>
      <td>1.180304</td>
      <td>2.287114</td>
      <td>1.068144</td>
      <td>1.344476</td>
      <td>0.013880</td>
      <td>1.226245</td>
    </tr>
    <tr>
      <th>3562</th>
      <td>Simon Benoit</td>
      <td>TOR</td>
      <td>D</td>
      <td>1.171758</td>
      <td>1.458202</td>
      <td>-0.031689</td>
      <td>1.286739</td>
      <td>-1.793812</td>
      <td>-0.580253</td>
    </tr>
    <tr>
      <th>2012</th>
      <td>Esa Lindell</td>
      <td>DAL</td>
      <td>D</td>
      <td>1.027736</td>
      <td>1.994416</td>
      <td>-0.065172</td>
      <td>1.123149</td>
      <td>-0.958009</td>
      <td>-0.761040</td>
    </tr>
    <tr>
      <th>1837</th>
      <td>Jordan Kyrou</td>
      <td>STL</td>
      <td>R</td>
      <td>1.027428</td>
      <td>1.973960</td>
      <td>2.303428</td>
      <td>0.825344</td>
      <td>0.664901</td>
      <td>2.392991</td>
    </tr>
    <tr>
      <th>732</th>
      <td>Juraj Slafkovsk</td>
      <td>MTL</td>
      <td>L</td>
      <td>1.011247</td>
      <td>1.128212</td>
      <td>-0.236128</td>
      <td>2.095865</td>
      <td>-0.606111</td>
      <td>0.953893</td>
    </tr>
  </tbody>
</table>
</div>




```python
print("TOP 10 UNLUCKIEST PLAYERS IN THE NHL (2024 Season, 5v5)")
luck_df.nsmallest(10, 'Luck_Score')[
    ['name', 'team', 'position', 'Luck_Score', 
     'component_pdo', 'component_goals_xgoals', 'component_teammate_goals_xgoals',
     'component_ipp', 'component_lowdanger']
]


```

    TOP 10 UNLUCKIEST PLAYERS IN THE NHL (2024 Season, 5v5)





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
      <th>Luck_Score</th>
      <th>component_pdo</th>
      <th>component_goals_xgoals</th>
      <th>component_teammate_goals_xgoals</th>
      <th>component_ipp</th>
      <th>component_lowdanger</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2127</th>
      <td>Sam Steel</td>
      <td>DAL</td>
      <td>C</td>
      <td>-1.586776</td>
      <td>-1.407685</td>
      <td>-0.760224</td>
      <td>-1.563049</td>
      <td>2.491995</td>
      <td>-0.282397</td>
    </tr>
    <tr>
      <th>4182</th>
      <td>Alexander Kerfoot</td>
      <td>UTA</td>
      <td>C</td>
      <td>-1.440901</td>
      <td>-1.245025</td>
      <td>-1.094082</td>
      <td>-2.189002</td>
      <td>1.375540</td>
      <td>-1.075225</td>
    </tr>
    <tr>
      <th>3037</th>
      <td>Logan O'Connor</td>
      <td>COL</td>
      <td>R</td>
      <td>-1.285091</td>
      <td>-1.548609</td>
      <td>-0.065648</td>
      <td>-1.725069</td>
      <td>1.481691</td>
      <td>-0.244229</td>
    </tr>
    <tr>
      <th>2872</th>
      <td>Conor Garland</td>
      <td>VAN</td>
      <td>R</td>
      <td>-1.283190</td>
      <td>-0.867702</td>
      <td>-0.628837</td>
      <td>-1.329323</td>
      <td>1.588155</td>
      <td>-2.176852</td>
    </tr>
    <tr>
      <th>1347</th>
      <td>Travis Konecny</td>
      <td>PHI</td>
      <td>R</td>
      <td>-1.263194</td>
      <td>-1.176164</td>
      <td>-0.623230</td>
      <td>-0.644318</td>
      <td>2.309489</td>
      <td>-0.263113</td>
    </tr>
    <tr>
      <th>147</th>
      <td>Scott Laughton</td>
      <td>TOR</td>
      <td>C</td>
      <td>-1.189860</td>
      <td>-1.792731</td>
      <td>-0.638392</td>
      <td>-0.312881</td>
      <td>1.525832</td>
      <td>-0.678760</td>
    </tr>
    <tr>
      <th>2262</th>
      <td>Brady Skjei</td>
      <td>NSH</td>
      <td>D</td>
      <td>-1.152939</td>
      <td>-1.660189</td>
      <td>0.246818</td>
      <td>-1.934162</td>
      <td>0.713596</td>
      <td>-0.786523</td>
    </tr>
    <tr>
      <th>3802</th>
      <td>Paul Cotter</td>
      <td>NJD</td>
      <td>C</td>
      <td>-1.123779</td>
      <td>-2.194399</td>
      <td>1.312482</td>
      <td>-2.047127</td>
      <td>1.108355</td>
      <td>1.452247</td>
    </tr>
    <tr>
      <th>577</th>
      <td>Jake Sanderson</td>
      <td>OTT</td>
      <td>D</td>
      <td>-1.080926</td>
      <td>-1.443615</td>
      <td>0.224575</td>
      <td>-1.889680</td>
      <td>0.781548</td>
      <td>-0.578983</td>
    </tr>
    <tr>
      <th>1432</th>
      <td>Brandon Tanev</td>
      <td>WPG</td>
      <td>L</td>
      <td>-1.071214</td>
      <td>-1.302198</td>
      <td>-0.779887</td>
      <td>-1.255948</td>
      <td>0.608273</td>
      <td>-1.688938</td>
    </tr>
  </tbody>
</table>
</div>



We can get a better idea of how much more luck JJ Peterka brings to his team, if we plot all of the components, and where JJ Peterka lies along the historgram.


```python
jj_peterka = luck_df[luck_df['name'] == 'JJ Peterka']

jj_peterka

fig, axes = plt.subplots(2, 3, figsize=(18, 10))
fig.suptitle('JJ Peterka Luck Score Components', fontsize=16, fontweight='bold')

# Component 1: PDO
axes[0, 0].hist(luck_df['component_pdo'], bins=30, edgecolor='black', alpha=0.7)
axes[0, 0].set_title('PDO Z-Score (30% weight)')
axes[0, 0].set_xlabel('Z-Score')
pdo_value = jj_peterka['component_pdo'].values[0]
axes[0, 0].axvline(pdo_value, color='red', linestyle='--', linewidth=2, label=f'JJ Peterka ({pdo_value:.2f})')
axes[0, 0].legend()

# Component 2: Goals vs xGoals
axes[0, 1].hist(luck_df['component_goals_xgoals'], bins=30, edgecolor='black', alpha=0.7)
axes[0, 1].set_title('Goals vs xGoals per 60 (10% weight)')
axes[0, 1].set_xlabel('Z-Score')
goals_xgoals_value = jj_peterka['component_goals_xgoals'].values[0]
axes[0, 1].axvline(goals_xgoals_value, color='red', linestyle='--', linewidth=2, label=f'JJ Peterka ({goals_xgoals_value:.2f})')
axes[0, 1].legend()

# Component 3: Teammate Goals vs xGoals (linemate luck)
axes[0, 2].hist(luck_df['component_teammate_goals_xgoals'], bins=30, edgecolor='black', alpha=0.7)
axes[0, 2].set_title('Teammate Goals vs xGoals per 60 (20% weight)')
axes[0, 2].set_xlabel('Z-Score (Linemate Luck)')
onice_goals_xgoals_value = jj_peterka['component_teammate_goals_xgoals'].values[0]
axes[0, 2].axvline(onice_goals_xgoals_value, color='red', linestyle='--', linewidth=2, label=f'JJ Peterka ({onice_goals_xgoals_value:.2f})')
axes[0, 2].legend()

# Component 4: IPP
axes[1, 0].hist(luck_df['component_ipp'], bins=30, edgecolor='black', alpha=0.7)
axes[1, 0].set_title('IPP Deviation (10% weight)')
axes[1, 0].set_xlabel('Z-Score')
ipp_value = jj_peterka['component_ipp'].values[0]
axes[1, 0].axvline(ipp_value, color='red', linestyle='--', linewidth=2, label=f'JJ Peterka ({ipp_value:.2f})')
axes[1, 0].legend()

# Component 5: Low Danger
axes[1, 1].hist(luck_df['component_lowdanger'], bins=30, edgecolor='black', alpha=0.7)
axes[1, 1].set_title('Low Danger Goals vs xGoals (20% weight)')
axes[1, 1].set_xlabel('Z-Score')
lowdanger_value = jj_peterka['component_lowdanger'].values[0]
axes[1, 1].axvline(lowdanger_value, color='red', linestyle='--', linewidth=2, label=f'JJ Peterka ({lowdanger_value:.2f})')
axes[1, 1].legend()

# Final Luck Score
axes[1, 2].hist(luck_df['Luck_Score'], bins=30, edgecolor='black', alpha=0.7, color='purple')
axes[1, 2].set_title('FINAL LUCK SCORE', fontweight='bold')
axes[1, 2].set_xlabel('Luck Score')
luck_score_value = jj_peterka['Luck_Score'].values[0]
axes[1, 2].axvline(luck_score_value, color='red', linestyle='--', linewidth=2, label=f'JJ Peterka ({luck_score_value:.2f})')
axes[1, 2].legend()

plt.tight_layout()
plt.show()

```


    
![png](luck_files/luck_medium_46_0.png)
    


## Interpreting Luck

Ok so here's where we get a little hand wavy, but stick with me! Let's try to make something fun out of this score. I want to know how many goals does 1 luck translate into for a team. 

What we're going to do is:

(On Ice Goals - On Ice xG) / luck

for each player, and then get the average of the On Ice Goals above expected per luck per player, we can see how many goals 1 luck is worth over a season.


```python
# calculate how many goals each team scored with the player on the ice
onice_f_g = luck_df['OnIce_F_goals']
onice_f_xg = luck_df['OnIce_F_xGoals']

luck_score = luck_df['Luck_Score']

goals_minus_xgoals = onice_f_g - onice_f_xg

# divide goals above expected by luck score to get a sense of how many goals per luck score
goals_per_luck_score = goals_minus_xgoals / luck_score

goals_per_luck_score_mean = np.mean(goals_per_luck_score)

print(f"Goals per Luck Score: {goals_per_luck_score_mean:.3f}")
```

    Goals per Luck Score: 8.876


So, JJ Peterka is good for 15.57 lucky goals from his team when he is on the ice, over the course of a whole season. If we agree that scoring 5 goals (waving our hand so often we have jazz hands), then JJ Peterka is good for 3 lucky wins on the season.


```python
luck_df['lucky_goals'] = luck_df['Luck_Score'] * goals_per_luck_score_mean

luck_df[
    ['name', 'team', 'position', 
     'OnIce_PDO_100', 'I_F_goals', 'I_F_xGoals', 
     'IPP_percentage', 'Luck_Score', 'lucky_goals']
     ].sort_values(by='lucky_goals', ascending=False).head(10)
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
      <th>OnIce_PDO_100</th>
      <th>I_F_goals</th>
      <th>I_F_xGoals</th>
      <th>IPP_percentage</th>
      <th>Luck_Score</th>
      <th>lucky_goals</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>4472</th>
      <td>JJ Peterka</td>
      <td>BUF</td>
      <td>R</td>
      <td>103.219283</td>
      <td>18.0</td>
      <td>11.20</td>
      <td>59.154930</td>
      <td>1.754177</td>
      <td>15.570500</td>
    </tr>
    <tr>
      <th>4537</th>
      <td>Kirill Marchenko</td>
      <td>CBJ</td>
      <td>R</td>
      <td>104.717091</td>
      <td>24.0</td>
      <td>12.78</td>
      <td>61.333333</td>
      <td>1.612333</td>
      <td>14.311460</td>
    </tr>
    <tr>
      <th>2392</th>
      <td>Brayden Point</td>
      <td>TBL</td>
      <td>C</td>
      <td>105.013382</td>
      <td>21.0</td>
      <td>14.34</td>
      <td>61.111111</td>
      <td>1.448174</td>
      <td>12.854343</td>
    </tr>
    <tr>
      <th>767</th>
      <td>Jake Guentzel</td>
      <td>TBL</td>
      <td>C</td>
      <td>104.784122</td>
      <td>19.0</td>
      <td>17.64</td>
      <td>56.944444</td>
      <td>1.269989</td>
      <td>11.272727</td>
    </tr>
    <tr>
      <th>3032</th>
      <td>Alex Iafallo</td>
      <td>WPG</td>
      <td>L</td>
      <td>103.525832</td>
      <td>8.0</td>
      <td>10.15</td>
      <td>50.000000</td>
      <td>1.242725</td>
      <td>11.030730</td>
    </tr>
    <tr>
      <th>1332</th>
      <td>Adrian Kempe</td>
      <td>LAK</td>
      <td>R</td>
      <td>105.055084</td>
      <td>21.0</td>
      <td>16.92</td>
      <td>68.852459</td>
      <td>1.180304</td>
      <td>10.476666</td>
    </tr>
    <tr>
      <th>3562</th>
      <td>Simon Benoit</td>
      <td>TOR</td>
      <td>D</td>
      <td>103.298599</td>
      <td>1.0</td>
      <td>1.37</td>
      <td>20.454545</td>
      <td>1.171758</td>
      <td>10.400804</td>
    </tr>
    <tr>
      <th>2012</th>
      <td>Esa Lindell</td>
      <td>DAL</td>
      <td>D</td>
      <td>104.434848</td>
      <td>3.0</td>
      <td>3.64</td>
      <td>27.272727</td>
      <td>1.027736</td>
      <td>9.122433</td>
    </tr>
    <tr>
      <th>1837</th>
      <td>Jordan Kyrou</td>
      <td>STL</td>
      <td>R</td>
      <td>104.391503</td>
      <td>25.0</td>
      <td>15.57</td>
      <td>74.603175</td>
      <td>1.027428</td>
      <td>9.119703</td>
    </tr>
    <tr>
      <th>732</th>
      <td>Juraj Slafkovsk</td>
      <td>MTL</td>
      <td>L</td>
      <td>102.599342</td>
      <td>13.0</td>
      <td>14.16</td>
      <td>62.295082</td>
      <td>1.011247</td>
      <td>8.976070</td>
    </tr>
  </tbody>
</table>
</div>



## Just Luck?

So is JJ Peterka not good, is he just lucky? **NO!!!** JJ Peterka is not good because he's lucky, he's lucky because he is good. And besides, we weren't measuring how lucky JJ Peterka is, but how lucky he is making his team! That's a very important distinction to understand. 

We even see that the highest value component for him, was how many goals his teammates scored. While he had a lower IPP than average, it's not significantly lower, just 1 std. He also became a much better playmaker in the 2024-25 season than he ever was before, which is shown in in his assists nearly doubling from 22 to 41. The low IPP can be explained by dry stretches that every young player goes through. Even still, he has improved his consistency tremendously which is what led to his career best points and assists. 

This season so far, he's already put up 7 points in his first 8 games. He's now part of a new team, new franchise, and one that just got a new logo and is in a market that's hungry for hockey. We've seen what Vegas accomplished in their first few seasons compared to Seattle. Well, one of those cities is world famous for their luck. Maybe it's not such a bad idea to take a bet on bringing in a lucky charm for a few clutch goals.

Thanks for reading!

# Thanks For Reading!

Hey everyone, thanks for reading (I know I said it 3 times in a row, sorry)! I wanted to add in this little note here since it's my first article ever on Medium. I want to try to write some fun articles on some cool anomalies, or making silly stats like this one. Hockey stats lag significantly far behind other sports like baseball and basketball, and I'd love to try to add to the community and try to make them more accessible. I'm not one who believes stats are the end all - be all, and capture everything, but they can bring interesting insights and can help inform a lot about the game. 

I've also started a YouTube channel, which I hope to be more digestible, and these blogs will be more in depth. [This is the link to this topic's video.](https://www.youtube.com/watch?v=SrwnINF2jcs)

Let me know if you enjoyed, and what your thoughts were on this!
