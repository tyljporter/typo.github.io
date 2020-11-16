---
layout: archive
permalink: /SpotifyP1/
title: "Spotify Data Collection Script"
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
```


```python
#Get token and authorize. Will prompt user to redirect URI on first run
username = 'REDACTED'
client_id ='REDACTED'
client_secret = 'REDACTED'
redirect_uri = 'https://google.com/'
scope = 'user-library-read'

token = util.prompt_for_user_token(username=username,
                                   scope=scope,
                                   client_id=client_id,   
                                   client_secret=client_secret,     
                                   redirect_uri=redirect_uri)
sp = spotipy.Spotify(auth=token)
```


```python
#Define function to pull saved tracks
def get_faves():
    results = sp.current_user_saved_tracks()
    tracks = results['items']
    while results['next']:
        results = sp.next(results)
        tracks.extend(results['items'])
    return tracks
```


```python
#Pull saved tracks, then demonstrate output.
#Nested JSON that is hard to follow.
faves = get_faves()
faves[1]
```




    {'added_at': '2020-11-12T17:17:48Z',
     'track': {'album': {'album_type': 'album',
       'artists': [{'external_urls': {'spotify': 'https://open.spotify.com/artist/6guC9FqvlVboSKTI77NG2k'},
         'href': 'https://api.spotify.com/v1/artists/6guC9FqvlVboSKTI77NG2k',
         'id': '6guC9FqvlVboSKTI77NG2k',
         'name': 'Dance Gavin Dance',
         'type': 'artist',
         'uri': 'spotify:artist:6guC9FqvlVboSKTI77NG2k'}],
       'available_markets': ['AD',
        'AE',
        'AL',
        'AR',
        'AT',
        'AU',
        'BA',
        'BE',
        'BG',
        'BH',
        'BO',
        'BR',
        'BY',
        'CA',
        'CH',
        'CL',
        'CO',
        'CR',
        'CY',
        'CZ',
        'DE',
        'DK',
        'DO',
        'DZ',
        'EC',
        'EE',
        'EG',
        'ES',
        'FI',
        'FR',
        'GB',
        'GR',
        'GT',
        'HK',
        'HN',
        'HR',
        'HU',
        'ID',
        'IE',
        'IL',
        'IS',
        'IT',
        'JO',
        'JP',
        'KW',
        'KZ',
        'LB',
        'LI',
        'LT',
        'LU',
        'LV',
        'MA',
        'MC',
        'MD',
        'ME',
        'MK',
        'MT',
        'MX',
        'MY',
        'NI',
        'NL',
        'NO',
        'NZ',
        'OM',
        'PA',
        'PE',
        'PH',
        'PL',
        'PS',
        'PT',
        'PY',
        'QA',
        'RO',
        'RS',
        'RU',
        'SA',
        'SE',
        'SG',
        'SI',
        'SK',
        'SV',
        'TH',
        'TN',
        'TR',
        'TW',
        'UA',
        'US',
        'UY',
        'VN',
        'XK',
        'ZA'],
       'external_urls': {'spotify': 'https://open.spotify.com/album/75Z98z7kl42oWcT0UvtO6e'},
       'href': 'https://api.spotify.com/v1/albums/75Z98z7kl42oWcT0UvtO6e',
       'id': '75Z98z7kl42oWcT0UvtO6e',
       'images': [{'height': 640,
         'url': 'https://i.scdn.co/image/ab67616d0000b273fc52f6fc7dca9af16569fb2b',
         'width': 640},
        {'height': 300,
         'url': 'https://i.scdn.co/image/ab67616d00001e02fc52f6fc7dca9af16569fb2b',
         'width': 300},
        {'height': 64,
         'url': 'https://i.scdn.co/image/ab67616d00004851fc52f6fc7dca9af16569fb2b',
         'width': 64}],
       'name': 'Instant Gratification',
       'release_date': '2015-04-10',
       'release_date_precision': 'day',
       'total_tracks': 12,
       'type': 'album',
       'uri': 'spotify:album:75Z98z7kl42oWcT0UvtO6e'},
      'artists': [{'external_urls': {'spotify': 'https://open.spotify.com/artist/6guC9FqvlVboSKTI77NG2k'},
        'href': 'https://api.spotify.com/v1/artists/6guC9FqvlVboSKTI77NG2k',
        'id': '6guC9FqvlVboSKTI77NG2k',
        'name': 'Dance Gavin Dance',
        'type': 'artist',
        'uri': 'spotify:artist:6guC9FqvlVboSKTI77NG2k'}],
      'available_markets': ['AD',
       'AE',
       'AL',
       'AR',
       'AT',
       'AU',
       'BA',
       'BE',
       'BG',
       'BH',
       'BO',
       'BR',
       'BY',
       'CA',
       'CH',
       'CL',
       'CO',
       'CR',
       'CY',
       'CZ',
       'DE',
       'DK',
       'DO',
       'DZ',
       'EC',
       'EE',
       'EG',
       'ES',
       'FI',
       'FR',
       'GB',
       'GR',
       'GT',
       'HK',
       'HN',
       'HR',
       'HU',
       'ID',
       'IE',
       'IL',
       'IS',
       'IT',
       'JO',
       'JP',
       'KW',
       'KZ',
       'LB',
       'LI',
       'LT',
       'LU',
       'LV',
       'MA',
       'MC',
       'MD',
       'ME',
       'MK',
       'MT',
       'MX',
       'MY',
       'NI',
       'NL',
       'NO',
       'NZ',
       'OM',
       'PA',
       'PE',
       'PH',
       'PL',
       'PS',
       'PT',
       'PY',
       'QA',
       'RO',
       'RS',
       'RU',
       'SA',
       'SE',
       'SG',
       'SI',
       'SK',
       'SV',
       'TH',
       'TN',
       'TR',
       'TW',
       'UA',
       'US',
       'UY',
       'VN',
       'XK',
       'ZA'],
      'disc_number': 1,
      'duration_ms': 250973,
      'explicit': False,
      'external_ids': {'isrc': 'USEK71527710'},
      'external_urls': {'spotify': 'https://open.spotify.com/track/0ZcQUwVyXgpUepJsvgOYgk'},
      'href': 'https://api.spotify.com/v1/tracks/0ZcQUwVyXgpUepJsvgOYgk',
      'id': '0ZcQUwVyXgpUepJsvgOYgk',
      'is_local': False,
      'name': 'Death Of A Strawberry',
      'popularity': 58,
      'preview_url': 'https://p.scdn.co/mp3-preview/a3bab2c85a055518ed1ab8d2229dcac175e24329?cid=aec85b8edcba466ea57b3fffb1bf1aac',
      'track_number': 10,
      'type': 'track',
      'uri': 'spotify:track:0ZcQUwVyXgpUepJsvgOYgk'}}




```python
#Can we see this a little bit better? What attributes at our disposal?
for col in pd.json_normalize(faves).columns:
    print(col)
```

    added_at
    track.album.album_type
    track.album.artists
    track.album.available_markets
    track.album.external_urls.spotify
    track.album.href
    track.album.id
    track.album.images
    track.album.name
    track.album.release_date
    track.album.release_date_precision
    track.album.total_tracks
    track.album.type
    track.album.uri
    track.artists
    track.available_markets
    track.disc_number
    track.duration_ms
    track.explicit
    track.external_ids.isrc
    track.external_urls.spotify
    track.href
    track.id
    track.is_local
    track.name
    track.popularity
    track.preview_url
    track.track_number
    track.type
    track.uri



```python
#Define function that pulls metadata and audio features for passed JSON of tracks
def analyze(tracks):
    #Initialize features to be added
    features_list = ['artist','album','track_name','track_id',
                     'danceability','energy','key','loudness',
                     'mode', 'speechiness','instrumentalness',
                     'liveness','valence','tempo', 'duration_ms',
                     'time_signature']
    df = pd.DataFrame(columns = features_list)
    # Loop through every track in the playlist, pull metadata and audio features using spotipy and append to df
    for track in tracks:
        # Create empty dict
        features = {}
        # Get metadata
        features['artist'] = track['track']['album']['artists'][0]['name']
        features['album'] = track['track']['album']['name']
        features['track_name'] = track['track']['name']
        features['track_id'] = track['track']['id']

        # Get audio features
        audio_features = sp.audio_features(features['track_id'])[0]
        for feature in features_list[4:]:
            features[feature] = audio_features[feature]

        # Concat the dfs
        track_df = pd.DataFrame(features, index = [0])
        df = pd.concat([df, track_df], ignore_index = True)
    return df

```


```python
#analyze tracks JSON
tracks = analyze(faves)
#Set target variable of saved songs to '1'
tracks['saved'] = '1'
```


```python
#Now have a dataframe of 'saved' tracks, with 1 in target variable to identify
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
      <td>0.2090</td>
      <td>75.019</td>
      <td>230142</td>
      <td>4</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
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
      <td>0.7520</td>
      <td>124.972</td>
      <td>250973</td>
      <td>4</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
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
      <td>0.4850</td>
      <td>78.318</td>
      <td>205707</td>
      <td>4</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>I Prevail</td>
      <td>TRAUMA</td>
      <td>Breaking Down</td>
      <td>4bE3cBcwAe7T5SklvbkiYZ</td>
      <td>0.503</td>
      <td>0.568</td>
      <td>4</td>
      <td>-7.182</td>
      <td>0</td>
      <td>0.0341</td>
      <td>0.001440</td>
      <td>0.0856</td>
      <td>0.0705</td>
      <td>126.935</td>
      <td>206147</td>
      <td>4</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>I Prevail</td>
      <td>TRAUMA</td>
      <td>Every Time You Leave</td>
      <td>5icbZiF6lcuEORG0UzMsS2</td>
      <td>0.473</td>
      <td>0.749</td>
      <td>1</td>
      <td>-5.047</td>
      <td>0</td>
      <td>0.0287</td>
      <td>0.000005</td>
      <td>0.0883</td>
      <td>0.1670</td>
      <td>150.022</td>
      <td>216627</td>
      <td>4</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Define function to pull tracks from a playlist
def get_playlist_tracks(ID):
    sample = sp.user_playlist_tracks(username, ID)
    tracks = sample['items']
    while sample['next']:
        sample = sp.next(sample)
        tracks.extend(sample['items'])
    return tracks
```


```python
#URI of larger playlist
ID = 'spotify:playlist:3dgfLqrR0v70pKNb961q6q'
#Pull tracks from larger playlist
playlist = get_playlist_tracks(ID)
```


```python
#Analyze larger playlist tracks
playlist = analyze(playlist)
```


```python
#Set target variable of suggested songs to '0'
playlist['saved'] = '0'
```


```python
#Append larger sample to other df
tracks = tracks.append(playlist, ignore_index = True)
```


```python
#See length that includes duplicates
len(tracks)
```




    276




```python
#Now have list of faves, sample of larger set with some faves in it. Need to remove duplicates,
#Leaving the first instance
tracks = tracks.drop_duplicates(subset = 'track_id', keep = 'first')
```


```python
#See final length with duplicated removed
len(tracks)
```




    232




```python
tracks
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
      <td>0.2090</td>
      <td>75.019</td>
      <td>230142</td>
      <td>4</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
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
      <td>0.7520</td>
      <td>124.972</td>
      <td>250973</td>
      <td>4</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
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
      <td>0.4850</td>
      <td>78.318</td>
      <td>205707</td>
      <td>4</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>I Prevail</td>
      <td>TRAUMA</td>
      <td>Breaking Down</td>
      <td>4bE3cBcwAe7T5SklvbkiYZ</td>
      <td>0.503</td>
      <td>0.568</td>
      <td>4</td>
      <td>-7.182</td>
      <td>0</td>
      <td>0.0341</td>
      <td>0.001440</td>
      <td>0.0856</td>
      <td>0.0705</td>
      <td>126.935</td>
      <td>206147</td>
      <td>4</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>I Prevail</td>
      <td>TRAUMA</td>
      <td>Every Time You Leave</td>
      <td>5icbZiF6lcuEORG0UzMsS2</td>
      <td>0.473</td>
      <td>0.749</td>
      <td>1</td>
      <td>-5.047</td>
      <td>0</td>
      <td>0.0287</td>
      <td>0.000005</td>
      <td>0.0883</td>
      <td>0.1670</td>
      <td>150.022</td>
      <td>216627</td>
      <td>4</td>
      <td>1</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>267</th>
      <td>Lydia</td>
      <td>Liquor</td>
      <td>Let It Cover Me Up</td>
      <td>2wEYnSmSU44HfluSkHQUJ5</td>
      <td>0.492</td>
      <td>0.601</td>
      <td>2</td>
      <td>-8.844</td>
      <td>0</td>
      <td>0.0431</td>
      <td>0.000000</td>
      <td>0.0650</td>
      <td>0.3620</td>
      <td>110.017</td>
      <td>221827</td>
      <td>4</td>
      <td>0</td>
    </tr>
    <tr>
      <th>269</th>
      <td>Dance Gavin Dance</td>
      <td>Downtown Battle Mountain II</td>
      <td>Spooks</td>
      <td>1Pxw1ZA2WQgQQHsHusR6HD</td>
      <td>0.166</td>
      <td>0.988</td>
      <td>9</td>
      <td>-2.988</td>
      <td>0</td>
      <td>0.2300</td>
      <td>0.000353</td>
      <td>0.1670</td>
      <td>0.4870</td>
      <td>183.498</td>
      <td>243733</td>
      <td>4</td>
      <td>0</td>
    </tr>
    <tr>
      <th>271</th>
      <td>Bring Me The Horizon</td>
      <td>Sempiternal (Deluxe Edition)</td>
      <td>Sleepwalking</td>
      <td>0ShUX5LYFHltP8T7PxRhRm</td>
      <td>0.399</td>
      <td>0.963</td>
      <td>2</td>
      <td>-3.280</td>
      <td>0</td>
      <td>0.1220</td>
      <td>0.000005</td>
      <td>0.2790</td>
      <td>0.2940</td>
      <td>148.892</td>
      <td>230091</td>
      <td>4</td>
      <td>0</td>
    </tr>
    <tr>
      <th>272</th>
      <td>Breathe Atlantis</td>
      <td>Futurestories</td>
      <td>Lost</td>
      <td>1ZtSeG7ldSZcF3IO4qDT4I</td>
      <td>0.492</td>
      <td>0.949</td>
      <td>0</td>
      <td>-3.478</td>
      <td>0</td>
      <td>0.0596</td>
      <td>0.000000</td>
      <td>0.1540</td>
      <td>0.3510</td>
      <td>99.968</td>
      <td>212453</td>
      <td>4</td>
      <td>0</td>
    </tr>
    <tr>
      <th>275</th>
      <td>Noija</td>
      <td>Through Fire All Things Are Renewed</td>
      <td>Broken Glass</td>
      <td>03XmVDeR1iflqj90vh7THk</td>
      <td>0.517</td>
      <td>0.828</td>
      <td>8</td>
      <td>-6.644</td>
      <td>0</td>
      <td>0.0439</td>
      <td>0.000000</td>
      <td>0.0484</td>
      <td>0.5370</td>
      <td>160.005</td>
      <td>238500</td>
      <td>4</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>232 rows × 17 columns</p>
</div>




```python
tracks.to_csv('tracks.csv', index = False)
```
