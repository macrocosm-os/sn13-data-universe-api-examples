# Macrocosmos SN13 Data Universe SDK & API Usage
A small coding guide on how to use the macrocosmos python sdk in order to pull scraped social media data from x, reddit and youtube, from the bittensor subnet #13.
Miners pull this data, and it's made available through the constellation platform, which is what the `pip install macrocosmos` python package does.

## PyPi Package Overview
The `macrocosmos` python package for subnet 13 is at:
- [pypi: macrocosmos](https://pypi.org/project/macrocosmos/)
- [github: macrocosm-os/macrocosmos-py](https://github.com/macrocosm-os/macrocosmos-py)

And includes 2 clients to help you pull data:

- The On Demand API, which is the realtime (sub-minute latency) API, to pull up to 1000 rows of data at a time:
```py
client = macrocosmos.AsyncSn13Client(api_key=<your api key>)
await client.sn13.OnDemandData(...)
```

- The Async / Offline Gravity Jobs API, which is a multi-step async API that lets you pull unlimited amounts of data
```py
client = macrocosmos.GravityClient(api_key=os.environ.get("MACROCOSMOS_API_KEY"))

client.gravity.CreateGravityTask(...)
client.gravity.GetGravityTasks(...)
client.gravity.BuildDataset(...)
client.gravity.GetDataset(...)
```

## How to get API Key
- Grab your API key from [Account Settings > Api Keys](https://app.macrocosmos.ai/account?tab=api-keys) 
- First 5$ of credits is on us, when you register
- Put this into your environment or a `.env` file

## Docs Reference
There are official docs you can browse for usage samples: https://docs.macrocosmos.ai/developers/readme/gravity and API Reference

## Sample Code

### API: On Demand

```py
import pandas as pd

from datetime import datetime, timezone, timedelta
from dotenv import load_dotenv
import os
import macrocosmos

load_dotenv()

client = macrocosmos.AsyncSn13Client(api_key=os.environ.get("MACROCOSMOS_API_KEY"))

end_dt = datetime.now(timezone.utc)
start_dt = end_dt - timedelta(days=7) # up to minute precision 

# sample for twitter/x
resp = await client.sn13.OnDemandData(
    source='x', # or 'reddit', 'youtube' -- more examples on following sections
    usernames=None, # None or username(s) ex. ['elonmusk'] or ['elonmusk', 'MikeTyson']
    keywords=["#bitcoin"], # keyword (ex. 'ai'), hashtag (ex. '#bittensor'), or cashtag ('$AAPL')
    start_date=start_dt.date().isoformat(),
    end_date=end_dt.date().isoformat(),  # up to minute precision
    limit=10, # up to 1000
    keyword_mode=None # 'any' or 'all', for all supported platforms
)

# sample for reddit
resp = await client.sn13.OnDemandData(
    source='reddit',
    usernames=None,
    keywords=["r/Bitcoin"], # to combine subreddit and keywords/hashtags, you can pass a list ex. ['r/Bitcoin', 'ethereum']
    start_date=start_dt.date().isoformat(),
    end_date=end_dt.date().isoformat(),
    limit=100, # up to 1000
)

# sample for youtube
resp = await client.sn13.OnDemandData(
    source='youtube',
    usernames=['MrBeast'],
    keywords=None,
    start_date=start_dt.date().isoformat(),
    end_date=end_dt.date().isoformat(),
    limit=100, # up to 1000
)

# you can load to data frame instantly
df = pd.json_normalize(data)
```

Sample Response and dtypes (always check reference -- this is just a sample and may lack more up-to-date additions of fields)

A successful response in general looks like this (across platforms):
```
{'status': 'success',
 'data': [{'text': 'Spring loaded \n$128,000 next \n#bitcoin',
   'datetime': '2025-09-23T23:59:57+00:00',
   'uri': 'https://x.com/NetHogger/status/1970639304105042187',
   'source': 'X',
   'tweet': {'quote_count': 1.0,
    'id': '1970639304105042187',
    'quoted_tweet_id': None,
    'retweet_count': 0.0,
    'like_count': 0.0,
    'conversation_id': '1970639304105042187',
    'hashtags': ['#bitcoin'],
    'reply_count': 0.0,
    'is_quote': False,
    'bookmark_count': 0.0,
    'in_reply_to_user_id': None,
    'language': 'en',
    'is_retweet': False,
    'is_reply': False,
    'view_count': 229.0,
    'in_reply_to': None,
    'in_reply_to_username': None},
   'content_size_bytes': 1090.0,
   'user': {'cover_picture_url': 'https://pbs.twimg.com/profile_banners/1434219190525726720/1743610028',
    'verified': False,
    'id': '1434219190525726720',
    'following_count': 9.0,
    'user_location': 'California, USA',
    'display_name': 'NetHogger',
    'followers_count': 960.0,
    'user_description': 'Comics | Quant 🇸🇻🇬🇹🇺🇸| E-Commerce|Find me on eBay link in bio',
    'user_blue_verified': False,
    'profile_image_url': 'https://pbs.twimg.com/profile_images/1954015153571475460/X8fSy1NZ_normal.jpg',
    'username': 'NetHogger'},
   'label': '#bitcoin'},],
 'meta': {'miners_queried': 5.0,
  'empty_response_miners': 0.0,
  'validation_success_rate': '0/10',
  'miners_responded': 5.0,
  'items_returned': 10.0,
  'non_responsive_miners': 0.0,
  'processing_time_ms': -0.000244140625,
  'api_version': '1.0'}}
```

Check `resp['status'] == 'success'` and `not resp['meta'].get('no_data_available', False)}"` to check if there are data available for your query.

If you load a succesful response to a dataframe, you can check the columns (might be missing latest column additions, always check real data for this):

```
# For Twitter/X
text                           object
datetime                       object
uri                            object
source                         object
content_size_bytes            float64
label                          object
tweet.quote_count             float64
tweet.id                       object
tweet.quoted_tweet_id          object
tweet.reply_count             float64
tweet.like_count              float64
tweet.conversation_id          object
tweet.hashtags                 object
tweet.retweet_count           float64
tweet.is_quote                   bool
tweet.bookmark_count          float64
tweet.in_reply_to_user_id      object
tweet.language                 object
tweet.is_retweet                 bool
tweet.is_reply                   bool
tweet.view_count              float64
tweet.in_reply_to              object
tweet.in_reply_to_username     object
user.cover_picture_url         object
user.verified                    bool
user.id                        object
user.user_location             object
user.following_count          float64
user.display_name              object
user.followers_count          float64
user.user_description          object
user.user_blue_verified          bool
user.profile_image_url         object
user.username                  object
dtype: object

# For Reddit
id                     object
url                    object
upvote_ratio          float64
communityName          object
media                  object
label                  object
num_comments          float64
datetime               object
uri                    object
source                 object
dataType               object
content_size_bytes    float64
title                  object
createdAt              object
score                 float64
parentId               object
body                   object
username               object
is_nsfw                  bool
```

You can use the text contents, or even analyse metrics and users for this, for example, most influential users by follower count, most liked posts, trending (if you send multiple requests), etc.

### API: Gravity Jobs

1. First, Create a job

- Gravity Tasks follows a similar pattern to OnDemand, so you can add channel names, subreddits, etc
- It's suggested to start with the OnDemand API to figure out what you wish to scrape first

Here is the current reference (might be out of date/new fields might be here)
```py
class GravityTask(BaseModel):
    """
     GravityTask defines a crawler's criteria for a single job (platform/topic)
    """

# topic: the topic of the job (e.g. '#ai' for X, 'r/ai' for Reddit)
    topic: typing.Optional[str] = Field(default="")
# platform: the platform of the job ('x' or 'reddit')
    platform: str = Field(default="")
# keyword: the keyword to search for in the job (optional)
    keyword: typing.Optional[str] = Field(default="")
# post_start_datetime: the start date of the job (optional)
    post_start_datetime: typing.Optional[datetime] = Field(default_factory=datetime.now)
# post_end_datetime: the end date of the job (optional)
    post_end_datetime: typing.Optional[datetime] = Field(default_factory=datetime.now)
```

```py
end_dt = datetime.now(timezone.utc)
start_dt = end_dt - timedelta(days=7)

task_name = f"<Display Title> - {datetime.now(timezone.utc).strftime('%Y-%m-%d %H:%M:%SZ')}"

gravity_tasks = [
                 {"topic": "#ai", "platform": "x", "post_start_datetime": start_dt, "post_end_datetime": end_dt}, 
                 {"topic": "#ai", "platform": "reddit", "post_start_datetime": start_dt, "post_end_datetime": end_dt},
                 {"topic": "MrBeast", "platform": "youtube", "post_start_datetime": start_dt, "post_end_datetime": end_dt}
                 ]

notification_requests=[{
        'type': 'email',
        'address': '<your email>@<>.com'
    }]


response = client.gravity.CreateGravityTask(
    gravity_tasks=gravity_tasks,
    name=task_name,
    notification_requests=notification_requests
)
task_id = response.gravity_task_id

# it might be useful to persist this on a local json file
with open('job.json', 'w') as f:
    f.write(json.dumps({
        'gravity_task_id': task_id
    }))
```

2. Poll the job status
It might take up to 1-2h for new data rows to be scraped for your job, till miners pick it up.

```py
resp = client.gravity.GetGravityTasks(
    gravity_task_id=task_id,
    include_crawlers=True  # bring crawler details/counts if available
)
```

Your response looks somewhat like this: You will get 1 dataset for each job.
```
gravity_task_states {
  gravity_task_id: "multicrawler-532b5b59-ac58-4d9e-9806-3f0aa93beffe"
  name: "Macrocosmos Sample - 2025-09-25 09:00:20Z"
  status: "Running"
  start_time {
    seconds: 1758790820
    nanos: 771210000
  }
  crawler_ids: "crawler-0-multicrawler-532b5b59-ac58-4d9e-9806-3f0aa93beffe"
  crawler_ids: "crawler-1-multicrawler-532b5b59-ac58-4d9e-9806-3f0aa93beffe"
  crawler_ids: "crawler-2-multicrawler-532b5b59-ac58-4d9e-9806-3f0aa93beffe"
  crawler_workflows {
    crawler_id: "crawler-0-multicrawler-532b5b59-ac58-4d9e-9806-3f0aa93beffe"
    criteria {
      platform: "x"
      topic: "#ai"
      notification {
        to: "theo.ntakouris@macrocosmos.ai"
        link: "https://app.macrocosmos.ai/gravity/datasets/multicrawler-532b5b59-ac58-4d9e-9806-3f0aa93beffe"
      }
      user_id: "c6d07878-ea54-4592-9677-7988de1bddc3"
      keyword: ""
      post_start_datetime {
        seconds: 1758183311
      }
      post_end_datetime {
        seconds: 1758788111
      }
    }
    start_time {
      seconds: 1758790821
      nanos: 75891000
    }
    deregistration_time {
      seconds: 1759395621
      nanos: 75891000
    }
    archive_time {
      seconds: 1761382821
      nanos: 75891000
    }
    state {
      status: "Running"
      bytes_collected: 316910
      records_collected: 766
    }
  }
  crawler_workflows {
    crawler_id: "crawler-1-multicrawler-532b5b59-ac58-4d9e-9806-3f0aa93beffe"
    criteria {
      platform: "reddit"
      topic: "#ai"
      notification {
        to: "theo.ntakouris@macrocosmos.ai"
        link: "https://app.macrocosmos.ai/gravity/datasets/multicrawler-532b5b59-ac58-4d9e-9806-3f0aa93beffe"
      }
      user_id: "c6d07878-ea54-4592-9677-7988de1bddc3"
      keyword: ""
      post_start_datetime {
        seconds: 1758183311
      }
      post_end_datetime {
        seconds: 1758788111
      }
    }
    start_time {
      seconds: 1758790821
      nanos: 96578000
    }
    deregistration_time {
      seconds: 1759395621
      nanos: 96578000
    }
    archive_time {
      seconds: 1761382821
      nanos: 96578000
    }
    state {
      status: "Running"
      bytes_collected: 182
    }
  }
  crawler_workflows {
    crawler_id: "crawler-2-multicrawler-532b5b59-ac58-4d9e-9806-3f0aa93beffe"
    criteria {
      platform: "youtube"
      topic: "mrbeast"
      notification {
        to: "theo.ntakouris@macrocosmos.ai"
        link: "https://app.macrocosmos.ai/gravity/datasets/multicrawler-532b5b59-ac58-4d9e-9806-3f0aa93beffe"
      }
      user_id: "c6d07878-ea54-4592-9677-7988de1bddc3"
      keyword: ""
      post_start_datetime {
        seconds: 1758183311
      }
      post_end_datetime {
        seconds: 1758788111
      }
    }
    start_time {
      seconds: 1758790821
      nanos: 60603000
    }
    deregistration_time {
      seconds: 1759395621
      nanos: 60603000
    }
    archive_time {
      seconds: 1761382821
      nanos: 60603000
    }
    state {
      status: "Running"
      bytes_collected: 158323
      records_collected: 2
    }
  }
}
```

You can quickly probe collected rows:

```py
task_state = resp.gravity_task_states[0]
print(f'Status: {task_state.status}')

for wf in task_state.crawler_workflows:
    print(f'Workflow: {wf.crawler_id} ({wf.criteria.platform})')
    print(f'Collected Rows: {wf.state.records_collected}')
```

3. Build your dataset(s)
Once you're happy with the amount of data or jobs can't pick up more data, you can build your datasets. Jobs expire in 7d if you don't build.

```py
workflow_ids_per_platform = {wf.criteria.platform: wf.crawler_id for wf in task_state.crawler_workflows}
workflow_ids_per_platform

# here we only build for X, but you can get it for all.
resp = client.gravity.BuildDataset(
    crawler_id=workflow_ids_per_platform['x'],
    max_rows=1000, # uncapped
    notification_requests=notification_requests
)

dataset_id = resp.dataset_id

# it might be a good idea to persist this to the same job json file:

with open('job.json', 'w') as f:
    f.write(json.dumps({
        'gravity_task_id': task_id,
        'workflow_ids_per_platform': workflow_ids_per_platform,
        'dataset_id': dataset_id
    }))
```

Response looks like:
```
dataset_id: "dataset-47b3581f-1e44-4dda-9d77-c0780612eac8"
dataset {
  crawler_workflow_id: "crawler-0-multicrawler-532b5b59-ac58-4d9e-9806-3f0aa93beffe"
  create_date {
    seconds: 1758873456
    nanos: 766847000
  }
  status: "Running"
  status_message: "Initializing"
  steps {
    step_name: "Initializing"
  }
  total_steps: 10
}
```

4. Download your dataset(s)

```py
dataset_resp = client.gravity.GetDataset(dataset_id=dataset_id)
```

Response should look somewhat like:
```
dataset {
  crawler_workflow_id: "crawler-0-multicrawler-532b5b59-ac58-4d9e-9806-3f0aa93beffe"
  create_date {
    seconds: 1758873456
    nanos: 766847000
  }
  expire_date {
  }
  files {
    file_name: "data_2025-09-26_x_377dq5zw.parquet"
    file_size_bytes: 105096
    last_modified {
      seconds: 1758873477
    }
    num_rows: 306
    s3_key: "production_datasets/rift/datetime=2025-09-26/platform=x/label=ai/keyword=no_keyword_requested/chunk=jiql6q1n/data_2025-09-26_x_377dq5zw.parquet"
    url: "https://data-universe-storage.nyc3.digitaloceanspaces.com/production_datasets/rift/datetime=2025-09-26/platform=x/label=ai/keyword=no_keyword_requested/chunk=jiql6q1n/data_2025-09-26_x_377dq5zw.parquet"
  }
  status: "Completed"
  status_message: "Dataset ready for download"
  steps {
    progress: 1
    step: 8
    step_name: "Billing correction"
  }
  total_steps: 10
  nebula {
    file_size_bytes: 537083
    url: "https://data-universe-storage.nyc3.digitaloceanspaces.com/nebulas/production_datasets/datetime=2025-09-26/platform=x/label=ai/keyword=no_keyword_requested/chunk=50i0225n/nebula.parquet"
  }
}
```

If your dataset has 1 shard (usual for most "small" datasets)
```py
df = pd.read_parquet(dataset_resp.dataset.files[0].url, engine='fastparquet')
```

Else, you have to pull all the `files[]` shards and combine them.

### Loading Jobs Files
If a user has downloaded file(s) from either the previous "API: Gravity Jobs" section (via the SDK), or clicked and downloaded data manually from our [Constellation Gravity Tasks](https://app.macrocosmos.ai/gravity/tasks) platform, you can very easily load these files (csvs, or parquet - preferred).

Use the `fastparquet` engine if possible.

The gravity jobs dataset have the same schema as the `OnDemand` data provided above.
