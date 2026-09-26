# Wireshark TCP Analysis Lab



## 1. Lab Objective



The objective of this lab was to understand how TCP establishes, maintains, and terminates a connection by capturing and analyzing real TCP traffic in Wireshark.



The lab focused on:



* TCP three-way handshake

* TCP sequence and acknowledgment numbers

* TCP flags

* TCP data transfer

* SSH running over TCP

* TCP connection termination using FIN and ACK packets

* Using Wireshark filters to isolate a specific TCP connection



---



## 2. Lab Environment



| Component       | Details                |

| --------------- | ---------------------- |

| Host OS         | Windows 11             |

| Virtualization  | VMware Workstation     |

| Network         | VMware Virtual Network |

| Attacker/Client | Kali Linux             |

| Kali IP         | 192.168.223.129        |

| Target/Server   | Ubuntu Linux           |

| Ubuntu IP       | 192.168.223.131        |

| Application     | SSH                    |

| SSH Port        | 22                     |

| Packet Analyzer | Wireshark              |



The TCP connection analyzed in the successful capture used:



* Kali client port: `55658`

* Ubuntu server port: `22`



Therefore, the connection was:



```text

192.168.223.129:55658  →  192.168.223.131:22

```



---



# 3. What is TCP?



TCP stands for **Transmission Control Protocol**.



TCP is a **transport-layer protocol** that provides reliable communication between two devices.



TCP is:



* Connection-oriented

* Reliable

* Ordered

* Port-based

* Designed to detect lost or missing data



Before TCP applications can exchange data, TCP normally establishes a connection using the **three-way handshake**.



---



# 4. TCP Three-Way Handshake



The TCP three-way handshake establishes a connection between the client and server.



The three packets are:



```text

1. SYN

2. SYN-ACK

3. ACK

```



The process can be represented as:



```text

Kali                                      Ubuntu

Client                                    Server

192.168.223.129                           192.168.223.131



&#x20;    |                                      |

&#x20;    |------------ SYN ------------------->|

&#x20;    |                                      |

&#x20;    |<--------- SYN + ACK -----------------|

&#x20;    |                                      |

&#x20;    |------------ ACK ------------------->|

&#x20;    |                                      |

&#x20;    |       Connection Established         |

```



### SYN



The client sends a SYN packet to request a TCP connection.



### SYN-ACK



The server responds with SYN and ACK.



The SYN acknowledges that the server is also participating in establishing the connection.



The ACK acknowledges the client's SYN.



### Final ACK



The client sends an ACK confirming the server's SYN.



The TCP connection is now established.



---



# 5. TCP Handshake Observed in Wireshark



The successful SSH connection used client port `55658` and server port `22`.



The first packet was Frame `13696`.



## Frame 13696 — SYN



```text

Source:      192.168.223.129

Destination: 192.168.223.131

Source Port: 55658

Destination Port: 22



Seq: 0

Ack: 0

Length: 0



Flags: SYN

```



The raw sequence number was:



```text

2629660350

```



Wireshark displayed the relative sequence number as:



```text

Seq = 0

```



Because the SYN consumes one sequence number, the next sequence number becomes:



```text

Seq = 1

```



The packet contained TCP options including:



* MSS

* SACK Permitted

* Timestamps

* Window Scale



---



# 6. TCP Sequence Numbers



TCP uses sequence numbers to keep track of data.



A useful rule is:



```text

SEQ changes when you send data.

ACK changes when you receive data.

```



For example, if a TCP segment starts at:



```text

Seq = 1

```



and contains:



```text

33 bytes

```



the next sequence number will be:



```text

1 + 33 = 34

```



The receiving host can then acknowledge:



```text

Ack = 34

```



This means:



> "I have received everything up to sequence 33 and I am expecting sequence 34 next."



---



# 7. TCP Data Transfer



After the three-way handshake, TCP can carry application data.



In this lab, the application was **SSH**.



SSH operates at the application layer while TCP provides the transport connection.



The relationship can be viewed as:



```text

Application Layer

&#x20;       SSH

&#x20;        |

&#x20;        v

Transport Layer

&#x20;       TCP

&#x20;        |

&#x20;        v

Internet Layer

&#x20;       IPv4

&#x20;        |

&#x20;        v

Network Access

&#x20;     Ethernet

```



---



# 8. First SSH Data Packet



After the TCP handshake, Frame `350` from the earlier capture contained the first observed SSH application data.



Important values were:



```text

Source:      192.168.223.129

Destination: 192.168.223.131



Source Port: 54238

Destination Port: 22



Seq: 1

Ack: 1

Length: 33



Flags: PSH, ACK

```



The TCP payload contained `33 bytes`.



Therefore:



```text

Starting Seq = 1

Data Length  = 33



Next Seq = 1 + 33

&#x20;        = 34

```



Ubuntu then acknowledged the data with:



```text

Ack = 34

```



This demonstrates how TCP uses sequence and acknowledgment numbers to track transmitted data.



---



# 9. Example of Bidirectional TCP Data



TCP maintains separate sequence-number spaces for each direction.



For example:



```text

Kali → Ubuntu



Seq = 34

Length = 1672



Next Seq = 1706

```



At the same time, Ubuntu maintains its own sequence numbers.



This means the two directions do not share one single sequence-number counter.



Conceptually:



```text

Kali sequence numbers

&#x20;       ↓

&#x20;  1 → 34 → 1706 → ...



Ubuntu sequence numbers

&#x20;       ↓

&#x20;  1 → 43 → ...

```



The ACK number tells the other side how much data has been successfully received.



---



# 10. TCP Flags



TCP uses flags to indicate the purpose or state of a TCP segment.



Important TCP flags observed or discussed in this lab include:



| Flag | Meaning                                                          |

| ---- | ---------------------------------------------------------------- |

| SYN  | Used to establish a TCP connection                               |

| ACK  | Acknowledges received data or TCP control information            |

| PSH  | Indicates data should be pushed toward the receiving application |

| FIN  | Indicates that the sender has finished sending data              |

| RST  | Abruptly resets a TCP connection                                 |

| URG  | Indicates urgent data                                            |



The most important flags for the connection lifecycle are:



```text

SYN → Establish connection



ACK → Acknowledge



FIN → Gracefully terminate



RST → Abruptly terminate

```



---



# 11. PSH + ACK in SSH Traffic



During the SSH session, Wireshark displayed packets with:



```text

PSH, ACK

```



These packets contained application data.



For example:



```text

PSH, ACK

Seq = 34

Ack = 43

Length = 1672

```



The `ACK` acknowledges data received from the other side.



The `PSH` indicates that the TCP segment is carrying application data that should be pushed toward the receiving application.



In this lab, the application data belonged to the SSH session.



The SSH payload was encrypted, so the actual contents of the user's SSH communication could not simply be read from the packet.



---



# 12. TCP Connection Termination



TCP connections can be terminated gracefully using the **FIN** flag.



A normal TCP connection termination commonly involves four packets:



```text

1. FIN + ACK

2. ACK

3. FIN + ACK

4. ACK

```



This happens because TCP is full-duplex.



Each direction of the connection is closed independently.



The normal sequence is:



```text

Kali                                      Ubuntu

192.168.223.129                           192.168.223.131

&#x20;     |                                         |

&#x20;     |-------- FIN, ACK --------------------->|

&#x20;     |                                         |

&#x20;     |<------------- ACK ---------------------|

&#x20;     |                                         |

&#x20;     |<---------- FIN, ACK --------------------|

&#x20;     |                                         |

&#x20;     |-------------- ACK -------------------->|

&#x20;     |                                         |

&#x20;     X             Closed                      X

```



---



# 13. Clean TCP Termination Captured in Wireshark



For the final experiment, a new SSH connection was created after starting a fresh Wireshark capture.



The SSH connection was:



```text

Kali:

192.168.223.129:55658



Ubuntu:

192.168.223.131:22

```



The SSH session was then closed normally using:



```text

exit

```



Wireshark captured the complete TCP termination sequence.



The four important packets were Frames:



```text

14542

14543

14544

14545

```



---



# 14. Frame 14542 — Kali Sends FIN



Frame `14542` was:



```text

Source:      192.168.223.129

Destination: 192.168.223.131



Source Port: 55658

Destination Port: 22



Seq: 9634

Ack: 11175

Length: 0



Flags: FIN, ACK

```



The important part was:



```text

FIN = 1

```



This means Kali was finished sending data in this direction.



The packet had:



```text

Seq = 9634

```



Because a FIN consumes one sequence number:



```text

Next Seq = 9635

```



So Kali's FIN was acknowledged using:



```text

Ack = 9635

```



---



# 15. Frame 14543 — Ubuntu Acknowledges FIN



Frame `14543` was:



```text

Source:      192.168.223.131

Destination: 192.168.223.129



Source Port: 22

Destination Port: 55658



Seq: 11175

Ack: 9635

Length: 0



Flags: ACK

```



Ubuntu acknowledged Kali's FIN:



```text

Ack = 9635

```



There was no FIN in this packet.



This means Ubuntu had acknowledged Kali's request to close its sending direction, but Ubuntu had not yet sent its own FIN.



---



# 16. Frame 14544 — Ubuntu Sends FIN



Frame `14544` was:



```text

Source:      192.168.223.131

Destination: 192.168.223.129



Source Port: 22

Destination Port: 55658



Seq: 11175

Ack: 9635

Length: 0



Flags: FIN, ACK

```



Ubuntu now sent its own FIN.



Its sequence number was:



```text

Seq = 11175

```



Since FIN consumes one sequence number:



```text

Next Seq = 11176

```



The ACK remained:



```text

Ack = 9635

```



because Ubuntu had not received any additional data from Kali.



---



# 17. Frame 14545 — Final ACK



Frame `14545` was:



```text

Source:      192.168.223.129

Destination: 192.168.223.131



Source Port: 55658

Destination Port: 22



Seq: 9635

Ack: 11176

Length: 0



Flags: ACK

```



Kali acknowledged Ubuntu's FIN:



```text

Ack = 11176

```



This was the final packet of the normal TCP termination sequence.



The sequence was therefore:



```text

Frame 14542

Kali → Ubuntu

FIN, ACK

Seq 9634

Ack 11175



&#x20;       ↓



Frame 14543

Ubuntu → Kali

ACK

Ack 9635



&#x20;       ↓



Frame 14544

Ubuntu → Kali

FIN, ACK

Seq 11175

Ack 9635



&#x20;       ↓



Frame 14545

Kali → Ubuntu

ACK

Seq 9635

Ack 11176

```



---



# 18. TCP Termination Summary



| Frame | Direction     |   Seq |   Ack | Length | Flags    |

| ----- | ------------- | ----: | ----: | -----: | -------- |

| 14542 | Kali → Ubuntu |  9634 | 11175 |      0 | FIN, ACK |

| 14543 | Ubuntu → Kali | 11175 |  9635 |      0 | ACK      |

| 14544 | Ubuntu → Kali | 11175 |  9635 |      0 | FIN, ACK |

| 14545 | Kali → Ubuntu |  9635 | 11176 |      0 | ACK      |



The complete termination sequence was:



```text

FIN/ACK → ACK → FIN/ACK → ACK

```



No RST was observed in these four termination packets.



This represents a **clean TCP connection termination**.



---



# 19. FIN vs RST



TCP has different ways of ending a connection.



## FIN



FIN is used for a graceful shutdown.



It means:



> "I am finished sending data."



The other side can acknowledge the FIN and close its own direction separately.



## RST



RST means reset.



It is used when a TCP connection is abruptly terminated or rejected.



Conceptually:



```text

FIN

↓

Graceful shutdown



RST

↓

Abrupt reset

```



During the successful termination experiment documented above, the connection closed using FIN and ACK packets rather than RST.



---



# 20. Wireshark Filters Used



### Show TCP traffic involving SSH



```text

tcp.port == 22

```



This displays TCP packets involving port `22`.



---



### Show the specific SSH connection



```text

tcp.port == 55658 && tcp.port == 22

```



This isolates the connection between:



```text

192.168.223.129:55658

```



and



```text

192.168.223.131:22

```



---



### Show FIN packets involving SSH



```text

tcp.port == 22 && tcp.flags.fin == 1

```



This displays TCP packets involving port 22 where the FIN flag is set.



---



### Show RST packets



```text

tcp.port == 22 && tcp.flags.reset == 1

```



This displays TCP packets involving port 22 where the RST flag is set.



---



# 21. Important Wireshark Concepts Learned



### Sequence number



The sequence number identifies the position of data in a TCP stream.



### Acknowledgment number



The acknowledgment number indicates the next sequence number the receiver expects.



For example:



```text

Seq = 1

Length = 33

```



results in:



```text

Next Seq = 34

```



The receiver can therefore send:



```text

Ack = 34

```



---



### TCP Segment Length



The TCP segment length represents the amount of application data carried by the TCP segment.



For example:



```text

Seq = 34

Length = 1672

```



results in:



```text

Next Seq = 1706

```



---



### SYN and FIN consume sequence numbers



Even though SYN and FIN do not carry normal application data, each consumes one sequence number.



For example:



```text

FIN Seq = 9634

Next Seq = 9635

```



---



# 22. TCP vs ICMP



TCP and ICMP serve different purposes.



| TCP                      | ICMP                                      |

| ------------------------ | ----------------------------------------- |

| Transport-layer protocol | Network-layer control/diagnostic protocol |

| Connection-oriented      | No connection                             |

| Uses ports               | Does not use TCP/UDP ports                |

| Uses sequence numbers    | Does not use TCP sequence numbers         |

| Uses ACKs                | Does not use TCP ACKs                     |

| Reliable delivery        | Not a reliable transport mechanism        |

| Uses SYN/ACK/FIN/RST     | Uses ICMP message types                   |

| Used by SSH              | Used by tools such as ping                |



The previous ICMP lab demonstrated:



```text

ICMP Echo Request

&#x20;       ↓

ICMP Echo Reply

```



This TCP lab demonstrated:



```text

SYN

&#x20;↓

SYN-ACK

&#x20;↓

ACK

&#x20;↓

Data

&#x20;↓

FIN/ACK

&#x20;↓

ACK

&#x20;↓

FIN/ACK

&#x20;↓

ACK

```



---



# 23. Final TCP Communication Flow



The complete SSH communication observed during the lab can be summarized as:



```text

&#x20;                   TCP CONNECTION



Kali                                           Ubuntu

192.168.223.129                                192.168.223.131

Port 55658                                     Port 22

&#x20;  |                                               |

&#x20;  |--------------- SYN -------------------------->|

&#x20;  |<-------------- SYN + ACK ---------------------|

&#x20;  |--------------- ACK -------------------------->|

&#x20;  |                                               |

&#x20;  |============== SSH DATA =======================|

&#x20;  |<============= SSH DATA =======================|

&#x20;  |============== SSH DATA =======================|

&#x20;  |                                               |

&#x20;  |--------------- FIN + ACK -------------------->|

&#x20;  |<---------------- ACK --------------------------|

&#x20;  |<------------- FIN + ACK ----------------------|

&#x20;  |---------------- ACK ------------------------->|

&#x20;  |                                               |

&#x20;             CONNECTION CLOSED

```



---



# 24. Key Lessons Learned



From this lab, I learned that:



1. TCP establishes connections using a three-way handshake.

2. SYN is used to begin a TCP connection.

3. SYN-ACK is the server's response to the client's SYN.

4. The final ACK completes the handshake.

5. TCP uses sequence numbers to track transmitted data.

6. TCP uses acknowledgment numbers to indicate the next expected sequence number.

7. TCP maintains separate sequence-number spaces for each direction.

8. TCP can carry application protocols such as SSH.

9. PSH + ACK packets can contain application data.

10. SSH data is encrypted even though TCP and packet metadata remain visible.

11. FIN is used for graceful TCP connection termination.

12. FIN consumes one sequence number.

13. A normal TCP termination commonly uses FIN, ACK, FIN, ACK.

14. RST represents an abrupt TCP reset rather than a graceful shutdown.

15. Wireshark filters can isolate specific TCP connections and TCP flags.

16. Looking at SEQ, ACK, Length, and Flags together makes TCP packet analysis much easier.



---



# 25. Conclusion



This lab provided practical experience analyzing TCP traffic with Wireshark.



The capture demonstrated the TCP connection lifecycle:



```text

Connection Establishment

&#x20;       ↓

Three-Way Handshake

&#x20;       ↓

Data Transfer

&#x20;       ↓

Connection Termination

&#x20;       ↓

Four-Way FIN/ACK Exchange

```



By analyzing the packets individually, I was able to observe how TCP uses:



* Ports

* Sequence numbers

* Acknowledgment numbers

* TCP flags

* Data lengths

* FIN and ACK packets



to establish, manage, and gracefully terminate a TCP connection carrying an SSH session.



