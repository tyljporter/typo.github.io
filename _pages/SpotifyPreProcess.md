---
layout: archive
permalink: /SpotifyP2/
title: "Spotify Data Preprocessing"
author_profile: true
---

```python
import spotipy
import spotipy.util as util
import pandas as pd
import sys
import math
import matplotlib.pyplot as plt
import numpy as np
from sklearn import preprocessing
import seaborn as sns
import scipy.stats as ss
import os
WD = os.getcwd()
```


```python
tracks = pd.read_csv('tracks.csv')
```


```python
tracks.head()
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
      <th>artist</th>
      <th>album</th>
      <th>track_name</th>
      <th>track_id</th>
      <th>danceability</th>
      <th>energy</th>
      <th>key</th>
      <th>loudness</th>
      <th>mode</th>
      <th>speechiness</th>
      <th>instrumentalness</th>
      <th>liveness</th>
      <th>valence</th>
      <th>tempo</th>
      <th>duration_ms</th>
      <th>time_signature</th>
      <th>saved</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Four Year Strong</td>
      <td>Talking Myself in Circles / Brain Pain</td>
      <td>Brain Pain</td>
      <td>2M13FTaarfjaViPg1kE4Mj</td>
      <td>0.261</td>
      <td>0.934</td>
      <td>2</td>
      <td>-3.182</td>
      <td>1</td>
      <td>0.1270</td>
      <td>0.000224</td>
      <td>0.0607</td>
      <td>0.216</td>
      <td>147.489</td>
      <td>226572</td>
      <td>4</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Too Close To Touch</td>
      <td>Haven't Been Myself</td>
      <td>Sympathy</td>
      <td>1s4Nc5XFspnFHEHLv92gak</td>
      <td>0.382</td>
      <td>0.799</td>
      <td>6</td>
      <td>-3.676</td>
      <td>0</td>
      <td>0.0607</td>
      <td>0.000045</td>
      <td>0.1380</td>
      <td>0.244</td>
      <td>140.059</td>
      <td>216120</td>
      <td>4</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>The Devil Wears Prada</td>
      <td>The Act</td>
      <td>Chemical</td>
      <td>4mGHzFvAiRCSOPIpjLvf80</td>
      <td>0.527</td>
      <td>0.789</td>
      <td>5</td>
      <td>-7.344</td>
      <td>0</td>
      <td>0.0331</td>
      <td>0.017200</td>
      <td>0.0850</td>
      <td>0.209</td>
      <td>75.019</td>
      <td>230142</td>
      <td>4</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Dance Gavin Dance</td>
      <td>Instant Gratification</td>
      <td>Death Of A Strawberry</td>
      <td>0ZcQUwVyXgpUepJsvgOYgk</td>
      <td>0.576</td>
      <td>0.955</td>
      <td>4</td>
      <td>-3.122</td>
      <td>1</td>
      <td>0.0458</td>
      <td>0.000000</td>
      <td>0.0540</td>
      <td>0.752</td>
      <td>124.972</td>
      <td>250973</td>
      <td>4</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Dance Gavin Dance</td>
      <td>Instant Gratification</td>
      <td>We Own The Night</td>
      <td>4rc2WNOt2w8BHfQjBC8PO0</td>
      <td>0.224</td>
      <td>0.905</td>
      <td>5</td>
      <td>-3.239</td>
      <td>0</td>
      <td>0.0722</td>
      <td>0.000000</td>
      <td>0.2810</td>
      <td>0.485</td>
      <td>78.318</td>
      <td>205707</td>
      <td>4</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>




```python
features_list = tracks.columns.values.tolist()
features_list
```




    ['artist',
     'album',
     'track_name',
     'track_id',
     'danceability',
     'energy',
     'key',
     'loudness',
     'mode',
     'speechiness',
     'instrumentalness',
     'liveness',
     'valence',
     'tempo',
     'duration_ms',
     'time_signature',
     'saved']




```python
numeric_cols = ['danceability','energy', 'key', 'loudness', 'mode',
                 'speechiness', 'liveness', 'instrumentalness',
                 'valence', 'tempo', 'duration_ms']
```

## Standardization
Utilizing function found [here](https://datamadness.github.io/Skewness_Auto_Transform) and modified


```python
##################################################
# A function that will accept a pandas dataframe
# and auto-transforms columns that exceeds threshold value
#  -  Offers choice between boxcox or log / exponential transformation
#  -  Automatically handles negative values
#  -  Auto recognizes positive /negative skewness
# Further documentation available here:
# https://datamadness.github.io/Skewness_Auto_Transform

def skew_autotransform(DF, include = None, exclude = None, plot = False, threshold = 1, exp = False):
    trans = DF
    #Get list of column names that should be processed based on input parameters
    if include is None and exclude is None:
        colnames = trans.columns.values
    elif include is not None:
        colnames = include
    elif exclude is not None:
        colnames = [item for item in list(trans.columns.values) if item not in exclude]
    else:
        print('No columns to process!')

    #Helper function that checks if all values are positive
    def make_positive(series):
        minimum = np.amin(series)
        #If minimum is negative, offset all values by a constant to move all values to positive teritory
        if minimum <= 0:
            series = series + abs(minimum) + 0.01
        return series

    #Go through desired columns in DataFrame
    for col in colnames:
        #Get column skewness
        skew = trans[col].skew()
        transformed = True
        if plot:
            #Prep the plot of original data
            sns.set_style("darkgrid")
            sns.set_palette("Blues_r")
            fig, axes = plt.subplots(1, 2, figsize=(10, 5))
            ax1 = sns.distplot(trans[col], ax=axes[0])
            ax1.set(xlabel='Original ' + col)

        #If skewness is larger than threshold and positively skewed; If yes, apply appropriate transformation
        if abs(skew) > threshold and skew > 0:
            skewType = 'positive'
            #Make sure all values are positive
            trans[col] = make_positive(trans[col])

            if exp:
               #Apply log transformation
               trans[col] = trans[col].apply(math.log)
            else:
                #Apply boxcox transformation
                trans[col] = ss.boxcox(trans[col])[0]
            skew_new = trans[col].skew()

        elif abs(skew) > threshold and skew < 0:
            skewType = 'negative'
            #Make sure all values are positive
            trans[col] = make_positive(trans[col])

            if exp:
               #Apply exp transformation
               trans[col] = trans[col].pow(10)
            else:
                #Apply boxcox transformation
                trans[col] = ss.boxcox(trans[col])[0]
            skew_new = trans[col].skew()

        else:
            #Flag if no transformation was performed
            transformed = False
            skew_new = skew

        #Compare before and after if plot is True
        if plot:
            print('\n ------------------------------------------------------')     
            if transformed:
                print('\n %r had %r skewness of %2.2f' %(col, skewType, skew))
                print('\n Transformation yielded skewness of %2.2f' %(skew_new))
                sns.set_palette("Paired")
                ax2 = sns.distplot(trans[col], ax=axes[1], color = 'r')
                ax2.set(xlabel='Transformed ' + col)
                plt.savefig(WD + '/SpotTrans/' + col +'.png')
                plt.show()
            else:
                print('\n NO TRANSFORMATION APPLIED FOR %r . Skewness = %2.2f' %(col, skew))
                ax2 = sns.distplot(trans[col], ax=axes[1])
                ax2.set(xlabel='NO TRANSFORM ' + col)
                plt.savefig(WD + '/SpotTrans/' + col +'.png')
                plt.show()


    return trans
```


```python
#Standardize into new df, keeping the OG df
standtracks = skew_autotransform(tracks.copy(deep = True), include = numeric_cols, plot = True, threshold = 1)
```


     ------------------------------------------------------

     NO TRANSFORMATION APPLIED FOR 'danceability' . Skewness = -0.45



![png](/images/SpotTrans/output_7_1.png)



     ------------------------------------------------------

     'energy' had 'negative' skewness of -1.88

     Transformation yielded skewness of -0.48



![png](/images/SpotTrans/output_7_3.png)



     ------------------------------------------------------

     NO TRANSFORMATION APPLIED FOR 'key' . Skewness = 0.18



![png](/images/SpotTrans/output_7_5.png)



     ------------------------------------------------------

     'loudness' had 'negative' skewness of -1.17

     Transformation yielded skewness of -0.33



![png](/images/SpotTrans/output_7_7.png)



     ------------------------------------------------------

     NO TRANSFORMATION APPLIED FOR 'mode' . Skewness = 0.00



![png](/images/SpotTrans/output_7_9.png)



     ------------------------------------------------------

     'speechiness' had 'positive' skewness of 2.00

     Transformation yielded skewness of 0.06



![png](/images/SpotTrans/output_7_11.png)



     ------------------------------------------------------

     'liveness' had 'positive' skewness of 1.86

     Transformation yielded skewness of 0.04



![png](/images/SpotTrans/output_7_13.png)



     ------------------------------------------------------

     'instrumentalness' had 'positive' skewness of 4.75

     Transformation yielded skewness of 1.52



![png](/images/SpotTrans/output_7_15.png)



     ------------------------------------------------------

     NO TRANSFORMATION APPLIED FOR 'valence' . Skewness = 0.34



![png](/images/SpotTrans/output_7_17.png)



     ------------------------------------------------------

     NO TRANSFORMATION APPLIED FOR 'tempo' . Skewness = -0.02



![png](/images/SpotTrans/output_7_19.png)



     ------------------------------------------------------

     NO TRANSFORMATION APPLIED FOR 'duration_ms' . Skewness = 0.61



![png](/images/SpotTrans/output_7_21.png)



```python
standtracks.head(5)
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
      <th>artist</th>
      <th>album</th>
      <th>track_name</th>
      <th>track_id</th>
      <th>danceability</th>
      <th>energy</th>
      <th>key</th>
      <th>loudness</th>
      <th>mode</th>
      <th>speechiness</th>
      <th>instrumentalness</th>
      <th>liveness</th>
      <th>valence</th>
      <th>tempo</th>
      <th>duration_ms</th>
      <th>time_signature</th>
      <th>saved</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Four Year Strong</td>
      <td>Talking Myself in Circles / Brain Pain</td>
      <td>Brain Pain</td>
      <td>2M13FTaarfjaViPg1kE4Mj</td>
      <td>0.261</td>
      <td>-0.056835</td>
      <td>2</td>
      <td>38.712045</td>
      <td>1</td>
      <td>-3.107250</td>
      <td>-11264.537406</td>
      <td>-4.118748</td>
      <td>0.216</td>
      <td>147.489</td>
      <td>226572</td>
      <td>4</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Too Close To Touch</td>
      <td>Haven't Been Myself</td>
      <td>Sympathy</td>
      <td>1s4Nc5XFspnFHEHLv92gak</td>
      <td>0.382</td>
      <td>-0.128339</td>
      <td>6</td>
      <td>34.998334</td>
      <td>0</td>
      <td>-4.941836</td>
      <td>-11711.629924</td>
      <td>-2.588755</td>
      <td>0.244</td>
      <td>140.059</td>
      <td>216120</td>
      <td>4</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>The Devil Wears Prada</td>
      <td>The Act</td>
      <td>Chemical</td>
      <td>4mGHzFvAiRCSOPIpjLvf80</td>
      <td>0.527</td>
      <td>-0.131842</td>
      <td>5</td>
      <td>12.976508</td>
      <td>0</td>
      <td>-6.876214</td>
      <td>-1297.368972</td>
      <td>-3.451869</td>
      <td>0.209</td>
      <td>75.019</td>
      <td>230142</td>
      <td>4</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Dance Gavin Dance</td>
      <td>Instant Gratification</td>
      <td>Death Of A Strawberry</td>
      <td>0ZcQUwVyXgpUepJsvgOYgk</td>
      <td>0.576</td>
      <td>-0.040633</td>
      <td>4</td>
      <td>39.174990</td>
      <td>1</td>
      <td>-5.786099</td>
      <td>-11829.386694</td>
      <td>-4.364367</td>
      <td>0.752</td>
      <td>124.972</td>
      <td>250973</td>
      <td>4</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Dance Gavin Dance</td>
      <td>Instant Gratification</td>
      <td>We Own The Night</td>
      <td>4rc2WNOt2w8BHfQjBC8PO0</td>
      <td>0.224</td>
      <td>-0.076647</td>
      <td>5</td>
      <td>38.274628</td>
      <td>0</td>
      <td>-4.464237</td>
      <td>-11829.386694</td>
      <td>-1.503340</td>
      <td>0.485</td>
      <td>78.318</td>
      <td>205707</td>
      <td>4</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>



## Normalization


```python
#View preliminary box and whiskers
sns.set_palette('plasma')
sns.set_context('talk', font_scale = 0.75)
fig = tracks[features_list].plot(kind = 'box', figsize = (20,10))
plt.savefig('Starting.png')
```


![png](/images/SpotTrans/output_10_0.png)



```python
#View standardized box and whiskers
sns.set_palette('plasma')
sns.set_context('talk', font_scale = 0.75)
fig = standtracks[features_list].plot(kind = 'box', figsize = (20,10))
plt.savefig('Standardized.png')
```


![png](/images/SpotTrans/output_11_0.png)



```python
#Define function that normalizes if the column contains non-string values or is the target column, saved.
def normalize(df):
    result = df.copy()
    for col in df:
        if type(df[col][1]) != str and col != 'saved':
            max_val = df[col].max()
            min_val = df[col].min()
            result[col] = (df[col] - min_val) / (max_val - min_val)
            print(col, ' -- ', '*applied*')
        else:
            print(col, ' -- ', 'not applied')
            pass
    return result
```


```python
#Run normalization on standardized data
proctracks = normalize(standtracks)
```

    artist  --  not applied
    album  --  not applied
    track_name  --  not applied
    track_id  --  not applied
    danceability  --  *applied*
    energy  --  *applied*
    key  --  *applied*
    loudness  --  *applied*
    mode  --  *applied*
    speechiness  --  *applied*
    instrumentalness  --  *applied*
    liveness  --  *applied*
    valence  --  *applied*
    tempo  --  *applied*
    duration_ms  --  *applied*
    time_signature  --  *applied*
    saved  --  not applied



```python
#Visualize box and whiskers for normalized and standardized dataset
sns.set_palette('plasma')
sns.set_context('talk', font_scale = 0.75)
fig = proctracks[features_list].plot(kind = 'box', figsize = (20,10))
plt.savefig('processed.png')
```


![png](https://tyljporter.github.io/images/SpotTrans/output_14_0.png)



```python
#export CSV
proctracks.to_csv('processedtracks.csv', index = False)
```


```python

```
