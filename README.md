# PcapStream

This is pcap(like) stream writer. It implements Stream interface and it is meant to be used as layer between application - like HttpClient - and actual transport stream. It will write all data read and writtent in format readable by tcpdump and Wireshark so it is easier to debug binary protocols or data wrappet in encrypted stream.

For HttpCLient the base use case is like 
```c#
  using var capture = new NetCapture($"{host}.pcap");

  var handler = new SocketsHttpHandler();
  handler.PlaintextStreamFilter = (context, token) => 
  { 
    return new ValueTask<Stream>(capture.AddStream(context.PlaintextStream)); 
  };
  using (HttpClient client = new HttpClient(handler))
  {
      ....
  }
```

It will try to detect transport paramaters, add IP and TCP headersw and it will write each Read or Write as a "packet". If transport IPEndpoints cannot be determined, it will use `127.0.0.1` addresses and generated TCP ports. IPEndpoiunt can also be handed explicitly to `AddStream` method. 
To make decoding easier, PcapStream will add 8400 to detected destination port. With that, connection to port 443 will be shown as 8843 in capture file. Without it Wireshark will have problems decoding the stream. 8843 is not used by well known protocos and it easy to set a default rule and decode 8843 automaticaly as HTTP. (or what ever). This behavir can be changed by passing `portOffset: 0` to `AddStream`.

# TODO list and BUGS:
- IPv6 support. 
  - IPv4 mapped addresses from dual mode socket will be shown as IPv4
  - Pure IPv6 will be convereted loopback IPV4 for now (as unknown/unsupported EndPoint)
- TCP Ack needs more work
  - This works now for simple and reasonably small request/responces
  - For large unidirectional streems we will need to inject extra ACK packets to keep Wireshark calulations happy
- no checksums
  - neither IP headers nor TCP has checksum fields filled
  - that seems OK at the moment as Wiresark assuems HW support and does not complain much
- needs more testing
  
