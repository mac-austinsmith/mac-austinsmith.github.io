# CMSC320 Final Project Prototype
---


```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
sns.set(style="darkgrid")
%matplotlib inline
# Get list of team initials to create nested for loop for schedules
teams = ['ATL','BOS','BRK','CHI','CHO','CLE','DAL','DEN','DET','GSW','HOU','IND','LAC','LAL','MEM','MIA','MIL',
         'MIN','NOP','NYK','OKC','ORL','PHI','PHO','POR','SAC','SAS','TOR','UTA','WAS']
defunct = ['NJN','CHA','CHH','SFW','SDR','SDC','BUF','VAN','NOK','NOH','SEA','KCK','KCO','CIN','NOJ','WSB','CAP',
           'BAL']
associated = {'BRK':['NJN'],'CHO':['CHA','CHH'],'GSW':['SFW'],'HOU':['SDR'],'LAC':['SDC','BUF'],'MEM':['VAN'],
             'NOP':['NOK','NOH'],'OKC':['SEA'],'SAC':['KCK','KCO','CIN'],'UTA':['NOJ'],'WAS':['WSB','CAP','BAL']}
```

## Schedule Database Builder:
    # Not feasible to run every time, export to csv
    for t in teams:
        games = [None]*51
        for i in range(0,51):
            try:
                table = pd.read_html('https://www.basketball-reference.com/teams/' + t + '/' + str(i+1969) \
                                     + '_games.html')[0]
            except:
                # print("Failed: " + t + " " + str(i+1969))
                continue
            games[i] = table
            games[i].drop(columns=['Unnamed: 3', 'Unnamed: 4', 'Unnamed: 5', 'Unnamed: 8', 'Notes'], inplace=True)
            games[i].rename(columns={'Unnamed: 7': 'Result'}, inplace=True)
            games[i].set_index(keys='G', inplace=True)
            if i < 2000:
                games[i].drop(columns=['Start (ET)'], inplace=True)
            games[i].dropna(inplace=True)
            if i >= 2000:
                games[i]['Date'] = pd.to_datetime(games[i]['Date'] + ' ' + games[i]['Start (ET)'])
                games[i].drop(columns=['Start (ET)'], inplace=True)
            else:
                games[i]['Date'] = pd.to_datetime(games[i]['Date'])
            games[i]['Record'] = games[i]['W'] +'-' + games[i]['L']
            games[i].drop(columns=['W','L'], inplace=True)
            games[i]['Margin'] = [int(x) - int(y) for x,y in zip(games[i]['Tm'],games[i]['Opp'])]
            # print("Checking: " + t + " " + str(i+1969))
            games[i].to_csv("Schedules/" + t + "_" + str(i + 1969) + '.csv')
    for t in defunct:
        games = [None]*51
        for i in range(0,51):
            try:
                table = pd.read_html('https://www.basketball-reference.com/teams/' + t + '/' + str(i+1969) \
                                     + '_games.html')[0]
            except:
                 print("Failed: " + t + " " + str(i+1969))
                continue
            games[i] = table
            games[i].drop(columns=['Unnamed: 3', 'Unnamed: 4', 'Unnamed: 5', 'Unnamed: 8', 'Notes'], inplace=True)
            games[i].rename(columns={'Unnamed: 7': 'Result'}, inplace=True)
            games[i].set_index(keys='G', inplace=True)
            if i < 2000:
                games[i].drop(columns=['Start (ET)'], inplace=True)
            games[i].dropna(inplace=True)
            if i >= 2000:
                games[i]['Date'] = pd.to_datetime(games[i]['Date'] + ' ' + games[i]['Start (ET)'])
                games[i].drop(columns=['Start (ET)'], inplace=True)
            else:
                games[i]['Date'] = pd.to_datetime(games[i]['Date'])
            games[i]['Record'] = games[i]['W'] +'-' + games[i]['L']
            games[i].drop(columns=['W','L'], inplace=True)
            games[i]['Margin'] = [int(x) - int(y) for x,y in zip(games[i]['Tm'],games[i]['Opp'])]
             print("Checking: " + t + " " + str(i+1969))
            games[i].to_csv("Schedules/" + "DEFUNCT_"+ t + "_" + str(i + 1969) + '.csv')


```python
schedules = {}
for t in teams:
    games = [None]*51
    for i in range(0,51):
        try:
            games[i] = pd.read_csv('Schedules/' + t + '_' + str(i+1969) + '.csv',index_col='G',
                                   parse_dates=['Date'],infer_datetime_format=True)
        except:
            continue
    schedules[t] = games
# Associate defunct team schedules with current team schedules
for d in defunct:
    games = [None]*51
    for i in range(0,51):
        try:
            games[i] = pd.read_csv('Schedules/' + 'DEFUNCT_' + d + '_' + str(i+1969) + '.csv',index_col='G',
                                  parse_dates=['Date'],infer_datetime_format=True)
            for a in associated:
                for x in associated[a]:
                    if x == d:
                        schedules[a][i] = games[i] 
        except:
            continue
```


```python
summary = pd.DataFrame(columns=['Team','W','L','Win_Pct'])
for t in schedules:
    wins = [None]*51
    losses = [None]*51
    i = 0
    for s in schedules[t]:
        try:
            wins[i] = pd.get_dummies(s['Result'])['W'].sum()
            losses[i] = pd.get_dummies(s['Result'])['L'].sum()
            i += 1
        except:
            continue
    wins = [x for x in wins if x is not None]
    losses = [x for x in losses if x is not None]
    win_pct = sum(wins)/(sum(wins) + sum(losses))
    summary = summary.append({'Team': t,'W': sum(wins),'L': sum(losses),'Win_Pct': win_pct}, ignore_index=True)
summary.sort_values(by='Win_Pct',ascending=False,inplace=True)
summary['Rank'] = summary['Win_Pct'].rank(ascending=False).astype(int)
summary.set_index('Rank',inplace=True)
summary
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
      <th>Team</th>
      <th>W</th>
      <th>L</th>
      <th>Win_Pct</th>
    </tr>
    <tr>
      <th>Rank</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1</td>
      <td>SAS</td>
      <td>2162</td>
      <td>1316</td>
      <td>0.621622</td>
    </tr>
    <tr>
      <td>2</td>
      <td>LAL</td>
      <td>2509</td>
      <td>1625</td>
      <td>0.606918</td>
    </tr>
    <tr>
      <td>3</td>
      <td>BOS</td>
      <td>2380</td>
      <td>1753</td>
      <td>0.575853</td>
    </tr>
    <tr>
      <td>4</td>
      <td>OKC</td>
      <td>2260</td>
      <td>1874</td>
      <td>0.546686</td>
    </tr>
    <tr>
      <td>5</td>
      <td>UTA</td>
      <td>1964</td>
      <td>1678</td>
      <td>0.539264</td>
    </tr>
    <tr>
      <td>6</td>
      <td>POR</td>
      <td>2134</td>
      <td>1836</td>
      <td>0.537531</td>
    </tr>
    <tr>
      <td>7</td>
      <td>HOU</td>
      <td>2210</td>
      <td>1924</td>
      <td>0.534591</td>
    </tr>
    <tr>
      <td>8</td>
      <td>PHO</td>
      <td>2186</td>
      <td>1948</td>
      <td>0.528786</td>
    </tr>
    <tr>
      <td>9</td>
      <td>MIA</td>
      <td>1294</td>
      <td>1200</td>
      <td>0.518845</td>
    </tr>
    <tr>
      <td>10</td>
      <td>CHI</td>
      <td>2143</td>
      <td>1991</td>
      <td>0.518384</td>
    </tr>
    <tr>
      <td>11</td>
      <td>MIL</td>
      <td>2129</td>
      <td>2005</td>
      <td>0.514998</td>
    </tr>
    <tr>
      <td>12</td>
      <td>DET</td>
      <td>2078</td>
      <td>2056</td>
      <td>0.502661</td>
    </tr>
    <tr>
      <td>13</td>
      <td>IND</td>
      <td>1744</td>
      <td>1733</td>
      <td>0.501582</td>
    </tr>
    <tr>
      <td>14</td>
      <td>DAL</td>
      <td>1572</td>
      <td>1578</td>
      <td>0.499048</td>
    </tr>
    <tr>
      <td>15</td>
      <td>ATL</td>
      <td>2048</td>
      <td>2086</td>
      <td>0.495404</td>
    </tr>
    <tr>
      <td>16</td>
      <td>NYK</td>
      <td>2039</td>
      <td>2095</td>
      <td>0.493227</td>
    </tr>
    <tr>
      <td>17</td>
      <td>DEN</td>
      <td>1703</td>
      <td>1775</td>
      <td>0.489649</td>
    </tr>
    <tr>
      <td>18</td>
      <td>PHI</td>
      <td>2022</td>
      <td>2112</td>
      <td>0.489115</td>
    </tr>
    <tr>
      <td>19</td>
      <td>GSW</td>
      <td>1996</td>
      <td>2138</td>
      <td>0.482825</td>
    </tr>
    <tr>
      <td>20</td>
      <td>ORL</td>
      <td>1158</td>
      <td>1254</td>
      <td>0.480100</td>
    </tr>
    <tr>
      <td>21</td>
      <td>TOR</td>
      <td>902</td>
      <td>1018</td>
      <td>0.469792</td>
    </tr>
    <tr>
      <td>22</td>
      <td>NOP</td>
      <td>643</td>
      <td>735</td>
      <td>0.466618</td>
    </tr>
    <tr>
      <td>23</td>
      <td>CLE</td>
      <td>1848</td>
      <td>2122</td>
      <td>0.465491</td>
    </tr>
    <tr>
      <td>24</td>
      <td>WAS</td>
      <td>1923</td>
      <td>2211</td>
      <td>0.465167</td>
    </tr>
    <tr>
      <td>25</td>
      <td>CHO</td>
      <td>1027</td>
      <td>1303</td>
      <td>0.440773</td>
    </tr>
    <tr>
      <td>26</td>
      <td>SAC</td>
      <td>1790</td>
      <td>2344</td>
      <td>0.432995</td>
    </tr>
    <tr>
      <td>27</td>
      <td>BRK</td>
      <td>1428</td>
      <td>1968</td>
      <td>0.420495</td>
    </tr>
    <tr>
      <td>28</td>
      <td>MEM</td>
      <td>792</td>
      <td>1128</td>
      <td>0.412500</td>
    </tr>
    <tr>
      <td>29</td>
      <td>LAC</td>
      <td>1610</td>
      <td>2360</td>
      <td>0.405542</td>
    </tr>
    <tr>
      <td>30</td>
      <td>MIN</td>
      <td>961</td>
      <td>1451</td>
      <td>0.398425</td>
    </tr>
  </tbody>
</table>
</div>




```python
melted = pd.melt(summary, id_vars='Team', value_vars=['W','L'], var_name='source', value_name='W & L')
plt.figure(figsize=(20,10))
p = sns.barplot(x='Team',y='W & L', hue='source',data=melted)
plt.title('NBA Wins & Losses [1969-2019]', fontweight="bold", fontsize=20)
plt.xlabel('Team')
plt.ylabel('Wins & Losses')
plt.show()
```


![png](Schedule%20Builder_files/Schedule%20Builder_5_0.png)



```python
plt.figure(figsize=(20,10))
p = sns.barplot(x='Team',y='Win_Pct',data=summary)
plt.title('NBA Winning Percentages [1969-2019]', fontweight="bold", fontsize=20)
plt.xlabel('Team')
plt.ylabel('Winning Percentage')
plt.show()
```


![png](Schedule%20Builder_files/Schedule%20Builder_6_0.png)



```python
seasons = {}
for t in schedules:
    seasons[t] = pd.DataFrame(columns=['Year','W','L','Win_Pct'])
    for s in schedules[t]:
        try:
            wins = pd.get_dummies(s['Result'])['W'].sum()
            losses = pd.get_dummies(s['Result'])['L'].sum()
        except:
            continue
        try:
            seasons[t] = seasons[t].append({'Year': pd.DatetimeIndex(s['Date']).year[0], 'W': wins, 
                                            'L': losses, 'Win_Pct': wins/(wins+losses)}, ignore_index=True)
        except:
            continue
    seasons[t].Year = seasons[t].Year.astype(int)
    seasons[t].W = seasons[t].W.astype(int)
    seasons[t].L = seasons[t].L.astype(int)
seasons['PHO']
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
      <th>Year</th>
      <th>W</th>
      <th>L</th>
      <th>Win_Pct</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>1968</td>
      <td>16</td>
      <td>66</td>
      <td>0.195122</td>
    </tr>
    <tr>
      <td>1</td>
      <td>1969</td>
      <td>39</td>
      <td>43</td>
      <td>0.475610</td>
    </tr>
    <tr>
      <td>2</td>
      <td>1970</td>
      <td>48</td>
      <td>34</td>
      <td>0.585366</td>
    </tr>
    <tr>
      <td>3</td>
      <td>1971</td>
      <td>49</td>
      <td>33</td>
      <td>0.597561</td>
    </tr>
    <tr>
      <td>4</td>
      <td>1972</td>
      <td>38</td>
      <td>44</td>
      <td>0.463415</td>
    </tr>
    <tr>
      <td>5</td>
      <td>1973</td>
      <td>30</td>
      <td>52</td>
      <td>0.365854</td>
    </tr>
    <tr>
      <td>6</td>
      <td>1974</td>
      <td>32</td>
      <td>50</td>
      <td>0.390244</td>
    </tr>
    <tr>
      <td>7</td>
      <td>1975</td>
      <td>42</td>
      <td>40</td>
      <td>0.512195</td>
    </tr>
    <tr>
      <td>8</td>
      <td>1976</td>
      <td>34</td>
      <td>48</td>
      <td>0.414634</td>
    </tr>
    <tr>
      <td>9</td>
      <td>1977</td>
      <td>49</td>
      <td>33</td>
      <td>0.597561</td>
    </tr>
    <tr>
      <td>10</td>
      <td>1978</td>
      <td>50</td>
      <td>32</td>
      <td>0.609756</td>
    </tr>
    <tr>
      <td>11</td>
      <td>1979</td>
      <td>55</td>
      <td>27</td>
      <td>0.670732</td>
    </tr>
    <tr>
      <td>12</td>
      <td>1980</td>
      <td>57</td>
      <td>25</td>
      <td>0.695122</td>
    </tr>
    <tr>
      <td>13</td>
      <td>1981</td>
      <td>46</td>
      <td>36</td>
      <td>0.560976</td>
    </tr>
    <tr>
      <td>14</td>
      <td>1982</td>
      <td>53</td>
      <td>29</td>
      <td>0.646341</td>
    </tr>
    <tr>
      <td>15</td>
      <td>1983</td>
      <td>41</td>
      <td>41</td>
      <td>0.500000</td>
    </tr>
    <tr>
      <td>16</td>
      <td>1984</td>
      <td>36</td>
      <td>46</td>
      <td>0.439024</td>
    </tr>
    <tr>
      <td>17</td>
      <td>1985</td>
      <td>32</td>
      <td>50</td>
      <td>0.390244</td>
    </tr>
    <tr>
      <td>18</td>
      <td>1986</td>
      <td>36</td>
      <td>46</td>
      <td>0.439024</td>
    </tr>
    <tr>
      <td>19</td>
      <td>1987</td>
      <td>28</td>
      <td>54</td>
      <td>0.341463</td>
    </tr>
    <tr>
      <td>20</td>
      <td>1988</td>
      <td>55</td>
      <td>27</td>
      <td>0.670732</td>
    </tr>
    <tr>
      <td>21</td>
      <td>1989</td>
      <td>54</td>
      <td>28</td>
      <td>0.658537</td>
    </tr>
    <tr>
      <td>22</td>
      <td>1990</td>
      <td>55</td>
      <td>27</td>
      <td>0.670732</td>
    </tr>
    <tr>
      <td>23</td>
      <td>1991</td>
      <td>53</td>
      <td>29</td>
      <td>0.646341</td>
    </tr>
    <tr>
      <td>24</td>
      <td>1992</td>
      <td>62</td>
      <td>20</td>
      <td>0.756098</td>
    </tr>
    <tr>
      <td>25</td>
      <td>1993</td>
      <td>56</td>
      <td>26</td>
      <td>0.682927</td>
    </tr>
    <tr>
      <td>26</td>
      <td>1994</td>
      <td>59</td>
      <td>23</td>
      <td>0.719512</td>
    </tr>
    <tr>
      <td>27</td>
      <td>1995</td>
      <td>41</td>
      <td>41</td>
      <td>0.500000</td>
    </tr>
    <tr>
      <td>28</td>
      <td>1996</td>
      <td>40</td>
      <td>42</td>
      <td>0.487805</td>
    </tr>
    <tr>
      <td>29</td>
      <td>1997</td>
      <td>56</td>
      <td>26</td>
      <td>0.682927</td>
    </tr>
    <tr>
      <td>30</td>
      <td>1999</td>
      <td>27</td>
      <td>23</td>
      <td>0.540000</td>
    </tr>
    <tr>
      <td>31</td>
      <td>1999</td>
      <td>53</td>
      <td>29</td>
      <td>0.646341</td>
    </tr>
    <tr>
      <td>32</td>
      <td>2000</td>
      <td>51</td>
      <td>31</td>
      <td>0.621951</td>
    </tr>
    <tr>
      <td>33</td>
      <td>2001</td>
      <td>36</td>
      <td>46</td>
      <td>0.439024</td>
    </tr>
    <tr>
      <td>34</td>
      <td>2002</td>
      <td>44</td>
      <td>38</td>
      <td>0.536585</td>
    </tr>
    <tr>
      <td>35</td>
      <td>2003</td>
      <td>29</td>
      <td>53</td>
      <td>0.353659</td>
    </tr>
    <tr>
      <td>36</td>
      <td>2004</td>
      <td>62</td>
      <td>20</td>
      <td>0.756098</td>
    </tr>
    <tr>
      <td>37</td>
      <td>2005</td>
      <td>54</td>
      <td>28</td>
      <td>0.658537</td>
    </tr>
    <tr>
      <td>38</td>
      <td>2006</td>
      <td>61</td>
      <td>21</td>
      <td>0.743902</td>
    </tr>
    <tr>
      <td>39</td>
      <td>2007</td>
      <td>55</td>
      <td>27</td>
      <td>0.670732</td>
    </tr>
    <tr>
      <td>40</td>
      <td>2008</td>
      <td>46</td>
      <td>36</td>
      <td>0.560976</td>
    </tr>
    <tr>
      <td>41</td>
      <td>2009</td>
      <td>54</td>
      <td>28</td>
      <td>0.658537</td>
    </tr>
    <tr>
      <td>42</td>
      <td>2010</td>
      <td>40</td>
      <td>42</td>
      <td>0.487805</td>
    </tr>
    <tr>
      <td>43</td>
      <td>2011</td>
      <td>33</td>
      <td>33</td>
      <td>0.500000</td>
    </tr>
    <tr>
      <td>44</td>
      <td>2012</td>
      <td>25</td>
      <td>57</td>
      <td>0.304878</td>
    </tr>
    <tr>
      <td>45</td>
      <td>2013</td>
      <td>48</td>
      <td>34</td>
      <td>0.585366</td>
    </tr>
    <tr>
      <td>46</td>
      <td>2014</td>
      <td>39</td>
      <td>43</td>
      <td>0.475610</td>
    </tr>
    <tr>
      <td>47</td>
      <td>2015</td>
      <td>23</td>
      <td>59</td>
      <td>0.280488</td>
    </tr>
    <tr>
      <td>48</td>
      <td>2016</td>
      <td>24</td>
      <td>58</td>
      <td>0.292683</td>
    </tr>
    <tr>
      <td>49</td>
      <td>2017</td>
      <td>21</td>
      <td>61</td>
      <td>0.256098</td>
    </tr>
    <tr>
      <td>50</td>
      <td>2018</td>
      <td>19</td>
      <td>63</td>
      <td>0.231707</td>
    </tr>
  </tbody>
</table>
</div>




```python
melted = pd.melt(seasons['PHO'], id_vars='Year', value_vars=['W','L'], var_name='source', value_name='W & L')
plt.figure(figsize=(20,10))
p = sns.barplot(x='Year',y='W & L', hue='source',data=melted)
plt.title('Phoenix Suns Wins & Losses per Year [1969-2019]', fontweight="bold", fontsize=20)
plt.xlabel('Year')
plt.ylabel('Wins & Losses')
plt.xticks(rotation='vertical')
plt.show()
```


![png](Schedule%20Builder_files/Schedule%20Builder_8_0.png)



```python
fig, ax = plt.subplots(figsize=(20, 10))
sns.lineplot(x='Year', y='Win_Pct', data=seasons['PHO'], ax=ax)
x = seasons['PHO'].Year.values[36]
y = seasons['PHO'].Win_Pct.values[36]
ax.annotate('Steve Nash Era', xy=(x+3.5, y-.32), xycoords='data', size=13, ha='center', textcoords='data')
ax.annotate('', xy=(x, y-.3), xytext=(x+7, y-.3), xycoords='data', textcoords='data',
            arrowprops=dict(arrowstyle='|-|,widthA=0.2,widthB=0.2', color='black'))
plt.title('Phoenix Suns Winning Percentage over Time', fontweight="bold", fontsize=20)
plt.xlabel('Year')
plt.ylabel('Winning Percentage')
plt.show()
```


![png](Schedule%20Builder_files/Schedule%20Builder_9_0.png)



```python
!jupyter nbconvert --to md 'Schedule Builder'.ipynb
```

    Traceback (most recent call last):
      File "/home/macsmith/miniconda3/bin/jupyter-nbconvert", line 11, in <module>
        sys.exit(main())
      File "/home/macsmith/miniconda3/lib/python3.7/site-packages/jupyter_core/application.py", line 266, in launch_instance
        return super(JupyterApp, cls).launch_instance(argv=argv, **kwargs)
      File "/home/macsmith/miniconda3/lib/python3.7/site-packages/traitlets/config/application.py", line 658, in launch_instance
        app.start()
      File "/home/macsmith/miniconda3/lib/python3.7/site-packages/nbconvert/nbconvertapp.py", line 340, in start
        self.convert_notebooks()
      File "/home/macsmith/miniconda3/lib/python3.7/site-packages/nbconvert/nbconvertapp.py", line 499, in convert_notebooks
        cls = get_exporter(self.export_format)
      File "/home/macsmith/miniconda3/lib/python3.7/site-packages/nbconvert/exporters/base.py", line 113, in get_exporter
        % (name, ', '.join(get_export_names())))
    ValueError: Unknown exporter "md", did you mean one of: asciidoc, custom, html, latex, markdown, notebook, pdf, python, rst, script, slides?

