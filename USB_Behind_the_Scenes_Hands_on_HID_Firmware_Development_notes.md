# USB Behind the Scenes: Hands-on HID Firmware Development

------

## Introduction to USB

------

### Definition:

- Serial Protocol used for transfer of data and power.
- Multiple protocols and ports and respective drivers were needed.

###  History:

- USB 1.x (Jan 1996)

  - Low speed (1.5 Mbits/s) and Full Speed (12 Mbits/sec)
  - No Extensions were allowed due to timing and power issues
  - Wide adoption after releasing USB 1.1 (Sept 1998)
  - Connectors: A and B.																	

- USB 2.0(Apr 2000):
  - High Speed (480 Mbits/s).
  - On-The-Go (OTG) was introduced.
  - Dedicated charging ports (up to 1.5 A).
  - Connectors: Mini-A, Mini-B and Micro-USB

- USB 3.x(Nov 2008):
  - SuperSpeed (5 Gbits/s).
  - SuperSpeed+ (10 Gbits/s) after releasing USB 3.1 (Jul. 2013).
  - SuperSpeed+ dual-lane (20 Gbits/s) after releasing USB 3.2 (Aug 2017).
  - Connectors: only A and C. [Type B and micro B are also used](https://en.wikipedia.org/wiki/USB_hardware)

- USB 4 (Aug 2019):
  - Thunderbolt 3 hardware interface (40 Gbits/s).
  - Connectors: only C .

### USB 2.0 Cable Structure:

![USB 2.0 Cable Structure](./Pictures/USB_2_0_Cable_Structure.png)

### USB Main Features

- Hot-pluggable (plug and play).
- Self configured.
- Some device can be powered directly from the USB(bus-powered).
- USB is host controlled (single host per bus).
- Every USB Product is programmed to have a vendor ID and Product ID(VID/PID)
- Every USB device is addressed by the host uniquely during device enumeration.
- USB is a **polled bus** (frequently sampled by both host and devices without including any interrupt mechanism in the USB controller and according to the state found on the bus after sampling the bus specific actions or events happen).

### Physical Bus Topology

- In USB, host and devices are connected physically to the bus according to the "tired-star" topology.

- Up top 127 devices  can be connected to the bus including the hubs (devices are 7 bit addressed , and address 0 is reserved as initial address for every new connected device)

- Up to 7 tiers are allowed.

- Up to 5 hubs can be connected in series.

  ![alt-text-1](./Pictures/Tiered_Star_Topology_figure.png "title-1") ![alt-text-2](./Pictures/Tiered_Star_Topology_Block_Diagram.png "title-2")

### USB Device Power Supply

- Bus-Powered (VBUS): up to 500 mA for USB 2.0 , or up to 900 mA for USB 3.x

- Self-powered (external power supply):

  Devices that require more current than what VBUS can supply must use an external power supply.

- Mixing the two types is also possible  

  

### VBUS  

- The nominal VBUS voltage is normally ~5 V.

- VBUS voltage can drop down to ~ 4.4 V (according to the load on the VBUS).

- USB device can normally draw current from the host through the VBUS depending on its state:
  - **Not configured** (default state, newly connected device):

    USB 2.0: up to 100 mA.
    USB 3.x: up to 150 mA.

  - **Configured**(host and device have negotiated):

    USB 2.0: may **ask** to draw up to 500 mA (high power device).
    USB 3.x: may **ask** to draw up to 900 mA (high power device). 

    - **Suspended**(device is idle):

      Up to 0.5 mA (2.5 mA if configured as high power). The current of the pull up and pull down resistors must be considered (they sink ~ 0.2 mA).


- USB device can draw more current -if needed- according to battery charging and   power delivery specifications.
- USB BC(Battery Charging) spec 1.2 (Dec 2010): up to 1.5 A
- USB PD(Power Delivery) spec Ver 2 rev 3(Aug 2019): allows up to 100 watt (5 A by 20 V).

### Smart Charger

- The smart charger has a dedicated charging port (DCP) controller.

- The DCP tries different states(sine voltage, square voltage signal etc) and monitors the amount of the drawn current.

  

## USB Protocol

------

### Deferential states

|                | Full and High Speed | Low Speed    |
| -------------- | ------------------- | ------------ |
| Differential 0 | D+Low D-High        | D-Low D+High |
| Differential 1 | D-Low D+High        | D+Low D-High |

​									The Difference should be >= 200 mV.

- Differential states allows external noise to be filtered as D+ and D- are both going to be effected by noise equally hence they output doesn't have noise



### Bus States

| State                | FS   | HS   | LS                      |
| -------------------- | :--: | ---- | ----------------------- |
| SE0 (Single Ended 0) |     both data lines are low      | same           | same           |
| Detached | SE0 | same | same |
| Idle | Differential 1 | SE0 | Differential 0 |
| J | Differential 1 (as idle state) | Differential 1 | Differential 0 |
| K | Differential 0 | Differential 0 | Differential 1 |
| ~~SE1 (single Ended 1)~~ | ~~both data lines are high (illegal)~~ | ~~same~~ | ~~same~~ |



### Bus States 2

| State                         | FS              | HS                               |
| ----------------------------- | --------------- | -------------------------------- |
| RESET                         | SE0 (>=10 ms)   | idle (3.15) then keeps the SE0   |
| Suspend                       | Idle (3 ms)     | Idle (3.125 ms) then FS idle (J) |
| Resume                        | K(20 ms) EOP.   | k (>= 20 ms)                     |
| Sync or Start of Packet (SOP) | K J K J K J K K | 15 times(K J) K K                |
| End of Packet (EOP)           | SE0 SE0 J       | 1111111(bit stuffing error)      |



### USB 2.0 Speed Identification

![USB 2.0 speed indentification](./Pictures/USB_2_0_Speed_Identification.png)



### Bit Stuffing

- The insertion of non-information bits(s) into the data.

- In USB FS/HS: **zero insertion after six consecutive ones**.
- Ensures adequate state transition on the line (to keep the clock in sync).

### Non-Return-To-Zero Inverted (NRZI)

- NRZI and Bit Stuffing are handled by hardware itself (USB controller)

- Zero inverts the line state as it is.

  <img src="./Pictures/Non-return-to-Zero_inverted.png" alt="NRZI" style="zoom:150%;" />



### USB Host Controllers

- USB 1.x:
  - Universal Host Controller Interface (UHCI): **software** has to do a lot of bus management work.
  - Open Host Controller Interface (OHCI): **hardware** is responsible for doing most of the bus management work.

- USB 2.0:
  - Extended Host Controller Interface (EHCI).

Host driver had to be written to support all of the these three controllers.

- USB 3.x (and the earlier versions):
  - Extensible Host Controller Interface (xHCI): the newest host controller which replaces the older controllers.



### Frames

- The times between two SOF signals is called a frame.

- SOF signal is used to keep the data on the bus in sync.

  ![Frames](./Pictures/Frames.png)



### Endpoints

- An endpoint is a logical entity and it can be seen as data sender or as data receiver (depending on the endpoint type).

- Endpoints can be found only in the USB device side not on the host. (hint: Endpoints are similar to ports on TCP)
- The **direction** of the endpoint is named from the **host perspective**.

- ​        IN: from device to host.          OUT: from host to device.
- Devices can have up to 16 IN and 16 OUT endpoints.
- IN endpoint X and OUT endpoint X are totally two different endpoints (exception endpoint 0). Example IN endpoint 2 is different from OUT endpoint 2.

- Every device must support the endpoint 0 IN and OUT.
- Endpoint zero is used to configure the device.
-  the host send configuration data to device over OUT endpoint 0 and device will reply to requests of the host using IN endpoint 0.

### [Packet and Transaction Types](./PDFs/USB+Under+the+Microscope+-+Packets+and+Transaction+Types.pdf)

