# EmonHub Configuration

emonHub is configured with `emonhub.conf` config file. On the emonPi / emonBase this file is located in the R/W data partition `/home/pi/data/emonhub.conf`. If the [Emoncms Config module](https://github.com/emoncms/config) is installed (as in the case of the emonPi / emonBase using pre-buit SD card image) the config file can be edited direct from the EmonHub tab in local Emoncms, see [User Guide](https://guide.openenergymonitor.org)

Emonhub.conf has 3 sections: 

## 1. `hub`

Hub is a section for emonhub global settings such as the loglevel.


## 2. `interfacers`

Interfacers holds the configuration for the different interfacers that emonhub supports such as the EmonHubJeeInterfacer for reading and writing to the RFM69Pi adapter board or emonPi board via serial, or the EmonHubMqttInterfacer which can be used to publish the data received from EmonHubJeeInterfacer to MQTT topics.


## 3. `nodes`

Nodes holds the decoder configuration for rfm12/69 node data which are sent as binary structures.


```
#######################################################################
        #######################      emonhub.conf     #########################
        #######################################################################
        ### emonHub configuration file, for info see documentation:
        ### http://github.com/openenergymonitor/emonhub/blob/master/configuration.md
        #######################################################################
        #######################    emonHub  settings    #######################
        #######################################################################
        [hub]
        ### loglevel must be one of DEBUG, INFO, WARNING, ERROR, and CRITICAL
        loglevel = DEBUG
        
        #######################################################################
        #######################       Interfacers       #######################
        #######################################################################
        [interfacers]
        
        #######################################################################
        #######################          Nodes          #######################
        #######################################################################
        [nodes]
```

**View full latest [default emonHub.conf](https://github.com/openenergymonitor/emonhub/blob/emon-pi/conf/emonpi.default.emonhub.conf)**

***

# 1. 'hub' Configuration 

Hub is a section for emonhub global settings such as the loglevel.

The hub configuration should be self explanatory. Emonhub log can be viewed in the `Setup > EmonHub` section of local Emoncms on an emonPi / emonBase. Default settings are:

```
### loglevel must be one of DEBUG, INFO, WARNING, ERROR, and CRITICAL
loglevel = DEBUG 
### Uncomment this to also send to syslog
# use_syslog = yes
```
***
# 2. 'interfaces' Configuration

Interfacers holds the configuration for the different interfacers that emonhub supports such as the EmonHubJeeInterfacer for reading and writing to the RFM69Pi adapter board or emonPi board via serial, or the EmonHubMqttInterfacer which can be used to publish the data received from EmonHubJeeInterfacer to MQTT topics:


### a.) [[RFM2Pi]]

The `[[RFM2Pi]]` interfacer section contains the settings to read from RFM69Pi / emonPi boards via GPIO internal serial port `/dev/ttyAMA0`. The default serial baud on all emonPi and RFM69Pi is `38400`. Older RFM12Pi boards using `9600` baud. 

The frequency and network group must match the hardware and other nodes on the network. 

The `calibration` config is used to set the calibration of the emonPi when using USA AC-AC adapters 110V. Set `calibration = 110V` when using USA AC-AC adapter.  

```
[[RFM2Pi]]
    Type = EmonHubJeeInterfacer
    [[[init_settings]]]
        com_port = /dev/ttyAMA0
        com_baud = 38400                        # 9600 for old RFM12Pi
    [[[runtimesettings]]]
        pubchannels = ToEmonCMS,
        subchannels = ToRFM12,

        group = 210
        frequency = 433
        baseid = 5                              # emonPi / emonBase nodeID
        quiet = true                            # Report incomplete RF packets (no implemented on emonPi)
        calibration = 230V                      # (UK/EU: 230V, US: 110V)
        # interval =  0                         # Interval to transmit time to emonGLCD (seconds)
``` 


### b.) [[MQTT]]


Emonhub supports publishing to MQTT topics through the EmonHubMqttInterfacer, defined in the interfacers section of emonhub.conf.

There are two formats that can be used for publishing node data to MQTT:

#### **1. Node only format**

(default base topic is `emonhub`)

    topic: basetopic/rx/10/values
    payload: 100,200,300

The 'node only format' is used with the emoncms [Nodes Module](https://github.com/emoncms/nodes) (now deprecated on Emoncms V9) and the emonPiLCD python service.

#### **2. Node variable format**

(default base topic is `emon`)

    topic: basetopic/emontx/power1
    payload: 100

The 'Node variable format' is the current default format on Emoncms V9. It's a more generic MQTT publishing format that can more easily be used by applications such as NodeRED and OpenHab. This format can also be used with the emoncms `phpmqtt_input.php` script in conjunction with the emoncms inputs module. See [User Guide > Technical MQTT](https://guide.openenergymonitor.org/technical/mqtt/).

Default `[MQTT]` config:

```
[[MQTT]]

    Type = EmonHubMqttInterfacer
    [[[init_settings]]]
        mqtt_host = 127.0.0.1
        mqtt_port = 1883
        mqtt_user = emonpi
        mqtt_passwd = emonpimqtt2016

    [[[runtimesettings]]]
        pubchannels = ToRFM12,
        subchannels = ToEmonCMS,

        # emonhub/rx/10/values format
        # Use with emoncms Nodes module
        node_format_enable = 1
        node_format_basetopic = emonhub/

        # emon/emontx/power1 format - use with Emoncms MQTT input
        # http://github.com/emoncms/emoncms/blob/master/docs/RaspberryPi/MQTT.md
        nodevar_format_enable = 1
        nodevar_format_basetopic = emon/
```

To enable the node variable format set `nodevar_format_enable = 1`. To disable the node only format set `node_format_enable = 0`.



### c.) [[emoncmsorg]]

The EmonHubEmoncmsHTTPInterfacer configuration that is used for sending data to emoncms.org can also be found in the interfacers section of emonhub.conf. If you wish to use emoncms.org the only change to make here is to replace the blank apikey with your write apikey from emoncms.org found on the user account page. See [Setup Guide > Setup > Remote logging](https://guide.openenergymonitor.org/setup/remote).

```
    [[emoncmsorg]]
        Type = EmonHubEmoncmsHTTPInterfacer
        [[[init_settings]]]
        [[[runtimesettings]]]
            pubchannels = ToRFM12,
            subchannels = ToEmonCMS,
            url = https://emoncms.org
            apikey = xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
            senddata = 1
            sendstatus = 1
```


**sendstatus** Its possible to the EmonHubEmoncmsHTTPInterfacer to send a 'ping' to the destination emoncms that can be picked up by the myip module which will then list the source IP address. This can be useful for remote login to an home emonpi if port forwarding is enabled on your router.

**senddata** If you only want to send the ping request and no data to emoncms.org set this to 0

You can create multiple of these sections to send data to multiple emoncms instances. For example, if you wanted to send to an emoncms running at emoncms.example.com you would add the following underneath the `emoncmsorg` section described above:

```
    [[emoncmsexample]]
        Type = EmonHubEmoncmsHTTPInterfacer
        [[[init_settings]]]
        [[[runtimesettings]]]
            pubchannels = ToRFM12,
            subchannels = ToEmonCMS,
            url = https://emoncms.example.com
            apikey = xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
            senddata = 1
            sendstatus = 1
```

This time, the API key will be the API key from your account at emoncms.example.com.

***

# 3. 'nodes' Configuration

The 2nd part of the emonhub.conf configuration concerns decoding of RFM12 and RFM69 nodes. Here's an example of what this section looks like from the default emonpi emonhub.conf. The rest of this readme explains what each line means and how to write your own node decoders or adapt existing decoders for new requirements.

```

    #######################################################################
    #######################          Nodes          #######################
    #######################################################################

    [nodes]

    ### List of nodes by node ID
    ### 'datacode' is default for node and 'datacodes' are per value data codes.
    ### if both are present 'datacode' is ignored in favour of 'datacodes'
    ### eg node 99 would expect 1 long and 4 ints, unless the "datacodes" line
    ### was removed, then "datacode" would make it expect any number of longs,
    ### likewise per value "scales" will override default node "scale"

    [[5]]
    nodename = emonPi
    firmware = emonPi_RFM69CW_RF12Demo_DiscreteSampling.ino
    hardware = emonpi
    [[[rx]]]
        names = power1,power2,power1_plus_power2,Vrms,T1,T2,T3,T4,T5,T6,pulseCount
        datacodes = h, h, h, h, h, h, h, h, h, h, L
        scales = 1,1,1,0.01,0.1,0.1,0.1,0.1,0.1,0.1,1
        units = W,W,W,V,C,C,C,C,C,C,p

```

### NodeID

`[[10]]`


A numeric NodeID. This identifies the node to emonHub, and every node within your system must have a unique ID. There may be only one definition for each NodeId. The NodeID is programmed into the node firmware, either in the sketch and/or by switches.

### nodename

    `nodename =`


A text string, for your benefit in identifying each node. *This field is optional.*

MQTT: The nodename can be used with the MQTT interfacer to send topics of the form nodes/nodename/variablename.

### firmware

    `firmware =`

A text string specifying the sketch running on the node. (At present, this is for information only. At some future time, it might be used to auto-configure emonHub and/or the sketch.) *This field is optional.*

### hardware

    `hardware =`

Indicates the host environment for human reference. **This field is optional.**

### rx

    `[[[rx]]]`

This must be "rx" and specifies that the next section is for the config of the sensor values received from a node. Its also possible to define a "tx" section for variables to be sent to the node such as control state's. Documentation on this to be added.

### Datacodes

An un-configured Emonhub will by default assume that RFM12 or RFM69 data packets received are a series of integers, each 2 bytes long. The radio packet format is quite minimal and non-descriptive and so emonhub cant know how to decode the packets from the received data if the packet structure is any different.

Earlier OpenEnergyMonitor nodes always sent a series of integers and so no decoder configuration was needed, more recent revisions now include the sending of pulse counts or watt hours which would overrun the maximum value that can be sent as an integer. The latest EmonPi, EmonTx3 and EmonTH firmware's all send pulse count as long datatypes at the end of their packets taking up 4 bytes.

Its possible to decode any radio packet that is packed as a binary structure with Emonhub. For example if we look at the relevant part of the node decoder for the emontx v3 we can see 11 integers (h) and one unsigned long at the end (L)

```
    [[8]]
        [[[rx]]]
           datacodes = h,h,h,h,h,h,h,h,h,h,h,L
```

The node decoder could be left as this if we only wanted to decode the packet structure correctly. Alternatively if the packet structure is a series of integers its possible to write:

```
    [[1]]
        [[[rx]]]
           datacode = h
```

Notice that the name is datacode rather than datacode**s** with an s. There are 13 different datatypes that can be decoded:

    b: byte, 1 byte
    h: short integer, 2 bytes
    i: integer, 4 bytes
    l: long, 4 bytes
    q: long long, 8 bytes
    f: float, 4 bytes
    d: double, 8 bytes
    B: unsigned byte, 1 byte
    H: unsigned integer, 2 bytes
    I: unsigned integer, 4 bytes
    L: unsigned long, 4 bytes
    Q: unsigned long long, 8 bytes
    c: char, 1 byte

**Note:** Arduino integers are 2 bytes long and so we use the short integer decoder: h.

### Names

It's possible to specify sensor value names to help with identification. The emoncms nodes module can also load these names for its node list. Another possibility not yet implemented is to use these names to publish sensor values to MQTT topics of the form nodes/emontx/power1.

```
    [[8]]
        [[[rx]]]
           names = power1, power2, power3, power4, Vrms, temp1, temp2, temp3, temp4, temp5, temp6, pulse
           datacodes = h,h,h,h,h,h,h,h,h,h,h,L
```

### Scales

In order to keep radio packet length small a sensor value measured as a float on an emontx or emonth (i.e temperature) is first multiplied by 10x or 100x, then sent as a 2-byte integer in the radio packet and then scaled back to the original value on receipt in emonhub. This saves 2 bytes per sensor value and provides a convenient way of providing 1 or 2 decimal place resolution.

The scales to be applied can either be specified for each sensor value as in this example for the emontx:

```
    [[8]]
        [[[rx]]]
           names = power1, power2, power3, power4, Vrms, temp1, temp2, temp3, temp4, temp5, temp6, pulse
           datacodes = h,h,h,h,h,h,h,h,h,h,h,L
           scales = 1,1,1,1,0.01,0.1,0.1,0.1,0.1,0.1,0.1,1
```

In this example the RMS Voltage is multiplied by 0.01 and temperature values by 0.1. Which means that the RMS voltage was multiplied by 100x and temperature values by 10x on the emontx.

or a single scale can be applied (note scale instead of scale**s** with an s)

```
    [[8]]
        [[[rx]]]
           datacodes = h,h,h,h,h,h,h,h,h,h,h,L
           scale = 1
```

The default scale value is 1 and so in the scale where no scaling is needed this line can be left out of the configuration.

The latest version of the emon-pi variant of emonhub does not require the number of scales to match the number of variables, it will scale according to the scales available or scale by 1 if scales are not available.

### Units

A comma-separated list of engineering units to describe the data. Common units are W, kW, V, A, C, %. These are only to help with identification. The are currently made use of in the emoncms nodes module UI.

```
    [[8]]
        [[[rx]]]
           names = power1, power2, power3, power4, Vrms, temp1, temp2, temp3, temp4, temp5, temp6, pulse
           datacodes = h,h,h,h,h,h,h,h,h,h,h,L
           scales = 1,1,1,1,0.01,0.1,0.1,0.1,0.1,0.1,0.1,1
           units =W,W,W,W,V,C,C,C,C,C,C,p
```

## Standard node decoders

The following lists the standard node decoders for recent versions of the EmonPi, EmonTx v3, EmonTH and EmonTxShield. These are currently included in emonhub.conf and provide automatic decoding of node data.

If you upload the firmware yourself to any of these nodes and wish to have the data decoded with names, units and scaled correctly these are the decoders for the standard firmwares. The node decoders are also included at the top of each firmware file for reference.

### EmonPi: emonPi_RFM69CW_RF12Demo_DiscreteSampling.ino, v2.1+

Firmware location: [emonPi_RFM69CW_RF12Demo_DiscreteSampling.ino](https://github.com/openenergymonitor/emonpi/blob/master/Atmega328/emonPi_RFM69CW_RF12Demo_DiscreteSampling/emonPi_RFM69CW_RF12Demo_DiscreteSampling.ino)

Copied here for reference:

```
    [[5]]
        nodename = emonPi
        firmware = emonPi_RFM69CW_RF12Demo_DiscreteSampling.ino
        hardware = emonpi
        [[[rx]]]
            names = power1,power2,power1_plus_power2,Vrms,T1,T2,T3,T4,T5,T6,pulseCount
            datacodes = h, h, h, h, h, h, h, h, h, h, L
            scales = 1,1,1,0.01,0.1,0.1,0.1,0.1,0.1,0.1,1
            units = W,W,W,V,C,C,C,C,C,C,p
```

### EmonTx v3, emonTxV3_4_DiscreteSampling.ino, v2.3+

Firmware location: [emonTxV3_4_DiscreteSampling.ino](https://github.com/openenergymonitor/emonTxFirmware/blob/master/emonTxV3/RFM/emonTxV3.4/emonTxV3_4_DiscreteSampling/emonTxV3_4_DiscreteSampling.ino)

Node ID when DIP switch1 is off = 8, node ID when DIP switch1 is on is 7

Copied here for reference:

```
    [[8]]
        nodename = emonTx_3
        firmware =V2_3_emonTxV3_4_DiscreteSampling
        hardware = emonTx_(NodeID_DIP_Switch1:OFF)
        [[[rx]]]
           names = power1, power2, power3, power4, Vrms, temp1, temp2, temp3, temp4, temp5, temp6, pulse
           datacodes = h,h,h,h,h,h,h,h,h,h,h,L
           scales = 1,1,1,1,0.01,0.1,0.1, 0.1,0.1,0.1,0.1,1
           units =W,W,W,W,V,C,C,C,C,C,C,p
```

### EmonTx v3, emonTxV3_4_DiscreteSampling.ino, v1.6+

Can be on either nodeid 10 or 9

    [[10]]
        nodename = emonTx_1
        firmware =V1_6_emonTxV3_4_DiscreteSampling
        hardware = emonTx_(NodeID_DIP_Switch1:OFF)
        [[[rx]]]
           names = power1, power2, power3, power4, Vrms, temp1, temp2, temp3, temp4, temp5, temp6, pulse
           datacode = h
           scales = 1,1,1,1,0.01,0.1,0.1, 0.1,0.1,0.1,0.1,1 #Firmware V1.6
           units =W,W,W,W,V,C,C,C,C,C,C,p

### EmonTx v3, emonTxV3_4_DiscreteSampling.ino, <v1.4

    [[10]]
        nodename = emonTx_1
        firmware =V1_6_emonTxV3_4_DiscreteSampling
        hardware = emonTx_(NodeID_DIP_Switch1:OFF)
        [[[rx]]]
           names = power1, power2, power3, power4, Vrms, temp
           datacode = h
           scales = 1,1,1,1,0.01,0.1
           units =W,W,W,W,V,C

### EmonTH, emonTH_DHT22_DS18B20_RFM69CW_Pulse.ino, v2.6+

Firmware location: [emonTH_DHT22_DS18B20_RFM69CW_Pulse.ino](https://github.com/openenergymonitor/emonTH/blob/master/emonTH_DHT22_DS18B20_RFM69CW_Pulse/emonTH_DHT22_DS18B20_RFM69CW_Pulse.ino)

Standard nodeid's: 23, 24, 25 & 26 depending on DIP switch positions:

    [[23]]
        nodename = emonTH_5
        firmware = V2.x_emonTH_DHT22_DS18B20_RFM69CW_Pulse
        hardware = emonTH_(Node_ID_Switch_DIP1:OFF_DIP2:OFF)
        [[[rx]]]
           names = temperature, external temperature, humidity, battery, pulseCount
           datacodes = h,h,h,h,L
           scales = 0.1,0.1,0.1,0.1,1
           units = C,C,%,V,p

### EmonTH, emonTH_DHT22_DS18B20_RFM69CW.ino v1.5 -> v1.6.1

Firmware location: [emonTH_DHT22_DS18B20_RFM69CW.ino](https://github.com/openenergymonitor/emonTH/blob/master/emonTH_DHT22_DS18B20_RFM69CW/emonTH_DHT22_DS18B20_RFM69CW.ino)

Standard nodeid's: 19, 20, 21 & 22 depending on DIP switch positions:

    [[19]]
        nodename = emonTH_1
        firmware = emonTH_DHT22_DS18B20_RFM69CW
        hardware = emonTH_(Node_ID_Switch_DIP1:OFF_DIP2:OFF)
        [[[rx]]]
           names = temperature, external temperature, humidity, battery
           datacode = h
           scales = 0.1,0.1,0.1,0.1
           units = C,C,%,V

### EmonTx Shield

Firmware location: [Shield_CT1234_Voltage.ino](https://github.com/openenergymonitor/emonTxFirmware/blob/master/emonTxShield/Shield_CT1234_Voltage/Shield_CT1234_Voltage.ino)

    [[6]]
        nodename = emonTxShield
        firmware =emonTxShield
        hardware = emonTxShield
        [[[rx]]]
           names = power1, power2, power3, power4, Vrms
           datacode = h
           scales = 1,1,1,1,0.01
           units =W,W,W,W,V

***
# 4. Troubleshooting

### Node data inactive or, node data does not appear for a configured node

Try replacing the datacodes = h,h,h,h,... line with **datacode = h** (note: datacode without an s). This will decode most of the radio packet content for the standard OpenEnergyMonitor emontx,emonth and emonpi firmwares, including historic versions.

### The data still does not appear on the nodeid I expect

The EmonTx and EmonTH nodes both have switches on their circuit boards for changing the nodeid to save having to reprogram. Depending on the switch positions and firmware version the EmonTx v3 can be on node 7,8,9 or 10. The EmonTH can be on 19,20,21,22,23,24,25 or 26.
