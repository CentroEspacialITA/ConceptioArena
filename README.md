# conceptio_arena
![MainBranchConceptioBridges](https://github.com/ConceptioLab/conceptio_bridges/actions/workflows/docker-image.yml/badge.svg?branch=main)
![MainBranchConceptioCore](https://github.com/ConceptioLab/conceptio_core/actions/workflows/docker-image.yml/badge.svg?branch=main)

#### A multi-domain virtual arena that empowers Air Domain Study projects

## Overview
conceptio_arena is a virtual environment for the Virtual Demonstrator project. It consists of a world visualization and simulation tool (VR-Forces), a data bus (ROS2 + MQTT broker) and a MBSE-capable tool (Capella).

## Joining the arena
### MQTT entities
For MQTT entities, it is necessary to publish an ArenaHeartbeat message directly to the MQTT broker. 

To "see" the arena, simply subscribe to all units' heartbeats using ```+``` wildcards: ```conceptio/unit/+/+/+/heartbeat```. To see the position, subscribe to ```conceptio/unit/+/+/+/data/kinematics```

### ROS2 entities
#### Local ROS2
If you are integrating entities into the same local network as the arena, simply publish to ROS2 topics. Mqtt_mirror will listen to ROS2 topics and broadcast them to the MQTT broker, and vice-versa. 

#### Remote ROS2 
As ROS2 does discovery with UDP multicasting, communicating with ROS2 entities across the internet is troublesome. Instead of using a VPN (which may require extra permissions to run/setup), remote ROS2 entities can use the [mqtt_client](https://github.com/ika-rwth-aachen/mqtt_client) bridge. 

The bridge allows ROS2 topics to be mapped to MQTT topics at the broker and vice-versa. The downside is that you must know all topic names when starting the node; mqtt_client does not allow you to subscribe to all topics using the ```#``` wildcard, nor it allows you to do late subscriptions. To deal with that, you can use our [forked version of mqtt_client](https://github.com/mvccogo/mqtt_client/) that has new services (NewRos2ToMqtt and NewMqttToRos2) for dynamic subscription (a pull request has been opened). Once you have all MQTT <-> ROS2 mappings, just follow the MQTT topic structure and instructions to join and "see" the arena. 


## API/Topics specification
- PREFIX1: ```conceptio```
- PREFIX2:
    - ```unit```: information related to units
    - ```arena```: information related to the arena
    - ```terrain```: information related to the terrain
    - Note: PREFIX3, PREFIX4, ROOT and SUFFIX1 are only applicable to ```unit```.
- PREFIX3: name of the unit domain
    - ```air```
    - ```ground```
    - ```maritime```
    - ```space```
    - ```cyberspace```
- PREFIX4: entity scope (digital triplets)
    - ```real```
    - ```virtual```
    - ```model```
- ROOT: unique name for the unit
    - Topics:
    - ```heartbeat``` is a topic inside ROOT that publishes the entities heartbeats.
- SUFFIX1: ```data```
    - Topics:
    - ```kinematics```

Topic naming structure for units: **PREFIX1/PREFIX2/PREFIX3/PREFIX4/ROOT/SUFFIX1**

Examples:

```conceptio/unit/air/real/dji_1/heartbeat```: contains heartbeat information for the ```dji_1``` unit.

```conceptio/unit/air/real/dji_1/data/kinematics```: contains kinematic information for the ```dji_1``` unit.

For the ```arena``` prefix, there is only one subtopic: ```entities```, where information about connected entities is shared. This is the topic that remote ROS2 entities need to subscribe, as for every new entity new subscriptions need to be set up using the mqtt_client services. 

The MQTT topic structure follows that of the ROS2 due to the mqtt_mirror node from the conceptio_bridges package, that replicates known topics and serializes them into ROS2 topics and messages.



### Message definitions
Full list [here](https://github.com/ConceptioLab/conceptio_interfaces/tree/main/msg)


#### conceptio/arena/entities
- Type: conceptio_interfaces/ArenaEntities

MQTT topic that periodically shares the updated list of arena entities. ROS2 remote entities need this information in order to set up new bridge mappings. 


#### ROOT/heartbeat
- Type: conceptio_interfaces/ArenaHeartbeat

Mandatory message that periodically sends information about an arena entity.

#### ROOT/data/kinematics
- Type: conceptio_interfaces/ArenaKinematics

Message that updates a given entity position and rotation (x, y, z, yaw, pitch, roll). 


