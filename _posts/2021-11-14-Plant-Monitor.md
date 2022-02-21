---
layout: post
title: Building a Plant Monitor for Soil Moisture and Ambient Conditions
published: true
---

Note: for a step-by-step walkthrough, please refer to the READMEs in [this GitHub Repository](https://github.com/augustweinbren/casa0014/tree/main/plantMonitor).

## Introduction

A plant monitor can be used to sense the conditions of a plant, providing insight into when it would best be watered, and whether the ambient conditions match those which suit the plant.

This particular plant monitor is used to track the ambient humidity and temperature along with the soil moisture to provide information to the plant gardener as to when the plant should be watered, and if the plant should be moved to a different space.

## Pre-requisites
- [Items needed](https://github.com/augustweinbren/casa0014/tree/main/plantMonitor#items-needed-).
- [Software installation guide](https://github.com/augustweinbren/casa0014/tree/main/plantMonitor#installations-).

## Arduino-based software development
[Step-by-step instructions](https://github.com/augustweinbren/casa0014/tree/main/plantMonitor#adapting-and-running-examples-to-ensure-board-is-working-).

In summary, the first development tasks required for this project involve sending a series of test scripts to the Arduino Feather Huzzah board to ensure that all of the necessary features work as needed. Specifically, this process consists of first testing the `Hello World` equivalent of Arduino, followed by testing the device's ability to connect to WiFi, moving onto testing its ability to get time data from the internet, and finally testing the device's ability to publish to and subscribe to an MQTT server. While not identical, one can see the parallels between this approach and Test-Driven-Development.

By deploying the test scripts in order of increasing complexity, one will be able to narrow down any bugs much more quickly than if they only ran `testMQTT`, precisely because all of the prior tests cover different necessary components of `testMQTT`. Additionally, by running all of these tests prior to breadboard prototyping, one will be able to catch a faulty board early and avoid the need to swap it out after already incorporating it into a circuit. Lastly, the iterative style of tests also is useful from a code interpretability perspective--creating tests in this way will allow other developers to understand each component of the final source code better because they will have additional examples in simpler files.

## Plant Monitor Circuitry

The second task involves prototyping the sensor on a breadboard before soldering an equivalent on a PCB.

For reproducability, a fritzing diagram detailing the integrated moisture-DHT22 circuitry is in [the GitHub repository](https://github.com/augustweinbren/casa0014/tree/main/plantMonitor#circuit-prototyping-).

![nail sensor in plant]({{ site.baseurl }}/images/01_plant_monitor/nail-sensor.jpg "Finished plant-monitor with nails inserted into plant")

To understand the circuitry, it makes sense to focus on the soil moisture sensor, as this is both more fault-prone and relevant for the care of a houseplant (because ambient humidity and temperature are less realistic to adjust in response to the outputs of the plant monitor).

 The sensor consists of two nails placed in different parts of the soil. When the separating soil's moisture rises, the electrical resistivity of the soil drops. By connecting the analog input pin before an additional 10K resistor, a voltage divider is created, and the following formula can describe the circuit:

 ![\Large V_out=V_in\frac{R_2}{R_1+R_2}](https://latex.codecogs.com/svg.latex?\Large&space;V_{out}=V_{in}\frac{R_2}{R_1+R_2})

where ![\V_{in}](https://latex.codecogs.com/svg.latex?V_{in}) represents the 5V input voltage, ![\V_{out}](https://latex.codecogs.com/svg.latex?V_{out}) represents the analog voltage measurement, ![\R_2](https://latex.codecogs.com/svg.latex?R_2) is the 10K resistor and ![\R_1](https://latex.codecogs.com/svg.latex?R_1) is the proxy-resistor created by the nail-soil-nail connection (Analog Input, 2018).

Based on this formula, a high measured voltage will correspond to a high soil conductivity and therefore high soil moisture.

![Moisture sensor circuit powered by digital pin]({{ site.baseurl }}/images/01_plant_monitor/fritzingBasicDiagram.jpg "Basic moisture sensor circuit (Tucker, 2015)")

While the basic circuit described above would function without issue for a time, the underlying electrolysis reaction that occurs when power is sent to the first nail leads to corrosion and increasing uncertainty in the moisture measurements. To preserve the device, it is therefore best to only send power through the nails when a reading is needed. 

There are a couple of ways to do this. The approach used in this iteration of the plant monitor makes use of a transistor, whose middle pin is linked to a digital pin to turn the sensor on and off. It also requires an additional resistor between the nail and the analog input to bring the maximum voltage down to 1V, based on the board's analog input standards.

![Fritzing diagram of transistor-enabled moisture sensor]({{ site.baseurl }}/images/01_plant_monitor/nailsAndTransistorBasic.png "Moisture sensor circuit enabled by transistor")

However, I believe a more lightweight (and potentially resilient) solution could be achieved by instead turning the powered nail on and off with a digital pin set to output. This would remove the need for a transistor.

![Fritzing diagram of digital pin-enabled moisture sensor]({{ site.baseurl }}/images/01_plant_monitor/NailsAndTransistorDigitallyControlled_bb.png "Moisture sensor circuit powered by digital pin")

### Raspberry Pi-based Data Collection and Visualisation

Please refer to [this section of the README](https://github.com/augustweinbren/casa0014/tree/main/plantMonitor#raspberry-pi-based-data-collection-and-visualisation-) for step-by-step instructions on setting up the plant monitor's data collection and visualisation.

To collect and visualise the plant monitor's data, the Telegraf-InfluxDB-Grafana stack was run on a Raspberry Pi. Telegraf reads data from the MQTT server. This data is stored in InfluxDB, a time-series database. Grafana lastly allows for visualisation of the data. As the three software tools are open source, running them on Raspberry Pi allows ultimately for a no-cost data dashboard which can be run indefinitely.

# Reflection

A plant monitor is a relatively simple IoT system. Still, its development illustrates a key challenge and opportunity that the Internet of Things community currently faces. Building an IoT system relies on a wide-range of skills: the crucial software and hardware development both rely on specialised technical knowledge. Additionally, understanding how the sensors work relies on scientific knowledge. It can thus be quite easy to skim over an area of understanding in the development of an IoT system, potentially leaving a device with less longevity and more data uncertainty. However, I believe that open source development can solve this issue by opening the door to improvement through the crowdsourcing of previously unconsidered information.

It is worth returning to the plant monitor to understand the value added by Open Source development. The above-described soil moisture sensor is governed by an electrolysis reaction. In electrolysis, a charge is propagated between two metallic electrodes (in this case two nails) via a conductive electrolyte (in this case the soil) (Atkins, de Paula, and Keeler, 2018). Notably, as a reduction-oxidation reaction, corrosion is an expected outcome at the oxidation site (the anode, or nail connected to the voltage source). (Electrolytic Cells, 2020)

Thus, the less often that electricity will be sent between the two nails to take moisture measurements, the less corrosion that will occur. This aspect of moisture sensing unfortunately is often missed during the development of plant monitors. For example, in the Write-Up [Moisture Detection With Two Nails](https://www.instructables.com/Moisture-Detection-With-Two-Nails/), the issue of electrolysis is ignored (Tucker, 2015). However, a mention is made in the comments by another user about the corrosive nature of this sensor (Tucker, 2015). In response, Tucker (2015) iterates on his project, and links [his new write-up](https://www.instructables.com/id/SoilMoisture-Detection-System/) in which he sends current through the nails only when a button is pressed. While there remain unsolved questions, such as how to account for instrumental drift or how to empirically measure the amount of corrosion that has occurred in the nails, open sourcing the development of this circuit allowed the maker to iterate on the design and thus greatly reduce the rate of corrosion.

On a different side of the Internet of Things community from the hobbyists, I believe that urban officials would also greatly benefit from open sourcing their IoT system information. For example, most of the sensors in London are opaque. They provide little information on what data they are collecting, not only to the public, but also between the technicians and engineers hired to develop and maintain the devices--as Duncan Wilson mentioned during a lecture on 18 October 2021, operators are only knowledgable about the specific sensors that they are hired to maintain. 

![Sensor in front of bike lane at traffic intersection]({{ site.baseurl }}/images/01_plant_monitor/bike-lane-sensor.jpg "Sensor at top right of traffic light with opaque usage")

Open sourcing government data has worked out very well in the past. In 1994, SEC first released data from its EDGAR database over the internet with Carl Malamud putting it on a non-governmental website. After announcing that he would shut the website down unless the SEC took over the webhosting, 15,000 people signed a petition to keep the website up, and the SEC took the website over. The initiative spawned such entrepreneurial ventures as Yahoo! Finance and Google Finance, proving the value that the open data provided (O'Reilly, 2017).

The Code for America initiative illustrates that open source government operations work well across an array of sectors. By building apps with the aid of commercial software developers and without the red tape that accompanies government procurement, development was faster, cheaper, and produced better results. By open sourcing these apps, other cities were also able to readily adapt software that, prior to this initiative, would have needed to be developed from scratch again, despite being easily replicable (O'Reilly, 2017). 

Although what I propose extends beyond the realm of open sourcing data and software, I believe that the demonstrated benefits of open government will only be magnified with the incorporation of hardware with the open sourcing of its sensor-based systems. Given there are more moving parts, there is more room for error. With more room for error comes more room for innovation, and thus even more benefit to consulting the community of civic hackers.

# References

Anon (2018). *Analog Input*.
https://www.arduino.cc/en/Tutorial/BuiltInExamples/AnalogInput [Accessed 16 November 2021].

Anon (2020). *Electrolytic Cells*. https://chem.libretexts.org/@go/page/270 [Accessed 16 November 2021].

<div class="csl-entry">Atkins, P., de Paula, J., &#38; Keeler, J. (2018). <i>Atkins’ Physical Chemistry, Thermodynamics and Kinetics</i>.</div>
<br>
O'Reilly, T. (2017). <i>WTF? What's The Future and Why It's Up to Us</i>.

Tucker, R. (2015). *Moisture Detection with Two Nails*. https://www.instructables.com/Moisture-Detection-With-Two-Nails/ [Accessed 16 November 2021].

Tucker, R. (2015). *Soil/Moisture Detection System*. https://www.instructables.com/SoilMoisture-Detection-System/ [Accessed 16 November 2021].



