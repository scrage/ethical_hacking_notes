# The OSI Model
**Mnemonic** device for remembering the layers:
1. **P**lease
2. **D**o
3. **N**ot
4. **T**hrow
5. **S**ausage
6. **P**izza
7. **A**way


**Layers:**

| layer number | layer name   | examples                       |
| ------------ | ------------ | ------------------------------ |
| 1            | Physical     | data cables, cat6              |
| 2            | Data         | Switching, MAC addresses       |
| 3            | Network      | IP addresses, routing          |
| 4            | Transport    | TCP/UDP                        |
| 5            | Session      | session management             |
| 6            | Presentation | WMV, JPEG, MOV, media files... |
| 7            | Application  | HTTP, SMTP                     |

The communication process entails both directions.
Going up the stack is called **encapsulating**, going down the stack is called **decapsulation**.

**Layers 5, 6, 7** can be grouped together. The purpose of the top 3 layers is to reduce the data to be transported into binary format. They produce a **PDU** (**Protocol Data Unit**) that we call data.

**Layer 4**'s first thing to do is to identify what application is making a request and what service is going to receive it. - Applications make request, services receive them.
Since identification in networking is done by addresses, that is what the Transport layer is working with: it works with port addresses. Each PDU (called **segment** on the Transport layer) is stamped with a source and destination addresses.

Data is segmented on the Transport layer for security and performance reasons, and to allow multiple communications to occur at the same time (multiplexing).
One of the protocols that define how to produce a PDU is **TCP**. It sacrifices time over reliability, and is the most popular protocol. The other protocol designed to transport data is **UDP**.

**Layer 3** produces a **packet** from the data given by layer 4, and stamps it with a source and destination **IP** address. While the Transport layer uses (port) addresses for identifying services or applications, the Network layer uses (IP) addresses to identify devices on the network

**Layer 2** produces a **frame** from the packet PDU, and one of the most popular protocol to use it is **Ethernet**. Ethernet has source and destination addresses, but they are physical. Those addresses are burnt onto the devices that the producing company make, and are called **MAC** addresses.

**Layer 1** takes the bits of data and produce a **signal** to carry them. 


# The TCP/IP Model
On of the most popular models we use to substitute the OSI model is the TCP/IP model, which is not as elaborate as the OSI model. This model was developed by the same people who developed the Internet.

- The top 3 layers of the OSI model (Application, Presentation, Session) is grouped together to what is called **Application** layer here, and it too produces data.
- That layer is connected to the **Transport** layer, which will produce segments.
- The OSI model's Network layer is called the **Internetwork** layer here, and it too will produce a packet.
- Those packets are sent on the **Network Access** layer. (The OSI model's Datalink and Physical layers corresponds to this layer.) The Network Access layer produces its PDU called a frame.


https://www.youtube.com/watch?v=3b_TAYtzuho