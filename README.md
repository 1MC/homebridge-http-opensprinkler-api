# homebridge-http-opensprinkler-api Plugin

[![npm](https://img.shields.io/npm/v/homebridge-http-opensprinkler-api?style=for-the-badge)](https://www.npmjs.com/package/homebridge-http-opensprinkler-api)
[![npm](https://img.shields.io/npm/dt/homebridge-http-opensprinkler-api?style=for-the-badge)](https://www.npmjs.com/package/homebridge-http-opensprinkler-api)
[![GitHub Workflow Status](https://img.shields.io/github/workflow/status/1MC/homebridge-http-opensprinkler-api/Node-CI?style=for-the-badge)](https://github.com/1MC/homebridge-http-opensprinkler-api/actions?query=workflow%3A%22Node-CI%22)
[![GitHub issues](https://img.shields.io/github/issues/1MC/homebridge-http-opensprinkler-api?style=for-the-badge)](https://github.com/Supereg/homebridge-http-switch/issues)
[![GitHub pull requests](https://img.shields.io/github/issues-pr/1MC/homebridge-http-opensprinkler-api?style=for-the-badge)](https://github.com/1MC/homebridge-http-opensprinkler-api/pulls)


`homebridge-http-opensprinkler-api` is a [Homebridge](https://github.com/nfarina/homebridge) plugin forked from `homebridge-http-opensprinkler-api` specifically to control an OpenSprinkler system using the firmware API from v2.0 on. Example configuration shown below for simple switching of sprinkler stations ON/OFF and syncing of status is shown below.

To get more details about configuration look at the 
[README](https://github.com/Supereg/homebridge-http-notification-server) from the original author.
automated equipment which can be controlled via http requests. Or you have built your own equipment, for example some sort 
of lightning controlled with an wifi enabled Arduino board which than can be integrated via this plugin into Homebridge.

`homebridge-http-switch` supports three different type of switches. A normal `stateful` switch and two variants of 
_stateless_ switches (`stateless` and `stateless-reverse`) which differ in their original position. For stateless switches 
you can specify multiple urls to be targeted when the switch is turned On/Off.   
More about on how to configure such switches can be read further down.

## Installation

First of all you need to have [Homebridge](https://github.com/nfarina/homebridge) installed. Refer to the repo for 
instructions.  
Then run the following command to install `homebridge-http-switch`

```
sudo npm install -g homebridge-http-switch
```

## Updating the switch state in HomeKit

The _'On'_ characteristic from the _'switch'_ service has the permission to `notify` the HomeKit controller of state 
changes. `homebridge-http-switch` supports two ways to send state changes to HomeKit.

### The 'pull' way:

The 'pull' way is probably the easiest to set up and supported in every scenario. `homebridge-http-switch` requests the 
state of the switch in an specified interval (pulling) and sends the value to HomeKit.  
Look for `pullInterval` in the list of configuration options if you want to configure it.

### The 'push' way:

When using the 'push' concept, the http device itself sends the updated value to `homebridge-http-switch` whenever 
the value changes. This is more efficient as the new value is updated instantly and `homebridge-http-switch` does not 
need to make needless requests when the value didn't actually change.  
However because the http device needs to actively notify the `homebridge-http-switch` there is more work needed 
to implement this method into your http device. 

#### Using MQTT:

MQTT (Message Queuing Telemetry Transport) is a protocol widely used by IoT devices. IoT devices can publish messages
on a certain topic to the MQTT broker which then sends this message to all clients subscribed to the specified topic.
In order to use MQTT you need to setup a broker server ([mosquitto](https://github.com/eclipse/mosquitto) is a solid 
open source MQTT broker running perfectly on a device like the Raspberry Pi) and then instruct all clients to 
publish/subscribe to it.  
For [shelly.cloud](https://shelly.cloud) devices mqtt is the best and only option to implement push-updates.

#### Using 'homebridge-http-notification-server':

For those of you who are developing the http device by themselves I developed a pretty simple 'protocol' based on http 
to send push-updates.   
How to implement the protocol into your http device can be read in the chapter 
[**Notification Server**](#notification-server)

## Configuration:

The configuration can contain the following properties:

#### Basic configuration options:

- `name` \<string\> **required**: Defines the name which is later displayed in HomeKit
- `switchType` \<string\> **optional** \(Default: **"stateful"**\): Defines the type of the switch:
    * **"stateful"**: A normal switch and thus the default value.
    * **"stateless"**: A stateless switch remains in only one state. If you switch it to on, it immediately goes back to off. 
    Configuration example is further [down](#stateless-switch).
    * **"stateless-reverse"**: Default position is ON. If you switch it to off, it immediately goes back to on. 
    Configuration example is further [down](#reverse-stateless-switch).
    * **"toggle"**: The toggle switch is a stateful switch however does not use the `statusUrl` to determine the current 
    state. It uses the last set state as the current state. Default position is OFF.
    * **"toggle-reverse""**: Same as **"toggle"** but switch default position is ON.

* `onUrl` \<string | \[string\] | [urlObject](#urlobject) | [[urlObject](#urlobject)]\> **required**: Defines the url 
(and other properties when using an urlObject) which is called when you turn on the switch.
* `offUrl` \<string | \[string\] | [urlObject](#urlobject) | [[urlObject](#urlobject)]\> **required**: Defines the url 
(and other properties when using an urlObject) which is called when you turn off the switch.
* `statusUrl` \<string | [urlObject](#urlobject)\> **required**: Defines the url 
(and other properties when using an urlObject) to query the current state from the switch. By default it expects the http 
server to return **'1'** for ON and **'0'** for OFF leaving out any html markup.  
You can change this using `statusPattern` option.  

#### Advanced configuration options:

- `statusPattern` \<string\> **optional** \(Default: **"1"**\): Defines a regex pattern which is compared to the body of the `statusUrl`.
When matching the status of the switch is set to ON otherwise OFF. [Some examples](#examples-for-custom-statuspatterns).
- `statusCache` \<number\> **optional** \(Default: **0**\): Defines the amount of time in milliseconds a queried state 
 of the switch is cached before a new request is made to the http device.  
 Default is **0** which indicates no caching. A value of **-1** will indicate infinite caching. 

* `timeout` \<integer\> **optional** \(Default: **1000**\): When using a stateless switch this timeout in 
**milliseconds** specifies the time after which the switch is reset back to its original state.
* `pullInterval` \<integer\> **optional**: The property expects an interval in **milliseconds** in which the plugin 
pulls updates from your http device. For more information read [pulling updates](#the-pull-way).  
(This option is only supported when `switchType` is **"stateful"**)

- `debug` \<boolean\> **optional**: If set to true debug mode is enabled and the plugin prints more detailed information.

Below is an example configuration for two sprinkler stations with run duration set to 10 minutes.  If you have a schedule setup on your OpenSprinkler device, that will continue to run and the status of the stations will be synced every second - as per the pullInterval setting.

```json
{
    "accessories": [
        {
          "accessory": "OPENSPRINKLER-STATION",
          "name": "Sprinkler Stn01",
          
          "switchType": "stateful",
          
          "onUrl": "http://localhost:8080/cm?sid=0&en=1&t=600&pw=xxx",
          "offUrl": "http://localhost:8080/cm?sid=0&en=0&pw=xxx",
          
          "statusUrl": "http://192.168.1.10:8080/js?pw=xxx",
          "statusPattern": "{\"sn\":[1,[0-9]+,[0-9]+,[0-9]+,[0-9]+,[0-9]+,[0-9]+,[0-9]+],\"nstations\":8}",
          "pullInterval": "1000"
        },
        {
          "accessory": "OPENSPRINKLER-STATION",
          "name": "Sprinkler Stn02",
          
          "switchType": "stateful",
          
          "onUrl": "http://localhost:8080/cm?sid=1&en=1&t=600&pw=xxx",
          "offUrl": "http://localhost:8080/cm?sid=1&en=0&pw=xxx",
          
          "statusUrl": "http://192.168.1.10:8080/js?pw=xxx",
          "statusPattern": "{\"sn\":[[0-9]+,1,[0-9]+,[0-9]+,[0-9]+,[0-9]+,[0-9]+,[0-9]+],\"nstations\":8}",
          "pullInterval": "1000"
        }
    ]
}
```

#### UrlObject

A urlObject can have the following properties:
* `url` \<string\> **required**: Defines the url pointing to your http server
* `method` \<string\> **optional** \(Default: **"GET"**\): Defines the http method used to make the http request
* `body` \<any\> **optional**: Defines the body sent with the http request. If value is not a string it will be 
converted to a JSON string automatically.
* `strictSSL` \<boolean\> **optional** \(Default: **false**\): If enabled the SSL certificate used must be valid and 
the whole certificate chain must be trusted. The default is false because most people will work with self signed 
certificates in their homes and their devices are already authorized since being in their networks.
* `auth` \<object\> **optional**: If your http server requires authentication you can specify your credential in this 
object. When defined the object can contain the following properties:
    * `username` \<string\> **required**
    * `password` \<string\> **required**
    * `sendImmediately` \<boolean\> **optional** \(Default: **true**\): When set to **true** the plugin will send the 
            credentials immediately to the http server. This is best practice for basic authentication.  
            When set to **false** the plugin will send the proper authentication header after receiving an 401 error code 
            (unauthenticated). The response must include a proper `WWW-Authenticate` header.  
            Digest authentication requires this property to be set to **false**!
* `headers` \<object\> **optional**: Using this object you can define any http headers which are sent with the http 
request. The object must contain only string key value pairs.  
* `requestTimeout` \<number\> **optional** \(Default: **20000**\): Time in milliseconds specifying timeout (Time to wait
    for http response and also setting socket timeout).
* `repeat` \<number\> **optional** \(Default: **1**\): Defines how often the execution of this urlObject should 
    be repeated.  
    Notice that this property only has an effect on ulrObject specified in `onUrl` or `offUrl`.
    Also have a look at the `multipleUrlExecutionStrategy` property. Using "parallel" execution could result in
    unpredictable behaviour.
* `delayBeforeExecution` \<number\> **optional** \(Default: **0**\): Defines the time in milliseconds to wait 
    before executing the urlObject.  
    Notice that this property only has an effect on ulrObject specified in `onUrl` or `offUrl`.
    Also have a look at the `multipleUrlExecutionStrategy` property.
  
Below is an example of an urlObject containing the basic properties:
```json
{
  "url": "http://example.com:8080",
  "method": "GET",
  "body": "exampleBody",
  
  "strictSSL": false,
  
  "auth": {
    "username": "yourUsername",
    "password": "yourPassword"
  },
  
  "headers": {
    "Content-Type": "text/html"
  }
}
```

### Examples for custom statusPatterns

The `statusPattern` property can be used to change the phrase which is used to identify if the switch should be turned on 
or off. So when you want the switch to be turned on when your server sends **"true"** in the body of the http response you
could specify the following pattern:
```json
{
    "statusPattern": "true"
}
```

<br>

However using Regular Expressions much more complex patterns are possible. Let's assume your http enabled device responds 
with the following json string as body, where one property has an random value an the other indicates the status of the 
switch:
```json
{
    "perRequestRandomValue": 89723789,
    "switchState": true
}
```
Then you could use the following pattern:
```json
{
    "statusPattern": "{\n    \"perRequestRandomValue\": [0-9]+,\n    \"switchState\": true\n}"
}
```
More on how to build regex patterns: https://www.w3schools.com/jsref/jsref_obj_regexp.asp

