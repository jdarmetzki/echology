Introduction
============

The InfoPoller tool provides the following functionality:
* *polling* at fixed intervals for environmental properties such as humidity, temperature, illumination, CO2/VOC, etc.,
* act as a *Proxy* for ECHONET Lite commands.

To run the program:
```
java -jar dist/InfoPoller.jar
```

Running the Program
-------------------
To run the program without any special options use:
```
java -jar dist/InfoPoller.jar
```
This will start both the polling and the proxy functionality. Bellow is the usage string printed at the start of the program:

```
USAGE: [--no-polling] [--no-proxy] [-t timeinterval] [-i IP] [-p pollingport] [-pp proxyport] [-f filterstring]
  filterstring is a comma-separated list of data type
  filterstring example: TMP,VOC,C02,HMDT etc.
```

The options available are as follows:

`--no-polling` disable the polling functionality.

`--no-proxy` disable the proxy functionality. The presence of both `--no-polling` and `--no-proxy` terminates the program at startup.

`-t timeinterval` specify the time interval (in seconds) for the polling functionality.

`-i IP` specify the IP address (and by extension the network interface) to use. If not specified, automatically select an IP address bound to the machine which is not peer-to-peer and can perform IP multicast. 

`-p pollingport` specify the port used to listen for clients interested in the polled data (default: 2345)

`-pp proxyport` specify the port used to listen for clients that want to execute ECHONET Lite commands (default: 3361)

`f filterstrnig` specify the type of data that will be *excluded from* the polled data. For example, specifying `-f HMDT` will exclude humidity data. Current known data types are `TMPR, HMDT, VOC, CO2, LUX, LGHT, PRSN`, standing in for temperature, humidity, voc, co2, illumination, light status (on/off) and human presence.

Connecting Clients
------------------

To confirm the correct operation of the program, use programs such as `nc` and `telnet` to connect to either the polling or the proxying endpoint. 

### Polling Endpoint 


For example, to connect to the polling endpoint on the same machine with `nc`, use:
```
nc localhost 2345
```

After a brief moment, you should see data streaming in, like:
```
10.0.xxx.aaa:001101:0xE0,TMPR,26.1
10.0.xxx.aaa:001201:0xE0,HMDT,55
10.0.xxx.bbb:001101:0xE0,TMPR,25.2
10.0.xxx.bbb:001102:0xE0,TMPR,49.8
...
```
The comma-separated data format is:
```
DATA_SOURCE,DATA_TYPE,DATA_VALUE
```

with `DATA_SOURCE` represented as:
```
IP ADDRESS:EOJ:PROPERTY_CODE
```

In the first line of the above example, the data source has an IP address of 10.0.xxx.aaa, an EOJ of 001101, and a property code of 0xE0. The type of the data is `TMPR`, implying temperature, and the value is `26.1`, presented as a literal string, in degrees Celsius.

### Proxy Endpoint

To connect to the proxy endpoint on the same machine with `telnet`, use:
```
telnet localhost 3361
```

Here is what a proxy session looks like:
```
telnet localhost 3361
Trying ::1...
Connected to localhost.
Escape character is '^]'.

224.0.23.0:001101:0xE0
OK,10.0.0.aaa:001101:0xE0,0x00FD
OK,10.0.0.bbb:001101:0xE0,0x0105

```

After `telnet` (or `nc`) has established a connection, nothing appears on the screen; the proxy endpoint is waiting for commands. In the above example, the command issued was `224.0.23.0:001101:0xE0` which specifies a `Get` command to all the ECHONET Lite nodes on the network (multicast IP address `224.0.23.0`), that have an ECHONET Lite object of `001101` (a temperature sensor), for property `0xE0`, which specifies the current temperature data of the sensor. As a result to this query, two responses came back, along with the binary data for these properties.

`Get` commands have the following structure:
```
IP_ADDRESS:EOJ:PROPERTY_CODE
```

The responses have the following structure:
```
[OK|NG],IP_ADDRESS:EOJ:PROPERTY_CODE,DATA_VALUE]
```
Responses starting with `OK` signify success, with the data value reporting back the actual value of the property specified. Responses starting with `NG` signify an error. An example resulting in an error follows:

```
10.0.0.aaa:001101:0xEE
NG,10.0.0.aaa:001101:0xEE,0x00
```
The above `Get` request resulted in an error, because the property `0xEE` is not present at the requested ECHONET Lite object.

Requests for non-existent objects/nodes will timeout after 5 seconds, and result in an `NG` error without further details:
```
10.0.0.ccc:001101:0xE0
NG
```

The `Set` commands have the following structure:
```
IP_ADDRESS:EOJ:PROPERTY_CODE,DATA_VALUE_TO_SET
```

`Set` commands follow the same syntax as `Get` commands, but they also specify one additional argument after a comma, with the value that the specified property must be set at.

A successful `Set` invocation looks like this:
```
10.0.0.ddd:000303:0xBF,ABCDEF
OK,10.0.0.ddd:000303:0xBF,0x00
```

A `Set` invocation which resulted in a failure looks like this:
```
10.0.0.eee:001101:0x80,0x30
NG,10.0.0.eee:001101:0x80,0x30
```

Building from Source
--------------------
### Software Prerequisites:
* Java (>= 1.8)
* Ant

or, alternatively:
* NetBeans (>=8.2)

### Building ProtoLite
TODO
* clone from github: `git clone https://github.com/s-marios/ProtoLite.git`
* build with `ant`

### Building InfoPoller
* clone from tangit: `git clone http://tangit.jaist.ac.jp/git/echology.git`
* add the `ProtoLite` dependency
* build with `ant`