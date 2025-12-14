# From Power-On → Bootloader Entry (efi_main())
──────────────────────────────────────────────────────────────────────────────
 STAGE 0 — POWER-ON / CPU RESET
──────────────────────────────────────────────────────────────────────────────
CPU RESET VECTOR: CS=0xF000, IP=0xFFF0 (physical 0xFFFFFFF0)
    │
    ▼
[SEC (Security Phase) Firmware in SPI Flash]
    - Minimal 16-bit real mode startup
    - Initializes temporary stack (cache-as-RAM)
    - Loads PEI core from flash volume
    - Establishes "handoff" structure with:
         → Temporary RAM base
         → Boot mode
         → PEI core entry address
    - Switches to flat 32/64-bit mode (if supported)
    │
    ▼
──────────────────────────────────────────────────────────────────────────────
 STAGE 1 — PEI (Pre-EFI Initialization)
──────────────────────────────────────────────────────────────────────────────
Runs from cache-as-RAM, then main DRAM once discovered.

[PEI Core]
    ├─ Initialize main memory (DRAM init via MRC/IMC)
    ├─ Build HOB (Hand-Off Block) list:
    │    • PHIT HOB (physical memory ranges)
    │    • CPU HOB
    │    • Firmware Volume HOBs
    │    • ACPI/NVS HOBs
    │    • End-of-HOB marker
    ├─ Discover PEI Modules (PEIMs)
    ├─ Call PEI Services (PPIs)
    └─ Hand control to DXE IPL (next stage)
    │
    ▼
──────────────────────────────────────────────────────────────────────────────
 STAGE 2 — DXE (Driver Execution Environment)
──────────────────────────────────────────────────────────────────────────────
Full 32/64-bit mode, DRAM ready, paging optional.

[DXE Core]
    ├─ Create EFI_SYSTEM_TABLE (global root table)
    │
    ├─ Initialize BOOT_SERVICES and RUNTIME_SERVICES tables
    │
    ├─ Install them into SystemTable
    │
    ├─ Create EFI_HANDLE database
    │
    ├─ Enumerate drivers from firmware volumes (FV)
    │    → PCI Bus, Graphics, Block IO, Console, etc.
    │
    ├─ Populate EFI_CONFIGURATION_TABLE[]
    │    with firmware data structures (ACPI, SMBIOS, etc.)
    │
    └─ Transfer control to BDS (Boot Device Selection)
    │
    ▼
──────────────────────────────────────────────────────────────────────────────
 STAGE 3 — BDS (Boot Manager)
──────────────────────────────────────────────────────────────────────────────
Boot Device Selection / Boot#### logic

[BDS Phase]
    ├─ Reads NVRAM variables: BootOrder[], BootNext, Boot#### entries
    ├─ Locates EFI_SIMPLE_FILE_SYSTEM_PROTOCOL devices (FAT partitions)
    ├─ Loads the file indicated by Boot#### (e.g. \EFI\BOOT\BOOTX64.EFI)
    ├─ Calls BootServices->LoadImage()
    │    → Reads PE/COFF headers, allocates memory, applies relocations
    │    → Creates EFI_HANDLE for the new image
    ├─ Calls BootServices->StartImage()
    │    → Invokes _start() in the .efi binary
    └─ Hands control to bootloader (gnu-efi, grub, etc.)





# The EFI_SYSTEM_TABLE is built by the DXE core.
It is the single root object describing the entire firmware environment.

+---------------------------------------------------------------+
| EFI_SYSTEM_TABLE (ST)                                         |
|---------------------------------------------------------------|
| Hdr.Signature = "IBI SYST"                                    |
| FirmwareVendor = "American Megatrends"                        |
| FirmwareRevision = 0x00020001                                 |
| ConsoleInHandle / ConIn  → SIMPLE_TEXT_INPUT_PROTOCOL          |
| ConsoleOutHandle / ConOut→ SIMPLE_TEXT_OUTPUT_PROTOCOL         |
| StdErrHandle / StdErr    → SIMPLE_TEXT_OUTPUT_PROTOCOL         |
| BootServices             → EFI_BOOT_SERVICES table             |
| RuntimeServices          → EFI_RUNTIME_SERVICES table          |
| NumberOfTableEntries     = N                                   |
| ConfigurationTable       → Array of (GUID, pointer)            |
+---------------------------------------------------------------+




# EFI_BOOT_SERVICES TABLE
Firmware functions available before ExitBootServices():
After ExitBootServices(), this table becomes invalid.

EFI_BOOT_SERVICES
 ├─ Memory Allocation
 │    • AllocatePages()
 │    • FreePages()
 │    • GetMemoryMap()
 │    • AllocatePool()
 │    • FreePool()
 │
 ├─ Event & Timer
 │    • CreateEvent(), WaitForEvent(), SetTimer()
 │
 ├─ Protocol Management
 │    • InstallProtocolInterface()
 │    • HandleProtocol()
 │    • LocateProtocol()
 │    • RegisterProtocolNotify()
 │
 ├─ Image Services
 │    • LoadImage()
 │    • StartImage()
 │    • Exit()
 │    • UnloadImage()
 │
 ├─ Driver Support
 │    • ConnectController()
 │    • DisconnectController()
 │
 ├─ Misc
 │    • Stall(), SetWatchdogTimer(), OpenProtocol()
 │
 └─ Termination
      • ExitBootServices()


# EFI_RUNTIME_SERVICES TABLE
These survive into OS runtime and are callable after boot (through RT).

EFI_RUNTIME_SERVICES
 ├─ Variable Services
 │    • GetVariable()
 │    • SetVariable()
 │    • GetNextVariableName()
 │
 ├─ Time Services
 │    • GetTime()
 │    • SetTime()
 │    • GetWakeupTime()
 │
 ├─ Virtual Memory Mapping
 │    • SetVirtualAddressMap()
 │
 ├─ Reset and Firmware Update
 │    • ResetSystem()
 │    • UpdateCapsule()
 │
 └─ Misc
      • QueryCapsuleCapabilities()



# EFI_CONFIGURATION_TABLE[]
An array pointed to by SystemTable->ConfigurationTable, containing static data
the OS or bootloader needs.

EFI_CONFIGURATION_TABLE[]
 ├─ [0]  ACPI_20_TABLE_GUID  → 0x000000007B000000 (RSDP)
 ├─ [1]  SMBIOS_TABLE_GUID   → 0x000000007B100000 (_SM_ anchor)
 ├─ [2]  MPS_TABLE_GUID      → 0x000000007B200000
 ├─ [3]  SAL_SYSTEM_TABLE_GUID (IA64 only)
 ├─ [4]  EFI_FDT_GUID        → 0x000000007B300000 (Device Tree Blob)
 ├─ [5]  EFI_TDT_GUID        → 0x000000007B400000 (TCG Event Log)
 └─ [N]  Vendor-Specific GUIDs (OEM, Platform Info)


# EFI TABLE MEMORY MAP CONTEXT
+──────────────────────────────────────────────+
| Physical Memory Map Before ExitBootServices  |
|----------------------------------------------|
| 00000000-00000FFF : EfiReservedMemoryType    |
| 00001000-0009FFFF : EfiLoaderCode/Data       |
| 000A0000-000FFFFF : EfiConventionalMemory    |
| 00100000-00FFFFFF : EfiBootServicesCode/Data |
| 01000000-010FFFFF : EfiRuntimeServicesData   |
| 01100000-011FFFFF : EfiRuntimeServicesCode   |
| 7B000000-7B01FFFF : EfiACPIReclaimMemory     ← ACPI RSDP/XSDT/FADT etc.     |
| 7B020000-7B02FFFF : EfiACPIMemoryNVS         ← NVS/SCI Data                |
| 7B030000-7B03FFFF : EfiRuntimeServicesData   ← SMBIOS / FDT                |
| 7B040000-7B04FFFF : EfiBootServicesData      ← GOP framebuffer / handles   |
+──────────────────────────────────────────────+



# BOOT MANAGER → BOOTLOADER TRANSFER
[Boot#### entry example]
  Description: "GRUB EFI"
  FilePath:    \EFI\GRUB\GRUBX64.EFI
  Attributes:  ACTIVE

BDS loads GRUBX64.EFI:
  • Reads file via SIMPLE_FILE_SYSTEM_PROTOCOL
  • Calls BS->LoadImage()
  • Applies relocations via reloc_x86_64.c
  • Creates ImageHandle
  • Calls BS->StartImage(ImageHandle)
       ↓
_start (crt0-efi-x86_64.S)
       ↓
efi_main(EFI_HANDLE ImageHandle, EFI_SYSTEM_TABLE *SystemTable)



# BOOTLOADER EXECUTION CONTEXT
efi_main():
   InitializeLib(ImageHandle, SystemTable);
   Print(L"Bootloader start...\n");
   ST, BS, RT = SystemTable->...;
   BS->GetMemoryMap(&map, &key, &desc, &desc_size);
   BS->LocateProtocol(&GraphicsOutputProtocolGuid, NULL, &gop);
   ...
   BS->ExitBootServices(ImageHandle, key);
   → switch to OS boot path (load kernel)



# EXIT BOOT SERVICES HANDOFF
After this, firmware shuts down:
    No Boot Services calls
    No new protocol handles
    Interrupts and timers disabled
    Only RuntimeServices and configuration tables remain mapped.
Linux, Windows, or your own OS will:
    Record memory map
    Parse SystemTable->ConfigurationTable
    Get ACPI RSDP or FDT pointer
    Transition to its own page tables and drivers


# FINAL STRUCTURE OVERVIEW
──────────────────────────────────────────────────────────────────────────────
                    COMPLETE UEFI STRUCTURE MAP
──────────────────────────────────────────────────────────────────────────────
 EFI_SYSTEM_TABLE (ST)
 ├── BootServices (BS)
 │     ├── AllocatePages()
 │     ├── HandleProtocol()
 │     └── ExitBootServices()
 │
 ├── RuntimeServices (RT)
 │     ├── GetVariable()
 │     ├── SetTime()
 │     └── ResetSystem()
 │
 └── ConfigurationTable[]
        ├── ACPI_20_TABLE_GUID → [RSDP] → XSDT → FADT, DSDT, MADT, etc.
        ├── SMBIOS_TABLE_GUID  → [DMI]
        ├── EFI_FDT_GUID       → [Device Tree Blob]
        ├── MPS_TABLE_GUID     → [MP Table]
        ├── EFI_TDT_GUID       → [TCG Event Log]
        └── OEM / Vendor GUIDs
──────────────────────────────────────────────────────────────────────────────



