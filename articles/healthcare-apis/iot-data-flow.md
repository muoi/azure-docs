---
title: 'Concepts: Data flow in IoT Connector (preview) feature of Azure API for FHIR'
description: Understand IoT Connector's data flow. IoT Connector ingests, normalizes, groups, transforms, and persists IoMT data to Azure API for FHIR.
services: healthcare-apis
author: ms-puneet-nagpal
ms.service: healthcare-apis
ms.subservice: iomt
ms.topic: conceptual
ms.date: 05/13/2020
ms.author: punagpal
---

# IoT Connector (preview) data flow

This article provides an overview of data flow in IoT Connector. You'll learn about different data processing stages within IoT Connector that transform device data into FHIR-based [Observation](https://www.hl7.org/fhir/observation.html) resources.

![IoT Connector data flow](media/concepts-iot-data-flow/iot-connector-data-flow.png)

Diagram above shows different data flow stages within IoT Connector. 

## Ingest
Ingest is the first stage where device data is received into IoT Connector. The ingestion endpoint for device data is hosted on an [Azure Event Hub](https://docs.microsoft.com/azure/event-hubs/). Azure Event Hub platform supports high scale and throughput with ability to receive and process millions of messages per second. It also enables IoT Connector to consume messages asynchronously, removing the need for devices to wait while device data gets processed.

> [!NOTE]
> JSON is the only supported format at this time for device data.

## Normalize
Normalize is the next stage where device data is retrieved from the above Azure Event Hub and processed using device mapping templates. This mapping process results in transforming device data into a normalized schema. 

The normalization process not only simplifies data processing at later stages but also provides the ability to project one input message into multiple normalized messages. For instance, a device could send multiple vital signs for body temperature, pulse rate, blood pressure, and respiration rate in a single message. This input message would create four separate FHIR resources. Each resource would represent different vital sign, with the input message projected into four different normalized messages.

## Group
Group is the next stage where the normalized messages available from the previous stage are grouped using three different parameters: device identity, measurement type, and time period.

Device identity and measurement type grouping enable use of [SampledData](https://www.hl7.org/fhir/datatypes.html#SampledData) measurement type. This type provides a concise way to represent a time-based series of measurements from a device in FHIR. And time period controls the latency at which Observation resources generated by IoT Connector are written to Azure API for FHIR.

> [!NOTE]
> The time period value is defaulted to 15 minutes and cannot be configured for preview.

## Transform
In the Transform stage, grouped-normalized messages are processed through FHIR mapping templates. Messages matching a template type get transformed into FHIR-based Observation resources as specified through the mapping.

At this point, [Device](https://www.hl7.org/fhir/device.html) resource, along with its associated [Patient](https://www.hl7.org/fhir/patient.html) resource, is also retrieved from the FHIR server using the device identifier present in the message. These resources are added as a reference to the Observation resource being created.

> [!NOTE]
> All identity look ups are cached once resolved to decrease load on the FHIR server. If you plan on reusing devices with multiple patients it is advised you create a virtual device resource that is specific to the patient and send virtual device identifier in the message payload. The virtual device can be linked to the actual device resource as a parent.

If no Device resource for a given device identifier exists in the FHIR server, the outcome depends upon the value of `Resolution Type` set at the time of creation. When set to `Lookup`, the specific message is ignored, and the pipeline will continue to process other incoming messages. If set to `Create`, IoT Connector will create a bare-bones Device and Patient resources on the FHIR server.  

## Persist
Once the Observation FHIR resource is generated in the Transform stage, resource is saved into Azure API for FHIR. If the FHIR resource is new, it will be created on the server. If the FHIR resource already existed, it will get updated.

## Next steps

Click below next step to learn how to create device and FHIR mapping templates.

>[!div class="nextstepaction"]
>[IoT Connector mapping templates](iot-mapping-templates.md)


FHIR is the registered trademark of HL7 and is used with the permission of HL7.
