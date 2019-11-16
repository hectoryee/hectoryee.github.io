---
title: Scraping Tweets for Sentiment Analysis
excerpt: Scraping tweets from Twitter using Tweepy and Twitter API. Script is hosted on Amazon EC2.
published: true
date: 2018-09-24
categories: project
tags: intern twitter web-scraping
---

Twitter has many a huge amount of data that is suitable to perform analysis. In this post I'll be using Tweepy to scrape Malaysian tweets. The collected data will be used for sentiment analysis.

## Twitter API
Apply for Twitter realtime streaming API from [developer website](https://developer.twitter.com/en/apps).

Create a new app and fill in the details required. Twitter will take some time to reply your application. Mine took two weeks.

I got my keys and access tokens after the application being approved.

## Dependencies
Libraries used:

- Tweepy is the main Python library I used to access the Twitter API.

- Boto3 is the Amazon Web Services (AWS) Software Development Kit (SDK) for Python, which allows Python developers to write software that makes use of services like Amazon S3 and Amazon EC2.

``` python
import datetime
import tweepy
import json
import sys
import time
import boto3

s3 = boto3.resource("s3")
bucket_name = "malaysian-tweets"
```

## Applying Twitter API
Keys and access tokens are from twitter developer website.
```python
# Authentication details. To  obtain these visit apps.twitter.com
consumer_key='your_key'
consumer_secret='your_secret'
access_token='your_token'
access_token_secret='your_token_secret'
```

## Downloading Tweets
Use Tweepy to listen to streaming Twitter data.
```python
# This is the listener, resposible for receiving data
class StdOutListener(tweepy.StreamListener):
    # def __init__(self):
    #     self.start_time = time.time()
    #     self.file_name = datetime.datetime.now().strftime("%m-%d.json")

    def on_data(self, data):
        # Parsing
        decoded = json.loads(data)
        # if "country_code" in decoded["place"]:
        #     if not decoded["place"]["country_code"] == "MY":
        #         return True
        file_name = datetime.datetime.now().strftime("%m-%d-%s.json")
        #open a file to store the status objects
        file = open(file_name, 'a')
        #write json to file
        json.dump(decoded,file,sort_keys = True,indent = 4)
        file.write(',\n')
        file.close()
        # object_name = sys.argv[2]
        try:
            response = s3.Object(bucket_name, file_name).put(Body=open(file_name, 'rb'))
            print (response)
        except Exception as error:
            print (error)
        #show progress
        # print ("Writing tweets to file,CTRL+C to terminate the program")
        # if time.time() - self.start_time > 1 * 60:
        #     self.start_time = time.time()
        return True

    def on_error(self, status):
        print (status)

if __name__ == '__main__':
    l = StdOutListener()
    auth = tweepy.OAuthHandler(consumer_key, consumer_secret)
    auth.set_access_token(access_token, access_token_secret)

    # There are different kinds of streams: public stream, user stream, multi-user streams
    # For more details refer to https://dev.twitter.com/docs/streaming-apis
    stream = tweepy.Stream(auth, l)
    #Hashtag to stream
    stream.filter(locations=[98.933, 0, 124.125, 15])
    # stream.filter(track = "M")
```

### Example Tweets
Example tweets  returned in json format.
```
{
    "contributors": null,
    "coordinates": null,
    "created_at": "Mon Oct 01 13:57:15 +0000 2018",
    "entities": {
        "hashtags": [],
        "symbols": [],
        "urls": [],
        "user_mentions": []
    },
    "favorite_count": 0,
    "favorited": false,
    "filter_level": "low",
    "geo": null,
    "id": 1046760960537452549,
    "id_str": "1046760960537452549",
    "in_reply_to_screen_name": null,
    "in_reply_to_status_id": null,
    "in_reply_to_status_id_str": null,
    "in_reply_to_user_id": null,
    "in_reply_to_user_id_str": null,
    "is_quote_status": false,
    "lang": "th",
    "place": {
        "attributes": {},
        "bounding_box": {
            "coordinates": [
                [
                    [
                        100.297188,
                        13.814769
                    ],
                    [
                        100.297188,
                        13.909759
                    ],
                    [
                        100.448393,
                        13.909759
                    ],
                    [
                        100.448393,
                        13.814769
                    ]
                ]
            ],
            "type": "Polygon"
        },
        "country": "Thailand",
        "country_code": "TH",
        "full_name": "Amphoe Bang Yai, Changwat Nonthaburi",
        "id": "0008c5c351581976",
        "name": "Amphoe Bang Yai",
        "place_type": "admin",
        "url": "https://api.twitter.com/1.1/geo/id/0008c5c351581976.json"
    },
    "quote_count": 0,
    "reply_count": 0,
    "retweet_count": 0,
    "retweeted": false,
    "source": "<a href=\"http://twitter.com/download/iphone\" rel=\"nofollow\">Twitter for iPhone</a>",
    "text": "\u0e09\u0e31\u0e19\u0e0a\u0e27\u0e19\u0e04\u0e38\u0e22\u0e44\u0e21\u0e48\u0e40\u0e01\u0e48\u0e07\u0e2d\u0e30;\u2014\u2014;",
    "timestamp_ms": "1538402235527",
    "truncated": false,
    "user": {
        "contributors_enabled": false,
        "created_at": "Wed Sep 30 04:17:36 +0000 2015",
        "default_profile": false,
        "default_profile_image": false,
        "description": "\u2601\ufe0f\ud83d\udc0b",
        "favourites_count": 1759,
        "follow_request_sent": null,
        "followers_count": 141,
        "following": null,
        "friends_count": 323,
        "geo_enabled": true,
        "id": 3732991574,
        "id_str": "3732991574",
        "is_translator": false,
        "lang": "en",
        "listed_count": 0,
        "location": null,
        "name": "n",
        "notifications": null,
        "profile_background_color": "000000",
        "profile_background_image_url": "http://abs.twimg.com/images/themes/theme1/bg.png",
        "profile_background_image_url_https": "https://abs.twimg.com/images/themes/theme1/bg.png",
        "profile_background_tile": false,
        "profile_image_url": "http://pbs.twimg.com/profile_images/1045137900428378112/xD16zw85_normal.jpg",
        "profile_image_url_https": "https://pbs.twimg.com/profile_images/1045137900428378112/xD16zw85_normal.jpg",
        "profile_link_color": "EFA694",
        "profile_sidebar_border_color": "000000",
        "profile_sidebar_fill_color": "000000",
        "profile_text_color": "000000",
        "profile_use_background_image": false,
        "protected": false,
        "screen_name": "new_3k",
        "statuses_count": 71900,
        "time_zone": null,
        "translator_type": "none",
        "url": null,
        "utc_offset": null,
        "verified": false
    }
}
```

## Run on Cloud

### Amazon EC2
Streaming API capture tweets in realtime, thus the code has to run continuously. Save the code and upload to Amazon EC2 and let it run.

We can [access EC2 instance using SSH](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html).

EC2 will stop running once we log out from terminal. Options are using Tmux or Screen. Using Screen:

- Start a screen session: `screen`
- Start a named session: `screen -S session_name`
- Detach from current screen session: `Ctrl+a` `d`
- List all screen session: `screen -ls`
- Reattach screen session: `screen -r session_id`

### Run on Restart
To ensure the file runs at all time:
```python
#!/usr/bin/python3
from subprocess import Popen
import sys

filename = "/home/ubuntu/streams.py"

while True:
    print("\nStarting " + filename)
    p = Popen("python3 " + filename, shell=True)
    p.wait()
```
Add to crontab so it starts when reboot.
```sh
crontab -e
```
Ad this line to cron file.
```
@reboot /path/forever
```

## All Codes

{% gist 8fb39e3e3a27d7832678ed16c7803d6d %}
{% gist 107c73c4a15d72682d7b11266de0e595 %}