﻿---
uid: 1_6_toolbox_fnsg
title: Extending the Multi-Run Framework
---
# Full Network Set Generator
This tool takes a "Base Scenario" and creates time period specific networks (AM, MD, PM, EV, ON). It also cleans those networks for use in Auto and Transit assignments. For more information on the tool algorithm and the various components within the tool please visit the [Full Network Set Generator Page](https://tmg.utoronto.ca/doc/1.6/gtamodel/model_design/full_network_generator.html)



## Opening the Tool with Modeller
This module can be found in Emme Modeller. Navigate to "TMG Toolbox" -> "Network Editing" -> "Full Network Set Generator". Double click the module to open it in Emme Modeller. 

## Using the Tool with Modeller

The "Base Scenario" is the scenario that contains all the information (other than headways and speeds for transit lines) for the different time periods. 

### Data Files
#### Transit Service Table
This file contains trip starts and trip ends for transit lines. It uses a csv file with three columns

- **emme_id**: This is the id of the transit line in Emme
- **trip_depart**: the departure time of the trip from the initial stop in a hh:mm format
- **trip_arrive**: the arrival time of the trip to the final destination in a hh:mm format

#### Data for non-service table lines (optional)
This file contains information for transit lines that do not have any entries in the "Transit service table" file. It is again a .csv file with the following columns

- **emme_id**: This is the id of the transit line in Emme
- **xxxx_hdw**: This is the headway column corresponding to the time period starting with "xxxx" in a "hhmm" format. For example: The "0600_hdw" column will contain the headways for the time period starting at 6:00 AM. 
- **xxxx_spd**: This is the speed column corresponding to the time period starting with "xxxx" in a "hhmm" format. For example: The "0600_spd" column will contain the speeds for the time period starting at 6:00 AM. 

If more than one time period is being created by this tool at the same time, the last two columns can be repeated for all time periods being created. 

>Transit Lines that do not have an entry in either the Service Table file or the Data for Non-Service Table Lines file will be deleted from the time period specific networks. If a line is present in both files, the Data for Non-Service Table Lines file will take precedent.

>The standard GTAModelV4 time periods are as follows:
> - AM: 6:00AM - 8:59AM
> - MD: 9:00AM - 2:59PM
> - PM: 3:00PM - 6:59PM
> - EV: 7:00PM - 11:59PM
> - ON: 12:00AM - 5:59AM

#### Aggregation Type Selection
This csv file contains the transit lines and how the headway is to be calculated for each line if a Transit Service Table file is used for that line. It contains two columns

- **emme_id**: this is the id of the transit line in Emme
- **agg_type**: this is the headway calculation method. It can either be 'naive' or 'average'

> Headway calculation methods are detailed as follows. For example if line "T501" has 3 departures in the AM period (6 AM- 9AM) at 8:01, 8:11, and 8:31.
> The Naive aggregation approach would simply take the number of departures in a time period. For line "T501" the naive aggregation approach would say that 3 hour AM period means 180 minutes. 180/3 = 60 minute headways.
>  The average aggregation method looks at the time in between departures and takes the average of that time. For line "T501", the average approach would say that the time in between departures is 10 and 20 mins, the average of which is 15 minutes. The headway is then imputed as 15 minutes. 

#### Batch Line Editing (optional)
This file allows the user to make changes to specific lines as specified by the filter. This is a csv file file that has at least the following three columns. 

- **filter**
- **x_hdwchange**: x is the scenario number
- **x_spdchange**: x is the scenario number

Additional headway and speed pairs can be added for more scenarios.

#### Default Aggregation Type
This lets the user choose the aggregation types for lines that are not specifically identified in the "Aggregation Type Selection" csv file. 

### Scenarios

- **Overwrite Full Scenarios**: This box allows the tool to overwrite scenarios that already exist
-  **Publish Network**: This publishes the networks
- **Use Custom Scenario List**: This uses the "Custom Scenario List" box rather than the scenario list 
- **Custom Scenario List**: Definitions for a custom set of scenarios. Use the following syntax. The .nup file is optional. 
Syntax: [Uncleaned scenario number] : [Cleaned scenario number] : [Uncleaned scenario description] : [Cleaned scenario description] : [Scenario start] : [Scenario End] : [.nup file]
Use integer hours for start and end times. For Example 2:30 PM = 1430 

> For each time period, there is an uncleaned and cleaned scenario. The uncleaned scenario is simply the Base scenario but with time period specific headways and speeds for the transit lines. The cleaned network removes cosmetic nodes and links in order to decrease processing times and allow for the creation of a transit fare hypernetwork.

- **Time Period List**
The tool then asks for the definitions of the scenarios being created, the scenario numbers, as well as the start and end times for each scenario.

### Transfer Modes
These are the modes that are there to transfer between transit lines and transit agencies.

### Network Filters
These filters are all used to clean the networks from the Uncleaned Scenario to the cleaned scenario. 
#### Node Filter Attribute
The node filter attribute will only remove candidate nodes whose attribute value != 0. Select 'No attribute' to remove all candidate nodes.
#### Stop Filter Attribute
The stop filter attribute will remove candidate transit stop nodes whose attribute value != 0. Select 'No attribute' to preserve all transit stops
#### Connector Filter Attribute
The connector filter attribute will remove centroid connectors attached to candidate nodes whose attribute value != 0. Select 'No attribute' to preserve all centroid connectors
#### Line Filter Expression
 The set of lines that are selected here will have prorated transit speeds input into us1. 
 
### Aggregation Functions

This lets the tool know how to handle segments, nodes and links when they are cleaned in the network. For example if a cosmetic node is removed and the links that are connected to the node are merged, what should happen to the attributes of those links. It might be obvious that the length should be summed, but the other attributes are much more up to the users judgement.

The syntax is as follows
"[attribute name] : [function], ..."

Separate (attribute-function) pairs with a comma or new line. Either the Emme Desktop attribute names (e.g. 'lanes') or the Modeller API names (e.g. 'num_lanes') can be used. Accepted functions are:
- first - Uses the first element's attribute
- last - Uses the last element's attribute
- sum - Add the two attributes
- avg - Averages the two attributes
- avg_by_length - Average the two attributes, weighted by link length
- min - The minimum of the two attributes
- max - The maximum of the two attributes
- and - Boolean AND
- or - Boolean OR
- force - Forces the tool to keep the node if the two attributes are different

The default function for unspecified extra attributes is 'sum.'

## Using the Tool with XTMF
The tool is called "FullNetworkSetGenerator". It is available to add under ExecuteToolsFromModellerResource or EmmeToolsToRun.

### Attribute Aggregator
This is a string which specifies how to handle links and segments when they are merged. For example if a cosmetic node is removed and the links that are connected to the node are merged, what should happen to the attributes of those links. It might be obvious that the length should be summed, but the other attributes are much more up to the users judgement.

The syntax is as follows
"[attribute name] : [function], ..."

Separate (attribute-function) pairs with a comma. Either the Emme Desktop attribute names (e.g. 'lanes') or the Modeller API names (e.g. 'num_lanes') can be used. Accepted functions are:
- first - Uses the first element's attribute
- last - Uses the last element's attribute
- sum - Add the two attributes
- avg - Averages the two attributes
- avg_by_length - Average the two attributes, weighted by link length
- min - The minimum of the two attributes
- max - The maximum of the two attributes
- and - Boolean AND
- or - Boolean OR
- force - Forces the tool to keep the node if the two attributes are different

The default function for unspecified extra attributes is 'sum.'

### Base Scenario Number
The scenario number from which Full Network Set Generator will build its time period out from. 
### Connector Filter Attribute
The connector filter attribute will remove centroid connectors attached to candidate nodes whose attribute value != 0. Input "None" to preserve all centroid connectors
### Default Aggregation
The default method to calculate headways for lines that are not in the "Transit Aggregation Selection Table"
### Line Filter Expression
The line that this network filter expression will select will have transit speeds prorated and input into us1.
### Node Filter Expression
The node filter attribute will only remove candidate nodes whose attribute value != 0. Select 'No attribute' to remove all candidate nodes.
### Stop Filter Attribute
The stop filter attribute will remove candidate transit stop nodes whose attribute value != 0. Select 'No attribute' to preserve all transit stops
### Transfer Modes
These are the modes that are there to transfer between transit lines and transit agencies.

### Additional Transit Alternative Table
Additional files containing hwo to modify transit aschedules. Each will be applied in order. They should use the same format as the "Transit Alternative Table" ie  a .csv file with the following columns

- **emme_id**: This is the id of the transit line in Emme
- **xxxx_hdw**: This is the headway column corresponding to the time period starting with "xxxx" in a "hhmm" format. For example: The "0600_hdw" column will contain the headways for the time period starting at 6:00 AM. 
-**xxxx_spd**: This is the speed column corresponding to the time period starting with "xxxx" in a "hhmm" format. For example: The "0600_spd" column will contain the speeds for the time period starting at 6:00 AM. 

Additional **xxxx_hdw** and **xxxx_spd** columns should be specified for all time periods that need to be modified.

It is added by adding in the appropriate module in XTMF under it and then selecting the file location.

### Batch Edit File
This file allows the user to make changes to specific lines as specified by the filter. This is a csv file file that has at least the following three columns. 

- **filter**
- **x_hdwchange**: x is the scenario number
- **x_spdchange**: x is the scenario number

Additional headway and speed pairs can be added for more scenarios.

It is added by adding in the appropriate module in XTMF and then selecting the file location.
### Time Periods
The time periods that will be generated by Full Network Set Generator. Each module added here will be a Time Period Module. 
#### Time Period Module Start Time
This is the start time of the time period. Use the hh:mm format
#### Time Period Module End Time
The time period ends the millisecond before the time specified here. Use the hh:mm format
#### Time Period Uncleaned Scenario Number
The scenario number to use for the uncleaned scenario in this time period
#### Time Period Uncleaned Scenario Description
The scenario description to use for the uncleaned scenario in this time period
#### Time Period Cleaned Scenario Number
The scenario number to use for the cleaned scenario in this time period
#### Time Period Cleaned Scenario Description
The scenario description to use for the cleaned scenario in this time period

> The difference between cleaned and uncleaned scenario lies mostly with the removal of cosmetic nodes and links in the cleaned version. For more information see the documentation referred to in the following link: [Full Network Set Generator Page](https://tmg.utoronto.ca/doc/1.5/gtamodel/full_network_generator.html)

#### Transit Aggregation Selection Table
A csv file is added here using the appropriate XTMF module. 

The csv file contains the transit lines and how the headway is to be calculated for each line if a Transit Service Table file is used for that line. It contains two columns

- **emme_id**: this is the id of the transit line in Emme
- **agg_type**: this is the headway calculation method. It can either be 'naive' or 'average'

> For more details on the calculation of headways, see above in the section "Aggregation Type Selection"

#### Transit Alternative Table
A csv file is added here using the appropriate XTMF module and then selecting the file location. The csv file contains the following columns

- **emme_id**: This is the id of the transit line in Emme. Transit lines that do not have an entry in the Transit Service Table need to have an entry here or they will not be present in the time periods that will be generated. 
- **xxxx_hdw**: This is the headway column corresponding to the time period starting with "xxxx" in a "hhmm" format. For example: The "0600_hdw" column will contain the headways for the time period starting at 6:00 AM. 
-**xxxx_spd**: This is the speed column corresponding to the time period starting with "xxxx" in a "hhmm" format. For example: The "0600_spd" column will contain the speeds for the time period starting at 6:00 AM. 

Additional **xxxx_hdw** and **xxxx_spd** columns should be specified for all time periods that need to be modified.

#### Transit Service Table
A csv file is added here using the appropriate XTMF module and then selecting the file location.
This file contains trip starts and trip ends for transit lines. It uses a csv file format with the following three columns

- **emme_id**: This is the id of the transit line in Emme
- **trip_depart**: the departure time of the trip from the initial stop in a hh:mm format
- **trip_arrive**: the arrival time of the trip to the final destination in a hh:mm format










