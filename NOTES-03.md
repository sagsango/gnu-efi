# call flow x86
Power-on
 └─> Reset Vector (0xFFFFFFF0)
       └─> SEC_Entry() → SecMain()
             └─> PeiCore() → BuildHOBs
                   └─> DXE IPL → Load DxeCore
                         └─> DxeMain()
                               ├─ Create EFI_SYSTEM_TABLE
                               ├─ Init BootServices + RuntimeServices
                               ├─ Install ConfigurationTables (ACPI, SMBIOS, etc.)
                               └─ Launch BDS
                                     └─ Boot#### → LoadImage() / StartImage()
                                           └─> _start (crt0-efi-x86_64.S)
                                                 └─> efi_main()
                                                       ├─ InitializeLib()
                                                       ├─ Use BS/RT/ST
                                                       ├─ Load kernel
                                                       └─ ExitBootServices()
                                                             └─→ OS Kernel


# Source File Mapping (gnu-efi + firmware) 
| Layer           | File                              | Purpose                    |
| --------------- | --------------------------------- | -------------------------- |
| Firmware SEC    | `SecMain.c`, `ResetVector.asm`    | Reset entry                |
| PEI             | `PeiMain.c`                       | Build HOBs, discover DRAM  |
| DXE Core        | `DxeMain.c`                       | Create BS, RT, ST tables   |
| DXE Drivers     | `AcpiTableDxe.c`, `SmbiosDxe.c`   | Install ConfigTables       |
| Boot Manager    | `BdsEntry.c`                      | Boot#### logic             |
| Image Loader    | `Image.c`                         | LoadImage() / StartImage() |
| gnu-efi         | `crt0-efi-x86_64.S`, `lib/init.c` | Runtime glue               |
| User bootloader | `efi_main()`                      | Entrypoint code            |



# Visual Timeline Summary
───────────────────────────────────────────────────────────────────────-------------------------+
| PHASE  | ENTRY FUNC      | MAIN WORK                          | OUTPUT / NEXT CALL            |
|--------|-----------------|------------------------------------|-------------------------------|
| SEC    | SecMain()       | Cache-as-RAM, locate PEI           | PeiCore()                     |
| PEI    | PeiCore()       | Init DRAM, build HOBs              | DxeCore()                     |
| DXE    | DxeMain()       | Create ST, BS, RT, ConfigTables    | BdsEntry()                    |
| BDS    | BdsEntry()      | Load Boot#### entry                | LoadImage()                   |
| BS     | LoadImage()     | Relocate .efi image                | StartImage()                  |
| CRT0   | _start          | Setup stack + call efi_main()      | efi_main()                    |
| USER   | efi_main()      | Bootloader logic                   | ExitBootServices()            |
| OS     | Kernel entry    | Initialize MMU/ACPI/etc.           | RuntimeServices only          |
───────────────────────────────────────────────────────────────────────-------------------------+


# master diagram
──────────────────────────────────────────────────────────────────────────────
 STAGE 0 — POWER-ON / CPU RESET
──────────────────────────────────────────────────────────────────────────────
CPU:
  CS=0xF000, IP=0xFFF0 → physical 0xFFFFFFF0
  Mode = 16-bit real mode
  Paging = Off, Cache = Disabled

Firmware Flash (BIOS region):
  └── ResetVector (ResetVector.asm)
         │
         ▼
     +-------------------+
     | SEC_Entry()       |
     |  (from SecMain.c) |
     +-------------------+
         │
         │ Initialize temporary stack (cache-as-RAM)
         │ Find PEI Core FV in flash
         │ Create SecHandOff structure:
         │   - TempRamBase/Size
         │   - PeiCoreEntryPoint
         │   - BootMode
         ▼
     SwitchStack(PeiCoreEntryPoint, SecHandOff)
         │
         ▼
──────────────────────────────────────────────────────────────────────────────
 STAGE 1 — PEI PHASE (Pre-EFI Init)
──────────────────────────────────────────────────────────────────────────────
[Entry] → PeiCore(SecHandOff)
   │
   ├─ Initialize DRAM (MemoryInitPeim.c)
   ├─ Build HOB (Hand-Off Block) List
   │     [PHIT, CPU, FV, Memory, NVS, END]
   ├─ Install PEI services (PPI database)
   ├─ Discover DXE IPL PEIM
   └─ HandOffToDxeIpl()

          ↓
──────────────────────────────────────────────────────────────────────────────
 STAGE 2 — DXE IPL (Initial Program Loader)
──────────────────────────────────────────────────────────────────────────────
HandOffToDxeIpl():
   │
   ├─ Load DXE Core image (PE/COFF from FV)
   ├─ Parse headers, allocate pages
   ├─ Relocate image
   └─ Jump to DxeMain(HobList)

          ↓
──────────────────────────────────────────────────────────────────────────────
 STAGE 3 — DXE CORE (Main EFI Foundation)
──────────────────────────────────────────────────────────────────────────────
DxeMain(HobList):
   │
   ├─ Initialize Boot Services table  (InitializeBootServices)
   │
   ├─ Initialize Runtime Services table (InitializeRuntimeServices)
   │
   ├─ Create System Table (InstallSystemTable)
   │     EFI_SYSTEM_TABLE {
   │       Hdr, FirmwareVendor, Revision,
   │       ConIn/ConOut handles,
   │       BootServices, RuntimeServices,
   │       ConfigurationTable[]
   │     }
   │
   ├─ InstallProtocolDatabase()
   ├─ Dispatch DXE drivers (PCI, BlockIo, FS, GOP, Console)
   ├─ Install ACPI/SMBIOS/FDT entries:
   │     InstallConfigurationTable(&ACPI_20_TABLE_GUID, RSDP);
   │     InstallConfigurationTable(&SMBIOS_TABLE_GUID, Smbios);
   │     InstallConfigurationTable(&EFI_FDT_GUID, FdtBlob);
   └─ Launch BDS Entry (Boot Device Selection)

          ↓
──────────────────────────────────────────────────────────────────────────────
 STAGE 4 — BDS PHASE (Boot Manager)
──────────────────────────────────────────────────────────────────────────────
BdsEntry():
   │
   ├─ Read BootOrder[] / Boot#### vars from NVRAM
   ├─ Select Boot#### option
   ├─ Use SIMPLE_FILE_SYSTEM_PROTOCOL to open device
   ├─ Load image file (e.g. \EFI\BOOT\BOOTX64.EFI)
   │     BS->LoadImage(FALSE, parent, FilePath, NULL, 0, &ImageHandle)
   ├─ Apply relocations (via reloc_x86_64.c)
   └─ Start image:
         BS->StartImage(ImageHandle, NULL, NULL)

          ↓
──────────────────────────────────────────────────────────────────────────────
 STAGE 5 — IMAGE LOAD / BOOT SERVICES
──────────────────────────────────────────────────────────────────────────────
LoadImage():
   │
   ├─ Parse PE/COFF headers (Image.c)
   ├─ Allocate pages for CODE + DATA sections
   ├─ Apply relocations for 64-bit (reloc_x86_64.c)
   ├─ Install LoadedImageProtocol on ImageHandle
   └─ Return ImageHandle

StartImage():
   │
   └─ Calls → Image->EntryPoint(ImageHandle, SystemTable)

          ↓
──────────────────────────────────────────────────────────────────────────────
 STAGE 6 — IMAGE ENTRY (crt0-efi-x86_64.S)
──────────────────────────────────────────────────────────────────────────────
_start:
   movq %rdi, ImageHandle
   movq %rsi, SystemTable
   call  efi_main
   movq  %rax, %rdi
   call  Exit  ; Return to firmware if desired

          ↓
──────────────────────────────────────────────────────────────────────────────
 STAGE 7 — LIB INITIALIZATION (gnu-efi/lib/init.c)
──────────────────────────────────────────────────────────────────────────────
efi_main(EFI_HANDLE image, EFI_SYSTEM_TABLE *systab)
   │
   └─ InitializeLib(image, systab):
          ST = systab;
          BS = systab->BootServices;
          RT = systab->RuntimeServices;

          BS->HandleProtocol(image, &LoadedImageProtocol, &LI);
          BS->HandleProtocol(LI->DeviceHandle, &DevicePathProtocol, &DP);
          → global variables now hold firmware interfaces

          ↓
──────────────────────────────────────────────────────────────────────────────
 STAGE 8 — BOOTLOADER LOGIC (User Code)
──────────────────────────────────────────────────────────────────────────────
Example:
   Print(L"Bootloader Start\n");

   // Inspect EFI tables
   VOID *rsdp;
   EfiGetSystemConfigurationTable(&ACPI_20_TABLE_GUID, &rsdp);

   // Get memory map
   BS->GetMemoryMap(&map_size, map, &key, &desc_size, &desc_ver);

   // Locate GOP, BlockIo, FS, etc.
   BS->LocateProtocol(&GraphicsOutputProtocolGuid, NULL, &gop);

   // Load next-stage kernel image
   LoadKernel();
   SetupPageTables();

          ↓
──────────────────────────────────────────────────────────────────────────────
 STAGE 9 — EXIT BOOT SERVICES
──────────────────────────────────────────────────────────────────────────────
BS->ExitBootServices(ImageHandle, key);
   │
   ├─ Disables interrupts
   ├─ Stops timers/events
   ├─ Invalidates BootServices memory regions
   └─ Leaves RuntimeServices + ConfigTables active

Memory now frozen for OS handoff.

          ↓
──────────────────────────────────────────────────────────────────────────────
 STAGE 10 — OS LOADER / KERNEL HANDOFF
──────────────────────────────────────────────────────────────────────────────
Bootloader prepares boot params:
   • Passes SystemTable pointer to kernel
   • Passes memory map and ACPI/FDT pointers
   • CPU in 64-bit long mode
   • CR0.PG=1, CR4.PAE=1, EFER.LME=1
   • Interrupts disabled
   • Stack valid

Kernel (Linux example):
   - Reads efi.systab
   - Finds ACPI RSDP or FDT blob
   - Maps RuntimeServices regions
   - Initializes EFI framebuffer (GOP)
   - Continues boot

          ↓
──────────────────────────────────────────────────────────────────────────────
 STAGE 11 — EFI RUNTIME PERIOD
──────────────────────────────────────────────────────────────────────────────
Now only Runtime Services remain callable:
   RT->GetVariable()
   RT->SetVariable()
   RT->GetTime()
   RT->ResetSystem()
   RT->SetVirtualAddressMap()

ACPI + SMBIOS tables stay in EfiRuntime/ACPI memory
OS uses them for power mgmt, enumeration, etc.

──────────────────────────────────────────────────────────────────────────────




# simplified
ResetVector
 └─> SecMain()
       └─> PeiCore()
             └─> HandOffToDxeIpl()
                   └─> DxeMain()
                         ├─> InitializeBootServices()
                         ├─> InitializeRuntimeServices()
                         ├─> InstallSystemTable()
                         ├─> InstallConfigurationTable()
                         └─> BdsEntry()
                               └─> BS->LoadImage()
                                     └─> BS->StartImage()
                                           └─> _start (crt0-efi-x86_64.S)
                                                 └─> efi_main()
                                                       └─> InitializeLib()
                                                             └─> Bootloader code
                                                                   └─> BS->ExitBootServices()
                                                                         └─> OS kernel

