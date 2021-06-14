# HA-NR-WaktuSolatJakim
Home Assistant &amp; Node Red Implementation of Waktu Solat Jakim ![visitors](https://visitor-badge.glitch.me/badge?page_id=anas-ivs.ha-nr-waktusolatjakim.visitor-badge)


![Node-Red Header of HA-NR-WaktuSolatJakim](https://github.com/anas-ivs/HA-NR-WaktuSolatJakim/blob/main/images/header-HA-NR-WaktuSolatJakim.PNG)



Original sharing by Brother [farxpeace](https://github.com/farxpeace/Home-Assistant-Waktu-Solat-Jakim) which implements REST calls to [AzanPro API's](https://api.azanpro.com/) to retrieve daily local Prayers times for use in [Home Assistant(HA)](https://www.home-assistant.io/) with HA configuration.yaml for sensors and REST calls. This piece of work was starting point and allowed me learn and expand further; building more and more time-based automations including 15-min prior notificaiton to Prayer time. 

Sometime earlier in February 2021 - My [pi-hole](https://pi-hole.net/) statistics was showing AzanPro being #1 in chart for most site being requested for. While we later understood the underlying reason was self triggering of the rest call by HA API - I set myself a challenge to implement an alternative solution via Node-Red as part of my learning journey in understanding and better application of this tool. 

First attempt in February failed - The JSON/MSG Payloads and data structures and flows for Node-Red was visually too heavy to be absored and understood. 

Second attempt in June was succesfull after going through more videos and learning / expanding from other shared flows in [Node-Red Library](https://flows.nodered.org/). This HR-NR-WaktuSolatFlow flow itself foundation is based [@aitalinassim](https://flows.nodered.org/flow/9d9a3abe9707d605c6b12d21ddf08658) shared flow of Muslim Prayer Times with API calls to [AlAdhan]( https://aladhan.com/). Like most external Prayer times database - there would be some differences against JAKIM published data. There is where [AzanPro API's](https://api.azanpro.com/) come in providing the same exact data. 

This flow reimplements similar functions as per [farxpeace](https://github.com/farxpeace/Home-Assistant-Waktu-Solat-Jakim) implementation in HA but with Node-Red, this has become much more easier to GET, Store, Match prayer times and define each functions.
> **WORK TO BE DONE** Switch daily retreival to monthly,yearly to be able for offline/help reduce load to AzanPro.

![HA Lovelace](https://github.com/anas-ivs/HA-NR-WaktuSolatJakim/blob/main/images/Lovelace-entities-WaktuSolat.PNG)

![Telegram sample](telegram statistics)

In addition to sending back for HA for lovelace; also included in node red function for Telegram request and reporting

AND a Node-Red dashboard too! Credit to originator [@aitalinassim](https://flows.nodered.org/flow/9d9a3abe9707d605c6b12d21ddf08658) for building it in and for me to find out what it was.

## How It Works (Extension to Node-Red)
- PART 1 - Retrieve Prayer Times 
![Part1 Flow](https://github.com/anas-ivs/HA-NR-WaktuSolatJakim/blob/main/images/Flow_part1.PNG)
1.  During initialize/once-daily/upon request - Node-Red via `http` nodes performs `GET` request to [AzanPro Get today prayer](https://api.azanpro.com/reference/times/today). 
> **REQUIRED** Determine your location `ZONE` from this [listing](https://api.azanpro.com/zones)
3.  Split the payload entities accordingly and save in HA entities defined as Sensors. Had the choice of sending all in one sensor entity with 5xprayer times as attributes `(sensor.waktu_solat` but exploring lovelace has limited features to call this out for display without use of custom mods. Other entities i.e. `sensor.subuh`, `sensor.syuruk` .. had to be created as well for Lovelace to be able to display easily. 

- PART 2 - Every minute - Check current time against Prayer Times
![Part2 Flow](https://github.com/anas-ivs/HA-NR-WaktuSolatJakim/blob/main/images/Flow_part2.PNG)
1.  Node-Red via `inject` node every minute triggers the flow to retrieve the store Prayer Times in HA `sensor.waktu_solat` entity. 
2.  In `function` node - current Date is requested and split down get current hour and minutes.
> **WORK TO BE DONE** Script does not check if dates retreived and current match.
3.  Node then continues to verify against each prayer time and set payload of `.waktu` to the match prayer time.
4.  While a direct flow can be used to trigger our wanted action i.e. play Adhan on `media_player` - using `switch` node this flag can then be used to split the flow to the prayer time accordingly i.e. `Subuh` trigger me the `turn_on.lights` scene and `Maghrib` triggers my other `/gethadith` flow for random Hadith of the day to be read later with family.

- PART 3 - OPTIONAL - Trigger 15 minutes notice prior to Adhan time.
![Part2 Flow](https://github.com/anas-ivs/HA-NR-WaktuSolatJakim/blob/main/images/Flow_part3.PNG)
> **Learning Journey** Rather than adjusting each prayer time forward by 15 minutes, it was much more easier to 'advance' the 'clock' by 15 minutes as reference.
1.  This flow is similar to above; except it checks and set the flag for pretime accordingly.
4.  Similarly using `switch` node this flag can then be used to split the flow to the 15 minute to prayer time accordingly i.e. `15 Minutes to Prayer` triggers Chromecast to play randomized playlist of either Live Makkah or Madinah Youtube feed, `15 Minutes to Maghrib` trigger scene to `turn_on.lights` and `cover.close` (tutup langsir). 

-  Telegram node `/getwaktusolat`  for manual call request and sets flags to identify at output to only send Telegram update when requested. 

- PRAYER TIMES ACTION - CUSTOMIZE TO YOUR PREFERENCE
![ACTION!](https://github.com/anas-ivs/HA-NR-WaktuSolatJakim/blob/main/images/Flow_Actions.PNG)
1. Trigger Telegram Notification:

2. Trigger Chromecast to play TTS message.

3. Trigger to play Youtube video on `media_player` i.e. TV based on randomize list. 


-  Telegram node `/getwaktusolat`  for manual call request and sets flags to identify at output to only send Telegram update when requested. 


## Pre-requisites 
1.  Home Assistant with Node-Red. Ada banyak tutorial/videos on this with difficulty level as easy. [This is one example](http://https://www.juanmtech.com/get-started-with-node-red-and-home-assistant/). Test that you have enabled and can load Node-red on side bar. Make sure to also install [Node-Red companion integration](https://github.com/zachowj/hass-node-red).
2.  Telegram bot and chat ids. I followed this [tutorial](https://www.thesmarthomebook.com/2020/10/13/a-guide-to-using-telegram-with-node-red-and-home-assistant/) which is clear and easy to follow. 
> **Tip:** Follow the steps to get botid/chatid only but you do not need to setup in Home Assistant Notify/Telegram platform. Use Node-Red fully for Telegram.
4.  In Node-red the following additional nodes may be required:
- `node-red-contrib-home-assistant-websocket` - Comes pre-installed if using default HA Node-Red Docker from Supervisor store. 
- `node-red-contrib-telegrambot` - For Telegrambot. Setup as guide above.
- `node-red-contrib-random-item ` - For random number generator to select random playlist.
5.  For Home Assistant, the following will be required:
 - [Node-Red companion integration](https://github.com/zachowj/hass-node-red); install via [HACS](https://hacs.xyz/). This allows entities to be setup from node-red instead of manual sensor entities in configuration.yaml or helpers. 
> **Tip:** Define name of entity node first before clicking 'deploy' to be able to register in HACS the exact sensor name you want instead of random numbers.
6.  For Home Assistant Lovelace to mimic my view, additional lovelace needed - mostly installed via HACS lovelace:
- [Vertical stack in card](https://github.com/ofekashery/vertical-stack-in-card) to combine cards together without border.
- [Mini graph card](https://github.com/kalkih/mini-graph-card)
    

## Installation
1. Node-Red.
- Import [Flow](https://github.com/anasothman-myy/HA-NR-MYCovidStats/blob/main/ha-nr-mycovidstats.json) into Node Red (Upper Right burger stack -> Import).
- Ensure Telegrambots and Home Assistants node Servers are configured before clicking deploy.
- Click on `inject` node to test you have data.
> **Tip:** Use `link in`/`link out` to simplify and link repetitive task i.e. send to Telegram. 
- Additional: The missing `link-in` Telegram node can be imported [here](https://github.com/anasothman-myy/HA-NR-MYCovidStats/blob/main/telegram_output.json).
2. Home Assistant
- Under`Configuration -> Integrations -> Node-Red`  following created entities should now be available:
-`switch.waktu_solat` - To enable the flow; set to true. Future expansion to trigger if Away from Home.
-`sensor.zone_waktu_solat`
-`sensor.waktu_solat` - With attributes for each prayer time and data from AzanPro
-`sensor.subuh`, `sensor.syuruk`, `sensor.zohor`, `sensor.asar`, `sensor.maghrib`, `sensor.isyak`

3. Lovelace - Import and customize to your liking.

```text
type: custom:vertical-stack-in-card
style: |
  ha-card {
    background-color: var(--primary-background-color);
    border-radius: 15px;
    margin: 10px;
    font-size: 6 px
    box-shadow:
      {% if is_state('sun.sun', 'above_horizon') %}
        -4px -4px 8px rgba(255, 255, 255, .5), 5px 5px 8px rgba(0, 0, 0, .03);
      {% elif is_state('sun.sun', 'below_horizon') %}
        -5px -5px 8px rgba(50, 50, 50, .2), 5px 5px 8px rgba(0, 0, 0, .08);
      {% endif %}
   }
    .card-header {
    font-size: 6 px
  }
cards:
  - type: entity
    entity: sensor.waktu_solat
    name: 'Waktu Solat '
  - type: glance
    style: |
      ha-card {
        background-color: var(--primary-background-color);
        border-radius: 15px;
        margin: 10px;
        font-size: 6 px
        box-shadow:
          {% if is_state('sun.sun', 'above_horizon') %}
            -4px -4px 8px rgba(255, 255, 255, .5), 5px 5px 8px rgba(0, 0, 0, .03);
          {% elif is_state('sun.sun', 'below_horizon') %}
            -5px -5px 8px rgba(50, 50, 50, .2), 5px 5px 8px rgba(0, 0, 0, .08);
          {% endif %}
       }
        .card-header {
        font-size: 6 px
      }
    entities:
      - entity: sensor.subuh
      - entity: sensor.syuruk
      - entity: sensor.zohor
      - entity: sensor.asar
      - entity: sensor.maghrib
      - entity: sensor.isyak
    state_color: true
    columns: 6


```

## Thanks
##### Credit to @aitalinassim / @farxpeace

##### [Home Assistant Malaysia](https://www.facebook.com/groups/homeassistantmalaysia)
