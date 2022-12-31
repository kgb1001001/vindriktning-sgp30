# Homebridge MQTT-Thing and MQTT

The original VINDRIKTNING project has the ESP8266 send its PM25 readings out via MQTT. MQTT used to stand for MQ Telemetry Transport (although now it's just an acryonym without meaning) and was originally invented by my old friend and IBM colleague Dr. Andy Stanford-Clark. What has happened over the last few years is that MQTT, which is a very lightweight internet messaging protocol, has become one of the defacto standards for Internet of Things devices.

MQTT is simple to understand if you're familiar with queueing or messaging software.  It's a topic-based notification system (so it's a best-effort fire and forget approach as opposed to the heavier-weight transactional queueing systems like the original MQ Series) which requires a piece of broker software to bridge between clients sending and receiving notifications.  The best example (and most common) of an MQTT Broker is the open-source [Eclipse Mosquitto project](https://mosquitto.org/), which is available on most major operating systems and hardware ecosystems, including, for our example, Raspberry Pi.

That's where I am running my Mosquitto broker, and it's probably one of the best options for most people.  Rather than create yet another tutorial on how to do it, I'll point you to the best one I've found for installing Mosquitto on the Pi, which can be [found here](https://randomnerdtutorials.com/how-to-install-mosquitto-broker-on-raspberry-pi/). 

One important consideraiton is that you need to follow the bottom part of the tutorial where you enable remote access with a user id and password. It's worthwhile to create a special MQTT client userid and password on the Pi specifically for clients to connect to. Make sure you add this user/password to the password file as the instructions show, and also make sure that you note down the address of the Raspberry Pi on your network, as you will probably need that as well in one of the later steps. (That is unless you want to get fancy and assign a .local hostname to your raspberry pi as [this](https://www.howtogeek.com/167190/how-and-why-to-assign-the-.local-domain-to-your-raspberry-pi/) tutorial describes).

Once Mosquitto is up and running on your Raspberry Pi, then the next step is to set up the Wifi of the ESP8266 board onboard the VINDRIKTNING.  The fantastic thing about that is that the developers of the original software made this super easy by using the [WifiManager](https://github.com/tzapu/WiFiManager/) library, which provides a configuration portal you can access with your mobile phone. The first time you plug in the VINDRIKTNING after installing the ESP8266 board and the SGP30, open up your mobile phone, look for the wifi access point from the VINDRIKTNING and then once it connects to that access point, press the button to configure Wifi.  

In that screen, select your wifi network, enter the WPA password, and then below where it asks for MQTT configuration, replace the example MQTT address with the network address of your raspberry pi, and enter the userid and password you added in the step above to allow the VINDRIKTNING to communicate with the Mosquitto broker.

Now, before you move on it's probably good to check to see if communication with the broker is actually happening. You can use Mosquitto's built in command line tools to check for this, but actually I prefer using Thomas Nordquist's awesome [MQTT Explorer](http://mqtt-explorer.com/) client instead.  You run it on another PC, connect to the Mosquitto server on the Raspberry Pi using the same address, userid and password as with the ESP8266, and can monitor all of the messages arriving on the various topics.

The VINDRIKTHING project originally assumed that you would be using the HomeAssistant project, which includes integration with MQTT for many devices.  As such, it defines a number of topics that would be automatically picked up by the software.  In my example, where we are instead going to use Homebridge, you just need to note down those topics for use later. 

The first thing you have to do after setting up Mosquitto is to then install the MQTT-Thing plugin for HomeBridge. You can find it in the Homebridge GUI by doing a search for MQTT-Thing, or you can find it [here](https://github.com/arachnetech/homebridge-mqttthing).  Once you install the plugin, the Homebridge GUI will ask you for the following information: 

  - The IP Address of the Mosquitto MQTT Broker
  - the Username and password for MQTT that you set up when installing Mosquitto
  
You will also have to select an accessory type and give it a name.  Select "airQualitySensor" and name it anything you like (I used "Vindriktning-SGP30").  The most complex part of this is setting up the topics for the Air Quality Sensor accessory.  There are four that you need to configure, "Get Air Quality", "Get Carbon Dioxide Level", "Get PM2_5 Density" and "Get VOC Density".  In each case, you need the exact topic names that were set up by Ardunio software - which use the name of the particular sensor as the second part of the topic name.  This is why we recommended you first check the connection with MQTTExplorer - this will give you the exact topic names.  Assuming you did not modify the software, there should be  six-digit id code that will be the same across all topics that you can substitute below for YYYYY.
  
The final part of the string you will need to enter is the particular JSON element on the topic that each element matches to.  You can also see this with MQTTExplorer. For more information on the structure of the MQTT-Thing JSON parsing mechanism, see the link above.
  
  - "getAirQuality" use the string: "esp8266-vindriktning-particle-sensor/VINDRIKTNING-YYYYYY/state$.quality"
  - "getCarbonDioxideLevel" use the string: "esp8266-vindriktning-particle-sensor/VINDRIKTNING-YYYYYY/state$.eCO2"
  - "getPM2_5Density" use the string : "esp8266-vindriktning-particle-sensor/VINDRIKTNING-YYYYYY/state$.pm25"
  - "getVOCDensity" use the string: "esp8266-vindriktning-particle-sensor/VINDRIKTNING-YYYYYY/state$.tVOC"

