# themeparks

An unofficial API library for accessing ride wait times and park opening times for many theme parks around the world, including Disney, Universal and SeaWorld parks.

[![Build Status](https://travis-ci.org/cubehouse/themeparks.svg?branch=master)](https://travis-ci.org/cubehouse/themeparks) [![npm version](https://badge.fury.io/js/themeparks.svg)](https://badge.fury.io/js/themeparks) [![Dependency Status](https://beta.gemnasium.com/badges/github.com/cubehouse/themeparks.svg)](https://beta.gemnasium.com/projects/github.com/cubehouse/themeparks)

[![GitHub stars](https://img.shields.io/github/stars/cubehouse/themeparks.svg)](https://github.com/cubehouse/themeparks/stargazers) ![Downloads](https://img.shields.io/npm/dt/themeparks.svg) ![Monthly Downloads](https://img.shields.io/npm/dm/themeparks.svg)

[Roadmap](https://github.com/cubehouse/themeparks/projects/1) | [Documentation](https://cubehouse.github.io/themeparks/) | [Change Log](CHANGELOG.md) | [Supported Parks](#supported-park-features)

## Install

    npm install themeparks --save

## Migrate from themeparks 4.x

If you have been using themeparks 4.x, please follow this guide to [migrate from themeparks 4.x to themeparks 5.x](https://github.com/cubehouse/themeparks/wiki/Migrating-from-4.x-to-5.x)

## Example Use

    // include the Themeparks library
    const Themeparks = require("themeparks");

    // configure where SQLite DB sits
    // optional - will be created in node working directory if not configured
    // Themeparks.Settings.Cache = __dirname + "/themeparks.db";

    // access a specific park
    //  Create this *ONCE* and re-use this object for the lifetime of your application
    //  re-creating this every time you require access is very slow, and will fetch data repeatedly for no purpose
    const DisneyWorldMagicKingdom = new Themeparks.Parks.WaltDisneyWorldMagicKingdom();

    // Access wait times by Promise
    DisneyWorldMagicKingdom.GetWaitTimes().then((rideTimes) => {
        for(var i=0, ride; ride=rideTimes[i++];) {
            console.log(`${ride.name}: ${ride.waitTime} minutes wait (${ride.status})`);
        }
    }).catch((error) => {
        console.error(error);
    });

    // Get park opening times
    DisneyWorldMagicKingdom.GetOpeningTimes().then((times) => {
        for(var i=0, time; time=times[i++];) {
            if (time.type == "Operating") {
                console.log(`[${time.date}] Open from ${time.openingTime} until ${time.closingTime}`);
            }
        }
    }).catch((error) => {
        console.error(error);
    });

### Using Promises or callbacks

Both GetWaitTimes and GetOpeningTimes work either through callback or Promises.

This is the same as the above example, but using a callback instead of a Promise.

    // access wait times via callback
    disneyMagicKingdom.GetWaitTimes((err, rides) => {
        if (err) return console.error(err);

        // print each wait time
        for(var i=0, ride; ride=rides[i++];) {
            console.log(`${ride.name}: ${ride.waitTime} minutes wait (${ride.status})`);
        }
    });

### Proxy

If you wish to use themeparks with a proxy, you can pass a proxy agent when you construct the park object.

    // include the Themeparks library
    const Themeparks = require("themeparks");

    // include whichever proxy library you want to use (must provide an http.Agent object)
    const SocksProxyAgent = require('socks-proxy-agent');

    // create your proxy agent object
    const MyProxy = new SocksProxyAgent("socks://socks-proxy-host", true);

    // create your park object, passing in proxyAgent as an option
    const DisneyWorldMagicKingdom = new Themeparks.Parks.WaltDisneyWorldMagicKingdom({
        proxyAgent: MyProxy
    });

## Change Log

[View themeparks Change Log](CHANGELOG.md)

## Parks available

<!-- START_SUPPORTED_PARKS_LIST -->

* Magic Kingdom - Walt Disney World Florida (ThemeParks.Parks.WaltDisneyWorldMagicKingdom)
* Epcot - Walt Disney World Florida (ThemeParks.Parks.WaltDisneyWorldEpcot)
* Hollywood Studios - Walt Disney World Florida (ThemeParks.Parks.WaltDisneyWorldHollywoodStudios)
* Animal Kingdom - Walt Disney World Florida (ThemeParks.Parks.WaltDisneyWorldAnimalKingdom)

<!-- END_SUPPORTED_PARKS_LIST -->

## Supported Park Features

<!-- START_PARK_FEATURES_SUPPORTED -->
|Park|Wait Times|Park Opening Times|Ride Opening Times|
|:---|:---------|:-----------------|:-----------------|
|Magic Kingdom - Walt Disney World Florida|&#10003;|&#10003;|&#10007;|
|Epcot - Walt Disney World Florida|&#10003;|&#10003;|&#10007;|
|Hollywood Studios - Walt Disney World Florida|&#10003;|&#10003;|&#10007;|
|Animal Kingdom - Walt Disney World Florida|&#10003;|&#10003;|&#10007;|

<!-- END_PARK_FEATURES_SUPPORTED -->

## Result Objects

### Ride Wait Times

    [
        {
            id: (string or number: uniquely identifying a ride),
            name: (string: ride name),
            waitTime: (number: current wait time in minutes),
            active: (bool: is the ride currently active?),
            fastPass: (bool: is fastpass available for this ride?),
            fastPassReturnTime: { (object containing current return times, parks supporting this will set FastPassReturnTimes to true - entire field may be null for unsupported rides or when fastPass has ran out for the day)
                startTime: (string return time formatted as "HH:mm": start of the current return time period),
                endTime: (string return time formatted as "HH:mm": end of the current return time period),
                lastUpdate: (JavaScript Date object: last time the fastPass return time changed),
            },
            status: (string: will either be "Operating", "Closed", or "Down"),
            lastUpdate: (JavaScript Date object: last time this ride had new data),
            schedule: { **schedule will only be present if park.SupportsRideSchedules is true**
              openingTime: (timeFormat timestamp: opening time of ride),
              closingTime: (timeFormat timestamp: closing time of ride),
              type: (string: "Operating" or "Closed"),
              special: [ (array of "special" ride times, usually Disney Extra Magic Hours or similar at other parks - field may be null)
                openingTime: (timeFormat timestamp: opening time for ride),
                closingTime: (timeFormat timestamp: closing time for ride),
                type: (string: type of schedule eg. "Extra Magic Hours", but can be "Event" or "Special Ticketed Event" or other)
              ]
            },
        },
        ...
    ]

### Schedules

    [
        {
            date: (dateFormat timestamp: day this schedule applies),
            openingTime: (timeFormat timestamp: opening time for requested park - can be null if park is closed),
            closingTime: (timeFormat timestamp: closing time for requested park - can be null if park is closed),
            type: (string: "Operating" or "Closed"),
            special: [ (array of "special" times for this day, usually Disney Extra Magic Hours or similar at other parks - field may be null)
              openingTime: (timeFormat timestamp: opening time for requested park),
              closingTime: (timeFormat timestamp: closing time for requested park),
              type: (string: type of schedule eg. "Extra Magic Hours", but can be "Event" or "Special Ticketed Event" or other)
            ],
        },
        ...
    ]

## Park Object values

There are some values available on each park object that may be useful.

|Variable|Description|
|:-------|:----------|
|Name|Name of the park|
|Timezone|The park's local timezone|
|LocationString|This park's location as a geolocation string|
|SupportsWaitTimes|Does this park's API support ride wait times?|
|SupportsOpeningTimes|Does this park's API support opening hours?|
|SupportsRideSchedules|Does this park return schedules for rides?|
|FastPass|Does this park have FastPass (or a FastPass-style service)?|
|FastPassReturnTimes|Does this park tell you the FastPass return times?|
|Now|Current date/time at this park (returned as a Moment object)|
|UserAgent|The HTTP UserAgent this park is using to make API requests (usually randomly generated per-park at runtime)|

    const ThemeParks = require("themeparks");

    // construct our park objects and keep them in memory for fast access later
    const Parks = {};
    for (const park in ThemeParks.Parks) {
      Parks[park] = new ThemeParks.Parks[park]();
    }

    // print each park's name, current location, and timezone
    for (const park in Parks) {
      console.log(`* ${Parks[park].Name} [${Parks[park].LocationString}]: (${Parks[park].Timezone})`);
    }

Prints:

<!-- START_PARK_TIMEZONE_LIST -->

* Magic Kingdom - Walt Disney World Florida [(28°23′6.72″N, 81°33′50.04″W)]: (America/New_York)
* Epcot - Walt Disney World Florida [(28°22′28.92″N, 81°32′57.84″W)]: (America/New_York)
* Hollywood Studios - Walt Disney World Florida [(28°21′27.00″N, 81°33′29.52″W)]: (America/New_York)
* Animal Kingdom - Walt Disney World Florida [(28°21′19.08″N, 81°35′24.36″W)]: (America/New_York)

<!-- END_PARK_TIMEZONE_LIST -->

## Development

### Running Tests

themeparks supports mocha unit tests. Install mocha with npm install -g mocha

Run the following to test the library's unit tests (this will build the library and then run functional offline unit tests):

    npm test

You can also run unit tests against the source js files using ```npm run testdev```.

There is a separate test for checking the library still connects to park APIs correctly. This is the "online test".

    npm run testonline

You can also test an individual park using the PARKID environment variable, for example:

    PARKID=UniversalStudiosFlorida npm run testonline

### Debug Mode

Themeparks supports the standard NODE_DEBUG environment variable. Pass the name of the library into NODE_DEBUG to turn on debug mode:

    NODE_DEBUG=themeparks npm run testonline

Environment variables can be combined:

    NODE_DEBUG=themeparks PARKID=UniversalStudiosFlorida npm run testonline

### Contributing

Each park inherits it's core logic from lib/park.js.

For each set of parks, a base object should be made with all the core logic for that API/park group.

Then, for each park, a basic shell object should be implemented that just configures the park's base object (and overrides anything in unusual setups).

Throughout the API, please make use of the this.Log() function so debugging parks when things break is easier.

Please raise issues and make pull requests with new features :)

See full contribution guide at [Themeparks Contribution Guide](https://github.com/cubehouse/themeparks/wiki/Contributing).

A rough guide for adding new parks is also available at [Adding New Parks to the ThemeParks API](https://github.com/cubehouse/themeparks/wiki/Adding-New-Parks).

## People using themeparks

If you're using themeparks for a project, please let me know! I'd love to see what people are doing!

### Websites and Mobile Apps

* [My Disney Visit](http://www.mydisneyvisit.com/) - Walt Disney World
* [ChronoPass](https://play.google.com/store/apps/details?id=fr.dechriste.android.attractions&hl=en_GB) - Walt Disney World, Disneyland Paris, Parc Asterix, EuropaPark

### Pebble Apps

* [Disneyland California Wait Times](https://apps.getpebble.com/en_US/application/5656424b4431a2ce6c00008d)
* [Disneyland Paris Wait Times](https://apps.getpebble.com/en_US/application/55e25e8d3ea1fb6fa30000bd)
* [Disney World Wait Times](https://apps.getpebble.com/en_US/application/54bdb77b54845b1bf40000bb)
