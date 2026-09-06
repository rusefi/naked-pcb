# T113 MMI — aftermarket BMW CarPlay / Android Auto interface

This teardown documents an aftermarket BMW infotainment interface built around an Allwinner T113 processor. Six photographs show both sides of three separate PCBs: a compute module, a vehicle-interface carrier board, and a display-interface board.

The notes below were compiled on September 6, 2026 from full-resolution photo inspection, the linked owner's account, and component documentation. Visible markings, manufacturer specifications, owner reports, and inferred functions are distinguished throughout. No powered measurements, continuity tracing, or firmware extraction were performed.

Original project: [MMI box reverse engineering and firmware customization discussion](https://www.reddit.com/r/CarHacking/comments/1vrb1w1/update_mmi_box_reverse_engineering_and/).

The owner identifies the device as an **ANYFAR-branded CarPlay / Android Auto retrofit box** installed between a BMW **NBT Evo head unit**, its display, and the iDrive CAN connection. They describe auxiliary-camera support and similarities with units sold under other brands. This is useful provenance, but does not establish firmware interchangeability with any particular Andream, Road Top, or Carlinkit product.

## Amazon identification and manufacturer manual

![ANYFAR product reference graphic showing enclosure dimensions and supplied power, display, RCA, and USB cables](img.png)

The supplied product graphic labels the enclosure **152 × 102 × 30 mm** and shows a power harness, separate display-input and display-output cables, and an RCA/USB harness. These are advertised dimensions and package contents, not measurements of the teardown hardware.

Investigated September 6, 2026. The best candidate found is the **ANYFAR EVO/MGU retrofit decoder**, associated with Amazon ASIN **`B0FQC6LTVB`**:

- [Amazon US product link — B0FQC6LTVB](https://www.amazon.com/dp/B0FQC6LTVB).
- [ANYFAR's official EVO/MGU product page](https://anyfar.com/products/carplay-module-interface-for-bmw-evo-mgu).
- [Manufacturer-hosted RM-BM2506F user manual](https://cdn-files.myshopline.com/file/store/1752490439429/RM-BM2506F-User-Manual-compressed.pdf), linked under Downloads on that product page.
- [ASIN-associated catalog entry](https://manuals.plus/asin/B0FQC6LTVB), used to locate the ASIN and its [Amazon-hosted product photograph](https://m.media-amazon.com/images/I/71s1m6r5VqL._AC_SL1500_.jpg). The photograph was downloaded and inspected; the catalog's generated installation prose is not used as a wiring reference.

The official manual identifies the product family as **RM-BM2506F**. The Amazon-associated photograph shows an ANYFAR enclosure with pink and turquoise display connectors in an arrangement consistent with these boards. The manufacturer's page describes an EVO ID5/ID6/ID7 application with a six-pin display connection, matching the owner's reported EVO application at the product-family level.

**This is a candidate commercial match, not a verified match to the photographed internal PCB revision.** The live Amazon product page failed to load during research, so current price, availability, seller, and selected variation could not be verified. Neither the listing image nor the manual establishes a T113 processor, `MSN_T113_CORE REV:03`, `CCX-AM4A_MAINPCB REV:8.0`, or `CCX-BM01_TP_LVDSPCB REV:6.0` inside a currently shipped unit. An exact match requires the seller or owner to confirm those markings. The plain NBT and CIC variants should not be substituted on the basis of the BMW brand or connector appearance alone.

## Archive contents and provenance

| File | Contents |
| --- | --- |
| [img.png](img.png) | ANYFAR product reference graphic: enclosure dimensions and cable set |
| [20260815_140213_2.jpg](20260815_140213_2.jpg) | Display board: large unidentified IC, stacking sockets, two automotive display connectors |
| [20260815_140252_2.jpg](20260815_140252_2.jpg) | Display board opposite side: INOVA APIX2 transmitter and receiver |
| [20260815_140346_2.jpg](20260815_140346_2.jpg) | Main carrier: GigaDevice MCU, CAN transceiver, power circuitry, harness connectors |
| [20260815_140445_1.jpg](20260815_140445_1.jpg) | Carrier underside and product/version sticker |
| [20260815_140652_1.jpg](20260815_140652_1.jpg) | Compute module: T113, storage, wireless module, additional multimedia ICs |
| [20260815_140716_2.jpg](20260815_140716_2.jpg) | Compute module underside, labeled test points, inspection sticker |

The six teardown JPEGs are **8160 × 6120 pixels**, approximately 50 megapixels. EXIF identifies a **Samsung Galaxy S23** and capture times on **August 15, 2026, between 14:02 and 14:07**. These are camera metadata, not independently verified chronology. Those six photos total approximately **57.7 MB / 55 MiB**; the product graphic is a separate reference image.

The six photos and original README were added together by `rusefillc` in commit `659c3be6db3fa626a5f1cc90a310314208d8e5eb`, **“t113 MMI naked”**, on September 6, 2026. The original README was a 92-byte Reddit URL with no embedded images; this document adds the findings and references all six photographs.

This directory contains no firmware dumps, schematics, source files, verified connector pinouts, or electrical measurements.

## Compute module

![Compute module component side showing the Allwinner T113, eMMC, wireless module, and multimedia ICs](20260815_140652_1.jpg)

PCB silkscreen:

```text
MSN_T113_CORE
REV:03 20250306
```

Underside inspection sticker:

```text
MSN-T113-CORE
25/33 -QC02
PC3K -2195
```

`20250306` looks like a design date. The remaining sticker codes appear to be production or inspection identifiers, but their exact meaning is undocumented.

| Visible identification | Interpretation |
| --- | --- |
| **Allwinner T113** | Main application processor |
| **THGBMNG5D1LBAIL** marking on the large BGA | Corresponds to Toshiba 4 GB eMMC storage |
| **Realtek RTL8733BS** | Wireless connectivity chip on a small soldered module |
| **Nextchip N5** | Additional multimedia/video IC; exact specification unconfirmed |
| **AB-logo IC, `CTYC14B5A`** | Consistent with Bluetrum's encoded chip markings; exact model and role require further identification |

### Processor and memory

The **T113 family uses dual Arm Cortex-A7 cores** and provides hardware video decoding and multiple display interfaces. Allwinner lists Tina Linux as a supported software system. The photographed package and absence of a separate obvious DDR chip are consistent with a **T113-S3-style implementation with integrated memory**, but the visible top marking says only `T113`. **128 MB RAM is a strong hypothesis, not a measured specification**; the S3 datasheet specifies 128 MB integrated DDR3. Sources: [Allwinner T113 overview](https://www.allwinnertech.com/index.php?a=index&c=product&id=106&solveid=43), [Allwinner T113-S3 datasheet](https://linux-sunxi.org/images/7/73/T113-s3_datasheet_v1.6.pdf).

The storage marking corresponds to a **4 GB, eMMC 5.0, 153-ball BGA** device. This is a concrete firmware-extraction target. It is managed flash with an MMC interface; a conventional SPI flash programmer is not the appropriate direct interface. Actual identity and capacity could be corroborated by reading its identification registers. Source: [Toshiba THGBMNG5D1LBAIL datasheet](https://media.digikey.com/pdf/Data%20Sheets/Toshiba%20PDFs/THGBMNG5D1LBAIL.pdf).

### Wireless and multimedia

The RTL8733BS is used in **dual-band Wi-Fi / Bluetooth modules**, with SDIO for Wi-Fi and UART for Bluetooth. This supports the expected wireless phone-projection role. Another module vendor's exact Bluetooth version or antenna specifications should not be transferred to this board without confirming its module revision. Source: [Fn-Link RTL8733BS module documentation](https://www.fn-link.com/6233A-SRB-Wi-Fi-Module-pd46827459.html).

Two miniature coaxial antenna connectors and printed antenna structures are visible. Their individual assignments require tracing.

The **Nextchip N5** is plausibly associated with auxiliary-camera input. Nextchip produces analog HD video-interface devices, but no authoritative documentation was found mapping this exact `N5` marking to supported formats or resolutions. AHD/CVBS support at this particular chip remains a hypothesis. Background: [Nextchip video technology](https://www.nextchip.com/en/ahd/ahd.php?idx=5).

The separate AB-logo IC may provide additional Bluetooth/audio functions, but its role is not established. Bluetrum's encoded markings are discussed in the independent [bluetrum-tools reverse-engineering project](https://github.com/kagaimiq/bluetrum-tools); the marking alone is not a verified identification of its firmware or function here.

### Exposed test points

![Compute module underside showing labeled serial, USB, audio, display, and power test points](20260815_140716_2.jpg)

| Labels | Likely purpose and limitation |
| --- | --- |
| `TXD`, `RXD` | Serial signals; processor endpoint and console availability unknown |
| `TX1`, `RX1` | Another serial pair; numbering need not match the SoC's UART numbering |
| `DP`, `DM` beside a four-hole header | Strongly suggests USB D+/D−; host versus OTG connection unconfirmed |
| `5V`, `3.3V`, `0.9V`, `1.8V` | Power-rail measurement points |
| `MIC+`, `MIC-` | Microphone-related signals |
| `L`, `R` | Likely audio channels |
| `IR` | Likely infrared signal |
| `LCD_HS`, `LCD_VS`, `LCD_DE`, `LCD_CLK` | Display timing signals, consistent with a parallel display interface |

There are signs of soldering/probing around several pads, but the photographs do not establish a definite severed connection.

The owner reports trying the serial pads, receiving CAN-related debug output, and damaging a trace while removing wires, after which MMI-side controls stopped working. **That failure does not itself prove the pad carried physical CAN signaling.** It could have interrupted an MCU–SoC serial connection carrying decoded CAN events. Readable CAN-related UART logs are compatible with a serial control channel rather than CAN-H/CAN-L or the Linux console. Source: [owner's report](https://www.reddit.com/r/CarHacking/comments/1vrb1w1/update_mmi_box_reverse_engineering_and/).

## Main carrier and vehicle interface

![Main carrier component side showing the GigaDevice MCU, CAN transceiver, power supplies, relay, and connectors](20260815_140346_2.jpg)

PCB silkscreen:

```text
CCX-AM4A_MAINPCB
REV:8.0  T=1.2MM
```

The clearly readable MCU is **GD32F305RBT6**. GigaDevice specifies:

- Arm **Cortex-M4**, up to **120 MHz**.
- **128 KB internal flash** and **64 KB SRAM**.
- **LQFP64** package.
- Two **CAN 2.0B** controllers.
- Multiple UART/USART, SPI, and I²C interfaces.

These are chip capabilities; the photos do not establish clock settings or which peripherals the firmware uses. Source: [GigaDevice GD32F305RBT6 product page](https://www.gigadevice.com/product/mcu/mcus-product-selector/gd32f305rbt6).

A nearby NXP IC's marking reads as **TJA1043**, a high-speed CAN transceiver with standby and sleep modes. This is strong physical evidence of the vehicle CAN interface. Source: [NXP TJA1043 datasheet](https://www.nxp.com/docs/en/data-sheet/TJA1043.pdf).

The likely division of responsibility is:

- **GD32:** vehicle messages, button events, power-state handling, and control coordination.
- **T113:** phone projection, graphical interface, media, and higher-level application behavior.

This split is an architectural inference. The GD32 and T113 have separate execution environments and likely separate firmware images, so accessing the MCU would not automatically provide a Linux shell.

Other visible carrier features include several switching power supplies and linear regulators, a **470 µF / 25 V** capacitor, a small **NEC/TOKIN relay**, harness connectors, board-stacking headers, unpopulated component footprints, and an unpopulated four-hole header near the MCU with a `GND` label.

The relay could serve audio or another switched signal path; its function is not established by the photographs. Capacitor voltage ratings do not establish a safe bench-power pinout or input range. The four-hole header is not a verified SWD or UART pinout.

![Main carrier underside showing PCB routing and BMW product/version sticker](20260815_140445_1.jpg)

Product sticker:

```text
BMW (宝马)01_TP
SW: 518 25-09-08
HW: AM4_V7.0
DATE: 2025.10.11
```

The **V7.0 sticker versus revision 8.0 PCB** is a visible discrepancy. It may reflect different assembly and PCB revision schemes, or a reused label. Neither establishes the firmware currently installed.

## Display interface board

![Display interface board with unidentified large IC, stacking sockets, and pink and turquoise automotive connectors](20260815_140213_2.jpg)

The large fine-pitch IC has **no readable identification**. Its position and extensive routing make video switching, processing, or programmable logic plausible functions. Identifying it as a particular FPGA, scaler, or MCU would exceed the available evidence.

The pink and turquoise connectors appear to be automotive HSD-style display connectors. The manufacturer manual now provides a product-family direction reference: **pink is input from the factory head-unit cable; turquoise is output to the display**. This comes from the RM-BM2506F installation drawing on PDF page 5, not color alone. The photographed board's individual contacts remain untraced. Source: [official installation manual](https://cdn-files.myshopline.com/file/store/1752490439429/RM-BM2506F-User-Manual-compressed.pdf#page=5).

![Display interface board opposite side showing INOVA INAP375T and INAP375R APIX2 devices](20260815_140252_2.jpg)

PCB silkscreen:

```text
CCX-BM01_TP_LVDSPCB
REV:6.0  T=1.2MM
```

Two devices are clearly identifiable:

- **INOVA INAP375R:** APIX2 receiver.
- **INOVA INAP375T:** APIX2 transmitter.

These devices support automotive serialized display links up to **3 Gbit/s**, with parallel RGB or OpenLDI/LVDS interfaces on their local sides. They also provide communication channels beyond the main video stream. These capabilities do not reveal the actual operating rate or screen resolution here. Sources: [INOVA INAP375R](https://inova-semiconductors.de/inap375r.html), [INOVA INAP375T](https://inova-semiconductors.de/inap375t.html).

The receiver/transmitter pair strongly supports an architecture that **receives the factory display stream and retransmits a selected output to the screen**.

### Newly identified high-speed switch

Closer inspection of the small rectangular IC between the connector solder joints in `20260815_140252_2.jpg` gives the marking **ASW3642**, with the Grand Micro logo and an `LGA42` line. This is a high-confidence visual identification, subject to physical confirmation.

Grand Micro documents the ASW3642 as a **12-channel bidirectional 2:1 / 1:2 analog multiplexer**, including four high-speed differential pairs, with a typical 8.5 GHz differential bandwidth. It switches electrical signals; it does not decode pixels, scale images, or translate RGB into APIX. Its advertised HDMI/DisplayPort applications do not imply that this board carries HDMI. Sources: [manufacturer product page](https://www.grandmicro.com.cn/producthall/hdmidpkg198.html), [manufacturer ASW3642 brief and pin diagram](https://www.grandmicro.com.cn/web/uploads/file/20240307/y0a0d0d9M81wGxxwTvV86173s7J0l21a.pdf).

The photo shows paired traces between this switch, the external connector contacts, and the lower serial-interface region of the INOVA devices. Its placement makes **switching or bypassing the serialized display links** a stronger hypothesis than using it as a full parallel-RGB selector. The exact A/B/common connections and switch-control source are not yet mapped. Its documented disabled state is high impedance, so its presence does not establish an unpowered, fail-safe bypass.

## Investigation of the T113 video path

### What leaves the compute module

The strongest working hypothesis is **parallel RGB plus pixel clock and timing signals**, routed through the compute-module sockets and carrier to the display board.

Evidence in the photos:

- On the component side of the core, a broad group of separate traces fans out from the T113 toward the upper portion of the left-hand socket, with the regulators at the top and wireless circuitry at the bottom of the photo.
- The reverse side exposes `LCD_CLK`, `LCD_DE`, `LCD_VS`, and `LCD_HS` beside the inspection sticker. Separate timing labels and a broad bus favor parallel video over a dedicated serial link, but labels do not establish the active pin-mux configuration.
- Both sides of the carrier show broad trace groups between the regions carrying the two daughterboards. This supplies a plausible physical route, but individual connector contacts have not been mapped across the stack.
- The display board has broad buses around the unidentified large IC and the INOVA pixel-interface regions, alongside the much smaller paired-trace groups near the external connectors.

These observations support a bus-level reconstruction, not a continuity-verified netlist. Vias, sockets, hidden copper, and the sticker interrupt visual tracing. A direct T113-to-INAP375T connection, or some signals bypassing the unidentified IC, cannot yet be excluded.

### T113 probe candidates

The T113-S3 reference path is **framebuffer / video surfaces → display engine → timing controller (TCON) → LCD pins**. The table provides datasheet candidates, not verified module-pad assignments:

| Module label | Candidate SoC function | T113-S3 package pin |
| --- | --- | --- |
| `LCD_CLK` | PD18 / LCD0-CLK | 75 |
| `LCD_DE` | PD19 / LCD0-DE | 76 |
| `LCD_HS` | PD20 / LCD0-HSYNC; alternate PE0 mapping exists | 54; alternate 44 |
| `LCD_VS` | PD21 / LCD0-VSYNC; alternate PE1 mapping exists | 53; alternate 45 |

PD0–PD17 provide 18 RGB data signals; PB2–PB7 can supply the six additional bits for 24-bit RGB. Actual bus width is unknown. These pins also have alternate functions, including LVDS/DSI on portions of port D. Confirm package orientation, mux settings, and continuity before applying this table. Source: [Allwinner T113-S3 datasheet, §§2.5–2.6 and Tables 4-2/4-3](https://linux-sunxi.org/images/7/73/T113-s3_datasheet_v1.6.pdf).

The module's `0.9V` and `1.8V` rail labels do not establish LCD signal voltage. Measure the relevant I/O supply and signal swing before attaching an analyzer or driving a pin.

### Display-board conversion and switching

The most plausible MMI-generated picture path is:

```text
T113 display output
  → probably parallel RGB through core sockets and carrier
  → unidentified display-board IC and/or pixel-bus routing
  → INAP375T pixel input
  → serialized APIX output
  → likely ASW3642 routing
  → turquoise connector
  → factory display
```

The **INAP375T is the identified serializer**. Its local pixel interface accepts RGB and/or OpenLDI/LVDS; which mode is configured on this board is unresolved. Its 100-pin LQFP table provides useful endpoints:

| Device | Signal | Package pins |
| --- | --- | --- |
| INAP375T | APIX downstream output, N / P | 45 / 46 |
| INAP375T | Upstream input, P / N | 37 / 38 |
| INAP375R | APIX downstream input, P / N | 43 / 44 |
| INAP375R | Optional downstream daisy-chain output, N / P | 39 / 40 |
| INAP375R | Upstream output, N / P | 49 / 50 |

Sources: INOVA [INAP375T datasheet, Table 3-1](https://inova-semiconductors.de/inap375t.html?cid=1027&file=files%2Fdaten%2FInap%20Downloads%2FINAP375T%2FDS_APIX_INAP375T_1_3%20%281%29.pdf) and [INAP375R datasheet, Table 3-1](https://inova-semiconductors.de/inap375r.html?cid=1071&file=files%2Fdaten%2FInap%20Downloads%2FINAP375R%2FDS_APIX_INAP375R_1_2.pdf). The silkscreen numbers around these ICs make package-pin counting practical. Their `PX1`–`PX30` pixel pins require the configured interface mapping; they must not be assumed to be sequential RGB bits.

The factory picture has at least two plausible routes: receive/process/retransmit through the INOVA pixel interfaces, or a serialized bypass/rerouting path involving the ASW3642. The receiver also has a documented daisy-chain output, providing another possible serial forwarding route. **The photos do not establish which route is selected in OEM mode.** The large unmarked IC may handle timing, conversion, or selection; its presence does not prove a framebuffer, scaler, overlay engine, or FPGA.

The manufacturer's term “LVDS IN/OUT” describes the external installation connections. On this board the INOVA components identify an **APIX serial link**; that should be distinguished from the T113's native OpenLDI/LVDS or parallel-RGB interfaces. No evidence places Nextchip N5 in series with the normal T113-to-display output; its suspected camera-input role remains separate.

### Measurements that would settle the path

1. With power removed, trace the four `LCD_*` pads to the candidate SoC pins and then record the mating core and carrier connector contacts. Continue those nets onto the display board. Check continuity through series resistors rather than assuming a direct zero-ohm connection.
2. Map the ASW3642 common/A/B channels to both external connectors and the INOVA serial pins above. Trace `SEL` and output-enable control to their source. Series coupling capacitors may block DC continuity; trace each side separately.
3. Capture pixel clock, DE, HS, and VS in OEM, MMI-menu, phone-projection, and camera modes. Compute frame timing from the measured clock and total horizontal/vertical periods, including blanking. A live T113 clock alone does not prove that the display is currently showing it.
4. Read the T113 display/pinctrl configuration and INOVA interface settings when runtime access is available. These would distinguish RGB versus LVDS and establish bit mapping, polarity, and timing.
5. Observe the switch-control state during OEM/MMI transitions. Until its channel map is known, treat a changing selector as evidence of switching, not proof of a particular bypass route. APIX requires suitable differential probing; an ordinary low-speed UART analyzer cannot decode it.

## Inferred system architecture

The following is a revised functional interpretation, not a traced schematic. Dashed connections mark proposed internal paths. The ASW3642 node represents several physical switch channels, not a single port; their exact mapping is unknown. Serial return channels are omitted for clarity.

```mermaid
flowchart LR
    HU["BMW head unit"] --> IN["Pink input"]
    IN -.-> SW["ASW3642 serial routing / possible bypass"]
    SW -.-> RX["INAP375R receiver"]
    RX -.-> LOGIC["Unidentified IC / pixel-bus routing"]
    PHONE["Phone"] <-->|"Wireless / USB"| SOC["T113 compute module"]
    SOC -.->|"Probably parallel RGB + timing"| CARRIER["Stacking connectors / carrier"]
    CARRIER -.-> LOGIC
    LOGIC -.-> TX["INAP375T transmitter"]
    TX -.-> SW
    SW -.-> OUT["Turquoise output"]
    OUT --> SCREEN["Factory display"]

    CAN["Vehicle / iDrive CAN"] <--> PHY["TJA1043"]
    PHY <--> MCU["GD32 MCU"]
    MCU <-.->|"Control link: protocol unknown"| SOC
```

The modular construction would make it practical to reuse a compute module with different vehicle-specific carriers and display adapters. This is a design interpretation, not proof of compatibility across products.

## Reverse-engineering opportunities

1. **Map the labeled serial pairs to their endpoints.** Continuity measurements should establish whether they reach the T113, GD32, wireless circuitry, or an intervening device. Capture startup traffic after measuring logic levels.
2. **Identify the USB header's full pinout and destination.** USB enumeration could reveal an existing device interface. The photographs do not establish ADB support.
3. **Investigate Allwinner FEL availability.** `sunxi-fel` communicates with Allwinner's ROM USB recovery handler. Whether this board exposes the required USB port and a practical entry mechanism remains untested. Source: [sunxi-tools documentation](https://github.com/linux-sunxi/sunxi-tools).
4. **Locate the GD32 debug interface.** SWD would target the MCU's firmware and control logic; readout protection status is unknown.
5. **Obtain and inspect the correct vendor update package.** It may reveal component firmware, partition layout, application binaries, and update checks without requiring physical storage access.
6. **Read the eMMC if necessary.** Preserve the user area, accessible boot partitions, and configuration information. In-circuit access requires confirming voltage levels and preventing contention with the SoC.
7. **Identify the unmarked display IC.** This becomes particularly relevant for changing video switching or display behavior.

For button behavior or quick camera selection, the **GD32–T113 control connection** looks especially valuable: it is the likely point where vehicle inputs become application commands. For modifying the graphical interface, the **T113 firmware and eMMC contents** are the more direct targets.

## Unresolved questions

The available evidence does not establish:

- Exact T113 variant, installed RAM capacity, or operating clock.
- Running OS, kernel, bootloader, or application versions.
- Firmware partition layout, encryption, secure boot, or signature enforcement.
- Root credentials, an enabled console, ADB, or an available recovery path.
- MCU readout-protection state or the MCU–SoC control protocol.
- Connector pinouts, UART endpoints, or serial voltage levels.
- Actual APIX link rate, display resolution, camera formats, or active RGB/LVDS pin-mux configuration.
- Individual video contacts across the stacking connectors, ASW3642 channel/control mapping, or the OEM-mode bypass/forwarding route.
- Exact identity and function of the unmarked display IC, Nextchip N5, or AB-logo IC.
- Exact internal-board match to Amazon ASIN B0FQC6LTVB, or firmware compatibility with another branded MMI box.

The presence of Android Auto does not establish that the box itself runs Android. Further conclusions require electrical measurements, a firmware image, or runtime access.
