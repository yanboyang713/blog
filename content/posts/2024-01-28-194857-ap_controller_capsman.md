---
title: "AP Controller (CAPsMAN)"
draft: false
---

## Overview {#overview}

CAPsMAN allows applying wireless settings to multiple MikroTik AP devices from a central configuration interface.

More specifically, the Controlled Access Point system Manager (CAPsMAN) allows centralization of wireless network management and if necessary, data processing. When using the CAPsMAN feature, the network will consist of a number of 'Controlled Access Points' (CAP) that provide wireless connectivity and a 'system Manager' (CAPsMAN) that manages the configuration of the APs, it also takes care of client authentication and optionally, data forwarding.

When a CAP is controlled by CAPsMAN it only requires the minimum configuration required to allow it to establish a connection with CAPsMAN. Functions that were conventionally executed by an AP (like access control, client authentication) are now executed by CAPsMAN. The CAP device now only has to provide the wireless link layer encryption/decryption.

Depending on the configuration, data is either forwarded to CAPsMAN for centralized processing (default) or forwarded locally at the CAP itself (local forwarding mode).

CAPsMAN stands for Controlled Access Point System Manager. It is a feature in MikroTik RouterOS that allows you to centralize the management and configuration of multiple MikroTik wireless access points (APs) from a single point, known as the CAPsMAN controller.

With CAPsMAN, you can configure and manage wireless networks more efficiently, especially in scenarios where you have multiple access points spread across an area. The CAPsMAN controller can handle tasks such as channel assignment, transmit power control, security settings, and more for all connected access points. This centralized approach simplifies the deployment and maintenance of wireless networks.

Here's a brief overview of how CAPsMAN works:

1.  CAPsMAN Controller: This is the central point that manages and controls one or more CAPs (Controlled Access Points). The controller is typically a MikroTik router or access point.
2.  CAPs (Controlled Access Points): These are the wireless devices that are managed by the CAPsMAN controller. Each CAP connects to the controller, and the controller takes care of configuring and controlling the settings of each CAP.
3.  Centralized Configuration: Instead of configuring each individual access point separately, you can define configurations on the CAPsMAN controller, and it will push these settings to the connected CAPs.
4.  Simplified Management: CAPsMAN provides a centralized point for monitoring and managing the entire wireless network. Changes to configurations, such as adding a new SSID or updating security settings, can be done on the CAPsMAN controller and applied to all connected CAPs simultaneously.

This setup is particularly useful in scenarios like large office buildings, hotels, or public spaces where multiple access points are needed for adequate coverage. Instead of manually configuring each access point, administrators can use CAPsMAN to streamline the management process.


## Reference List {#reference-list}

1.  <https://help.mikrotik.com/docs/pages/viewpage.action?pageId=1409149>
