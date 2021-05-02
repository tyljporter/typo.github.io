---
layout: archive
permalink: /BitcoinPart2/
title: "Bitcoin Sentiment Classifier"
author_profile: true
---

# Text Analytics | BAIS:6100
# Team Project Part 2
### Sentiment Classifier


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
#check the dataframe
#article date time needs to be assessed... it appears each recent web page is today - web page count(days) until a certain point
df.head(15)
#df.tail(15)
len(df)
```




    4498




```python
#check format of dates
df.article_date_time
#df.article_text
```




    0        53 mins ago
    1        3 hours ago
    2        4 hours ago
    3       12 hours ago
    4       20 hours ago
                ...     
    4493     Mar 8, 2019
    4494     Mar 8, 2019
    4495     Mar 8, 2019
    4496     Mar 8, 2019
    4497     Mar 8, 2019
    Name: article_date_time, Length: 4498, dtype: object




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
#check format of dates
df.article_date_time[:50]
```




    0     April 08, 2021
    1     April 08, 2021
    2     April 08, 2021
    3     April 08, 2021
    4     April 08, 2021
    5     April 06, 2021
    6     April 05, 2021
    7     April 05, 2021
    8     April 06, 2021
    9     April 05, 2021
    10    April 05, 2021
    11    April 05, 2021
    12    April 05, 2021
    13    April 05, 2021
    14    April 05, 2021
    15    April 05, 2021
    16    April 05, 2021
    17    April 05, 2021
    18    April 05, 2021
    19    April 05, 2021
    20    April 04, 2021
    21    April 04, 2021
    22    April 04, 2021
    23    April 04, 2021
    24    April 04, 2021
    25    April 04, 2021
    26    April 04, 2021
    27    April 04, 2021
    28    April 04, 2021
    29    April 04, 2021
    30    April 04, 2021
    31    April 03, 2021
    32    April 03, 2021
    33    April 03, 2021
    34    April 03, 2021
    35    April 03, 2021
    36    April 03, 2021
    37    April 03, 2021
    38    April 02, 2021
    39    April 02, 2021
    40      Mar 19, 2020
    41    April 02, 2021
    42    April 02, 2021
    43    April 02, 2021
    44    April 02, 2021
    45    April 02, 2021
    46    April 01, 2021
    47    April 01, 2021
    48    April 01, 2021
    49    April 01, 2021
    Name: article_date_time, dtype: object




```python
#tokenize article text into sentences
df["sentences"] = df.article_text.apply(lambda x: nltk.sent_tokenize(x))
df["num_sentences"] = df.sentences.apply(lambda x: len(x))
```


```python
#check df for issues
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
      <th>sentences</th>
      <th>num_sentences</th>
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
      <td>[Domestic and Foreign Buyers Acquired a 'Tokenized' Apartment in a Spanish City by Paying With Ethereum                        National and foreig...</td>
      <td>23</td>
    </tr>
    <tr>
      <th>1</th>
      <td>zeitgeist-brings-a-prediction-markets-ecosystem-to-polkadots-kusama.html</td>
      <td>Zeitgeist Brings a Prediction Markets Ecosystem to Polkadot’s Kusama</td>
      <td>April 08, 2021</td>
      <td>Reuben Jackson</td>
      <td>Zeitgeist Brings a Prediction Markets Ecosystem to Polkadot’s Kusama                        To capitalize on the growing popularity of blockchain ...</td>
      <td>[Zeitgeist Brings a Prediction Markets Ecosystem to Polkadot’s Kusama                        To capitalize on the growing popularity of blockchain...</td>
      <td>29</td>
    </tr>
    <tr>
      <th>2</th>
      <td>taraxa-listing-on-asecendex.html</td>
      <td>Taraxa Listing on AsecendEX</td>
      <td>April 08, 2021</td>
      <td>NaN</td>
      <td>Taraxa Listing on AsecendEX                press release         PRESS RELEASE. AscendEX, formerly BitMax, an industry-leading digital asset tradi...</td>
      <td>[Taraxa Listing on AsecendEX                press release         PRESS RELEASE., AscendEX, formerly BitMax, an industry-leading digital asset tra...</td>
      <td>19</td>
    </tr>
    <tr>
      <th>3</th>
      <td>fio-protocol-marks-first-year-anniversary.html</td>
      <td>FIO Protocol Marks First Year Anniversary</td>
      <td>April 08, 2021</td>
      <td>NaN</td>
      <td>FIO Protocol Marks First Year Anniversary                press release         PRESS RELEASE. The Foundation for Interwallet Operability (FIO) cel...</td>
      <td>[FIO Protocol Marks First Year Anniversary                press release         PRESS RELEASE., The Foundation for Interwallet Operability (FIO) c...</td>
      <td>26</td>
    </tr>
    <tr>
      <th>4</th>
      <td>alchemy-chooses-flow-blockchain-to-accelerate-game-changing-developer-ecosystem.html</td>
      <td>Alchemy Chooses Flow Blockchain to Accelerate Game-Changing Developer Ecosystem</td>
      <td>April 08, 2021</td>
      <td>NaN</td>
      <td>Alchemy Chooses Flow Blockchain to Accelerate Game-Changing Developer Ecosystem                press release         PRESS RELEASE. The leading bl...</td>
      <td>[Alchemy Chooses Flow Blockchain to Accelerate Game-Changing Developer Ecosystem                press release         PRESS RELEASE., The leading ...</td>
      <td>21</td>
    </tr>
  </tbody>
</table>
</div>




```python
#check sentences for issues
df.sentences[0][2]

```




    'Transaction Was Worth Over $64,000 According to El Correo Web, the Spanish crypto exchange Criptan and the investment firm Rental IT arranged the transaction of the property, tokenized by the real estate company.'



## Collect 2000 random sentences


```python
#collect 2000 random sentences

#merge all sentences to a single list

rows = len(df)

All_Sentences = []

for row in range(rows):
    All_Sentences.extend(df.sentences[row])
```


```python
#check All_Sentences for issues

All_Sentences[0]
len(All_Sentences)
```




    161198




```python
All_Sentences
```




    ["Domestic and Foreign Buyers Acquired a 'Tokenized' Apartment in a Spanish City by Paying With Ethereum                        National and foreign buyers acquired a “tokenized” apartment in Spain with ethereum, which involved a domestic crypto exchange and a real estate investment firm.",
     'The city of Sevilla was the scenario for the crypto deal between the parties.',
     'Transaction Was Worth Over $64,000 According to El Correo Web, the Spanish crypto exchange Criptan and the investment firm Rental IT arranged the transaction of the property, tokenized by the real estate company.',
     'The media outlet specified the tokenization — split into several tokens and digitalized through a blockchain — was done to target 32 investors from Spain, Argentina, and Mexico.',
     'With the acquisition, the buyers that disbursed their funds paying with ethereum (ETH) participated in the investment starting from 100 euros ($117).',
     'Afterward, Rental IT exchanged the cryptos into fiat (euros) and then purchased the property from the owner.',
     'The transaction was reportedly worth 54,773 euros ($64,430), said El Correo Web.',
     'Jorge Soriano, the cofounder of Criptan, commented on the transaction: It is the start for small savers to access a sector that until now has been difficult to access if it was not in your own country and eliminate excess bureaucracy.',
     'In addition to confirming the total security that exists behind these transactions through blockchain technology.',
     'In the same line, Eric Sánchez, Rental IT’s CEO, believes such transactions pave the way for other brokers to sell real estate and other goods via crypto.',
     'Moreover, he forecasts that in ten years, the real estate sector could shift towards such adoption.',
     'Crypto Real Estate on the Rise in the Iberian Peninsula Crypto real estate’s boom in Spain has been making the headlines in the last few months, despite the coronavirus pandemic.',
     'Bitcoin.com News reported early in March that Piso Barato Inmobiliaria, a Tenerife-based real estate firm, has enabled the feature that allows customers to acquire properties by paying with bitcoin (BTC).',
     'Also, another Spanish real estate platform, Idealista, listed in February its first-ever property for sale for cryptocurrency — an apartment located in the center of Barcelona.',
     'What do you think about this transaction involving a “tokenized” apartment?',
     'Let us know in the comments section below.',
     "Iconic 'Obey' Street Artist Shepard Fairey to Auction an NFT Mural on Superrare                                        NEWS | 5 hours ago                                Kim Dotcom Insists 'Bitcoin Cash Will Serve the Mass Market,' Criticizes BTC Fees, Elon Musk Responds                                        NEWS | 9 hours ago                             Tags in this story                   crypto exchange, Cryptocurrencies, Europe, property, Real estate, Spain, Spain Crypto, tokenized, tokenized assets    Image Credits: Shutterstock, Pixabay, Wiki Commons Spot-markets for Bitcoin, Bitcoin Cash, Ripple, Litecoin and more.",
     'Start your trading here.',
     'Disclaimer: This article is for informational purposes only.',
     'It is not a direct offer or solicitation of an offer to buy or sell, or a recommendation or endorsement of any products, services, or companies.',
     'Bitcoin.com does not provide investment, tax, legal, or accounting advice.',
     'Neither the company nor the author is responsible, directly or indirectly, for any damage or loss caused or alleged to be caused by or in connection with the use of or reliance on any content, goods or services mentioned in this article.',
     'Read disclaimerShow comments',
     'Zeitgeist Brings a Prediction Markets Ecosystem to Polkadot’s Kusama                        To capitalize on the growing popularity of blockchain trends like decentralized finance (defi), Zeitgeist will be rolling out a new native prediction platform for Polkadot to expand the scope of opportunities available for retail crypto investors.',
     'New Network to Build on Substrate and Launch Exclusively on Kusama Long before Satoshi Nakamoto unveiled the Bitcoin whitepaper that led to a flood of blockchain innovation, prediction markets were available in the form of organized betting, often on political outcomes.',
     'For centuries these markets existed, helping corral crowd-based wisdom in the form of betting odds, which remain popular to this day.',
     'The blockchain iteration of these prediction markets promotes the same idea: determining the probability of informational outcomes based on speculation.',
     'Unlike cryptocurrency trading, these markets are binary, meaning either a positive or negative outcome results from a contract between two parties.',
     'Predictions made using these systems are considered relatively accurate, either matching or outpacing alternative results from focus groups, experts, and opinion polls.',
     'As prediction markets intersect with the latest defi craze, one company is probing new uses for blockchain’s architecture in another significant expansion of decentralized finance’s horizons.',
     'Zeitgeist, a startup that recently exited stealth mode, is pioneering a new base layer blockchain prediction market based on Substrate, the development framework pioneered by Polkadot.',
     'A Base Layer Approach After revealing the project and raising a $1.5 million seed round from a consortium of investors, Zeitgeist has unveiled its road map.',
     'The project, which will be exclusively governed by users holding its native ZTG token, expects to launch its testnet titled “Battery Park” and a Kusama parachain by the end of 2021.',
     'Unlike other prediction markets that operate as decentralized applications running on top of blockchains, Zeitgeist is building its chain, or base layer, which will be launched exclusively on Kusama.',
     'Although related to Polkadot, Kusama is designed to host experimental protocols while providing a testing ground that reflects real-world conditions.',
     'This will enable the prediction market to operate at scale while minimizing transaction costs, opening access to parachain assets, and enabling for forkless upgrades.',
     'Cryptocurrency exchanges ordinarily handle a limited number of cryptocurrency pairs and derivatives.',
     'By contrast, Zeitgeist plans to host thousands of different markets.',
     'Zeitgeist will initially support markets in cryptocurrency, sports and esports, politics, startups, and insurance.',
     'Together, these will open up immense potential opportunities to retail traders seeking to speculate on all manner of outcomes while also reviewing sentiment across informational areas.',
     'Over time the platform intends to aggregate the information and data collected on the network to improve decision-making, envisioning itself as a future hub for Web3 predictions.',
     'Interestingly, it is also rolling out a feature called Futarchy, which will use insights from its own prediction markets to help gradually evolve the platform’s governance.',
     'Moreover, its construction on Substrate will help Zeitgeist be compatible with Polkadot if it intends to build on a network parachain in the future.',
     'With the rise of blockchain prediction markets, what’s next for defi – sports betting?',
     'Let us know what you think in the comments section below.',
     'Livepeer Token Surges After Grayscale Investment Announced Creation of Trust to Invest in LPT                                        ALTCOINS | 23 hours ago                                Another Binance Smart Chain Project Turtledex Rug Pulls With Tokens Worth $2.5M Confirmed Stolen                                        ALTCOINS | 4 days ago                             Tags in this story                   DeFi, future of blockchain, Kusama, parachains, Polkadot, Prediction markets    Image Credits: Shutterstock, Pixabay, Wiki Commons, Zeitgeist, Kusama Purchase Bitcoin without visiting a cryptocurrency exchange.',
     'Buy BTC and BCH here.',
     'Disclaimer: This article is for informational purposes only.',
     'It is not a direct offer or solicitation of an offer to buy or sell, or a recommendation or endorsement of any products, services, or companies.',
     'Bitcoin.com does not provide investment, tax, legal, or accounting advice.',
     'Neither the company nor the author is responsible, directly or indirectly, for any damage or loss caused or alleged to be caused by or in connection with the use of or reliance on any content, goods or services mentioned in this article.',
     'Read disclaimerShow comments',
     'Taraxa Listing on AsecendEX                press release         PRESS RELEASE.',
     'AscendEX, formerly BitMax, an industry-leading digital asset trading platform built by Wall Street quant trading veterans, has announced the listing of Taraxa tokens (TARA) under the pair USDT/TARA  on Mar 23 at 9:00 a.m. EDT.',
     'Taraxa is a public ledger platform purpose-built for audit logging of informal transactions.',
     'Taraxa provides decentralized infrastructure to track informal transactional agreements and data to help minimize the cost and risk of confusion while eliminating disputes.',
     'The Taraxa token (TARA) is used for gas, staking, and governance on the Taraxa network.',
     'The vast majority of the world’s transactions are informal, uncaptured, and unverifiable, often leading to costly confusion and unintended disputes.',
     'Taraxa leverages inline capturing and audit logging to make these transaction data more trusted and seamless.',
     'Before blockchain technology emerged, it was next to impossible to make this informal data trustworthy.',
     'Taraxa has targeted and is in the process of capturing what they believe to be the largest opportunity for blockchain adoption.',
     'Taraxa’s goals for 2021 are launching their mainnet, and driving adoption for their application platforms.',
     'About AscendEX  Originally founded in 2018 as BitMax.io, AscendEx is a leading crypto and digital asset exchange catering to both professional and retail traders.',
     'The venue offers spot, futures, margin trading and staking products and incorporates key elements from the DeFi space to foster a unique market structure for users.',
     'AscendEx is led by a team of Wall Street veterans who have applied traditional markets’ rigor to create a secure, safe, and reliable experience for all participants; and a reliable source of liquidity for primary offerings.',
     'For more information and updates, please visit: Website: https://ascendex.com Twitter: https://twitter.com/AscendEX_Global Telegram: https://t.me/AscendEXEnglish Medium: https://medium.com/ascendex \xa0 About Taraxa  Taraxa is a public ledger platform purpose-built for audit logging of informal transactions.Taraxa provides decentralized infrastructure to track informal transactional agreements and data to help minimize the cost and risk of confusion while eliminating disputes.',
     'For more information and updates, please visit: Website: https://www.taraxa.io/ Twitter: https://twitter.com/taraxa_project  Telegram: https://t.me/taraxa_project Medium: https://medium.com/taraxa-project \xa0  This is a press release.',
     'Readers should do their own due diligence before taking any actions related to the promoted company or any of its affiliates or services.',
     'Bitcoin.com is not responsible, directly or indirectly, for any damage or loss caused or alleged to be caused by or in connection with the use of or reliance on any content, goods or services mentioned in the press release.',
     "Domestic and Foreign Buyers Acquired a 'Tokenized' Apartment in a Spanish City by Paying With Ethereum                                        NEWS | 53 mins ago                                 Zeitgeist Brings a Prediction Markets Ecosystem to Polkadot’s Kusama                                        ALTCOINS | 3 hours ago                             Tags in this story                   AsecendEX, Taraxa    Image Credits: Shutterstock, Pixabay, Wiki Commons Purchase Bitcoin without visiting a cryptocurrency exchange.",
     'Buy BTC and BCH here.',
     'FIO Protocol Marks First Year Anniversary                press release         PRESS RELEASE.',
     'The Foundation for Interwallet Operability (FIO) celebrates the first year anniversary since the community launched the mainnet of the open-source FIO protocol.',
     'With over one-hundred and fifty thousand users and some of the industry’s leading service providers such as ShapeShift, Coinomi, AscendEX (formerly Bitmax), Liquid.com and Edge integrating the FIO protocol, the team are on course to ensure cryptocurrency transactions are simple, private, secure and joyful.',
     'The FIO Protocol was designed to ensure the cryptocurrency user experience is uncomplicated and lays the foundation for cryptocurrency mass adoption.',
     'Long complex addresses have been replaced with user-customized FIO Addresses such as “user@fio” that enable decentralized sending, receiving and payment requests (invoices, order carts, etc.',
     '), as well as, cross-chain end-to-end encrypted metadata (the information for those invoices, order carts, etc.)',
     'that can work across all blockchains.',
     'Luke Stokes, Managing Director, FIO: The last year has seen the FIO Protocol and cryptocurrency industry mark a number of significant milestones.',
     'Many people are predicting continued exponential adoption for cryptocurrencies over the next few years mirroring the path the Internet took towards a billion users.',
     'For this to become an everyday reality, the user experience must improve.',
     'Sending an email used to be hard.',
     'Using the internet was difficult.',
     'Prior to the creation of http, we had no usability layer to bring about the browser revolution and the World Wide Web.',
     'We are now at a similar stage with cryptocurrencies and the FIO Protocol is designed to ensure the user experience for sending digital assets is as simple as sending an email.',
     'Within the first year since the mainnet launch, the FIO Protocol has been adopted by more than a dozen of the industry’s leading wallets ane exchanges, the FIO Token has been listed on Binance, Liquid, AscendEX, Whitebit, ChangeNow and other exchanges, and over 150,000 FIO Addresses have been registered.',
     'The FIO community is actively improving the protocol to include greater features such as FIO Staking and a FIO Dashboard to make buying, selling and managing a user’s FIO Addresses and Domain NFT’s easier than ever.',
     'As more and more institutional and retail investors begin to realize the potential benefits of blockchain application and asset allocation, the need for standardized infrastructure is only going to increase.',
     'About FIO: The (FIO) Protocol is a blockchain that enables easier, safer and more joyful crypto transactions on any type of token or coin.',
     'It serves as a usability layer for the entire blockchain ecosystem enabling, among other things, human readable wallet names that replace complex public addresses, the ability to request crypto payments versus just sending and the ability to include data such as notes, order carts or invoices alongside crypto transactions on every blockchain.',
     'The Foundation for Interwallet Operability is an industry guided non-profit entity whose mission is to facilitate improvement to and adoption of the FIO Protocol.',
     'Many of the industry’s leading wallets and exchanges have integrated the FIO protocol thereby enabling access to its capabilities for their users which, combined, represent millions of people.',
     'This is a press release.',
     'Readers should do their own due diligence before taking any actions related to the promoted company or any of its affiliates or services.',
     'Bitcoin.com is not responsible, directly or indirectly, for any damage or loss caused or alleged to be caused by or in connection with the use of or reliance on any content, goods or services mentioned in the press release.',
     "Domestic and Foreign Buyers Acquired a 'Tokenized' Apartment in a Spanish City by Paying With Ethereum                                        NEWS | 53 mins ago                                 Zeitgeist Brings a Prediction Markets Ecosystem to Polkadot’s Kusama                                        ALTCOINS | 3 hours ago                             Tags in this story                   FIO Protocol    Image Credits: Shutterstock, Pixabay, Wiki Commons Spot-markets for Bitcoin, Bitcoin Cash, Ripple, Litecoin and more.",
     'Start your trading here.',
     'Alchemy Chooses Flow Blockchain to Accelerate Game-Changing Developer Ecosystem                press release         PRESS RELEASE.',
     'The leading blockchain developer platform, Alchemy, will bring its best-in-class suite of APIs and tools to Flow blockchain, empowering Fortune 500 companies as well as thousands of independent developers building on Flow.',
     'Alchemy is the leading API platform for blockchain developers, already powering more than $25 billion in annual transaction volume on the Ethereum blockchain.',
     'Flow blockchain will be a milestone expansion beyond Ethereum for the San Francisco-based team.',
     'Home to leading consumer experiences like NBA Top Shot and upcoming titles such as Dr Seuss, MotoGP and UFC, Flow is witnessing unprecedented momentum in terms of developer traction, with over 1,000 developers and teams actively building on top of the network.',
     '“Alchemy’s mission is to enable blockchain developers to build the future, so the activity on NBA Top Shot and explosive growth of the overall ecosystem made the need to support Flow obvious.” said Nikil Viswanathan, CEO of Alchemy.',
     '“The demand we’ve seen from blockchain teams to build on Flow has been incredible.',
     'Flow delivers the scalability developers need, and will help usher in the next generation of massively adopted NFTs and consumer blockchain applications.”  With the Alchemy team bringing their best-in-class platform to Flow, developers will now be able to leverage one of most sophisticated scaling solutions to address an ever increasing demand from both crypto and mainstream consumers.',
     '“We’re excited to see Alchemy becoming a key contributor to Flow,” said Dapper Labs CEO Roham Gharegozlou, adding, “Having a strong team focused on creating a seamless developer experience strengthens the Flow ecosystem and provides the support needed to empower the next generation of innovators and builders.”  The combination of Flow’s highly scalable network with Alchemy’s developer centric platform will ensure that the world’s most successful enterprises and small teams alike can deliver incredible products and experiences empowered by the blockchain.',
     'About Alchemy Alchemy provides the leading blockchain development platform powering millions of users from 99% of countries worldwide.',
     'Alchemy’s mission is to empower developers with the fundamental building blocks they need to create the future of technology, and lower the barrier to entry for developers to build blockchain applications.',
     'Alchemy currently powers 60% of the top Ethereum applications and over $22 billion in on-chain transactions, and has been featured on TechCrunch, Wired, Bloomberg and numerous other media outlets.',
     'The Alchemy team draws from decades of deep expertise in massively scalable infrastructure, AI, and blockchain from leadership roles at technology pioneers like Google, Microsoft, Facebook, Stanford, and MIT.',
     'About Flow  Flow is a fast, decentralized, and developer-friendly blockchain, designed as the foundation for a new generation of entertainment apps, games and the digital assets that power them.',
     'Flow is the only layer-one blockchain originally created by a team that has consistently delivered great consumer blockchain experiences: CryptoKitties, Dapper Wallet, NBA Top Shot.',
     'Flow boasts a rich ecosystem of top entertainment brands, development studios, and venture-backed startups.',
     'Flow ecosystem partners include global IP brands like Warner Music, Ubisoft, NBA and UFC; leading game developers, including Animoca Brands, Sumo Digital and nWay; and many leaders in the crypto space such as Circle and Binance.',
     'This is a press release.',
     'Readers should do their own due diligence before taking any actions related to the promoted company or any of its affiliates or services.',
     'Bitcoin.com is not responsible, directly or indirectly, for any damage or loss caused or alleged to be caused by or in connection with the use of or reliance on any content, goods or services mentioned in the press release.',
     "Domestic and Foreign Buyers Acquired a 'Tokenized' Apartment in a Spanish City by Paying With Ethereum                                        NEWS | 53 mins ago                                 Zeitgeist Brings a Prediction Markets Ecosystem to Polkadot’s Kusama                                        ALTCOINS | 3 hours ago                             Tags in this story                   Alchemy, Flow blockchain    Image Credits: Shutterstock, Pixabay, Wiki Commons Use Bitcoin and Bitcoin Cash to play online casino games here.",
     'Bitcoin.com Exchange to List XCUR, the Token Behind Curate’s All-in-One Marketplace                press release         PRESS RELEASE.',
     'Bitcoin.com Exchange is thrilled to announce the upcoming listing of XCUR on the 24th of March 2021 at 14:00 UTC.',
     'XCUR was created by the Curate team, under founding member James Hakim.',
     'For the time being, XCUR will be listed with USDT, with BTC and ETH trading pairs coming next month.',
     'What is Curate?',
     'Curate is the worlds first all in one marketplace app utilizing blockchain.',
     'Curate provides a fairer and more transparent open marketplace by offering an industry low 4% seller commission and provides buyers with zero fees when they use XCUR, curates native token, to purchase in-app.',
     'All buyers and sellers are also rewarded in XCUR on all transactions.',
     'Why choose XCUR?',
     'XCUR is a utility and reward-based cryptographic token which can be used to purchase in-app goods, accepted as a form of in-app payment for selling or staked to collect rewards for holding.',
     'XCUR tokens serve as a multi-use utility token offering users a wide number of use cases on both the Curate DApp and outside the Curate ecosystem, as so:  Purchase items on the Curate app marketplace Rewarded to buyers and sellers on all transactions Stake on the DeFi platform and DeFi partner pools Spend on over 3M+ travel products on Travala Spend on global activities on Viator  Spend on over 7M+ holidays and flights on Expedia Spend on Curate partnerships listings via Shopping IO Many more as Curate partners and integrates XCUR with more platforms  Strong Endorsement Danish Chaudhry, CEO of Bitcoin.com Exchange, shared his views on Curate’s marketplace app: “Curate is a project which focuses a lot on putting users, and its shoppers first, where I believe merchants on the line of clothing/fashion, gaming/tech, health/beauty and of course NFT’s will benefit from utilising their app.” Chaudhry continues on by saying, “The team at Curate have provided multiple use cases behind XCUR outside of their marketplace purchases and rewards programme and we’re very eager to see where the Curate platform, and the token $XCUR goes in the future.” James Hakim, Founder & CEO at Curate, added: ““This is an exciting time for Curate as we provide our community more ways to buy and trade XCUR.',
     'The timing couldn’t have been better as our app launches at the end of the month, making now the perfect time to be a part of Curate!” \xa0 About Bitcoin.com Exchange The mission of Bitcoin.com Exchange is to empower people from all over the world to trade cryptocurrencies with ease and confidence, from first-time traders to advanced trading professionals.',
     'With high liquidity, 24/7 multilingual support and dozens of trading pairs, complemented with a high level of security, the exchange offers an attractive platform for trading any cryptocurrency.',
     'Within one year since launch, on average, the exchange has been visited by more than 500K active traders per month, and this number continues to grow as you read this sentence.',
     'About Curate Curate is the worlds first all in one marketplace app that uses blockchain technology as a payment infrastructure and rewards buyers/sellers on all transactions.',
     'The marketplace is centered around the theme of content discovery on fashion, gaming, NFT’s, and much more.',
     'XCUR is the native token of Curate which provides zero gas fees when used for purchases.',
     'For more information about Curate, visit: http://www.curate.style \xa0  This is a press release.',
     'Readers should do their own due diligence before taking any actions related to the promoted company or any of its affiliates or services.',
     'Bitcoin.com is not responsible, directly or indirectly, for any damage or loss caused or alleged to be caused by or in connection with the use of or reliance on any content, goods or services mentioned in the press release.',
     "Domestic and Foreign Buyers Acquired a 'Tokenized' Apartment in a Spanish City by Paying With Ethereum                                        NEWS | 53 mins ago                                 Zeitgeist Brings a Prediction Markets Ecosystem to Polkadot’s Kusama                                        ALTCOINS | 3 hours ago                             Tags in this story                   Bitcoin.com Exchange, Curate, XCUR    Image Credits: Shutterstock, Pixabay, Wiki Commons Purchase Bitcoin without visiting a cryptocurrency exchange.",
     'Buy BTC and BCH here.',
     'Unmarshal Raises $2.6M USD From Industry Heavyweights to Build Multi-Chain Data Indexing Protocol                press release         PRESS RELEASE.',
     'Unmarshal is proud to announce that the seed and private sale funding rounds have been successfully closed with strong investment support from Industry bigwigs.',
     'A total of $2.6m was raised in the seed and private rounds.',
     'With the fresh capital introduced, Unmarshal will focus on building an advanced network of Data Indexers that can cater to the entire DeFi Ecosystem.',
     'Unmarshal is a Multi-chain DeFi Data Network that facilitates seamless access to data on blockchains for any kind of decentralized applications.',
     'So far, they have added support for Ethereum, Binance Smart Chain, Elrond, PolkaDot, and Substrate-based projects such as StaFi and with Polygon, they are diving into the Layer-2 networks and empowering dApp developers to have easy access to on-chain data.',
     'The investors have a strong trust in the team and they believe that Unmarshal will be a game changer in the space that can build a strong and vibrant Web3.',
     'Some of the well known top investors and like minds have made contributions to the success.',
     'The Venture Capitalists who have invested: Woodstock, Blackedge, GBV, NGC Ventures, SPARK Digital, Particle XYZ, AU21, Amesten, X21, Zokyo, Altonomy, Brilliance Ventures, ExNetwork, AscendEx (Previously BitMax.io).',
     'DeFi Entrepreneurs and accredited individual investors including Ravindra Kumar & Philip Arthur Moore(Frontier), Sandeep nailwal (Polygon previously Matic Network), Aniketh Jindal (Biconomy), Danish Chaudhari (Bitcoin.com exchange), Chandresh Aharwar (Unilend Finance), Ran Neuner have vouched for the idea of building a massive Web3 revolution.',
     'As the blockchain space is evolving with addition of new chains accompanied with advanced use cases to solve inherent problems of existing chains, the DeFi ecosystem is at a constant lookout for an infrastructure layer that can help fetch them reliable and contextual data from multi-chains.',
     'Unmarshal is here to fill the gaps.',
     'It acts as a liaison sitting in between blockchains and application developers to index on-chain data in artibarty formats that can be integrated easily and effortlessly.',
     '“As the DeFi and NFT space is progressing at an unprecedented rate, data is exploding and becoming increasingly complex.',
     'At Unmarshal, we aim at simplifying the intricacies to deliver customised, rich and decoded information in many formats like APIs, Notifications, Insights and Analytics” \u200a—\u200aManohar Kolagondanahalli, CEO of Unmarshal “Team Unmarshal would like to express their gratitude to the community and investors for their incredible support in our journey of driving Web3.',
     'We will remain as a solid trusted partner and a strong value creator for our investors.',
     'We will always be indebted for the immense trust you have put in us.” Website|Telegram Chat|Telegram Announcement|Twitter|Medium \xa0  This is a press release.',
     'Readers should do their own due diligence before taking any actions related to the promoted company or any of its affiliates or services.',
     'Bitcoin.com is not responsible, directly or indirectly, for any damage or loss caused or alleged to be caused by or in connection with the use of or reliance on any content, goods or services mentioned in the press release.',
     "Domestic and Foreign Buyers Acquired a 'Tokenized' Apartment in a Spanish City by Paying With Ethereum                                        NEWS | 53 mins ago                                 Zeitgeist Brings a Prediction Markets Ecosystem to Polkadot’s Kusama                                        ALTCOINS | 3 hours ago                             Tags in this story                   Unmarshal    Image Credits: Shutterstock, Pixabay, Wiki Commons Use Bitcoin and Bitcoin Cash to play online casino games here.",
     "TimeCoin's Off-Market Token Sale                press release         PRESS RELEASE.",
     'The Special Token Sale of TimeCoin (TMCN) was launched on February 15th, 2021, with a 93% discount on the token’s current market value.',
     'The token was launched on November 11th, 2020 on BitForex as an IEO, at an initial price of $0.7.',
     'After the launch, the price rose to $11, and it has continued to rise.',
     'TMCN’s price hit a record high of $8,500 on February 18th, 2021 and that price was more than 12,142 times the initial price.',
     'Also, TimeCoin is listed on BitMart on March 24th, 2021 and will be listed on other exchanges near the future.',
     'The project, the TimeCoinProtocol, was initially financed by a few investors who invested around $4 million, and only a very limited number of tokens were placed on the market.',
     'Now with the Special Token Sale, the tokens will be sold on the OTC market.',
     'The project bought back most of the tokens available on the market so as not to increase the selling pressure.',
     'The tokens will also be made available for trading on other crypto exchanges in the future.',
     'So far, no public or private sale of TimeCoin has been conducted, but rather small quantities of tokens have been sold during the IEO.',
     'The Special Token Sale serves specifically to raise funds for the development and enhancement of the project, and in particular for the eSportStars dApp, which was released in October 2020 and is expected to grow faster than anticipated.',
     'The project requires further development, as well as the implementation of DeFi and NFT functionalities in the TimeCoinProtocol, and new funds to support development and marketing costs.',
     'For this reason, the decision has been made to launch two special, off-market token sales in order to enable the implementation of DeFi functionality in the eSportStars and TimeTicket dApps, so that fans can support creators such as eSport players, streamers, VTubers, etc.',
     'This will allow both creators and fans to receive additional tokens, as well as in-game items, art and anime characters traded using NFTs.',
     'Out of a total of 100,000,000 existing TMCN tokens, 10,000,000 tokens will be sold during the two special token sales.',
     'The first one was launched on February 15th, and will run until April 30th, 2021.',
     '5,000,000 TMCN tokens will be offered for sale at a price of 0.00004 BTC, which is more than 90% discount on the current market value.',
     'The second one will be launched on May 1st, 2021 and will last until May 31st, 2021.',
     'It will cover another 5,000,000 TMCNs at a price of 0.00009 BTC, which is more than 80% discount on the current market value.',
     'The total amount expected to be raised is approximately $10 million.',
     'It will be possible to purchase in BTC, USDT or ETH via the TimeCoinProtocol.com/sale page, where it is also possible to consult the table of the token release periods.',
     'To purchase TMCN, an investor is STRONGLY RECOMMENDED to prepare a private wallet like MetaMask.',
     '(An investor is NOT recommended to purchase TMCN using a crypto exchange account.)',
     'Subscribe to the TimeCoinProtocol Telegram to earn TMCN Free Rewards and buy TMCN.',
     'This is a press release.',
     'Readers should do their own due diligence before taking any actions related to the promoted company or any of its affiliates or services.',
     'Bitcoin.com is not responsible, directly or indirectly, for any damage or loss caused or alleged to be caused by or in connection with the use of or reliance on any content, goods or services mentioned in the press release.',
     "Domestic and Foreign Buyers Acquired a 'Tokenized' Apartment in a Spanish City by Paying With Ethereum                                        NEWS | 53 mins ago                                 Zeitgeist Brings a Prediction Markets Ecosystem to Polkadot’s Kusama                                        ALTCOINS | 3 hours ago                             Tags in this story                   TimeCoin    Image Credits: Shutterstock, Pixabay, Wiki Commons Use Bitcoin and Bitcoin Cash to play online casino games here.",
     'New FATF Guidance Applies Regulatory Standards to Decentralized Exchanges, Defi and NFTs                        On March 19, the Financial Action Task Force (FATF) published draft guidance on the risk-based approach to virtual assets.',
     'The newly updated guidance now applies anti-money laundering and know-your-customer rules to stablecoins, decentralized finance (defi), and non-fungible token (NFT) assets.',
     'FATF Defines Decentralized Exchanges and Defi as a Virtual Asset Service Providers For a while now, cryptocurrency proponents have said that one day, global regulators would likely target decentralized finance (defi) and the latest non-fungible token (NFT) hype.',
     'For a while now, the Financial Action Task Force (FATF) has been trying to come up with a regulatory standard for cryptocurrencies businesses called a “virtual asset service provider” (VASP).',
     'Things like the FATF’s Travel Rule have always been controversial, but regulators from a number of countries have been adopting the organization’s guidance.',
     'Recently Gibraltar updated its guidance notes to align with the FATF rules and South Africa has been attempting the same.',
     'The FATF’s latest guidance is merely an update of some of its past recommendations toward virtual assets (VA) and VASPs.',
     'However, the updated version now discusses stablecoins, defi, and NFTs, as things like decentralized exchanges (dex) are considered VASPs.',
     'The newly revised guidance suggests imposing anti-money laundering and know-your-customer (AML/KYC) rules toward dex applications.',
     'The FATF calls these platforms “Decentralized or distributed applications (dapp) or a platform that offers “exchange or transfer services.” “A dapp, for example, is a term that refers to a software program that operates on a P2P network of computers running a blockchain protocol— a type of distributed public ledger that allows the development of other applications,” the latest guidelines note.',
     '“These applications or platforms are often run on a distributed ledger but still usually have a central party with some measure of involvement, such as creating and launching an asset, setting parameters, holding an administrative ‘key’ or collecting fees.” The global regulator’s guidance adds: Dapps can facilitate or conduct the exchange or transfer of [virtual assets].',
     '‘The FATF Is No Stranger to Defi, Dex, and NFTs’ FATF’s latest guidance essentially defines a non-fungible token (NFT) as a VA as it defines a stablecoin.',
     'The latest guidance on NFT, defi, and stablecoins indicate that the FATF has noticed these emerging trends within the crypto space.',
     'Additionally, the FATF tweeted about the guidance and asked for commentary about the 99-page report.',
     '“The FATF wants to hear your views on draft guidance for taking a risk-based approach to virtual assets and virtual asset service providers,” the organization tweeted.',
     '“Relevant private sector stakeholders can take part in the public consultation.” The report also touched upon VA transfers to and from unhosted wallets.',
     '“The FATF recognizes that unlike traditional fiat wire transfers, not every VA transfer may involve (or be book-ended by) two obliged entities,” the FATF guidance says.',
     'In instances in which a VA transfer involves only one obliged entity on either end of the transfer (e.g., when an ordering VASP or other obliged entity sends VAs on behalf of its customer, the originator, to a beneficiary that is not a customer of a beneficiary institution but rather an individual VA user who receives the VA transfer.” Of course, the conversation about the FATF’s new definitions for defi, NFTs, stablecoins, dapps, and dex applications was a topical discussion on social media.',
     '“FATF ain’t no stranger to defi, dex & NFT,” Björn Godenrath tweeted.',
     '“Classification as a virtual asset service provider brings market participants into the scope of traditional money laundering regulations (if they can be identified),” he added.',
     'What do you think about the FATF guidelines and definitions tied to defi and NFTs?',
     'Let us know what you think about this subject in the comments section below.',
     'Microsoft President Says the World Is Best Served When Currencies Are Issued and Backed by Governments                                        REGULATION | 7 hours ago                                Indian Government May Block IP Addresses of Crypto Exchanges                                        REGULATION | 2 days ago                             Tags in this story                   "The Big Bitcoin Heist", 99-page report, AML, Bitcoin, Björn Godenrath, decentralized exchange, DeFi, DEX, fatf, FATF guidance, Financial Action Task Force, KYC, nft, Regulation, Regulatory standards, The FATF, VA, vasp, VASPs    Image Credits: Shutterstock, Pixabay, Wiki Commons Spot-markets for Bitcoin, Bitcoin Cash, Ripple, Litecoin and more.',
     'Start your trading here.',
     'Disclaimer: This article is for informational purposes only.',
     'It is not a direct offer or solicitation of an offer to buy or sell, or a recommendation or endorsement of any products, services, or companies.',
     'Bitcoin.com does not provide investment, tax, legal, or accounting advice.',
     'Neither the company nor the author is responsible, directly or indirectly, for any damage or loss caused or alleged to be caused by or in connection with the use of or reliance on any content, goods or services mentioned in this article.',
     'Read disclaimerShow comments',
     'After BTC Slid by 14%, Onchain Analyst Says a US Based Institution Is Currently Buying the Dip                        After setting a new all-time high (ATH) of $61,699 on March 13, bitcoin dropped to $53,300 on March 23, a decline of more than 14% in just ten days.',
     'However, following this latest drop, onchain analyst Willy Woo speculates that another institutional investor could be buying the dip.',
     'Strong Hands Accumulating In his tweet on March 23, Woo uses two charts to support his assertion, that the large volumes of coins that are being removed from exchanges, are in fact flowing to ” strong hands with minimal history of selling.” However, it is the involvement of Coinbase, which has facilitated large BTC acquisitions by U.S. entities in past, that has convinced Woo that a yet-to-be-announced institution is currently buying.',
     'Consequently, in his warning to holders currently selling, Woo tweeted: Anyone selling right now is cray-cray.',
     'Seriously strong long term holders are buying this dip.',
     'However, some Twitter users reacting to Woo’s post like Twitter user “Phive26,” are unconvinced with his predictions, while others have questioned the data that is used to arrive at the conclusion that an institution is currently buying.',
     'In his tweet, Phive26 said: “We have seen several times people mistaking internal transfers of crypto for withdrawals.',
     'How do you ensure not to bring internal transfers into your withdrawal data?”         Recently, an on-chain analysis firm Cryptoquant sparked controversy after it disputed Glassnode’s characterization of the movement of certain coins on the Gemini platform as an internal transfer.',
     'Not the Right Time to Sell Meanwhile, other Twitter users are in agreement with Woo’s assertion that the aggressive drop in the coins on exchanges is a signal that institutions are buying.',
     'For instance, a user named “Wages of Freedom” said: Wow, the recent drop in coins on exchanges is aggressive.',
     'Seems like the $60k milestone hit a profit target for a good chunk of investors, but a huge accumulation by convicted buyers.',
     'Similarly, another user, “Exonumia” agreed with Woo’s warning to holders.',
     'He tweeted: “Sell for what?',
     'Exchange the hardest asset on earth for dollars which are inflating this at 20% per year?',
     'That makes zero sense.',
     'There is no need to sell bitcoin.',
     'It’s the perfect collateral asset and SOV.” The price of bitcoin (BTC) jumped 3% higher after Tesla and Elon Musk revealed the electric car company will now accept BTC payments.',
     'Do you agree with Woo’s argument that an institution is currently buying the dip?',
     'You can share your thoughts in the comments section below.',
     'Fidelity Applies to Launch Bitcoin ETF Amid Accelerating Institutional Demand for BTC                                        MARKETS AND PRICES | 11 hours ago                                JMP Securities Expects $1.5 Trillion to Flow Into Bitcoin as Retail Wealth Management Industry Follows Morgan Stanley to Offer BTC to Clients                                        MARKETS AND PRICES | 1 day ago                             Tags in this story                   All time high, BTC, Coinbase, Cryptoquant, glassnode, institutional investors, long term holders, on-chain analyst, strong hands, Willy Woo    Image Credits: Shutterstock, Pixabay, Wiki Commons Spot-markets for Bitcoin, Bitcoin Cash, Ripple, Litecoin and more.',
     'Start your trading here.',
     'Disclaimer: This article is for informational purposes only.',
     'It is not a direct offer or solicitation of an offer to buy or sell, or a recommendation or endorsement of any products, services, or companies.',
     'Bitcoin.com does not provide investment, tax, legal, or accounting advice.',
     'Neither the company nor the author is responsible, directly or indirectly, for any damage or loss caused or alleged to be caused by or in connection with the use of or reliance on any content, goods or services mentioned in this article.',
     'Read disclaimerShow comments',
     'Tesla Begins Accepting Bitcoin — Elon Musk Says BTC Payments Will Not Be Converted to Fiat Currency                        Elon Musk’s electric car company, Tesla, has started accepting bitcoin payments.',
     'Musk, whose title is now Technoking of Tesla, also confirmed that all “Bitcoin paid to Tesla will be retained as bitcoin, not converted to fiat currency.” Tesla Now Accepts Bitcoin Tesla has begun accepting bitcoin payments.',
     'Elon Musk, who recently changed his title from the CEO of Tesla to the Technoking of Tesla, announced via Twitter Wednesday morning, “You can now buy a Tesla with bitcoin.” The bitcoin payment option has also been added to the Tesla website along with a page explaining how to pay with the cryptocurrency.',
     'Musk further tweeted that the “pay by bitcoin capability” will be available outside the U.S. “later this year.” In addition, the Tesla technoking confirmed, “Bitcoin paid to Tesla will be retained as bitcoin, not converted to fiat currency,” noting that his company “is using only internal & open source software & operates bitcoin notes directly.”  A page about paying with bitcoin has been added to the Tesla website, stating that “Bitcoin is the only cryptocurrency Tesla accepts.” As for how to pay with BTC, the page details: Initiate payment from your bitcoin wallet.',
     'You can either scan the QR code or copy and paste the bitcoin address and the exact bitcoin amount into your wallet.',
     'Regarding how long it takes to process a bitcoin payment, Tesla wrote: “Once you send bitcoin from your wallet, the page will refresh within one minute … We will email you a confirmation once payment is received.',
     'This can take up to six hours to process.” Tesla first revealed its plans to accept bitcoin payments in a filing with the U.S. Securities and Exchange Commission (SEC) last month, when it also informed the regulator that it had purchased bitcoin worth $1.5 billion.',
     'What do you think about Tesla accepting and retaining bitcoin?',
     'Let us know in the comments section below.',
     "Mystery Whale Moves 20 Bitcoin Block Rewards from 2010, Entity Moved 10,000 BTC Since Last Year                                        FEATURED | 2 days ago                                NFT Criticism Heightens: Skeptic Calls Tech a 'House of Cards,' Claims NFTs Will be 'Broken in a Decade'                                        FEATURED | 5 days ago                             Tags in this story                   Bitcoin, Bitcoin Payments, BTC, elon musk bitcoin, elon musk btc, elon musk cryptocurrency, tesla accepts bitcoin, tesla bitcoin, tesla bitcoin nodes, tesla btc, tesla converts bitcoin    Image Credits: Shutterstock, Pixabay, Wiki Commons Use Bitcoin and Bitcoin Cash to play online casino games here.",
     'Disclaimer: This article is for informational purposes only.',
     'It is not a direct offer or solicitation of an offer to buy or sell, or a recommendation or endorsement of any products, services, or companies.',
     'Bitcoin.com does not provide investment, tax, legal, or accounting advice.',
     'Neither the company nor the author is responsible, directly or indirectly, for any damage or loss caused or alleged to be caused by or in connection with the use of or reliance on any content, goods or services mentioned in this article.',
     'Read disclaimerShow comments',
     'Armed Criminals Who Posed as Bitcoin Buyers Stole Over $5K From a 26-Year-Old Man in Argentina                        In-person crypto robberies are becoming a global issue lately for the authorities, and this time, Argentina is the place of the latest case reported.',
     'In Mendoza, an unnamed man was assaulted by two criminals who posed as bitcoin buyers.',
     'Identities of the Robbers Are Still Unknown According to Mendo Voz, the 26-years-old man, identified by his initials as F.T., agreed to meet with the two robbers at 7:30 pm local time via telephone in a street corner of the Guaymallén neighborhood.',
     'In a first instance, F.T.',
     'believed the two criminals were legit bitcoin (BTC) buyers who wished to buy his cryptos in exchange for Argentinean pesos.',
     'There are no reports on how many exact bitcoins the men agreed to acquire.',
     'However, the situation became suspicious, as the alleged BTC buyers never arrived.',
     'Afterward, two robbers appeared on the scene by threatening him with a firearm.',
     'Per the local newspaper Los Andes, the individuals are suspected to be the same people who posed as crypto buyers.',
     'Criminals managed to steal him an iPhone, $2,500 and 120,000 Argentinean pesos ($3,810).',
     'F.T.',
     'wasn’t injured during the incident, but the robbers fled from the scene in a van.',
     'Residents of the neighborhood helped the victim by calling the police.',
     'Per the local media, the stolen iPhone was found by the authorities on a highway.',
     'The Attorney Office Nº19 of the 35th Argentinean Police Station is now pursuing the correspondent investigations.',
     'However, there are no arrests made, nor hints on the identities of the robbers as of press time.',
     'Hong Kong Is Living Its Own Drama With Crypto-Related Robberies As the year kicked off, in Hong Kong, a 37-year-old man also was a victim of a theft from a gang of robbers who fled with 15 bitcoin (BTC), worth HKD 3 million ($387,000) in cash at that time.',
     'They had also agreed to meet in person for a crypto exchange transaction.',
     'Also, in the same country, Bitcoin.com News reported another robbery that involved an offline crypto transaction.',
     'A female trader was a victim of robbers who stole her HKD 3.5 million ($451,000) worth in tether (USDT) tokens.',
     'What do you think about this case of “crypto robbery” in Argentina?',
     'Let us know in the comments section below.',
     "Domestic and Foreign Buyers Acquired a 'Tokenized' Apartment in a Spanish City by Paying With Ethereum                                        NEWS | 53 mins ago                                 Iconic 'Obey' Street Artist Shepard Fairey to Auction an NFT Mural on Superrare                                        NEWS | 5 hours ago                             Tags in this story                   Argentina, argentina bitcoin, Bitcoin Robbery, BTC transactions, Criminal, Latin America, Police, robbery    Image Credits: Shutterstock, Pixabay, Wiki Commons Spot-markets for Bitcoin, Bitcoin Cash, Ripple, Litecoin and more.",
     'Start your trading here.',
     'Disclaimer: This article is for informational purposes only.',
     'It is not a direct offer or solicitation of an offer to buy or sell, or a recommendation or endorsement of any products, services, or companies.',
     'Bitcoin.com does not provide investment, tax, legal, or accounting advice.',
     'Neither the company nor the author is responsible, directly or indirectly, for any damage or loss caused or alleged to be caused by or in connection with the use of or reliance on any content, goods or services mentioned in this article.',
     'Read disclaimerShow comments',
     'Tipb.ch App Allows You to Share a Short URL Rather Than a BCH Address on Social Media                        On March 23, the creator of the blog platform read.cash and the social media application noise.cash revealed a new project called tipb.ch.',
     'The application allows anyone to create a customizable short address that can be used for tips rather than having to share a long BCH address on social media platforms.',
     'Platform Tipb.ch Allows You to Share a Short URL Rather Than a Long Bitcoin Cash Address The founder of the blogging platform read.cash announced a new project on Reddit this Tuesday revealing a service called tipb.ch (https://tipb.ch).',
     'The Reddit post led to another post on the social media application noise.cash which described what tipb.ch does.',
     'Essentially, the program allows anyone to create a short URL and tether it to a public bitcoin cash (BCH) address.',
     'This way, instead of having to share a long BCH to others on social media, the person can create a short address to post on Instagram, Twitter, Noise.cash, Facebook, or Read.cash.',
     'Rather than leveraging a long BCH address, the announcement post says: Wouldn’t it be much better to post something like tipb.ch/readcash?',
     'That’s exactly what tipb.ch does.',
     'You can register a name and use it.',
     'You can actually register up to 5 names and addresses.',
     '(In case of an obvious impersonation, like if you register a name like tipb.ch/elonmusk and you aren’t provably Elon Musk, we reserve the right to delete your address and return it to the rightful owner).',
     'The platform takes only a few minutes to set up a short address to a public BCH address.',
     'On social media, the announcement was welcomed with open arms.',
     'The post on the subreddit r/btc had a lot of people using Chaintip to tip the creator of the new tipb.ch application.',
     '“This is awesome,” one individual wrote on Reddit.',
     '“And because I got in from the beginning, I got to choose my first name as my username,” the Redditor added.',
     'Taking the Tipping Further With a Twitter Faucet In addition to the tipb.ch application, the creators wanted to take things a bit further.',
     '“What if you could post on Twitter using a hashtag #bitcoincash and your tipb.ch address and maybe get a bitcoin cash tip automatically for doing just that from us?” the creators asked.',
     '“It will help bitcoin cash gain in popularity and we could give bitcoin cash to even more people to try it out.',
     'That’s exactly what we plan to do with our newest tipb.ch bitcoin cash faucet for Twitter,” the tipb.ch creators noted.',
     'An example of a tipb.ch address the Read.cash creators have made.',
     'Users can tether up to five different bitcoin cash addresses to a unique short URL.',
     'Currently, the faucet doesn’t work yet, but the faucet should be functional within a week the developers added.',
     'The team still needs to debug the software and people can start posting on Twitter now using the hashtag #bitcoincash and their tipb.ch address.',
     'If there are posts on Twitter that match those requirements (#bitcoincash + tipb.ch address), it will help the programmers develop things faster.',
     'What do you think about the tipb.ch platform and being able to share a short URL rather than a long bitcoin cash address?',
     'Let us know what you think about this subject in the comments below.',
     'Jigstack DAO Acquires Icorating.com Platform to Strengthen Its Token Launch Pad Lemonade                                        SHARING ECONOMY | Feb 23, 2021                                Noise.cash: Social Microblogging App Fueled by Bitcoin Cash Tips Gathers Traction                                        SHARING ECONOMY | Jan 18, 2021                             Tags in this story                   #BitcoinCash, BCH, BCH tips, bitcoin cash, bitcoin cash BCH, Chaintip, Crypto, crypto assets, Crypto Tipping, faucet, Noise.cash, Noise.cash Social Media, read.cash, read.cash blog, Sharing Economy, Social Media, tipping, tips, Twitter    Image Credits: Shutterstock, Pixabay, Wiki Commons, tipb.ch Purchase Bitcoin without visiting a cryptocurrency exchange.',
     'Buy BTC and BCH here.',
     'Disclaimer: This article is for informational purposes only.',
     'It is not a direct offer or solicitation of an offer to buy or sell, or a recommendation or endorsement of any products, services, or companies.',
     'Bitcoin.com does not provide investment, tax, legal, or accounting advice.',
     'Neither the company nor the author is responsible, directly or indirectly, for any damage or loss caused or alleged to be caused by or in connection with the use of or reliance on any content, goods or services mentioned in this article.',
     'Read disclaimerShow comments',
     "Digital Yuan Won't Be Fully Anonymous but Offers 'Controllable' Privacy Protection, Says Official                        Privacy has been a discussion that has not gone unnoticed when it comes to the forthcoming launch of the digital yuan.",
     'The head of the People’s Bank of China-run Digital Currency Research Institute had firmly pointed out that anonymity cannot be guaranteed at all.',
     'Feature Seeks to Avoid Tax Evasion and Finance Terrorism During an interview with the Securities Times, Ma Changchun told the 2021 China Development Forum that a fully anonymous central bank digital currency (CBDC) is impossible.',
     'The reason, said the official, is that the government needs to make it compliant with the anti-money laundering (AML), know-your-customer (KYC), and counter-terrorist financing (CTF) measures.',
     'Per the official, such transactions should also be scrutinized to avoid tax evasion schemes through the upcoming Chinese CBDC, developed by the PBoC.',
     'However, Ma clarified that there would be a “controllable anonymization” technology implemented in the digital yuan.',
     'Although he didn’t explain in-depth the “controllable anonymization” concept, Ma pointed out that such anonymity is the one that it’s commonly used for privacy consumer protection purposes.',
     'The Role of the Telecom Providers Still, he explained how the digital yuan wallet could work out with KYC measures implemented: The wallet with the weakest KYC strength is an anonymous wallet, which can be opened with only a mobile phone number.',
     'Of course, this type of wallet has the lowest balance and daily transaction limit, which can only meet the daily needs of small payments.',
     'If you want to make large payments, the wallet needs to be upgraded, and the wallet balance and payment limit will increase as the KYC intensity increases.',
     'The head of the Digital Currency Research Institute also clarified which would be the role of the domestic telecommunication providers in the digital yuan’s development process: Although the payment departments of telecom operators are also involved in the research and development of digital renminbi, according to the current national laws and regulations, telecom operators are not allowed to disclose mobile phone customer information to third parties such as the central bank.',
     'Of course, they are not allowed to provide it to the departments that operate digital renminbi.',
     'Therefore, wallets opened with mobile phone numbers are completely anonymous to the People’s Bank of China and various operating agencies.',
     'What do you think about the “controllable anonymization” mentioned by the Chinese official?',
     'Let us know in the comments section below.',
     "Snowden Puzzled by Bitcoin's Lack of Scaling and Privacy, Says Devs 'Had Years to Do It'                                        PRIVACY | Dec 25, 2020                                Study: Over 13% of All Proceeds of Crimes in Bitcoin Passed Through Privacy Wallets in 2020                                        PRIVACY | Dec 10, 2020                             Tags in this story                   CBDC, china digital currency, chinese digital currency, Digital Currency, Digital Yuan, PBOC, Privacy, wallet privacy    Image Credits: Shutterstock, Pixabay, Wiki Commons Purchase Bitcoin without visiting a cryptocurrency exchange.",
     'Buy BTC and BCH here.',
     'Disclaimer: This article is for informational purposes only.',
     'It is not a direct offer or solicitation of an offer to buy or sell, or a recommendation or endorsement of any products, services, or companies.',
     'Bitcoin.com does not provide investment, tax, legal, or accounting advice.',
     'Neither the company nor the author is responsible, directly or indirectly, for any damage or loss caused or alleged to be caused by or in connection with the use of or reliance on any content, goods or services mentioned in this article.',
     'Read disclaimerShow comments',
     "Crypto Exchange FTX on the Verge of Winning Naming Rights for the Miami Heat's Arena                        Samuel Bankman-Fried’s FTX crypto exchange is reportedly one vote away from winning a naming-rights deal for the arena of the NBA’s Miami Heat.",
     'The deal, which is believed to be worth $135 million, is set to be approved on March 26 by the Miami-Dade County Commission.',
     'Miami-Dade to Get $5 Million a Year According to a report, as part of this agreement, the venue, which has been known as the “American Airlines Arena” for two decades, will be renamed the “FTX Arena.” In return, Miami-Dade, which controls the naming rights for the building, will receive “an undisclosed amount of yearly payments from FTX.”  However, as part of the deal, Miami-Dade is expected to “pay the (Miami) Heat $2 million a year under an option the county exercised in 2018 to find the arena’s next sponsor.” While officials have not yet disclosed the net revenue to accrue to Miami-Dade, journalist Joe Pompliano tweeted the city will realize a net gain of $5 million from the deal.',
     'FTX Going Mainstream Meanwhile, the report also explains that this deal would present the exchange with the opportunity to showcase cryptocurrencies to a mainstream audience.',
     'The report said: If approved, the agreement would bring the NBA its first arena sponsor for the crypto industry, and inject FTX into the Miami sports vocabulary at a time when the city’s mayor is courting tech executives and embracing bitcoin.',
     'In early February of 2021, Bitcoin.com’s newsdesk reported that Miami city commissioners had voted to support mayor Francis Suarez’s proposal that enabled the city to hold bitcoin in the treasury.',
     'The commissioners similarly endorsed the proposal for the city to accept the payments of taxes and salaries in crypto form.',
     'When approved, FTX’s deal with Miami-Dade will represent another significant milestone for the pro-bitcoin mayor.',
     'On the other hand, the Miami-Dade and FTX deal will represent yet another mainstream foray by Bankman-Fried.',
     'Before revelations of FTX’s agreement with Miami-Dade, the 29-year-old billionaire had emerged as the second biggest donor to U.S. President Joe Biden’s election campaign.',
     'What are your thoughts on FTX’s imminent sponsorship deal with Miami-Dade?',
     'You can tell us what you think in the comments section below.',
     'Decentralized Exchange Uniswap Reveals Protocol Version 3 With New Automated Elements                                        EXCHANGES | 13 hours ago                                South African Exchange\xa0Ice3x Suspends Trading After Noting Discrepancies in Balances of BTC and LTC                                        EXCHANGES | Mar 18, 2021                             Tags in this story                   bitcoin treasury, Cryptocurrencies, FTX CEO, FTX Exchange, Mayor Francis Suarez, Miami Heat, pro bitcoin, Sam Bankman-Fried, U.S. President Joe Biden    Image Credits: Shutterstock, Pixabay, Wiki Commons Purchase Bitcoin without visiting a cryptocurrency exchange.',
     'Buy BTC and BCH here.',
     'Disclaimer: This article is for informational purposes only.',
     'It is not a direct offer or solicitation of an offer to buy or sell, or a recommendation or endorsement of any products, services, or companies.',
     'Bitcoin.com does not provide investment, tax, legal, or accounting advice.',
     'Neither the company nor the author is responsible, directly or indirectly, for any damage or loss caused or alleged to be caused by or in connection with the use of or reliance on any content, goods or services mentioned in this article.',
     'Read disclaimerShow comments',
     "Hut 8 Joins Foundry's US Mining Pool, Adds Over 14,000 Bitcoin Mining Rigs of Hashpower                        On Tuesday, the firm Hut 8 Mining Corp., announced that it is joining the Foundry U.S.-based bitcoin mining pool.",
     'The publicly-listed mining company said that it added 14,400 machines to Foundry’s mining operation and it also plans to deploy another 5,000 miners by August.',
     'Hut 8 Joins Foundry’s US Pool and Adds 14,400 Bitcoin Mining Rigs With Close to an Exahash of Hashpower The Toronto-based company Hut 8 (TSX: HUT) has revealed the firm is joining the U.S. mining pool invoked by the firm Foundry Digital LLC.',
     'Foundry is a wholly-owned subsidiary of Digital Currency Group (DCG) and the firm is dedicated to cryptocurrency mining and staking services.',
     'The announcement explains that Hut 8 is now mining on the Foundry USA Pool and a fraction of the Hut 8 fleet of miners are being deployed to Foundry’s pool.',
     'Around 0.81 exahashes of hashpower or 14,400 bitcoin mining rigs will be added.',
     '“We are focused on growing shareholder value, in every decision we make,” said Jaime Leverton, CEO of Hut 8 during the announcement.',
     '“Having a formidable bitcoin mining pool based and operated entirely in North America is important to us, as we continue our momentum as one of the largest and most innovative Bitcoin miners in the western hemisphere,” Leverton added.',
     'Hut 8 Aims to Deploy an Additional 1.20 Exahashes of Hashpower by the End of the Summer Hut 8 plans to deploy another 5,000 mining rigs by August 2021 and the press announcement sent to Bitcoin.com highlights that Foundry’s mining pool “serves as a strong U.S.-based alternative to the China-dominated pool industry.” Mike Colyer, CEO of Foundry said that Hut 8’s team is also a client of the firm’s equipment financing business.',
     '“We are pleased to welcome them to our pool as we work towards securing North America’s place among the world’s top-5 bitcoin mining pools,” Colyer stressed.',
     'By August 2021 when the additional 5,000 miners are tethered to Foundry’s U.S. pool, it will amount to roughly 1.20 exahashes of hashpower.',
     'Foundry says that the pool offers “Miner Treasury Management Services,” which consists of services tethered to the DCG-subsidiary Genesis.',
     '“Among those services is an option to generate yield in bitcoin (BTC) on the mined BTC, which allows clients to maximize the overall BTC yield of their operations,” the press release concludes.',
     '“Other Genesis-powered products include high-security custody, BTC collateralized lending, derivative products, and seamless liquidation.” What do you think about Hut 8 joining Foundry’s U.S. bitcoin mining pool?',
     'Let us know what you think about this subject in the comments section below.',
     "500 MW by 2025: Bitcoin Miner Greenidge and It's Wholly-Owned Power Plant to Be Listed on Nasdaq                                        MINING | 3 days ago                                AMD Won't Restrict Crypto Miners From Using Its Graphic Cards                                        MINING | 3 days ago                             Tags in this story                   14400 machines, 5000 machines, Bitcoin (BTC), Bitcoin mining, BTC, BTC hashpower, BTC Mining, DCG, Digital Currency Group, Exahash, Foundry, Foundry US Pool, Hut 8, Hut 8 bitcoin mining, mining, Mining Devices, Mining Operations, mining pool, mining rigs, US Pool    Image Credits: Shutterstock, Pixabay, Wiki Commons Purchase Bitcoin without visiting a cryptocurrency exchange.",
     'Buy BTC and BCH here.',
     'Disclaimer: This article is for informational purposes only.',
     'It is not a direct offer or solicitation of an offer to buy or sell, or a recommendation or endorsement of any products, services, or companies.',
     'Bitcoin.com does not provide investment, tax, legal, or accounting advice.',
     'Neither the company nor the author is responsible, directly or indirectly, for any damage or loss caused or alleged to be caused by or in connection with the use of or reliance on any content, goods or services mentioned in this article.',
     'Read disclaimerShow comments',
     'Mystery Whale Moves 20 Bitcoin Block Rewards from 2010, Entity Moved 10,000 BTC Since Last Year                        On Tuesday, March 23, the infamous whale that’s been spending numerous strings of 2010 block rewards since last year, has spent another 1,000 bitcoin that sat idle for over a decade.',
     'The string of 20 block rewards transferred on Tuesday, follows the exact same patterns our newsdesk has tracked since March 12, 2020, otherwise known as ‘Black Thursday.’ Mystery Whale Spends Another 20 Block Rewards from 2010 On February 28, 2021, the mysterious whale who has been spending a great number of 2010 block rewards for roughly a year, had once again spent another 20 blocks.',
     '23 days later, on March 23, 2021, the pattern repeated, as another 20 block rewards from 2010 were transferred today.',
     'Since Bitcoin.com’s newsdesk started investigating this old school whale on March 12, 2020, to-date the entity has spent 10,000 BTC worth over $550 million using today’s BTC exchange rates.',
     'Bitcoin.com’s newsdesk caught the mystery whale again on March 23, 2021, with the help from researchers at Btcparser.com after the entity moved another 20 block rewards with 1,000 BTC.',
     'Two days prior, a single block reward from 2010 was spent at block height 675,652.',
     'Today, the spending pattern followed the exact same pattern as the previous strings of blocks spent from 2010.',
     'All 20 blocks from 2010 were coinbase rewards, which also means each block reward contains 50 BTC.',
     'All 20 block rewards moved by the entity were also processed at block height 675,887.',
     'Per usual and similar to previous patterns, the string today had mined blocks stemming from July 2010 up until November 2010.',
     'All the strings of 20 block rewards caught since March 12, 2020, were mined within these months.',
     'Tuesday’s transfer of 1,000 bitcoin mined in 2010 was caught by the onchain blockchain parser Btcparser.com.',
     'With close to 700,000 BTC blocks processed so far, the web portal’s parsers monitor the activity of so-called ‘sleeping bitcoin‘ addresses.',
     'Flex or Force?',
     '10,000 Bitcoin Transferred So Far, All Strings Have Followed the Exact Same Spending Pattern Just like the past spends, the corresponding bitcoin cash (BCH) block rewards have also been moved.',
     'However, the corresponding bitcoinsv (BSV) coinbase rewards remain untouched like the whale’s previous string movements.',
     'Once again, the whale took all of the coins (1,000 BTC in total) and consolidated them into a single address.',
     'The first consolidation scheme is done with hardly any privacy tactics and the coins seem to be swept.',
     'After the consolidation, the coins are then moved again and split into fractions, and with far more privacy than the original sweep.',
     'It is still hard to say why this whale has been spending this way and our research team assumes there are two possible reasons.',
     'One reason may be the whale may be forced to use some sort of Script-based spending program that forces the user to move coins in this manner.',
     'Otherwise, the whale is flexing wealth with these transfers to show the public his/her movements at certain times.',
     'Why the individual would do this on purpose is unknown and it may never be known why these bitcoins are being moved in this fashion.',
     'Out of the 10,000 BTC spent in over a year’s time, the whale has moved strings of 20 decade-old bitcoin block rewards on March 12, 2020, October 11, 2020, November 7, 2020, November 8, 2020, December 27, 2020, January 3, 2021 (Bitcoin’s 12th anniversary), January 10, 2021, January 25, 2021, February 28, 2021, and today.',
     'What do you think about the string of sleeping bitcoins from 2010 that was transferred today?',
     'Let us know what you think about this subject in the comments section below.',
     "Tesla Begins Accepting Bitcoin — Elon Musk Says BTC Payments Will Not Be Converted to Fiat Currency                                        FEATURED | 2 days ago                                NFT Criticism Heightens: Skeptic Calls Tech a 'House of Cards,' Claims NFTs Will be 'Broken in a Decade'                                        FEATURED | 5 days ago                             Tags in this story                   $10000 BTC, 200 block rewards, 2010 Block Reward, 2010 Mined Coins, Bitcoin, bitcoin cash, bitcoinsv, block rewards, BTC, Btcparser.com, Cryptocurrency, Mined Coins, moved coins, parser, Satoshi Nakamoto, Satoshi-Era coins, sleeping bitcoin, spent, whale, whale miner    Image Credits: Shutterstock, Pixabay, Wiki Commons, Btcparser.com, Bitcoin.com, Holyroger.com,  Use Bitcoin and Bitcoin Cash to play online casino games here.",
     'Disclaimer: This article is for informational purposes only.',
     'It is not a direct offer or solicitation of an offer to buy or sell, or a recommendation or endorsement of any products, services, or companies.',
     'Bitcoin.com does not provide investment, tax, legal, or accounting advice.',
     'Neither the company nor the author is responsible, directly or indirectly, for any damage or loss caused or alleged to be caused by or in connection with the use of or reliance on any content, goods or services mentioned in this article.',
     'Read disclaimerShow comments',
     'Mad Money’s Jim Cramer Says Gold Let Him Down, Put 5% in Bitcoin                        Mad Money host Jim Cramer, a long-term gold proponent, says that gold has let him down.',
     'He has become more bullish on bitcoin, advising investors to put 5% of their portfolios in the cryptocurrency to protect their assets.',
     'Jim Cramer Reduces Gold Holdings, Increases Bitcoin Allocation In an interview published Monday with Morgan Creek Digital partner Anthony Pompliano, Jim Cramer talked about his new investment strategy for bitcoin and gold.',
     'Cramer is the host of Mad Money on CNBC, a former hedge fund manager, and a co-founder of Thestreet.com, a financial news and literacy website.',
     'Emphasizing that bitcoin is what he had been searching for with gold, Cramer opined: I have, for years, said that you should have gold … but gold let me down.',
     'Gold is subject to too many vicissitudes.',
     'It’s subject to mining issues.',
     'It’s frankly subject to failing in many cases.',
     'The Mad Money host became interested in bitcoin after Pompliano explained the benefits of the cryptocurrency to him in September last year.',
     'At the time, Cramer revealed that he owned “so much gold.” He became fixated with BTC in fear of massive inflation and he subsequently invested in the crypto.',
     'He explained that his investment strategy was to regard “bitcoin as a precious stock.”         Cramer was then asked if he expects other investors will dump gold and buy bitcoin in the way he described.',
     'The Mad Money host exclaimed: If they listen to me, they’re going to drop half their gold.',
     'I’ve been saying 10% in gold since 1983.',
     'And now I say 5% in gold, 5% in bitcoin.',
     'He clarified that he did not think about investing 5% of his portfolio in bitcoin until very recently, claiming that he has “thoroughly researched” the subject to come up with this recommendation.',
     'He admitted that he was skeptical because he had done fine with his gold but now realized that the only way to protect his assets, in terms of insurance, is with bitcoin.',
     'Would you take Jim Cramer’s advice about bitcoin?',
     'Let us know in the comments section below.',
     'Fidelity Applies to Launch Bitcoin ETF Amid Accelerating Institutional Demand for BTC                                        MARKETS AND PRICES | 11 hours ago                                JMP Securities Expects $1.5 Trillion to Flow Into Bitcoin as Retail Wealth Management Industry Follows Morgan Stanley to Offer BTC to Clients                                        MARKETS AND PRICES | 1 day ago                             Tags in this story                   bitcoin allocation, boom thesis, boomer, gold, Jim Cramer, jim cramer advice, jim cramer bitcoin, jim cramer crypto, jim cramer cryptocurrency, mad money, mad money bitcoin, mad money crypto    Image Credits: Shutterstock, Pixabay, Wiki Commons Purchase Bitcoin without visiting a cryptocurrency exchange.',
     'Buy BTC and BCH here.',
     'Disclaimer: This article is for informational purposes only.',
     'It is not a direct offer or solicitation of an offer to buy or sell, or a recommendation or endorsement of any products, services, or companies.',
     'Bitcoin.com does not provide investment, tax, legal, or accounting advice.',
     'Neither the company nor the author is responsible, directly or indirectly, for any damage or loss caused or alleged to be caused by or in connection with the use of or reliance on any content, goods or services mentioned in this article.',
     'Read disclaimerShow comments',
     'American News Magazine Time to Accept Crypto for Subscriptions, Firm Auctions 3 NFT Covers                        The American news magazine Time has revealed it is getting into the cryptocurrency and blockchain space, as a new job listing shows the 98-year-old publication is in search of a chief financial officer (CFO) who understands bitcoin.',
     'Additionally, Time will be selling non-fungible token (NFT) assets via Superrare and in 30 days, the news outlet will be accepting crypto assets for digital subscription payments.',
     'American News Company Time Is Looking for a CFO That Understands Bitcoin A job listing posted to Linkedin shows that Time magazine, the American publishing company, plans to hire a new CFO who understands bitcoin and blockchain technology.',
     '“The media industry is undergoing a rapid evolution,” the job advert notes.',
     '“Time is seeking a Chief Financial Officer who can help guide its transformation.” The job offer also states that the interested party must have “comfort with bitcoin and cryptocurrencies” according to the listing’s qualifications section.',
     'Additionally, the individual must have over seven years in executive leadership positions.',
     'Time Sells NFTs, Magazine Will Accept Digital Currencies for Digital Subscriptions Soon In addition to looking for a CFO who understands bitcoin, Time is also auctioning off non-fungible token (NFT) assets based off of iconic cover designs.',
     'The publisher is auctioning the NFT called “Is Fiat Dead?” on Superrare with two other NFT covers.',
     'Currently, the highest bid for the “Is Fiat Dead?” NFT is 27.5 ETH or roughly $46k.',
     'The Time magazine special edition NFT is a repurposed cover from a cover published on April 8, 1966, called “Is God Dead?” “The typography on the two most recent covers needed to be hand-drawn,” Time explains on the Superrare auction website.',
     '“As we could find no modern type foundry which has an exact interpretation of the one used on the original cover.” Lastly, in another move from Time magazine, the company’s President Keith Grossman tweeted that Time will be accepting crypto assets for payments in the next 30 days.',
     'Collectibles will always be important but Time’s crypto ambitions are larger as we continue our digital transformation,” Grossman tweeted.',
     '“Within 30-days, Time will accept cryptocurrencies for digital subscriptions with more to come via Julia LaRoche,” Grossman added.',
     'What do you think about Time magazine getting into cryptocurrencies and blockchain technology?',
     'Let us know what you think about this subject in the comments section below.',
     "Domestic and Foreign Buyers Acquired a 'Tokenized' Apartment in a Spanish City by Paying With Ethereum                                        NEWS | 53 mins ago                                 Iconic 'Obey' Street Artist Shepard Fairey to Auction an NFT Mural on Superrare                                        NEWS | 5 hours ago                             Tags in this story                   Bitcoin, Bitcoin (BTC), crypto assets, Digital Currencies, digital subscriptions, Julia LaRoche, Keith Grossman, nft, Non-fungible artwork, Non-fungible Token, Time, time magazine    Image Credits: Shutterstock, Pixabay, Wiki Commons Purchase Bitcoin without visiting a cryptocurrency exchange.",
     'Buy BTC and BCH here.',
     'Disclaimer: This article is for informational purposes only.',
     'It is not a direct offer or solicitation of an offer to buy or sell, or a recommendation or endorsement of any products, services, or companies.',
     'Bitcoin.com does not provide investment, tax, legal, or accounting advice.',
     'Neither the company nor the author is responsible, directly or indirectly, for any damage or loss caused or alleged to be caused by or in connection with the use of or reliance on any content, goods or services mentioned in this article.',
     'Read disclaimerShow comments',
     'Indian Government May Block IP Addresses of Crypto Exchanges                        As the Indian government prepares to introduce the cryptocurrency bill, there are reports that it is considering blocking the IP addresses of cryptocurrency exchanges.',
     'Despite ban reports, many people in the Indian crypto community believe that the government will not impose an outright ban on cryptocurrencies, including bitcoin.',
     'Blocking IP Addresses of Crypto Trading Platforms The Indian government is reportedly “considering the blocking of internet protocol (IP) addresses of companies/exchanges on which trading in cryptocurrencies is happening,” Business Standard publication reported Monday, citing an unnamed “source in the know.” This measure, if implemented, will be part of the government’s efforts to impose a ban on cryptocurrencies.',
     'Indian Finance Minister Nirmala Sitharaman recently said that the cabinet note regarding cryptocurrency legislation is being finalized and will be soon submitted to the cabinet.',
     'A bill entitled “The Cryptocurrency and Regulation of Official Digital Currency Bill 2021” is listed for consideration in the current parliamentary session.',
     'The Indian government has attempted to block the IP addresses of adult sites and hundreds of Chinese applications before, the publication conveyed, noting that people were still able to access the sites through virtual private networks (VPNs).',
     'An industry expert was quoted by local media as saying: Various routes — like VPNs, peer-to-peer trading, using cash to buy/sell cryptocurrencies and use wallets outside India to store and transfer cryptos, using part of the money permitted to send abroad for investment within the liberalised remittance scheme limit of $250,000 can be diverted for buying cryptocurrencies — remain loopholes.',
     'Earlier this month, the Indian finance minister said that “There will be a very calibrated position taken” regarding cryptocurrencies.',
     'She subsequently emphasized that the government is “not shutting all options off.” Her statements have given the Indian crypto community hope that India will regulate the crypto industry instead of imposing an outright ban.',
     'Do you think the Indian government will block the IP addresses of crypto exchanges?',
     'Let us know in the comments section below.',
     'Microsoft President Says the World Is Best Served When Currencies Are Issued and Backed by Governments                                        REGULATION | 7 hours ago                                New FATF Guidance Applies Regulatory Standards to Decentralized Exchanges, Defi and NFTs                                        REGULATION | 1 day ago                             Tags in this story                   Bitcoin ban, block ip addresses, crypto ban, crypto exchanges, Crypto regulation, Cryptocurrency regulation, india ban bitcoin, india bans crypto, indian government, VPN    Image Credits: Shutterstock, Pixabay, Wiki Commons Use Bitcoin and Bitcoin Cash to play online casino games here.',
     'Disclaimer: This article is for informational purposes only.',
     'It is not a direct offer or solicitation of an offer to buy or sell, or a recommendation or endorsement of any products, services, or companies.',
     'Bitcoin.com does not provide investment, tax, legal, or accounting advice.',
     'Neither the company nor the author is responsible, directly or indirectly, for any damage or loss caused or alleged to be caused by or in connection with the use of or reliance on any content, goods or services mentioned in this article.',
     'Read disclaimerShow comments',
     'Dvision Network Set to Release Limited-Edition NFT Characters                sponsored         The NFT sector continues to grow, and it is no surprise that Dvision Network is looking to release NFTs.',
     'The Virtual Reality blockchain platform offers users the opportunity to purchase virtual space within its ecosystem, and NFTs are a significant part of this.',
     'Limited-Edition NFT Characters Dvision Network, a virtual reality blockchain ecosystem, has announced that it is set to pre-release the designs of five limited-edition NFTs.',
     'This information was revealed on March 15 and signaled the first major foray into the Non-fungible tokens sector by Dvision Network.',
     'According to the report, the five designs are from the limited-edition NFT originally scheduled to be available before the end of March.',
     'The NFT characters are built based on the avatar of major stakeholders within the cryptocurrency industry and will be auctioned in the coming days on NFT Marketplace OpenSea.',
     'The NFTs released by Dvision Network are unique and scarce since they are obtained from the limited edition collection.',
     'Investors that acquire one of these five NFT characters will be eligible to acquire additional NFT assets from the limited edition series.',
     'Also, the NFT characters will be usable within the Dvision Metaverse in the future and can also be traded on the primary and secondary markets.',
     'Dvision Network will also incentivize those who purchase the limited edition NFTs in the form of DVI tokens.',
     'Eom, Jung-hyun, CEO of Dvision Network echoes similar sentiments.',
     '“We intend to supply these sort of unique NFT items to the prospective Dvision Network users in order add the fun elements to their gamified metaverse ecosystem experience, and to let them play joyfully, surfing across the virtual reality and open world via the usage of the celebrity characters,” he said in a statement.',
     'Bithumb Set to List DVI/KRW Trading Pairs Bithumb, the largest exchange in South Korea, is set to list DVI/KRW trading pairs.',
     'This comes after the exchange listed the DVI/BTC trading pairs on its platform.',
     'Dvision Network took part in the Bithumb’s native competition, the “BTC Open“, where the winner would be guaranteed listing on the KRW market within three months after the contest ends.',
     'Dvision Network led the competition in the third round of the “BTC Open” and is guaranteed the listing into the KRW market in three months.',
     'This is a significant development for DVI token holders as the listing will enable Koreans the opportunity to trade their native KRW directly to DVI, increasing liquidity for the token.',
     'Dvision Network earlier in January secured the listing of DVI on the popular custodial platform “Coinbase Custody.”  The Dvision Network continues to develop its ecosystem and is set to conduct the DVI conference in Q3 2021.',
     'About DVision Network Dvision Network is a blockchain-based VR content ecosystem.',
     'Dvision Network presents a new virtual reality world and humanity that can lead a prosperous life at the center of the 4th Industrial Revolution, which refers to a new industrial era based on information and communication technology (ICT).',
     'Precisely, the Dvision Network consists of three major products.',
     'The VR-Market is essentially an NFT marketplace where users can create, develop and transact NFT items.',
     'VR-Space, where gamers can acquire virtual spaces and sell for-profits, and the VR-City, serves as an open VR world where gamers can interact across multiple virtual spaces.',
     'These products are powered by Dvision Network native token DVI, which is listed on Uniswap, Bithumb, Coinone, and Hoo.',
     'This is a sponsored post.',
     'Learn how to reach our audience here.',
     'Read disclaimer below.',
     'SmartKey Shows There Is Real Value and Utility in NFT Tokens                                        SPONSORED | 7 days ago                                A Ledger Hardware Wallet Is the Best Solution to Protect and Own Your Private Keys                                        SPONSORED | Mar 17, 2021                             Tags in this story                   Dvision Network, nft    Image Credits: Shutterstock, Pixabay, Wiki Commons Spot-markets for Bitcoin, Bitcoin Cash, Ripple, Litecoin and more.',
     'Start your trading here.',
     'Disclaimer: This article is for informational purposes only.',
     'It is not a direct offer or solicitation of an offer to buy or sell, or a recommendation or endorsement of any products, services, or companies.',
     'Bitcoin.com does not provide investment, tax, legal, or accounting advice.',
     'Neither the company nor the author is responsible, directly or indirectly, for any damage or loss caused or alleged to be caused by or in connection with the use of or reliance on any content, goods or services mentioned in this article.',
     'Read disclaimer',
     'The Turkish Lira Plunges Following Removal of a Central Bank Governor, BTC Searches up                        The Turkish lira currency plunged by more than 10% on March 22 after the country’s President Recep Tayyip Erdogan unexpectedly fired Naci Agbal, the country’s central bank governor.',
     'Having initially touched a new low of 8.280 a dollar, the lira recovered to close at 7.75 a dollar by 20.00 hours GMT.',
     'Inflation Fears On the other hand, a day before the lira crashed, bitcoin became the most searched word in Turkey.',
     'As Google trends data shows, searches for bitcoin spiked by more than 500% as inflation fears grew following Agbal’s ouster.',
     'Before his sacking, Agbal, who favored using higher interest rates to tame inflation, had helped make the lira “one of the best performing emerging market currency this year.” According to a report, satisfied overseas money managers had reacted to Agbal’s policies by pouring funds into the Turkish economy.',
     'The report said: Overseas money managers had added a net of $4.6 billion to Turkish stocks and local currency bonds during Agbal’s tenure, betting that higher interest rates would help limit inflation and stabilize the lira.',
     'However, some money managers have expressed fears the new governor, Sahap Kavcioglu will pursue populist policies that are favored by President Erdogan.',
     'Yet in his initial comments following his appointment, Kavcioglu, who is the central bank’s fourth chief in less than two years, attempted to calm markets by pledging to maintain the same objectives as those of his predecessor.',
     'Depreciation of Lira Nevertheless, the new governor has also pledged “to foster economic stability by lowering borrowing costs and bolstering growth.” Concerned money managers believe these remarks could be a signal that the central bank will in the future “allow the lira to depreciate, and accept elevated inflation levels, to lower interest rates.” At the time of writing, Turkey’s interest rates stood at 19%.',
     'In addition to sparking the lira sell-off, the report states that Agbal’s sacking may have contributed to the plunge of Turkey’s benchmark Borsa Istanbul 100 stock index by 9.8% on March 22.',
     'According to the report, this plunge is the Borsa Istanbul Index’s “sharpest sell-off since June 2013 and triggering two trading halts.” Similarly, the Nasdaq-listed iShares MSCI Turkey exchange-traded fund fell more than 19% in U.S. trading.',
     'In the meantime, the report states that some investors have grown concerned that Turkey “would restrict their ability to sell local assets to stem the market turmoil.”However, Lütfi Elvan, Turkey’s Minister of Finance and Treasury, has issued a statement allaying the fears saying Turkey “wouldn’t impose capital controls or determine a fixed exchange rate.” What are your thoughts on the lira’s plunge following the appointment of a new governor?',
     'You can tell us what you think in the comments section below.',
     "Bitcoin In Contango: How Pricing Differences Can Foreshadow Further Gains                                        ECONOMICS | 1 day ago                                Report: JP Morgan Boss 'Not Happy' as Capital Positions Exemption for Big Banks Ends                                        ECONOMICS | 3 days ago                             Tags in this story                   borrowing costs, capital control, central bank governor, Emerging Markets, Exchange rate, Google bitcoin search, inflation, interest rates, Naci Agbal, Recep Tayyip Erdoğan, turkish lira crashing    Image Credits: Shutterstock, Pixabay, Wiki Commons Use Bitcoin and Bitcoin Cash to play online casino games here.",
     'Disclaimer: This article is for informational purposes only.',
     'It is not a direct offer or solicitation of an offer to buy or sell, or a recommendation or endorsement of any products, services, or companies.',
     'Bitcoin.com does not provide investment, tax, legal, or accounting advice.',
     'Neither the company nor the author is responsible, directly or indirectly, for any damage or loss caused or alleged to be caused by or in connection with the use of or reliance on any content, goods or services mentioned in this article.',
     'Read disclaimerShow comments',
     "Report: JP Morgan Boss 'Not Happy' as Capital Positions Exemption for Big Banks Ends                        On March 19, 2021, the U.S. Federal Reserve Board published a press release that detailed the temporary supplementary leverage ratio easements will be expiring as scheduled.",
     'U.S. banks will no longer have the relaxed capital requirements they once held since the onset of the Covid-19 outbreak.',
     'Despite Wall Street’s Protest Federal Regulators Deny SLR Extension During the first week of March, members of the U.S. Federal Reserve shrugged off concerns over inflation and erratic bond markets.',
     'More recently, Fed Chairman Jerome Powell explained that he looks at bitcoin as a substitute for gold.',
     'Now analysts and economists think the Fed may have some different issues ahead, as the U.S. central bank is ending the supplementary leverage ratio (SLR) easements invoked back in April 2020.',
     '“[The] Federal Reserve Board announces temporary change to its supplementary leverage ratio rule to ease strains in the Treasury market resulting from the coronavirus and increase banking organizations’ ability to provide credit to households and businesses,” the central bank noted and enacted at the time.',
     'The loosened rules supplementary leverage ratio meant that banks could exclude certain items from capital positions like cash and bonds on their balance sheets.',
     'The ruling was a stark difference to the words uttered back in 2013, when former Chairman Ben Bernanke wanted to “maintain strong capital positions.” When the Fed eased up on the SLR in April 2020, the central bank and major financial institutions caught a lot of flak from the media and economists.',
     'SLR Changes Will End March 31, CEO of JP Morgan Is ‘Not Happy’ Bank failures can severely harm an economy and without transparency, the importance of bank capital can be tossed to the wayside.',
     'The major Wall Street banking institutions attempted to get U.S. regulators to extend the SLR easements but they were ultimately denied.',
     'In April 2020, the Federal Reserve exempted big bank’s from certain capital requirements in order to stabilize the economy because of Covid-19.',
     'Now the temporary supplementary leverage ratio (SLR) easements will end this month.',
     'CNN Business contributor Matt Egan called the SLR measure the “big banks’ ‘get out of jail free’ card.” Then on March 19, 2021, the Federal Reserve Board announced the ending of the SLR provisions from last year.',
     '“The federal bank regulatory agencies today announced that the temporary change to the supplementary leverage ratio, or SLR, for depository institutions issued on May 15, 2020, will expire as scheduled on March 31, 2021,” the press release details.',
     '“The temporary change was made to provide flexibility for depository institutions to provide credit to households and businesses in light of the Covid-19 event.” A recent report from Bloomberg’s Peter Coy says Jamie Dimon, the American billionaire businessman and CEO of JP Morgan Chase is “not happy” with the temporary change ending.',
     'Coy also noted that Dimon “may soon turn away deposits” after revealing a snippet from the January 15 earnings call.',
     'Because the money supply spigot was turned on during the same time capital requirements easements were enacted, banks have a lot of liquidity.',
     'Coy notes that it’s a “strange problem” and “JP Morgan Chase and other big banks are getting more assets than they even want.” What do you think about the supplementary leverage ratio easements ending on March 31?',
     'Let us know what you think about this subject in the comments section below.',
     'Bitcoin In Contango: How Pricing Differences Can Foreshadow Further Gains                                        ECONOMICS | 1 day ago                                The Turkish Lira Plunges Following Removal of a Central Bank Governor, BTC Searches up                                        ECONOMICS | 3 days ago                             Tags in this story                   big banks, capital, capital requirements, economics, Fed, Federal Reserve, Finance, Jamie Dimon, Jamie Dimon JP Morgan, JP Morgan Chase, Liquidity, Matt Egan, Peter Coy, SLR, SLR easements    Image Credits: Shutterstock, Pixabay, Wiki Commons Purchase Bitcoin without visiting a cryptocurrency exchange.',
     'Buy BTC and BCH here.',
     'Disclaimer: This article is for informational purposes only.',
     'It is not a direct offer or solicitation of an offer to buy or sell, or a recommendation or endorsement of any products, services, or companies.',
     'Bitcoin.com does not provide investment, tax, legal, or accounting advice.',
     'Neither the company nor the author is responsible, directly or indirectly, for any damage or loss caused or alleged to be caused by or in connection with the use of or reliance on any content, goods or services mentioned in this article.',
     'Read disclaimerShow comments',
     'US Authorities Indict a Canadian Telecom Provider in a Crypto Money Laundering Case With Encrypted Phones                        A communications provider is in legal trouble with the U.S. authorities for a money laundering case involving cryptocurrencies.',
     'A federal court incited Sky Global for allegedly having facilitated criminal organizations to launder millions worth in crypto.',
     'Sky Global Is Also Accused of Remotely Destroying Devices to Avoid Seizing Per the filings, the Southern District of California stated that the Canadian-based company had been selling encryption devices to allegedly facilitate drug trafficking into Australia, Asia, Europe, and North America.',
     'The court believes Sky Global committed obstruction of justice by “creating, maintaining, and controlling” a system whereby the firm would remotely delete evidence of the illegal activities once authorities arrived to seize the equipment.',
     'Moreover, law enforcement accused the Canada-based telecom provider of using its infrastructure for the following purpose: To avoid detection of its illicit conduct by, among other things, laundering its illegal proceeds, communicating with encrypted devices, and transferring illegally obtained funds into cryptocurrency, specifically bitcoin.',
     'Sky Global allegedly charged their clients a subscription fee of approximately $1,200 – $1,200 per six-month period.',
     'Also, authorities commented that the firm used bitcoin (BTC) “to facilitate illegal transactions” on the customers’ website “to protect the membership’s anonymity, and to facilitate the laundering of the membership’s ill-gotten gains.”         US Govt Vilifies Anyone Who Is Against ‘Unwarranted Surveillance’, Says CEO In response to the indictment, Jean-François Eap, CEO of Sky Global, rejected the accusations from the U.S. authorities, and issued the following statement: To date, the authorities have not informed us of any of the proceedings which have been reported in the media.',
     'As I have been following the media reports in the past week, it is with great sadness that I see far-reaching coverage of what can only be described as erosion of the right to privacy.',
     'Eap added that Sky Global “was not created to prevent the police from monitoring criminal organizations; it exists to prevent anyone from monitoring and spying on the global community.” “The indictment against me personally in the United States is an example of the police, and the government trying to vilify anyone who takes a stance against unwarranted surveillance,” he said.',
     'What do you think about the indictment against Sky Global?',
     'Let us know in the comments section below.',
     "Domestic and Foreign Buyers Acquired a 'Tokenized' Apartment in a Spanish City by Paying With Ethereum                                        NEWS | 53 mins ago                                 Iconic 'Obey' Street Artist Shepard Fairey to Auction an NFT Mural on Superrare                                        NEWS | 5 hours ago                             Tags in this story                   Bitcoin (BTC), Canada, encrypted, Encrypted Communications, encrypted phone, Money Laundering, Privacy, telecommunications, U.S. District Court    Image Credits: Shutterstock, Pixabay, Wiki Commons Use Bitcoin and Bitcoin Cash to play online casino games here.",
     'Disclaimer: This article is for informational purposes only.',
     'It is not a direct offer or solicitation of an offer to buy or sell, or a recommendation or endorsement of any products, services, or companies.',
     'Bitcoin.com does not provide investment, tax, legal, or accounting advice.',
     'Neither the company nor the author is responsible, directly or indirectly, for any damage or loss caused or alleged to be caused by or in connection with the use of or reliance on any content, goods or services mentioned in this article.',
     'Read disclaimerShow comments',
     'Tokenized Jack Dorsey Tweet Sells for $2.9M, Funds From NFT Auction Will Be Sent to Charity                        On Monday, Jack Dorsey the founder of Twitter and Square witnessed a digital certificate or non-fungible token (NFT) of his first tweet being sold for $2.9 million.',
     'The new owner of the NFT Jack Dorsey tweet is the CEO of Bridge Oracle, Sina Estavi, who outbid the founder of Tron Justin Sun for the tokenized tweet.',
     'The non-fungible token (NFT) asset hype has grown immensely and on Monday, March 22, 2021, Jack Dorsey’s tokenized tweet from March 21, 2006, was sold for $2.9 million in ether.',
     'Bitcoin.com reported on the tokenized 2006 Dorsey tweet during the first week of March when bids were up to $1.5 million for the tweet.',
     'Jack Dorsey’s first tweet was tokenized by an application called Valuables, which is operated by a company named Cent.',
     'The Web3 compatible website v.cent.co leverages the Matic blockchain for the tokenization process.',
     'The final bid from the Bridge Oracle CEO Sina Estavi was for $2,915,835 or a touch over 1,630 ETH.',
     'The project Bridge Oracle is an oracle system built on the Tron network.',
     'Tron founder Justin Sun has also been trying to bid on the tweet and his last bid for Jack Dorsey’s first tweet was for $2 million or 1,304 ether.',
     'Twitter founder Jack Dorsey plans to donate the $2.9 million in ETH to a charity called Givedirectly, after transferring the ether into bitcoin (BTC).',
     '“Ending this March 21st [we] will immediately convert proceeds to bitcoin.',
     'And send to Givedirectly Africa Response,” Dorsey said on March 9, 2021.',
     'The tokenized tweet sale also follows the recently heightened criticism aimed at permanence vulnerability issues when it comes to long-term non-fungible token (NFT) asset storage.',
     'The debate has grown thick, as tokenized tweets similar to Jack Dorsey’s recent tokenized tweet, have been deleted and NFT images have been replaced.',
     'The NFT hype continues to swell as Monty Python’s John Cleese is selling an iPad drawing of the Brooklyn Bridge on Opensea, and Swedish musician and artist Jonas Quant announced launching an NFT called Dreamfng.',
     'What do you think about Jack Dorsey’s tokenized tweet selling for $2.9 million?',
     'Let us know what you think about this subject in the comments section below.',
     'Atari to Launch a Crypto-Fueled Gaming Platform in the Ethereum Metaverse Decentraland                                        BLOCKCHAIN | Mar 10, 2021                                A Crypto-Infused Professional Sports League: Billionaires Form a Blockchain Advisory Committee for the NBA                                        BLOCKCHAIN | Mar 6, 2021                             Tags in this story                   $2.9 Million, Auction, Jack Dorsey, Jack Dorsey Tweet, justin sun, nft, NFT Assets, NFT Auction, NFT Sale, Non-fungible Token, Outbid, Sina Estavi, tron, Twitter, Twitter CEO    Image Credits: Shutterstock, Pixabay, Wiki Commons Spot-markets for Bitcoin, Bitcoin Cash, Ripple, Litecoin and more.',
     'Start your trading here.',
     'Show comments',
     'Legendary Skateboarder Tony Hawk to Auction NFT of Trick Footage                        Tony Hawk joins a growing list of celebrities digitizing and selling their collectibles as non-fungible tokens.',
     'The skateboarding icon plans to auction a film of the last time he performed the Ollie 540 – the trick he first mastered 32 years ago.',
     'Ethernity Chain to Mint and Auction the Digital Collectible as NFT Boom Accelerates On the heels of numerous blockbuster NFT auctions by artists, celebrities, entertainers, and crypto evangelists, professional skateboarder Tony Hawk is set to auction his digital collectible at a still-unannounced date and time.',
     'The 52-year old pop-culture icon announced on Instagram that he had completed his final performance of the 540-degree “Ollie” that he landed first in 1989, underlining the scare and danger of doing the trick.',
     'On Twitter, he added, “Did my last ever ollie 540 today.',
     'You can compare it with my first one 32 years ago.',
     'This one meant a lot to me.”  Did my last ever ollie 540 today.',
     'You can compare it with my first one 32 years ago.',
     'This one meant a lot to me https://t.co/tExDDRFzb4 pic.twitter.com/Ggql2uADtg — Tony Hawk (@tonyhawk) March 18, 2021          According to the pro skateboarder, @tweestophers, angle of the shot of him completing the trick would be minted as an NFT on the Ethernity Chain and auctioned.',
     'Ethernity, a rapidly-expanding NFT blockchain network, is known for its social bent, given that all NFTs auctions donate a portion of the proceeds to charitable causes.',
     'The network has recently been tasked with building a collection of Muhammad Ali NFTs to benefit the Ali Center.',
     'Growing Chorus of Celebrity NFT Sales Echo Big Auction Results Hawk’s decision to turn to Ethernity for this event highlights the network’s growing cache following the launch of a 2500-piece lineup of Bosslogic’s NFT images specifically for Ethernity’s community.',
     'Bosslogic’s sales, while significant, were dwarfed by Beeple’s recent results in a Christie’s auction, which saw the artist sell one single JPEG made of 5000 smaller images for an eye-watering $69.3 million by Singapore-based cryptocurrency investor Metakovan.',
     'Beeple has also auctioned NFTs for charity.',
     'The NFT-craze has attracted many other celebrities, including Lindsey Lohan, Grimes, and key cryptocurrency evangelists like Twitter’s Jack Dorsey and Gemini’s Winklevoss twins.',
     'Still, some are more hesitant to embrace the movement, like Litecoin’s Charlie Lee, who likens the auctions to the speculative ICO bubble.',
     'Tony Hawk hasn’t been as vocal as some of his celebrity counterparts in talking up cryptocurrency and NFTs, yet he’s been a surprisingly long-time advocate of Bitcoin.',
     'Tweets in past years establish that he’s “hodled” Bitcoin since 2013.',
     'Hawk was also slated to present at the Bitcoin 2020 conference in San Francisco last March before it was canceled due to the pandemic.',
     'Are NFTs turning into a go-to quick-cash machine for celebrities, or are they seeing it as a legacy-cementing instrument?',
     'Let us know what you think in the comments section below.',
     "Domestic and Foreign Buyers Acquired a 'Tokenized' Apartment in a Spanish City by Paying With Ethereum                                        NEWS | 53 mins ago                                 Iconic 'Obey' Street Artist Shepard Fairey to Auction an NFT Mural on Superrare                                        NEWS | 5 hours ago                             Tags in this story                   Blockchain network, celebrity NFT, Ethernity, NFT Auctions, Tony Hawk    Image Credits: Shutterstock, Pixabay, Wiki Commons Spot-markets for Bitcoin, Bitcoin Cash, Ripple, Litecoin and more.",
     'Start your trading here.',
     'Disclaimer: This article is for informational purposes only.',
     'It is not a direct offer or solicitation of an offer to buy or sell, or a recommendation or endorsement of any products, services, or companies.',
     'Bitcoin.com does not provide investment, tax, legal, or accounting advice.',
     'Neither the company nor the author is responsible, directly or indirectly, for any damage or loss caused or alleged to be caused by or in connection with the use of or reliance on any content, goods or services mentioned in this article.',
     'Read disclaimerShow comments',
     "500 MW by 2025: Bitcoin Miner Greenidge and It's Wholly-Owned Power Plant to Be Listed on Nasdaq                        On Monday, the New York-based company Greenidge Generation Holdings revealed the business aims to be the first publicly-listed mining operation with a wholly-owned power plant.",
     'Greenidge expects to be publicly listed on Nasdaq via a merger with the firm Support.com.',
     'Greenidge Bitcoin Mining Operation and Gas Power Plant Plans to be Listed on Nasdaq Last year in March, Bitcoin.com reported on the company Greenidge Generation Holdings joining the bitcoin mining ecosystem, as it launched a fleet of mining rigs at the 65,000 square-foot natural gas utility plant in Dresden New York.',
     'Because Greenidge leverages the utility plant’s natural gas as its native energy source the company developed a direct “behind-the-meter cryptocurrency mining operation.” The following month, the gas power plant sold a fully compliant hashpower contract of 106 petahash to an undisclosed buyer as well.',
     'On March 22, 2021, Greenidge announced it plans to be publicly listed via Nasdaq through a merger with Support.com (Nasdaq: SPRT).',
     'The two companies signed an agreement to merge through a stock-for-stock transaction and after Support.com approves, Greenidge will become the firm’s wholly-owned subsidiary and will be listed on Nasdaq.',
     'At the time of publication, Greenidge’s power plant is a 106 MW gas plant with 19 MW dedicated to bitcoin mining.',
     'Greenidge is shooting toward 41 MW of capacity by Q2 2021 and 85 MW by the end of 2022.',
     '‘500 MW of Mining Capacity by the Year 2025’ Greenidge’s CEO Jeff Kirt explains that the merger is a milestone for the power plant.',
     'Moreover, investing in bitcoin mining companies has been very profitable in 2021 according to a recent report published by Fundstrat Global Advisors.',
     'With plans to replicate its vertically integrated mining model, Greenidge aims to capture 500 MW of mining capacity by the year 2025.',
     '“This merger is an important next step for Greenidge as we build upon our existing, integrated and proven platform for bitcoin mining and generation of lower-carbon affordable power,” Greenidge’s chief executive Jeff Kirt said.',
     'Kirt added: The transaction is a validation of our transformational journey, our proprietary relationships, and our industry expertise.',
     'It will allow for public market growth capital to propel Greenidge as we look to replicate the business model, which we have successfully executed in Upstate New York, in other locations.',
     'A Greenidge worker tends to bitcoin miners installed at the natural gas plant.',
     'Low-Cost Gas Gives Greenidge 1,186 Bitcoins at a Cost of Approximately $2,869 per Bitcoin During the last six months, the bitcoin mining ecosystem has grown more robust and expanded toward a large number of institutional investments.',
     'The President and CEO of Support.com, Lance Rosenzweig, explained the customer and technical support firm looks forward to evolving with Greenidge in the crypto economy.',
     '“As Greenidge looks to scale and seize new opportunities for growth, we are their ideal partner,” Rosenzweig said in regard to the Greenidge merger.',
     '“In addition, the transaction represents a significant value proposition for our shareholders by providing them with enhanced liquidity and the opportunity to participate in the growth of what we believe will be a successful competitor in the rapidly evolving domestic bitcoin mining space.” The Greenidge’s Upstate New York company claims to offer the “lowest-cost natural gas in North America.” The firm details that in 12 months the company’s mining operation raked in “1,186 bitcoins at a net variable cost of approximately $2,869 per bitcoin.” The power plant and its upcoming listing prospects aim to bolster the company as the first “U.S.',
     'public company operating a vertically integrated power generation asset and bitcoin mining operation.” What do you think about the merger between Greenidge and Support.com and the company’s mining expansion plans?',
     'Let us know what you think about this subject in the comments section below.',
     "Hut 8 Joins Foundry's US Mining Pool, Adds Over 14,000 Bitcoin Mining Rigs of Hashpower                                        MINING | 2 days ago                                AMD Won't Restrict Crypto Miners From Using Its Graphic Cards                                        MINING | 3 days ago                             Tags in this story                   Bitcoin, Bitcoin mining, BTC Mining, Cryptocurrency, Digital Assets, Greenidge, Greenidge CEO, Greenidge Gas Plant, Greenidge Generation Holdings, Greenidge mining, Jeff Kirt, Lance Rosenzweig, mining, natural gas, power plant, Support.com    Image Credits: Shutterstock, Pixabay, Wiki Commons Spot-markets for Bitcoin, Bitcoin Cash, Ripple, Litecoin and more.",
     'Start your trading here.',
     'Disclaimer: This article is for informational purposes only.',
     'It is not a direct offer or solicitation of an offer to buy or sell, or a recommendation or endorsement of any products, services, or companies.',
     'Bitcoin.com does not provide investment, tax, legal, or accounting advice.',
     'Neither the company nor the author is responsible, directly or indirectly, for any damage or loss caused or alleged to be caused by or in connection with the use of or reliance on any content, goods or services mentioned in this article.',
     'Read disclaimerShow comments',
     'Fed Chairman Jerome Powell Says Bitcoin Is a Substitute for Gold                        Federal Reserve Chairman Jerome Powell sees cryptocurrency as a substitute for gold, rather than the dollar.',
     'However, he cautioned that crypto assets, including bitcoin, are highly volatile and not useful as a store of value.',
     'Fed Chair Sees Bitcoin as a Gold Substitute During a webinar sponsored by the Bank of International Settlements (BIS) on Monday, Federal Reserve Chairman Jerome Powell talked about cryptocurrency, mentioning bitcoin in particular.',
     'The Fed chair detailed: “Crypto assets are highly volatile, see bitcoin, and therefore are not really useful as a store of value.',
     'And, they are not backed by anything.',
     'They are more of an asset for speculation.',
     'They are also not particularly in use as a means of payment.” He elaborated: It’s more a speculative asset.',
     'It’s essentially a substitute for gold, rather than for the dollar.',
     'Powell proceeded to say that with crypto assets, “the public needs to understand the risks.',
     'The principal thing is there’s the volatility.',
     'There’s also the outsized energy requirement for mining, and the fact that they’re not backed by anything.” What do you think about what Fed Chair Powell said?',
     'Let us know in the comments section below.',
     'Microsoft President Says the World Is Best Served When Currencies Are Issued and Backed by Governments                                        REGULATION | 7 hours ago                                New FATF Guidance Applies Regulatory Standards to Decentralized Exchanges, Defi and NFTs                                        REGULATION | 1 day ago                             Tags in this story                   bitcoin usd, crypto assets, Cryptocurrency, dollar substitute, Fed Chair, gold, gold substitute, Jay Powell, jerome powell, US Dollar    Image Credits: Shutterstock, Pixabay, Wiki Commons Purchase Bitcoin without visiting a cryptocurrency exchange.',
     'Buy BTC and BCH here.',
     'Disclaimer: This article is for informational purposes only.',
     'It is not a direct offer or solicitation of an offer to buy or sell, or a recommendation or endorsement of any products, services, or companies.',
     'Bitcoin.com does not provide investment, tax, legal, or accounting advice.',
     'Neither the company nor the author is responsible, directly or indirectly, for any damage or loss caused or alleged to be caused by or in connection with the use of or reliance on any content, goods or services mentioned in this article.',
     'Read disclaimerShow comments',
     "AMD Won't Restrict Crypto Miners From Using Its Graphic Cards                        Major chipmaker Advanced Micro Devices (AMD) keeps making headlines amid the battle between crypto miners and PC gamers.",
     'Now, the company seems to have made a step forward to become friendlier to those with crypto mining purposes.',
     'AMD Doesn’t Want to Follow Nvidia’s Path According to a PC Gamer report, citing a product manager from AMD, the chipmaker has zero intentions to block crypto mining operations on its devices.',
     '“The short answer is no,” said Nish Neelalojanan, a product manager of the firm.',
     'Crypto miners were concerned about AMD’s possibility of implementing the same measures as its competitor, Nvidia, which recently implemented a limiter on its graphic processor units (GPUs) for ethereum (ETH) mining.',
     'Specifically, Nvidia made the maneuver on the Geforce RTX 3090 graphics cards in the wake of the gaming community’s complaints.',
     'Neelalojanan said: We will not be blocking any workload, not just mining for that matter.',
     'That said, there are a couple of things.',
     'First of all, RDNA was designed from the ground up for gaming, and RDNA 2 doubles up on this.',
     'And what I mean by this is, Infinity Cache and a smaller bus width were carefully chosen to hit a very specific gaming hit rate.',
     'PC Gaming Will Remain the Highest Priority, AMD Says But the AMD product manager further clarified: However, mining specifically enjoys, or scales with, higher bandwidth and bus width, so there are going to be limitations from an architectural level for mining itself.',
     'That said, Neelalojanan pointed out that AMD’s focus is still on improving its devices by giving gaming the highest priority.',
     'AMD Navi 21 chip is capable of handling around 58-64 MH/s for mining ETH.',
     'As Bitcoin.com News reported on March 9, 2021, the Navi 12 GPU won’t support VCN (video core next), a requirement for PC gaming purposes, and making it the proper candidate for crypto mining.',
     'What do you think about the statement from AMD on its stance towards crypto mining?',
     'Let us know in the comments section below.',
     "Hut 8 Joins Foundry's US Mining Pool, Adds Over 14,000 Bitcoin Mining Rigs of Hashpower                                        MINING | 2 days ago                                500 MW by 2025: Bitcoin Miner Greenidge and It's Wholly-Owned Power Plant to Be Listed on Nasdaq                                        MINING | 3 days ago                             Tags in this story                   AMD, crypto mining, ETH, ETH Price, ether, Ethereum (ETH), Ethereum mining, GPU, GPU manufacturers, GPU Miners, Graphic processing units (GPUs), Nvidia    Image Credits: Shutterstock, Pixabay, Wiki Commons Spot-markets for Bitcoin, Bitcoin Cash, Ripple, Litecoin and more.",
     'Start your trading here.',
     'Show comments',
     "'Wolf of Wallstreet' Jordan Belfort Now Bullish on BTC, Says Crypto Asset Will Reach $100,000                        Jordan Belfort, the self-proclaimed “Wolf of Wallstreet” has moved away from his bearish views on bitcoin and now predicts the crypto asset might reach $100,000.",
     'Belfort, who has reinvented himself as a motivational speaker and consultant, says bitcoin unlike company stocks is more about “pure supply and demand.” He adds that the crypto asset is “free from the distortion of effectively printing more of your own corporate currency.” Bitcoin’s Fixed Supply Distinguishes It From Stocks Belfort’s remarks about bitcoin (BTC) were made during an interview with Fortune where he offered his take on Gamestop stock’s recent Reddit inspired price rally.',
     'While some observers have viewed the Gamestop price rally as another form of pump and dump, Belfort disagrees.',
     'Although he concedes that traders were “collectively pumping up these stocks” he, however, insists there is “no dump.” Meanwhile, when compared to company stocks, Belfort concludes that the crypto asset’s fixed supply gives it an advantage over shares.',
     'The former penny stock trader argues that “if a stock stays up too long, the company keeps printing shares.” However, this continuing issuance of shares will lead to dilution of value.',
     'On the other hand, bitcoin “does not have the equities’ problem that high prices tend to bring on more supply (thus) turning the surge into a swoon.”        Belfort Now Bullish on Bitcoin On why he turned from being a bitcoin bear, Belfort again points to the crypto asset’s fixed supply as its key “advantage over company stocks.” He adds: New investors will keep pumping the price, but the rush of new shares that deflates a soaring stock can’t happen.',
     'In 2017, when the crypto asset surpassed $19,000, Belfort, who was sent to prison for nearly two years for fleecing small investors out of $200 million, predicted that bitcoin would crash and “go away forever.” The former penny stock trader argued then that regulators “would nix the coins.” However, after seeing the crypto asset set another all-time high, Belfort says he “now believes it’s here to stay since a fatal regulatory onslaught hasn’t happened.” What are your thoughts on Belfort’s changed stance on BTC?',
     'You can share your views in the comments section below.',
     'Fidelity Applies to Launch Bitcoin ETF Amid Accelerating Institutional Demand for BTC                                        MARKETS AND PRICES | 11 hours ago                                JMP Securities Expects $1.5 Trillion to Flow Into Bitcoin as Retail Wealth Management Industry Follows Morgan Stanley to Offer BTC to Clients                                        MARKETS AND PRICES | 1 day ago                             Tags in this story                   bitcoin fixed supply, BTC, company stocks, Crypto asset, GameStop Bets, jordan belfort, penny stock, pump and dump, Wolf of Wallstreet    Image Credits: Shutterstock, Pixabay, Wiki Commons Purchase Bitcoin without visiting a cryptocurrency exchange.',
     'Buy BTC and BCH here.',
     'Show comments',
     'Ebang Hopes to Capture a ‘Competitive Edge’ After Developing a Next-Generation 6nm Bitcoin Mining Chip                        The publicly-listed bitcoin mining rig manufacturer Ebang has announced the completion of a new 6nm ASIC mining chip.',
     'Ebang believes the next-generation mining chip will give the firm’s ASIC products a “competitive edge” in the ambitious bitcoin mining industry.',
     'Mass Production of 6nm Chips to Begin When Market Conditions Become Suitable During the last few weeks, Bitcoin’s hashrate has been climbing to new heights as the network tapped 190 exahash per second on March 22, 2021.',
     'The price of bitcoin (BTC) has also helped propel bitcoin mining profits, but improved mining rig semiconductors have also increased the hashpower a great deal.',
     'Firms like Bitmain, Canaan, Microbt, and Ebang have seen massive demand in 2020 and into 2021.',
     'Most of these manufacturers are out of stock after selling mass quantities of ASIC miners to institutional buyers.',
     'On March 17, the China-based Ebang International Holdings Inc. (Nasdaq: EBON), announced the development of a new 6nm ASIC mining chip.',
     'The standard chips used today in some of the top mining devices are between 12nm, 10nm, and 7nm.',
     'A 6nm chip would be an improvement from today’s ASIC standards, and Ebang thinks the new product will bolster the company’s performance.',
     'When market conditions become more suitable, Ebang said it will begin mass production of the 6nm chip.',
     'The Chairman and CEO of Ebang, Mr. Dong Hu, believes the development of the chip is a watershed moment for Ebang’s research and development (R&D) team.',
     '“The completion of our 6nm ASIC chip is a significant milestone of our constant R&D efforts in recent years,” Mr. Dong Hu said during the announcement.',
     '“We have devoted a lot of resources to building our R&D team and product innovation and iteration.',
     'Looking forward, we plan to continue developing ASIC chips designed for more mainstream cryptocurrency machines, which we believe will increase our revenue in the digital currency business and optimize our product offering structure along the blockchain industry value chain.”        Ebang Launches Invite-Only Beta Exchange, Stock Jumps 48% Since January 2021 The Hangzhou mining rig manufacturer Ebang also launched the company’s invite-only beta cryptocurrency exchange.',
     'Beta testing began on March 15 and Ebang’s CEO Mr. Dong Hu said that the exchange would also “accelerate the company’s development.” Since the exchange launch, Ebang’s shares listed on Nasdaq jumped in value.',
     'Since January 2021, Ebang’s shares have increased in value by 48.45%.',
     'As far as the new 6nm chip production is concerned, Ebang has highlighted that commercialization of such chips is not guaranteed.',
     'The company plans to continue developing ASIC mining rigs for bitcoin with hopes to gather widespread appeal.',
     'The company has stiff competition with manufacturing firms like Bitmain, Canaan, Microbt, Innosilicon, and Strongu.',
     'What do you think about Ebang developing a 6nm ASIC mining chip?',
     'Let us know what you think about this subject in the comments section below.',
     "Hut 8 Joins Foundry's US Mining Pool, Adds Over 14,000 Bitcoin Mining Rigs of Hashpower                                        MINING | 2 days ago                                500 MW by 2025: Bitcoin Miner Greenidge and It's Wholly-Owned Power Plant to Be Listed on Nasdaq                                        MINING | 3 days ago                             Tags in this story                   10nm, 12nm, 6nm chip, 6nm chip production, 7nm, ASIC Mining, Bitcoin, Bitcoin (BTC), Bitcoin mining, Bitmain, BTC Mining, Canaan, Ebang, Microbt, mining, Mining Chip    Image Credits: Shutterstock, Pixabay, Wiki Commons Use Bitcoin and Bitcoin Cash to play online casino games here.",
     'Disclaimer: This article is for informational purposes only.',
     'It is not a direct offer or solicitation of an offer to buy or sell, or a recommendation or endorsement of any products, services, or companies.',
     'Bitcoin.com does not provide investment, tax, legal, or accounting advice.',
     'Neither the company nor the author is responsible, directly or indirectly, for any damage or loss caused or alleged to be caused by or in connection with the use of or reliance on any content, goods or services mentioned in this article.',
     'Read disclaimerShow comments',
     'Another Binance Smart Chain Project Turtledex Rug Pulls With Tokens Worth $2.5M Confirmed Stolen                        The team behind Turtledex protocol, a Binance Smart Chain (BSC) project has reportedly rug pulled its investors shortly after raising 9,000 Binance tokens worth $2.5 million.',
     'Immediately following the confirmation of the theft, the value of the protocol’s native token TTDX, reportedly plunged to near zero.',
     'At the time of writing, the social media accounts associated with the protocol appear to have been deleted.',
     'The Rug Pull As one report explains, Turtledex, which touted itself as a decentralized storage platform for users, successfully duped users by claiming it could help them “keep data and preserve files without needing to keep them on their computer.” These claims initially helped the Turtledex team to raise funds via the pre-sale round “in just two hours on Monday 15th.” Next, the team would follow up this successful fundraising round by “opening their liquidity pools on two major BSC Decentralized exchanges (DEXs): Pancakeswap and Apeswap.” The native tokens, TTDX, “were unlocked and opened to circulation on March 18th at 15:00 UTC and the price quickly pumped above its pre-sale value.”  However, just 24 hours later, a Twitter user named Defistalker warned TTDX holders of a possible exit scam after noticing the removal of liquidity on the two DEXs.',
     'In an initial tweet, which shares the screenshot of the alleged theft, Defistalker said: Liquidity on both Apeswap and Pancakeswap has been removed 10hrs ago and swapped to ETH, split to 9 wallets, all sent to Binance.',
     'Stopping Exit Scammers Consequently, some users are now asking Binance to intervene by freezing the stolen funds and reimbursing the affected investors.',
     'Although Binance had not formally responded at the time of writing, some users are pinning hopes on remarks made in the past by the exchange’s executives.',
     'For instance, in a tweet on March 15, 2021, Binance CEO Changpeng Zhao (CZ) hints that his organization is actively involved in helping investors track down funds after rug pulls.',
     'Still, some users have expressed doubts about Binance’s willingness or ability to will help after noting the exchange’s failure to help Meerkat Finance investors following the rug pull.',
     'Approximately $31 million was siphoned from Meerkat Finance (another BSC project) on March 4 shortly after developers reported a hacking attack.',
     'However, Binance has since told Bitcoin.com News that its security and legal teams were in fact involved in the investigation and the subsequent recovery of funds stolen from Meerkat Finance.',
     'Binance adds that Meerkat Finance went on to launch a refund program and to date, “at least 95% of users’ losses have been recovered successfully, with ongoing distributions to remaining victims.”         Meanwhile, some investors like Twitter user Defistalker are now querying the value of auditing smart contracts when “developers can simply pull liquidity to dump.” Defistalker said: “Turtledex (TTDX) smart contract was audited by Techrate1 on February 21, 2021.',
     'Whilst no big issues were found, it asks the question, what is the purpose of a smart contract audit?',
     'If developers can simply pull liquidity to dump, are we even asking the right questions?” On March 23, at 4.30 EST, this article was updated to reflect Binance’s involvement in the recovery of funds stolen from Meerkat Finance.',
     'Can smart contracts audits help pre-empt rug puls in the Defi space?',
     'You can share your thoughts in the comments section below.',
     'Zeitgeist Brings a Prediction Markets Ecosystem to Polkadot’s Kusama                                        ALTCOINS | 3 hours ago                                Livepeer Token Surges After Grayscale Investment Announced Creation of Trust to Invest in LPT                                        ALTCOINS | 23 hours ago                             Tags in this story                   Binance Smart Chain, Changpeng Zhao (CZ), decentralized exchanges, Defi rug pull, Exit Scam, Liquidity, Meerkat Finance, Techrate1, Turtledex, Turtledex rug pull    Image Credits: Shutterstock, Pixabay, Wiki Commons Use Bitcoin and Bitcoin Cash to play online casino games here.',
     'Disclaimer: This article is for informational purposes only.',
     'It is not a direct offer or solicitation of an offer to buy or sell, or a recommendation or endorsement of any products, services, or companies.',
     'Bitcoin.com does not provide investment, tax, legal, or accounting advice.',
     'Neither the company nor the author is responsible, directly or indirectly, for any damage or loss caused or alleged to be caused by or in connection with the use of or reliance on any content, goods or services mentioned in this article.',
     'Read disclaimerShow comments',
     "Scaramucci's Skybridge Capital Registers With SEC to Launch a Bitcoin ETF                        Anthony Scaramucci’s Skybridge Capital has filed an S-1 registration form with the Securities and Exchange Commission (SEC) for a bitcoin exchange-traded fund (ETF).",
     'The prospectus for the “First Trust Skybridge Bitcoin ETF Trust” notes that the ETF aims to list common shares on NYSE Arca.',
     'Skybridge Capital Files to List a Bitcoin ETF Skybridge Capital and its founder Anthony Scaramucci have been all about bitcoin (BTC) these days and even more so after the company’s successful launch of the Skybridge Capital Bitcoin Fund.',
     'At the end of February, Scaramucci said in an interview that he expects $100k BTC prices this year.',
     'On March 19, 2021, Skybridge Capital filed an S-1 registration prospectus for a bitcoin exchange-traded fund (ETF).',
     'Skybridge Capital will be among a number of firms such as Vaneck and Valkyrie Digital Assets looking to get their ETFs approved.',
     'The S-1 registration filed on Friday notes that the ETF will be dubbed the “First Trust Skybridge Bitcoin ETF Trust” but the ticker it will leverage has been left blank.',
     'Skybridge is hoping to list common shares on NYSE Arca but notes that the trust is “newly organized.” Because there is no prior history, common shares of the Skybridge bitcoin trust may trade a discount from their net asset value (“NAV”).',
     'The former White House communications director Scaramucci’s company also says that the trust is an “emerging growth company.” “The trust is an ‘emerging growth company,’” the Skybridge Capital registration filing says.',
     '“As that term is used in the Jumpstart Our Business Startups Act (the “JOBS Act”) and, as such, may elect to comply with certain reduced reporting requirements,” the ETF filing adds.',
     'The registration explains the risks involved with the “Skybridge Bitcoin ETF Trust” which is the typical standard statements like: “Bitcoin is a new technological innovation with a limited history.” “There is no assurance that usage of bitcoin will continue to grow,” the ETF filing from Skybridge Capital notes.',
     '“A contraction in use of bitcoin may result in increased volatility or a reduction in the price of bitcoin, which could adversely impact the value of the common shares.”        Scaramucci’s Spacex Tweet In addition to the filing, Anthony Scaramucci has been discussing bitcoin an awful lot on Twitter and speculated that Elon Musk’s firm Spacex has bitcoin (BTC) on its balance sheet.',
     'The speculative tweet from Scaramucci also was focused on the subject of climate change and renewable energy.',
     '“Elon Musk didn’t stop with Tesla,” Scaramucci insisted in a tweet.',
     '“I understand that Spacex owns bitcoin on its balance sheet.',
     'Elon Musk owns over $5 billion in bitcoin via Tesla, SpaceX, and personally.',
     'No living person has done more to protect the planet against climate change,” the Skybridge Capital founder added.',
     'Scaramucci continued: [The] idea Elon Musk would invest in [a] “dirty asset” is absurd.',
     '[The] future of bitcoin mining is renewable energy.',
     'Elon Musk sees future 1) renewables replace fossil fuels 2) bitcoin demonetizes gold, RE, equities & art + becomes [the] dominant store of value.',
     '[The] answer to energy FUD is Elon Musk.',
     'Scaramucci also did an interview on March 19, and explained that more businesses should hold bitcoin treasuries.',
     'Skybridge Capital manages $600 million in bitcoin and Scaramucci discussed the possibility of Skybridge ethereum investment products as well.',
     'The Skybridge founder further touched on his $100k BTC price prediction and said “I’m sticking with $100,000 because if I tell people what I really think… they’re gonna think that I’m nuts.” What do you think about Scaramucci’s Skybridge Capital filing to list an ETF on NYSE Arca in a SEC registration prospectus?',
     'Let us know what you think about this subject in the comments section below.',
     'Stock Exchange SIX Aims to List Bitcoin on SDX, Crypto Trading Platform to Launch This Summer                                        FINANCE | Mar 18, 2021                                Morgan Stanley Making 3 Bitcoin Funds Available to Clients                                        FINANCE | Mar 18, 2021                             Tags in this story                   $100K, $600 million in BTC, Anthony Scaramucci, balance sheets, Bitcoin (BTC), BTC, Elon Musk, ETF, exchange traded fund, Finance, Scaramucci, skybridge bitcoin, Skybridge Bitcoin ETF, Skybridge Capital, Skybridge ethereum, SpaceX, Tesla, Treasuries, White House Staffer    Image Credits: Shutterstock, Pixabay, Wiki Commons Purchase Bitcoin without visiting a cryptocurrency exchange.',
     'Buy BTC and BCH here.',
     'Disclaimer: This article is for informational purposes only.',
     'It is not a direct offer or solicitation of an offer to buy or sell, or a recommendation or endorsement of any products, services, or companies.',
     'Bitcoin.com does not provide investment, tax, legal, or accounting advice.',
     'Neither the company nor the author is responsible, directly or indirectly, for any damage or loss caused or alleged to be caused by or in connection with the use of or reliance on any content, goods or services mentioned in this article.',
     'Read disclaimerShow comments',
     'Deutsche Bank: Bitcoin Now 3rd Largest Currency, Too Important to Ignore                        Deutsche Bank has published a report stating that bitcoin is too important to ignore, noting that it is now the third-largest currency in terms of the total value in circulation.',
     'In addition, the bank says that governments and central banks know that cryptocurrencies are here to stay and are expected to start regulating the industry this year.',
     'Bitcoin Is Too Important to Ignore Deutsche Bank Research published a report last week entitled: “Bitcoins: Can the Tinkerbell Effect Become a Self-Fulfilling Prophecy?” It is part three of “The Future of Payments: Series 2.” The report author, research analyst Marion Laboure, Ph.D., wrote: Bitcoin’s market cap of $1 trillion makes it too important to ignore.',
     'As long as asset managers and companies continue to enter the market, bitcoin prices could continue to rise.',
     'At the time of writing, the price of bitcoin stands at $57,455 and the cryptocurrency’s market cap is approximately $1.07 trillion based on data from markets.Bitcoin.com.',
     'The report also discusses bitcoin as a commodity, currency, and equity.',
     'While noting that “bitcoin transactions and tradability are still limited,” the report states that the cryptocurrency’s “market cap is among the top ten, both as a currency and as a stock.” Comparing bitcoin to fiat currencies, the report details: In terms of total currency in circulation, bitcoin is the third-largest in the world, after the US dollar and the euro.',
     'Value of currencies in circulation.',
     'Source: Deutsche Bank “This is mainly due to the vast increase in bitcoin’s value recently,” the report continues, adding that “In early 2019, bitcoin represented ‘only’ 3% of the US dollars in circulation, but in February 2021 it surged beyond 40% of the US dollars in circulation.” The fourth-largest currency, according to Deutsche Bank Research, is the Japanese yen, followed by the Indian rupee.',
     'Laboure asserted that “Bitcoin’s value will continue to rise and fall depending on what people believe it is worth.” She explained that “This is sometimes called the Tinkerbell Effect,” which is “a recognised economic term stating that the more people believe in something, the likelier it is to happen based on Peter Pan’s assertion that Tinkerbell exists because children believe she exists.” Furthermore, the Deutsche Bank analyst opined: Central banks and governments understand that cryptocurrencies are here to stay, so they are expected to start regulating crypto-assets late this year or early next year.',
     'The Deutsche Bank report also notes that central banks “are also speeding up research on their own central bank digital currencies (CBDCs) and launching pilots.”         Laboure proceeded to discuss the future of bitcoin.',
     'In the short term, she said, “bitcoin is here to stay and its value will remain volatile.” In the medium to long run, the analyst believes that “due to very strong network effects, there will likely be little room for using cryptocurrencies as a widespread means of payment.” Moreover, she cautioned that in the long term, bitcoin “will have to transform potential into results to sustain its value proposition,” elaborating: In the long run, central banks are unlikely to give up their monopolies.',
     'And as long as governments and central banks exist and hold the power to regulate money, there will be little room for bitcoin—as a means of payment—to replace traditional currencies.',
     'What do you think about Deutsche Bank’s view on bitcoin?',
     'Let us know in the comments section below.',
     'Fidelity Applies to Launch Bitcoin ETF Amid Accelerating Institutional Demand for BTC                                        MARKETS AND PRICES | 11 hours ago                                JMP Securities Expects $1.5 Trillion to Flow Into Bitcoin as Retail Wealth Management Industry Follows Morgan Stanley to Offer BTC to Clients                                        MARKETS AND PRICES | 1 day ago                             Tags in this story                   Bitcoin, bitcoin currency, Bitcoin regulation, Crypto regulation, db bitcoin, Deutsche Bank, deutsche bank bitcoin, deutsche bank crypto, deutsche bank cryptocurrency, tinkerbell effect    Image Credits: Shutterstock, Pixabay, Wiki Commons, Deutsche Bank Research Use Bitcoin and Bitcoin Cash to play online casino games here.',
     'Disclaimer: This article is for informational purposes only.',
     'It is not a direct offer or solicitation of an offer to buy or sell, or a recommendation or endorsement of any products, services, or companies.',
     'Bitcoin.com does not provide investment, tax, legal, or accounting advice.',
     'Neither the company nor the author is responsible, directly or indirectly, for any damage or loss caused or alleged to be caused by or in connection with the use of or reliance on any content, goods or services mentioned in this article.',
     'Read disclaimerShow comments',
     "Bridgewater's Ray Dalio Warns Government Could Restrict Bitcoin Investments, Impose 'Shocking' Taxes                        The founder and chief investment officer of Bridgewater Associates, the world’s largest hedge fund firm, has warned that the government could “impose prohibitions against capital movements” into assets such as bitcoin.",
     'He added that regulators may also impose changes in taxes that “could be more shocking than expected.” Ray Dalio Warns About Government Prohibitions and Taxes Ray Dalio, founder and chief investment officer of Bridgewater Associates, wrote a post on Linkedin last week entitled: “Why in the World Would You Own Bonds When…” He pointed out that the bond markets currently offer “ridiculously low yields,” which “do not meet these asset holders’ funding needs.” The executive wrote, “There is now over $75 trillion of US debt assets of varying maturities,” adding that their holders will at some point want to sell them to get cash to buy goods and services with.',
     'However, Bridgewater’s chief investment officer estimates that “at current valuations, there is way too much money in these financial assets for it to be a realistic expectation that any significant percentage of that bond money can be turned into cash and exchanged for goods and services.” He elaborated: “It has to be accommodated … via printing a lot of money and devaluing it, and restructuring a lot of debt and government finances, usually including large increases in taxes.” Dalio explained: “Based both on how things have worked historically and what is happening now, I am confident that tax changes will also play an important role in driving capital flows to different investment assets and different locations, and those movements will influence market movements.” The billionaire fund manager emphasized that “If history and logic are to be a guide, policymakers who are short of money will raise taxes and won’t like these capital movements out of debt assets and into other storehold of wealth assets and other tax domains,” warning: They could very well impose prohibitions against capital movements to other assets (e.g., gold, bitcoin, etc.)',
     'and other locations.',
     'These tax changes could be more shocking than expected.',
     'The Bridgewater Associates founder used Elizabeth Warren’s proposed wealth tax as an example, stating that it “is of an unprecedented size.” Citing his study of “wealth taxes in other countries at other times,” he expects this proposal “will most likely lead to more capital outflows and other moves to evade these taxes.” Consequently, “The United States could become perceived as a place that is inhospitable to capitalism and capitalists,” Dalio opined, emphasizing that “the chances of a sizable wealth tax bill passing over the next few years are significant.” In conclusion, the Bridgewater executive cautioned: One should be mindful of tax changes and the possibility of capital controls.',
     'Dalio has been studying bitcoin over the recent months.',
     'In November last year, he admitted that he may be wrong about bitcoin but was nonetheless worried about governments outlawing cryptocurrency.',
     'In December, he said bitcoin could “serve as a diversifier to gold and other such storehold of wealth assets.” Then, in January this year, he said that “bitcoin is one hell of an invention,” revealing that his firm looking closely at the cryptocurrency.',
     'What do you think about Ray Dalio’s warning?',
     'Let us know in the comments section below.',
     'Microsoft President Says the World Is Best Served When Currencies Are Issued and Backed by Governments                                        REGULATION | 7 hours ago                                New FATF Guidance Applies Regulatory Standards to Decentralized Exchanges, Defi and NFTs                                        REGULATION | 1 day ago                             Tags in this story                   bridgewater associates, bridgewater associates bitcoin, capital control, fund manager, Government, government prohibition, Ray Dalio, Ray Dalio bitcoin, restrictions, Taxation    Image Credits: Shutterstock, Pixabay, Wiki Commons Spot-markets for Bitcoin, Bitcoin Cash, Ripple, Litecoin and more.',
     'Start your trading here.',
     'Disclaimer: This article is for informational purposes only.',
     'It is not a direct offer or solicitation of an offer to buy or sell, or a recommendation or endorsement of any products, services, or companies.',
     'Bitcoin.com does not provide investment, tax, legal, or accounting advice.',
     'Neither the company nor the author is responsible, directly or indirectly, for any damage or loss caused or alleged to be caused by or in connection with the use of or reliance on any content, goods or services mentioned in this article.',
     'Read disclaimerShow comments',
     'FBI Arrests 6 Free Keene Members Operating Cryptocurrency Exchange                        Six people involved with the Free Keene movement have been arrested and charged for allegedly operating an illegal cryptocurrency exchange business.',
     '“They operated their virtual currency exchange business using websites, as well as operating virtual currency ATM machines in New Hampshire,” the U.S. Department of Justice (DOJ) said.',
     'FBI Raids Properties and Charges 6 People The DOJ announced this week that it has charged six New Hampshire residents with crimes related to a cryptocurrency exchange business.',
     'The Federal Bureau of Investigation (FBI) raided several Keene properties associated with members of the liberty-minded Free Keene movement on Tuesday and arrested the six defendants.',
     'The individuals charged are Ian Freeman (formerly Ian Bernard), 40, of Keene; Colleen Fordham, 60, of Alstead; Renee Spinella, 23, of Derry; Andrew Spinella, 35, of Derry; Nobody (formerly Richard Paul), 52, of Keene; and Aria DiMezzo (formerly James Baker), 34, of Keene.',
     'DiMezzo is the self-described “transsexual satanist anarchist” who ran as the Republican candidate for Cheshire County sheriff in 2020.',
     'Both Freeman and Nobody ran for Governor.',
     'Freeman reportedly had accessed $1.66 million worth of bitcoins and had $178,000 in cash in his safe when his house was raided by the FBI, prosecutors said on Friday, adding that he also has access to gold, silver, and possibly other liquid assets and cryptocurrencies.',
     'The indictment details that since 2016: The defendants have operated a business that enabled customers to exchange over ten million dollars in fiat currency for virtual currency, charging a fee for their service.',
     'They operated their virtual currency exchange business using websites, as well as operating virtual currency ATM machines in New Hampshire.',
     'The Free Keene Project describes itself as “New Hampshire’s liberty activism destination.” Long-time supporters of cryptocurrency, in 2018, the group launched a Bitcoin Embassy in Keene.',
     'The indictment also “alleges that the defendants knowingly operated the crypto exchange business in violation of federal anti-money laundering laws and regulations,” noting that “Some defendants opened bank accounts in the names of purported religious entities.” They include Shire Free Church, the Crypto Church of NH, the Church of the Invisible Hand, and the Reformed Satanic Church.',
     'According to the DOJ, “some defendants then engaged in substantial efforts to evade detection of their unlawful virtual currency exchange scheme by avoiding answering financial institutions’ questions about the nature of the business and misleading financial institutions into believing their unlawful virtual currency exchange business was instead a religious organization receiving charitable contributions.” Acting U.S. Attorney John J. Farley explained that all six individuals were indicted “and charged with participating in a conspiracy to operate an unlicensed money transmitting business and other offenses.” Freeman and DiMezzo are additionally charged with operating an unlicensed money transmitting business.',
     'Moreover, Freeman, Fordham, Renee Spinella, Andrew Spinella, and Nobody are charged with wire fraud and participating in a conspiracy to commit wire fraud.',
     'Freeman is further charged with money laundering and operating a continuing financial crimes enterprise.',
     'What do you think about this case?',
     'Let us know in the comments section below.',
     'Microsoft President Says the World Is Best Served When Currencies Are Issued and Backed by Governments                                        REGULATION | 7 hours ago                                New FATF Guidance Applies Regulatory Standards to Decentralized Exchanges, Defi and NFTs                                        REGULATION | 1 day ago                             Tags in this story                   Aria DiMezzo, Arrested, Bitcoin exchange, crypto exchange, FBI, Free Keene, free keene members, Ian Freeman, New Hampshire, Nobody    Image Credits: Shutterstock, Pixabay, Wiki Commons Use Bitcoin and Bitcoin Cash to play online casino games here.',
     'Disclaimer: This article is for informational purposes only.',
     'It is not a direct offer or solicitation of an offer to buy or sell, or a recommendation or endorsement of any products, services, or companies.',
     'Bitcoin.com does not provide investment, tax, legal, or accounting advice.',
     'Neither the company nor the author is responsible, directly or indirectly, for any damage or loss caused or alleged to be caused by or in connection with the use of or reliance on any content, goods or services mentioned in this article.',
     'Read disclaimerShow comments',
     'Opioid Premiums, Jobseekers and Vaccines: Covid-19 Fuels Darknet Markets in a Different Way                        The impact of the Covid-19 pandemic has found its way onto the deep web and during the last few months, researchers have found a great number of individuals turning to darknet markets (DNMs) for employment.',
     'The organization Check Point Research has recorded a rising trend of people looking for jobs on hacking forums and DNMs.',
     'Additionally, alleged versions of the coronavirus vaccines from the top manufacturers are being sold on DNMs alongside vaccination records.',
     'Mass Darknet Market Closures in 2020 Invoked Opioid Product Premiums In 2020, a great number of darknet markets (DNMs) went dark by either voluntarily closing, exit scamming, or getting busted by global law enforcement.',
     'Despite the loss of many DNMs, the vast amount of spontaneous underground markets saw record revenues in 2020.',
     'However, the numerous amounts of DNMs disappearing have led to an increased demand for opioids.',
     'A new report from the Australian National University (ANU) shows that the effects of global law enforcement takedowns and other types of DNM closures have decreased the accessibility for opioids.',
     'The study by ANU was led by Emeritus Professor Roderic Broadhurst, and for 352 days the team collected data from DNMs like Berlusconi, Valhalla, Wall Street, Apollon, Empire, Dream, Nightmare, and Tochka.',
     'Broadhurst noted that DNM operators, after closing shop, simply create another illicit shop again with higher prices on opioid products.',
     'Essentially, the coronavirus pandemic lockdowns and travel restrictions fueled DNM operations.',
     '“Since the pandemic, everybody’s going online,” Professor Broadhurst said.',
     '“Markets have been boosted by increased demand by buyers and vendors able to supply or substitute products such as opioids, partly to get around problems with lockdowns, and border closures.” Covid-19 Vaccines and Vaccination Records Advertised on Darknet Markets The Covid-19 pandemic has not only put a stronghold on opioid sales, but researchers have also noticed that ostensible vaccines meant for the coronavirus are being sold on DNMs on the deep web.',
     'A number of reports have been published on this subject in the last few weeks, and just recently Kaspersky researchers published a comprehensive report on a number of coronavirus vaccine and vaccination record adverts found on darknet marketplaces.',
     'Kaspersky said that while scraping data across 15 markets they found advertisements for the top three Covid-19 vaccines.',
     'Of course, the legitimacy of the Pfizer/Biontech, Astrazeneca, and Moderna products being sold on DNMs is quite questionable, to say the least.',
     'Additionally, Kaspersky’s team found generic or unnamed Covid-19 vaccines being sold on these markets as well.',
     'Of course, Kaspersky researchers recommend not buying anything on DNMs and the research team doesn’t think it’s a good idea to purchase a so-called vaccine online either.',
     '“Over the past year, there have been a whole host of scams exploiting the COVID topic, and many of them have been successful,” Dmitry Galov, a security expert at Kaspersky said.',
     'Right now, not only are people selling vaccine doses, but they’re also selling vaccination records— pieces of paper that can help you travel freely.',
     'It’s important for users to be cautious of any ‘deal’ related to the pandemic, and, of course, it’s never a good idea to buy a vaccine from the darknet.”        ‘Breaking Bad:’ Check Point Research Notices an Uptrend in the Number of People Looking for DNM Work Recent research reports have found that Covid-19 has also fueled DNMs in another way.',
     'A study published last week by Check Point Research called “Breaking bad: desperate job seekers turn to the Darknet and hacking forums for opportunities,” explains how people are looking for employment from DNM operators.',
     'Check Point has noticed a rise in the number of people engaged in looking for jobs on DNMs.',
     'Unemployment levels have risen and Check Point documented that there are continued lockdowns in Italy, Greece, Spain, France, and other nations.',
     '“Within this backdrop, Check Point Research noticed a growing trend that began towards the end of 2020 and continues to develop in 2021– people are turning to the darknet and various hacking forums to offer their services and availability for work for any kind of work available, including less than legitimate roles,” the study details.',
     '“From the beginning of 2021, we noticed that there was an increase in the number of individuals taking the initiative to send out ads seeking work.',
     'In fact, we started observing 10-16 new ads being placed monthly in select hacking forums,” the Check Point Research study adds.',
     'Check Point highlights one individual who is looking for employment and says: This jobseeker, who says he is in a difficult financial situation, is offering to work in the sphere of ‘financial cash-out, DDOS, security’ and promises not to ask too many stupid questions.',
     'The researchers surmise that because of “desperation,” jobseekers are willing to participate in DNM activities even if they are illegal.',
     'Check Point Research says that countries need to realize that the shadow economy is growing under the backdrop of the gloomy economy.',
     '“From our research,” Check Point concludes.',
     '“You can see that people from different parts of the world are offering their abilities, expertise, and availability on these platforms.',
     'As countries grapple with the realities of a struggling global economy, it is important they also turn their attention to the underground job marketplace that is growing.” What do you think about the effects Covid-19 has had on darknet markets (DNMs)?',
     'Let us know what you think about this subject in the comments section below.',
     "Domestic and Foreign Buyers Acquired a 'Tokenized' Apartment in a Spanish City by Paying With Ethereum                                        NEWS | 54 mins ago                                 Iconic 'Obey' Street Artist Shepard Fairey to Auction an NFT Mural on Superrare                                        NEWS | 5 hours ago                             Tags in this story                   ANU, Australian National University, Bitcoin, Check Point Research, Coronavirus, COVID-19, Covid-19 vaccination records, Covid-19 vaccines, Cryptocurrency, Darknets, Deep Web, Jobseekers, Kaspersky researchers, Opioid Premiums, Opioids, pandemic, Professor Broadhurst    Image Credits: Shutterstock, Pixabay, Wiki Commons Purchase Bitcoin without visiting a cryptocurrency exchange.",
     'Buy BTC and BCH here.',
     'Show comments',
     'Central Bank of Nigeria Official Reiterates Nigerian Residents Free to Trade Cryptos: Directive Only Applies to Banking Sector                        A senior executive with Nigeria’s central bank, Adamu Lamtek, has reportedly denied that his institution had placed restrictions on the use of cryptocurrencies.',
     'Instead, Lamtek, who is the deputy governor responsible for corporates services at the Central Bank of Nigeria (CBN), says his organisation’s directive only applies to the banking sector.',
     'Prohibition Only Applies to Banks The remarks by Lamtek, who spoke on behalf of the CBN governor Godwin Emefiele, follow the central bank’s February 5, 2021, directive which prohibited banks from facilitating crypto-related transactions.',
     'Immediately following the announcement, financial institutions began to comply with the directive.',
     'Since then volumes on centralized crypto exchanges have plunged while reports suggest that volumes on peer to peer platforms are on the rise.',
     'In the meantime, before the latest communication from the CBN, rumours circulated that authorities would soon clampdown on peer to peer trading like Binance’s P2P platform.',
     'However, Lamtek, who spoke at a seminar for the country’s finance correspondents and business editors, clarified that the CBN directive did not in fact ban cryptocurrency trading.',
     'Meanwhile, after the CBN ordered banks to stop facilitating crypto transactions, many leaders from Nigeria’s blockchain and fintech space reacted by expressing their opposition.',
     'One of these leaders, Adedayo Adebajo, the Managing Director at Jelurida Africa DLT, tells Bitcoin.com News that after extensive deliberations, many were in favour of regulating the industry rather than a total ban.',
     'CBN Has No Power to Stop P2P Trading As previously reported by Bitcoin.com News, this CBN prohibition has been opposed by non-crypto parties that include members of the Nigerian Senate as well as the country’s Vice President, Yemi Osinbajo.',
     'Now, according to Adebajo, this latest CBN stance on cryptocurrencies could well be a response to the voices that are opposed to a complete ban.',
     'Adebajo said: Let’s not forget that the initial attempt only scratched the surface a bit as trading continues on the peer to peer platforms despite the CBN directives.',
     'Allowing trades to continue can’t be said to be subject to CBN’s power anymore as the financial sector has partially been decentralized.',
     'While the CBN is not in a position to stop peer-to-peer trading, Adebajo however believes this latest clarification to be “a way forward one which we all hope with continuous effort to regulate the industry, adoption will be broad and fast.” What do you make of the CBN’s clarification that crypto trading is not banned?',
     'Tell us what you think in the comments section below.',
     'Nigeria to Pay 1.2 Cents for Each Dollar Remitted to the Country to Combat Cryptocurrency Use                                        EMERGING MARKETS | Mar 9, 2021                                Nigerian Vice President Yemi Osinbajo Contradicts Central Bank, Says Cryptocurrencies Must Be Regulated and Not Prohibited                                        EMERGING MARKETS | Feb 26, 2021                             Tags in this story                   cbn, CBN crypto ban, Central Bank, Centralized Exchanges, Crypto regulation, Godwin Emefiele, peer to peer trading    Image Credits: Shutterstock, Pixabay, Wiki Commons Purchase Bitcoin without visiting a cryptocurrency exchange.',
     'Buy BTC and BCH here.',
     'Disclaimer: This article is for informational purposes only.',
     'It is not a direct offer or solicitation of an offer to buy or sell, or a recommendation or endorsement of any products, services, or companies.',
     'Bitcoin.com does not provide investment, tax, legal, or accounting advice.',
     'Neither the company nor the author is responsible, directly or indirectly, for any damage or loss caused or alleged to be caused by or in connection with the use of or reliance on any content, goods or services mentioned in this article.',
     'Read disclaimerShow comments',
     "Bytetree Founder Believes Grayscale Should Lower Its 'Unrealistically High' Fee: Warns of Possible 'Systemic Risk'                        The founder of Bytetree Asset Management, Charlie Morris, has suggested the reduction of the GBTC management fee as one way Grayscale Investment can ease the current selling pressure and possibly reduce the discount on the shares.",
     'Morris says if no action is taken, “the selling pressure will build” and “could morph into a systemic risk.” Recent GBTC Buyback Insignificant On the other hand, Morris, who is also the chief investments officer (CIO) at Bytetree, thinks Grayscale’s recent $250 million GBTC may be too insignificant to make a difference.',
     'However, in his Bytetree Market Health Update, Morris does acknowledge that 2020 was a year of “record inflows” for Grayscale Investment.',
     'During this period, the premium on GBTC shares averaged 30% while the value of assets under management (AUM) soared to over $30 billion.',
     'However, when the premium began to ease, the inflows into the trust similarly began to slow down.',
     'This subsequently led to the premium on the GBTC shares turning into a discount.',
     'Yet as Morris explains, this discount might still widen as the supply of GBTC shares continues to outstrip the demand for the same.',
     'In his analysis, Morris said: The trouble is, there is nothing in place to prevent the discount from widening further.',
     'Grayscale has announced a $250m share purchase, which is great, but it is too small to make a difference within a $33 bn fund.',
     'To be taken seriously, they need to pledge to buy back that quantity each week.',
     'Threat From Competitors Still, as Morris notes, Grayscale Investments may not be in a position to buy-back more GBTC shares due to “their legal structure (which) prevents this.” Similarly, the CIO says he is “not entirely sure of the benefits” of Grayscale’s plan “to launch an ETF that invests in GBTC in a feeder structure of sorts.” The CIO goes on to explain: “When Osprey (OBTC) offers Bitcoin exposure at 0.49% per annum, you might think GBTC would drop their fees down from 2%; they are unrealistically high given the new world of competition.” While Morris touts ETFs as a better alternative, he, however, acknowledges that the only reason investors will/may stay in GBTC is capital gains taxes.',
     'Meanwhile, Morris concludes his report by saying he “very much hopes they (Grayscale) restore investor confidence without delay.” Do you agree that lowering fees will help Grayscale’s cause?',
     'You can share your views in the comments section below.',
     'Fidelity Applies to Launch Bitcoin ETF Amid Accelerating Institutional Demand for BTC                                        MARKETS AND PRICES | 11 hours ago                                JMP Securities Expects $1.5 Trillion to Flow Into Bitcoin as Retail Wealth Management Industry Follows Morgan Stanley to Offer BTC to Clients                                        MARKETS AND PRICES | 1 day ago                             Tags in this story                   AUM, bitcoin etf, BTC exposure, bytetree, capital gains tax, GBTC, GBTC premium, Grayscale Investment Trust, management fee    Image Credits: Shutterstock, Pixabay, Wiki Commons Spot-markets for Bitcoin, Bitcoin Cash, Ripple, Litecoin and more.',
     'Start your trading here.',
     'Disclaimer: This article is for informational purposes only.',
     'It is not a direct offer or solicitation of an offer to buy or sell, or a recommendation or endorsement of any products, services, or companies.',
     'Bitcoin.com does not provide investment, tax, legal, or accounting advice.',
     'Neither the company nor the author is responsible, directly or indirectly, for any damage or loss caused or alleged to be caused by or in connection with the use of or reliance on any content, goods or services mentioned in this article.',
     'Read disclaimerShow comments',
     "NFT Criticism Heightens: Skeptic Calls Tech a 'House of Cards,' Claims NFTs Will be 'Broken in a Decade'                        On the week of March 7 through the 13th, the term “NFT” touched an all-time record high in terms of popular search queries according to Google Trends (GT).",
     'The highest score is 100 and this week, GT shows interest has dwindled a hair down to 97.',
     'Meanwhile, while the NFT hype affects the cryptocurrency and blockchain community, not everyone is thrilled with this technology.',
     'Some critics think NFT tech needs to be more robust in order for collectibles to last more than a decade without them disappearing.',
     'The Rift Between NFT Hype and NFT Criticism Over the last few weeks, the NFT craze has jumped to new levels as people have sold artwork for millions of dollars, unique audio recordings of farts have been tokenized, and even blockchain-backed signature dance moves are NFTs.',
     'And of course, by now everyone has heard about Beeple, the digital artist who sold an NFT via Christie’s auction house for a whopping $69 million.',
     'This sale propelled NFT mania to new heights, and Christie’s auction invoked Sotheby’s to partner with an artist named Pak.',
     'The mysterious digital artist Pak was selling an NFT on Makersplace called “Metarift.” An NFT art collector named “888” placed a bid on Metarift for $888,888 on March 19.',
     'The following day, the bid had been upped to $904,413 by a bidder named “Danny.” The auction ended on Pak’s latest NFT artwork sale on Saturday at 12:30 p.m. (PST) with Danny’s winning bid.',
     'As of 3:30 p.m. (EST), Pak’s “Metarift” NFT’s highest bid was for $904,413 by a bidder named Danny who now owns the NFT.',
     'Despite all the mega-auctions and all the celebrities joining in on the fun, non-fungible token (NFT) assets have come under a lot of criticism.',
     'Bitcoin.com’s newsdesk discussed the topic of NFT immutability which has been the industry’s biggest weak point so far.',
     'In that editorial tokenized NFT tweets were deleted and artwork was replaced with other images.',
     'On March 17, the Twitter user Jonty Wareing told his 6,932 followers how non-fungible tokens reference the media people are purchasing today.',
     'His first tweet went viral with more than 27,000 likes and close to 10,000 retweets.',
     'Out of curiosity I dug into how NFT\'s actually reference the media you\'re "buying" and my eyebrows are now orbiting the moon — Jonty Wareing (@jonty) March 17, 2021  File Vanishes from the Host ‘Your Token Is Now Worthless’ He discussed a sale of an original Beeple sold on Nifty Gateway, and detailed how the Beeple JSON file was hosted on Nifty Gateway’s servers.',
     'Wareing then explained how the NFT is also hosted on Cloudinary.',
     '“So if Nifty goes bust, your token is now worthless,” Wareing said.',
     '“It refers to nothing.',
     'This can’t be changed,” he added.',
     'Then Wareing also talked about files that are using the Interplanetary File System (IPFS) as well.',
     'In this tweet, he specifically discussed the Beeple sold by Christie’s which is connected to IPFS.',
     '“So, well done for referring to IPFS – it references the specific file rather than a URL that might break,” Wareing continued and showed the metadata link to an IPFS gateway run by Makersplace.',
     'But if something happened to Makersplace, then once again, the link may be useless over time.',
     '“You might say ‘Just refer to the IPFS hash in both places’” Wareing added.',
     'But further scorned that “IPFS only serves files as long as a node in the IPFS network intentionally keeps hosting it.” Wareing also said: Which means when the startup [that] sold you the NFT goes bust, the files will probably vanish from IPFS too.',
     'Twitter Account Dubbed Check My NFT Discovers Current NFT Flaws in the Wild Moreover, he also tweeted about an example of the issue happening today and said “NFT’s are built on an absolute house of cards constructed by the people selling them.” He claims the tech will be “broken” within a decade.',
     'The Twitter account called “Check My NFT” (@checkmynft) has also been tweeting about a bunch of missing art, and inaccessible NFT assets.',
     'The Twitter account called “Check My NFT” has been investigating the permanence issue with non-fungible token (NFT) assets.',
     'In our last report, Check My NFT explained that metadata, URLs, and image files “should be backed by an immutable and permanent storage provider.” One way of doing this is leveraging IPFS2Arweave.com, which utilizes a project called Arweave.',
     'IPFS2Arweave.com leverages the Arweave network to store the files and also pin the NFT data to the Interplanetary File System.',
     'The IPFS + Arweave solution claims to solve the permanence problem by using Arweave for storage and pinning the NFT data to the Interplanetary File System.',
     '“IPFS is a popular P2P network for sharing data in a decentralized way, but it lacks one feature: Permanence.',
     'Content on the IPFS network can disappear.',
     'If no-one hosts the data, it could be lost forever,” the Arweave website details.',
     '“The Arweave blockchain can now store and pin files onto IPFS and keep them available permanently.',
     'This simple API takes an IPFS Hash, stores the data onto Arweave, and pins it to IPFS.” It will be interesting to see if NFT projects take the recent scrutiny seriously, and leverage immutable and permanent storage as Check My NFT and Arweave recommends.',
     'Projects going forward may work toward building stronger permanence models.',
     'Projects that have already minted million-dollar NFT works might want to make sure the non-fungible token assets are truly deemed immutable.',
     'Meanwhile, other blockchain NFT solutions believe their distributed ledger protocol alone is reliable enough to preserve an NFT for centuries to come.',
     'Much of this NFT debate will still require the test of time or exposure to vulnerabilities.',
     'What do you think about the NFT permanence problem?',
     'Let us know what you think about this subject in the comments section below.',
     'Tesla Begins Accepting Bitcoin — Elon Musk Says BTC Payments Will Not Be Converted to Fiat Currency                                        FEATURED | 2 days ago                                Mystery Whale Moves 20 Bitcoin Block Rewards from 2010, Entity Moved 10,000 BTC Since Last Year                                        FEATURED | 2 days ago                             Tags in this story                   Arweave, Beeple, Check My NFT, Disappearing NFTs, Interplanetary File System, IPFS gateway, IPFS hash, IPFS Hosting, Jonty Wareing, JSON file, Makersplace, NFT art, NFT Criticism, NFT sales, NFTs, Nifty Gateway, Non-fungible token (NFT), Non-fungible tokens, Pak    Image Credits: Shutterstock, Pixabay, Wiki Commons, Twitter, @checkmynft, Arweave, IPFS+Arweave, Pak, Metarift, Makersplace,  Use Bitcoin and Bitcoin Cash to play online casino games here.',
     'Disclaimer: This article is for informational purposes only.',
     'It is not a direct offer or solicitation of an offer to buy or sell, or a recommendation or endorsement of any products, services, or companies.',
     'Bitcoin.com does not provide investment, tax, legal, or accounting advice.',
     'Neither the company nor the author is responsible, directly or indirectly, for any damage or loss caused or alleged to be caused by or in connection with the use of or reliance on any content, goods or services mentioned in this article.',
     'Read disclaimerShow comments',
     'Tether Launches Stablecoin Token on Bitcoin Cash via Simple Ledger Protocol                        Tether, the world’s most popular stablecoin, will be using the Simple Ledger Protocol built upon the Bitcoin Cash network for issuing its tokens.',
     'Among other things, this means that millions of Bitcoin.com Wallet users will now be able to send and receive USDT via SLP tokens right within the app.',
     'Also Read: Paypal Pullout Prompts Pornhub to Add Tether as Payment Option for Its Adult Models Tether Joins The SLP Ecosystem Tether, the venture behind the largest stablecoin by market cap, has announced the launch of its token on Bitcoin Cash via the Simple Ledger Protocol (SLP).',
     'USDT has a capitalization of over $5.5 billion, with the majority of tokens running on Ethereum as of today.',
     'Other cryptocurrency networks supported by Tether include Algorand, EOS, Liquid Network, Omni and most notably Tron.',
     'Tether’s launch on SLP underlines its growing role within the cryptocurrency ecosystem.',
     'The protocol is an easy to use, robust and extensible token management system that allows anyone to create tokens on the Bitcoin Cash network in a permissionless way.',
     'It has helped the development of an ecosystem for BCH that replicates the variety of ERC20 tokens on the ETH network.',
     '“A key strength of Tether is that it is underpinned by a rich diversity of different blockchains,” said Paolo Ardoino, CTO at Tether.',
     '“Our latest collaboration with Bitcoin Cash will provide Tether with a variety of benefits.',
     'We expect the adoption after launch to be pretty easy for any integrator.',
     'The launch will also support more applications on the Bitcoin Cash chain, with Tether facilitating payment for these applications.” The Simple Ledger Protocol SLP tokens can easily be created, traded, and managed on the Bitcoin Cash blockchain within seconds, while costing users only fractions of a penny for each transaction.',
     'All transactions are recorded onchain, and custom token behavior can be defined using a bitcoin Script development language.',
     'Following a recent upgrade, millions of Bitcoin.com Wallet users have access to SLP tokens right in the app.',
     'With built-in token management support, users can send, receive, and store a variety of tokens that represent anything from dollar-pegged stablecoins to virtual gaming assets and company loyalty points, with the private keys held only by the user.',
     'SLP Tokens in the Bitcoin.com Wallet “It’s extremely exciting to hear that the world’s biggest stablecoin will be using the Bitcoin Cash Blockchain and that the millions of Bitcoin.com wallet holders will be able to send and receive Tether using SLP tokens,” said Roger Ver, Executive Chairman of Bitcoin.com.',
     'What do you think about Tether launching on SLP?',
     'Share your thoughts in the comments section below.',
     'Images courtesy of Shutterstock.',
     'Verify and track bitcoin cash transactions on our BCH Block Explorer, the best of its kind anywhere in the world.',
     'Also, keep up with your holdings, BCH and other coins, on our market charts at Bitcoin.com Markets, another original and free service from Bitcoin.com.',
     'Tokenized Jack Dorsey Tweet Sells for $2.9M, Funds From NFT Auction Will Be Sent to Charity                                        BLOCKCHAIN | 3 days ago                                Atari to Launch a Crypto-Fueled Gaming Platform in the Ethereum Metaverse Decentraland                                        BLOCKCHAIN | Mar 10, 2021                             Tags in this story                   BCH, bitcoin cash, Simple Ledger Protocol, SLP, Stablecoin, Tether, Token, USDT    Use Bitcoin and Bitcoin Cash to play online casino games here.',
     'Show comments',
     'Thai Central Bank Declares Baht-Pegged Stablecoin Illegal in Violation of Currency Law                        Thailand’s central bank has declared a stablecoin pegged to the Thai Baht illegal, citing violation of the country’s currency law.',
     'The Bank of Thailand (BOT) is currently working on a regulatory framework for stablecoins.',
     'Thailand Bans Baht-Pegged Stablecoin The Bank of Thailand, the country’s central bank, issued a press release Wednesday declaring a stablecoin called the Thai Baht Digital (THT) illegal.',
     '“It has come to the BOT’s attention that a new stablecoin, THT, has been created abroad on the Terra platform.',
     'One unit of THT is denominated in and valued at one Thai baht,” the announcement describes.',
     ...]




```python
#Get 2000 random sentences

Random_Sentences = []
Unique_Sentences = []
pick = 2000
i = 0

while i < (pick):
    Random_Sentences.extend(random.sample(All_Sentences, 1))
    Unique_Sentences = set(Random_Sentences)
    i = len(Unique_Sentences)


Random_Sentences = list(Unique_Sentences)
```


```python
#Check random sentences for issues (duplicates)

#len(Random_Sentences)

Random_Sentences[1000:1020]
```




    ['Mechanisms Built Specifically to Curb Volatility The high returns are a result of Axion’s innovative new approach to crypto wealth generation.',
     '“Poynt OS lays the foundation for developers and merchants to collaborate and drive limitless growth.',
     'On Saturday, November 24, one whale moved 44,000 BTC ($314 million) in a single transaction according to the Twitter account Whale Alert.',
     'Mike Novogratz’s Bitcoin Prediction Galaxy Digital CEO Michael Novogratz said on Tuesday that bitcoin and gold have more room to grow and will continue to rise due to global “liquidity pump,” afforded by governments’ stimulus packages, coupled with an influx of retail investors.',
     'Live footage of tulip trust bonded courier pic.twitter.com/eBH9w2nG0K — hodlonaut🌮⚡🔑 (@hodlonaut) December 8, 2019  ‘Extraordinary Claims Require Extraordinary Evidence’ Now many Wright and BSV supporters believe that the coins will be returned to Wright on the first of the new year.',
     'The famed investor founded Duquesne Capital in 1981 and closed the fund in August 2010.',
     'One of the photos featured in Tencent’s article also shows a hospital staff member holding a card with 144.75 yuan on it and three offline payments available.',
     'At the moment, the Menu.cash service is free, but the founder of the project has noted that at some point down the line it will take a percentage per order.',
     'HTC Exodus 1 and later the more affordable Exodus 1s devices have a built-in hardware wallet that stores major coins and tokens.',
     'Only a black swan market rout will be able to test whether or not Yearn’s ethereum liquidity vault strategy is strong enough.',
     'The U8 is a bit louder too as the unit’s fan has a noise level of around 76dB.',
     'If today’s prices are partly determined by yesterday’s prices, and yesterday’s by those of the day before yesterday, etc., is not the regression simply pushed back infinitely, and part of the deter\xadmination of prices thus left unexplained?',
     'Traders have noticed a number of “filling the gap” or “closing the gap” scenarios throughout 2019 and 2020.',
     'According to data from Cryptowhale, the firm had bought over 12 million XRP tokens on New Year’s eve.',
     'The two laser engraved sheets allow owners to store 12,18 and 24-word seeds or other types of encrypted secrets.',
     'Economists believe that the Fed’s low interest rates make savings an unattractive goal and with extremely low rates the idea of savings doesn’t make sense.',
     'For the people that actually didn’t pay or underpaid, they’re trying to scare you into paying what you actually do owe.”   A Pretty Penny for Crypto Help Commenters on the thread happily joined the venting after u/IRS_throwaway_BT opened the floor for catharsis and commiseration via the post.',
     'When one pizza can cost half the salary of a government teacher, that’s not surprising.',
     'Overthrowing the violent fiat monopoly on money for the greater good of humankind is indeed a noble pursuit.',
     'DFI will determine the appropriate licensing for the company to continue operations, if applicable,” the website clarifies.']




```python
#merge list of 2000 sentences to data frame for export
dfSent = DataFrame(Random_Sentences,columns=['Sentence'])
```


```python
#check for duplicates
dfSent[dfSent.Sentence.duplicated(keep="last")]
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
      <th>Sentence</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
</div>




```python
#Export to CSV to mannually classify
dfSent.to_csv("RandomSentences/Sentences2.csv",sep='\t')
```

## Rating guidance for sentence sentiment classification

Multiple sentiments ->  Add all positive, negative and neutral adjectives, adverbs. The category with the most will be the overall classification. https://monkeylearn.com/sentiment-analysis/

Sarcasm -> Classify as positive statements. Sarcasm contains positive words about a negative situation. Determining negative situations is beyond the scope of this assignment. https://www.aclweb.org/anthology/D13-1066.pdf

No clear sentiment -> neutral


```python
#import manually classified sentences
dfSent = pd.read_csv("RandomSentences/SentencesTab.csv", sep='\t', encoding = 'unicode_escape')
```


```python
dfSent
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
      <th>Ct</th>
      <th>Class</th>
      <th>Sentence</th>
      <th>Assigned</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>positive</td>
      <td>With aspirations to become one of the foremost international hubs for finance, we must keep up with the rapid technological changes taking place a...</td>
      <td>Troy</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>negative</td>
      <td>These and other Orwellian nightmares are possibilities.</td>
      <td>Troy</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>neutral</td>
      <td>According to the report, the transformation-related effects were largely in line with the expectations.</td>
      <td>Troy</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>neutral</td>
      <td>Zug-registered Seba Crypto and Zurich-based Sygnum became Switzerlands first regulated crypto banks.</td>
      <td>Troy</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>neutral</td>
      <td>An example of people moving money into safe haven assets because of the economic forecast shows gold touching an all-time high of $1,450 per ounce.</td>
      <td>Troy</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>1995</th>
      <td>1995</td>
      <td>neutral</td>
      <td>These Tools Can Give Anyone the Skills to Issue NFT Assets                                        TECHNOLOGY | Mar 17, 2021                       ...</td>
      <td>Praveen</td>
    </tr>
    <tr>
      <th>1996</th>
      <td>1996</td>
      <td>neutral</td>
      <td>What do you think about Bitcoin.com Exchange listing ESH?</td>
      <td>Praveen</td>
    </tr>
    <tr>
      <th>1997</th>
      <td>1997</td>
      <td>neutral</td>
      <td>Moreover, there are also 148,849 services allowing people to buy (and sell) cryptocurrencies, such as newspaper and mall kiosks.</td>
      <td>Praveen</td>
    </tr>
    <tr>
      <th>1998</th>
      <td>1998</td>
      <td>neutral</td>
      <td>For instance, have you seen our Tools page?</td>
      <td>Praveen</td>
    </tr>
    <tr>
      <th>1999</th>
      <td>1999</td>
      <td>negative</td>
      <td>Your stimulus cost us (taxpayers) $831 billion for one of the slowest recoveries in history while doubling the national debt, an individual said...</td>
      <td>Praveen</td>
    </tr>
  </tbody>
</table>
<p>2000 rows × 4 columns</p>
</div>




```python
#Review classification balance
dfSent.Class.value_counts()
```




    positive    744
    neutral     700
    negative    556
    Name: Class, dtype: int64




```python
#Review values present in classified sentiments
dfSent.Class.unique()
```




    array(['positive', 'negative', 'neutral'], dtype=object)




```python
# Prediction cases
#neutral
text1 = 'Rothbard writes: “Suppose the economy has a supply of 10,000 gold ounces, and counterfeiters [the state]…pump in 2000 ‘ounces’ more.'
text2 = 'The group revealed: Nearly 100 countries carried out automatic exchange of information in 2019, enabling their tax authorities to obtain data on 84 million financial accounts held offshore by their residents, covering total assets of EUR 10 trillion.'
text3 = 'The problem arose while he was in Iran as he says he simply looked at his account but performed no transactions or made no transfers during the checkup.'
#positive
text4 = '“Currently Bitcoin appears to be struggling to stay above its support, however, it is showing some bullish divergences so I’d expect a rally in the short term,” Pelecanos noted.'
text5 = 'Data collected by blockchain forensics company Chainalysis shows a significant increase of volume in the first half of the year.'
text6 = 'Already, Mastercard has announced it will be supporting cryptocurrencies in its payments network.'
#negative
text7 = 'Looming US Real Estate Crisis - Freddie Mac Warns of Housing Market Uncertainty, Homebuilder sentiment Drops 58%                        U.S. real estate agents and lenders are bracing for the biggest housing crash in over a decade.'
text8 = 'After a rogue employee deleted President Donald Trump’s account two years ago, the company limited access to national leaders’ accounts to a much smaller number of people.'
text9 = 'A small number of people are starting to focus on the economic situation and the possible destructive aftermath the finance world will see.'
text10 = 'If they eventually dump on you, there is no way you will recover.'
```

## Sentiment Classifiers


```python
#import vectorizer
from sklearn.feature_extraction.text import TfidfVectorizer

vectorizer = TfidfVectorizer(use_idf=True, norm="l2", stop_words="english", max_df=0.7)
X = vectorizer.fit_transform(dfSent.Sentence)
y = dfSent.Class

from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.20, random_state=0)
```


```python
#vectorize prediction texts
new_texts = [text1, text2, text3, text4, text5, text6, text7, text8, text9, text10]

X_new = vectorizer.transform(new_texts)
```


```python
#check data groupings
X_train.shape, X_test.shape, y_train.shape, y_test.shape
```




    ((1600, 7593), (400, 7593), (1600,), (400,))




```python
#function for classification report
from sklearn.metrics import classification_report, confusion_matrix

def train_test(X_train, X_test, y_train, y_test, classifier):
    classifier.fit(X_train, y_train)
    pred = classifier.predict(X_test)

    print("Train score: {:.2f}".format(classifier.score(X_train, y_train)))
    print("Test score: {:.2f}\n".format(classifier.score(X_test, y_test)))
    print("Classification report:\n{}".format(classification_report(y_test, pred, zero_division=0)))
    print(confusion_matrix(y_test,pred))

    return classifier
```

## KNN Model


```python
#KNN model
from sklearn.neighbors import KNeighborsClassifier
from sklearn.model_selection import cross_val_score


score_max = 0                      
for param in [1, 3, 5, 7, 9, 11]:
    model = KNeighborsClassifier(n_neighbors=param)
    scores = cross_val_score(model, X_train, y_train, cv=10)
    print("k = {}: {}\n{:.3f}, {:.3f}\n".format(param, scores, scores.mean(), scores.std()))

    if scores.mean() > score_max:
        score_max = scores.mean()
        param_best = param         

print("Highest score : {:.3f} when k = {}".format(score_max, param_best))

print("k = {}".format(param_best))
knn = KNeighborsClassifier(n_neighbors=param_best)
knn = train_test(X_train, X_test, y_train, y_test, knn)
summary["KNN"] = round(knn.score(X_test, y_test), 3)
```

    k = 1: [0.35625 0.34375 0.3375  0.34375 0.35    0.34375 0.3375  0.3375  0.325
     0.35   ]
    0.343, 0.008

    k = 3: [0.34375 0.34375 0.34375 0.34375 0.3375  0.3375  0.34375 0.34375 0.33125
     0.35   ]
    0.342, 0.005

    k = 5: [0.375   0.35    0.3375  0.3375  0.3375  0.35625 0.34375 0.3375  0.375
     0.35   ]
    0.350, 0.014

    k = 7: [0.40625 0.3625  0.33125 0.3875  0.35625 0.44375 0.375   0.3625  0.39375
     0.35   ]
    0.377, 0.031

    k = 9: [0.44375 0.3625  0.35    0.3875  0.4125  0.41875 0.40625 0.39375 0.43125
     0.3625 ]
    0.397, 0.030

    k = 11: [0.475   0.3625  0.3375  0.3875  0.40625 0.375   0.4     0.4     0.38125
     0.34375]
    0.387, 0.037

    Highest score : 0.397 when k = 9
    k = 9
    Train score: 0.46
    Test score: 0.42

    Classification report:
                  precision    recall  f1-score   support

        negative       0.50      0.13      0.21        99
         neutral       0.40      0.90      0.56       156
        positive       0.61      0.10      0.17       145

        accuracy                           0.42       400
       macro avg       0.50      0.38      0.31       400
    weighted avg       0.50      0.42      0.33       400

    [[ 13  85   1]
     [  7 141   8]
     [  6 125  14]]



```python
knn_pred = knn.predict(X_new)
```

## Logistic Regression Model


```python
from sklearn.linear_model import LogisticRegression

lr = LogisticRegression()

summary = {}

lr = train_test(X_train, X_test, y_train, y_test, lr)

summary["Logistic Regression"] = round(lr.score(X_test, y_test), 3)
```

    Train score: 0.97
    Test score: 0.53

    Classification report:
                  precision    recall  f1-score   support

        negative       0.55      0.42      0.48        99
         neutral       0.55      0.47      0.51       156
        positive       0.50      0.66      0.57       145

        accuracy                           0.53       400
       macro avg       0.53      0.52      0.52       400
    weighted avg       0.53      0.53      0.52       400

    [[42 25 32]
     [19 73 64]
     [16 34 95]]



```python
#predictions based on logistic Regression model
lr_pred = lr.predict(X_new)
```

## Multinomial Naive Bayes


```python
from sklearn.naive_bayes import MultinomialNB

mnb = MultinomialNB()

mnb = train_test(X_train, X_test, y_train, y_test, mnb)

summary["Multinomial Naive Bayes"] = round(mnb.score(X_test, y_test), 3)
```

    Train score: 0.92
    Test score: 0.52

    Classification report:
                  precision    recall  f1-score   support

        negative       0.67      0.22      0.33        99
         neutral       0.60      0.42      0.50       156
        positive       0.46      0.82      0.59       145

        accuracy                           0.52       400
       macro avg       0.58      0.49      0.47       400
    weighted avg       0.57      0.52      0.49       400

    [[ 22  22  55]
     [  7  66  83]
     [  4  22 119]]



```python
mnb_pred = mnb.predict(X_new)
```

## Linear Support Vector Machines


```python
from sklearn.svm import LinearSVC

svm = LinearSVC(C=1)

svm = train_test(X_train, X_test, y_train, y_test, svm)

score_max = 0
for param in [0.01, 0.03, 0.1, 0.3, 1, 3, 10]:
    model = LinearSVC(C=param)
    scores = cross_val_score(model, X_train, y_train, cv=5)
    print("C = {}: {}\n{:.3f}, {:.3f}\n".format(param, scores, scores.mean(), scores.std()))

    if scores.mean() > score_max:
        score_max = scores.mean()
        param_best = param

print("Highest score : {:.3f} when C = {}".format(score_max, param_best))
```

    Train score: 1.00
    Test score: 0.52

    Classification report:
                  precision    recall  f1-score   support

        negative       0.43      0.44      0.44        99
         neutral       0.57      0.53      0.54       156
        positive       0.54      0.57      0.55       145

        accuracy                           0.52       400
       macro avg       0.51      0.51      0.51       400
    weighted avg       0.52      0.52      0.52       400

    [[44 27 28]
     [31 82 43]
     [27 36 82]]
    C = 0.01: [0.40625  0.390625 0.41875  0.409375 0.440625]
    0.413, 0.016

    C = 0.03: [0.440625 0.39375  0.45     0.425    0.446875]
    0.431, 0.021

    C = 0.1: [0.4625   0.453125 0.496875 0.490625 0.4875  ]
    0.478, 0.017

    C = 0.3: [0.478125 0.484375 0.48125  0.49375  0.515625]
    0.491, 0.014

    C = 1: [0.471875 0.465625 0.4875   0.496875 0.496875]
    0.484, 0.013

    C = 3: [0.45625  0.45     0.484375 0.4875   0.490625]
    0.474, 0.017

    C = 10: [0.4375   0.453125 0.471875 0.490625 0.490625]
    0.469, 0.021

    Highest score : 0.491 when C = 0.3



```python
print("C = {}".format(param_best))
svm = LinearSVC(C=param_best)
svm = train_test(X_train, X_test, y_train, y_test, svm)
```

    C = 0.3
    Train score: 0.98
    Test score: 0.53

    Classification report:
                  precision    recall  f1-score   support

        negative       0.47      0.45      0.46        99
         neutral       0.56      0.49      0.52       156
        positive       0.53      0.62      0.57       145

        accuracy                           0.53       400
       macro avg       0.52      0.52      0.52       400
    weighted avg       0.53      0.53      0.53       400

    [[45 27 27]
     [27 76 53]
     [23 32 90]]



```python
summary["Linear SVMs"] = round(svm.score(X_test, y_test), 3)
```


```python
lsvm_pred = svm.predict(X_new)
```

## Kernelized Support Vector Machines


```python
from sklearn.svm import SVC

ksvm = SVC(C=1, kernel="rbf", gamma="scale")

ksvm = train_test(X_train, X_test, y_train, y_test, ksvm)

score_max = 0
for param in [0.01, 0.03, 0.1, 0.3, 1, 3, 10]:
    model = SVC(C=param, kernel="rbf", gamma="scale")
    scores = cross_val_score(model, X_train, y_train, cv=5)
    print("C = {}: {}\n{:.3f}, {:.3f}\n".format(param, scores, scores.mean(), scores.std()))

    if scores.mean() > score_max:
        score_max = scores.mean()
        param_best = param

print("Highest score : {:.3f} when C = {}".format(score_max, param_best))
```

    Train score: 1.00
    Test score: 0.51

    Classification report:
                  precision    recall  f1-score   support

        negative       0.63      0.24      0.35        99
         neutral       0.73      0.34      0.46       156
        positive       0.44      0.88      0.59       145

        accuracy                           0.51       400
       macro avg       0.60      0.49      0.47       400
    weighted avg       0.60      0.51      0.48       400

    [[ 24  10  65]
     [  7  53  96]
     [  7  10 128]]
    C = 0.01: [0.375    0.375    0.375    0.371875 0.375   ]
    0.374, 0.001

    C = 0.03: [0.375    0.375    0.375    0.371875 0.375   ]
    0.374, 0.001

    C = 0.1: [0.375    0.375    0.375    0.371875 0.375   ]
    0.374, 0.001

    C = 0.3: [0.38125  0.38125  0.38125  0.378125 0.378125]
    0.380, 0.002

    C = 1: [0.44375  0.415625 0.45625  0.440625 0.45    ]
    0.441, 0.014

    C = 3: [0.478125 0.46875  0.50625  0.51875  0.4875  ]
    0.492, 0.018

    C = 10: [0.484375 0.46875  0.50625  0.521875 0.484375]
    0.493, 0.019

    Highest score : 0.493 when C = 10



```python
print("C = {}".format(param_best))
ksvm = SVC(C=param_best)
ksvm = train_test(X_train, X_test, y_train, y_test, ksvm)
```

    C = 10
    Train score: 1.00
    Test score: 0.50

    Classification report:
                  precision    recall  f1-score   support

        negative       0.46      0.40      0.43        99
         neutral       0.57      0.44      0.49       156
        positive       0.48      0.64      0.55       145

        accuracy                           0.50       400
       macro avg       0.50      0.49      0.49       400
    weighted avg       0.51      0.50      0.50       400

    [[40 24 35]
     [23 68 65]
     [24 28 93]]



```python
summary["Kernelized SVMs"] = round(ksvm.score(X_test, y_test), 3)
```


```python
ksvm_pred = ksvm.predict(X_new)
```

## Neural Networks


```python
from sklearn.neural_network import MLPClassifier

mlp = MLPClassifier(hidden_layer_sizes=(10, ), activation="relu", random_state=0)

mlp = train_test(X_train, X_test, y_train, y_test, mlp)

score_max = 0
for param in [10, 30, 100]:
    model = MLPClassifier(hidden_layer_sizes=(param, ), activation="relu", random_state=0)
    scores = cross_val_score(model, X_train, y_train, cv=10)
    print("hidden_layer_size = {}: {}\n{:.3f}, {:.3f}\n".format(param, scores, scores.mean(), scores.std()))

    if scores.mean() > score_max:
        score_max = scores.mean()
        param_best = param

print("Highest score : {:.3f} when hidden_layer_sizes = {}".format(score_max, param_best))
```

    /usr/local/lib/python3.7/dist-packages/sklearn/neural_network/_multilayer_perceptron.py:585: ConvergenceWarning: Stochastic Optimizer: Maximum iterations (200) reached and the optimization hasn't converged yet.
      % self.max_iter, ConvergenceWarning)


    Train score: 1.00
    Test score: 0.50

    Classification report:
                  precision    recall  f1-score   support

        negative       0.42      0.35      0.38        99
         neutral       0.52      0.54      0.53       156
        positive       0.53      0.56      0.54       145

        accuracy                           0.50       400
       macro avg       0.49      0.49      0.49       400
    weighted avg       0.50      0.50      0.50       400

    [[35 36 28]
     [27 85 44]
     [21 43 81]]


    /usr/local/lib/python3.7/dist-packages/sklearn/neural_network/_multilayer_perceptron.py:585: ConvergenceWarning: Stochastic Optimizer: Maximum iterations (200) reached and the optimization hasn't converged yet.
      % self.max_iter, ConvergenceWarning)
    /usr/local/lib/python3.7/dist-packages/sklearn/neural_network/_multilayer_perceptron.py:585: ConvergenceWarning: Stochastic Optimizer: Maximum iterations (200) reached and the optimization hasn't converged yet.
      % self.max_iter, ConvergenceWarning)
    /usr/local/lib/python3.7/dist-packages/sklearn/neural_network/_multilayer_perceptron.py:585: ConvergenceWarning: Stochastic Optimizer: Maximum iterations (200) reached and the optimization hasn't converged yet.
      % self.max_iter, ConvergenceWarning)
    /usr/local/lib/python3.7/dist-packages/sklearn/neural_network/_multilayer_perceptron.py:585: ConvergenceWarning: Stochastic Optimizer: Maximum iterations (200) reached and the optimization hasn't converged yet.
      % self.max_iter, ConvergenceWarning)
    /usr/local/lib/python3.7/dist-packages/sklearn/neural_network/_multilayer_perceptron.py:585: ConvergenceWarning: Stochastic Optimizer: Maximum iterations (200) reached and the optimization hasn't converged yet.
      % self.max_iter, ConvergenceWarning)
    /usr/local/lib/python3.7/dist-packages/sklearn/neural_network/_multilayer_perceptron.py:585: ConvergenceWarning: Stochastic Optimizer: Maximum iterations (200) reached and the optimization hasn't converged yet.
      % self.max_iter, ConvergenceWarning)
    /usr/local/lib/python3.7/dist-packages/sklearn/neural_network/_multilayer_perceptron.py:585: ConvergenceWarning: Stochastic Optimizer: Maximum iterations (200) reached and the optimization hasn't converged yet.
      % self.max_iter, ConvergenceWarning)
    /usr/local/lib/python3.7/dist-packages/sklearn/neural_network/_multilayer_perceptron.py:585: ConvergenceWarning: Stochastic Optimizer: Maximum iterations (200) reached and the optimization hasn't converged yet.
      % self.max_iter, ConvergenceWarning)
    /usr/local/lib/python3.7/dist-packages/sklearn/neural_network/_multilayer_perceptron.py:585: ConvergenceWarning: Stochastic Optimizer: Maximum iterations (200) reached and the optimization hasn't converged yet.
      % self.max_iter, ConvergenceWarning)
    /usr/local/lib/python3.7/dist-packages/sklearn/neural_network/_multilayer_perceptron.py:585: ConvergenceWarning: Stochastic Optimizer: Maximum iterations (200) reached and the optimization hasn't converged yet.
      % self.max_iter, ConvergenceWarning)


    hidden_layer_size = 10: [0.4875  0.3875  0.425   0.5     0.475   0.50625 0.49375 0.51875 0.46875
     0.4375 ]
    0.470, 0.039

    hidden_layer_size = 30: [0.475   0.41875 0.425   0.4625  0.48125 0.49375 0.4875  0.51875 0.4625
     0.44375]
    0.467, 0.030

    hidden_layer_size = 100: [0.4625  0.3875  0.43125 0.4875  0.50625 0.49375 0.4875  0.51875 0.45
     0.45   ]
    0.468, 0.037

    Highest score : 0.470 when hidden_layer_sizes = 10



```python
print("hidden_layer_size = {}".format(param_best))
mlp = MLPClassifier(hidden_layer_sizes=(param_best, ), random_state=0)
mlp = train_test(X_train, X_test, y_train, y_test, mlp)
```

    hidden_layer_size = 10
    Train score: 1.00
    Test score: 0.50

    Classification report:
                  precision    recall  f1-score   support

        negative       0.42      0.35      0.38        99
         neutral       0.52      0.54      0.53       156
        positive       0.53      0.56      0.54       145

        accuracy                           0.50       400
       macro avg       0.49      0.49      0.49       400
    weighted avg       0.50      0.50      0.50       400

    [[35 36 28]
     [27 85 44]
     [21 43 81]]


    /usr/local/lib/python3.7/dist-packages/sklearn/neural_network/_multilayer_perceptron.py:585: ConvergenceWarning: Stochastic Optimizer: Maximum iterations (200) reached and the optimization hasn't converged yet.
      % self.max_iter, ConvergenceWarning)



```python
summary["Neural Networks"] = round(mlp.score(X_test, y_test), 3)
```


```python
nn_pred = mlp.predict(X_new)
```

## Random Forest


```python
#could loop number of estimators for optimization
from sklearn.ensemble import RandomForestClassifier

clf=RandomForestClassifier(n_estimators=300)

clf.fit(X_train,y_train)

clf = train_test(X_train, X_test, y_train, y_test, clf)
```

    Train score: 1.00
    Test score: 0.50

    Classification report:
                  precision    recall  f1-score   support

        negative       0.60      0.21      0.31        99
         neutral       0.46      0.69      0.56       156
        positive       0.55      0.50      0.52       145

        accuracy                           0.50       400
       macro avg       0.54      0.47      0.46       400
    weighted avg       0.53      0.50      0.48       400

    [[ 21  62  16]
     [  4 108  44]
     [ 10  63  72]]



```python
rf_pred = clf.predict(X_new)
```


```python
summary['Random Forest'] = round(clf.score(X_test, y_test), 3)
```

## Summarize Models and Best


```python
summary
```




    {'Logistic Regression': 0.525,
     'Multinomial Naive Bayes': 0.518,
     'Linear SVMs': 0.528,
     'Kernelized SVMs': 0.502,
     'Neural Networks': 0.502,
     'Random Forest': 0.502,
     'KNN': 0.42}




```python
pd.DataFrame.from_dict(summary, orient = 'index', columns = ['Score'])
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
      <th>Score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Logistic Regression</th>
      <td>0.525</td>
    </tr>
    <tr>
      <th>Multinomial Naive Bayes</th>
      <td>0.518</td>
    </tr>
    <tr>
      <th>Linear SVMs</th>
      <td>0.528</td>
    </tr>
    <tr>
      <th>Kernelized SVMs</th>
      <td>0.502</td>
    </tr>
    <tr>
      <th>Neural Networks</th>
      <td>0.502</td>
    </tr>
    <tr>
      <th>Random Forest</th>
      <td>0.502</td>
    </tr>
    <tr>
      <th>KNN</th>
      <td>0.420</td>
    </tr>
  </tbody>
</table>
</div>



Overall, the best model is the Linear SVM model with Logistic Regression coming in a close second.<br>
The worst model, KNN, was only marginally worse with a 22.8% difference in score. <br>
<br>
When comparing these models against random-chance accuracy, a random guess of sentiment would have a 33.33% change of being accurate; with this in mind, our worst model(KNN) is 8.6% more accurate than random guessing, and the best model(Linear SVM) is 19.2% more accurate than guessing.<br>


```python
#Create dataframe of model predictions
models = [knn_pred, lr_pred, mnb_pred, lsvm_pred, ksvm_pred, nn_pred, rf_pred]
predictions = pd.DataFrame.from_records(models).transpose()
predictions.columns = ['KNN', 'LR', 'MNB', 'Lin. SVM', 'Kern. SVM', 'Neur. Net', 'Rand. For.']
```


```python
#Add predicted sentences into dataframe
predictions['sentence'] = new_texts
predictions['sent'] = ['neutral', 'neutral', 'nautral', 'positive', 'positive', 'positive', 'negative', 'negative', 'negative', 'negative']
predictions
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
      <th>KNN</th>
      <th>LR</th>
      <th>MNB</th>
      <th>Lin. SVM</th>
      <th>Kern. SVM</th>
      <th>Neur. Net</th>
      <th>Rand. For.</th>
      <th>sentence</th>
      <th>sent</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>neutral</td>
      <td>neutral</td>
      <td>neutral</td>
      <td>neutral</td>
      <td>neutral</td>
      <td>neutral</td>
      <td>neutral</td>
      <td>Rothbard writes: “Suppose the economy has a supply of 10,000 gold ounces, and counterfeiters [the state]…pump in 2000 ‘ounces’ more.</td>
      <td>neutral</td>
    </tr>
    <tr>
      <th>1</th>
      <td>neutral</td>
      <td>neutral</td>
      <td>positive</td>
      <td>neutral</td>
      <td>neutral</td>
      <td>neutral</td>
      <td>neutral</td>
      <td>The group revealed: Nearly 100 countries carried out automatic exchange of information in 2019, enabling their tax authorities to obtain data on 8...</td>
      <td>neutral</td>
    </tr>
    <tr>
      <th>2</th>
      <td>neutral</td>
      <td>negative</td>
      <td>positive</td>
      <td>negative</td>
      <td>negative</td>
      <td>positive</td>
      <td>neutral</td>
      <td>The problem arose while he was in Iran as he says he simply looked at his account but performed no transactions or made no transfers during the ch...</td>
      <td>nautral</td>
    </tr>
    <tr>
      <th>3</th>
      <td>neutral</td>
      <td>positive</td>
      <td>positive</td>
      <td>positive</td>
      <td>positive</td>
      <td>positive</td>
      <td>positive</td>
      <td>“Currently Bitcoin appears to be struggling to stay above its support, however, it is showing some bullish divergences so I’d expect a rally in th...</td>
      <td>positive</td>
    </tr>
    <tr>
      <th>4</th>
      <td>neutral</td>
      <td>positive</td>
      <td>positive</td>
      <td>positive</td>
      <td>positive</td>
      <td>positive</td>
      <td>positive</td>
      <td>Data collected by blockchain forensics company Chainalysis shows a significant increase of volume in the first half of the year.</td>
      <td>positive</td>
    </tr>
    <tr>
      <th>5</th>
      <td>neutral</td>
      <td>positive</td>
      <td>positive</td>
      <td>positive</td>
      <td>positive</td>
      <td>positive</td>
      <td>neutral</td>
      <td>Already, Mastercard has announced it will be supporting cryptocurrencies in its payments network.</td>
      <td>positive</td>
    </tr>
    <tr>
      <th>6</th>
      <td>neutral</td>
      <td>negative</td>
      <td>negative</td>
      <td>negative</td>
      <td>negative</td>
      <td>negative</td>
      <td>positive</td>
      <td>Looming US Real Estate Crisis - Freddie Mac Warns of Housing Market Uncertainty, Homebuilder sentiment Drops 58%                        U.S. real ...</td>
      <td>negative</td>
    </tr>
    <tr>
      <th>7</th>
      <td>neutral</td>
      <td>negative</td>
      <td>positive</td>
      <td>negative</td>
      <td>negative</td>
      <td>negative</td>
      <td>negative</td>
      <td>After a rogue employee deleted President Donald Trump’s account two years ago, the company limited access to national leaders’ accounts to a much ...</td>
      <td>negative</td>
    </tr>
    <tr>
      <th>8</th>
      <td>neutral</td>
      <td>positive</td>
      <td>positive</td>
      <td>positive</td>
      <td>positive</td>
      <td>positive</td>
      <td>positive</td>
      <td>A small number of people are starting to focus on the economic situation and the possible destructive aftermath the finance world will see.</td>
      <td>negative</td>
    </tr>
    <tr>
      <th>9</th>
      <td>neutral</td>
      <td>neutral</td>
      <td>positive</td>
      <td>neutral</td>
      <td>neutral</td>
      <td>neutral</td>
      <td>neutral</td>
      <td>If they eventually dump on you, there is no way you will recover.</td>
      <td>negative</td>
    </tr>
  </tbody>
</table>
</div>



This view allows us to compare the predictions across the various models by each sentence. <br>
As you can see, the models did well in predicting some sentences, such as index '0'. <br>
Another example is index position '4'--KNN, our worst model, was the only one that predicted incorrectly.<br>
In index position '9', it is interesting to note that all models predicted this incorrectly. <br>

If given more time, in order to increase the accuracy of our models, a first step would be to re-visit our coding technique to create a process that would allow three coders to be in further agreement; if all parties rated the same sentences, then came together to dispute coding differences, we may have a much more consistent rating that is used to train the models.
