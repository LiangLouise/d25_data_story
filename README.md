# What's The Hotest Anime in Each Season?


## Abstract

No matter any kind of many screenplays, the most common topics among those watcher or fans are probably:

- Which *xxx* is the best?
- Which *xxx* do you recommend?
- Do you think what's the best *xxx* of 2021?

*xxx* here can be substituted by any type of screenplays, i.e. movies, TV series, TV show and etc. This rule applies to one type of screenplays as well - [TV anime][3].

In this post, I would present with the help the data from [anime community][1] in Reddit and [anime record data][2] to explore what's most welcomed TV animes in different period of time and what are the potential features make them become popular among viewers.

## Background

TV series in US usually have 23-24 episode as a "full season" and many of them run across the fall and winter, in between late September to May of the next year. However, unlike the conventions in North America, anime producers in Japan had very different traditions. The TV animes in Japan are usually played by seasons and last for three months, containing 12-13 episodes:
    
- Winter: January - March
- Spring: April - June
- Summer: July - September
- Fall: October - December

Each season, there would be around 40 - 60 animes get released and each year, there would be more than 350 TV anime produced. 

Besides, the genres of the animes could very rich, covering a lot of topics and multiple themes.

![](report_imgs/catgory.png)

[1]: https://www.reddit.com/r/anime/ "Reddit Anime"
[2]: https://github.com/manami-project/anime-offline-database	"anime-offline-database"
[3]: https://en.wikipedia.org/wiki/Anime "Anime Wiki Page"

<script src="https://cdn.plot.ly/plotly-2.6.3.min.js"></script>


## Finding the Hotest Anime in Different Period

### How to Extract the Data We Want?

First of all, we need to determine what submissions are we actually want. Unlike IMDB or Rotten Tomatoes, there is no individual page in reddit for each anime so that people would only discuss or review that specific work under such page. The topics could be relatively spare and board. This can be also proved by a word cloud.


```python
from wordcloud import WordCloud
import matplotlib.pyplot as plt

wordcloud = WordCloud(width = 1000, height = 600, background_color="white",
                min_font_size = 16, font_step=2)
wordcloud.generate(sub_titles['text'].str.cat(sep=' '))
plt.figure(figsize=(20, 10))
plt.imshow(wordcloud, interpolation='bilinear')
plt.axis("off")
plt.show()
```


    
![png](output_2_0.png)
    

> Word cloud of title data among all submission in anime subreddit from `text_submissions` dataset

From the word cloud we can see that, people discussed a lot topics in this subreddit, including but not limited to plots, characters they like, anime recommendations. It would be difficult for us to determine if they are talking about a specific anime and if they are talking about the animes currently on air or not by simply applying `LatentDirichletAllocation` from scikit-learn. As it's very likely that the model will fail to extract the name of the anime properly.

Fortunately, The moderators of the anime subreddit and other contributors wrote a post bot[4] which monitors the latest streaming info and will create a post automatically for each episode of anime after it's released. And this is bot is currently operates under the account *AutoLovepon*.

![sub_one_punch](report_imgs/sub_one_punch.png)

> A typical discussion submission created by this bot

[4]: https://github.com/r-anime/holo "holo"


Then we can simple go through the full data set we have, select the posts created by `AutoLovepon`, and use regex to extract anime title and episode number. Besides, since the full data set only contains meta data, I also used the [praw](https://github.com/praw-dev/praw) library to help obtain the title name from reddit. As a result, we can acquire the data like below:

- `anime`: title of anime
- `created_utc`: date this submission created, which can be used to identify which season this anime belongs to
- `com_mean`, `com_median`, `com_count`: The mean, median of the scores of the comments under this submission and count the total comments
- `score`: The score of this submission


```python
sta_19[sta_19['anime'] == "One Punch Man Season 2"].head()
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
      <th>id</th>
      <th>created_utc</th>
      <th>anime</th>
      <th>season</th>
      <th>ep</th>
      <th>com_mean</th>
      <th>com_median</th>
      <th>com_count</th>
      <th>score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>749</th>
      <td>t3_bbatev</td>
      <td>2019-04-09 17:34:10</td>
      <td>One Punch Man Season 2</td>
      <td>2</td>
      <td>1.0</td>
      <td>26.809337</td>
      <td>2.0</td>
      <td>2035.0</td>
      <td>7756</td>
    </tr>
    <tr>
      <th>784</th>
      <td>t3_bdwte2</td>
      <td>2019-04-16 17:40:48</td>
      <td>One Punch Man Season 2</td>
      <td>2</td>
      <td>2.0</td>
      <td>24.634706</td>
      <td>2.0</td>
      <td>1919.0</td>
      <td>3743</td>
    </tr>
    <tr>
      <th>830</th>
      <td>t3_bgja8c</td>
      <td>2019-04-23 17:39:02</td>
      <td>One Punch Man Season 2</td>
      <td>2</td>
      <td>3.0</td>
      <td>42.697674</td>
      <td>3.0</td>
      <td>1290.0</td>
      <td>5468</td>
    </tr>
    <tr>
      <th>881</th>
      <td>t3_bj65ne</td>
      <td>2019-04-30 17:37:40</td>
      <td>One Punch Man Season 2</td>
      <td>2</td>
      <td>4.0</td>
      <td>28.279570</td>
      <td>3.0</td>
      <td>930.0</td>
      <td>3887</td>
    </tr>
    <tr>
      <th>927</th>
      <td>t3_bltood</td>
      <td>2019-05-07 17:42:06</td>
      <td>One Punch Man Season 2</td>
      <td>2</td>
      <td>5.0</td>
      <td>22.986000</td>
      <td>3.0</td>
      <td>1000.0</td>
      <td>2530</td>
    </tr>
  </tbody>
</table>
</div>



### Rank Top 5 Animes of each season

The score of each discussion submission the most obvious data we can use to compare. Thus, we can calculate the mean of each anime's discussion submission's mean and rank them.



```python
import plotly.express as px

top5_by_score = mean_scores.groupby('season').apply(lambda x: x.sort_values(by='score', ascending=False, na_position='first').head(5).reset_index()).droplevel(0)

fig = px.line(
    top5_by_score, 
    x=top5_by_score.index, 
    y='score', 
    color='season', 
    symbol='season', 
    hover_data=['anime'],
    labels={
        "index": "Rank",
        "score": "Mean of Submission Score",
        "season": "Season"
    },
)
fig.show()
```


```python
rank_2019 = rank_seasons(2019)
rank_2019
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
      <th>anime</th>
      <th>score</th>
      <th>season</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Kaguya-sama wa Kokurasetai: Tensai-tachi no Re...</td>
      <td>7204.916667</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Mob Psycho 100 Season 2</td>
      <td>6884.000000</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Yakusoku no Neverland</td>
      <td>4324.083333</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Tate no Yuusha no Nariagari</td>
      <td>3800.400000</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Tensei shitara Slime Datta Ken</td>
      <td>3382.166667</td>
      <td>1</td>
    </tr>
    <tr>
      <th>0</th>
      <td>Shingeki no Kyojin Season 3</td>
      <td>10257.600000</td>
      <td>2</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Kimetsu no Yaiba</td>
      <td>4872.259259</td>
      <td>2</td>
    </tr>
    <tr>
      <th>2</th>
      <td>One Punch Man Season 2</td>
      <td>3825.666667</td>
      <td>2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Isekai Quartet</td>
      <td>2550.916667</td>
      <td>2</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Hitori Bocchi no ○○ Seikatsu</td>
      <td>1643.750000</td>
      <td>2</td>
    </tr>
    <tr>
      <th>0</th>
      <td>Dr. Stone</td>
      <td>4207.291667</td>
      <td>3</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Vinland Saga</td>
      <td>4027.791667</td>
      <td>3</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Enen no Shouboutai</td>
      <td>2533.791667</td>
      <td>3</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Tsuujou Kougeki ga Zentai Kougeki de Ni-kai Ko...</td>
      <td>1760.000000</td>
      <td>3</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Dungeon ni Deai o Motomeru no wa Machigatte Ir...</td>
      <td>1641.583333</td>
      <td>3</td>
    </tr>
    <tr>
      <th>0</th>
      <td>Boku no Hero Academia Season 4</td>
      <td>4608.181818</td>
      <td>4</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Sword Art Online: Alicization - War of Underworld</td>
      <td>2264.833333</td>
      <td>4</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Fate/Grand Order: Zettai Majuu Sensen Babylonia</td>
      <td>2143.090909</td>
      <td>4</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Shinchou Yuusha: Kono Yuusha ga Ore Tueee Kuse...</td>
      <td>2067.833333</td>
      <td>4</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Ore o Suki na no wa Omae Dake ka yo</td>
      <td>1840.363636</td>
      <td>4</td>
    </tr>
  </tbody>
</table>
</div>




```python
rank_2020 = rank_seasons(2020)
rank_2020
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
      <th>anime</th>
      <th>score</th>
      <th>season</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Boku no Hero Academia Season 4</td>
      <td>4224.571429</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Fate/Grand Order: Zettai Majuu Sensen Babylonia</td>
      <td>2351.200000</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Eizouken ni wa Te wo Dasu na!</td>
      <td>2117.833333</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Itai no wa Iya nano de Bougyoryoku ni Kyokufur...</td>
      <td>2000.166667</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Haikyuu!! To the Top</td>
      <td>1984.384615</td>
      <td>1</td>
    </tr>
    <tr>
      <th>0</th>
      <td>Kaguya-sama wa Kokurasetai?: Tensai-tachi no R...</td>
      <td>10105.000000</td>
      <td>2</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Kaguya-sama wa Kokurasetai?: Tensai-tachi no R...</td>
      <td>9362.000000</td>
      <td>2</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Kami no Tou</td>
      <td>8229.000000</td>
      <td>2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Kami no Tou: Tower of God</td>
      <td>8040.500000</td>
      <td>2</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Otome Game no Hametsu Flag shika Nai Akuyaku R...</td>
      <td>3128.000000</td>
      <td>2</td>
    </tr>
    <tr>
      <th>0</th>
      <td>Re:Zero kara Hajimeru Isekai Seikatsu Season 2</td>
      <td>12289.615385</td>
      <td>3</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Yahari Ore no Seishun Love Comedy wa Machigatt...</td>
      <td>6253.833333</td>
      <td>3</td>
    </tr>
    <tr>
      <th>2</th>
      <td>The God of High School</td>
      <td>4913.384615</td>
      <td>3</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Maou Gakuin no Futekigousha: Shijou Saikyou no...</td>
      <td>3780.916667</td>
      <td>3</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Sword Art Online: Alicization - War of Underwo...</td>
      <td>3347.000000</td>
      <td>3</td>
    </tr>
    <tr>
      <th>0</th>
      <td>Shingeki no Kyojin: The Final Season</td>
      <td>16821.500000</td>
      <td>4</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Jujutsu Kaisen</td>
      <td>6458.266667</td>
      <td>4</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Haikyuu!! To the Top 2nd Season</td>
      <td>4450.000000</td>
      <td>4</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Haikyuu!!: To the Top Part 2</td>
      <td>2831.363636</td>
      <td>4</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Higurashi no Naku Koro ni [Reboot only thread]</td>
      <td>2758.666667</td>
      <td>4</td>
    </tr>
  </tbody>
</table>
</div>




```python
rank_2021 = rank_seasons(2021)
rank_2021
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
      <th>anime</th>
      <th>score</th>
      <th>season</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Shingeki no Kyojin: The Final Season</td>
      <td>18219.307692</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Re:Zero kara Hajimeru Isekai Seikatsu Season 2...</td>
      <td>12320.250000</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Jujutsu Kaisen</td>
      <td>10981.727273</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Mushoku Tensei: Isekai Ittara Honki Dasu</td>
      <td>8043.727273</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Horimiya</td>
      <td>6959.461538</td>
      <td>1</td>
    </tr>
    <tr>
      <th>0</th>
      <td>86 EIGHTY-SIX</td>
      <td>7757.090909</td>
      <td>2</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Vivy: Fluorite Eye's Song</td>
      <td>5536.461538</td>
      <td>2</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Fumetsu no Anata e</td>
      <td>5449.750000</td>
      <td>2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Ijiranaide, Nagatoro-san</td>
      <td>4158.750000</td>
      <td>2</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Hige wo Soru. Soshite Joshikousei wo Hirou.</td>
      <td>3315.076923</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>



Since we have both score of a submission and score of a comments, I ranked the animes with

* mean of each submission's score



```python
import plotly.express as px
import plotly.io as pio 
pio.renderers.default='iframe'

top5_by_score = mean_scores.groupby('season').apply(lambda x: x.sort_values(by='score', ascending=False, na_position='first').head(5).reset_index()).droplevel(0)

fig_1 = px.line(
    top5_by_score, 
    x=top5_by_score.index, 
    y='score', 
    color='season', 
    symbol='season', 
    hover_data=['anime'],
    labels={
        "index": "Rank",
        "score": "Mean of Submission Score",
        "season": "Season"
    },
)

fig_1.show()
```


<iframe
    scrolling="no"
    width="100%"
    height="545px"
    src="iframe_figures/figure_32.html"
    frameborder="0"
    allowfullscreen
></iframe>



* mean of total comments' score under the same submission


```python
sta_19.fillna(0, inplace=True)
mean_com_count = sta_19.groupby('anime').agg({'com_count': 'mean', 'season': 'min'})
top5_by_com_count = mean_com_count.groupby('season').apply(lambda x: x.sort_values(by='com_count', ascending=False, na_position='first').reset_index().head(5)).droplevel(0)

fig_2 = px.line(
    top5_by_com_count, 
    x=top5_by_com_count.index, 
    y='com_count', 
    color='season', 
    symbol='season', 
    hover_data=['anime'],
    labels={
        "index": "Rank",
        "com_count": "Mean of comment's count",
        "season": "Season"
    },
)
fig_2.show()

```


<iframe
    scrolling="no"
    width="100%"
    height="545px"
    src="iframe_figures/figure_28.html"
    frameborder="0"
    allowfullscreen
></iframe>



* median of total comments' score under the same submission


```python
med_com_score = sta_19.groupby('anime').agg({'com_median': 'mean', 'season': 'min'})
top5_by_com_score_med = med_com_score.groupby('season').apply(lambda x: x.sort_values(by='com_median', ascending=False, na_position='first').reset_index().head(5)).droplevel(0)

fig_3 = px.line(
    top5_by_com_score_med, 
    x=top5_by_com_score_med.index, 
    y='com_median', 
    color='season', 
    symbol='season', 
    hover_data=['anime'],
    labels={
        "index": "Rank",
        "com_median": "Median of comment's Score",
        "season": "Season"
    },
)
fig_3.show()
```


<iframe
    scrolling="no"
    width="100%"
    height="545px"
    src="iframe_figures/figure_27.html"
    frameborder="0"
    allowfullscreen
></iframe>



* total number of comments under the same submission


```python
sta_19.fillna(0, inplace=True)
mean_com_score = sta_19.groupby('anime').agg({'com_mean': 'mean', 'season': 'min'})
top5_by_com_mean = mean_com_score.groupby('season').apply(lambda x: x.sort_values(by='com_mean', ascending=False, na_position='first').reset_index().head(5)).droplevel(0)

fig_4 = px.line(
    top5_by_com_mean, 
    x=top5_by_com_mean.index, 
    y='com_mean', 
    color='season', 
    symbol='season', 
    hover_data=['anime'],
    labels={
        "index": "Rank",
        "com_mean": "Mean of comment's Score",
        "season": "Season"
    },
)

fig_4.to_html("")

fig_4.show()
```


<iframe
    scrolling="no"
    width="100%"
    height="545px"
    src="iframe_figures/figure_26.html"
    frameborder="0"
    allowfullscreen
></iframe>



So we now can see that Each of these metric would produce a relatively different result. In order to test if we should keep both of them in our ranking model as metric, I tested if they are relevant, particularly mean of submission's score and mean of comment's score.


```python
sta_19['comment_total'] = sta_19['com_mean'] * sta_19['com_count']
sta_19.plot.scatter(x='com_mean', y='score', logy=True, logx=True)
sta_19.plot.scatter(x='comment_total', y='score', logy=True, logx=True)
```




    <AxesSubplot:xlabel='comment_total', ylabel='score'>




    
![png](output_20_1.png)
    



    
![png](output_20_2.png)
    


Apparently, both the mean of and the sum of the comments' score are highly relevant to submission's score, thus we only need to rely on the score of the submission to rank the animes. Then we can rank the anime in each season from 2019 to 2021 using the mean score of the discussion submission post.

Now we can rank the top 5 animes in each season of each year as below

* 2019


```python
rank_2019 = rank_seasons(2019)
rank_2019
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
      <th>anime</th>
      <th>score</th>
      <th>season</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Kaguya-sama wa Kokurasetai: Tensai-tachi no Re...</td>
      <td>7204.916667</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Mob Psycho 100 Season 2</td>
      <td>6884.000000</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Yakusoku no Neverland</td>
      <td>4324.083333</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Tate no Yuusha no Nariagari</td>
      <td>3800.400000</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Tensei shitara Slime Datta Ken</td>
      <td>3382.166667</td>
      <td>1</td>
    </tr>
    <tr>
      <th>0</th>
      <td>Shingeki no Kyojin Season 3</td>
      <td>10257.600000</td>
      <td>2</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Kimetsu no Yaiba</td>
      <td>4872.259259</td>
      <td>2</td>
    </tr>
    <tr>
      <th>2</th>
      <td>One Punch Man Season 2</td>
      <td>3825.666667</td>
      <td>2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Isekai Quartet</td>
      <td>2550.916667</td>
      <td>2</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Hitori Bocchi no ○○ Seikatsu</td>
      <td>1643.750000</td>
      <td>2</td>
    </tr>
    <tr>
      <th>0</th>
      <td>Dr. Stone</td>
      <td>4207.291667</td>
      <td>3</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Vinland Saga</td>
      <td>4027.791667</td>
      <td>3</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Enen no Shouboutai</td>
      <td>2533.791667</td>
      <td>3</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Tsuujou Kougeki ga Zentai Kougeki de Ni-kai Ko...</td>
      <td>1760.000000</td>
      <td>3</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Dungeon ni Deai o Motomeru no wa Machigatte Ir...</td>
      <td>1641.583333</td>
      <td>3</td>
    </tr>
    <tr>
      <th>0</th>
      <td>Boku no Hero Academia Season 4</td>
      <td>4608.181818</td>
      <td>4</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Sword Art Online: Alicization - War of Underworld</td>
      <td>2264.833333</td>
      <td>4</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Fate/Grand Order: Zettai Majuu Sensen Babylonia</td>
      <td>2143.090909</td>
      <td>4</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Shinchou Yuusha: Kono Yuusha ga Ore Tueee Kuse...</td>
      <td>2067.833333</td>
      <td>4</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Ore o Suki na no wa Omae Dake ka yo</td>
      <td>1840.363636</td>
      <td>4</td>
    </tr>
  </tbody>
</table>
</div>



* 2020


```python
rank_2020 = rank_seasons(2020)
rank_2020
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
      <th>anime</th>
      <th>score</th>
      <th>season</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Boku no Hero Academia Season 4</td>
      <td>4224.571429</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Fate/Grand Order: Zettai Majuu Sensen Babylonia</td>
      <td>2351.200000</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Eizouken ni wa Te wo Dasu na!</td>
      <td>2117.833333</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Itai no wa Iya nano de Bougyoryoku ni Kyokufur...</td>
      <td>2000.166667</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Haikyuu!! To the Top</td>
      <td>1984.384615</td>
      <td>1</td>
    </tr>
    <tr>
      <th>0</th>
      <td>Kaguya-sama wa Kokurasetai?: Tensai-tachi no R...</td>
      <td>10105.000000</td>
      <td>2</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Kaguya-sama wa Kokurasetai?: Tensai-tachi no R...</td>
      <td>9362.000000</td>
      <td>2</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Kami no Tou</td>
      <td>8229.000000</td>
      <td>2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Kami no Tou: Tower of God</td>
      <td>8040.500000</td>
      <td>2</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Otome Game no Hametsu Flag shika Nai Akuyaku R...</td>
      <td>3128.000000</td>
      <td>2</td>
    </tr>
    <tr>
      <th>0</th>
      <td>Re:Zero kara Hajimeru Isekai Seikatsu Season 2</td>
      <td>12289.615385</td>
      <td>3</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Yahari Ore no Seishun Love Comedy wa Machigatt...</td>
      <td>6253.833333</td>
      <td>3</td>
    </tr>
    <tr>
      <th>2</th>
      <td>The God of High School</td>
      <td>4913.384615</td>
      <td>3</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Maou Gakuin no Futekigousha: Shijou Saikyou no...</td>
      <td>3780.916667</td>
      <td>3</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Sword Art Online: Alicization - War of Underwo...</td>
      <td>3347.000000</td>
      <td>3</td>
    </tr>
    <tr>
      <th>0</th>
      <td>Shingeki no Kyojin: The Final Season</td>
      <td>16821.500000</td>
      <td>4</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Jujutsu Kaisen</td>
      <td>6458.266667</td>
      <td>4</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Haikyuu!! To the Top 2nd Season</td>
      <td>4450.000000</td>
      <td>4</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Haikyuu!!: To the Top Part 2</td>
      <td>2831.363636</td>
      <td>4</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Higurashi no Naku Koro ni [Reboot only thread]</td>
      <td>2758.666667</td>
      <td>4</td>
    </tr>
  </tbody>
</table>
</div>



* 2021 (First two season)


```python
rank_2021 = rank_seasons(2021)
rank_2021
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
      <th>anime</th>
      <th>score</th>
      <th>season</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Shingeki no Kyojin: The Final Season</td>
      <td>18219.307692</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Re:Zero kara Hajimeru Isekai Seikatsu Season 2...</td>
      <td>12320.250000</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Jujutsu Kaisen</td>
      <td>10981.727273</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Mushoku Tensei: Isekai Ittara Honki Dasu</td>
      <td>8043.727273</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Horimiya</td>
      <td>6959.461538</td>
      <td>1</td>
    </tr>
    <tr>
      <th>0</th>
      <td>86 EIGHTY-SIX</td>
      <td>7757.090909</td>
      <td>2</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Vivy: Fluorite Eye's Song</td>
      <td>5536.461538</td>
      <td>2</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Fumetsu no Anata e</td>
      <td>5449.750000</td>
      <td>2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Ijiranaide, Nagatoro-san</td>
      <td>4158.750000</td>
      <td>2</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Hige wo Soru. Soshite Joshikousei wo Hirou.</td>
      <td>3315.076923</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>



Furthermore, we then can generate the ranks in all three years we have in a facet plot which is categorized by year:


```python
rank_2019['year'] = 2019
rank_2020['year'] = 2020
rank_2021['year'] = 2021

total = rank_2019.append(rank_2020).append(rank_2021)
fig = px.line(
        total, 
        x=total.index, 
        y='score', 
        color='season', 
        symbol='season',
        facet_col='year',
        hover_data=['anime'],
        labels={
            "index": "Rank",
            "score": "Mean of Submission Score",
            "season": "Season"
        },
    )

sp = total[total['anime'].str.contains('Shingeki no Kyojin')]
sp_data = px.scatter(
        sp, 
        x=sp.index, 
        y='score', 
        text="anime",
        facet_col='year'
    ).update_traces(mode="text")["data"]

for trace in sp_data:
    fig.add_trace(trace)

save_ploty(f"{Export_Path}/trend.html", [fig])

fig.show()
```


<iframe
    scrolling="no"
    width="100%"
    height="545px"
    src="iframe_figures/figure_43.html"
    frameborder="0"
    allowfullscreen
></iframe>



Based on the facet plot above, over the years, the community is actually more and more active, as we can see that the mean of the score is increasing. 

And we can see that some series is extremely welcomed in fact, the season 3 and season 4 (which has 23 episodes) of *Shingeki no Kyojin* were released in season 2 2019, season 4 2020 and season 1 2021, The average score they received are significantly higher than the usual animes.

Therefore, Apart from knowing which Anime is the hotest in it's own season we can also see that *Shingeki no Kyojin* is the most popular anime in the community.

## Explore What Would Make an Anime Hot

Since in the full dataset, we were only given the meta data of each submission and comment and the text data set is only a small subset of the full dataset. We are unable to do much further analysis to detect what would make an anime popular. 

Luckily, during doing this project, I also encountered an interesting dataset [anime-offline-database](https://github.com/manami-project/anime-offline-database), which contains the tag of each anime so we can pick up the tag data from it.

The columns I care about are:

- `title`: The name of anime
- `tags`: list of tags of the anime

we can try look into what kind of theme or subject would make a anime popular, in other words, try to connect the presence of tag in the anime with the score it can and build a model to predict the score with tag data.

### Merge the Records

The only difficulty is that name of the same anime would be slightly different in two dataset, making us hard to do the exact match. The common differences are:

- Lower cases vs. Upper cases
- Series numbering
- Translation Habits

Some examples are shown as below:


```python
merged_table[merged_table['anime'] != merged_table['anime_title']][['anime', 'anime_title']].head()
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
      <th>anime</th>
      <th>anime_title</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>3D Kanojo: Real Girl Season 2</td>
      <td>3D Kanojo: Real Girl 2nd Season</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Africa no Salaryman</td>
      <td>Africa no Salaryman (TV)</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Aikatsu Friends!</td>
      <td>Aikatsu on Parade!</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Ani ni Tsukeru Kusuri wa Nai! Season 3</td>
      <td>Ani ni Tsukeru Kusuri wa Nai! 3</td>
    </tr>
    <tr>
      <th>14</th>
      <td>BEM</td>
      <td>EMOMOMO</td>
    </tr>
  </tbody>
</table>
</div>



So I solved the problem with Python's built-in difflib to get the closet match of anime name in two dataset, then try to merge the animes which have name presented in the search result.


```python
def find_closet(x):
    # perform fuzzy search on the anime title from reddit dataset 
    # with title in the anime-offline-database   
    res = difflib.get_close_matches(x, anime_meta_year['title'], cutoff=0.4)
    if len(res) == 0:
        return np.nan
    return res[0]
```

As a result, only 50 animes were dropped due to no matchin, which is pretty good given the orginal total number of animes in the reddit is 537.

### Regression With Linear Model

After merging the tables, by applying `explode()` and `pivot()`, we can obtain a tag matrix, where columns are the presented tags and index are the anime names.


```python
tag_matrix = tags.pivot('anime', columns='tags', values='present').fillna(0)
tag_matrix = tag_matrix.sort_index(axis=1, key=lambda x: tag_matrix[x].sum(), ascending=False)
tag_matrix.head()
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
      <th>tags</th>
      <th>comedy</th>
      <th>action</th>
      <th>drama</th>
      <th>fantasy</th>
      <th>slice of life</th>
      <th>present</th>
      <th>based on a manga</th>
      <th>male protagonist</th>
      <th>female protagonist</th>
      <th>shounen</th>
      <th>...</th>
      <th>shounen ai</th>
      <th>soccer</th>
      <th>shounen-ai</th>
      <th>flat chested</th>
      <th>shrine maiden</th>
      <th>flash animation</th>
      <th>skateboarding</th>
      <th>slow when it comes to love</th>
      <th>fake romance</th>
      <th>exhibitionism</th>
    </tr>
    <tr>
      <th>anime</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>100-man no Inochi no Ue ni Ore wa Tatte Iru</th>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2.43: Seiin Koukou Danshi Volley-bu</th>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>22/7</th>
      <td>1.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>3D Kanojo: Real Girl Season 2</th>
      <td>1.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>A3! Season Autumn &amp; Winter</th>
      <td>1.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 717 columns</p>
</div>



In this case, we would explore with the Linear and Logistic Regressions in the linear model. We can split data into three parts - 70% as training data, 15% as test data and 15 % as validation data.


```python
from sklearn.linear_model import LinearRegression, LogisticRegression
from sklearn.metrics import mean_squared_error
from sklearn.model_selection import train_test_split

def rating_test(start, end, useLinear=True, step=1):
    test_mses = []
    vali_mses = []
    if useLinear:
        Model = LinearRegression
    else:
        Model = LogisticRegression
    for i in range(start, end, step):
        reg = Model().fit(train_set.iloc[:, 0:i], train_y)
        test_mses.append(mean_squared_error(test_y, reg.predict(test_set.iloc[:, 0:i])))
        vali_mses.append(mean_squared_error(vali_y, reg.predict(vali_set.iloc[:, 0:i])))
    test_mses = pd.Series(test_mses, index=[i for i in range(start, end, step)], name='Training MSE').rename_axis('n_features')
    vali_mses = pd.Series(vali_mses, index=[i for i in range(start, end, step)], name='Validation MSE').rename_axis('n_features')
    return test_mses, vali_mses

test_mses, vali_mses = rating_test(1, 30)
test_mses.plot.line(grid=True, marker='o', legend=True)
vali_mses.plot.line(grid=True, marker='^', legend=True)
```




    <AxesSubplot:xlabel='n_features'>




    
![png](output_39_1.png)
    


From the plot of MSE between predicted y and actual y is over $10^6$, so apparently, Linear Regression won't work well with this data set.

Now, let's test with Logistic Regression


```python
test_mses, vali_mses = rating_test(1, 30, False)
test_mses.plot.line(grid=True, marker='o', legend=True)
vali_mses.plot.line(grid=True, marker='^', legend=True)
```




    <AxesSubplot:xlabel='n_features'>




    
![png](output_41_1.png)
    


Similarly, Logistic Model doesn't work well on this dataset.

Therefore, we cannot use the simple classifiers to predict scores of an anime.

## Conclusion

With given reddit dataset, it's possible for us to determine what are the hottest animes over different period of time and find out how to rank the top n animes for simple recommendation system. But there are still some limitations using this dataset, such as detect what would be the worst anime, since the score mostly reflects how people are happy to discuss this submission but we are unable to tell what's the attitude of people towards to a TV series. For example, if the quality of an episode is extremely, users might go to the submission and leave comments like analyzing why it's bad and compilations about the plots which may still receive a very good score as a result. It might be a good idea to combine with sentiment score of the texts in the thread to detect if people are complaining, praising or just don't care about the episode.

I also attempted to use the linear model to find potential connection between the tags of animes and mean score of anime episode's discussion submission. However, it turns out these two models didn't work well given data, thus it may require some more powerful models to test if we can find some connections.
