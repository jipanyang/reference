# Dial out mode for gNMI telemetry service

**Date**: June 19th, 2018

**Version**: 0.1.0

## Background

In dial out mode of telemetry service, the target initiates connections towards pre-configured collectors then streams telemetry data. The exact list of collectors, path set, and various connection related variables are configured via channels like gNMI set RPC or  netconf or even CLI.

The typical use cases for dial out mode of telemetry service:

* Firewall/NAT service sits between network elements and telemetry collectors, such that inbound connections for telemetry to network elements are undesired or not possible.
* The collection infrastructure does not know the set of targets that it is to gather data from, and rather these clients dial in.
Or the collection infrastructure prefers to work in stateless mode and shed the complexity of maintaining telemetry configuration of each and every network element to another configuration system.


## gNMIDialOut service
gNMIDialout service is defined for telemetry in dialout mode. It has one streaming RPC: Publish. The message from network element to collector reuses SubscribeResponse from gNMI base spec. While the PublishResponse message serves the purpose of acknowledgement, it is optional and skipped by default.

```
// gNMIDialOut defines a service which is used by a target system (typically a
// network element) to initiate connections to one or more collectors. The server
// is implemented at the collector, such that the target can initiate connections
// to the collector, based on a set of telemetry subscriptions.
service gNMIDialOut {
  // Publish allows the target to send telemetry updates (in the form of
  // SubscribeResponse messaages, which have the same semantics as in the
  // gNMI Subscribe RPC, to a client. The client may optionally return the
  // PublishResponse message in response to the dial-out connection from the
  // target as acknowledgement to the SubscribeResponse message
  //
  // The configuration of subscriptions associated with the publish RPC may
  // be through the OpenConfig telemetry configuration and operational state
  // model:
  // https://github.com/openconfig/public/blob/master/release/models/telemetry/openconfig-telemetry.yang
  rpc Publish(stream SubscribeResponse) returns (stream PublishResponse);
}

message PublishResponse {
  // A string identifying the client to the target.
  // The meaning and exact format of client_id is specified by the configuration
  // for dial out telemetry service.
  string client_id = 1;
  // Timestamp in nanoseconds since Epoch.
  int64 timestamp = 2;
  // Prefix used for paths in the message.
  Path prefix = 3;
  // An alias for the path specified in the prefix field.
  // Reference: gNMI Specification Section 2.4.2
  string alias = 4;
  repeated Path path = 5;     // Paths for which the notifications have been received.
}
```

## Configurations on target for dial out mode

The gNMIDialOut service defined within this document is assumed to work with [openconfig-telemetry.yang](https://github.com/openconfig/public/blob/master/release/models/telemetry/openconfig-telemetry.yang) as to the configurations on target, but can be used with any configuration model with the following key elements:

1.  Destination group: A destination group may contain one or more telemetry destinations. The network element will initiate an outbound connection for telemetry towards the destination.


2.  Subscription: A telemetry subscription consists of a set of collection destinations, stream attributes, and associated paths to state information in the model.

## Contributors
 *
 *