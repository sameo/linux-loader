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


