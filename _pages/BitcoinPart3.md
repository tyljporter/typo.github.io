# Text Analytics | BAIS:6100
# Team Project Part 3
### Cluster & Topic Analysis


Deliverables:
1. CSV file
2. Jupyter Notebook with code and output

Instructor: Kang-Pyo Lee 

Team: CryptoPredictors

Team Members:
- Tyler Porter
- Praveen Gadkari
- Troy Parker


```python
#packages and other imports

#data frames
import pandas as pd
pd.set_option('display.max_colwidth', 150)
from pandas import DataFrame

#file directory
import os
   
#text analytic packages
import nltk
import re
from nltk.util import ngrams
from textblob import TextBlob

# importing random module 
import random
```


```python
#read in csv file containing articles
df = pd.read_csv("CSV_File/html_metadata1.csv", sep="\t")
```


```python
df.head()
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
      <th>file_name</th>
      <th>article_title</th>
      <th>article_date_time</th>
      <th>article_author</th>
      <th>article_text</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>domestic-and-foreign-buyers-acquired-a-tokenized-apartment-in-a-spanish-city-by-paying-with-ethereum.html</td>
      <td>Domestic and Foreign Buyers Acquired a 'Tokenized' Apartment in a Spanish City by Paying With Ethereum</td>
      <td>53 mins ago</td>
      <td>Felipe Erazo</td>
      <td>Domestic and Foreign Buyers Acquired a 'Tokenized' Apartment in a Spanish City by Paying With Ethereum                        National and foreign...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>zeitgeist-brings-a-prediction-markets-ecosystem-to-polkadots-kusama.html</td>
      <td>Zeitgeist Brings a Prediction Markets Ecosystem to Polkadot’s Kusama</td>
      <td>3 hours ago</td>
      <td>Reuben Jackson</td>
      <td>Zeitgeist Brings a Prediction Markets Ecosystem to Polkadot’s Kusama                        To capitalize on the growing popularity of blockchain ...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>taraxa-listing-on-asecendex.html</td>
      <td>Taraxa Listing on AsecendEX</td>
      <td>4 hours ago</td>
      <td>NaN</td>
      <td>Taraxa Listing on AsecendEX                press release         PRESS RELEASE. AscendEX, formerly BitMax, an industry-leading digital asset tradi...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>fio-protocol-marks-first-year-anniversary.html</td>
      <td>FIO Protocol Marks First Year Anniversary</td>
      <td>12 hours ago</td>
      <td>NaN</td>
      <td>FIO Protocol Marks First Year Anniversary                press release         PRESS RELEASE. The Foundation for Interwallet Operability (FIO) cel...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>alchemy-chooses-flow-blockchain-to-accelerate-game-changing-developer-ecosystem.html</td>
      <td>Alchemy Chooses Flow Blockchain to Accelerate Game-Changing Developer Ecosystem</td>
      <td>20 hours ago</td>
      <td>NaN</td>
      <td>Alchemy Chooses Flow Blockchain to Accelerate Game-Changing Developer Ecosystem                press release         PRESS RELEASE. The leading bl...</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Change the article dates where they are non standard format - Troy 
#Code is designed to handle file scraped on same day as acquired
import datetime

today = datetime.datetime.today()
today = today.strftime("%B %d, %Y")

#the website maxes at 7 days with nonstandard format... use 10 for extra insurance
num = 10

#initialize date list
date_list = []

#create list of dates older than today
for i in range(num):
     tmp_date = datetime.datetime.today() - datetime.timedelta(days=i+1)
     date_list.append(tmp_date.strftime("%B %d, %Y"))

#change current day dates
df.article_date_time[df['article_date_time'].str.contains("hour")] = today
df.article_date_time[df['article_date_time'].str.contains("sec")] = today
df.article_date_time[df['article_date_time'].str.contains("min")] = today

#change articles older than read date
for i in range(num):
    test = str(i) + ' day'
    df.article_date_time[df['article_date_time'].str.contains(test)] = date_list[i]
```


```python
df.head()
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
      <th>file_name</th>
      <th>article_title</th>
      <th>article_date_time</th>
      <th>article_author</th>
      <th>article_text</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>domestic-and-foreign-buyers-acquired-a-tokenized-apartment-in-a-spanish-city-by-paying-with-ethereum.html</td>
      <td>Domestic and Foreign Buyers Acquired a 'Tokenized' Apartment in a Spanish City by Paying With Ethereum</td>
      <td>April 08, 2021</td>
      <td>Felipe Erazo</td>
      <td>Domestic and Foreign Buyers Acquired a 'Tokenized' Apartment in a Spanish City by Paying With Ethereum                        National and foreign...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>zeitgeist-brings-a-prediction-markets-ecosystem-to-polkadots-kusama.html</td>
      <td>Zeitgeist Brings a Prediction Markets Ecosystem to Polkadot’s Kusama</td>
      <td>April 08, 2021</td>
      <td>Reuben Jackson</td>
      <td>Zeitgeist Brings a Prediction Markets Ecosystem to Polkadot’s Kusama                        To capitalize on the growing popularity of blockchain ...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>taraxa-listing-on-asecendex.html</td>
      <td>Taraxa Listing on AsecendEX</td>
      <td>April 08, 2021</td>
      <td>NaN</td>
      <td>Taraxa Listing on AsecendEX                press release         PRESS RELEASE. AscendEX, formerly BitMax, an industry-leading digital asset tradi...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>fio-protocol-marks-first-year-anniversary.html</td>
      <td>FIO Protocol Marks First Year Anniversary</td>
      <td>April 08, 2021</td>
      <td>NaN</td>
      <td>FIO Protocol Marks First Year Anniversary                press release         PRESS RELEASE. The Foundation for Interwallet Operability (FIO) cel...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>alchemy-chooses-flow-blockchain-to-accelerate-game-changing-developer-ecosystem.html</td>
      <td>Alchemy Chooses Flow Blockchain to Accelerate Game-Changing Developer Ecosystem</td>
      <td>April 08, 2021</td>
      <td>NaN</td>
      <td>Alchemy Chooses Flow Blockchain to Accelerate Game-Changing Developer Ecosystem                press release         PRESS RELEASE. The leading bl...</td>
    </tr>
  </tbody>
</table>
</div>




```python
#check for duplicate articles
df.article_title.value_counts()
```




    Cryptocurrencyalerting.com Will Notify You of Price Movements                                          1
    Riot Blockchain Buys 15,000 Antminers, Operation Will Command 37,640 Bitcoin Miners                    1
    Sideshift SLP Token SAI Gains Over 30% in Less Than 24 Hours                                           1
    US Seizes Previously Undetected Silk Road Bitcoins Worth Over $1 Billion                               1
    Bill Miller's Multibillion-Dollar Hedge Fund to Add Bitcoin Exposure via GBTC                          1
                                                                                                          ..
    Coinbase Files Paperwork With SEC to Launch Public Offering                                            1
    Crypto Users Claim Popular Bitcoin Paper Wallet Generator Is Compromised, Millions Allegedly Stolen    1
    How to Buy a Cryptocurrency Hardware Wallet With Bitcoin Cash                                          1
    50 Companies Back New Cryptocurrency Project Competing With Facebook's Libra                           1
    Hayek's 1984: Rediscovered Footage Shows Austrian Economist Predicting Bitcoin                         1
    Name: article_title, Length: 4498, dtype: int64




```python
df.info()  #4,498 articles
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 4498 entries, 0 to 4497
    Data columns (total 5 columns):
     #   Column             Non-Null Count  Dtype 
    ---  ------             --------------  ----- 
     0   file_name          4498 non-null   object
     1   article_title      4498 non-null   object
     2   article_date_time  4498 non-null   object
     3   article_author     4492 non-null   object
     4   article_text       4498 non-null   object
    dtypes: object(5)
    memory usage: 175.8+ KB



```python
from nltk.corpus import stopwords
import string 

global_stopwords = stopwords.words("english")

local_stopwords = [c for c in string.punctuation] +\
                  ['’', '``', '…', '...', "''", '‘', '“', '”', "'m", "'re", "'s", "'ve", 'amp',
                   'https', "n't", 'rt','bitcoin','#bitcoin','#Bitcoin', 'crypto', 'cryptocurrency', 'com', 'new', 'bch','blockchain','blockchain.com',
                   'btc', 'us', 'bitcoin.com', 'million', 'cash', 'billion', '—', 'says', 'Bitcoin', 'Crypto', 'update', '-', '1' ]
```


```python
from sklearn.feature_extraction.text import TfidfVectorizer

vectorizer = TfidfVectorizer(use_idf=True, norm="l2", stop_words=global_stopwords+local_stopwords, max_df=0.7)
X = vectorizer.fit_transform(df.article_title)
```


```python
X.shape
```




    (4498, 7736)



## K-Means Clustering

We tried different values for K from 2 to 6.  K=4 seemed to make the most sense because the others had lopsided number of records with the largest cluster containing most of the data (even more than when K=4). Besides that, the grouping did not make sense with a lot of jumbled up data in those clusters.  We could draw almost no inference from them.


```python
#Step 1: Choose the number of clusters
k = 4
```


```python
#Step 2: Initialize the model object for k-means clustering
from sklearn.cluster import KMeans

kmeans = KMeans(n_clusters=k, random_state=0)
kmeans
```




    KMeans(n_clusters=4, random_state=0)




```python
#Step 3: Fit the model using the input data
%time kmeans.fit(X)
```

    CPU times: user 4.65 s, sys: 36.1 ms, total: 4.69 s
    Wall time: 611 ms





    KMeans(n_clusters=4, random_state=0)




```python
#Step 4: Examine the clustering outcome
df["cluster"] = kmeans.labels_

df[["article_title", "cluster"]]
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
      <th>article_title</th>
      <th>cluster</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Domestic and Foreign Buyers Acquired a 'Tokenized' Apartment in a Spanish City by Paying With Ethereum</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Zeitgeist Brings a Prediction Markets Ecosystem to Polkadot’s Kusama</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Taraxa Listing on AsecendEX</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>FIO Protocol Marks First Year Anniversary</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Alchemy Chooses Flow Blockchain to Accelerate Game-Changing Developer Ecosystem</td>
      <td>0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>4493</th>
      <td>Trading Platform Voltaire Rewards Users With BCH for Every Trade</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4494</th>
      <td>UN Panel: North Korea Hacked $571M From Asian Crypto Exchanges</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4495</th>
      <td>P2P Markets: Russian Localbitcoins Trade Outpaces Venezuela</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4496</th>
      <td>Markets Update: Cryptocurrencies Struggle to Surpass Upper Resistance</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4497</th>
      <td>Universal Protocol Alliance Launches Interest-Paying Stablecoin UPUSD</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
<p>4498 rows × 2 columns</p>
</div>




```python
#Check the number of values for each cluster
df.cluster.value_counts()
```




    0    3507
    1     445
    2     363
    3     183
    Name: cluster, dtype: int64




```python
#sample size
N = 15
```


```python
#check 10 random texts from largest cluster (Regulation, Legislation)
df[df.cluster == 0].sample(N, random_state=0)[["article_title", "cluster"]]
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
      <th>article_title</th>
      <th>cluster</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>3720</th>
      <td>'Not an Audit': IRS Notice Could Spell Trouble for Crypto Traders</td>
      <td>0</td>
    </tr>
    <tr>
      <th>146</th>
      <td>South Korean Crypto Transactions Command an Average of $7 Billion per Day on Domestic Exchanges</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1607</th>
      <td>Prepping for an Airdrop: Prominent BCH Community Members Bid Bitcoin ABC Devs Farewell</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3080</th>
      <td>Iran, Turkey, Malaysia Discuss Creating Unified Muslim Cryptocurrency</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3102</th>
      <td>Indian Crypto Community Gathers to Dispel Confusion About Regulation</td>
      <td>0</td>
    </tr>
    <tr>
      <th>620</th>
      <td>Crypto Mining Hardware Manufacturer Whatsminer Plans to Launch Public Offering in the US</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2431</th>
      <td>ICO Crackdown: 11 Class-Action Lawsuits Filed Against Cryptocurrency Companies</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1941</th>
      <td>Advocacy Group Claims IRS-Issued Crypto-Warning Letters Violated Taxpayer's Rights</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1786</th>
      <td>Bitcoin Cash Difficulty Algorithm Debate Heats Up With Fears of Another Chain Split</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4417</th>
      <td>Governments Exploit Christchurch Tragedy to Enforce Censorship and Thought Policing</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2651</th>
      <td>Bitcoin Rich: 6 Crypto Entrepreneurs Make the 2020 Hurun Billionaire List</td>
      <td>0</td>
    </tr>
    <tr>
      <th>278</th>
      <td>Pricing Gold, Food, and Altcoins With the BTC Denominator: How to Measure an Asset's Worth in Bitcoin</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2871</th>
      <td>Bitcoin Games Celebrates Lunar New Year 2020 With Introduction of Free Spins</td>
      <td>0</td>
    </tr>
    <tr>
      <th>60</th>
      <td>Harris Poll Data Shows 1 in 10 Americans Will Use Stimulus Checks to Buy Bitcoin and Ethereum</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1866</th>
      <td>Here's How to Check If a Bitcoin Address Is a Scam</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```python
#check N random texts from various clusters
df[df.cluster == 1].sample(N, random_state=0)[["article_title", "cluster"]]
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
      <th>article_title</th>
      <th>cluster</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2465</th>
      <td>Free from Tron: Steemit’s Blockchain Fork Hive Outperforms Steem Token Value</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1721</th>
      <td>Andreas Antonopoulos: Hex Team Offered Me 10 BTC to Speak Well of Their Token</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3042</th>
      <td>Shenzhen Stock Exchange Launches Index of Top 50 Blockchain Public Companies</td>
      <td>1</td>
    </tr>
    <tr>
      <th>342</th>
      <td>Mirror Trading International Named Biggest Crypto Scam of the Year After Raking in $589 Million</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3240</th>
      <td>Bankrupt DX Exchange Reportedly Didn’t Pay Israeli Suppliers and Employees</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2165</th>
      <td>Eidoo to Burn 80% of Company’s Token Holdings to Kickstart One of the Biggest Daos in Defi</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1796</th>
      <td>European Bitcoin Exchange Hacked for $1.4 Million, Claims It Cannot Afford to Repay Users</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4086</th>
      <td>Massive Growth by P2P Exchange and AT&amp;T Accepting BCH in the Weekly Update From Bitcoin.com</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4493</th>
      <td>Trading Platform Voltaire Rewards Users With BCH for Every Trade</td>
      <td>1</td>
    </tr>
    <tr>
      <th>44</th>
      <td>Ahead of IPO, Coinbase Pays CFTC $6.5 Million to Settle False Reporting, Wash Trading Charges</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2319</th>
      <td>CoinTracking Launches Two Powerful, Free Plug-ins for Crypto Traders</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2972</th>
      <td>Honk Launches Token Sale to Fund SLP-Powered Sports Betting Site</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1340</th>
      <td>Centralized Exchange Operators Believe Low Liquidity on DEX Platforms Stops User Migration</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4041</th>
      <td>Malaysia Begins Approving Crypto Exchange Registrations</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1530</th>
      <td>NFT Economy Grows Exponential: $1M in Non-Fungible Token Sales Last Week</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>




```python
#check 10 random texts from various clusters
df[df.cluster == 2].sample(N, random_state=0)[["article_title", "cluster"]]
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
      <th>article_title</th>
      <th>cluster</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>3535</th>
      <td>Why Bitcoin Is Better Than Banks: Major Credit Card Breach Exposes 60M Accounts</td>
      <td>2</td>
    </tr>
    <tr>
      <th>3502</th>
      <td>Coinbase Receives E-Money License From the Central Bank of Ireland</td>
      <td>2</td>
    </tr>
    <tr>
      <th>521</th>
      <td>Bahrain Central Bank Issues License to Shariah-Compliant Crypto Exchange</td>
      <td>2</td>
    </tr>
    <tr>
      <th>283</th>
      <td>Crypto Earning vs. Savings Accounts: How You Can Get Up to 17% Annually Holding Digital Assets</td>
      <td>2</td>
    </tr>
    <tr>
      <th>2506</th>
      <td>Liquid Gold vs Digital Gold: Why Bitcoin Beats Oil in the Current Climate</td>
      <td>2</td>
    </tr>
    <tr>
      <th>3904</th>
      <td>Swedish Central Bank Ponders Even Lower NIRP</td>
      <td>2</td>
    </tr>
    <tr>
      <th>954</th>
      <td>'Absolutely Inevitable': Standard Chartered Bank CEO Sees Widespread Cryptocurrency Adoption</td>
      <td>2</td>
    </tr>
    <tr>
      <th>1312</th>
      <td>Peter Schiff's Euro Pacific Bank Under Investigation by Tax Authorities in 5 Countries</td>
      <td>2</td>
    </tr>
    <tr>
      <th>1191</th>
      <td>Swiss Bank Sygnum Begins Storing Crypto With Taurus Group, Launches Staking Service</td>
      <td>2</td>
    </tr>
    <tr>
      <th>56</th>
      <td>Bank of America Says the Only Good Reason for Holding Bitcoin Is 'Sheer Price Appreciation'</td>
      <td>2</td>
    </tr>
    <tr>
      <th>3439</th>
      <td>Bank Crisis Spreads in India - Deaths, Strike, Supreme Court Denial</td>
      <td>2</td>
    </tr>
    <tr>
      <th>2420</th>
      <td>Deutsche Bank Envisions Post Covid-19 Economy Accelerating Digital Payments</td>
      <td>2</td>
    </tr>
    <tr>
      <th>1883</th>
      <td>Philippines Now Has 16 Cryptocurrency Exchanges Approved by Central Bank</td>
      <td>2</td>
    </tr>
    <tr>
      <th>615</th>
      <td>New Jersey Gym Claims State Seized $173K From Bank, Owner Discusses Cryptocurrency Solutions With Tucker Carlson</td>
      <td>2</td>
    </tr>
    <tr>
      <th>3653</th>
      <td>Big Banks Won't Touch Crypto Clients – But These Smaller Banks Will</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>




```python
#check 10 random texts from various clusters
df[df.cluster == 3].sample(N, random_state=0)[["article_title", "cluster"]]
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
      <th>article_title</th>
      <th>cluster</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2341</th>
      <td>The Greatest Wealth Transfer: Economists Predict the Emergence of New Bitcoin Millionaires, BTC Capturing 3% of Gold Market</td>
      <td>3</td>
    </tr>
    <tr>
      <th>741</th>
      <td>'Noxious Poison' - Bitcoin's Market Cap Surpasses Warren Buffet's Berkshire Hathaway Valuation</td>
      <td>3</td>
    </tr>
    <tr>
      <th>1046</th>
      <td>Market Update: Bull Trap Warnings After Bitcoin Shoots Above $18k Handle</td>
      <td>3</td>
    </tr>
    <tr>
      <th>2394</th>
      <td>Market Outlook: Bitcoin's Inverse Head and Shoulders, Covid-19 Fears Decline, CME Futures Gaps</td>
      <td>3</td>
    </tr>
    <tr>
      <th>4228</th>
      <td>Crypto of the Day App Gauges Investor Sentiment and Market Trends</td>
      <td>3</td>
    </tr>
    <tr>
      <th>4127</th>
      <td>Haasbot Is an Automated Trading Tool for the Crypto Market</td>
      <td>3</td>
    </tr>
    <tr>
      <th>341</th>
      <td>Darknet Market Joker’s Stash Retires After Raking in $1 Billion in Cryptocurrencies</td>
      <td>3</td>
    </tr>
    <tr>
      <th>3142</th>
      <td>SLP Token Market Cap Surpasses $50 Million</td>
      <td>3</td>
    </tr>
    <tr>
      <th>1699</th>
      <td>Capital One Files Patent for Cryptocurrency Market AI Prediction System</td>
      <td>3</td>
    </tr>
    <tr>
      <th>1923</th>
      <td>Market Outlook: Crypto Economy Sheds $10B Overnight, Exchange Inflow and Sell Pressure Rises</td>
      <td>3</td>
    </tr>
    <tr>
      <th>291</th>
      <td>Bitcoin's Watershed Moment- Crypto Asset Now Commands a $1 Trillion Market Capitalization</td>
      <td>3</td>
    </tr>
    <tr>
      <th>2545</th>
      <td>Traders Flock to Tether, USDC, PAX - Stablecoins See Great Demand After Crypto Market Havoc</td>
      <td>3</td>
    </tr>
    <tr>
      <th>4372</th>
      <td>How a Large Cryptocurrency Mining Operation Is Handling the Current Market</td>
      <td>3</td>
    </tr>
    <tr>
      <th>2941</th>
      <td>White House Market Wants to Become the Darknet’s Toughest DNM</td>
      <td>3</td>
    </tr>
    <tr>
      <th>3889</th>
      <td>Market Outlook: Gold and Crypto Reap the Benefits of Economic Fear</td>
      <td>3</td>
    </tr>
  </tbody>
</table>
</div>




```python
#checking top words by cluster
import nltk
df["words"] = df.article_title.apply(lambda x: nltk.word_tokenize(x))
df["tagged_words"] = df.words.apply(lambda x: nltk.pos_tag(x))

from collections import Counter

def get_counter(dataframe, stopwords=[]):
    counter = Counter()
    
    for l in dataframe.tagged_words:
        word_set = set()

        for t in l:
            word = t[0].lower()
            tag = t[1]

            if word not in stopwords:
                word_set.add(word)
            
        counter.update(word_set)
        
    return counter
```


```python
#Second  cluster - Theme around market performance/pricing
counter0 = get_counter(df[df.cluster == 0], global_stopwords+local_stopwords)
counter0.most_common(30)
```




    [('mining', 144),
     ('price', 106),
     ('government', 93),
     ('markets', 79),
     ('gold', 78),
     ('cryptocurrencies', 78),
     ('indian', 75),
     ('miners', 73),
     ('exchanges', 72),
     ('worth', 68),
     ('network', 67),
     ('defi', 66),
     ('money', 66),
     ('report', 63),
     ('investors', 63),
     ('–', 63),
     ('ethereum', 62),
     ('year', 61),
     ('court', 61),
     ('buy', 59),
     ('satoshi', 59),
     ('china', 58),
     ('wallet', 57),
     ('could', 55),
     ('tax', 54),
     ('fund', 54),
     ('shows', 53),
     ('2020', 53),
     ('payments', 50),
     ('sec', 50)]




```python
#Fourth Cluster
counter0 = get_counter(df[df.cluster == 1], global_stopwords+local_stopwords)
counter0.most_common(30)
```




    [('exchange', 204),
     ('trading', 116),
     ('token', 103),
     ('launches', 80),
     ('slp', 39),
     ('platform', 24),
     ('launch', 20),
     ('stock', 20),
     ('tokens', 18),
     ('users', 16),
     ('indian', 16),
     ('south', 14),
     ('defi', 14),
     ('coinbase', 13),
     ('p2p', 13),
     ('major', 12),
     ('investors', 12),
     ('first', 12),
     ('support', 11),
     ('court', 10),
     ('fund', 10),
     ('company', 10),
     ('bank', 10),
     ('adds', 10),
     ('wallet', 10),
     ('futures', 10),
     ('tax', 10),
     ('traders', 9),
     ('binance', 9),
     ('india', 9)]




```python
#Top cluster -- Theme around banking and electronic exhanges/trading
counter0 = get_counter(df[df.cluster == 2], global_stopwords+local_stopwords)
counter0.most_common(30)
```




    [('bank', 178),
     ('digital', 127),
     ('banks', 85),
     ('central', 80),
     ('currency', 33),
     ('china', 22),
     ('yuan', 20),
     ('services', 19),
     ('gold', 19),
     ('major', 18),
     ('cryptocurrencies', 17),
     ('interest', 16),
     ('currencies', 15),
     ('rates', 15),
     ('dollar', 13),
     ('assets', 13),
     ('asset', 12),
     ('negative', 12),
     ('government', 12),
     ('accounts', 12),
     ('governor', 11),
     ('deutsche', 11),
     ('chinese', 11),
     ('indian', 11),
     ('crisis', 11),
     ('regulation', 10),
     ('financial', 10),
     ('launches', 10),
     ('money', 10),
     ('trading', 9)]




```python
#Third Cluster
counter0 = get_counter(df[df.cluster == 3], global_stopwords+local_stopwords)
counter0.most_common(30)
```




    [('market', 181),
     ('cap', 27),
     ('prices', 19),
     ('outlook', 19),
     ('price', 13),
     ('stock', 11),
     ('darknet', 9),
     ('gold', 9),
     ('token', 8),
     ('traders', 8),
     ('report', 7),
     ('economy', 7),
     ('futures', 7),
     ('defi', 7),
     ('uncertainty', 7),
     ('since', 6),
     ('2020', 6),
     ('surpasses', 6),
     ('estate', 6),
     ('real', 6),
     ('trillion', 5),
     ('spike', 5),
     ('global', 5),
     ('bull', 5),
     ('drop', 5),
     ('tether', 5),
     ('fear', 5),
     ('markets', 5),
     ('bullish', 5),
     ('predictions', 5)]



## K = 4

1) The largest cluster is '0' and the count is 3507. The theme is hard to decipher with a hodge podge of topics suggesting it may contain too many clusters.
2) The second largest cluster is '1' and the count is 445. The theme is around electronic exhanges/trading.
3) The third cluster is '2' and the count is 363. The theme is around institutions and banking.
4) The smallest cluster is '3' and the count is 183. The theme is potentially the value of bitcoin. 


```python
df
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
      <th>file_name</th>
      <th>article_title</th>
      <th>article_date_time</th>
      <th>article_author</th>
      <th>article_text</th>
      <th>cluster</th>
      <th>words</th>
      <th>tagged_words</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>domestic-and-foreign-buyers-acquired-a-tokenized-apartment-in-a-spanish-city-by-paying-with-ethereum.html</td>
      <td>Domestic and Foreign Buyers Acquired a 'Tokenized' Apartment in a Spanish City by Paying With Ethereum</td>
      <td>April 08, 2021</td>
      <td>Felipe Erazo</td>
      <td>Domestic and Foreign Buyers Acquired a 'Tokenized' Apartment in a Spanish City by Paying With Ethereum                        National and foreign...</td>
      <td>0</td>
      <td>[Domestic, and, Foreign, Buyers, Acquired, a, 'Tokenized, ', Apartment, in, a, Spanish, City, by, Paying, With, Ethereum]</td>
      <td>[(Domestic, JJ), (and, CC), (Foreign, NNP), (Buyers, NNP), (Acquired, VBD), (a, DT), ('Tokenized, VBN), (', POS), (Apartment, NNP), (in, IN), (a, ...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>zeitgeist-brings-a-prediction-markets-ecosystem-to-polkadots-kusama.html</td>
      <td>Zeitgeist Brings a Prediction Markets Ecosystem to Polkadot’s Kusama</td>
      <td>April 08, 2021</td>
      <td>Reuben Jackson</td>
      <td>Zeitgeist Brings a Prediction Markets Ecosystem to Polkadot’s Kusama                        To capitalize on the growing popularity of blockchain ...</td>
      <td>0</td>
      <td>[Zeitgeist, Brings, a, Prediction, Markets, Ecosystem, to, Polkadot, ’, s, Kusama]</td>
      <td>[(Zeitgeist, JJ), (Brings, VBZ), (a, DT), (Prediction, NN), (Markets, NNP), (Ecosystem, NNP), (to, TO), (Polkadot, NNP), (’, NNP), (s, VBD), (Kusa...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>taraxa-listing-on-asecendex.html</td>
      <td>Taraxa Listing on AsecendEX</td>
      <td>April 08, 2021</td>
      <td>NaN</td>
      <td>Taraxa Listing on AsecendEX                press release         PRESS RELEASE. AscendEX, formerly BitMax, an industry-leading digital asset tradi...</td>
      <td>0</td>
      <td>[Taraxa, Listing, on, AsecendEX]</td>
      <td>[(Taraxa, NNP), (Listing, NNP), (on, IN), (AsecendEX, NNP)]</td>
    </tr>
    <tr>
      <th>3</th>
      <td>fio-protocol-marks-first-year-anniversary.html</td>
      <td>FIO Protocol Marks First Year Anniversary</td>
      <td>April 08, 2021</td>
      <td>NaN</td>
      <td>FIO Protocol Marks First Year Anniversary                press release         PRESS RELEASE. The Foundation for Interwallet Operability (FIO) cel...</td>
      <td>0</td>
      <td>[FIO, Protocol, Marks, First, Year, Anniversary]</td>
      <td>[(FIO, NNP), (Protocol, NNP), (Marks, NNP), (First, NNP), (Year, NNP), (Anniversary, NNP)]</td>
    </tr>
    <tr>
      <th>4</th>
      <td>alchemy-chooses-flow-blockchain-to-accelerate-game-changing-developer-ecosystem.html</td>
      <td>Alchemy Chooses Flow Blockchain to Accelerate Game-Changing Developer Ecosystem</td>
      <td>April 08, 2021</td>
      <td>NaN</td>
      <td>Alchemy Chooses Flow Blockchain to Accelerate Game-Changing Developer Ecosystem                press release         PRESS RELEASE. The leading bl...</td>
      <td>0</td>
      <td>[Alchemy, Chooses, Flow, Blockchain, to, Accelerate, Game-Changing, Developer, Ecosystem]</td>
      <td>[(Alchemy, NNP), (Chooses, NNP), (Flow, NNP), (Blockchain, NNP), (to, TO), (Accelerate, VB), (Game-Changing, NNP), (Developer, NNP), (Ecosystem, N...</td>
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
    </tr>
    <tr>
      <th>4493</th>
      <td>trading-platform-voltaire-rewards-users-with-bch-for-every-trade.html</td>
      <td>Trading Platform Voltaire Rewards Users With BCH for Every Trade</td>
      <td>Mar 8, 2019</td>
      <td>Jamie Redman</td>
      <td>Trading Platform Voltaire Rewards Users With BCH for Every Trade                        On March 8, London-based exchange Voltaire announced a new...</td>
      <td>1</td>
      <td>[Trading, Platform, Voltaire, Rewards, Users, With, BCH, for, Every, Trade]</td>
      <td>[(Trading, NN), (Platform, NNP), (Voltaire, NNP), (Rewards, NNP), (Users, NNP), (With, IN), (BCH, NNP), (for, IN), (Every, NNP), (Trade, NNP)]</td>
    </tr>
    <tr>
      <th>4494</th>
      <td>un-panel-north-korea-hacked-571m-from-asian-crypto-exchanges.html</td>
      <td>UN Panel: North Korea Hacked $571M From Asian Crypto Exchanges</td>
      <td>Mar 8, 2019</td>
      <td>Jeffrey Gogo</td>
      <td>UN Panel: North Korea Hacked $571M From Asian Crypto Exchanges                        The U.N. Security Council has heard that North Korea uses cy...</td>
      <td>0</td>
      <td>[UN, Panel, :, North, Korea, Hacked, $, 571M, From, Asian, Crypto, Exchanges]</td>
      <td>[(UN, JJ), (Panel, NNP), (:, :), (North, NNP), (Korea, NNP), (Hacked, VBD), ($, $), (571M, CD), (From, NNP), (Asian, JJ), (Crypto, NNP), (Exchange...</td>
    </tr>
    <tr>
      <th>4495</th>
      <td>russian-localbitcoins-trade-outpaces-venezuela.html</td>
      <td>P2P Markets: Russian Localbitcoins Trade Outpaces Venezuela</td>
      <td>Mar 8, 2019</td>
      <td>Samuel Haig</td>
      <td>P2P Markets: Russian Localbitcoins Trade Outpaces Venezuela                        During the week of March 2, roughly 14,365 BTC changed hands vi...</td>
      <td>0</td>
      <td>[P2P, Markets, :, Russian, Localbitcoins, Trade, Outpaces, Venezuela]</td>
      <td>[(P2P, NN), (Markets, NNS), (:, :), (Russian, JJ), (Localbitcoins, NNP), (Trade, NNP), (Outpaces, NNP), (Venezuela, NNP)]</td>
    </tr>
    <tr>
      <th>4496</th>
      <td>markets-update-cryptocurrencies-struggle-to-surpass-upper-resistance.html</td>
      <td>Markets Update: Cryptocurrencies Struggle to Surpass Upper Resistance</td>
      <td>Mar 8, 2019</td>
      <td>Jamie Redman</td>
      <td>Markets Update: Cryptocurrencies Struggle to Surpass Upper Resistance                        Cryptocurrency markets have been looking more optimis...</td>
      <td>0</td>
      <td>[Markets, Update, :, Cryptocurrencies, Struggle, to, Surpass, Upper, Resistance]</td>
      <td>[(Markets, NNS), (Update, NNP), (:, :), (Cryptocurrencies, NNS), (Struggle, VBP), (to, TO), (Surpass, NNP), (Upper, NNP), (Resistance, NNP)]</td>
    </tr>
    <tr>
      <th>4497</th>
      <td>universal-protocol-alliance-launches-interest-paying-stablecoin-upusd.html</td>
      <td>Universal Protocol Alliance Launches Interest-Paying Stablecoin UPUSD</td>
      <td>Mar 8, 2019</td>
      <td>Kai Sedgwick</td>
      <td>Universal Protocol Alliance Launches Interest-Paying Stablecoin UPUSD                        A consortium of blockchain companies including Bittre...</td>
      <td>1</td>
      <td>[Universal, Protocol, Alliance, Launches, Interest-Paying, Stablecoin, UPUSD]</td>
      <td>[(Universal, NNP), (Protocol, NNP), (Alliance, NNP), (Launches, NNP), (Interest-Paying, NNP), (Stablecoin, NNP), (UPUSD, NNP)]</td>
    </tr>
  </tbody>
</table>
<p>4498 rows × 8 columns</p>
</div>



## LDA Topic Modeling


```python
#step 1: set the number of topics
num_topics = 4
```


```python
#step 2: Initialize a model for LDA topic modeling
from sklearn.decomposition import LatentDirichletAllocation as LDA

lda = LDA(n_components=num_topics, random_state=0)
lda
```




    LatentDirichletAllocation(n_components=4, random_state=0)




```python
#step 3: fit the model
%time lda.fit(X)
```

    CPU times: user 5.26 s, sys: 0 ns, total: 5.26 s
    Wall time: 5.26 s





    LatentDirichletAllocation(n_components=4, random_state=0)




```python
#step 4: Examine the output topic

def show_topics(model, feature_names, num_top_words):
    for topic_idx, topic_scores in enumerate(model.components_):
        print("***Topic {}:".format(topic_idx))
        print(" + ".join(["{:.2f} * {}".format(topic_scores[i], feature_names[i]) for i in topic_scores.argsort()[::-1][:num_top_words]]))
        print()
        
show_topics(lda, vectorizer.get_feature_names(), 10)
```

    ***Topic 0:
    13.01 * mining + 11.60 * 000 + 9.98 * token + 9.86 * market + 9.76 * scam + 8.21 * exchange + 7.93 * prices + 7.32 * markets + 7.22 * video + 6.99 * miners
    
    ***Topic 1:
    12.97 * market + 12.51 * price + 12.12 * digital + 11.21 * bank + 10.77 * fees + 10.48 * satoshi + 9.53 * buy + 9.34 * fund + 9.12 * government + 8.92 * markets
    
    ***Topic 2:
    21.29 * exchange + 16.65 * gold + 16.30 * trading + 15.43 * futures + 14.68 * bank + 14.66 * court + 13.87 * indian + 13.59 * market + 13.24 * platform + 11.90 * launches
    
    ***Topic 3:
    15.24 * banks + 15.00 * bank + 14.92 * exchanges + 11.77 * exchange + 11.63 * tax + 9.97 * services + 9.50 * rates + 8.87 * interest + 8.56 * year + 8.53 * launches
    



```python
from collections import Counter
import networkx as nx
from matplotlib import pyplot as plt
###################################################################################
# The 'counter' object will have all the word count information. 
# The 'co_counter' object will have all the co-occurrence count information.
###################################################################################
counter = Counter()
co_counter = dict()

for l in df.words:
    word_set = set()
    
    for item in l:
        word = item.lower()
        
        if word not in (global_stopwords + local_stopwords):
            word_set.add(word)

    counter.update(word_set)
    
    ###################################################################################
    # Calculate co-occurrence count of two words and save it in 'co_counter'.
    # Co_counter is a dictionary of dictionaries.
    ###################################################################################
    words = list(word_set)
    for word1 in words:
        if word1 not in co_counter:
            co_counter[word1] = dict()
        
        for word2 in words:
            ######################################
            # Skip if the two words are the same.
            ######################################
            if word1 == word2:
                continue
            
            if word2 not in co_counter[word1]:
                co_counter[word1][word2] = 1
            else:
                co_counter[word1][word2] += 1
                

G = nx.Graph()
nodes = [item[0] for item in counter.most_common(20)]
node_weights = [item[1] * 15 for item in counter.most_common(20)]

for word in nodes:
    G.add_node(word, weight=counter.get(word))

for word1 in nodes:
    for word2 in nodes:
        if (word1 != word2) & (word2 in co_counter[word1]):
            G.add_edge(word1, word2, weight=co_counter[word1][word2])

edges = nx.get_edge_attributes(G, "weight").keys()
edge_weights = nx.get_edge_attributes(G, "weight").values()

edge_weights = [item / 5 for item in edge_weights]

plt.figure(figsize=(10, 10))
nx.draw_networkx(G, pos=nx.circular_layout(G), 
                 nodelist=nodes, node_size=node_weights, edgelist=edges, width=edge_weights,
                 node_color="yellow", with_labels=True, font_size=10)
plt.draw()
```


    
![png](output_38_0.png)
    


SUMMARY:
1) There seems to be a resonable corelation between the clusters and the topic analysis when they both are set to 4.<br> 
<br>
2) The weighted words in each of the topics are aligned with what we observed through the cluster analysis.<br>
<br>
3) Another interesting observation was the comparison between bitcoin and gold in cluster=2/topic=2. Random analysis of our sentences revealed that there was a discussion around the limited quantity of both bitcoin and gold. In both cases, their value is supported, in part, by scarcity. Gold is limited by physical supply and the difficulty of extraction, while bitcoin creation is capped at 21 million by its source code. These qualities, as well as the deep liquid markets, mean that both gold and bitcoin have the potential to retain value, and in fact appreciate, during difficult economic cycles. 
