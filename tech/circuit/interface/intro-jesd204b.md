# Introduction to JESD204B

[Original post](https://www.analog.com/en/technical-articles/understanding-layers-in-jesd204b-specification.html#){:target="_blank"}

## 1. Overview

![Simplified Data Flow Through JESD204B Layers](https://www.analog.com/-/media/analog/en/landing-pages/technical-articles/understanding-layers-in-jesd204b-specification/figure1.png)

##### Figure 1. Simplified Data Flow Through JESD204B Layers.

- The **application layer** allows for configuration and data mapping of the JESD204B link. 

- The **transport layer** maps conversion samples to and from framed nonscrambled octets. 

- The **scrambling layer** can optionally take those octets and scramble or descramble them in order to reduce EMI effects by spreading the spectral peaks. Scrambling would be done in the transmitter and descrambling done in the receiver. 

- The **data link layer** is where the optionally scrambled octets are encoded to 10-bit characters. This layer is also where control character generation or detection is done for lane alignment monitoring and maintenance. 

- The **physical layer** is the serializer/deserializer (SERDES) layer responsible for transmitting or receiving the characters at line rate speeds. This layer includes the serializer, drivers, receivers, the clock,and data recovery. 


## 2. Application layer

The application layer allows for special user configurations and for sample data to be mapped outside of the typical JESD204B specification. This can allow for a more efficient use of the interface to accomplish power reductions and other benefits. It is important to remember, that the transmitter (ADC) and receiver (FPGA) must both be configured for these special configurations. The receiver and transmitter must be configured identically so that data is transmitted and interpreted correctly. Configuring the application layer in a unique way can be beneficial for ADCs that need to pass data in sample sizes that are different than the N' (the number of transmitted bits per sample). This could allow for multiple samples to be repacked in such a way that the lane rate can be reduced, and the overall link efficiency increased.

## 3. Transport layer

Take a closer examination of the transport layer of the JESD204B specification. The transport layer takes the ADC samples and adds information (padding) to generate nibble groups (usually on 4-bit boundaries). This information is in the form of tail bits or control bits, which can provide additional information about the transmitted data. The transport layer arranges these nibble groups into frames. It is important to note that the transport layer delivers the samples to the data link layer as parallel data. The width of the parallel data bus is determined by the framer architectures, in which a single byte is eight bits, a dual byte is 16 bits, and so on. The serializer has not yet been reached in the data flow at this point.

A single ADC can be mapped to a single lane link, or can be mapped to a multilane link. This configurability is especially handy for GSPS ADCs used in wideband RF applications where the sample rate dictates that multiple lanes be used in order to meet limits on lane rates. Multiple converters can also be mapped onto multiple lanes for M number of ADCs in the same device. The ADCs can be mapped to a single lane link or into a multilane link consisting of L number of lanes. In some cases an ADC may need multiple lanes. The lane rate maximum of a given ADC determines this. For example, the 12-bit, 2.5 GSPS AD9625 has a lane rate maximum of 6.5 Gbps. This means that with N' equal to 16, a total of eight lanes are required. Sometimes the lane rate may be limited by the FPGA in the system. For customers using GSPS ADCs in their RF applications, one design parameter may be cost. In order to drive the cost down, an FPGA with lower lane rate capability may be used. For example, the 14-bit, 1.0 GSPS dual-channel AD9680 has a maximum lane rate of 12.5 Gbps. The AD9680 has four output lanes and can be configured to use decimation to lower the sample rate, and thus the lane rate. This is serving two purposes: a lane rate reduction and a bandwidth selection for a specific RF application.

Now, moving back to the JESD204B parameters, the N' parameter gives the JESD204B word size. The converter sample resolution is broken down into 4-bit nibbles. A 14-bit converter, as well as a 16-bit converter, has four nibbles, while a 12-bit converter has three nibbles. If N' is set to 12 for the AD9625, the number of required lanes can be reduced by two such that six lanes are required to maintain a lane rate of less than 6.5 Gbps. The conversion samples (S) are recommended to be mapped into JESD204B words on 4-bit nibble boundaries. **Figure 2** shows the mapping of ADC samples into the serial lanes. It is parameterized such that it covers the many potential cases that can be realized with JESD204B.

![](https://www.analog.com/-/media/analog/en/landing-pages/technical-articles/understanding-layers-in-jesd204b-specification/figure2.png)

##### Figure 2. Transport Layer ADC Sample Mapping

The N' parameter is found by multiplying the number of nibbles by four. It can be advantageous to both the transmitter and receiver to set N' to 16 for converters with resolutions ranging from eight bits to 16 bits. This allows for the same transmitter and receiver to be used for multiple converters, easing overall system design. A noncomplete nibble has room for either control bits (CS) or tail bits (shown as TT below in **Figure 2**), as defined by the JESD204B standard. The equation N' = N + CS + T must be satisfied. Control bits, if any, are appended after the LSB to each conversion sample. After using the number of converters, the number of samples per frame, the JESD204B word size, and the maximum lane rate to calculate the number of lanes, we can determine the number of octets transmitted per frame, F. In order to determine this parameter, the following equation can be used:  F = (M × S × N')/(8 × L). For more on JESD204 link parameters, refer to Reference 1, which describes the link parameters in greater detail. In addition, a four part webinar series provides further information on the JESD204 standard beginning with the transport layer.

The transport layer determines how to pack the data from the ADC based on the link configuration parameters that have been defined for a given device. These parameters are transmitted from the ADC to the FPGA during the initial lane alignment sequence (ILAS). These settings are configured via a serial port interface (SPI) that would set register values on the ADC and the FPGA to define the link configuration parameters. A checksum is generated from the parameters and transmitted so that the receiver (FPGA) can verify the link configuration parameters were received correctly. The parameters sent across the link are not used to configure the receiver; they are only used to verify that the link parameters match. If an error is detected, the FPGA will report this error via an interrupt that is defined in the error reporting of the JESD204B specification. For more on the link configuration parameters, please see more in Reference 1 at the end of this article.

## 4. Data link layer

The data link layer takes in the parallel framed data (containing ADC samples, control bits, and tail bits) and outputs 8B/10B words that are serialized in the physical layer and can optionally be scrambled. The 8B/10B scheme adds some overhead, but provides dc balanced output data and built in error checking. The data link layer synchronizes the JESD204B link through the link establishment process. The link establishment consists of three distinct phases:

- **Code Group Synchronization (CGS**) – Interface clocks are not required, so the **RX** must align its bit and word boundaries with the **TX** serial outputs. The **RX** sends a **SYNC** request to the **TX** to transmit a known repetitive-bit-sequence on all of its lanes, in this case, **K28.5 /K/** characters. The exact bit sequence of the characters can be found in the standard. The **RX** will shift the bit data on each lane until it finds four consecutive **K28.5** characters. At this point, it will know the bit and word boundaries and have achieved **CGS**. It can then de-assert the **SYNC** and both the **TX** and **RX** can drop into the next state – initial lane alignment sequence (**ILAS**).

- **Initial Lane Alignment Sequence (ILAS)** – A nice feature of the JESD204B protocol allows for absorbing lane skew with some internal FIFO/buffers within the **RX** block. After CGS is achieved, the **TX** transmits a known set of frames of characters on each of the lanes – known as the lane alignment sequence (starts with **K28.0 /R/** character, ends with **K28.3 /A/** character). Upon receiving the alignment sequence, the **RX** will FIFO buffer the data until all lanes have received the entire alignment sequence.  Since the entire sequence is known, the lanes can then be re-aligned so that any lane skew is absorbed by the FIFO memories on each lane, and the lanes can then have the data released at the same point in time within the **RX** block. This alleviates the need for having a matched layout for the SERDES lanes, since lane skew is absorbed by the FIFO memory.

- **User data** – Once the code groups have been synchronized and the lanes have been aligned, the user data can be correctly received. If during this last state when the user data is valid, there is a need to restart the process, the **RX** can send a request for **SYNC**, which will restart the process.

![](https://e2e.ti.com/cfs-file/__key/communityserver-blogs-components-weblogfiles/00-00-00-03-25/6710.Image-1.png)

##### Figure 3. JESD204B protocol state diagram

During the code group synchronization (**CGS**), each receiver (FPGA) must locate **K28.5** characters in its input data stream being transmitted from the ADC using clock and data recovery (**CDR**) techniques. Once a certain number of consecutive **K28.5** characters have been detected on all link lanes, the receiver block deasserts the **SYNC~** signal to the transmitter block. In JESD204A, the transmit block captures the change in **SYNC~**, and after a fixed number of frame clocks, starts the **ILAS**. In JESD204B, the transmit block captures the change in **SYNC~** and starts the **ILAS** on the next local multiframe clock (**LMFC**) boundary.

In the **ILAS**, the main purpose is to align all the lanes of the link, to verify the parameters of the link, and to establish where the frame and multiframe boundaries are in the incoming data stream at the receiver. During **ILAS**, the link parameters are sent to the receiver (FPGA) to designate how data will be sent to the receiver. The **ILAS** consists of four or more multiframes. The last character or each multiframe is a multiframe alignment character **/A/**. The first, third, and fourth multiframes begin with an **/R/** character and ends with an /A/ character. For the case of ADI ADCs, the data in between them is ramp data. The receiver uses the final **/A/** of each lane to align the ends of the multiframes within the receiver.

The second multiframe contains an **/R/** and **/Q/** character followed by link parameters. The **/Q/** character indicates that the proceeding data is the link configuration parameters. Additional multiframes can be added to ILAS if needed by the receiver. After the last **/A/** character of the last **ILAS** multiframe, user data starts. In systems were no interlane skew management is needed, **ILAS** can be bypassed given both the transmitter and receiver support the mode. 

After the **CGS** and **ILAS** phases have completed, the transmitter can begin sending out user data (which are the samples from the ADC). In this phase, user data is streamed from the transmitter to the receiver according to the link parameters that have been defined in the transmitter (ADC) and relayed to the receiver (FPGA). This is where all the bandwidth from the RF spectrum that has been digitized by the GSPS ADC is now being transmitted for processing. The receiver block processes and monitors the data it receives for errors, including incorrect running disparity (8B/10B error), not in table (8B/10B error), unexpected control character, incorrect **ILAS**, and interlane skew (note: 8B/10B is designed such that the running disparity is maintained such that the output data is dc balanced while maintaining sufficient output transitions for the clock and data recovery circuitry in the receiver). If any of these errors exists, it is reported back to the transmitter in one two ways:

- **SYNC~** assertion—resynchronization (**SYNC~** pulled low) is called for at each error.
- **SYNC~** reporting—the **SYNC~** is pulsed high for a frame clock period if an error occurs.

During the initial lane alignment sequence, the data link layers are responsible for aligning the lanes in the receiver. The placement of /A/ characters are used to align the lanes in the receiver. The JESD204 A and B specifications require that the /A/ characters be separated by at least 17 octets. This mitigates the effects ofa large amount of system skew. In JESD204 A and B systems, skew is defined in three possible scenarios:

- one transmitter block and one receiver block
- multiple transmitter blocks and one receiver block
- one transmitter block and multiple receiver blocks

Upon reaching the user data phase, character replacement in the data link layer allows frame and lane alignment to be monitored and corrected if necessary. Character replacement is performed on both frame and multiframe boundaries. There are two cases, one for frame-based character replacement and the other for multiframe-based character replacement. In frame-based character replacement, if the last character of a frame is identical to the last character of the previous frame on a given lane, then the transmitter will substitute that character with an /F/ character. This is also done if the last character of the previous frame is 0xFC when scrambling is enabled. In multiframe-based character replacement, if the last character of a multiframe is identical to the last character of the previous frame on a given lane, then the transmitter will substitute the character with an /A/ character. In this case, character replacement is also done if the last character of the previous multiframe is 0x7C when scrambling is enabled. An illustration of the CGS, ILAS, and user data phases along with the character replacement is given in **Figure 4**.

![](https://www.analog.com/-/media/analog/en/landing-pages/technical-articles/understanding-layers-in-jesd204b-specification/figure3.jpg?w=900&la=en)

##### Figure 4. Data Link Layer—ILAS, CGS, Data Sequencing

In receiver character replacement, the receiver must do the exact opposite of what is done in the transmitter. If an **/F/** character is detected, it is replaced with the final character of the previous frame. When an **/A/** is detected, it is replaced with the final character of the previous multiframe. When scrambling is enabled, the **/F/** characters are replaced by **0xFC** and the **/A/** characters are replaced by **0x7C**. If the receiver detects two consecutive errors, it can realign the lanes. However, data will be corrupted while it performs this operation. A brief list of all the JESD204 control characters is provide in Table 1. For more information on the control characters, see Reference 3.

##### Table 1. JESD204B Control Characters

Control Character  | Control Symbol | 8-Bit Value | 10-Bit Value, RD = -1 |	10-Bit Value, RD = +1 |	Description
:-:|:-:|:-:|:-:|:-:|-
/R/ |	K28.0 |	000 11100 |	001111 0100 |	110000 1011 |	Start of multiframe
/A/ |	K28.3 |	011 11100 |	001111 0011 |	110000 1100 |	Lane alignment
/Q/ |	K28.4 |	100 11100 |	001111 0010 |	110000 1101 |	Start of link configuration data
/K/ |	K28.5 |	101 11100 |	001111 1010 |	110000 0101 |	Group synchronization
/F/ |	K28.7 |	111 11100 |	001111 1000 |	110000 0111 |	Frame alignment

Data can be optionally scrambled, but it is important to note that the scrambling does not start until the very first octet is following the **ILAS**. This means that the **CGS** and **ILAS** are not scrambled. Scrambling can be optionally implemented in order to reduce spectral peak emissions on the high speed serial lanes between the transmitter and receiver. In certain system designs, this can be advantageous where particular data patterns may result in the generation of spectra detrimental to the frequencies of operation in a given system. The scrambling block utilizes a self synchronous scrambling pattern that has the polynomial `1 + x^14 + x^15` (block diagram shown in **Figure 5**). The data is scrambled prior to the **8B/10B** encoder and is descrambled in the receiver after decoding. Since the scrambling pattern is self synchronous, the two shift registers at the input and output must not be set to the same initial setting, otherwise the scrambling function would not work. The descrambler is done in such that it will always catch up and self synchronize to the scrambler after two octets of data. This layer should have the ability to be bypassed since not all systems may require the data stream to be scrambled.


![JESD204B Scrambling/Descrambling](https://www.analog.com/-/media/analog/en/landing-pages/technical-articles/understanding-layers-in-jesd204b-specification/figure4.jpg)

##### Figure 5. JESD204B Scrambling/Descrambling

##  5. Physical Layer

The physical layer is where the data is serialized, and the 8B/10B encoded data is transmitted and received at line rate speeds. The physical layer includes serial/deserializer (SERDES) blocks, drivers, receivers, and **CDR**. These blocks are often designed using custom cells since the data transfer rates are very high. The JESD204 and JESD204A both support speeds up to 3.125 Gbps. The JESD204B specification supports three possible speed grades. Speed Grade 1 supports up to 3.125 Gbps and is based on the OIF-SxI5-0.10 specification. Speed Grade 2 supports up to 6.375 Gbps and is based on the CEI-6G-SR specification. The third speed grade supports up to 12.5 Gbps and is based on the CEI-11G-SR specification. Table 2 provides an overview of some of the specifications for the physical layer for each of the three speed grades.

##### Table 2. JESD204B Physical Layer Specifications

 Parameter | OIF-Sx15-01.0 | CEI-6G-SR | CEI-11G-SR
 :-:|:-:|:-:|:-:
Line Rate (Gbps) | ≤3.125 |	≤6.375 | ≤12.5
Output Differential Voltage (mVppd) | 500 (min)<br>1000 (max) |	400 (min)<br>750 (max) | 360 (min)<br>770 (max)
Output Rise/Fall Time (ps) | >50 | >30 | >24
Output Total Jitter (pp UI) | 0.35 | 0.30 | 0.30

Table 2 gives the line rate, differential voltage, rise/fall time, and total jitter for the signals in the physical layer of the JESD204B standard according to each speed grade. The higher speed grades have reduced signal amplitudes to make it easier to maintain a high slew rate,and thus maintain an open data eye for proper signal transmission. These high speed signals, with fast rising and falling edges, place tight constraints on board level design. For many individuals designing wideband RF systems, this should not be something new.  However, the one key difference with high speed digital is the wide bandwidth. Typical RF systems have signal bandwidths on the order of 10% or less of the operating RF frequency. With these high speed serial lane rates, the bandwidth to consider for system design is typically 3× to 5× the lane rate. For a lane rate of 5 Gbps, the bandwidth of the signal would be 7.5 GHz to 12.5 GHz. With this amount of bandwidth, it is important to maintain proper signal integrity and to understand how to measure for signal integrity.

In serial differential interfaces, the eye diagram is a common measurement of the integrity of the signal. **Figure 6** shows the transmitter eye diagram mask for JESD204 operating at speeds up to 3.125 Gbps. Table 3 gives the details on timing, voltage levels, impedances, and return loss. The signal must not encroach onto the beige area of the figure, but must stay in the white at all times. The table defines the conditions for which the transmitter must meet the eye mask. There are similar eye diagram masks for the other two speed grades within the JESD204B specification as well. These are detailed in the CEI-6G-SR and CEI-11G-SR physical layer specifications. For more information on the eye diagram masks, see Reference 2, which describes the physical layer measurements.

##### Table 3. Eye Diagram Measurements 

Parameter |	Value |	Unit
:-:|:-:|:-:|
XT1 |	0.175 |	UI
XT2 |	0.45 |	UI
YT1 |	0.50 |	UI
YT |	0.25 |	UI
DJ |	0.17 |	pp UI
TJ |	0.35 |	pp UI
 

![](https://www.analog.com/-/media/analog/en/landing-pages/technical-articles/understanding-layers-in-jesd204b-specification/figure5.png)

 ##### Figure 6. Example TxEye Diagram Mask.

## 6. Conclusion

The number of designs employing JESD204B is increasing each day and across many market segments such as communications, instrumentation, and military and aerospace. The push in these market segments toward systems that employ wideband RF designs utilize GSPS ADCs, which need the JESD204B serial interface. FPGAs that have transceivers capable of serializing/deserializing JESD204B are becoming increasingly available as well as becoming less expensive. As the utilization of the JESD204B interface becomes more popular it is important to understand the layers that exist in the JESD204B specification. As described, each layer within the specification has its own function to perform. The configuration and data mapping is performance in the application layer, while the transport layer maps conversion samples to and from nonscrambled octets. Scrambling can optionally be enabled to reduce EMI effects by spreading the spectral peaks. The data link layer is where the optionally scrambled octets are encoded to 8B/10B characters, and is also the layer where control character generation or detection is done for lane alignment monitoring and maintenance. The drivers, receivers, clock, and data recovery circuits make up the physical layer where the data is transmitted and received. This article should have provided a better understanding of the layers in JESD204B so that system designers can be more prepared to implement JESD204B in their next design.

## 7. References

- Harris, Jonathan. "**Understanding JESD204B Link Parameters**", Planet Analog, 2013.

- Harris, Jonathan. "**Three Key Physical Layer (PHY) Performance Metrics for a JESD204B Transmitter**", EE Times, 2013.

- Harris, Jonathan. "**Link Synchronization and Alignment in JESD204B: Understanding Control Characters**", EE Times, 2013.

- Palkert, Thomas. "**System Interface Level 5: Common Electrical Characteristics for 2.488-3.125 Gbps Parallel Interfaces**", Optical Internetworking Forum, 2002. 

- "**Common Electrical I/O (CEI)—Electrical and Jitter Interoperability Agreements for 6G+ bps, 11G+ and 25G+ bps I/O**", Optical Internetworking Forum, 2005.