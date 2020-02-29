# homebridge-http-opensprinkler-api Plugin

[![npm](https://img.shields.io/npm/v/homebridge-http-opensprinkler-api?style=for-the-badge)](https://www.npmjs.com/package/homebridge-http-opensprinkler-api)
[![npm](https://img.shields.io/npm/dt/homebridge-http-opensprinkler-api?style=for-the-badge)](https://www.npmjs.com/package/homebridge-http-opensprinkler-api)
[![GitHub Workflow Status](https://img.shields.io/github/workflow/status/1MC/homebridge-http-opensprinkler-api/Node-CI?style=for-the-badge)](https://github.com/1MC/homebridge-http-opensprinkler-api/actions?query=workflow%3A%22Node-CI%22)
[![GitHub issues](https://img.shields.io/github/issues/1MC/homebridge-http-opensprinkler-api?style=for-the-badge)](https://github.com/Supereg/homebridge-http-switch/issues)
[![GitHub pull requests](https://img.shields.io/github/issues-pr/1MC/homebridge-http-opensprinkler-api?style=for-the-badge)](https://github.com/1MC/homebridge-http-opensprinkler-api/pulls)


`homebridge-http-opensprinkler-api` is a [Homebridge](https://github.com/nfarina/homebridge) plugin forked from `homebridge-http-switch` specifically to control an [OpenSprinkler](https://opensprinkler.com/) system using the [firmware API from v2.0]('https://openthings.freshdesk.com/support/solutions/articles/5000716363') on. The intention is to make it simpler for others wanting basic control of their OpenSprinkler by having this as a reference configuration that worked for me. Example configuration for switching of sprinkler stations ON/OFF and syncing of status is shown below.

To get more details about configuration look at the original 
[homebridge-http-switch](https://github.com/Supereg/homebridge-http-switch) by Supereg.

## Installation

First of all you need to have [Homebridge](https://github.com/nfarina/homebridge) installed. Refer to the repo for 
instructions.  
Then run the following command to install `homebridge-http-opensprinkler-api`

```
sudo npm install -g homebridge-http-opensprinkler-api
```

## Configuration:

The configuration settings below are tailored for the basic operation of OpenSprinkler.  All API commands require the **OpenSprinkler password** to be supplied in the **'pw'** parameter.  **The password should
be MD5 hashed too (all lower-case)**. You can find online MD5 hash tools to convert plaintext password to MD5.

Note that I've over simplified the configuration settings below, there is much more advanced configuration you can make use of by referring to the original `homebridge-http-switch`.

## Configuration used for OpenSprinkler:

- `name` \<string\> **required**: Defines the name which is later displayed in HomeKit

* `onUrl` \<string\> **required**: Should be set to the Manual Station Run API call with the enable bit set to **'1'**.  Set **sid** to the station index (starting at 0) and **t** to the run duration. /cm?pw=**xxx**&sid=**xxx**&en=1&t=**xxx**
* `offUrl` \<string\> **required**: Should be set to the Manual Station Run API call with the enable bit set to **'0'**.  Set **sid** to the station index (starting at 0) to be stopped. /cm?pw=**xxx**&sid=**xxx**&en=0
* `statusUrl` \<string\> **required**: Should be set to the url for the OpenSprinkler Get Status Status API call.  /js?pw=xxx
* `statusPattern` \<string\> **required**: Defines a regex pattern which is compared to the body of the `statusUrl`.  Example configuration based on 8 station setup.  Each accessory requires the regex pattern to be adjusted to match the station number. Refer to the [OpenSprinkler API documentation]('https://openthings.freshdesk.com/support/solutions/articles/5000716363'). 
More on how to build regex patterns: https://www.w3schools.com/jsref/jsref_obj_regexp.asp
* `pullInterval` \<integer\> **optional**: The property expects an interval in **milliseconds** in which the plugin 
pulls updates from your OpenSprinkler firmware.


```json
{
    "accessories": [
        {
          "accessory": "OPENSPRINKLER-STATION",
          "name": "Sprinkler Stn01",
          
          "switchType": "stateful",
          
          "onUrl": "http://localhost:8080/cm?pw=xxx&sid=0&en=1&t=600",
          "offUrl": "http://localhost:8080/cm?pw=xxx&sid=0&en=0",
          
          "statusUrl": "http://localhost:8080/js?pw=xxx",
          "statusPattern": "{\"sn\":[1,[0-9]+,[0-9]+,[0-9]+,[0-9]+,[0-9]+,[0-9]+,[0-9]+],\"nstations\":8}",
          "pullInterval": "1000"
        },
        {
          "accessory": "OPENSPRINKLER-STATION",
          "name": "Sprinkler Stn02",
          
          "switchType": "stateful",
          
          "onUrl": "http://localhost:8080/cm?&pw=xxx&sid=1&en=1&t=600",
          "offUrl": "http://localhost:8080/cm?&pw=xxx&sid=1&en=0",
          
          "statusUrl": "http://localhost:8080/js?pw=xxx",
          "statusPattern": "{\"sn\":[[0-9]+,1,[0-9]+,[0-9]+,[0-9]+,[0-9]+,[0-9]+,[0-9]+],\"nstations\":8}",
          "pullInterval": "1000"
        }
    ]
}
```

