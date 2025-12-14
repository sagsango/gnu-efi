gnu-efi ❱❱❱ tree -d
.
├── apps
├── gnuefi
├── inc
│   ├── aarch64
│   ├── arm
│   ├── ia32
│   ├── ia64
│   ├── loongarch64
│   ├── mips64el
│   ├── protocol
│   │   └── ia64
│   ├── riscv64
│   └── x86_64
└── lib
    ├── aarch64
    ├── arm
    ├── ia32
    ├── ia64
    ├── loongarch64
    ├── mips64el
    ├── riscv64
    ├── runtime
    └── x86_64

24 directories




gnu-efi ❱❱❱ tree
.
├── apps
│   ├── AllocPages.c
│   ├── bltgrid.c
│   ├── ctors_dtors_priority_test.c
│   ├── ctors_fns.c
│   ├── ctors_test.c
│   ├── debughook.c
│   ├── drv0_use.c
│   ├── drv0.c
│   ├── drv0.h
│   ├── exit.c
│   ├── FreePages.c
│   ├── lfbgrid.c
│   ├── Makefile
│   ├── modelist.c
│   ├── printenv.c
│   ├── route80h.c
│   ├── setdbg.c
│   ├── setjmp.c
│   ├── t.c
│   ├── t2.c
│   ├── t3.c
│   ├── t4.c
│   ├── t5.c
│   ├── t6.c
│   ├── t7.c
│   ├── t8.c
│   ├── tcc.c
│   ├── tpause.c
│   ├── trivial.S
│   └── unsetdbg.c
├── ChangeLog
├── gnuefi
│   ├── crt0-efi-aarch64.S
│   ├── crt0-efi-arm.S
│   ├── crt0-efi-ia32.S
│   ├── crt0-efi-ia64.S
│   ├── crt0-efi-loongarch64.S
│   ├── crt0-efi-mips64el.S
│   ├── crt0-efi-riscv64.S
│   ├── crt0-efi-x86_64.S
│   ├── elf_aarch64_efi.lds
│   ├── elf_arm_efi.lds
│   ├── elf_ia32_efi.lds
│   ├── elf_ia32_fbsd_efi.lds
│   ├── elf_ia64_efi.lds
│   ├── elf_loongarch64_efi.lds
│   ├── elf_mips64el_efi.lds
│   ├── elf_riscv64_efi.lds
│   ├── elf_x86_64_efi.lds
│   ├── elf_x86_64_fbsd_efi.lds
│   ├── gnu-efi.pc.in
│   ├── Makefile
│   ├── reloc_aarch64.c
│   ├── reloc_arm.c
│   ├── reloc_ia32.c
│   ├── reloc_ia64.S
│   ├── reloc_loongarch64.c
│   ├── reloc_mips64el.c
│   ├── reloc_riscv64.c
│   └── reloc_x86_64.c
├── inc
│   ├── aarch64
│   │   ├── efibind.h
│   │   ├── efilibplat.h
│   │   └── efisetjmp_arch.h
│   ├── arm
│   │   ├── efibind.h
│   │   ├── efilibplat.h
│   │   └── efisetjmp_arch.h
│   ├── efi_nii.h
│   ├── efi_pxe.h
│   ├── efi.h
│   ├── efiapi.h
│   ├── eficompiler.h
│   ├── eficon.h
│   ├── eficonex.h
│   ├── efidebug.h
│   ├── efidef.h
│   ├── efidevp.h
│   ├── efierr.h
│   ├── efifs.h
│   ├── efigpt.h
│   ├── efiip.h
│   ├── efilib.h
│   ├── efilink.h
│   ├── efinet.h
│   ├── efipart.h
│   ├── efipciio.h
│   ├── efipoint.h
│   ├── efiprot.h
│   ├── efipxebc.h
│   ├── efirtlib.h
│   ├── efiser.h
│   ├── efisetjmp.h
│   ├── efishell.h
│   ├── efishellintf.h
│   ├── efistdarg.h
│   ├── efitcp.h
│   ├── efiudp.h
│   ├── efiui.h
│   ├── ia32
│   │   ├── efibind.h
│   │   ├── efilibplat.h
│   │   ├── efisetjmp_arch.h
│   │   └── pe.h
│   ├── ia64
│   │   ├── efibind.h
│   │   ├── efilibplat.h
│   │   ├── efisetjmp_arch.h
│   │   ├── pe.h
│   │   └── salproc.h
│   ├── inc.mak
│   ├── lib.h
│   ├── libsmbios.h
│   ├── loongarch64
│   │   ├── efibind.h
│   │   ├── efilibplat.h
│   │   └── efisetjmp_arch.h
│   ├── make.inf
│   ├── Makefile
│   ├── makefile.hdr
│   ├── mips64el
│   │   ├── efibind.h
│   │   ├── efilibplat.h
│   │   └── efisetjmp_arch.h
│   ├── pci22.h
│   ├── protocol
│   │   ├── adapterdebug.h
│   │   ├── eficonsplit.h
│   │   ├── efidbg.h
│   │   ├── efivar.h
│   │   ├── ia64
│   │   │   └── eficontext.h
│   │   ├── intload.h
│   │   ├── legacyboot.h
│   │   ├── make.inf
│   │   ├── makefile.hdr
│   │   ├── piflash64.h
│   │   ├── readme.txt
│   │   └── vgaclass.h
│   ├── riscv64
│   │   ├── efibind.h
│   │   ├── efilibplat.h
│   │   └── efisetjmp_arch.h
│   ├── romload.h
│   └── x86_64
│       ├── efibind.h
│       ├── efilibplat.h
│       ├── efisetjmp_arch.h
│       └── pe.h
├── lib
│   ├── aarch64
│   │   ├── efi_stub.S
│   │   ├── initplat.c
│   │   ├── math.c
│   │   └── setjmp.S
│   ├── arm
│   │   ├── div.S
│   │   ├── edk2asm.h
│   │   ├── efi_stub.S
│   │   ├── initplat.c
│   │   ├── ldivmod.S
│   │   ├── llsl.S
│   │   ├── llsr.S
│   │   ├── math.c
│   │   ├── mullu.S
│   │   ├── setjmp.S
│   │   └── uldiv.S
│   ├── boxdraw.c
│   ├── cmdline.c
│   ├── console.c
│   ├── crc.c
│   ├── ctors.S
│   ├── data.c
│   ├── debug.c
│   ├── dpath.c
│   ├── entry.c
│   ├── error.c
│   ├── event.c
│   ├── exit.c
│   ├── guid.c
│   ├── hand.c
│   ├── hw.c
│   ├── ia32
│   │   ├── efi_stub.S
│   │   ├── initplat.c
│   │   ├── math.c
│   │   └── setjmp.S
│   ├── ia64
│   │   ├── initplat.c
│   │   ├── math.c
│   │   ├── palproc.h
│   │   ├── palproc.S
│   │   ├── salpal.c
│   │   └── setjmp.S
│   ├── init.c
│   ├── lock.c
│   ├── loongarch64
│   │   ├── efi_stub.S
│   │   ├── initplat.c
│   │   ├── math.c
│   │   └── setjmp.S
│   ├── Makefile
│   ├── mips64el
│   │   ├── efi_stub.S
│   │   ├── initplat.c
│   │   ├── math.c
│   │   └── setjmp.S
│   ├── misc.c
│   ├── pause.c
│   ├── print.c
│   ├── riscv64
│   │   ├── initplat.c
│   │   ├── math.c
│   │   └── setjmp.S
│   ├── runtime
│   │   ├── efirtlib.c
│   │   ├── rtdata.c
│   │   ├── rtlock.c
│   │   ├── rtstr.c
│   │   └── vm.c
│   ├── smbios.c
│   ├── sread.c
│   ├── str.c
│   └── x86_64
│       ├── callwrap.c
│       ├── efi_stub.S
│       ├── initplat.c
│       ├── math.c
│       └── setjmp.S
├── Make.defaults
├── Make.rules
├── Makefile
├── NOTES-01.md
├── README.efilib
├── README.elilo
├── README.git
└── README.gnuefi

24 directories, 216 files
