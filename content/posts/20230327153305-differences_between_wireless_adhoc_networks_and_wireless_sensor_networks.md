---
title: "Differences between Wireless Adhoc Networks and Wireless Sensor Networks"
draft: false
---

| [Wireless Adhoc Network]({{< relref "20230324135753-wireless_adhoc_network.md" >}}) | [wireless sensor network (WSN)]({{< relref "20230327152952-wireless_sensor_network_wsn.md" >}})                   |
|-------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------|
| The medium used in wireless ad-hoc networks is radio waves.                         | The medium used in wireless sensor networks is radio waves, infrared, and optical media.                          |
| Application independent network is used.                                            | The application-dependent network is used.                                                                        |
| Hop-to-Hop routing takes place.                                                     | Query-based (data-centric routing) or location-based routing takes place.                                         |
| It is heterogeneous in type.                                                        | It is homogeneous in type.                                                                                        |
| The traffic pattern is point-to-point.                                              | The traffic pattern is any-to-any, many-to-one, many-to-few, and one-to-many.                                     |
| Wireless router is used as an inter-connecting device.                              | Application level gateway is used as an interconnecting device.                                                   |
| The data rate is high.                                                              | The data rate is low.                                                                                             |
| Supports common services.                                                           | Supports specific applications.                                                                                   |
| Traffic triggering depends on application needs.                                    | Triggered by sensing events.                                                                                      |
| IP address is used for addressing.                                                  | Local unique MAC address or spatial IP is used for addressing.                                                    |
| Network Type Peer-to-Peer                                                           | Network type Hierarchical or Mesh                                                                                 |
| Nodes Can be any wireless device                                                    | Nodes Limited to sensor nodes                                                                                     |
| Communication Range Variable depends on node placement                              | Communication Range Limited by the sensor node’s transmission power                                               |
| Communication Range Standard network protocols (TCP/IP)                             | Communication Range Customized protocols for efficient data transfer and low energy consumption                   |
| Data Type General data (voice, video, files, etc.)                                  | Data Type Sensor data (temperature, humidity, light, etc.)                                                        |
| Power Consumption Can be high due to constant communication                         | Power Consumption Designed to minimize energy consumption to extend network lifetime                              |
| Security Security protocols can be implemented                                      | Security Security protocols are critical as sensor data can be sensitive                                          |
| Applications General wireless communication                                         | Applications  Environmental monitoring, industrial automation, home automation, etc.                              |
| Deployment Can be deployed in any environment                                       | Deployment Typically deployed in remote or hard-to-reach locations, such as forests, oceans, or industrial sites. |


## Reference List {#reference-list}

1.  <https://www.geeksforgeeks.org/differences-between-wireless-adhoc-network-and-wireless-sensor-network/>
