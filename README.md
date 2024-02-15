# Watchlistarr

<img src="watchlistarr.png" alt="Alternate Text" width="240"/>

Sync plex watchlists in realtime with Sonarr and Radarr.

![watchlistarr-gif](https://github.com/nylonee/watchlistarr/assets/4732553/3ff083f2-5c5d-411b-8280-ea76958542bf)

## How it works

There are several ways of fetching watchlist information from Plex

| Method                                  | Pros                                                                    | Cons                                                                                                            | Supported by Watchlistarr | Supported by Overseer/Ombi   | Supported by Sonarr/Radarr             |
|-----------------------------------------|-------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------|---------------------------|------------------------------|----------------------------------------|
| Plex Watchlist RSS Feeds                | Fast and instantly updated, can create one for self and one for friends | Will only show the most recent 50 movies/shows                                                                  | Yes                       | X                            | Only for self, refreshes every 6 hours |
| Plex Watchlist Metadata                 | Can fetch the full watchlist for the user                               | Need a plex token for each user who wants to have their watchlist synced, Plex token expires after a few months | Yes                       | Yes, up to ~20 min intervals | X                                      |
| Plex Watchlist GraphQL Call             | Can fetch the full watchlist for every friend of the main user          | Slow, relies on main user providing a plex token                                                                | Yes                       | X                            | X                                      |
| Plex Watchlist Metadata for other users | Can fetch the full watchlist for every user who provides a token        | Difficult to ask every user to generate a Plex token, or log into a service                                     | Yes                       | Yes, up to ~20 min intervals | X                                      |

In order to fully integrate with Plex Watchlists, Watchlistarr uses a combination of multiple methods to ensure that it
does a comprehensive, yet fast real-time sync.

### Full Delete Sync

Watchlistarr also supports a full delete sync with your watchlist. This means that **if
a user
removes an item off their watchlist, Watchlistarr can detect that and delete content from Sonarr/Radarr.**

This feature is disabled by default, refer to the Environment Variables below to see the config required to enable it.
Whether you've enabled this or not, you can enjoy a little "sneak peek"
upon startup of the app, where the logs will list the movies/tv shows that are out of sync.

### Requirements

* Plex Pass Subscription (Recommended, otherwise see "Plex Pass Alternative" section below)
* Sonarr v3 or higher
* Radarr v3 or higher
* Friends' Watchlists [Account Visibility](https://app.plex.tv/desktop/#!/settings/account) must be changed to 'Friends
  Only' or 'Friends of Friends'
* Docker or Java v11 or higher (Recommended
  version [JDK 21](https://www.oracle.com/java/technologies/downloads/#jdk21-windows))
* Plex Token (see [here](https://support.plex.tv/articles/204059436-finding-an-authentication-token-x-plex-token/))

## Getting Started

### Docker

The easiest way to try this code is using docker:

```bash
docker run \
  -e SONARR_API_KEY=YOUR_API_KEY \
  -e RADARR_API_KEY=YOUR_API_KEY \
  -e PLEX_TOKEN=YOUR_PLEX_TOKEN \
  -e REFRESH_INTERVAL_SECONDS=5 \
  nylonee/watchlistarr
```

For a full list of possible environment variables to configure the app with, see the Environment Variables section of
this Readme

Docker tag options:

* `latest` - Stable version, follows the Releases
* `beta` - Beta version, follows the main branch
* `alpha` - Experimental version, follows the latest successful PR build

### Java

#### Running the Java command

Running this using native java requires the fat jar, download the latest from the Releases tab, and run:

```bash
java "-Dsonarr.apikey=YOUR_API_KEY"\
  "-Dradarr.apikey=YOUR_API_KEY"\
  "-Dplex.token=YOUR_PLEX_TOKEN"\
  -Xmx100m\
  -jar watchlistarr.jar
```

#### Starting Watchlistarr on Windows startup

Once you confirm that this command works, you may want to set up a script to auto-run this on startup of Windows. This
can be done using a .bat file with the following contents:

```
@ECHO OFF
java -Dsonarr.apikey=YOUR_API_KEY -Dradarr.apikey=YOUR_API_KEY -Dplex.token=YOUR_PLEX_TOKEN -Xmx100m -jar watchlistarr.jar
```

Save this file in the same directory as the .jar file, then create a shortcut to this .bat file and place it in the
Windows startup folder. In the properties of the shortcut, set it to start minimized (Thanks Redditor u/DanCBooper for
tip)

#### Java variables

For a full list of options to pass in when running the application on native java,
refer to the environment variables chart below, and cross-reference the key to the internal key
in [entrypoint.sh](https://github.com/nylonee/watchlistarr/blob/main/docker/entrypoint.sh)

### Environment Variables

| Key                            | Default               | Description                                                                                                                                                                                         |
|--------------------------------|-----------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| SONARR_API_KEY*                |                       | API key for Sonarr, found in your Sonarr UI -> General settings                                                                                                                                     |
| RADARR_API_KEY*                |                       | API key for Radarr, found in your Radarr UI -> General settings                                                                                                                                     |
| PLEX_TOKEN*                    |                       | Token for Plex, retrieved via [this tutorial](https://support.plex.tv/articles/204059436-finding-an-authentication-token-x-plex-token/). Note that multiple tokens can be provided, comma separated |
| REFRESH_INTERVAL_SECONDS       | 60                    | Number of seconds to wait in between checking the watchlist                                                                                                                                         |
| SONARR_BASE_URL                | http://localhost:8989 | Base URL for Sonarr, including the 'http' and port and any configured urlbase                                                                                                                       |
| SONARR_QUALITY_PROFILE         |                       | Quality profile for Sonarr, found in your Sonarr UI -> Profiles settings. If not set, will grab the first one it finds on Sonarr                                                                    |
| SONARR_ROOT_FOLDER             |                       | Root folder for Sonarr. If not set, will grab the first one it finds on Sonarr                                                                                                                      |
| SONARR_BYPASS_IGNORED          | false                 | Boolean flag to bypass tv shows that are on the Sonarr Exclusion List                                                                                                                               |
| SONARR_SEASON_MONITORING       | all                   | Default monitoring for new seasons added to Sonarr. Full list of options are found in the [Sonarr API Docs](https://sonarr.tv/docs/api/#/Series/post_api_v3_series) under **MonitorTypes**          |
| SONARR_TAGS                    |                       | Tags to assign to tv shows that are added via Watchlistarr, comma separated                                                                                                                         |
| RADARR_BASE_URL                | http://127.0.0.1:7878 | Base URL for Radarr, including the 'http' and port and any configured urlbase                                                                                                                       |
| RADARR_QUALITY_PROFILE         |                       | Quality profile for Radarr, found in your Radarr UI -> Profiles settings. If not set, will grab the first one it finds on Radarr                                                                    |
| RADARR_ROOT_FOLDER             |                       | Root folder for Radarr. If not set, will grab the first one it finds on Radarr                                                                                                                      |
| RADARR_BYPASS_IGNORED          | false                 | Boolean flag to bypass movies that are on the Radarr Exclusion List                                                                                                                                 |
| RADARR_TAGS                    |                       | Tags to assign to movies that are added via Watchlistarr, comma separated                                                                                                                           |
| SKIP_FRIEND_SYNC               | false                 | Boolean flag to toggle between only syncing your own content, vs syncing your own and all your friends content                                                                                      |
| ALLOW_MOVIE_DELETING           | false                 | Boolean flag to enable/disable the full Watchlistarr sync for movies. If enabled, movies that are not watchlisted will be deleted from Radarr                                                       |
| ALLOW_ENDED_SHOW_DELETING      | false                 | Boolean flag to enable/disable the full Watchlistarr sync for ended shows. If enabled, shows that have no more planned seasons and are not watchlisted will be deleted from Sonarr                  |
| ALLOW_CONTINUING_SHOW_DELETING | false                 | Boolean flag to enable/disable the full Watchlistarr sync for continuing shows. If enabled, shows that still have planned seasons and are not watchlisted will be deleted from Sonarr               |
| DELETE_INTERVAL_DAYS           | 7                     | Number of days to wait before deleting content from the arrs (Deleting must be enabled)                                                                                                             |
| LOG_LEVEL                      | INFO                  | Level of logging, set to DEBUG for more verbose logs, or WARN for less logs                                                                                                                         |

## Plex Pass Alternative

The Plex Pass subscription is required to generate the RSS Feed URLs. Without a Plex Pass, the normal API calls are too
heavy-hitting on Plex's servers.

If the app detects that you are not a Plex Pass user (i.e. the app tries to generate an RSS URL, and it fails), it will
fall back into a periodic sync.

The periodic sync will run every 19 minutes, ignoring the configuration for REFRESH_INTERVAL_SECONDS

All other settings will still be valid

## Developers Corner

Build the docker image:

```
docker build -t nylonee/watchlistarr:latest -f docker/Dockerfile .
```

Run the docker image:

```
docker run nylonee/watchlistarr
```
