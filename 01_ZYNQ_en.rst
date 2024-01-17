 .. image:: images/images_0/88.png

========================================
Introduction to ZYNQ
========================================

 The highlight of the Zynq series is that the FPGA contains a complete ARM processing subsystem (PS), Each Zynq series of processor includes the Cortex-A9 processor, The entire processor construction is centered on the processor, And the integrated memory controller and a lot of peripherals into the processor subsystem, Making the kernel of Cortex-A9 completely independent of the programmable logic unit in Zynq-7000, That is to say, if the programmable logic unit section (PL) is not used for the time being, Subsystems of the ARM processor can also work independently, This is fundamentally different from the previous FPGA, It is centered on the processor.

Zynq is two functional blocks, the PS part and the PL part, to put it bluntly, the SOC part of ARM and the FPGA part. Among them, PS integrates two ARM Cortex™ -A9 processors, AMBA ® interconnect, internal memory, external memory interfaces and peripherals. These peripherals mainly include USB bus interface, Ethernet interface, SD / SDDIO interface, I2C bus interface, CAN bus interface, UART interface, GPIO, etc.

 .. image:: images/images_1/image1.png
:align: center

* * Overall block diagram of the ZYNQ chip * *

-PS: Processing system (Processing System), which is the part of the SOC of an ARM unrelated to the FPGA.
-PL: programmable logic (Progarmmable Logic), which is the FPGA part.
The PL part is the same as the 7 series, and the corresponding 7 series products can be seen in the DS190 document.

 .. image:: images/images_1/image2.png
:align: center

1.1PS and PL interconnection technologies
========================================

-ZYNQ is the first product that closely combines high-performance ARM Cortex-A9 series processor with high-performance FPGA within a single chip. In order to realize high-speed communication and data interaction between ARM processor and FPGA, and play the performance advantages of ARM processor and FPGA, it is necessary to design efficient interconnection channels between high-performance processor and FPGA. Therefore, how to design efficient PL and PS data interaction pathway is the top priority of ZYNQ chip design, and also one of the keys to the success or failure of product design. In this section, we will mainly introduce the connection between PS and PL, and let users understand the technology of the connection between PS and PL.
-In fact, in the specific design, we often do not need to do much work in the connection. After we join the IP core, the system will automatically use the AXI interface to connect our IP core and processor, we only need to do a little more supplement.
-AXI, the full name of Advanced eXtensible Interface, is an interface protocol introduced by Xilinx from the series 6 of FPGA, which mainly describes the data transmission mode between the master device and the slave device. Continue to use in ZYNQ, the version is AXI 4, so we often see AXI 4.0, ZYNQ internal devices have AXI interface. In fact, AXI is a part of AMBA (Advanced Microcontroller Bus Architecture) proposed by ARM. It is a high performance, high bandwidth and low latency bus, which is also used to replace the previous AHB and APB buses. The first version of AXI (AXI 3) was included in AMBA 3.0 released in 2003, and the second version of AXI AXI (AXI 4) is included in AMBA 4.0 released in 2010.
-AXI protocol mainly describes the data transmission mode between the master devices and the slave devices, and the connection is established by the handshake signal. The READY signal is sent when it is ready to receive data from the device. When the data of the main device is ready, the VALID signal is issued and maintained, indicating that the data is valid. Data transmission only starts when both the VALID and READY signals are valid. When the two signals remain valid, the main device continues to transmit the next data. The main device may undo the VALID signal or terminate the READY signal from the device. The protocol of AXI is shown in the figure. At T2, the READY signal of the device is valid, the VILID signal of the main device is valid at T3, and the data transmission starts.

 .. image:: images/images_1/image3.png
:align: center

 The AXI handshake timing diagram

In ZYNQ, AXI-Lite, AXI 4 and AXI-Stream three buses are supported. Through Table 5-1, we can see the characteristics of the AXI interface in these three.

.. csv-table:: the interface feature
  :header: "Interface protocol", "properties", "application"
  :widths: 15, 20, 30
  
  
  "AXI 4-Lite", address / single data transmission, "Low speed peripheral or control"
  "AXI 4", Address / burst data transfer, "bulk transfer of addresses"
  "AXI 4-Stream", transfer only data, burst transmission, "data flow and media streaming transmission"

- AXI4-Lite:
With lightweight, simple structure characteristics, suitable for small batch data, simple control occasions. Batch transmission is not supported, and you can only read and write one word at a time (32bit). Mainly used to access some low-speed peripherals and peripheral control.
- AXI4:
 The interface is similar to AXI-Lite, but an added function is batch transmission, which can read and write an address in a row. That is to say, it has the burst function of data reading and writing.
The above two kinds use memory mapping control mode, that is, ARM user custom IP into a certain address for access, reading and writing like reading and writing their own RAM, programming is also very convenient, the development difficulty is low. The cost is too much resources, need additional read address lines, write address lines, read data lines, write data lines, write response lines these signal lines.
- AXI4-Stream:
This is a continuous flow interface that doesn't need an address line (much like FIFO, just read or write all the time). For this type of IP, the ARM cannot be controlled by the above memory mapping mode (FIFO has no concept of address at all), and there must be a conversion device, such as the AXI-DMA module, to convert the memory mapping to a streaming interface. AXI-Stream is applicable in many situations: video stream processing; communication protocol conversion; digital signal processing; wireless communication. In essence, data channels are built for numerical flow, building continuous data streams from sources (such as ARM memory, DMA, wireless receiving front end, etc.) to hosts (such as HDMI display, high-speed AD audio output, etc.). This interface is suitable for real-time signal processing.

The AXI 4 and AXI 4-Lite interfaces contain 5 different channels:

- Read Address Channel
- Write Address Channel
- Read Data Channel
- Write Data Channel
- Write Response Channel

Each of the channel is an independent AXI handshake protocol. The following two figures show the read and write models, respectively:

 .. image:: images/images_1/image4.png
:align: center

The AXI Read data channel

 .. image:: images/images_1/image5.png
:align: center

 The AXI writes to the data channel

-The AXI bus protocol is implemented in the ZYNQ chip with hardware, including 9 physical interfaces, namely AXI-GP 0 ~ AXI-GP3, AXI-HP 0 ~ AXI-HP3, and AXI-ACP interface.
-AXI _ ACP interface is an interface defined under the ARM multi-core architecture, translated into the accelerator consistency port in Chinese to manage AXI peripherals without cache such as DMA, and the PS end is the Slave interface.
-AXI _ HP interface, is the high-performance / bandwidth AXI 3.0 standard interface, a total of four, the PL module as the main device connection. Mainly for PL access memory on PS (DDR and on-Chip RAM)
-AXI _ GP interface, which is the general-purpose AXI interface, has four interfaces in total, including two 32-bit master device interfaces and two 32-bit slave device interfaces.

 .. image:: images/images_1/image6.png
:align: center

 As you can see, only two AXI-GPs are Master Port, namely the host interface, and the other seven ports are Slave Port (slave interface). The host interface has the permission to initiate reading and write. ARM can use two AXI-GP host interfaces to actively access PL logic. In fact, it is to map PL to an address, and read and write PL register is like reading and writing its own memory. The rest of the slave interfaces are passive interfaces, which accept read and write from PL and accept accordingly.
In addition, the nine AXI interface performance is also different. The GP interface is a 32-bit low-performance interface with a theoretical bandwidth of 600 MB/s, while the HP and ACP interfaces are a 64-bit high-performance interface with a theoretical bandwidth of 1200 MB/s. Some people will ask, why is the high-performance interface not made into a host interface? This enables high-speed data transfer initiated by ARM. The answer is that the high-performance interface does not need the ARM CPU to be responsible for the data transfer at all, and the real porter is the DMA controller located in the PL.
The ARM on the PS side directly has the hardware to support the AXI interface, while the PL needs to implement the corresponding AXI protocol using logic. Xilinx In the Vivado development environment to provide off-the-shelf IP such as AXI-DMA, AXI-GPIO, AXI-Dataover, AXI-Dataover, AXI-Stream have implemented the corresponding interface, when used directly added from the Vivado IP list to achieve the corresponding functions. The following below shows various DMA IP at Vivado:

 .. image:: images/images_1/image7.png
:align: center

The following is the introduction of several commonly used AXI interface IP:

-AXI-DMA: To realize the conversion from PS memory to PL high-speed transmission high-speed channel AXI-HP <- - - -> AXI-Stream
-AXI-FIFO-MM2S: Implement the conversion from PS memory to PL universal transmission channel AXI-GP <- - - - -> AXI-Stream
-AXI-Datamover: realize the conversion from PS memory to PL high-speed transmission high-speed channel AXI-HP <- - - -> AXI-Stream, but this time is completely controlled by PL, PS is completely passive.
-AXI-VDMA: To realize the conversion from PS memory to PL high-speed transmission high-speed channel AXI-HP <- - - -> AXI-Stream, which is only specifically for video, image and other two-dimensional data.
-AXI-CDMA: This is done by PL to move data from one location in memory to another, without CPU involvement.
-On how to use these IP, we will give examples in later sections. Sometimes, the user needs to develop their own defined IP to communicate with the PS, and then they can use the wizard to generate the corresponding IP. User custom IP cores can have AXI 4-Lite, AXI 4, AXI-Stream, PLB and FSL interfaces. The latter two are not supported by the ARM end.
-With these official IP and wizard generated custom IP, users don't actually need to know much about AXI timing (unless there is a problem), because Xilinx has encapsulates all the details related to AXI timing, and users only need to pay attention to their own logical implementation.
-AXI protocol is strictly speaking a peer-to-peer master and slave interface protocol, when multiple peripherals need to interact with each other data, we need to add a AXI Interconnect module, namely AXI interconnection matrix, function is to provide one or more AXI master devices to one or more AXI from a switching mechanism (somewhat similar to the switch exchange matrix).
-This AXI Interconnect IP core can support up to 16 master devices, 16 slave devices, and add more interfaces with more IP cores.

AXI Interconnect Basic connection modes include the following types:

- N-to-1 Interconnect
- to-N Interconnect
- N-to-M Interconnect (Crossbar Mode)
- N-to-M Interconnect (Shared Access Mode)

 .. image:: images/images_1/image8.png
:align: center

The case of multiple to one

 .. image:: images/images_1/image9.png
:align: center

One-to-many situations

 .. image:: images/images_1/image10.png
:align: center

Multi-to-multiple read-write address channels

 .. image:: images/images_1/image11.png
:align: center

 Multi-to-multiple read and write data channel

The AXI interface devices inside ZYNQ are connected through the interconnection matrix, which not only ensures the efficiency of data transmission, but also ensures the flexibility of connection. Xilinx In the Vivado, we provide the IP core axis _ interconnect to implement this interconnection matrix, and we can just call it.

 .. image:: images/images_1/image12.png
:align: center

AXI Interconnect IP

1.2 Introduction to the development process of ZYNQ chip
========================================
-As ZYNQ integrates the CPU with the FPGA, developers need to design both the ARM OS application and device drivers and the hardware logic design for the FPGA part. The development is to understand the Linux operating system, the architecture of the system both, and to build a hardware design platform between the FPGA and the ARM system. Therefore, the development of ZYNQ requires the collaborative design and development of software personnel and hardware personnel. This is the so-called "software and hardware collaborative design" in ZYNQ development.
-The design and development of the hardware system and software system of the ZYNQ system requires the following development environment and debugging tools: Xilinx Vivado.
-Vivado Design suite realizes the design and development of FPGA parts, constraints on pins and timing, compilation and simulation, and realizes the design process of RTL to bitstream. Vivado Is not a simple upgrade of the ISE design kit, but a brand new design kit. It replaces all the important tools of the ISE design suite, such as Project Navigator, Xilinx Synthesis Technology, Implementation, CORE Generator, Constraint, Simulator, Chipscope Analyzer, FPGA Editor, etc.
-Xilinx SDK (Software Development Kit), SDK is the Xilinx software development suite (SDK), based on the Vivado hardware system, the system will automatically configure some important parameters, including tools and library paths, compiler options, JTAG and flash settings, debugger connected to the bare board support package (BSP). The SDK also provides drivers for all supported Xilinx IP hardcores. SDK supports IP hardcore (on FPGA) and processor software collaborative debugging. We can use advanced C or C + + language to develop and debug ARM and FPGA systems to test whether the hardware system is working properly. The SDK software is also built from the Vivado software and does not need to be installed separately.

 The development of ZYNQ is also a hardware before software approach. The specific process is as follows:

1. Create a new project on the Vivado and add an embedded source file.
2. Add and configure basic peripherals for PS and PL in Vivado, or need to add custom peripherals.
3. Generate the top-level HDL file in the Vivado and add the constraint file. Recompile to generate the bitstream file (.bit)。
4. Export hardware information to the SDK software development environment. In the SDK environment, you can write some debugging software verification hardware and software, and debug ZYNQ system separately combined with bitstream files.
5. Generate the FSBL files in the SDK.
6. ate u-boot in VMware virtual machine. Mirror images of elf and bootloader.
7 in the SDK through the FSBL file, bit stream file system. The bit and the u-boot. The elf file generates a BOOT.bin document.
8. Generate the kernel mirror file Zimage of Ubuntu and the root file system of Ubuntu in VMware. In addition, you need to customize the FPGA IP write driver.
9. Put the BOOT, kernel, device tree, and root file system file into the SD card, start the development board power supply, and the Linux operating system will start from the SD card.

The above is a typical ZYNQ development process, but ZYNQ can also be used as an ARM alone, so that there is no need for a relationship with PL end resources, and it is not much different from the traditional ARM development. ZYNQ can also only use the PL part, but the configuration of PL still needs to be completed by PS, but the firmware of only PL cannot be cured through the traditional curing Flash method.

1.3 What skills are required for learning ZYNQ
========================================
Learning ZYNQ is more demanding than learning FPGA, MCU, ARM and other traditional tools development, and it is not impossible to learn ZYNQ well overnight.

Software developer

-Principles of computer composition
-C, C + + language
-Computer operating system
-tcl script
-Good foundation of English reading

Logic developer

-Principles of computer composition
-C language
-Digital circuit foundation
-Verilog, and the VHDL language
-Good foundation of English reading

 .. image:: images/images_0/888.png

* ZYNQ-7000 development platform FPGA tutorial * - ` Alinx official website <http: / / www.alinx.com>`_