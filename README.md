NSK OS v0.3 — A From-Scratch Operating System

NSK OS is a custom-built operating system designed for 32-bit x86 systems running in i686 Protected Mode. It features its own kernel, memory management system, and bootable graphical interface.

Unlike Linux-based distributions, NSK OS is being developed from the ground up, without relying on Linux or Debian as its foundation. The goal is to build a lightweight operating system with a solid kernel architecture that can eventually support a modern desktop experience.

Phase 1: Boot Process and Core Kernel

Status: Core implementation completed

Phase 1 establishes the foundation of NSK OS, including the boot process, CPU configuration, interrupt handling, memory management, and kernel debugging.

The following components have been implemented:

1. Multiboot2 Boot Header — The "boot/boot.asm" file contains a Multiboot2-compliant header using the magic value "0xE85250D6". It also handles the initial 32-bit protected-mode entry point, stack setup, and memory and framebuffer requests.

2. Global Descriptor Table (GDT) — Configures flat memory segments for kernel code, kernel data, user code, and user data, providing the foundation for memory segmentation.

3. Interrupt Descriptor Table (IDT) — Sets up 256 interrupt gates, with handlers for CPU exceptions and hardware interrupts.

4. 8259 Programmable Interrupt Controller (PIC) — Remaps IRQs to avoid conflicts with CPU exception vectors. The master PIC uses vectors "0x20–0x27", while the slave PIC uses "0x28–0x2F".

5. 8254 Programmable Interval Timer (PIT) — Configures the system timer to generate 100 interrupts per second, enabling system tick counting and uptime tracking.

6. 16550 UART Serial Driver — Initializes the COM1 serial port at "0x3F8" with a baud rate of 38,400 for kernel debugging and diagnostic logging.

7. Freestanding "kprintf" Engine — Provides formatted kernel output with support for "%s", "%d", "%u", "%x", "%p", and "%c".

8. Physical Memory Manager (PMM) — Parses the Multiboot2 memory map and uses a bitmap-based allocator to manage physical memory in 4 KiB page frames.

9. Kernel Heap Allocator — Implements "kmalloc" and "kfree", with support for a dynamically managed heap, boundary tags, and free-block coalescing. The planned heap region spans from 4 MiB to 16 MiB.

Together, these components provide the initial infrastructure needed to move beyond basic booting and begin developing the operating system's graphics and desktop subsystems.

Repository Structure

├── boot/
│   ├── boot.asm             # Multiboot2 header and boot entry point
│   └── grub.cfg             # GRUB Multiboot2 configuration
├── include/
│   ├── gdt.h                # GDT declarations
│   ├── idt.h                # IDT and ISR definitions
│   ├── io.h                 # x86 I/O port helpers
│   ├── kheap.h              # Kernel heap allocator interface
│   ├── multiboot2.h         # Multiboot2 structures and tags
│   ├── pic.h                # 8259 PIC definitions
│   ├── pit.h                # PIT timer interface
│   ├── pmm.h                # Physical memory manager interface
│   ├── printf.h             # Kernel formatted output
│   ├── serial.h             # COM1 serial driver
│   ├── string.h             # Freestanding string and memory functions
│   └── types.h              # Fixed-width and freestanding types
├── kernel/
│   ├── gdt.c                # GDT initialization
│   ├── gdt_flush.asm        # Segment reload and far jump
│   ├── idt.c                # IDT setup and interrupt dispatch
│   ├── isr.asm               # CPU exception and IRQ stubs
│   ├── kernel.c             # Kernel entry point and self-tests
│   ├── kheap.c              # Kernel heap allocation functions
│   ├── multiboot2.c         # Multiboot2 tag parser
│   ├── pic.c                # PIC remapping and EOI handling
│   ├── pit.c                # 100 Hz PIT timer
│   ├── pmm.c                # Bitmap-based physical memory allocator
│   ├── printf.c             # Formatted kernel output
│   ├── serial.c             # UART serial implementation
│   └── string.c             # String and memory utilities
├── tests/
│   └── test_phase1.py       # QEMU serial smoke tests
├── .github/workflows/
│   └── build-iso.yml        # Automated build and ISO workflow
├── linker.ld                # Kernel linker script
├── Makefile                 # Build, ISO, run, test, and clean targets
└── README.md                # Project documentation

🛠️ Building and Running NSK OS

Prerequisites

The following tools are required to build and test the project:

- GCC with 32-bit compilation support ("gcc-multilib")
- NASM
- GNU Binutils, including "ld"
- QEMU System Emulator for x86
- GRUB utilities ("grub-pc-bin")
- "xorriso"
- "mtools"

1. Compile the kernel

make all

This builds the kernel binary at:

build/kernel.bin

2. Create a bootable ISO

make iso

The build process generates:

nsk-os-0.3.iso

The expected ISO size is approximately 8 MB, keeping the image well below 20 MB.

3. Run NSK OS in QEMU

make run

To test the kernel binary directly, use:

make run-kernel

These commands provide a convenient way to test the boot process and kernel initialization without requiring a physical machine.

Expected Boot Output

During a successful boot, NSK OS reports its initialization progress through the COM1 serial console.

The following is an example of the expected output:

*****************************************************************
*                 NSK OS v0.3 - Starting Kernel                 *
*             From-Scratch 32-bit x86 Protected Mode            *
*****************************************************************
NSK OS booting...

[NSK KERNEL] Kernel binary loaded at 0x00100000 - 0x00105000
[NSK MULTIBOOT2] Valid Multiboot2 header detected
[NSK MULTIBOOT2] Bootloader: GRUB 2.06
[NSK GDT] Global Descriptor Table initialized
[NSK PIC] 8259 PIC remapped successfully
[NSK IDT] Interrupt Descriptor Table loaded
[NSK PIT] 8254 Timer configured at 100 Hz

[NSK PMM] Memory map parsed successfully
[NSK PMM] Physical memory manager initialized
[NSK HEAP] Kernel heap initialized

[NSK TEST] Running kernel heap allocation tests...
  Memory allocation tests: PASSED
  Memory content checks: PASSED
  Free-block reuse: PASSED
  Block coalescing: PASSED

[NSK KERNEL] Hardware interrupts enabled
[NSK KERNEL] System timer running

=================================================================
       >>> PHASE 1 CORE KERNEL INITIALIZATION COMPLETE <<<
 System: x86 (i686), 32-bit Protected Mode
 Core  : Multiboot2, GDT, IDT, PIC, PIT, COM1, PMM, Heap
 Status: Ready for Phase 2 — Graphics Engine
=================================================================

Note: This is a simplified example of the expected serial output, not a guarantee of the exact messages or values produced by every build. Actual memory addresses, detected RAM, timer values, and test results depend on the implementation and runtime environment.

Phase 2: Graphics Engine

With the core kernel foundation in place, the next milestone is to develop the graphics engine that will power the NSK OS desktop environment.

Phase 2 is planned to introduce the following features:

- Linear Framebuffer Driver — A 32-bit color framebuffer targeting a resolution of 1536 × 1024, with support for back-buffer rendering and double buffering.
- Smooth Shape Rendering — Anti-aliased rounded rectangles and other basic graphical primitives for a cleaner interface.
- Alpha Blending — Transparency support for layered windows, panels, and desktop elements.
- Fast Box Blur — A lightweight blur implementation for translucent glass-style UI components.
- TrueType Font Rendering — Integration of "stb_truetype" to render scalable fonts.
- Wallpaper Support — Bitmap decoding and integration for desktop backgrounds.

The focus is on building these features directly into the operating system rather than relying on a conventional desktop environment.

🔭 Project Direction

NSK OS is being developed incrementally, starting with the kernel and moving toward a complete graphical operating system.

The immediate priority is to establish a reliable graphics pipeline on top of the existing kernel. Once the framebuffer and rendering primitives are working, development can move toward desktop windows, application interfaces, input handling, and other essential user-facing components.

The long-term goal is a lightweight, self-contained operating system with its own kernel and native desktop experience.

Development status should be verified against the current source code and successful build and runtime tests. Features listed in the roadmap are planned until they have been implemented and tested.Phase
