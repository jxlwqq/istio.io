---
title: Protocol Selection
description: Information on how to specify protocols.
weight: 10
keywords: [protocol,protocol sniffing,protocol selection,protocol detection]
aliases:
  - /help/ops/traffic-management/protocol-selection
  - /help/ops/protocol-selection
  - /help/tasks/traffic-management/protocol-selection
  - /docs/ops/traffic-management/protocol-selection
owner: istio/wg-networking-maintainers
test: n/a
---

Istio supports proxying any TCP traffic. This includes HTTP, HTTPS, gRPC, as well as raw TCP protocols.
In order to provide additional capabilities, such as routing and rich metrics, the protocol must be determined. This can be done automatically or explicitly specified.

Non-TCP based protocols, such as UDP, are not proxied. These protocols will continue to function as normal, without
any interception by the Istio proxy but cannot be used in proxy-only components such as ingress or egress gateways.

## Automatic protocol selection

Istio can automatically detect HTTP and HTTP/2 traffic. If the protocol cannot automatically be determined, traffic will be treated as plain TCP traffic.

{{< tip >}}
Server First protocols, such as MySQL, are incompatible with automatic protocol selection. See [Server first protocols](/docs/ops/deployment/requirements#server-first-protocols) for more information.
{{< /tip >}}

## Explicit protocol selection

Protocols can be specified manually in the Service definition.

This can be configured in two ways:

- By the name of the port: `name: <protocol>[-<suffix>]`.
- In Kubernetes 1.18+, by the `appProtocol` field: `appProtocol: <protocol>`.

Note that behavior at the Gateway differs in some cases as the gateway can terminate TLS and the protocol can be negotiated.

The following protocols are supported:

| Protocol                              | Sidecar Purpose                                                                                                                                                         | Gateway Purpose                                                                                                                                                         |
| ------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `http`                                | Plaintext HTTP/1.1 traffic                                                                                                                                              | Plaintext HTTP (1.1 or 2) traffic                                                                                                                                       |
| `http2`                               | Plaintext HTTP/2 traffic                                                                                                                                                | Plaintext HTTP (1.1 or 2) traffic                                                                                                                                       |
| `https`                               | TLS Encrypted data. Because the Sidecar does not decrypt TLS traffic, this is the same as `tls`                                                                         | TLS Encrypted HTTP (1.1 or 2) traffic                                                                                                                                   |
| `tcp`                                 | Opaque TCP data stream                                                                                                                                                  | Opaque TCP data stream                                                                                                                                                  |
| `tls`                                 | TLS Encrypted data                                                                                                                                                      | TLS Encrypted data                                                                                                                                                      |
| `grpc`                                | Same as `http2`                                                                                                                                                         | Same as `http2`                                                                                                                                                         |  |
| `grpc-web`, `mongo`, `mysql`, `redis` | Experimental application protocol support. To enable them, configure the corresponding Pilot [environment variables](/docs/reference/commands/pilot-discovery/#envvars) | Experimental application protocol support. To enable them, configure the corresponding Pilot [environment variables](/docs/reference/commands/pilot-discovery/#envvars) |

Below is an example of a Service that defines a `https` port by `appProtocol` and an `http` port by name:

{{< text yaml >}}
kind: Service
metadata:
  name: myservice
spec:
  ports:
  - number: 3306
    name: database
    appProtocol: https
  - number: 80
    name: http-web
{{< /text >}}
