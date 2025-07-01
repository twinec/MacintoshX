# 🖥️ MacintoshX: Turn Your Genesis into a Macintosh Plus Hybrid

A community-driven hardware/software project to build a **Genesis cartridge-style addon** that transforms your Genesis (Model 1/2/3/Nomad) into a working **Macintosh 128K/512K/Plus** clone — while still allowing standard Genesis gameplay. Compatibility with Sega CDX and JVC X'Eye is a design priority, ensuring that their CD drive lids can still open with the MacintoshX cartridge attached.

---

## 🚀 Project Goals

* Use a **single 68010 CPU** (mounted on the cartridge) shared between Genesis *and* Macintosh environments.
* Provide **memory isolation** using a small FPGA/CPLD MMU so neither system interferes with the other.
* Enable **clean, instruction-bound context switching** triggered during idle cycles—no flags, no mid-instruction jumps.
* Offer **pixel-perfect VGA output** (640×480) from a 512×342 1‑bit framebuffer, avoiding composite fuzz.
* Include a **standard DB‑19 floppy interface** for use with real Macintosh floppy drives or BMOW Floppy Emu.
* Include a **non-optional cartridge passthrough** so original Genesis games can still be played.
* Power via a **required PicoPSU module** for full voltage support, including negative rails.

> **NOTE:** The 68010 CPU is installed on the MacintoshX cartridge itself rather than inside the console. The onboard Genesis 68000 must be electrically disabled, likely via the cartridge port’s access to `/HALT` and `/RESET`, to avoid bus contention. This design ensures reversibility for unmodified consoles and facilitates CPU arbitration.

---

## 🔧 Hardware Design Overview

1. **68010 CPU** — mounted on the cartridge; Genesis CPU is halted.
2. **FPGA/CPLD** — handles MMU, bus arbitration, memory map switching.
3. **Separate ROM (128–256 KB)** and **SRAM (512 KB)** mapped as Mac memory.
4. **VGA pixel-doubler** logic (via FPGA or MCU like RP2040) for crisp display.
5. **PS/2 → Mac DE‑9 I/O converter** (keyboard/mouse).
6. **DB‑19 Floppy port** for Macintosh drive connectivity.
7. **Genesis passthrough cartridge connector** for running games or accessories.
8. **PicoPSU + 7905** module to generate **+12V and –5V** rails required by Macintosh floppy drives.
9. **Logic simplification opportunities** using insights from open Mac Plus implementations like the **Plus Too** project.
10. **Low-profile cartridge shell option** to maintain compatibility with top-loading CD units like the **Sega CDX** and **JVC X'Eye**, ensuring their CD drive lids are not obstructed.

---

## ⏱️ Cooperative Task Switching

Using the 68010’s bus signals (`/AS`, `/DTACK`, `/BR`, `/BG`) and a hardware timer:

* Monitor for a *safe idle period* (no bus activity).
* Assert **Bus Request (`/BR`)** at any time.
* The CPU will assert **Bus Grant (`/BG`)** only after it finishes its current instruction and the bus is idle.
* Upon receiving `/BG`, the MMU:

  * Swaps memory maps (Genesis ↔ Mac).
  * Temporarily takes control of address/data bus.
* Releases `/BR`, CPU resumes execution in the alternate environment.

This ensures **mid-instruction safety** and a clean context swap without needing mode flags or software interrupts.

---

## 🧬 CPU Arbitration Strategy

To allow the cartridge-mounted 68010 to take over, the onboard Genesis 68000 must be electrically disabled:

* Assert the **/HALT** line (active low) through the cartridge interface. This tri-states the onboard CPU's bus lines.
* Optionally assert **/RESET** for extra safety on startup.
* Once halted, the 68010 on the cartridge drives all address/data/control lines.

This method avoids any need to physically remove the Genesis CPU and maintains full reversibility.

---

## 💾 Power Requirements

* ROM + RAM: \~100 mA
* FPGA/MMU: \~75 mA
* VGA/MCU logic: \~150 mA
* Floppy drive: requires **+12V and –5V**, powered by required PicoPSU + 7905 module
  **Total ≈330–450 mA @5 V** (not including 12V/–5V floppy demand)

*Reference Genesis & 32X operate on shared supplies; 32X uses an external adaptor due to higher power demands.*

---

## 📂 Repository Contents (TBD)

* `docs/` — design docs, pin maps, memory layouts
* `fpga/` — HDL source & simulation testbenches
* `board/` — PCB/FPC layout files
* `mcus/` — PS/2→DE‑9 and VGA pixel‑doubler firmware
* `sw/` — prototype code for context switching, floppy routines

---

## 🙌 Get Involved!

We need help from retro-hardware enthusiasts, FPGA devs and 68K gurus!

* **Schematics** for MMU logic & bus monitoring
* **Verilog/VHDL drivers** for VGA and I/O conversion
* **Prototype board layouts** sized to fit within Genesis/Nomad casing
* **Proof of concept firmware** for floppy interface and context switch routines
* **3D printed case designs** matching the 32X aesthetic
* **Explore logic reduction** by adapting techniques from **Plus Too** or similar projects
* **Design low-profile enclosures** that work with **CDX** and **X'Eye** systems

---

## 💬 Join the Conversation

Jump into the discussion on our Discord:
**[discord.gg/VEw2tt5EMz](https://discord.gg/VEw2tt5EMz)**

Use the dedicated #macintoshx-genesis-discussion channel to brainstorm, share progress, or contribute ideas.

---

## 📜 References

* Genesis and 32X power ratings ([Wikipedia](https://en.wikipedia.org/wiki/32X), [SegaRetro](https://segaretro.org/AC_adaptor))
* Open-source cartridge projects ([EverDrive](https://krikzz.com/store/), MegaSD, etc.)
* Mac Plus FPGA clone inspiration ([Plus Too project](https://www.wired.com/2011/10/plus-too-a-homebrew-mac-plus)) — valuable for minimizing component count and adapting Mac logic to FPGA/CPLD efficiently

---

**Two systems. One CPU. Game like it’s 1991. Compute like it’s 1986.**
