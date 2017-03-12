
# Toronto Apartment Finder
<p align="center">
  <img src=images/robot.png alt="robot" style="width: 150px;" style="height: 150px;"/>
</p>

This repo contains the code for a bot that will scrape Craigslist & Kijiji for real-time listings matching specific criteria, then alert you in Slack. This will let you quickly see the best new listings, and contact the owners. You can adjust the settings to change your price range, what neighborhoods you want to look in, and what transit stations and other points of interest you'd like to be close to.

The tool is ideal for people who are looking to find a rental with more than 1 person, since the same listings can be conveniently viewed by all people in Slack, and favourited listings can be tracked.

The project was inspired by the work of [Vik Paruchuri](https://github.com/VikParuchuri/apartment-finder). Please visit his repo for setup instructions and the original work.


## Overview
<p align="center">
  <img src=images/af_overview.png alt="overview" style="width: 600px;" style="height: 200px;"/>
</p>

The apartment-finder works by scraping listings from Kijiji and Craigslist. A package created by [juliomalegria](https://github.com/juliomalegria/python-craigslist) was used for getting the Craigslist listings. The package was modified to scrape the first 3 pages of listings, rather than just the first page. I also changed it to use selenium + a Chrome webdriver in order to extract the image urls for previewing within Slack. The Kijiji scraping was accomplished using requests & BeautifulSoup (see src/Kijiji.py).

Latitude/longitude coordinates can be scraped from Craigslist. For Kijiji, I use the scraped address and Google Maps API to get the coordinates, which can then be used for location based filtering.

Each listing is then passed through various filters, including a check to see if it's already been posted, before listing the result in the relevant Slack channel.

## Features

### Hood
<p align="center">
  <img src=images/hoods.png alt="hoods" style="width: 600px;" style="height: 200px;"/>
</p>

```python
BOXES = [
    ("distillery", [
        [43.650516, -79.35236],
        [43.655841,	-79.370513],
    ]),
    ("st-lawrence", [
        [43.644507, -79.370513],
        [43.655841, -79.376349],
    ]),
    ("financial-district", [
        [43.644662, -79.376521],
        [43.649879, -79.387422],
    ]),
    ("yonge-corridor", [
        [43.649879, -79.383602],
        [43.670557, -79.387422]
    ])
]

```
### Distance to Subway Station
<p align="center">
  <img src=images/metro_dist.png alt="metro_dist" style="width: 600px;" style="height: 200px;"/>
</p>

```python
## Transit preferences
# The farthest you want to live from a transit stop.
MAX_TRANSIT_DIST = 4 # kilometers

# Transit stations you want to check against.  Every coordinate here will be checked against each listing,
# and the closest station name will be added to the result and posted into Slack.
TRANSIT_STATIONS = {
    "st_andrew": [43.647400, -79.384358],
    "osgoode": [43.650754, -79.386718],
    "union": [43.645599, -79.380367],
    "king": [43.648674, -79.377835],
    "queen": [43.652307, -79.379208],
    "college": [43.6613247, -79.3852633],
    "dundas": [43.6552859, -79.3797379],
    "st_patrick": [43.6548307,-79.3905372],
    "queens_park": [43.6598804,-79.3926655],
    "museum": [43.6671223,-79.3956618],
    "spadina": [43.6673568,-79.4059985],
    "st_george": [43.6682622,-79.402047],
    "wellesley": [43.6654593,-79.3860771],
    "bloor_yonge": [43.6709058,-79.3878259],
    "bay": [43.6701472,-79.3928834],
    'dupont': [43.6748551,-79.4092697],
    'bathurst': [43.6666064,-79.4110757],
    'christie': [43.6641199,-79.4205082],
    'ossington': [43.662437,-79.4283647],
    'dufferin': [43.6601077,-79.437617]
}

```
### Commute Time to Work

```python
#time of the day you leave for work
HOUR_DEPART = 8
MINUTE_DEPART = 0

## how do you get to work?
## accepts: driving, walking, bicycling, transit
TRAVEL_MODE = 'transit'
WORK_ADDRESS = "5140+Yonge+St+North+York+ON+M2N+6X7"

## longest work commute time your willing to endure!
MAX_COMMUTE_TIME = 90
```

### Enhanced Slack Posts

```python
# True if you would like posts with the image preview, and other parameters
# False if you would prefer simple posts with default description & url
ENHANCED_POSTS = True

# enter the parameters you would like to have colour coded
# there are two types of colour coding methodologies
# 1) "range" will check if the parameter value falls within specified range
# 2) "list" will check if the parameter value falls within a specified list
# 3 standard colours - good is green, warning is yellow, danger is red.
# feel free to swap those out with any hex colour code
COLOURS = {
    'price': {
        'levels': {
            'good': [0, 1500],
            'warning': [1501, 1700],
            'danger': [1701, 10000]
        },
        'type': "range"
    },
    'area': {
        'levels': {
            'good': ['st-lawrence', 'queen-west', 'liberty-village',
                'ossington', 'Queen', 'King', 'Liberty'],
            'warning': ['distillery','financial-district', 'mid-west',
                'Spadina', 'College', 'Downtown'],
            'danger': ['yonge-corridor','bloor-west', 'Yonge',
                'Bloor', 'Toronto']
        },
        'type': "list"
    }
}

```
### Keywords
Use regexes to determine if the unit is furnished or a studio/bachelor apartment rather than relying on Kijiji/Craigslist filters.

For future work, the full listing description can be pulled in and parsed for relevant information.

## TODO

- [x] Craigslist Integration
- [x] Kijiji Integration
- [x] Enhanced Slack Posts
- [x] Work Commute time
- [ ] Parse additional listing info from the listing description
- [ ] Scrape Viewit or other rental listing sites
- [ ] Build a recommender based on liked listings
- [ ] Refactoring
- [ ] Add docstrings
