# Stop Watcher
Stop Watcher is an all-in-one package to stay updated on your local Waypoints. it can:
- Update names/photos of new Gyms and Stops using Portal (or Stop) information
- Delete Stops that converted to Gyms
- Send notifications about new Portals, Stops and Gyms to Discord and Telegram
- Track location, title and photo edits, as well as removals of Portals, Stops and Gyms and send noticiations for them

### Join the [Pogo Tools Discord Server](https://discord.gg/CWFyy6s) for help and updates

Note that a working Ingress Scraper is needed for everything related to Portals and a working MAD/RDM instance for everything related to Stops and Gyms. Stop Watcher will work fine if you only have on of those set up.

## Setup
1. `git clone https://github.com/ccev/stopwatcher.git && cd stopwatcher` and `pip[3[.6]] install -r requirements.txt`
2. `cp -r config_example config`, fill out config/config.ini, config/filters.json and config/geofence.json ([Explainations](https://github.com/ccev/stopwatcher#config-files)). Config and filters are explained below.
3. That's it. Now run `python3 stop_watcher.py`. I recommend using cron or pm2 to loop it

## Additional Steps
### Static Maps
Mapbox and tileserver static maps are the coolest ones. If you don't already have one of the four providers set up, I'd recommend making a free [Mapbox](https://www.mapbox.com/) account for 50,000 free requests.

#### [Flo's Tileserver](https://github.com/123FLO321/SwiftTileserverCache)
To get working tileserver Static Maps, make sure you're on the latest version (>1.0.3 / March 4 2020), then copy stopwatcher.json to the `Templates` folder in your TileServer directory

### Ingress Scraper
Use [ClarkyKent's ingress scraper](https://github.com/ClarkyKent/ingress_scraper) for everything related to Portals. Have it loop and put the restart delay in Stop Watcher's config

## Features
[TODO (it's super cool i promise)]

## Config files
### config.ini
This file provides the settings, api keys, and generic configuration needed to function. Since this file is parsed by python make sure to **not** include quotation marks.

 Heading | Option | Description | What to put in
|-|-|-|-|
| Config | **`language`** | The language you want your notifications to be in. | `en`/`de`/`fr`/`es`/`pl`
| Config | **`portal_scraper_interval`** | The restart delay of your Ingress Scraper. Used to help determine when portals have been removed. | A number in seconds
| Maps | **`static_map`** | Wheter to include a static map in the notification or not. | `True`/`False`
| Maps | **`static_map_provider`** | Your static map provider. | `tileserver`/`mapbox`/`osm`/`google`
| Maps | **`key`** | The additional information needed to generate Static Maps. Either your API key or Tileserver URL. | String, mapbox/mapquest/google api key or tileserver url
| Maps | **`frontend_map`** | Whether to include a link to your frontend map or not. | `True`/`False`
| Maps | **`map_url`** | URL to your frontend map. | `https://www.map.com/`
| Maps | **`frontend`** | Frontend that you're using. | `pmsf`/`rdm`/`rmad`
| Maps | **`map_name`** | This is the text that links to your frontend map. | Your Map Name
| Maps | **`geocoding`** | Wheter to include the address in the notification or not. | `True`/`False`
| Maps | **`geocoding_provider`** | Your geocoding provider. | `google`/`osm`/`mapbox`
| Maps | **`geocoding_key`** | API key for geocoding, required for google & mapbox. | `google`/`osm`/`mapbox`
| DB | **`scanner`** | Select your scanner type. | `mad`/`rdm`
| DB | **`scanner_db_name`** | Name of your scanner database. | Database name
| DB | **`portal_db_name`** | Name of your portal database (must follow [pmsf](https://github.com/pmsf/PMSF/blob/master/sql/manualdb/manualdb.sql) data structure). | Database name
| DB | **`host`** | Hostname or ip address of your database server. | Address
| DB | **`port`** | Port of database server. | Port
| DB | **`user`** | Username that has access to scanner/pmsf database tables. | Username
| DB | **`password`** | User password that has access to scanner/pmsf database tables. | Password

### filters.json
Filters tell Stop Watcher what to look for and where to send it. Every filter option is optional, except `"area"`. So if you don't want something, just delete the option to avoid confusion (e.g. remove `"webhook"` completely instead of putting `"webhook": []` if you don't want Discord notifications)

| Option | Description | What to put in
|-|-|-|
| **`area`** | Needs to match an area name from your geofence.json, else it will default to your whole DB. | String, `"berlin"`
| **`update`** | What kind of Waypoints' photos/titles you want to get updates, if they're empty. Works with Stops and Gyms. Stops take their info from Portal, Gyms from Stops or Portals. | List with "stop" and/or "gym" in, `["stop", "gym"]` |
| **`delete_converted_stops`** | Whether to delete Stops that turned into Gyms or not. You can just delete the option instead of putting in `false` | `true`/`false` |
| **`send`** | The kinds of new Waypoints you want to get sent. | List with "portal", "stop" and/or "gym", `["portal", "stop", "gym"]` |
| **`send_empty`** | Whether or not to send Stops/Gyms with no photo/title. If true, the Waypoint will be sent again if its info is known. If you have an Ingress Scraper running and updates set to true, you can leave this option on, in case the systems fail. You can just delete the option instead of putting in `false` | `true`/`false`
| **`edits`** | The kinds of Waypoints you want edits to be sent of. | List with "portal", "stop" and/or "gym", `["portal", "stop", "gym"]` |
| **`edit_types`** | The kinds of edits you want to get sent. (Should have at least one if `"edits"` has something in) | List with "location", "title", "photo" and/or "removal", `["location", "title", "photo", "removal"]` |
| **`deleted`** | Stop Watcher looks for removals by searching for Waypoints that haven't been updated in a certain time interval. If it finds more than {max}, it will send a notification for it. That time interval is (4x(Scraper Interval))/60 for Portals and 5 hours for Stops/Gyms. {max} is 5 for each. Since those values can very per area, you can use this option to set your own ones | JSON that looks like this: (Every option is optional!) `{"max": {"scraper": 10, "scanner": 3}, "timespan": {"scraper": 3600, "scanner": 1440}}` |
| **`webhook`** | The Discord Webhook edited/new waypoints should get sent to. Can also be multiple | Webhook links, `["url.com"]` or `["url.com","url2.com"]` |
| **`bot_id`** | You Telegram's Bot ID | String, `"8762682"` |
| **`chat_id`** | You Telegram's Chat ID. Can also be multiple. (Needs a set `"bot_id"` to work) | Chat IDs, `["24254535"]` or `["4636363","970785"]` |

### geofence.json
The areas you want to use in your filters.json. It's the same format Poracle (or Discordopole) uses, so you can copy it from there. You can use [poracle.world](https://maps.poracle.world/) or [geo.jasparke.net](http://geo.jasparke.net/) to help create this file.

## Extras
Command line arguments that can be passed to the `stop_watcher.py` script.

### --init / -i
Fills your cache files with up-to-date data. Useful if you want to add a new area to Stop Watcher or haven't ran it in a while.

### --delete / -d
Gives a report of all (possibly) removed Portals/Stops/Gyms together with their names, IDs, coordinates and a SQL Query to delete them from your database. The way Stop Watcher checks for removed Waypoints is not 100% accurate, so please check before you delete!
