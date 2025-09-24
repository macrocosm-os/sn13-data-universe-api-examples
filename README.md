# sn13-data-universe-api-examples
Macrocosmos API &amp; SDK Usage examples for SN13 Data Universe

## Setup
- Grab your API key from [Account Settings > Api Keys](https://app.macrocosmos.ai/account?tab=api-keys) 
- First 5$ of credits is on us, when you register
- Put this into your environment or a `.env` file

## Install the SDK

### Python
The official python sdk:
- [pypi: macrocosmos](https://pypi.org/project/macrocosmos/)
- [github: macrocosm-os/macrocosmos-py](https://github.com/macrocosm-os/macrocosmos-py)

```sh
pip install macrocosmos
```

### TypeScript
It's also available for typescript:
- [npm: macrocosmos](https://www.npmjs.com/package/macrocosmos)
- [github: macrocosm-os/macrocosmos-ts](https://github.com/macrocosm-os/macrocosmos-ts)

```sh
npm install macrocosmos
```

## Examples
You can find examples under the `examples/` directory of this repository.

Feel free to copy/clone them and use, or, get inspired by them and move on as you wish.

Right now, the examples contain sample code to use:

- The Real Time OnDemand API: pulling up to 1000 unique tweets per datetime (up to minute precision) matching your filters (usernames, keywords, hashtags, etc), including impression data (likes, retweets, etc)

- The Async Offline "Jobs" (or "Gravity Jobs"), which is identical to the [jobs creation UI on the constellation platform](https://app.macrocosmos.ai/gravity/tasks), where you can pull massive amounts of data, after waiting for a couple of hours for the scraping to finish.

## Data Sources
Currently, we support the following sources:
- X
- Reddit
- YouTube (Beta)

and there are plans to expand this list to also include real time web search and others