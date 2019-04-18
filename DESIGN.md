# ELF Image parsing and loading

The boot process is explained from the following two sides.

## Loader side

It follows ELF standard which is specified in elf.rs. The entry header and program headers will be inerpreted, and PT_LOAD segments will be loaded into guest memory.

### Where kernel is loaded 

There are two ways on deciding where the program segments will be loaded.

- One way is to provide an option and allow vmm to specify where to load the image, considering its memory layout.
- The other way is to load image into phdr.p_paddr by default.

## Vmm side

### Construct zero page

- According to the 64-bit boot protocol, the boot parameters (traditionally known as "zero page") should be setup, including setup_header, e820 table and other stuff. ELF does not contain a setup_header, so vmm should be responsible for the construction. 

### Configure vcpu

- RIP, the start offset of guest memory where kernel is loaded, which is returned from loader
- 64 bit mode with paging enabled
- GDT must be configured and loaded

# bzImage

The boot process is also explained from the following two sides.

## Loader side

### What will be returned from loader

- Regarding to the 64-bit boot protocol, kernel is started by jumping to the 64-bit kernel entry point, which is the start address of loaded 64-bit kernel plus 0x200. bzImage includes two parts, setup and compressed kernel. The compressed kernel part will be loaded into guest memory, additionally, the start address of loaded kernel, the offset of end address, and the setup header begin at the offset 0x01f1 of bzImage will be returned to vmm.

### Where kernel is loaded

The same as ELF image loader, there are two ways for deciding where the compressed kernel will be loaded.

- Vmm specify where to load kernel image.
- Load into code32_start (Boot load address) be default.

### Additional checking

- As what the boot protocol said, the kernel is a bzImage kernel if the protocol >= 2.00 and the 0x01 bit(LOAD_HIGH) is the loadflags field is set. Thus, checking this to ensure it is a valid bzImage.

## Vmm side

### Construct zero page

- While vmm build "zero page", bzImage loader will return the setup header to fill the boot parameters, it also covers other parts. One point to emphasize, that setup_header.init_size should be loaded into zero page, which will be used during head_64.S boot process.

### Configure vcpu

- RIP, the start address of loaded 64-bit kernel returned from loader + 0x200 (required by 64 bit boot protocol)
- 64 bit mode with paging enabled
- GDT must be configured and loaded


