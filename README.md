# PCILeech DMA Firmware Development Guide

## Project Status & Personal Update

I sincerely apologize for my recent hiatus; I've been navigating some significant personal challenges. I'm aware that there may have been developments in the interim, and I am actively working to get up to speed and will share relevant findings. I'll do my best to respond to messages as my capacity allows. Your understanding and patience are greatly appreciated.

## Overview

This repository is dedicated to providing the documentation, tools, and code examples necessary to create custom firmware. This firmware enables Field-Programmable Gate Array (FPGA) devices to accurately emulate PCI Express (PCIe) hardware. By leveraging this guide and the provided resources, developers can unlock advanced capabilities for various applications.

### Key Capabilities Enabled by Custom DMA Firmware:

*   Hardware Testing & Development: Facilitate rigorous testing and rapid prototyping of hardware components.
*   System Debugging & Diagnostics: Gain deep insights into system behavior for effective troubleshooting.
*   Security Research & Malware Analysis: Investigate system vulnerabilities and analyze sophisticated threats at the hardware level.
*   Hardware Emulation & Legacy Support: Emulate specific PCIe devices or provide support for older hardware.

## Hardware Requirements

To successfully develop and deploy custom DMA firmware, you'll need:

*   FPGA-based DMA Device: Compatible options include Squirrel 35T, Enigma-X1 75T, ZDMA 100T, or Kintex-7 based boards.
*   JTAG Programmer: For flashing the compiled firmware onto the FPGA.
*   Host System: A computer with an available PCIe slot for installing the FPGA device.
*   Donor PCIe Device (Optional but Recommended): A legitimate PCIe device from which to extract identification information (Vendor ID, Device ID, Subsystem ID, etc.) to make your custom firmware appear more authentic to the host system.

## Software Requirements

Ensure you have the following software installed:

*   Xilinx Vivado Design Suite: The primary tool for FPGA development, synthesis, and implementation. (Specify version if critical, e.g., Vivado 2019.1 or later).
*   Text Editor/IDE: Visual Studio Code, Sublime Text, or any preferred editor for code development.
*   PCIe Device Scanning Tools: Utilities like RWEverything, AIDA64, HWInfo, lspci (Linux), Arbor, or Telescan PE for inspecting PCIe device information.

## Repository Structure

```
.
├── .github/ # GitHub-specific files (workflows, issue templates)
├── docs/ # Documentation and guides
│ ├── CN/ # Chinese translations of documentation
│ └── Guide v3.5.md # Comprehensive firmware development guide
├── examples/ # Example implementations and code samples
├── src/ # Source code and core implementation
│ └── pcileech-wifi-main/ # Reference implementation for PCILeech firmware
│ ├── ip/ # IP cores and configurations
│ │ ├── 100t/ # Components specific to 100T FPGAs
│ │ └── ... # Other FPGA-specific IP components
│ ├── pcie_7x/ # PCIe interface components (e.g., for 7-series FPGAs)
│ │ └── zdma/ # ZDMA-specific components
│ └── src/ # Core firmware source code
└── tools/ # Utility scripts and helper tools
```

## Getting Started

1.  Clone the Repository:

    ```bash
    git clone git@github.com:JPShag/PCILEECH-DMA-FW-Guide-2.0.git
    cd <repository-directory>
    ```

2.  Review the Documentation: Thoroughly read the `docs/Guide v3.5.md`. This guide is crucial for understanding the firmware development process, architecture, and best practices.

3.  Set Up Your Environment: Ensure all [Software Requirements](#software-requirements) are installed and correctly configured.

4.  Explore the Source Code: Examine the `src/` directory, particularly the `pcileech-wifi-main/` reference implementation, to understand the structure and coding patterns.

5.  Study the Examples: Check the `examples/` directory for practical code samples that can serve as a starting point or inspiration for your custom firmware.

6.  Hardware Setup: Prepare your [FPGA device](#hardware-requirements) and JTAG programmer.

## Looking for DMA Hardware?

For purchasing compatible FPGA-based DMA hardware, you can visit: [DMAPolice.com](https://dmapolice.com/)

## Support and Contact

If you require assistance or have questions, please submit an issue on GitHub. Due to ongoing personal circumstances, response times may be delayed. Your patience is highly appreciated.

## Contributing

Contributions to this project are welcome! Please read our `CONTRIBUTING.md` file (if available) for guidelines on how to submit pull requests, report issues, and suggest improvements. (Consider creating a `CONTRIBUTING.md` file).

## License

It's highly recommended to add a `LICENSE` file to your repository (e.g., MIT, GPL, Apache 2.0). This clarifies how others can use and contribute to your work. (Example: This project is licensed under the MIT License - see the `LICENSE` file for details.)

## Donations

If you find this guide and the accompanying resources valuable, consider supporting its development:

*   LTC: `MPMyQD5zgy2b2CpDn1C1KZ31KmHpT7AwRi`