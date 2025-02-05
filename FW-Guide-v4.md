# **Part 1: Foundational Concepts**

## **1. Introduction**

### **1.1 Purpose of the Guide**

This guide outlines a detailed roadmap for creating custom Direct Memory Access (DMA) firmware on FPGA-based devices, with the ultimate goal of accurately emulating PCIe hardware. Such emulation can serve a wide range of applications, including:

- **Hardware Development & Testing**  
  - Use FPGA-based emulation to replicate various hardware devices during development.  
  - Run system-level tests without the expense or availability constraints of specific donor hardware.

- **System Debugging & Diagnostics**  
  - Reproduce complex hardware behaviors in a controlled environment to pinpoint bugs or driver-related issues.  
  - Conduct trace analysis at the transaction layer (TLP) or memory-mapped I/O level.

- **Security & Malware Research**  
  - Investigate low-level PCIe vulnerabilities or advanced malware that interacts with hardware directly.  
  - Observe how certain device drivers behave when hardware signatures are partially or fully spoofed.

- **Hardware Emulation & Legacy Support**  
  - Replace aging hardware with an FPGA-based solution that mimics the original device’s PCIe IDs, BARs, and interrupts.  
  - Preserve legacy workflows on newer systems by emulating older or discontinued PCIe devices.

By following this guide, you will learn to:
1. **Gather** essential device information from a physical “donor” PCIe card.  
2. **Customize** FPGA firmware to present the same device/vendor IDs, BAR layouts, and capabilities.  
3. **Build & Configure** your development environment (Xilinx Vivado, Visual Studio Code, etc.).  
4. **Understand** the fundamentals of PCIe and DMA that are crucial to reliable device emulation.

> **Why This Matters**  
> Proper hardware emulation saves effort, reduces costs, and often allows for rapid iteration. FPGA-based cards can be reprogrammed on-the-fly, letting you adapt to multiple devices or firmware variations far more easily than with fixed hardware.

---

### **1.2 Target Audience**

This resource caters to a broad spectrum of professionals and enthusiasts:

- **Firmware Developers**  
  Interested in manipulating low-level system interactions, driver design, or advanced debugging of hardware/firmware stacks.

- **Hardware & Validation Engineers**  
  Seeking a controllable way to test system components with a wide variety of device profiles and conditions—without physically swapping PCIe cards every time.

- **Security Researchers**  
  Focused on analyzing the threat vectors introduced by DMA, exploring potential vulnerabilities in PCIe interactions, or performing safe sandbox emulations of malicious code.

- **FPGA Hobbyists & Makers**  
  Eager to expand their FPGA knowledge by building custom PCIe cores, learning advanced hardware description languages, and exploring real-world device enumeration.

---

### **1.3 How to Use This Guide**

The guide is split into three parts, each building on the last:

1. **Part 1: Foundational Concepts**  
   - Covers the prerequisite knowledge, environment setup, capturing donor device data, and making initial firmware adjustments.
2. **Part 2: Intermediate Concepts and Implementation**  
   - Delves into deeper firmware customization, TLP-level manipulations, debugging strategies, and how to refine an emulated device’s behavior to match or surpass its donor’s functionality.
3. **Part 3: Advanced Techniques and Optimization**  
   - Explores in-depth debugging tools, performance tuning, and best practices to ensure long-term maintainability of your FPGA-based DMA solutions.

> **Recommendation**: Complete Part 1 thoroughly before moving on. Skipping or partially implementing these foundational steps can lead to confusion and misconfigurations in later stages.

---

## **2. Key Definitions**

Having precise terminology is crucial for success in FPGA-based PCIe emulation. The following list expands on each relevant term:

1. **DMA (Direct Memory Access)**  
   - **Definition**: Hardware-mediated transfers between devices and system memory without CPU intervention.  
   - **Relevance**: Emulated devices heavily rely on DMA for throughput. Ensuring correct DMA configuration is central to a functional FPGA design.

2. **TLP (Transaction Layer Packet)**  
   - **Definition**: The fundamental communication unit in PCIe, encapsulating both header information and data payload.  
   - **Relevance**: Understanding TLP structure is vital if you plan to modify or analyze data at the PCIe transaction layer.

3. **BAR (Base Address Register)**  
   - **Definition**: Registers specifying the address ranges (memory or I/O) where a PCIe device’s resources appear in the system address space.  
   - **Relevance**: Accurately replicating a donor device’s BAR layout is key for correct driver loading and memory-mapped I/O handling.

4. **FPGA (Field-Programmable Gate Array)**  
   - **Definition**: A reconfigurable chip whose internal circuitry can be redesigned (via HDL) to implement custom hardware logic.  
   - **Relevance**: FPGAs let you quickly iterate on PCIe device designs, swapping out emulated devices with minimal hardware changes.

5. **MSI/MSI-X (Message Signaled Interrupts)**  
   - **Definition**: PCIe-compliant interrupt methods allowing devices to trigger CPU interrupts through in-band messages rather than dedicated lines.  
   - **Relevance**: Replicating donor interrupt behavior (especially the number of MSI vectors) can be critical for the driver that expects a specific interrupt mechanism.

6. **Device Serial Number (DSN)**  
   - **Definition**: A 64-bit unique identifier some PCIe devices use for licensing, authentication, or advanced driver checks.  
   - **Relevance**: Some drivers refuse to load or function unless the DSN matches the expected hardware.

7. **PCIe Configuration Space**  
   - **Definition**: A defined region (256 bytes for PCI or 4 KB for PCIe extended) detailing device ID, vendor ID, capabilities, and operational parameters.  
   - **Relevance**: Ensuring your FPGA device’s configuration space mirrors the donor’s (or includes the right subset) is essential to fool a host into treating it as the genuine article.

8. **Donor Device**  
   - **Definition**: The actual PCIe card from which you obtain data (IDs, class codes, etc.) for emulation.  
   - **Relevance**: The more data you accurately replicate, the closer your FPGA will behave to the original hardware in enumerations and function.

---

## **3. Device Compatibility**

### **3.1 Supported FPGA-Based Hardware**

1. **Squirrel (35T)**  
   - A cost-effective Artix-7–based FPGA board that supports basic DMA operations. Recommended if you’re new to FPGA-based PCIe development.

2. **Enigma-X1 (75T)**  
   - Offers more logic resources (LUTs, Block RAM) than a 35T, useful for moderate complexity tasks or extended debugging/tracing features.

3. **ZDMA (100T)**  
   - Targets higher performance applications with substantial FPGA resources for intensive data transfers or multiple concurrent DMA channels.

4. **Kintex-7**  
   - A robust, more premium FPGA family with advanced PCIe IP cores, typically used in demanding or large-scale emulation tasks.

> **Tip**: Always check the specific lane configuration (x1, x4, x8) and speed rating (Gen1, Gen2, etc.) for your FPGA card, verifying it meets or exceeds what your host motherboard can support.

### **3.2 PCIe Hardware Considerations**

- **IOMMU / VT-d**  
  - *Recommendation*: Temporarily disable to avoid restricted DMA regions, especially important if you need full memory access for thorough testing.

- **Kernel DMA Protection**  
  - *Windows VBS / Secure Boot*: In some cases, these features intercept or limit direct PCIe memory mapping.  
  - *Linux IOMMU or AppArmor/SELinux rules*: Adjust accordingly to ensure the FPGA can access the memory ranges it needs for emulation.

- **PCIe Slot Requirements**  
  - Choose a physical PCIe slot with enough lanes and confirm the BIOS is set to allocate those lanes appropriately.  
  - If you notice performance issues or partial enumeration, confirm your system is not forcing x1 operation on a physically larger slot.

### **3.3 System Requirements**

1. **Hardware**  
   - **CPU**: At least a quad-core from Intel or AMD for smooth Vivado synthesis and to manage OS overhead.  
   - **RAM**: 16 GB or more for comfortable Vivado usage, especially for multi-hour synthesis runs.  
   - **Storage**: 100 GB of SSD space recommended for faster project builds; mechanical HDDs can slow the build process drastically.  
   - **OS**: Windows 10/11 (64-bit) or a well-supported Linux distribution (e.g., Ubuntu LTS, RHEL/CentOS) to run Xilinx Vivado.

2. **Peripheral Devices**  
   - **JTAG Programmer**: (Xilinx Platform Cable USB II, Digilent HS3, or similar) needed to program your FPGA with the bitstream you create.  
   - **Dedicated Machine**: Strongly suggested if you’re altering BIOS-level settings (VT-d) or require an environment free of unexpected conflicts with existing PCIe devices.

---

## **4. Requirements**

### **4.1 Hardware**

1. **Donor PCIe Device**  
   - Purpose: You’ll extract the vendor/device ID, subsystem IDs, class code, BAR size, and capabilities.  
   - Examples: An older network interface card (NIC), a basic storage controller, or even a specialized PCIe device that you want to replicate/extend.

2. **DMA FPGA Card**  
   - Purpose: The actual hardware platform that runs the FPGA logic implementing the PCIe interface.  
   - Examples: Squirrel 35T, Enigma-X1, or ZDMA 100T boards.

3. **JTAG Programmer**  
   - Connects to the JTAG pins on your FPGA board, letting Vivado load the synthesized bitstream or debug firmware in real time.

### **4.2 Software**

1. **Xilinx Vivado Design Suite**  
   - Required for creating, synthesizing, and implementing the FPGA design.  
   - Download from [Xilinx](https://www.xilinx.com/support/download.html), ensuring you pick the correct version for your board’s IP requirements.

2. **Visual Studio Code**  
   - A flexible, cross-platform editor supporting Verilog/SystemVerilog with additional plugins.  
   - Helps maintain consistent code style, track changes, and streamline collaboration with version control (Git).

3. **PCILeech-FPGA**  
   - GitHub repository: [PCILeech-FPGA](https://github.com/ufrisk/pcileech-fpga).  
   - Offers baseline DMA designs for various FPGA boards, which you can further customize to replicate your donor device’s PCIe configuration.

4. **Arbor** (PCIe Device Scanner)  
   - A user-friendly GUI tool that provides in-depth analysis of connected PCIe devices.  
   - Alternatives: Telescan PE for traffic capture, or `lspci -vvv` in Linux for command-line introspection.

### **4.3 Environment Setup**

1. **Installing Vivado**  
   - Follow Xilinx’s official installer, selecting the appropriate FPGA family (Artix-7, Kintex-7, etc.).  
   - A Xilinx account may be required to download the Design Suite or access updates.

2. **Installing Visual Studio Code**  
   - Download from [Visual Studio Code](https://code.visualstudio.com/).  
   - Install recommended plugins: *Verilog-HDL/SystemVerilog* and a *Git* integration if you plan to maintain your project under source control.

3. **Cloning PCILeech-FPGA**  
   ```bash
   cd ~/Projects/
   git clone https://github.com/ufrisk/pcileech-fpga.git
   cd pcileech-fpga
   ```
   - Ensure you have Git installed and configured.

4. **Isolated Development Environment**  
   - Consider using a dedicated test machine (or a carefully configured dual-boot/VM) to reduce risk.  
   - This approach allows you to disable kernel DMA protections, IOMMU, or secure boot features more freely, without compromising a primary production system.

---

## **5. Gathering Donor Device Information**

Emulating a device effectively requires replicating its PCIe configuration space. That means capturing everything from device/vendor IDs to advanced capabilities.

### **5.1 Using Arbor for PCIe Device Scanning**

#### **5.1.1 Install & Launch Arbor**

1. **Obtain Arbor**  
   - Register and download from the official Arbor site.  
   - Install with administrator rights.

2. **Start Arbor**  
   - If prompted by UAC on Windows, confirm the application can run with elevated privileges.  
   - You should see an interface listing PCI/PCIe devices.

#### **5.1.2 Scan for Devices**

1. **Local System Tab**  
   - Navigate to the “Local System” or “Scan” area in Arbor.  
2. **Click “Scan”**  
   - Arbor enumerates all devices on your PCIe bus.  
3. **Identify the Donor**  
   - Match the brand name or vendor ID to your known donor hardware. If it’s not easily recognized, cross-reference with known IDs from hardware documentation.

#### **5.1.3 Extract Key Attributes**

Collect the following from Arbor’s detailed view:

- **Vendor ID / Device ID**: e.g., 0x8086 / 0x10D3 (Intel NIC).  
- **Subsystem Vendor ID / Subsystem ID**: e.g., 0x8086 / 0xA02F.  
- **Revision ID**: e.g., 0x01.  
- **Class Code**: e.g., 0x020000 for an Ethernet controller.  
- **BARs (Base Address Registers)**:  
  - For each BAR, note if it’s enabled, the memory size (256 MB, 64 KB, etc.), and whether it’s prefetchable or 32-bit/64-bit.  
- **Capabilities**:  
  - MSI or MSI-X details (number of interrupt vectors supported).  
  - Extended configuration or advanced power management features.  
- **Device Serial Number (DSN)** (if present):  
  - Some devices have a unique DSN field, especially if used for licensing or special driver checks.

> **Organization Tip**: Use a spreadsheet or structured document to save these values. This ensures you don’t overlook details like advanced features or extended capabilities.

---

## **6. Initial Firmware Customization**

With the donor’s PCIe attributes in hand, begin customizing your FPGA firmware to match those settings.

### **6.1 Modifying the PCIe Configuration Space**

Your FPGA design likely includes a top-level file that sets the PCIe configuration registers. For example, in the `pcileech-fpga` repository, look for a file such as `pcileech_pcie_cfg_a7.sv` or `pcie_7x_0_core_top.v`.

1. **Open File in VS Code**  
   - Search for lines defining `cfg_deviceid`, `cfg_vendorid`, `cfg_subsysid`, etc.

2. **Assign the Correct IDs**  
   ```verilog
   cfg_deviceid        <= 16'h10D3; // Example device ID
   cfg_vendorid        <= 16'h8086; // Example vendor ID
   cfg_subsysid        <= 16'h1234;
   cfg_subsysvendorid  <= 16'h5678;
   cfg_revisionid      <= 8'h01;
   cfg_classcode       <= 24'h020000; // Example for Ethernet
   ```
   - Replace these with the exact values from Arbor (or your donor’s datasheet).

3. **Insert DSN If Needed**  
   ```verilog
   cfg_dsn             <= 64'h0011223344556677;
   ```
   - Omit or set to 0 if your donor device doesn’t rely on a DSN.

4. **Save & Review**  
   - A single-digit error in any field could cause the OS to misidentify or reject the device. Double-check each line.

### **6.2 Consider BAR Configuration**

While some PCIe IP cores store BAR settings in the same SystemVerilog file, others rely on Vivado’s IP customization GUI:

- **Check how many BARs** your donor device uses (0 to 6).  
- **Set each BAR** (e.g., memory type, size, prefetchable, 64-bit vs. 32-bit).  
- If your donor has a large BAR region (e.g., 256 MB or bigger), ensure your FPGA board can accommodate it in the IP core settings.

---

## **7. Vivado Project Setup and Customization**

### **7.1 Generating Vivado Project Files**

To organize all design files properly, many repositories include Tcl scripts:

1. **Launch Vivado**  
   - Confirm you are using the correct version for your FPGA series (Artix-7, Kintex-7, etc.).

2. **Open Tcl Console**  
   - **Window > Tcl Console** in Vivado’s top menu.

3. **Navigate to Project Directory**  
   ```tcl
   cd C:/path/to/pcileech-fpga/pcileech-wifi-main/
   pwd
   ```
   - Confirm with `pwd` that the console is in the correct folder.

4. **Run the Generation Script**  
   ```tcl
   source vivado_generate_project_squirrel.tcl -notrace
   ```
   - If you’re using Enigma-X1 or ZDMA, run the corresponding script (e.g., `vivado_generate_project_enigma_x1.tcl`).

5. **Open the Generated Project**  
   - **File > Open Project**. Locate and select the `.xpr` (e.g., `pcileech_squirrel_top.xpr`).  
   - Check the **Project Manager** window for properly imported sources.

### **7.2 Customizing the PCIe IP Core**

In Vivado, you may find a PCIe IP core (e.g., `pcie_7x_0.xci`) under **Sources**:

1. **Right-Click -> Customize IP**  
   - Update vendor/device IDs, revision, and subsystem fields.  
   - Match your desired BAR configurations (sizes, memory type, etc.).

2. **Generate/Update the IP**  
   - Click **OK** or **Generate** to rebuild.  
   - Vivado might prompt you to upgrade or confirm dependencies if IP versions have changed.

3. **Lock the IP Core**  
   ```tcl
   set_property -name {IP_LOCKED} -value true -objects [get_ips pcie_7x_0]
   ```
   - This prevents future scripts from overwriting your manual changes inadvertently.

---

## **Additional Best Practices**

1. **Version Control** - *Highly Recommended*  
   - Commit your changes often (Git or another SCM).  
   - Tag or branch major changes so you can revert quickly if something breaks.

2. **Documentation**  
   - Keep a notebook, spreadsheet, or wiki summarizing donor device details, any special offsets, or capability quirks.  
   - Document each step you take in customizing your FPGA firmware.

3. **Testing on the Host**  
   - After generating a bitstream, program the FPGA, then check:  
     - **Windows**: Device Manager or `devcon.exe` to confirm the device enumerates with the correct IDs.  
     - **Linux**: `lspci -vvv` to see if the device identifies correctly, including BAR, Class Code, Subsystem, etc.

4. **Security Considerations**  
   - Disabling features like VT-d or Secure Boot can open up the system to vulnerabilities. Use a dedicated test rig or isolate the environment to maintain operational security.

5. **Where to Next?**  
   - In **Part 2**, you will learn to build on these basics with deeper TLP manipulation, partial reconfiguration strategies, firmware debugging, and any advanced ID spoofing or handshake emulations.
