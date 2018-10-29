
# Data mining with Twitter API and Basic NLP - Lab 

## Introduction
In this lab, we shall use our Twitter developer account created in the previous lessson and the keys we generated to make some API calls to twitter. We shall look at a number of ways of accessing twitter data which may suit different use cases for twitter API calling. We shall also get an introduction to the `tweepy` Python library to help us with tweet mining and parsing. We shall perform some basic NLP to count the term frequencies etc.

**Note:** You are encourages to consult the Twitter API documentation and `tweepy` library documentation to understand structure of tweets and their properties for accessing specific elements. 


## Objectives
You will be able to:
* Successfully request tweet data from the Twitter API using the Tweepy library
* Understand the structure of Twitter JSON object structure
* Parse tweet data and perform basic text analysis using basic NLP techniques
* Visualize hashtags, mentions from tweets using number of visualization techniques
* Collect and Parse tweets for a number of use cases

## `tweepy`

Tweepy is open-source library, hosted on GitHub and enables Python to communicate with Twitter platform and use its API. Remember , we can access Twitter API and access tweets even without this library. Tweepy makes the whole process simple and easy to understand and execute. Visit [HERE](https://pythonhosted.org/tweepy/index.html) for Tweepy's official documentation and a full list of offered features. 

Installing tweepy is easy, it can be pip installed as shown below:


```python
# uncomment and pip install tweepy if you havent done so already
# !pip install tweepy
```

We can now simply import import tweepy in the python working environment


```python
# Import tweepy
import tweepy

import operator 
import json
from collections import Counter
import pandas as pd
from nltk import bigrams
```

So we are now good to move on with setting up tweepy with our user and access tokens. 

### Using tweepy
Tweepy supports accessing Twitter through OAuth. Twitter has stopped accepting Basic Authentication so OAuth is now the only way to use the Twitter API. In order to create the API object, however, we must first authenticate ourselves with our developer information.
* Enter your credentials into access_token, access_token_secret, consumer_key, and consumer_secret below


```python
# Set credential variables with appropriate values
consumer_key = "Bqi4VGVT34L55ePRiZF80QlGX"
consumer_secret = "PuxXMs4z04MWIgc2AnDIuLS7gow2P3DVu0B0pjC5vBehpb5jS6"
access_token = "1019612699288915970-Zs9genL06wJmu8dCdwnCIJZt9tFind"
access_token_secret = "6fzYuRBpyR51h7ByTyYdSattJlM3LBSNrEiUpTJQKW60z"
```

### Create the Authentication Object

Our next step would be to create tweepy  OAuthHandler instance with our consumer key and secret and set access token and secret using `tweepy.set_access_token`. We can then create an API object with this information.

We shall set it up as shown below:

```python
# Create the tweepy authentication object with consumer key and secret
auth = tweepy.OAuthHandler(consumer_key, consumer_secret)
# Setting your access token and secret
auth.set_access_token(access_token, access_token_secret)
# Creating the API object while passing in auth information
my_first_api = tweepy.API(auth)
```


```python
# Paste above code here with your credentials to create the Oauth Handler Instance and API object

# Create the tweepy authentication object with consumer key and secret
auth = tweepy.OAuthHandler(consumer_key, consumer_secret)
# Setting your access token and secret
auth.set_access_token(access_token, access_token_secret)
# Creating the API object while passing in auth information
my_first_api = tweepy.API(auth)
```

If the above cell executed without any error.. congratulations. You can now start mining the Twitter API through `my_first_api` object. 

NOTE: If you have a web application and are using a callback URL that needs to be supplied dynamically, you would pass it in like shown below:
```python
auth = tweepy.OAuthHandler(consumer_token, consumer_secret, callback_url)
```

## Collecting Tweets From Own Timeline - `home_timeline()` method
A detailed indsight into Tweepy's API object with supported methods to post, retrieve and select tweets can be found at [Tweepy's API Dcoumentation](https://pythonhosted.org/tweepy/api.html#api-reference). It is strongly recommended for you to visit the documentation and study all the relevant methods and arguments that they take. 

We can directly collect tweets from our home timeline by applying the method `api.home_timeline()`, which collects 20 most recent tweets by default (including retweeted tweets). To adjust the desired number of tweets (take 100 tweets for example),  we can pass in a parameter value as (count = 100).

By default we get first 140 characters of a tweet. We can, however, optionally pass in an extra argument `tweet_mode='extended'` to get the full length of the tweet. This might be useful if you are interested in analyzing the full text of tweets rather than geographical locations, and hashtags etc. only. 

Perform the following 
* See how to get extended tweets [Here](https://github.com/sferik/twitter/issues/880). Use the API object to get tweets from your timeline ,collect 500 tweets.  
* Save the results into `my_timeline_tweets`. 


```python
# Store the tweets it in a variable called my_timeline_tweets

my_timeline_tweets = my_first_api.home_timeline(count=500, tweet_mode='extended')

```

### Tweet JSON

All Twitter APIs that return Tweets provide that data encoded using JavaScript Object Notation (JSON). Here is a summary of information you may find in a typical Tweet JASON. 

* Each Tweet has an **author**, a **message**, a **unique ID**, a **timestamp** of when it was posted. 
* Some tweets contain **geo metadata** shared by the user. 
* Each User has a Twitter **name**, an **ID**, a **number of followers**, and most often an account **bio**.
* With each Tweet we also get **"entity"** objects, which are arrays of common Tweet contents such as **hashtags**, **mentions**, **media**, and **links**. 
* If there are links, the JSON payload can also provide metadata such as the fully unwound **URL** and the webpage’s **title** and **description**.

You may not need all this information for every experiment, but its a good idea to have a general understanding of what data we receive as the result of our API call. 

Here is what a Tweet JSON structure looks like:

---

```pythpn
{
  "created_at": "Thu Apr 06 15:24:15 +0000 2017",
  "id_str": "850006245121695744",
  "text": "1\/ Today we\u2019re sharing our vision for the future of the Twitter API platform!\nhttps:\/\/t.co\/XweGngmxlP",
  "user": {
    "id": 2244994945,
    "name": "Twitter Dev",
    "screen_name": "TwitterDev",
    "location": "Internet",
    "url": "https:\/\/dev.twitter.com\/",
    "description": "Your official source for Twitter Platform news, updates & events. Need technical help? Visit https:\/\/twittercommunity.com\/ \u2328\ufe0f #TapIntoTwitter"
  },
  "place": {   
  },
  "entities": {
    "hashtags": [      
    ],
    "urls": [
      {
        "url": "https:\/\/t.co\/XweGngmxlP",
        "unwound": {
          "url": "https:\/\/cards.twitter.com\/cards\/18ce53wgo4h\/3xo1c",
          "title": "Building the Future of the Twitter API Platform"
        }
      }
    ],
    "user_mentions": [     
    ]
  }
}



```

---

## Parsing Tweet JSON

Let's try to print a single tweet to see what it looks like


```python
# Unomment below to view contents of the tweet
#my_timeline_tweets[0]
```

So the Tweet here is in the JSON format , which we can parse as dictionaries. 

We can iterate through collected tweets in `my_timeline_tweets` to access any of the properties of each tweet. Let's print the name and location of users with `.user.name`, `.user.locations` propoerties, and content of the each tweet using the `.text` property. 

**NOTE:** Use `.full_text` in case of using `tweet_mode='extended'` OR just `.text` if you havent passed in the extended argument. 

#### Iterate through `my_timeline_tweets` and print the usernames, location and text of the each collected tweet. 


```python
# Iterate through my_timeline_tweets and print the name, location and text for each tweet using k:v pairs addressing
for tweet in my_timeline_tweets:
   # printing the name and full text stored inside the tweet object
    print (tweet.user.name)
    print (tweet.user.location)
    print (tweet.full_text)
    print ('----------------------------------------------------------------------------')
```

    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @RalstonReports: The turnout models are looking better for Democrats in Nevada as they boost their statewide lead a bit, but long way to…
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    [Cheat Sheet] Python Basics For Data Science https://t.co/fH07ZFF9zw
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @Popehat: NYT:  Well, Okay, But Are People Mean To Trump?
    Everyone: Can you not
    NYT:  Jazz: What If Black People Got Involved In Jazz
    Ev…
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    https://t.co/txeJ3g9IMW
    ----------------------------------------------------------------------------
    Open Data Science
    Boston, MA
    IBM and NVIDIA collaborate to expand machine learning tools for data scientists. Learn how:  https://t.co/nslIPVLGOD #ODSC #ODSCWest
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    Argyle only smiles when you vote 🐶
    
    @darth https://t.co/SfJv4VKHRh
    ----------------------------------------------------------------------------
    Data Science Central
    Los Angeles, CA
    Simple Proof of the Prime Number Theorem https://t.co/Iif9cOgrco
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    Analyzing the relationship of Twitter users towards brands (e. g. Air Berlin) https://t.co/TyKd8nwTIw
    ----------------------------------------------------------------------------
    Data Science Central
    Los Angeles, CA
    7 Steps to Build a Data-Driven Culture in your Organization https://t.co/Xfw1jvHqIO
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @mdub2829: @JohnCornyn @FreeBeacon @JonnCornyn This is from an article published a week ago about the economics of climate change, and y…
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @xeni: Don’t know about you but I woke up ready to fight Nazis and defend my fellow Americans and vote these white supremacist sons of b…
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @LeslieMac: White Nationalist Who Threatened Stacey Abrams' Campaign Poses for Picture With Brian Kemp https://t.co/bn3NXxtsRp
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @ClaraJeffery: Do not work for Fox.
    Do not appear on Fox.
    Boycott Fox's advertisers.
    Do not act as an enabler. 
    Make spewing poison have…
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @Lawrence: Steve Bannon's ex-wife said under oath he didn't want his daughters to go to a school because it had too many Jews.
    No Bannon…
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @alejandracano1: Georgia https://t.co/xK3uMnuyHe
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @MotherJones: A GOP congressman met with a group founded by a Nazi—while on a Holocaust memorial trip https://t.co/jPaEqCV9eR https://t.…
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @washingtonpost: Fact Checker: In an effort to justify holding a campaign rally after 11 people at a synagogue were gunned down in Pitts…
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @hgoemans: Makes me sick: ''Rabbi David Lau said that “any murder of any Jew in any part of the world for being Jewish is unforgivable.”…
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @Victor_Asal: Israel's chief rabbi refuses to call Pittsburgh massacre site a synagogue because it's non-Orthodox - Israel News https://…
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @teamabrams: #TeamAbrams, @staceyabrams, and @common are leading Souls to the Polls! Get out and VOTE: https://t.co/RnfXUctIio #GAGov #g…
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @Yair_Rosenberg: "Synagogue victims Cecil and David Rosenthal remembered as loving, inseparable" https://t.co/Py9M5f0vWv
    ----------------------------------------------------------------------------
    Open Data Science
    Boston, MA
    Join Olivier Blais, Head of Data at Flinks at Accelerate AI in San Francisco, Oct 31 - Nov 1. #AI #ODSC #ODSCWest
    https://t.co/AwwiS915Ki https://t.co/kM6ichWmlK
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    What pays most: R, Python, or SQL? https://t.co/86CsHViVQT
    ----------------------------------------------------------------------------
    Data Science Central
    Los Angeles, CA
    How to Solve the New $1 Million Kaggle Problem - Home Value Estimates https://t.co/Vr4naGZLz2
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @MicheleChabin1: Via someone who knew the two youngest victims: "Cecil and David Rosenthal, were two developmentally disabled brothers w…
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @tsengputterman: Remember when the Trump Administration issued a Holocaust Remembrance Day statement that failed to mention "Jews" or "a…
    ----------------------------------------------------------------------------
    Data Science Central
    Los Angeles, CA
    Essential Math for Data Science https://t.co/StKvtDL0iJ
    ----------------------------------------------------------------------------
    Data Science Unicorn
    London, England
    RT @BecomingDataSci: How to learn data science if you're broke
    https://t.co/syTz9glXSC
    
    h/t @patrickDurusau
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    Someone put this owl in the planter in front of my house and is it a voodoo curse? https://t.co/zsSNk4gGoG
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    RT @Ronald_vanLoon: The Future of #DataScience in One Picture
     by @granvilledsc @DataScienceCtrl @AnalyticBridge @BoozAllen |
    
    Read more he…
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    RT @KirkDBorne: A Research #DataScientist Explains #DeepLearning: https://t.co/btd9Bkx60i #abdsc 
    
    #BigData #DataScience #AI #MachineLearni…
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    RT @schmarzo: At least 21% of #CIOs are confused by #DigitalTransformation! Try these 3 elements of 'Intelligence Transformation' to reengi…
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    🚨The Midterms are in 8 days 🚨
    
    ¡Vote!
    
    ¡Vote!
    
    ¡Vote!
    
    ¡Vote!
    
    ¡Vote!
    
    ¡Vote!
    
    ¡Vote!
    
    ¡Vote!
    
    ¡Vote!
    
    ¡Vote!
    
    ¡Vote!
    
    ¡Vote!
    
    ¡Vote!
    
    ¡Vote!
    
    ¡Vote!
    
    ¡Vote!
    
    ¡Vote!
    
    ¡Vote!
    
    ¡Vote!
    
    ¡Vote!
    
    ¡Vote!
    
    ¡Vote!
    
    ¡Vote!
    
    ¡Vote!
    
    ¡Vote!
    
    ¡Vote!
    
    ¡Vote!
    
    ¡Vote!
    
    https://t.co/cpxzEChUAR
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    RT @KirkDBorne: Prediction at Scale with Scikit-learn and PySpark #Pandas: https://t.co/QF4o7A1hwW #abdsc
    
    #BigData #PredictiveAnalytics #D…
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    RT @KirkDBorne: 3 reasons why #Coding (specifically, #Python) Should Become Your Official Corporate Language:
    1)Teaches to think and solve…
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    RT @schmarzo: Single best #Design tool for driving organizational alignment. Period. Prioritization Matrix:  Aligning Business-IT on #BigDa…
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    RT @KirkDBorne: 29 Statistical Concepts Explained in Simple English - Part 1: https://t.co/DMIDGllR0z #abdsc #BigData #DataScience #DataLit…
    ----------------------------------------------------------------------------
    Open Data Science
    Boston, MA
    Intel has announced plans to open a joint AI research center in Haifa, Israel. The center will be run in a collaboration between Intel AI Research and Technion – Israel Institute of Technology. #ODSC #ODSCWest #AI https://t.co/Xz7KYrYvkB
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    Weekly Digest, October 29: https://t.co/XUkx3NMOZ4
    ----------------------------------------------------------------------------
    IBM Data Science
    San Francisco, CA
    RT @IBMcommunity: #AskaDataScientist: @IBM_Champions @NirKaldero just published #DataScience for Executives, which is #1 on the @Inc top 10…
    ----------------------------------------------------------------------------
    IBM Data Science
    San Francisco, CA
    RT @IBMAnalytics: Join IBM in San Francisco for #Think2019. Early bird rates expire Nov. 16th — Register now: https://t.co/6wMSnDeR8y https…
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @LisPower1: Hey @FoxNews, while you're covering your ass and pretending to be outraged by people using your network to spread conspiracy…
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @KevinMKruse: In 1860, Lincoln ran on a platform that called for open immigration, called for an easy path to citizenship for those who…
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @SarcasmRulz1: If that’s comfortable 🙄 @YeahItWasSaid https://t.co/ZYyZe5HH3X
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @Popehat: Looks like Gab has deleted a lot of their old tweets.  Like the time they repeatedly retweeted an adolescent white nationalist…
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @SwiftOnSecurity: Gab’s entire “neutral free speech” thing is a scam. The owner is an InfoWars commentator. They retweet antisemitic shi…
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @SwiftOnSecurity: Gab is NOT an “alternative to Twitter.”
    Everyone there has Twitter accounts, multiple ones. Gab is just where they dum…
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @TomSteyer: .@realdonaldtrump just tweeted at me in his typical insulting style after watching @CNNsotu. It is unthinkable that in the m…
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @Weinsteinlaw: This is the 300th day of the year. 
    
    There have been 294 mass shootings in 2018. 
    
    There are 10 days until Election Day.…
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @LatinoUSA: President Trump is preparing to announce a sweeping border crackdown in a speech Tuesday, a week before the midterm election…
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @GeeDee215: But the reason you consistently push back on his bullshit at Thanksgiving dinner is not to change his mind, but because the…
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    AI and the Future of Millennials https://t.co/b4IkqJwyM6
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @TheRickWilson: You're also a gleeful swamp of raging anti-semites, alt-right incel cockwombles, slack-jawed Klan dickweeds, and neo- cr…
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    Amazing thread with lots of new Russian election data 👇 https://t.co/dNa58ZswY9
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @ClaraJeffery: Tom Steyer was sent a pipe bomb by a Trump supporter... https://t.co/uLxClvLZhV
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @mikeydickerson: Facebook, Twitter, and Youtube knew that they were the main vehicles for radicalizing violent terrorists overseas no la…
    ----------------------------------------------------------------------------
    Data Science Central
    Los Angeles, CA
    A Study of Reddit Politics https://t.co/1xORK8refj
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    Machine Learning Explained: Understanding Supervised, Unsupervised &amp; Reinforcement Learning https://t.co/4sN9tmR9AD
    ----------------------------------------------------------------------------
    Data Science Central
    Los Angeles, CA
    Book: Java Deep Learning Essentials https://t.co/hkXTG40i7f
    ----------------------------------------------------------------------------
    Open Data Science
    Boston, MA
    Get ready for the the most exciting Data Science event event of the fall! @ODSC West 2018 is 3 days TO GO. And you still have a chance to join us in San Francisco, Oct 31- Nov 3. 
    Register NOW for FINAL Tickets https://t.co/NOa68TVI7L  
    #ODSCWest #DataScience #AI https://t.co/OAcpZQT4SS
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @eaglesboy2: From deepest Wales to the Malvern Hills! Our new puppy Whisky! #welshsheepdog #bordercollie #puppy 💙❤️ https://t.co/AzbypZa…
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @foxcrusade: Rly missing my doggo rn, here's some pictures of her! She really likes tennis balls #bestdoggo #bordercollie https://t.co/z…
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @PiebaldThe: Remember to wrap up warm for your walks 😛
    
    #patch #dogs #dogsoftwitter #collie #bordercollie #pup #colliesoftwitter #pets #…
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @existentialfish: And it's not that Meet the Press wasn't aware of all of this. We (@mmfa) sent them an email  asking for comment about…
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @MSignorile: Rep. Kevin McCarthy Deletes Tweet Singling Out 3 Jews Helping Bankroll Democrats https://t.co/GfXv62aecs via @HuffPostPol
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    Data Science Summarized in One Picture https://t.co/fLWfofC7Q0
    ----------------------------------------------------------------------------
    Open Data Science
    Boston, MA
    Join some of the best and brightest in #DataScience at @ODSC West 2018. This is the ideal opportunity to network with thought leaders and professionals across industry. 
    
    Final Tickets Available https://t.co/rzsieZK5Kd    
    #ODSCWest https://t.co/EQ2kqLJet8
    ----------------------------------------------------------------------------
    Data Science Central
    Los Angeles, CA
    Full Stack Data Scientist: The Elusive Unicorn and Data Hacker https://t.co/Sdh7P1otIE
    ----------------------------------------------------------------------------
    WeWork
    Global
    Female-founded AND sustainable. WeWork member company @oui_shave is bringing the heat by leaving the (razor) burn behind. https://t.co/AVMqVM6PmY
    ----------------------------------------------------------------------------
    Open Data Science
    Boston, MA
    Join @ODSC West LiveStream to see our opening Keynotes: Paul Taylor, Andrej Karpathy and Virginia Eubanks on Friday, Nov 2 at 9:30 AM PST @IBMWatson @karpathy @Tesla @PopTechWorks #ODSCWest #AI
    https://t.co/qAuyVtRHOa https://t.co/qpVYceRlrr
    ----------------------------------------------------------------------------
    Open Data Science
    Boston, MA
    The Microsoft Infer NET machine learning framework goes open source #MachineLearning #ODSC #ODSCWest https://t.co/6o0tIyRPaU
    ----------------------------------------------------------------------------
    WeWork
    Global
    Sundays are for self reflection #sundaymorning
    📸 @CYDPLAYGROUNDS &amp; @PopUpYogaParis https://t.co/pAk5nApJj3
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    88 Resources &amp; Tools to Become a Data Scientist https://t.co/WWD8Q8UWzi
    ----------------------------------------------------------------------------
    Open Data Science
    Boston, MA
    Watch the @ODSC West 2018 keynotes and 20+ other sessions with our LIVESTREAM Pass #ODSCWest #AI
    
    https://t.co/71vzQkAVik https://t.co/9Bi4QtFKPz
    ----------------------------------------------------------------------------
    Open Data Science
    Boston, MA
    Join Ahna Girshick, Ford Garberson, Brianna Schuyler and Marc Smith at @ODSC West in San Francisco, Oct. 31 - Nov. 3. @AhnaGirshick @BriannaSchuyler @marc_smith #ODSCWest #DataScience #ODSC
    https://t.co/29flVRSWjF https://t.co/MlcVIZvFZ7
    ----------------------------------------------------------------------------
    Data Science Central
    Los Angeles, CA
    5 Phases To Successfully Complete a Data Science Project https://t.co/EGl9b1H5rI
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    Sample Job Titles and Salaries for Recently filled Data Science Positions in US https://t.co/fkO5buj50c
    ----------------------------------------------------------------------------
    Open Data Science
    Boston, MA
    Data science is able to solve problems and answer questions, but it’s not without its own challenges. When is data really worth it? Are you infringing on privacy? #DataScience #ODSC #ODSCWest https://t.co/eaUubE1tDS
    ----------------------------------------------------------------------------
    IBM Data Science
    San Francisco, CA
    #ICYMI: The top 3 things you need to know about IBM Watson Studio: https://t.co/aHRyzyOjiu #machinelearning #datascience #WatsonStudio
    ----------------------------------------------------------------------------
    Data Science Central
    Los Angeles, CA
    The Top 100 Analytics Startups of 2015 https://t.co/bGPclVL6uK
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    Is Data Science Ripe for a Massive Merger &amp; Acquisition / Consolidation? https://t.co/GDY3rVV9F8
    ----------------------------------------------------------------------------
    Open Data Science
    Boston, MA
    This article describes TensorLayer, a modular Python wrapper library for TensorFlow allowing data scientists to streamline the development of complex deep learning systems. #TensorFlow #ODSC #ODSCWest https://t.co/WbnRqtOYk4
    ----------------------------------------------------------------------------
    Open Data Science
    Boston, MA
    Check out The Amazing, Anti-Jargon, Insight Filled, and Totally Free Handbook to Integrating AI in Highly Regulated Industries, a free handbook that covers the entire breadth of the “black box” issue #AI #ODSC #ODSCWest  @basistechnology https://t.co/nKwQXqB0Z8
    ----------------------------------------------------------------------------
    Open Data Science
    Boston, MA
    A question and an answer about recoding several factors simultaneously in R. #DataScience #ODSC #ODSCWest https://t.co/MlJNoPpOgK
    ----------------------------------------------------------------------------
    Data Science Central
    Los Angeles, CA
    Predicting animal adoption with Random Forest, SVM https://t.co/0cr1SKoQz2
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    Office Politics: A survival guide for data scientists https://t.co/0OgDGx2eOT
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @JordanUhl: [cake voice] i like a guy with short arms and a looooooong jacket https://t.co/3kRLn9JglQ
    ----------------------------------------------------------------------------
    Data Science Central
    Los Angeles, CA
    So, How Many ML Models You Have NOT Built? https://t.co/8qUlQLaT2f
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    Learn under the hood of Gradient Descent algorithm using excel https://t.co/BpRH2Nt67g
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @PascaliwagED: @dataandpolitics @SocialSciNerd Anything on political efficacy. Clarke and Acock 1989 : national elections, if your side…
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @smotus: Steadily all other amendments are sacrificed to the Second.
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @AnandWrites: This is what mourners keeping vigil chanted in Squirrel Hill tonight:
    
    Vote.
    
    Vote.
    
    Vote.
    
    Vote.
    
    Vote.
    
    Vote.
    
    Vote.
    
    Vo…
    ----------------------------------------------------------------------------
    Data Science Central
    Los Angeles, CA
    Question: Time series hourly data plot, in Python https://t.co/7O9ifza2F2
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    Quantile Regression in Python https://t.co/0ca949YRRJ
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @PSchoettmer: @poliscigrl @ADobranic @amyericasmith @dataandpolitics Regardless of who you are actually working for and wherever you are…
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @ElectProject: Georgia had another day of 100K+ votes and becomes the 7th state to exceed their 2014 *total* #earlyvote, joining DE, IN,…
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @ElectProject: Georgia's African-Americans represented today, adding 37,065 #earlyvote and raising the statewide percentage of African-A…
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @tom_s_clark: Almost a month ago, @Delta called me to apologize for a flight disruption and offer to send me $100 in cash as a friendly…
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @FrankManic: @dataandpolitics And you'll get tons of likes when you post your sticker selfie
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @EmbryoResist: @dataandpolitics @OldTehanu @darth https://t.co/4EaPMWJUKW
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    What is Big Data And How Does It Work? https://t.co/9ffLT4Zjla
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    Data Scientist versus Data Architect https://t.co/qbzSK4cneI
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    How to pass multiple inputs (features) to LSTM using Tensorflow? https://t.co/SKHnMQLij5
    ----------------------------------------------------------------------------
    Data Science Central
    Los Angeles, CA
    Open Source automated ETL of web-based data sources https://t.co/qwwgLe3QDl
    ----------------------------------------------------------------------------
    Data Science Central
    Los Angeles, CA
    Application of Image Processing and Convolution Networks in Intelligent Character Recognition for Digitized Forms Processing https://t.co/3PoGGrUSkr
    ----------------------------------------------------------------------------
    DataScienceWeekly
    
    RT @stevenstrogatz: A very nice article. Clear and explicit. Includes an excellent treatment of the strict triangle inequality in 2-D hyper…
    ----------------------------------------------------------------------------
    DataScienceWeekly
    
    RT @conormyhrvold: Michelangelo PyML: Introducing #Uber's Platform for Rapid #Python #ML Model Development https://t.co/2mcbPfSN0P via @ube…
    ----------------------------------------------------------------------------
    Data Science Central
    Los Angeles, CA
    An overview of feature selection strategies https://t.co/7yLTMmSnXP
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    Dome Team Performance in the NFL https://t.co/jwpiGzKKLr
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    Trump Solves Famous Mathematical Conjecture: https://t.co/48CE3rZWdA
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @mirandayaver: Oh come the fuck on, @nytimes. Enough with "both sides." When the media and politicians nonviolently critical of the pres…
    ----------------------------------------------------------------------------
    Open Data Science
    Boston, MA
    The recent, widespread attention given to knowledge graphs from the research community has helped foment a significant debate among knowledge representation experts: what even is a knowledge graph? #DataScience #ODSC #ODSCWest
    https://t.co/VIYT0R4UIM
    ----------------------------------------------------------------------------
    Data Science Central
    Los Angeles, CA
    Using Analytics to Improve Customer Engagement https://t.co/do9R3EAukA
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    Contingency Tables in R https://t.co/qYRuZGHE0S
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @stlbf: @dataandpolitics @darth We all need doggo love! Especially on a rotten day like today. https://t.co/cfq7pFSXDT
    ----------------------------------------------------------------------------
    Data Science Central
    Los Angeles, CA
    Apache Spark Introduction  A Comprehensive Guide for beginners https://t.co/dIE1cbxCoo
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    CDO or CAO? https://t.co/jsB4JPkwaD
    ----------------------------------------------------------------------------
    Open Data Science
    Boston, MA
    Can't make it to @ODSC in San Francisco this year?  Don’t miss out! Access more than 20 Deep Learning, Machine Learning + other Data Science talks &amp; workshops with your #ODSCWest 2018 LIVESTREAM Pass.
    #ODSC  https://t.co/SNvrge398H https://t.co/CaH0FKXaWg
    ----------------------------------------------------------------------------
    Open Data Science
    Boston, MA
    Join Karthik Ramasamy, Co - Founder at Streamlio at @ODSC West 2018 in #SanFrancisco, Oct 31 - Nov 3.  #ODSC #ODSCWest
    @streamlio
    https://t.co/BMQ602X2jw https://t.co/XOvCsXWWiA
    ----------------------------------------------------------------------------
    Data Science Central
    Los Angeles, CA
    The Future of IoT and Machine to Machine Payments https://t.co/3eDcJlbzuG
    ----------------------------------------------------------------------------
    WeWork
    Global
    Our members explain how showcasing their personal style allows them to approach work with confidence and power. @RenttheRunway makes it easy. #weekendreads
    https://t.co/6Sh2HyhzEn
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @ArielTroster: A corgi Halloween parade is exactly as ridiculous as it sounds https://t.co/Om3p96b40c
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @sgadarian: @VegHistory I’m on the board of our synagogue and we spend a lot of our budget on  armed security - every week at services a…
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    Great Github list of public data sets https://t.co/Vg5H3eR7Ux
    ----------------------------------------------------------------------------
    Open Data Science
    Boston, MA
    Deep Lens raises $3.2 million for AI cloud pathology platform #AI #ODSC #ODSCWest https://t.co/atjRCRlf7c
    ----------------------------------------------------------------------------
    Data Science Central
    Los Angeles, CA
    Question: Recommendation System https://t.co/SGvSSzuWZD
    ----------------------------------------------------------------------------
    Open Data Science
    Boston, MA
    Check out the new additions to the @ODSC West schedule. Unprecedented choice of AI + #DataScience talks, workshops, and trainings for all levels. 
    
    Limited Tickets Remaining https://t.co/aLNkAZl3Qw 
    #MachineLearning #ODSCWest https://t.co/3ISLo2sLoc
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    Six categories of Data Scientists https://t.co/S6T47CQz80
    ----------------------------------------------------------------------------
    Open Data Science
    Boston, MA
    See how Google is applying deep learning to metastatic breast cancer detection #DeepLearning #ODSC #ODSCWest https://t.co/s59p6GyOxV
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    Silicon Valley Firms Make It Nearly 'Impossible' For Cambridge To Hire AI Staff https://t.co/h2BfUx9Bfm
    ----------------------------------------------------------------------------
    Data Science Central
    Los Angeles, CA
    Developers outlook on modern AI in 2018 https://t.co/MhsYnkVuiI
    ----------------------------------------------------------------------------
    Data Science Central
    Los Angeles, CA
    Question: Measuring Accuracy for a Multiclass Classification Model https://t.co/QsgrgY23UJ
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    Cross-Validation: Concept and Example in R https://t.co/GyxQuG1WFC
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    McGonagall is the world’s sweetest senior #bordercollie!
    
    She’s available for adoption at @Muttville in San Francisco! 
    
    @darth @morgfair https://t.co/ak0Omyrjvx
    ----------------------------------------------------------------------------
    Uber
    Global
    Get out and seize the Saturday. Learn more about getting there on two wheels.
    
    https://t.co/KdP6x5hNLD https://t.co/8Dfv2R1yb6
    ----------------------------------------------------------------------------
    Data Science Central
    Los Angeles, CA
    Cheat Sheet: Data Visualization with R https://t.co/jylcVkkgpw
    ----------------------------------------------------------------------------
    Data Science Central
    Los Angeles, CA
    One Trillion Random Digits
    https://t.co/9xSbj1TN4a
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    The news this morning is terrible. 
    
    Here is Argyle to bring a moment of zen to your timeline. 
    
    @darth #SeniorDog https://t.co/IjA2wZHKkR
    ----------------------------------------------------------------------------
    DataScienceWeekly
    
    RT @techreview: A controversial artwork created by AI has hauled in $435,000 at auction https://t.co/izEAPt2s4E
    ----------------------------------------------------------------------------
    DataScienceWeekly
    
    RT @xamat: "Sorry I didn’t get that! — How to understand what your users want (using unsupervised AI)" by @_langAI https://t.co/KooeMmNsly
    ----------------------------------------------------------------------------
    DataScienceWeekly
    
    RT @fchollet: Nice guide to text classification using Keras: https://t.co/iIh0hY6akM
    ----------------------------------------------------------------------------
    DataScienceWeekly
    
    RT @seb_ruder: Are you interested in summarization? @tbsflk compiled the results on the most common datasets (CNN/DailyMail, Gigaword, DUC0…
    ----------------------------------------------------------------------------
    Open Data Science
    Boston, MA
    Join @ODSC West 2018. Only 4 days left before the start of the leading #DataScience event of the year. Join our global community of 4500 attendees &amp; learn, network, and connect. Tickets Available:  https://t.co/fVmNE4sng6    
    #ODSC #AI https://t.co/52H1pssVN4
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @shallow_state: @poliscigrl @ADobranic @amyericasmith @dataandpolitics If you had a strong expectation the voter would support your cand…
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    10 Big Data Use Cases Everyone Must Read https://t.co/LLUElpqo1d
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @Cliff_M85: "If they had guns" - Three police officers shot by mad man
    "Thoughts and prayers" - There were plenty of those in the synago…
    ----------------------------------------------------------------------------
    Open Data Science
    Boston, MA
    According to Dell, these are the 8 key steps for successful AI implementation #AI #ODSC #ODSCWest https://t.co/OQsq8WrPWT
    ----------------------------------------------------------------------------
    IBM Data Science
    San Francisco, CA
    How do you deliver business value to an insurance firm that must balance reducing expenses &amp; giving customers a positive claims experience? Listen in on this Q&amp;A with Irina Saburova &amp; @rosiepongracz to see how Watson Studio can help: https://t.co/iTTyJ1eFle #datascience https://t.co/cdNlboNitD
    ----------------------------------------------------------------------------
    Deep Learning Group
    Palo Alto, CA
    https://t.co/jmIhOSZqqZ, Progress Report #2: It works. – Monday Note https://t.co/XMAZzahd1k
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @happydogs0301: 今日のお客様🐶🐈ワクチン接種とお耳の治療のための通院でした🏥
    ふたりとも、あっという間に大きくなってました😆これからが楽しみですね♥️
    https://t.co/pxodN1RlTW #ペットタクシー #ペットの送迎 #ハッピードッグズ #…
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @ianseren: Attention ! ⁦@ruthwignall⁩ ⁦@kelseyredmore⁩ ⁦@ItsYourWales⁩ #bordercollie https://t.co/0PUiTXUdXX
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @MarinaRiveraR: Cuando alguien quiere madrugue para aprovechar la mañana de sábado. #felizsabado #BorderCollie #misombra https://t.co/uQ…
    ----------------------------------------------------------------------------
    Data Science Central
    Los Angeles, CA
    Top 20 Big Data Experts to Follow (Includes Scoring Algorithm) https://t.co/GaTwMjZKvZ
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @northeastguides: After the initial clag and snow, it turned into a cracking day in #Northumberland 
    #dogfriendly #bordercollie #Cheviot…
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @LifeAsACollie: #Sunset at the #Beach #Goldenhour #happydog #BorderCollie #BorderFame https://t.co/TL1kqK9jC8
    ----------------------------------------------------------------------------
    Data Science Central
    Los Angeles, CA
    Graph-based intelligence analysis https://t.co/KObzGSVvF4
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    14 Great Articles About Cross-Validation, Model Fitting and Selection https://t.co/cSlQAHjJHK
    ----------------------------------------------------------------------------
    Open Data Science
    Boston, MA
    Fall’s most exciting Data Science conference, @ODSC West 2018, is only 4 days away. We have a few remaining ticket left.
    
    Register NOW for our FINAL 58 Tickets: https://t.co/9x87hHbAy4
    #ODSCWest #DataScience #AI https://t.co/4VyVK6X68B
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    Three officers were shot. https://t.co/xak52ENGCt
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @EricBoehlert: Trump just blamed synagogue
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @mollycrabapple: This piece of shit murdered Jews because a Jewish group helps refugees and immigrants. Because they protest the Muslim…
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @KagroX: Waiting for Trump's call for armed mosques.
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @rebeccakreitzer: @poliscigrl @emayfarris Party for the Disclosure of Information about an Unborn Baby’s Genitalia so Family and Friends…
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    Predicting Car Prices Part 1: Linear Regression https://t.co/Itb6Q1yyp2
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @desdemanhattan: Tiroteo en sinagoga de Pittsburgh, última hora:
    - Asesino bajo custodia tras ser herido por la policía. Se sabe por aho…
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @GaryRHeaders: What normally happens when we have guests #Bilbo #BorderCollie #dogsoftwitter https://t.co/7Axqoxb8bU
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @BoneCanis: One of Mark's sessions from today.  #bordercollie #collie #puppy #dogs https://t.co/DbVo1fm4Rl
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @bordercolliefc: Cool Tricks with Coco the Border Collie https://t.co/fELW8UN0IS #bordercollie #Dogs #bordercolliefc https://t.co/zVZS7I…
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    Omg it’s a witch https://t.co/jJyGc2i0nE
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @corgicrazytime: The face I wake up to! Love my little Jasper! #BorderCollie https://t.co/qtzVXM9eJc
    ----------------------------------------------------------------------------
    IBM Data Science
    San Francisco, CA
    RT @IBMEmbed: Learn more about #datascience and #deeplearning with presentations from the recent Global Data Science Forum held in Germany…
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @mehdirhasan: Let’s be clear: whoever turns out to have done this shooting in a synagogue, and for whatever motive, the fact is that the…
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @juliaioffe: It starts with anti-Semitic trolls and coded language and it ends in violence. Don’t say I didn’t warn you. Two and a half…
    ----------------------------------------------------------------------------
    Open Data Science
    Boston, MA
    We are almost sold out with only a few days to go until the start of @ODSC West 2018.
    FINAL TICKETS Available: https://t.co/0oqrsQVU40  
    #ODSCWest #DataScience #AI https://t.co/6To6WqVCx4
    ----------------------------------------------------------------------------
    Dr. Data&Science 🎃
    SFO | ATL | 🛫
    RT @chris_e_larkin: @dataandpolitics Assuming social pressure isn't possible, the next best non-partisan gotv messaging is the idea of grat…
    ----------------------------------------------------------------------------
    Data Science Central
    Los Angeles, CA
    Comparing AI Strategies – Systems of Intelligence https://t.co/MSUpviMW0A
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    How to Treat Missing Values in Your Data https://t.co/XZzcyaQuPj
    ----------------------------------------------------------------------------
    Open Data Science
    Boston, MA
    Watch the #ODSCWest 2018 Keynotes Daphne Koller, PhD, Carrie Grimes Bostock and Derek Haoyang Li with our LIVESTREAM Pass on Saturday, Nov. 3
    @DaphneKoller @coursera @Google
    https://t.co/ha9eCS7oOC https://t.co/mX1G9rP98L
    ----------------------------------------------------------------------------
    Open Data Science
    Boston, MA
    Learn more about some of the leading research centers and institutions around data science, machine learning, and artificial intelligence in Europe. #DataScience #ODSC #ODSCWest https://t.co/YzPq0WyOV7
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    RT @KirkDBorne: Chart Suggestions — a thought-starter for your #DataStorytelling (with a link to the R code for the different visualization…
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    RT @schmarzo: Sorry if I'm pounding this point too hard but heart of good #DataScience is asking smart questions in order to set the contex…
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    RT @granvilleDSC: 10 Deep Learning Terms Explained in Simple English https://t.co/P9KeQRrRRn
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    RT @KirkDBorne: 140 #MachineLearning Formulas for #DataScientists : https://t.co/axS4Pg3fF9 #abdsc #Mathematics #Statistics #BigData #DataS…
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    RT @granvilleDSC: R and Python cheatsheets https://t.co/eMIDiwmqTv
    ----------------------------------------------------------------------------
    WeWork
    Global
    In partnership with @hackneycouncil and @DianaAward, we're moving away from the old idea of work placement and helping train the next generation of leaders. #NationalMentoringDay https://t.co/FORPf1WVSc
    ----------------------------------------------------------------------------
    Open Data Science
    Boston, MA
    Don’t miss your chance to network with 200+ foremost #MachineLearning &amp; #AI experts and 4,000+ other #DataScience professionals. Learn the latest trends, tools and insights at @ODSC West 2018.
    
    Limited Tickets Remaining: https://t.co/bkT31m1FIG    
    #ODSCWest https://t.co/zKdYNgIe5Q
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    40 Techniques Used by Data Scientists https://t.co/apEVTtGg5p
    ----------------------------------------------------------------------------
    Open Data Science
    Boston, MA
    Here are just a few of the many extremely insightful topic sessions that will presented at @ODSC Accelerate AI business summit in San Francisco,Oct 31- Nov 3.  https://t.co/yPAWF9RZal
    #ODSCWest #AI #ODSC https://t.co/6VdGTafPJh
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    Book: Mastering Python for Data Science https://t.co/V9U7OgK7op
    ----------------------------------------------------------------------------
    Data Science Central
    Los Angeles, CA
    Basics of Bayesian Decision Theory https://t.co/QcRLjoa27n
    ----------------------------------------------------------------------------
    Open Data Science
    Boston, MA
    Join ODSC LIVESTREAM &amp; get access to 22 sessions  &amp; 6 keynotes over 2 days
    https://t.co/CmPleeRrRm #ODSC #ODSCWest https://t.co/5KSOr6ITMJ
    ----------------------------------------------------------------------------
    Data Science Unicorn
    London, England
    RT @betzibetzner: Just published a beginner's guide ro scala https://t.co/d5nRUmg4NG
    ----------------------------------------------------------------------------
    WeWork
    Global
    RT @AndiamoHQ: An amazing night with @WeWork @WeWorkUK  and a gorgeous photo of our founders Sam and Naveed with @MiguelMcKelvey, @aplusk,…
    ----------------------------------------------------------------------------
    WeWork
    Global
    For WeWork members like Jennifer Charles, Director at @BuildngBlockRes, clothing helps with confidence. "It affects the way people see me, which I then project back to them.” Here's why we're teaming up with @RenttheRunway to change the future of fashion in the workplace.
    ----------------------------------------------------------------------------
    WeWork
    Global
    Who says you have to leave the office to immerse yourself in nature? 📷 sailortaylor13 #saturdaymorning https://t.co/huyrcOVYOU
    ----------------------------------------------------------------------------
    Open Data Science
    Boston, MA
    Win with AI: Insurance company Guidewell looks to get data for AI. #ODSC #ODSCWest
    https://t.co/zn4hUovWiD
    ----------------------------------------------------------------------------
    IBM Data Science
    San Francisco, CA
    Essential math for data science: A curated list including functions, variables, equations, graphs, statistics, linear algebra, calculus, discrete math, optimization, and operation research topics, whew. Check it out here: https://t.co/0JtcPuJ04N via @kdnuggets #datascience https://t.co/E2m2trORG9
    ----------------------------------------------------------------------------
    Data Science Central
    Los Angeles, CA
    The Mathematics of Machine Learning https://t.co/GUBZVT8NUj
    ----------------------------------------------------------------------------
    Data Science Central
    Seattle, WA
    75 Big Data Terms To Make Your Dad Proud of You on Father's Day https://t.co/mYEx0wO1jK
    ----------------------------------------------------------------------------
    Open Data Science
    Boston, MA
    Using AWS DeepLens, you can build a simple face detection application that counts the number of cups of coffee that people drink and displays the tally on a leaderboard. #ODSC #ODSCWest #DataScience https://t.co/RmdZkdwMMh
    ----------------------------------------------------------------------------


Thats is great. We are getting some meaningful information indeed. 

#### Before we proceed ....
*We are using a test account and following a number of AI, Data science and Tech groups/companies to progress through the lab. Based on your own preferences , your results may be different than what you see here. You are encouraged to ask some meaningful questions and parse the JSON around how YOU use twitter.**



###  Store the Tweets JSON File

It is always a good idea to store the tweets as a JSON file as executing above cells will bring in new tweets and will keep re-writing the object. This can have negative impacts on your analysis (unless you are analyzing live streams). 
* Store the `my_timeline_tweets` JSON object in `tweets.json` file.


```python
# Store my_timeline_tweets as a JSON file

import json
my_list_of_dicts = []
for each_json_tweet in my_timeline_tweets:
    my_list_of_dicts.append(each_json_tweet._json)

with open('tweets.json', 'w') as file:
    file.write(json.dumps(my_list_of_dicts, indent=4))
```

#### Open `tweets.json` and inspect its contents in relation to the JSON object shown above. 

## NATURAL LANGUAGE PROCESSING

Twitter data is of the most favorite dataset for data scientists and Natural Language Processing (NLP) researchers. Besides its magnanimous size, Twitter data has other unique qualities as well: it comprises of real-life conversations, limited word length, rich variety, and real-time data stream. Advanced analytics on Twitter data needs one to go beyond the words and parse sentences into syntactic representations to develop a better contextual understanding of the tweet content. We shall cover more of that in our sections dedicated to NLP. 

![](https://static1.squarespace.com/static/5552e2d6e4b098f713cf2613/t/56e3930327d4bdfdac7c34a9/1457754894624/?format=750w)

Here we shall look at some basic techniques that you might want to use for cleaning text data and look for important words, mentions, URLs  or hashtags etc.

#### Good old word count ..

A very fundamental NLP skill, without using any complicated algorithms is to count the occurrences of words/terms (known as **TERM FREQUENCIES** of TFs). With this, we can observe what are the terms most commonly used in the data set.

### Tokenization

Tokenization is an NLP specific term which concerns with breaking down a text document (like a tweet) into its tokens (words/terms). Below we have a simple custom tokenization routine that will parse each tweet and split it into words/terms to capture Twitter-specific aspects of the text, such as **#hashtags**, **@mentions**, **emoticons** and **URLs**. In order to tokenize , you simply need to pass each tweet to the `preprocess_tweet()` function below and it will be split into its constituent terms. 


```python
# Import Regular Expressions
import re

# Identify some basic emoticon elements (face, nose, mouth )- you are encouraged to add more for other emoticons
emoticons_str = r"""
    (?:
        [:=;] # Eyes
        [oO\-]? # Nose (optional)
        [D\)\]\(\]/\\OpP] # Mouth
    )"""

# Identify Twitter specific elements in a tweet
regex_str = [
    emoticons_str,
    r'<[^>]+>', # HTML tags
    r'(?:@[\w_]+)', # @-mentions
    r"(?:\#+[\w_]+[\w\'_\-]*[\w_]+)", # hash-tags
    r'http[s]?://(?:[a-z]|[0-9]|[$-_@.&amp;+]|[!*\(\),]|(?:%[0-9a-f][0-9a-f]))+', # URLs
 
    r'(?:(?:\d+,?)+(?:\.?\d+)?)', # numbers
    r"(?:[a-z][a-z'\-_]+[a-z])", # words with - and '
    r'(?:[\w_]+)', # other words
    r'(?:\S)' # anything else
]
    
tokens_re = re.compile(r'('+'|'.join(regex_str)+')', re.VERBOSE | re.IGNORECASE)
emoticon_re = re.compile(r'^'+emoticons_str+'$', re.VERBOSE | re.IGNORECASE)
 
def tokenize(s):
    return tokens_re.findall(s)
 
def preprocess_tweet(s, lowercase=False):
    tokens = tokenize(s)
    if lowercase:
        tokens = [token if emoticon_re.search(token) else token.lower() for token in tokens]
    return tokens
```

The code above uses a lot of regular expressions to search for twitter specific terms. Let's pass in a sample tweet to `preprocess_tweet` function and inspect the output.


```python
example_tweet = 'ExampleTweet @datascienceguy: You are an NLP superstar :D http://example.com #NLP #datascience :P'
print(preprocess_tweet(example_tweet))

```

    ['ExampleTweet', '@datascienceguy', ':', 'You', 'are', 'an', 'NLP', 'superstar', ':D', 'http://example.com', '#NLP', '#datascience', ':P']


So here you can see that each word, URL, emoticon has been sucessfully identified as a **Term**. Now we can do some basic analysis with these terms. 

### Count Term Frequencies

As a first step, we can simply count the total number of occurances of teach term. Keep in mind that at this stage , we are also counting punctuation marks and common words like "the" "of" "for etc. 

Perform following tasks. 
* Load the JSON file `tweets.json` into a new variable called `data`. 
* In order to keep track of the frequencies while we are processing the tweets, we can use `collections.Counter()`  from the `Counter` library, which behaves is a dictionary (term: count) with some useful methods like `update()` and `most_common()`. [Here is a link to official doc. for consultation](https://docs.python.org/2/library/collections.html).
* Iterate through the list of tweets update the term counter using `.update()` method for all terms after each iteration.
* Count all the terms and list top ten most common terms


```python
# Count all terms frequency from the saved JSON file

with open('tweets.json', encoding='utf-8') as json_file:  

    data = json.load(json_file)
    count_all = Counter()
    
    for tweet in data:
        
        # Create a list with all the terms
        terms_all = [term for term in preprocess_tweet(tweet['full_text'])]
        # Update the counter
        count_all.update(terms_all)

print(count_all.most_common(10))
        
```

    [(':', 142), ('.', 94), (',', 92), ('RT', 91), ('the', 81), ('to', 65), ('…', 65), ('of', 65), ('and', 64), ('a', 58)]


Right so in above output , we see that our punctuation marks and common terms like "the" and "of" etc. are being shown as top terms. The next step is to remove these words to focus on more IMPORTANT words. this step is called the "Stop-word Removal".

### Stop Word Removal

Every language has words that are very common. While their use in the language is very important, they do not carry any clear meaning on their own like conjunctions,  adverbs, etc. In the language of NLP , these are referred to as **Stop-words**. 
Stop-word removal is one important step that should be considered during the pre-processing stages. One can build a custom list of stop-words, or use available lists (e.g. [NLTK](https://www.nltk.org/) provides a simple list for English stop-words). You can search Internet for domain specific stop-words as well as some words may carry a deeper meaning in one domain while could be meaningless in another. (Try thinking of a few examples).

Given the nature of our data and our tokenization, we should also be careful with all the punctuation marks and with terms like RT (used for re-tweets), via (used to mention the original author of an article or a re-tweet), ... (used for ongoing tweet) etc. Below is a stop word list that we build from nltk + punctuation + some twitter specific stop words. Your are encouraged to add more to it. 


```python
# Create a stop word list from nltk stopwords, punctuation marks and some twitter specific stop terms
from nltk.corpus import stopwords
import string
punctuation = list(string.punctuation)
stop = stopwords.words('english') + punctuation + ['rt', 'RT',  'via', 'The', 'the', '…', '¡']
print(stop)
```

    ['i', 'me', 'my', 'myself', 'we', 'our', 'ours', 'ourselves', 'you', "you're", "you've", "you'll", "you'd", 'your', 'yours', 'yourself', 'yourselves', 'he', 'him', 'his', 'himself', 'she', "she's", 'her', 'hers', 'herself', 'it', "it's", 'its', 'itself', 'they', 'them', 'their', 'theirs', 'themselves', 'what', 'which', 'who', 'whom', 'this', 'that', "that'll", 'these', 'those', 'am', 'is', 'are', 'was', 'were', 'be', 'been', 'being', 'have', 'has', 'had', 'having', 'do', 'does', 'did', 'doing', 'a', 'an', 'the', 'and', 'but', 'if', 'or', 'because', 'as', 'until', 'while', 'of', 'at', 'by', 'for', 'with', 'about', 'against', 'between', 'into', 'through', 'during', 'before', 'after', 'above', 'below', 'to', 'from', 'up', 'down', 'in', 'out', 'on', 'off', 'over', 'under', 'again', 'further', 'then', 'once', 'here', 'there', 'when', 'where', 'why', 'how', 'all', 'any', 'both', 'each', 'few', 'more', 'most', 'other', 'some', 'such', 'no', 'nor', 'not', 'only', 'own', 'same', 'so', 'than', 'too', 'very', 's', 't', 'can', 'will', 'just', 'don', "don't", 'should', "should've", 'now', 'd', 'll', 'm', 'o', 're', 've', 'y', 'ain', 'aren', "aren't", 'couldn', "couldn't", 'didn', "didn't", 'doesn', "doesn't", 'hadn', "hadn't", 'hasn', "hasn't", 'haven', "haven't", 'isn', "isn't", 'ma', 'mightn', "mightn't", 'mustn', "mustn't", 'needn', "needn't", 'shan', "shan't", 'shouldn', "shouldn't", 'wasn', "wasn't", 'weren', "weren't", 'won', "won't", 'wouldn', "wouldn't", '!', '"', '#', '$', '%', '&', "'", '(', ')', '*', '+', ',', '-', '.', '/', ':', ';', '<', '=', '>', '?', '@', '[', '\\', ']', '^', '_', '`', '{', '|', '}', '~', 'rt', 'RT', 'via', 'The', 'the', '…', '¡']


Now we can simply use this list to clean up our tweets. Perform following tasks
* Clear count_all to start a new counter
* Change the routine above to collect tweets while filtering with above stop word list. 
* Use list comprehension to only include terms NOT found in the stop-word list 
* Print the top ten most common terms. 


```python
count_all.clear()    

for tweet in data:

    # Create a list with all the terms
    terms_minus_stop = [term for term in preprocess_tweet(tweet['full_text']) if term not in stop]

    # Update the counter
    count_all.update(terms_minus_stop)
print(count_all.most_common(10))
        
```

    [('Vote', 35), ('#ODSCWest', 29), ('Data', 26), ('#ODSC', 21), ('#DataScience', 18), ('’', 16), ('AI', 16), ('#AI', 14), ('@ODSC', 13), ('data', 12)]


So this looks much better, still not perfect though. Remember this is an iterative process and you would want to refine your stop word list. Here we see some useful terms like commonly mentioned hashtags etc. with lots of unwanted terms that xcould be included as stop words. Do some survey on other stop-word lists available online and include them with you own criteria. 

## More Term Filtering

Besides stop-word removal, we can further customize the list of terms we are interested in. 

### Calculate Document  Frequency

> The number of documents in the collection that contain a term t.

We can also calculate document frequency which shows us the number of documents (tweets) that show a specific term. For this we can make small tweaks to the code we used to calculate all terms frequencies for all document. Here we shall count every term only once in every document and see how many documents contain a specific term. For this you need to clear the counter, and perform following tasks for every tweet.

* Calculate frequencies for all terms while filtering out the stop words
* Make a set of unique terms (counted only once) for each tweet
* Update the counter 

* Show the top 20 terms from the counter


```python
count_all.clear()    

for tweet in data:
    
    terms_minus_stop = [term for term in preprocess_tweet(tweet['full_text']) if term not in stop]
    # Get unique terms
    terms_single = set(terms_minus_stop)    
    # Update the counter
    count_all.update(terms_single)
    
# Count terms only once, equivalent to Document Frequency
terms_single = set(terms_all)
print(count_all.most_common(20))

```

    [('#ODSCWest', 29), ('Data', 24), ('#ODSC', 21), ('#DataScience', 18), ('’', 14), ('AI', 14), ('#AI', 14), ('@ODSC', 13), ('data', 12), ('Science', 11), ('A', 11), ('2018', 11), ('West', 11), ('3', 9), ('amp', 9), ('Join', 8), ('Nov', 8), ('How', 8), ('#bordercollie', 8), ('Python', 7)]


Do you see any improvements in results ? You may need to work more on the stop word list and maybe filter out the numerical terms like dates and other numbers. You could also filter out terms with length less than a set threshold e.g. only include terms that carry 4 or more characters. Like we saw earlier, it is an iterative process and we may have to revisit our parsing code a few times to get more meaningful answers.

### Calculate Hashtag Frequencies

A Hashtag in a tweet is just another term , identified by a hash (#) sign. Let's iterate through our corpus of tweets again collecting all terms that are hashtags i.e. start with a #. Perform following tasks for each tweet:
* Reset the counter
* Collect the terms starting with # sign
* Update the counter
* List and visualize top 20 hashtags as bar charts and label properly


```python
count_all.clear()    

for tweet in data:
    
    # Count hashtags only
    terms_hash = [term for term in preprocess_tweet(tweet['full_text']) if term.startswith('#')]
    # Update the counter
    count_all.update(terms_hash)

print(count_all.most_common(20))
top20 = count_all.most_common(20)

# Plot top 20 hashtags

import matplotlib.pyplot as plt
plt.style.use('seaborn')
from pylab import rcParams
rcParams['figure.figsize'] =15, 5
plt.bar(range(len(top20)), [val[1] for val in top20], align='center')
plt.xticks(range(len(top20)), [val[0] for val in top20])
plt.xticks(rotation=70)
plt.title('Top 20 Hashtags')
plt.xlabel('Hashtags')
plt.ylabel('Frequency')
plt.show()

```

    [('#ODSCWest', 29), ('#ODSC', 21), ('#DataScience', 18), ('#AI', 14), ('#bordercollie', 8), ('#', 6), ('#abdsc', 4), ('#BigData', 4), ('#MachineLearning', 4), ('#datascience', 4), ('#BorderCollie', 4), ('#DeepLearning', 2), ('#Python', 2), ('#puppy', 2), ('#dogs', 2), ('#dogsoftwitter', 2), ('#collie', 2), ('#earlyvote', 2), ('#TeamAbrams', 1), ('#GAGov', 1)]



![png](index_files/index_38_1.png)


### Calculate Bi-gram Frequencies 

While the frequent terms post cleaning, stop removal etc. represent a clear topic, simple term frequencies don’t give us a deeper explanation of what the text is about. To put things in context, let’s consider sequences of two terms combined (a.k.a. bigrams).

> **A bigram is a sequence of two adjacent elements from a string of tokens, which are typically letters, syllables, or words. A bigram is an n-gram for n=2. The frequency distribution of every bigram in a string is commonly used for simple statistical analysis of text in many applications, including in computational linguistics, cryptography, speech recognition, and so on.**(wiki)

We can import bigrams function from nltk library to automatically create bigrams from a given sequence of text. Here is a quick demo:


```python
from nltk import bigrams 

text = 'This is a very simple sentence'
terms = [term for term in preprocess_tweet(text)]
terms_bigram = bigrams(terms)
list(terms_bigram)

```




    [('This', 'is'),
     ('is', 'a'),
     ('a', 'very'),
     ('very', 'simple'),
     ('simple', 'sentence')]



The combination of two words may help get the deeper understanding of the tweet. You could also calculate trigrams or even higher , but it might get computationally expensive. 

You are required to calculate the frequencies of bigrams in the collected tweets, and find the most frequent combination of two adjacent elements. Perform following tasks:

* Clear the counter 
* Collect all the terms excluding stop words, hashtags and mentions
* Calculte the bigrams for each tweet and update counter with frequencies
* Display and top 20 bigrams


```python
# Show the frequency of top 20 bigrams in the corpus
count_all.clear() 

for tweet in data:
    
    # Count terms only (no hashtags, no mentions)
    terms_only = [term for term in preprocess_tweet(tweet['full_text']) if term not in stop and not term.startswith(('#', '@'))] 
    # Update the counter
    terms_bigrams= bigrams(terms_only)
    count_all.update(list(terms_bigrams))
    
print (count_all.most_common(20))

```

    [(('Vote', 'Vote'), 33), (('Data', 'Science'), 11), (('San', 'Francisco'), 7), (('West', '2018'), 7), (('Oct', '31'), 5), (('31', 'Nov'), 5), (('Nov', '3'), 5), (('Don', '’'), 4), (('Francisco', 'Oct'), 4), (('Big', 'Data'), 4), (('machine', 'learning'), 3), (('data', 'scientists'), 3), (('data', 'science'), 3), (('Machine', 'Learning'), 3), (('Deep', 'Learning'), 3), (('Data', 'Scientist'), 3), (('LIVESTREAM', 'Pass'), 3), (('I', '’'), 3), (('\u2066', '\u2069'), 3), (('Cheat', 'Sheet'), 2)]


So we start to see something a bit more meaningful. For this case, we see more meaningul combinations of words which clearly indicate themes or recurring topics within tweets. A bit more effort can really get us results that can be easily analyzed to get some real insights. 

## Collecting Tweets From A User's Timeline - `user_timeline()` method


For some analyses, you may be interested in collecting tweets from someone else's timeline. We can use `user_timeline()` method to achieve this while passing `id=name` parameter to identify the user. Let's collect last 100 tweets from "FlatironSchool" and visualize top 10 hashtags and mentions. Apart from the new method `user_timeline()`, everything else will remain the same as above. Perform following tasks:

* Collect last 200 tweets from "FlatironSchool"
* Calculate top 10 hashtags and mentions as terms from collected tweets
* Visualize results as bargraphs in both cases


```python
# Calculate and Visulize top 10 hashtags in last 200 tweets from FlatironSchool's Timeline

# The Twitter profile You want to mine
name = "FlatironSchool"
# Number of tweets to pull
tweetCount = 200

# Calling the user_timeline function with our parameters
results = my_first_api.user_timeline(id=name, count=tweetCount, tweet_mode='extended')

count_all.clear()

for tweet in results:
    

        terms_hash = [term for term in preprocess_tweet(tweet.full_text) if term.startswith('#')]
        # Update the counter
        count_all.update(terms_hash)

top10 = count_all.most_common(10)

# Plot top 10 hashtags
import matplotlib.pyplot as plt
plt.style.use('seaborn')
from pylab import rcParams
rcParams['figure.figsize'] =15, 5
plt.bar(range(len(top10)), [val[1] for val in top10], align='center')
plt.xticks(range(len(top10)), [val[0] for val in top10])
plt.xticks(rotation=70)
plt.title('Top 10 Hashtags from FlatironSchool Timeline')
plt.xlabel('Hashtags')
plt.ylabel('Frequency')
plt.show()

```


![png](index_files/index_45_0.png)



```python
# Calculate and Visulize top 10 mentions in last 200 tweets from FlatironSchool's Timeline 
# Exclude @FlatironSchool mentions 

count_all.clear()
for tweet in results:
    
        terms_hash = [term for term in preprocess_tweet(tweet.full_text) \
                      if (term.startswith('@') and \
                          term != "@FlatironSchool")]
        
        count_all.update(terms_hash)

top10 = count_all.most_common(10)

# Plot top 20 hashtags

import matplotlib.pyplot as plt
plt.style.use('seaborn')
from pylab import rcParams
rcParams['figure.figsize'] =15, 5
plt.bar(range(len(top10)), [val[1] for val in top10], align='center')
plt.xticks(range(len(top10)), [val[0] for val in top10])
plt.xticks(rotation=70)
plt.title('Top 10 Mentions at FlatironSchool Timeline')
plt.xlabel('Hashtags')
plt.ylabel('Frequency')
plt.show()

```


![png](index_files/index_46_0.png)


## Collecting Tweets Using a Keyword -  `.search()` method

Let’s look at one last use case i.e. getting the most recent tweets that contain a specific keyword. This can be extremely useful if you want to monitor specifically mentioned topics in the Twitter world, or even to see how your business is getting mentioned. We shall use Twitter standard serach API whith `q=query` parameter to pass our search query.  

Let’s say we want to see how FlatironSchool is being been mentioned on Twitter. We would like to see which Twitter  users are being mentioned in conjunction with FlatironSchool. Remember maxiumum number of tweets you can mine is 180 per 15 minutes window. 

Perform following tasks
* Clear the counter
* Collect 200 tweets that mention Flatiron School using `.search()` method. (Use `language=en` to collect english tweets only)
* Collect the top 30 @mentions from the all tweets that mention FlatironSchool.
* Visualize the results as a wordcloud


```python
# The search term you want to find
query = "FlatironSchool"
# Language code (follows ISO 639-1 standards)
language = "en"
tweetCount = 200

# Calling the user_timeline function with our parameters

results = my_first_api.search(q=query, lang=language, tweet_mode='extended', count=tweetCount)

count_all.clear()
for tweet in results:
    terms_hash = [term for term in preprocess_tweet(tweet.full_text) if term.startswith('@')]
    count_all.update(terms_hash)

top20 = count_all.most_common(20)
print(top20)
```

    [('@FlatironSchool', 83), ('@fulmolightning', 52), ('@pierre_rochard', 21), ('@peterktodd', 19), ('@MsJisola', 17), ('@ElectrumWallet', 17), ('@BtcpayServer', 14), ('@r0ckstardev', 13), ('@Blockstream', 12), ('@Excellion', 12), ('@rootzoll', 11), ('@CryptoCloaks', 8), ('@CasaHODL', 7), ('@overtorment', 6), ('@CReckhow', 5), ('@starkness', 4), ('@CodeNewbies', 4), ('@TeachFirst', 4), ('@jeetsidhu_', 3), ('@Snyke', 3)]


Another common way to view dominant terms in a corpus of text is to visualize a wordcloud of top n terms. Here is an API reference for [WordCloud for Python](https://amueller.github.io/word_cloud/references.html). 
* Create a word cloud from top 20 mentions above to highlight who is being mentioned most. 


```python
import numpy as np
from PIL import Image
from os import path
import wordcloud
mask = np.array(Image.open("FS.png"))
cloud = wordcloud.WordCloud(max_font_size=50, max_words=100, background_color="white")

cloud.generate_from_frequencies(dict(top10))

plt.figure(figsize=(15,10))
plt.imshow(cloud, interpolation="bilinear")
plt.axis("off")
plt.show()
```


![png](index_files/index_50_0.png)


This looks great. Try collecting a large number of tweets using the cfietria seen previously and create a more populated word cloud. You can also customize and prettify it to your heart's content. 

## From JSON to DataFrames

So far, we have been processing our tweets directly from the JSON files. It may be desired for an advanced analystical experiment to store data as pandas dataframe and enjoy all the goodness of pandas, scikitlearn and buult in statistics. Let's try to select some selected parts of the tweet and save it nicely as pandas dataframe and csv file. 

#### Load `tweets.json`  and save following contents as pandas dataframe `tweet_json_df`
    * id
    * full_text
    * favorite_count
    * retweet_count
    * created_at

* Set some meaningful names for dataframe columns to store these values.
* Print the dataframe head to inspect the contents
* save data as "tweets.csv"


```python
# Parse the JSON file, selecting above attributes for each tweet and saving as a new row in a pandas dataframe
import pandas as pd
my_tweets_list = []
with open('tweets.json', encoding='utf-8') as json_file:  
    all_data = json.load(json_file)
    for each_dictionary in all_data:
        tweet_id = each_dictionary['id']
        text = each_dictionary['full_text']
        favorite_count = each_dictionary['favorite_count']
        retweet_count = each_dictionary['retweet_count']
        created_at = each_dictionary['created_at']
        my_tweets_list.append({'tweet_id': str(tweet_id),
                             'text': str(text),
                             'favorite_count': int(favorite_count),
                             'retweet_count': int(retweet_count),
                             'created_at': created_at,
                            })
        #print(my_demo_list)
        tweet_json_df = pd.DataFrame(my_tweets_list, 
                        columns = ['tweet_id', 'text', 
                                   'favorite_count', 
                                   'retweet_count', 
                                   'created_at'])

tweet_json_df.to_csv('tweets.csv')
tweet_json_df.head()
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
      <th>tweet_id</th>
      <th>text</th>
      <th>favorite_count</th>
      <th>retweet_count</th>
      <th>created_at</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1056647627062661121</td>
      <td>RT @RalstonReports: The turnout models are loo...</td>
      <td>0</td>
      <td>15</td>
      <td>Sun Oct 28 20:43:20 +0000 2018</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1056647041093373952</td>
      <td>[Cheat Sheet] Python Basics For Data Science h...</td>
      <td>2</td>
      <td>1</td>
      <td>Sun Oct 28 20:41:00 +0000 2018</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1056640802401615872</td>
      <td>RT @Popehat: NYT:  Well, Okay, But Are People ...</td>
      <td>0</td>
      <td>620</td>
      <td>Sun Oct 28 20:16:13 +0000 2018</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1056639940438544384</td>
      <td>https://t.co/txeJ3g9IMW</td>
      <td>1</td>
      <td>1</td>
      <td>Sun Oct 28 20:12:47 +0000 2018</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1056638994203729922</td>
      <td>IBM and NVIDIA collaborate to expand machine l...</td>
      <td>3</td>
      <td>1</td>
      <td>Sun Oct 28 20:09:02 +0000 2018</td>
    </tr>
  </tbody>
</table>
</div>



So there we have it. The skills that you have learned in this lab can be used towards data journalism, data science related articles etc. These skills will also come in handy for more advanced NLP techniques like topic modelling, sentiment analysis, emoticon analysis etc. 

## Level Up - Optional 

* Try generating more wordclouds from all the examples we saw in this lab. 
* Ask yourself an analytical question and mine Twitter to get tweets, process them in tweepy and visualize the results. 
* Count emoticons for smily face, frowning face, excited face etc. and see what are people's sentiments about a specific topic of interest i.e. Brexit, War in Syria, Best actor from last Oscars etc. 
* See which Twitter member is mentioning you the most at Twitter.
* ON-going : Repeatedly mine twitter (say once per day) on a topic of interest and collect,parse,visualize tweets to see any changing trends. 


## Further Reading 

* [Introduction to NLP](https://blog.algorithmia.com/introduction-natural-language-processing-nlp/)
* [Python/NLTK Language Processing](http://blog.chapagain.com.np/python-nltk-twitter-sentiment-analysis-natural-language-processing-nlp/)
* [Applying NLP to Twitter in Python](https://dzone.com/articles/applying-nlp-to-tweets-with-python)
* [Using the Twitter API and NLP to analyze the tweets of different users.](https://towardsdatascience.com/twitter-api-and-nlp-7a386758eb31)

## Summary 

In this lab, we saw how to collect data from twitter using OAuth and collect Tweets under for a number of different use cases. We looked at parsing the JSON file for collecting relevant information and also visualized the outcome of our processing. As a reminder, this lab is supposed to be a data engineering oriented lab, i.e. getting data ready for processing. We shall see more on NLP later in next module of the course. This lab should get you all ready for some serious experiments like sentiment analysis , predictions and classification tasks. 
