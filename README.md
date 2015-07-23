# DRePCap
Distributed Remote Packet Capturing (DRePCap)

Please note: The current prototype only supports Linux.

## Overview
DRePCap consists of four components.

### [clj-jms-activemq-toolkit](https://github.com/fg-netzwerksicherheit/clj-jms-activemq-toolkit)
clj-jms-activemq-toolkit is a simple runner for starting a JMS broker.
It also contains additional functionality that is required by the frontend, e.g.,
for querying the available sensors and merges that are currently connected to the broker.

### [drepcap-sensor](https://github.com/fg-netzwerksicherheit/drepcap-sensor)
The drepcap-sensor captures packets and sends them to the JMS infrastructure.

### [drepcap-merger](https://github.com/fg-netzwerksicherheit/drepcap-merger)
In cooperative scenarios, the drepcap-merger is used for merging the data streams from individual sensors.

### [drepcap-frontend](https://github.com/fg-netzwerksicherheit/drepcap-frontend)
The drepcap-frontend is a graphical user interface (GUI) for controlling DRePCap.
Additionally, drepcap-frontend consists the self-adaptivity logic and functionality for emitting sensor data to fifos.

### Architecture Overview

<p align="center">
<img alt="DRePCap Architecture Overview" src="https://github.com/fg-netzwerksicherheit/drepcap/raw/master/doc/images/drepcap_architecture.png"/>
</p>

### Paper and Referencing
In the paper "Monitoring Traffic in Computer Networks with Dynamic Distributed Remote Packet Capturing" by Ruediger Gad, Martin Kappes, and Inmaculada Bulo, which was presented at IEEE ICC 2015 in London, the concepts of DRePCap as well as results of a detailed evaluation are presented.
For referencing DRePCap or this paper, the preliminary bibtex entry is as follows:

    @inproceedings{gad_monitoring_2015,
	title = {Monitoring {Traffic} in {Computer} {Networks} with {Dynamic} {Distributed} {Remote} {Packet} {Capturing}},
	booktitle = {2015 {IEEE} {International} {Conference} on {Communications} ({ICC})},
	author = {Gad, Ruediger and Kappes, Martin and Medina-Bulo, Inmaculada},
	month = jun,
	year = {2015},
    note = {in press},
    }



## Important Preparations
Please note: Before starting the drepcap-sensor make sure that the native libpcap library is installed and that it can be accessed via "libpcap.so".
To check this you can do:

    find /usr -name "libpcap*"

If libpcap is installed but no "libpcap.so" can be found, a link has to be added as follows to make DRePCap work.

    sudo ln -s /usr/lib64/libpcap.so.1 /usr/lib64/libpcap.so

This example is for a current Fedora installation and it may need to be adapted to the specific distribution and version of libpcap.

## Usage
Please note that the components have to be started in the given order.
Otherwise, problems may occur.

### Broker
At first, the JMS broker instance has to be started as follows:

    java -jar clj-jms-activemq-toolkit-1.0.0-standalone.jar

By default the broker is only accessible via localhost.
In order to make the broker accessible externally the "-u" option is used, e.g.: "-u tcp://10.0.0.123:61616".
For more information please use the "--help" command line argument.

### Sensor
Next, the sensor can be started, e.g., as follows:

    java -jar drepcap-sensor-1.0.0-standalone.jar -i lo -q 10 -f ""

With these options, the sensor will start actively capturing data at the loopback interface and will use a queue size of 10.
For our experiments, we used a fixed queue size of 5000.
For a description of all options please use the "--help" command line argument.

Please note: When starting multiple sensors it is crucial that each sensor is assigned with a unique id via the respective command line option.

### Merger
Optionally, a merger can be started as follows:

    java -jar drepcap-merger-1.0.0-standalone.jar -q 10

This starts a merger with a queue size of 10.
For our experiments, we used a fixed queue size of 5000.
For a description of all options please use the "--help" command line argument.

Please note: When starting multiple mergers it is crucial that each merger is assigned with a unique id via the respective command line option.

### Frontend
The frontend is started as follows:

    java -jar drepcap-frontend.jar

The pre-configured broker URL in the frontend connects to the localhost.
In order to connect to the broker press "Connect".

After connecting to the broker the list with connected components will be updated.
Single sensors are indicated with "pcap.single".
Mergers are indicated with "pcap.merged".
In order to open an according tab, please double click the respective entry in the list.

#### FiFo Output
In order to emit data to a fifo press "To fifo".
The fifo will be created in the directory from which the frontend was started and the name will be the entity name from the list with the suffix ".fifo".

One way for testing the fifo output is to use cat for reading from it, e.g.:

    cat pcap.single.raw.1.fifo

The emitted data is prefixed with a pcap file header such that arbitrary applications can read the emitted data like a pcap file.
A simple way to test this is to use Wireshark for reading from the fifo.

To test the fifo output with Wireshark, please start Wireshark.
Click "Capture Option" in order to configure the capturing.
Next, click "Manage Interfaces".
Click "New" followed by "Browse".
In the file selection dialog, select the respective fifo (the one with the ".fifo" suffix).
Click "Save" followed by "Close".
In the list of capture devices, double check that the new entry for the fifo is selected.
Finally, press start to start reading from the fifo.

Please note: We purposely used a small queue size of 10 in the examples given above in order to shorten the time the data is delayed due to queueing.

#### Single Sensor Self-adaptivity
In order to activate single sensor self-adaptivity press "Enable Simple Adapt." in the respective sensor tab in the drepcap-frontend.

For generating traffic for testing the self-adaptivity the "traffic-generation-helper.sh" script can be used.
Please note that this script requires hping3 which has to be installed before.

The traffic generation helper script can be started as follows:

    ./traffic-generation-helper.sh 127.0.0.1

Please note that the HPING3_EXECUTABLE variable in the script may need to be set accordingly to match your setup.

After starting, please press a key to start the actual traffic generation.
The traffic generation rate can be changed interactively with the "+" and the "-" keys.
For stopping the generation press "q".
Please note that the depicted packet rates are theoretical values and that the actually achieved rates may differ.

For actually testing the self-adaptivity, we suggest to start the sensor such that it is limited to run on a single CPU core, e.g., via taskset:

    taskset 0x01 java -jar drepcap-sensor-1.0.0-standalone.jar -i lo -q 10 -f ""

## Download
Pre-built Jar files can be found in the [dist](https://github.com/fg-netzwerksicherheit/drepcap/tree/master/dist) directory of this repository.
The source code of the individual components can be found in the respective repositories:

- [clj-jms-activemq-toolkit](https://github.com/fg-netzwerksicherheit/clj-jms-activemq-toolkit)
- [drepcap-sensor](https://github.com/fg-netzwerksicherheit/drepcap-sensor)
- [drepcap-merger](https://github.com/fg-netzwerksicherheit/drepcap-merger)
- [drepcap-frontend](https://github.com/fg-netzwerksicherheit/drepcap-frontend)

## License
Copyright 2014, Frankfurt University of Applied Sciences

This software is released under the terms of the Eclipse Public License 
(EPL) 1.0. You can find a copy of the EPL at: 
http://opensource.org/licenses/eclipse-1.0.php

