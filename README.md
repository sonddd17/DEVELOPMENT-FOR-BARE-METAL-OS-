# Bare-Metal OS for Raspberry Pi (AArch64)

A minimal bare-metal kernel for the Raspberry Pi 3, written from scratch in C and ARM64 Assembly. It runs with no operating system and no standard library. It boots on its own, talks directly to the hardware (UART, mailbox, framebuffer) and runs a small shooter game on the screen.

<!-- TODO: add a screenshot or GIF of the game running in QEMU -->
<!-- ![Game screenshot](docs/screenshot.png) -->

## Features

- **Boot code in ARM64 Assembly** (`boot.S`): parks the secondary CPU cores and keeps only core 0 running, sets up the stack, clears the BSS section, then jumps to `main()` in C.
- **Custom linker script** (`link.ld`) that controls the memory layout of the kernel image.
- **UART1 driver** for serial input and output.
- **Mailbox interface** for talking to the VideoCore GPU.
- **Framebuffer graphics** at 1060×700 with 32-bit colour:
  - drawing pixels and rectangles
  - a bitmap font for drawing text
  - displaying images with transparency and rotation
  - video playback (12 FPS), switched on with `ENABLE_VIDEO` in `image.h`
- **Shooter game** that runs directly on the kernel:
  - a game loop at 25 frames per second (40 ms per frame)
  - a game state machine: `new_game → play → next_lv → win / quit`
  - player movement and 45° rotation, bullets with a shooting cooldown
  - up to 10 enemies with HP, collision detection and invulnerability frames after the player is hit
  - score, health bar and 10 rounds
- No standard library, so `memset`, `memcpy`, maths functions, a random number generator and `int_to_str` are written by hand.

## Tech Stack

| | |
|---|---|
| Languages | C, ARM64 (AArch64) Assembly |
| Toolchain | `aarch64-none-elf-gcc`, `aarch64-none-elf-ld`, `aarch64-none-elf-objcopy` |
| Build | Make |
| Emulator | QEMU (`raspi3` machine) |

## Project Structure

```
.
├── Makefile          # Build and run rules
└── kernel/
    ├── boot.S        # Entry point: core check, stack setup, BSS clear
    ├── link.ld       # Linker script (kernel memory layout)
    ├── kernel.c/.h   # main() function
    ├── uart1.c/.h    # UART1 serial driver
    ├── gpio.h        # GPIO register definitions
    ├── mbox.c/.h     # Mailbox interface to the GPU
    ├── framebf.c/.h  # Framebuffer: pixels, shapes, images, text
    ├── font.c/.h     # Bitmap font data
    ├── image.c/.h    # Image, sprite and video data
    ├── game.c/.h     # Shooter game logic
    └── math.c/.h     # Helper maths functions (no stdlib)
```

## Getting Started

### Prerequisites

- [Arm GNU Toolchain](https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads), target `aarch64-none-elf`
- [QEMU](https://www.qemu.org/download/) with `qemu-system-aarch64`
- `make`
- [Git LFS](https://git-lfs.com/). The C source files are stored with Git LFS because the image data is large.

### Clone

```bash
git lfs install
git clone https://github.com/sonddd17/DEVELOPMENT-FOR-BARE-METAL-OS-.git
cd DEVELOPMENT-FOR-BARE-METAL-OS-
```

### Build and Run

```bash
mkdir build      # the Makefile writes object files here
make             # clean, build kernel8.img, then start QEMU
```

To build or run separately:

```bash
make kernel8.img # build only
make run         # run in QEMU
```

QEMU runs the kernel with:

```bash
qemu-system-aarch64 -M raspi3 -kernel kernel8.img -serial null -serial stdio
```

UART1 output appears in your terminal (`-serial stdio`). The game is drawn in the QEMU window.

> **Note:** The Makefile is written for Windows (`SHELL = cmd`, `del`). On Linux or macOS, remove the `SHELL = cmd` line and change the `clean` rule to:
> ```make
> clean:
> 	rm -f *.img ./build/*.elf ./build/*.o
> ```

## Controls

<!-- TODO: fill in the keys used in game.c -->
| Key | Action |
|---|---|
| `?` | Move |
| `?` | Rotate |
| `?` | Shoot |
| `?` | Quit |

## Author

**Son Tran Truong** · [GitHub](https://github.com/sonddd17) · [LinkedIn](https://linkedin.com/in/sontran17)
