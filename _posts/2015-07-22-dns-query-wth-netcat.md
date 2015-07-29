---
layout: post
title: "DNS query with netcat"
date: 2015-07-22 17:00
---

First, set up tcpdump for traffic capture to file and display to stdout at the same time:

~~~
$ tcpdump -n -U -w - -i gprs0 | tee /tmp/test.cap | tcpdump -n -r -
~~~

Execute 'host' query towards google public DNS:

~~~
$ host -t a google-public-dns-a.google.com 8.8.8.8
Using domain server:
Name: 8.8.8.8
Address: 8.8.8.8#53
Aliases:

google-public-dns-a.google.com has address 8.8.8.8
~~~

tcpdump shows:

~~~
10:59:38.665865 IP 10.120.149.34.63989 > 8.8.8.8.53: 46669+ A? google-public-dns-a.google.com. (48)
10:59:40.886812 IP 8.8.8.8.53 > 10.120.149.34.63989: 46669 1/0/0 A 8.8.8.8 (64)
~~~

We can dig into the request packet with tshark (decode part skipped to DNS):

~~~
$ tshark -r /tmp/test.cap -V -x -R frame.number==1
...
Domain Name System (query)
    Transaction ID: 0xb64d
    Flags: 0x0100 Standard query
        0... .... .... .... = Response: Message is a query
        .000 0... .... .... = Opcode: Standard query (0)
        .... ..0. .... .... = Truncated: Message is not truncated
        .... ...1 .... .... = Recursion desired: Do query recursively
        .... .... .0.. .... = Z: reserved (0)
        .... .... ...0 .... = Non-authenticated data: Unacceptable
    Questions: 1
    Answer RRs: 0
    Authority RRs: 0
    Additional RRs: 0
    Queries
        google-public-dns-a.google.com: type A, class IN
            Name: google-public-dns-a.google.com
            Type: A (Host address)
            Class: IN (0x0001)

0000  00 04 03 35 00 00 00 00 00 00 00 00 00 00 08 00   ...5............
0010  45 00 00 4c 0d 66 00 00 40 11 bd 91 0a 78 95 22   E..L.f..@....x."
0020  08 08 08 08 f9 f5 00 35 00 38 36 b4 b6 4d 01 00   .......5.86..M..
0030  00 01 00 00 00 00 00 00 13 67 6f 6f 67 6c 65 2d   .........google-
0040  70 75 62 6c 69 63 2d 64 6e 73 2d 61 06 67 6f 6f   public-dns-a.goo
0050  67 6c 65 03 63 6f 6d 00 00 01 00 01               gle.com.....
~~~

In hex dump we can see DNS payload starting with transaction ID "b6 4d" at 002c.
With this sample we can easyly encode our own DNS query using hex ASCII escape chars,
and pass it to netcat (reply is piped to hexdump):

~~~
$ echo -n -e "\x13\x37\x01\x00\x00\x01\x00\x00\x00\x00\x00\x00\x13google-public-dns-a\x06google\x03com\x00\x00\x01\x00\x01" | nc -u -w1 8.8.8.8 53 | hexdump -C
00000000  13 37 81 80 00 01 00 01  00 00 00 00 13 67 6f 6f  |.7...........goo|
00000010  67 6c 65 2d 70 75 62 6c  69 63 2d 64 6e 73 2d 61  |gle-public-dns-a|
00000020  06 67 6f 6f 67 6c 65 03  63 6f 6d 00 00 01 00 01  |.google.com.....|
00000030  c0 0c 00 01 00 01 00 00  53 b2 00 04 08 08 08 08  |........S.......|
00000040
~~~

tcpdump shows:

~~~
11:02:32.045931 IP 10.120.149.34.55204 > 8.8.8.8.53: 4919+ A? google-public-dns-a.google.com. (48)
11:02:33.750337 IP 8.8.8.8.53 > 10.120.149.34.55204: 4919 1/0/0 A 8.8.8.8 (64)
~~~

We see valid DNS session with ID 4919 (decimal for our 0x1337, crafted in first two octets of our string), and we can decode reply packet from from server:

~~~
 tshark -r /tmp/test.cap -V -x -R frame.number==4
...
Domain Name System (response)
    [Request In: 3]
    [Time: 1.704406000 seconds]
    Transaction ID: 0x1337
    Flags: 0x8180 Standard query response, No error
        1... .... .... .... = Response: Message is a response
        .000 0... .... .... = Opcode: Standard query (0)
        .... .0.. .... .... = Authoritative: Server is not an authority for domain
        .... ..0. .... .... = Truncated: Message is not truncated
        .... ...1 .... .... = Recursion desired: Do query recursively
        .... .... 1... .... = Recursion available: Server can do recursive queries
        .... .... .0.. .... = Z: reserved (0)
        .... .... ..0. .... = Answer authenticated: Answer/authority portion was not authenticated by the server
        .... .... ...0 .... = Non-authenticated data: Unacceptable
        .... .... .... 0000 = Reply code: No error (0)
    Questions: 1
    Answer RRs: 1
    Authority RRs: 0
    Additional RRs: 0
    Queries
        google-public-dns-a.google.com: type A, class IN
            Name: google-public-dns-a.google.com
            Type: A (Host address)
            Class: IN (0x0001)
    Answers
        google-public-dns-a.google.com: type A, class IN, addr 8.8.8.8
            Name: google-public-dns-a.google.com
            Type: A (Host address)
            Class: IN (0x0001)
            Time to live: 5 hours, 57 minutes, 6 seconds
            Data length: 4
            Addr: 8.8.8.8 (8.8.8.8)

0000  00 00 03 35 00 00 00 00 00 00 00 00 00 00 08 00   ...5............
0010  45 40 00 5c 7d c2 00 00 35 11 57 e5 08 08 08 08   E@.\}...5.W.....
0020  0a 78 95 22 00 35 d7 a4 00 48 57 a5 13 37 81 80   .x.".5...HW..7..
0030  00 01 00 01 00 00 00 00 13 67 6f 6f 67 6c 65 2d   .........google-
0040  70 75 62 6c 69 63 2d 64 6e 73 2d 61 06 67 6f 6f   public-dns-a.goo
0050  67 6c 65 03 63 6f 6d 00 00 01 00 01 c0 0c 00 01   gle.com.........
0060  00 01 00 00 53 b2 00 04 08 08 08 08               ....S.......
~~~

String contents in detail:

| \x13\x37                                     | Transaction ID
| \x01\x00\x00\x01\x00\x00\x00\x00\x00\x00     | Flags for standard, recursive, 1-question query
| \x13google-public-dns-a\x06google\x03com \x00 | Query string, null-terminated, each section starts with length number
| \x00\x01\x00\x01                             | Query type (A) and class (IN)

See [RFC 1035](https://www.ietf.org/rfc/rfc1035.txt).
