
1. [CDBUS Protocol](#cdbus-protocol)
2. [CDBUS IP Core](#cdbus-ip-core)
3. [Ready To Use Devices](#ready-to-use-devices)
4. [Code Examples](#code-examples)
5. [License](#license)


## CDBUS Protocol

CDBUS is a protocol for Asynchronous Serial Communication,
it has a 3-byte header: `[src_addr, dst_addr, data_len]`, then user data, and finally 2 bytes of checksum.

It's suitable for one-to-one communication, e.g. UART or RS232.
In this case, the address for each side are usually carefully selected and fixed,
e.g: `[0x55, 0xaa, data_len, ...]`, and the backward is: `[0xaa, 0x55, data_len, ...]`.

The CDBUS protocol is more valuable for bus communication, e.g. RS485 or Single Line UART.
In this case:

* It introduces an arbitration mechanism that automatically avoids conflicts like the CAN bus.
* Support dual baud rate, provide high speed communication, maximum rate ≥ 10 Mbps.
* Support broadcast. (set `dst_addr` to `255`)
* Max payload data size is 253 byte. (you can increase it to 255 byte, but not recommended)
* Hardware packing, unpacking, verification and filtering, save your time and CPU usage.
* Backward compatible with traditional RS485 hardware. (still retains arbitration function)

The protocol example timing, include only one byte user data:  
(How long to enter idle and how long to allow sending can be set.)

<img alt="protocol" src="img/protocol.svg" width="100%">

Arbitration example:

<img alt="arbitration" src="img/arbitration.svg" width="75%">

The idea of CDBUS was first designed and implemented by me in 2009.


## CDBUS IP Core

Source code and more details:
 - https://github.com/dukelec/cdbus_ip
 - https://github.com/dukelec/cdbus_doc

### Block Diagram
<img alt="block_diagram" src="img/block_diagram.svg" width="100%">


## Ready To Use Devices

The CDCTL controller family uses the CDBUS IP Core, which provide SPI, I<sup>2</sup>C and PCIe peripheral interfaces.  
E.g. The tiny CDCTL-Bx module support both SPI and I<sup>2</sup>C interfaces:  
<img alt="cdctl_bx" src="img/cdctl_bx.jpg" width="80%">
<img alt="cdctl_bx" src="img/cdctl_bx_sch.svg" width="80%">

For more information, visit: http://dukelec.com


## Code Examples
 
```python
    # Configuration
    
    write(REG_SETTING, [0x01])                  # Enable push-pull output
    
    
    # TX
    
    write(REG_TX, [0x0c, 0x0d, 0x01, 0xcd])     # Write frame without CRC
    while (read(REG_INT_FLAG)[0] & 0x10) == 0:  # Make sure we can successfully switch to the next page
        pass
    write(dut, REG_TX_CTRL, [0x02])             # Trigger send by switching TX page
    
    
    # RX
    
    while (read(REG_INT_FLAG)[0] & 0x02) == 0:  # Wait for RX page ready
        pass
    header = read(REG_TX, len = 3)
    data = read(REG_TX, len = header[2])
    write(dut, REG_RX_CTRL, [0x02])             # Ack read by switching RX page

```


## License
```
This Source Code Form is subject to the terms of the Mozilla
Public License, v. 2.0. If a copy of the MPL was not distributed
with this file, You can obtain one at https://mozilla.org/MPL/2.0/.
Notice: The scope granted to MPL excludes the ASIC industry.
The CDBUS protocol is royalty-free for everyone except chip manufacturers.

Copyright (c) 2017 DUKELEC, All rights reserved.
```

