<!-- Breadcrumbs -->
< [Lab.SCHC FullSDK Documentation](/#) /
<!-- /Breadcrumbs -->

*On this page:*

<!-- TOC -->

- [FullSDK Reference Manual](#fullsdk-reference-manual)
    - [Preface](#preface)
        - [Intended audience](#intended-audience)
        - [Related documents](#related-documents)
    - [About lab.SCHC FullSDK](#about-labschc-fullsdk)
    - [Public API functions](#public-api-functions)
        - [Management interface functions](#management-interface-functions)
            - [Initialization](#initialization)
                - [List of Callbacks](#list-of-callbacks)
                - [Memory block pointer](#memory-block-pointer)
                - [Maximum sizes](#maximum-sizes)
            - [Synchronization bootstrap](#synchronization-bootstrap)
            - [Processing](#processing)
            - [Polling](#polling)
            - [Version](#version)
            - [Extension of the Management interface](#extension-of-the-management-interface)
                - [Header rules](#header-rules)
                - [Payload rules](#payload-rules)
                - [Fragmentation rules](#fragmentation-rules)
                - [Template parameters](#template-parameters)
        - [Datagram interface functions](#datagram-interface-functions)
            - [Initialization](#initialization)
            - [Socket creation](#socket-creation)
            - [Local port binding](#local-port-binding)
            - [Transmission](#transmission)
            - [Reception](#reception)
            - [Socket closing](#socket-closing)
        - [Network interface functions](#network-interface-functions)
            - [Initialization](#initialization)
            - [Configuration](#configuration)
            - [Transmission](#transmission)
            - [Reception](#reception)
        - [L2A interface functions](#l2a-interface-functions)
            - [Integration](#integration)
            - [Initialization](#initialization)
            - [L2 Technology](#l2-technology)
            - [Transmission](#transmission)
            - [MTU Size](#mtu-size)
            - [Device IID](#device-iid)
            - [Next Transmission Delay](#next-transmission-delay)
            - [L2 Processing](#l2-processing)
        - [Additional interfaces](#additional-interfaces)
            - [Statistics interface](#statistics-interface)
                - [Packet counter](#packet-counter)
                - [Fragment counter](#fragment-counter)
                - [ACK counter](#ack-counter)
                - [Reset](#reset)
            - [Advanced SCHC Configuration interface](#advanced-schc-configuration-interface)
                - [Change retransmission timer](#change-retransmission-timer)
                - [Change inactivity timer](#change-inactivity-timer)
                - [Polling](#polling)
                - [ACK-Always mode](#ack-always-mode)
                - [Maximum ACK requests](#maximum-ack-requests)

<!-- /TOC -->


# FullSDK Reference Manual

## Preface

### Intended audience

This manual is intended for integrators/developers. It presumes that its readers
have some familiarity with embedded software development, and with Internet of
Things (IoT) domain.

### Related documents

You may also find the following documents useful:

1. [FullSDK Concepts](/docs/concepts.md)
2. [SCHC: Generic Framework for Static Context Header Compression and
   Fragmentation](https://datatracker.ietf.org/doc/html/rfc8724)

## About lab.SCHC FullSDK

Lab.SCHC FullSDK was designed with the following characteristics in mind;
developers should consider them during the development process:

- Event-based library to facilitate the integration in different execution
  environments( bare-metal, RTOS).
- Provide well-defined interfaces and behaviour, so that testing stubs can be
  implemented.
- Application agnostic (CoAP, DLMS,...).
- Support any L2 stack.
- Modularity. Different layers or services may be enabled or disabled.
- Platform agnostic: must let the users (i.e. the developers using the SDK) use
  their usual tools and paradigms.
- Low-power modes support.
- Forbid dynamic memory allocation.


For more information, please refer to the [FullSDK
Concepts](/docs/concepts.md#how-to-use-the-labschc-fullsdk) page.

## Public API functions

The following header files are provided for the [SCHC
interfaces](/docs/concepts.md#interfaces), located in `full-sdk/core/`
case.

| File Name | API |
| --- | --- |
| `fullsdkmgt.h` | Management interface |
| `fullsdkextapi.h` | Management extension interface |
| `fullsdkdtg.h` | Datagram interface |
| `fullsdknet.h` | Network interface |
| `fullsdkl2a.h` | Level 2 adaptation (L2A) interface |

### Management interface functions

The header file for the Management interface is `fullsdkmgt.h`.

The management interface has two roles:

- Configure the lab.SCHC FullSDK stack.
- Provide the stack with resources required at execution time: processing time,
  memory and timers.

> **_Note_**: The management interface functions must be used to configure the
> stack before any functions of any interface are called.

#### Initialization

The first function that must be called is `mgt_initialize()`.

This function must be provided with:

- A list of callbacks
- A pointer to a memory block
- The maximum MTU size
- The maximum payload size

##### List of Callbacks

Each callback will be called for a specific event.

| Callback | Event |
| --- | --- |
| `mgt_processing_required` | Informs the application that one of the layers in the FullSDK stack has to perform some processing. The application must call the `mgt_process()` function as soon as possible but not from within the callback. |
| `mgt_connectivity_state` | Informs the application about any modification of the connectivity state. The application should not try to send a message until connectivity is available. |
| `mgt_start_timer` | Informs the application that a timer should be started. Timer period and ID are specified by the callback parameters. When the timer period expires, the `mgt_timer_timeout()` function must be called with the same ID used for the start timer. |
| `mgt_stop_timer` | Informs the application that a timer must be stopped. Timer ID is specified by the callback parameter. |
| `mgt_sync_bootstrap_result` (optional) | Informs the application about the result of the synchronization with the SCHC gateway. This callback is triggered when a response is received following a call to `mgt_sync_boostrap()`. Note that synchronization is an extended feature of lab.SCHC FullSDK. This callback can be omitted (i.e. set to NULL) when Sync is not used or when the SCHC gateway does not support it.

##### Memory block pointer

The initialization function must be provided with a pointer to a memory block allocated for the stack and the size of the memory block.

The application that integrates Lab.SCHC FullSDK stack has to allocate some memory (based on a formula, see below) from the volatile memory of the execution environment, and then provide it to the stack.

The formula is the following:

```
(4 * (max_mtu_size + max_app_payload_len + max_proto_size))
```

where: 

- `max_mtu_size` is the maximum MTU size of the L2 technology used;
- `max_app_payload_len` is the maximum payload length that the application expects to send and receive;
- `max_proto_size` is predefined as `MGT_PROTO_SIZE` in fullsdkmgt.h.

##### Maximum sizes

Finally, the initialization function must be completed with:

- The maximum MTU size of the L2 technology used;
- The maximum payload size that the application expects to send and receive.

#### Synchronization bootstrap

This function is used to synchronize the SDK with the SCHC gateway. It is optional, and only compatible with Actility's SCHC IPCore.

`mgt_sync_bootstrap()` must be provided with the following parameters:

- `retransmission_delay`: The delay (in ms) between each request retransmission.
- `max_retrans`: The maximum number of request retransmission before considering that the synchronization has failed.

#### Processing

This section refers to the FullSDK [paradigm](/docs/concepts.md#paradigm).

Every time a layer of the FullSDK stack needs to perform a processing as the result of some event, the `mgt_processing_required` callback is called.

It is then up to the application to call the `mgt_process()` function as soon as possible, but not from within the callback. In fact, a callback called by the lab.SCHC FullSDK **must not** call *any* lab.SCHC FullSDK function.

#### Polling

While using L2 technology having quasi-bidirectional links, which is the case with LoRaWAN in class A, polling packets must be send in order to trigger downlink packet reception.

Two functions are dedicated for that purpose:

- `mgt_enable_polling()` function must be called once after `mgt_initialize()` to activate implicit polling mechanism inside the lab.SCHC FullSDK stack. The aim being to retrieve the downlink packets, in particular when they are fragmented.
- `mgt_trigger_polling()` function must be called every time the application wants to trigger explicitly a polling frame. The aim being to generate a downlink possibility (fport: 0 with no payload for LoRaWAN, etc.).

These functions have no use for Class C and the Class B is not supported.

#### Version 

The `mgt_get_version()` function returns the version of the FullSDK library as a null-terminated string.

#### Extension of the Management interface

The header file for the extension of the Management interface is `fullsdkextapi.h`, located in `full-sdk/api/<api>`, depending on the application API.

The extension API provisions SCHC compression and fragmentation rules in binary format.

##### Header rules

> This function should be called only when the FullSDK library is compiled using
> dynamic rule loading.

The `mgt_provision_header_rules()` function provisions the header rules, and considers the following parameters:

- `bin_rules`: SCHC rules in binary format.
- `bin_rules_len`: The length of `bin_rules`, in bytes.
- `memory`: A memory area or buffer that can be used by the parser.
- `memory_len`: The length in bytes of the memory area.

##### Payload rules

The `mgt_provision_payload_rules()` function provisions payload pattern rules in a binary format. The following parameters must be provided:

- `bin_rules`: Payload pattern rules in a binary format.
- `bin_rules_len`: The length of `bin_rules`, in bytes.
- `memory`: A memory area or buffer that can be used by the parser.
- `memory_len`: The length of the memory area.

##### Fragmentation rules

The `mgt_provision_frag_profile()` function provisions the fragmentation rule for a single direction. In order to set a specific fragmentation profile for both directions (uplink and downlink), this function must be called twice with the selected rules.

The function must be provided with the following parameters:

- `buffer`: Fragmentation rule in a binary format.
- `size`: The length of the fragmentation rule, in bytes.

##### Template parameters

Template parameters can be set with `mgt_set_template_param()` using the following parameters:

- `index`: 0-based index corresponding to the template parameters to set.
- `value`: The value to associate to the index.
- `size`: The size of the given value in bytes.

Template parameters can also be retrieved using the `mgt_get_template_param()` function by providing the corresponding parameter `index` in the template.

Other functions related to templates include: 

- `mgt_get_nb_template_params()`: gives the number of configurable parameters in the provided template.
- `mgt_get_template_id()`: retrieves the template ID of the current template.
- `mgt_reset_template_params()`: clears the template parameters in the memory.

Refer to the `fullsdkextapi.h` header files for more details.

### Datagram interface functions

The header file for the Datagram interface is `fullsdkdtg.h`.

> **_NOTE_**: Use the functions of the Management interface to configure lab.SCHC FullSDK stack before using the Datagram interface.

The datagram interface allows the application to send and receive data packets to and from a remote application.

The interface is similar to the Berkeley datagram socket interface:

- **Sending packets** – If a data packet to send is larger than what Layer 2 can accept, the fragmentation mechanism is activated. This fragmentation is transparent to the remote application: it will receive the data packet all at once.
- **Receiving packets** – Fragmentation can also be activated in a receive operation, if the packet sent by the remote application is larger than what Layer 2 can accept.

#### Initialization 

The first function that the application must call is `dtg_initialize()` to initialize the the datagram interface before using it.

This function must be provided with the following parameters:

- Callbacks parameter: `transmission_result` will inform the application of the end of a send request (success or failure). `data_received` will inform the application when a data packet is received.
- Protocol parameter: Select an IPv6 or IPv4 network layer.

#### Socket creation

In order to be able to send or receive data packets, the application must now call the `dtg_socket()` function.

This function must be provided with the following parameters:

- `socket`: The socket for which transmission result is returned.
- `p_buffer`: A pointer to the buffer where the received packets will be written.
- `data_size`: The length of this buffer (in bytes).
- `dtg_sockaddr`: The address f the socket.
- `status`: The status of the tramission. No data is raised to the user in case of error.

`dtg_socket()` returns a socket structure that needs to be used as parameter for the following local port binding functions.

#### Local port binding

`dtg_socket()` returns a socket structure that can be used by `dtg_bind()` to bind the application to an IPv6/v4 address and a local port for data packet reception.

> If the application is not supposed to receive data packets from the remote application, it does not have to call `dtg_bind()`.

#### Transmission

A *send* operation is requested by calling the `dtg_sendto()` function on the corresponding socket structure.
The function immediately returns a status that informs the caller whether the request has been accepted or not.

The request can be refused, for example when there is an ongoing send operation, or when the connectivity is not available.

If the request has been accepted, the result of the send operation will be provided later, by a call to the `dtg_transmission_result` callback (which is provided at initialization by `dtg_initialize()`) with the corresponding socket as parameter.

A new request for the same socket will not be accepted as long as the `dtg_transmission_result` callback with the corresponding socket is called.

#### Reception

When a data packet is received, the `dtg_data_received` callback (which is provided at initialization by `dtg_initialize()`) is called.

Check the header file (`fullsdkdtg.h`) for information about flow control for reception.

#### Socket closing

When a socket is no longer used, it must be closed, by calling `dtg_close()`. Any ongoing fragmentation process that could exist when calling `dtg_close()` is then terminated.

### Network interface functions

The header file for the Network interface is `fullsdknet.h`.

> **_NOTE:_**: Use the functions of the Management interface to configure lab.SCHC FullSDK stack before using the Network interface.

The network interface allows the application to send and receive raw IPv6 packets to and from a remote application.

> In order to be able to use this interface, extension API functions (fullsdkextapi.h) must be formerly used to configure the network information.

- **Sending packets** – If a data packet to send is larger than what Layer 2 can accept, the fragmentation mechanism is activated. This fragmentation is transparent to the remote application: it will receive the data packet all at once.
- **Receiving packets** – Fragmentation can also be activated in a receive operation, if the packet sent by the remote application is larger than what Layer 2 can accept.

#### Initialization

The first function that the application must call is `net_initialize()` to be able to send or receive data packets.

This function must be provided with the following callback parameters:

- `transmission_result`: It will inform the application of the end of a send request (success or failure).
- `data_received`: It will inform the application when a data packet is received.

`net_initialize()` returns a status code indicating whether the initialization is successful or not.

#### Configuration

The configuration of the network interface highly depends on the application that uses it.

Configuration functions are provided in the extension API (`fullsdkextapi.h` header file) to meet the integrator's specific application needs.

#### Transmission

A *send* operation is requested by calling the `net_sendto()` function.
The function immediately returns a status that informs the caller whether the request has been accepted or not.

The request can be refused, for example when there is an ongoing send operation, or when the connectivity is not available.

If the request has been accepted, the result of the send operation will be provided later, by a call to the `net_transmission_result` callback (which is provided at initialization by `net_initialize()`).

A new request will not be accepted until the `net_transmission_result` callback is called.

#### Reception

When a data packet is received, the related callback declared via `net_initialize()`, is called.

Check the header file (`fullsdknet.h`) for information about flow control for reception.

### L2A interface functions

Lab.SCHC FullSDK provides a reference implementation of the L2A layer based on LoRaWAN technology.

The targets of the L2A reference implementation is currently  the ST NUCLEO-L476RG microcontroller board, with either the SX1276MB1MAS or SX1272MB2xAS LoRaWAN shields and running the Semtech LoRa stack, in classes A or C. Class B is not supported.

The header file for the L2A interface is `fullsdkl2a.h`.

The layer-two adaptation interface (also called L2A) is required to bind the lab.SCHC FullSDK with the L2 stack (LoRa, etc).

#### Integration

The L2A layer is usually provided by the integrator who has to implement the list of functions available in the `fullsdkl2a.h` header file.

Note that these functions are only called by the lab.SCHC FullSDK and must not be called directly from the application code.

#### Initialization

When the application calls the `mgt_initialize()` function (initialization by the Management interface), the `l2a_initialize()` function is implicitly called by lab.SCHC FullSDK.

This function can be used by the integrator to initialize the L2 stack and to store the callback functions provided as parameters:

- `l2a_processing_required`: To be called by the integrator to inform the lab.SCHC FullSDK when an asynchronous task needs to be handled by the layer. See `l2a_process()` function below for more details.
- `l2a_transmission_result`: To be called by the integrator to inform the lab.SCHC FullSDK when the uplink packet has been transmitted (with or without success) by the L2 stack.
- `l2a_data_received`: To be called by the integrator to inform the lab.SCHC FullSDK when a downlink packet is received from the L2 stack.
- `l2a_connectivity_lost`: To be called by the integrator to inform the lab.SCHC FullSDK when the L2 connectivity is lost.
- `l2a_connectivity_available`: To be called by the integrator to inform the lab.SCHC FullSDK when the L2 connectivity is available. For example, in LoRa, it corresponds to the Join Accept reception.

`l2a_initialize()` returns a status code indicating whether the initialization is successful or not.

#### L2 Technology

Lab.SCHC FullSDK calls the `l2a_get_technology()` function to get information on the L2 stack used by the integrator.

The following technology profiles are supported:

| Technology Profile | Description |
| --- | --- |
| `L2A_DEFAULT` | Recommended technology profile for LoRa with class C |
| `L2A_DEFAULT_CLASS_A` | Recommended technology profile for LoRa with class A |
| `L2A_LORA` | Technology profile described in RFC 9011 |
| `L2A_SIGFOX` | Technology profile for Sigfox |

#### Transmission

Lab.SCHC FullSDK calls the `l2a_send_data()` function to transmit a packet to the L2 stack.
This function requires two parameters:

- The buffer containing the packet to be transmitted
- The size of the packet

> The integrator should handle the transmission of the packet to the L2 stack.
> When the size of the packet is set to 1 and the value is 0x00 in hexadecimal, the integrator must send an empty frame (see Polling in the Management interface functions).

#### MTU Size

Lab.SCHC FullSDK calls the `l2a_get_mtu()` function to get information on the maximum packet size (in bytes) that can be transmitted by the L2 stack.

Lab.SCHC FullSDK calls this function from time to time because the MTU may change. For example, with a LoRa L2 stack, the MTU may change when ADR is enabled according to the radio conditions. It is up to the integrator to retrieve this value from the L2 stack.

#### Device IID

Lab.SCHC FullSDK calls the `l2a_get_dev_iid()` function when the IPv6 address of the device is derived from L2 stack information.

It is used for LoRaWAN technology and can be implemented according to the formula referenced in [RFC 9011 (section 5.3)](https://datatracker.ietf.org/doc/html/rfc9011#section-5.3).

For other technologies, it must be returned false.

#### Next Transmission Delay

Lab.SCHC FullSDK calls the `l2a_get_next_tx_delay()` function to get information on the next uplink transmission slot (in ms) according to the L2 stack type and its configuration.

The next uplink packet transmitted by Lab.SCHC FullSDK using `l2a_send_data()` (see above) will be based on this delay.

For example, with a LoRa L2 stack, there is duty cycle limitation to follow before emitting over the radio.

#### L2 Processing

Lab.SCHC FullSDK calls the `l2a_process()` function every time `l2a_processing_required` callback (see above) is called by the integrator.

This function is used to handle asynchronous events received from the L2 stack. Call the following callbacks inside `l2a_process()`:

| Direction | Callback | Description |
| Uplink | `l2a_transmission_result` | When the event corresponding to the end of the transmission of an uplink packet is received from the L2 stack. |
| Downlink | `l2a_data_received` | When the event corresponding to the reception of a downlink packet is received from the L2 stack. |

Handle in the same way any other asynchronous event (timer, etc.) that might need to be implemented by the integrator and can be processed here.

### Additional interfaces

Additional functionalities for the lab.SCHC FullSDK are declared in separate interfaces that enhance and extend the capabilities of the main interfaces and the SDK itself. These interfaces are designed to provide advanced features, improve usability, and offer greater flexibility in integrating with our SDK

#### Statistics interface

The header file for the Statistics interface is `fullsdkstats.h`.

This statistics interface is used to provide SCHC information related to the number of SCHC compressed packets and SCHC fragments exchanged in uplink and downlink directions by the lab.SCHC FullSDK stack.

##### Packet counter

- `stats_schc_packets_tx_counter()`: Returns the number of SCHC unfragmented packets that are transmitted by the library.
- `stats_schc_packets_rx_counter()`: Returns the number of SCHC unfragmented packets received by the library.

##### Fragment counter

- `stats_schc_fragments_tx_counter()`: Returns the number of SCHC fragments transmitted by the library.
- `stats_schc_fragments_rx_counter()`: Returns th enumber of SCHC fragments received by the library.

##### ACK counter

These functions do not concern the No-ACK mode.

- `stats_schc_ack_tx_counter()`: Returns the number of SCHC ACK transmitted by the library.
- `stats_schc_ack_rx_counter()`: Returns the number of SCHC ACK received by the library.

##### Reset

- `stats_reset()`: Resets all metrics to 0.

#### Advanced SCHC Configuration interface

The functions defined in the advanced SCHC configuration interface are used for tuning and customization. They will affect the way lab.SCHC FullSDK handle the applicative packets.

The header file for the SCHC Configuration interface is `fullsdkschc.h`.

This interface can be used to configure SCHC technology profile and fragmentation parameters.

##### Change retransmission timer

- `schc_set_retransmission_timer()`: Allows to modify the value of the retransmission timer used by the current profile. This function must be provided with the parameter `rt_exp_time` which value is a 32-bits integer.

##### Change inactivity timer

- `schc_set_inactivity_timer()`: Allows to modify the value of the inactivity timer used by the current profile. This function must be provided with the parameter `it_exp_time` which value is a 32-bits integer.

##### Polling

- `schc_set_polling_status()`: Allows to activate polling for downlink sessions as well as suspending uplink sessions in the meanwhile. This function must be provided with the following boolean parameters:
    - `enable`: `true` to activate polling.
    - `suspend_uplinks`: `true` to suspend uplink sessions.

##### ACK-Always mode

These functions are only used with ACK-Always downlink fragmentation mode.

- `schc_set_schc_ack_req_dn_opportunity()`: To set the new value of ACK_REQ_DN_OPPORTUNITY in the computation of the ACK retransmission timer expiration. This function must be provided with the parameter `ack_req_dn_opportunity` whose value is a 8-bits integer. 
- `schc_set_idle_polling_timer()`: To set the value of the idle polling timer that is used to create downlink opportunities on quasi-bidirectional links. This function must be provided with the parameter `time` whose value is a 32-bits integer.
- `schc_set_ack_retransmission_timer()`: To set the value of the ACK retransmission timer that is used to create downlink opportunities on quasi-bidirectional links. This function must be provided with the parameter `time` whose value is a 32-bits integer.

##### Maximum ACK requests

- `schc_set_max_ack_req()`: Allows to set the value of `MAX ACK REQUESTS` that indicates the number of time a sender will send an ACK before sending an ABORT message. This function must be provided with the parameter `max_ack_req` which value is a 8-bits integer.

