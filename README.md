# Sharp PC-1600 ROM dumps

This repository contains dumps of the Sharp PC-1600's internal
ROM (both CPUs — the Z-80-compatible SC7852 and the LH-5803 co-processor)
and, kept separately, the ROM of the CE-1600P peripheral, plus the tool used
to produce them. Everything here was captured directly from a European PC-1600 /
PC-1600P / PC-1600F over its built-in serial port (`COM1:`). As this is a European unit, no kanji ROM is present.

The ROM got dumped on a plain PC-1600, no additional memory modules installed.

## ROM versions

Sharp shipped two BASIC ROM revisions; see Sharp's bulletin No. 1600-010E. On
the PC-1600, `PRINT PEEK #(0,&7FFF)` tells them apart. The machines look
identical on the outside, so this is the only way to tell:

| Version | Directory | `PEEK #(0,&7FFF)` | Status |
|---|---|---|---|
| **New** | [`dumps/new/`](dumps/new) | `5` (dumped); `4` exists but is still missing | Complete (all 6 files), from a `5` machine |
| **Old** | [`dumps/old/`](dumps/old) | `130` | Complete (all 6 files), dumped twice — see [Old ROM](#old-rom) |

The `4` variant of the new ROM has not been dumped yet, so a dump from a
machine that reports `4` is the most wanted one. Further dumps of the old
(`130`) ROM can only confirm what is already here.

### Source machines

| ROM | Version | Machine production date |
|---|---|---|
| Calculator (PC-1600) | New | August 1990 and July 1989 (two machines) |
| Calculator (PC-1600) | Old | May 1986 |
| Peripheral (CE-1600P) | New | March 1988 |
| Peripheral (CE-1600P) | Old | October 1986 |

The tables below describe the **new** ROM's files. The old ROM uses the same
file names.

The calculator ROM (`dumps/new/`, `dumps/old/`) and the CE-1600P ROM
([`dumps/ce1600p/`](dumps/ce1600p), with `new/` and `old/` as well) are
independent. A peripheral's ROM lives in the peripheral itself, not in the
calculator, and is independent of the calculator's ROM version. If more
peripheral ROMs turn up, each gets its own directory under `dumps/`.

Want to dump the ROM of your own PC-1600? Follow the step-by-step guide in
[`DUMPING.md`](DUMPING.md).

## Contents

| File | What it is |
|---|---|
| `dumper/pc1600-rom-dumper.asm` | Z-80 assembly source for the dumper program (zasm dialect) |
| `dumper/pc1600-rom-dumper.bin` | Assembled machine-language binary, ready to load at `C0C5H` |
| `dumps/new/PC1600-P0-B0.BIN` | Page 0 (0000H–3FFFH), Bank 0 — main system ROM (always resident) |
| `dumps/new/PC1600-P1-B0.BIN` | Page 1 (4000H–7FFFH), Bank 0 — system ROM continuation |
| `dumps/new/PC1600-P1-B3.BIN` | Page 1, Bank 3 — system ROM (CS24 chip, normal half) |
| `dumps/new/PC1600-P1-B3B.BIN` | Page 1, Bank 3b — "hidden BASIC ROM" (CS24 chip, other half, selected via Port 3DH bit 2) |
| `dumps/new/PC1600-P2-B6.BIN` | Page 2 (8000H–BFFFH), Bank 6 — system ROM (CS123 chip, Z-80-visible half) |
| `dumps/new/PC1600-LH5803-C000-FFFF.BIN` | LH-5803 co-processor's own private ROM (LH-5803 view C000H–FFFFH) — see note below |

### Peripheral ROMs

| File | What it is |
|---|---|
| `dumps/ce1600p/new/PC1600-P1-B4-CE1600P.BIN` | Page 1, Bank 4 — CE-1600P printer/plotter ROM |
| `dumps/ce1600p/new/PC1600-P1-B5-CE1600P-OR-F.BIN` | Page 1, Bank 5 — CE-1600P floppy/cassette ROM (see note below) |

The `old/` directory holds the same two files (same names) from the older
peripheral version.

There are at least two versions of the peripheral code. With the peripheral
attached, `PRINT PEEK #(5,&7FFE)` and `PRINT PEEK #(5,&7FFF)` tell them apart
(the last two bytes of the Bank 5 file). If you own a CE-1600P, please check
it as well, and dump it if it differs from both versions listed here (see the
[dumping guide](DUMPING.md)):

| Version | Directory | `PEEK #(5,&7FFE)` / `PEEK #(5,&7FFF)` |
|---|---|---|
| New | [`dumps/ce1600p/new/`](dumps/ce1600p/new) | `5` / `18` (`05 12` hex) |
| Old | [`dumps/ce1600p/old/`](dumps/ce1600p/old) | `4` / `16` (`04 10` hex) |

Each `.BIN` file is exactly 16384 bytes (16KB).

The first 5 calculator files and the two peripheral files were each read starting at the page's base address while that
bank was paged in via the PC-1600's `BANKSET` IOCS routine (and, for Bank
3b, Port 3DH). The last calculator file, `PC1600-LH5803-C000-FFFF.BIN`, is different: it's
the *other* half of the same physical chip as `PC1600-P2-B6.BIN` (CS123
selects both), and the Z-80 side has no port combination that reaches it —
only the LH-5803 co-processor itself can read it, via its own address bus.
It was captured by running a tiny 2-byte LH-5801 program (`lda (x)` /
`rtn`) *on* the LH-5803 itself, one byte per round-trip, via the documented
`CALLH` bridge (`CALL 01C6H`). Same dumper program, menu option 3 — see
below.

**Bank 5 naming caveat:** two sources in the wider PC-1600 documentation
disagree on whether Bank 5 belongs to the CE-1600P (printer/plotter) or to
the separate CE-1600F (floppy) peripheral. The filename reflects that
ambiguity rather than picking one. Content-wise it's real ROM either way —
just the peripheral attribution is unconfirmed.

## Verification

MD5 checksums of the `.BIN` dumps as captured:

```
404bf6f2df489e09649167078acd9a25  dumps/new/PC1600-P0-B0.BIN
bddbb8bbf0b2bd2d95038f67b8d002ac  dumps/new/PC1600-P1-B0.BIN
6f6e1a9d46db7dc91d4322c93583ac81  dumps/new/PC1600-P1-B3.BIN
2483319acf35da4e848e59ab954abf46  dumps/new/PC1600-P1-B3B.BIN
05548a8dda3e572d50d4bd281a650ea8  dumps/ce1600p/new/PC1600-P1-B4-CE1600P.BIN
a675c6dbdf7dc4c10e8d96891e196f8f  dumps/ce1600p/new/PC1600-P1-B5-CE1600P-OR-F.BIN
df41b050acbc29c83214cbaaf29bee91  dumps/ce1600p/old/PC1600-P1-B4-CE1600P.BIN
33f3ef7207eac06587cc4c6d70c6cbd0  dumps/ce1600p/old/PC1600-P1-B5-CE1600P-OR-F.BIN
86cb9036da284de2b04c7946d140a9fd  dumps/new/PC1600-P2-B6.BIN
56168830b46d637b08529a74609bee3f  dumps/new/PC1600-LH5803-C000-FFFF.BIN
```

The MD5s above are just a fixed reference for these specific files. The
actual check made during capture, for every page: the 16-bit sum computed
on the PC-1600 itself (shown on the LCD immediately before sending) and
the 16-bit sum `sde get --raw` reports on receipt agree.

## Old ROM

`dumps/old/` holds the older ROM (`PEEK #(0,&7FFF)` = `130`). It was dumped
twice; a clean second dump reproduced five of the six files bit for bit. The
first dump's `PC1600-P1-B3B.BIN` was 16368 bytes (truncated, and wrong from
byte 11343 on), so it was replaced by the complete 16384-byte file from the
second dump (16-bit sum `0x5AF6`, matching the sum on receipt).

- The last byte of each probe page matches the values Sharp's bulletin gives
  for the old ROM: `82H` in `P1-B0`, `A1H` in `P2-B6`, `C1H` in `P1-B3`.
- Only the six files below are kept. The two CE-1600P pages (Bank 4 and
  Bank 5) from this dump were taken without a CE-1600P attached and contain
  no peripheral ROM, so they were not kept. The peripheral ROMs are in
  `dumps/ce1600p/` (see above); they do not depend on the calculator's
  ROM version.

```
5afcc22134e106bfd63b899febe9df7c  dumps/old/PC1600-P0-B0.BIN
3bcb6b178f5967c7c8e32afe560a3e75  dumps/old/PC1600-P1-B0.BIN
ded92d8280f8f83ce3498fb9cbfb9b3d  dumps/old/PC1600-P1-B3.BIN
2e8e075cac8f9696c5e833ceef130a4f  dumps/old/PC1600-P1-B3B.BIN
2c977fdd8c924c1492a2c23f67a20f23  dumps/old/PC1600-P2-B6.BIN
6005b6420bd5e191e81a1562f3242ec9  dumps/old/PC1600-LH5803-C000-FFFF.BIN
```

---

## How the dump was made

### 1. Assemble `dumper/pc1600-rom-dumper.asm`

Requires [zasm](https://k1.spdns.de/Projects/zasm/) (an 8080/Z80/Z180
assembler; version 4.5.0 or later recommended — the syntax used here is
plain Z80, nothing exotic). Prebuilt binaries for Linux/macOS/Windows are
on the project page; it's also easy to build from source.

```
cd dumper
zasm -uwy pc1600-rom-dumper.asm pc1600-rom-dumper.lst pc1600-rom-dumper.bin
```

`-u` embeds object code in the listing, `-w` appends a label listing, `-y`
adds cycle counts — all optional, just useful for inspection. The only
required output is `pc1600-rom-dumper.bin`, a flat binary with no header,
assembled to run at `C0C5H` (the standard load address for a small PC-1600
machine-language program living in internal RAM, right after the 197-byte
reserved header area).

If you already have `dumper/pc1600-rom-dumper.bin` from this directory, this
step can be skipped.

### 2. Load it onto the PC-1600

You need a way to get bytes onto the PC-1600's `COM1:` port. This was done
with [SharpDataExchange](https://github.com/tinue/SharpDataExchange) (`sde`, a
command-line tool for exchanging data with Sharp pocket computers over
serial), used only as a convenient example — any tool that can send an
arbitrary byte stream to the PC-1600's serial port and let you type BASIC
commands works.

Download the archive for your platform from the
[Releases page](https://github.com/tinue/SharpDataExchange/releases) and put `sde` on your `PATH`.

**On the PC-1600**, reserve space for the program before loading:

```
NEW"S0:",&B00
```

(`&B00` = 2816 decimal bytes — comfortably more than `dumper/pc1600-rom-dumper.bin`'s
size plus the 197-byte header offset; adjust upward if a future version of
the program grows past that.)

**On the PC**, send the binary with a machine-language transfer header,
giving both the load and auto-run address (`C0C5H`):

```
sde put --device pc1600 \
    --start-address C0C5 --run-address C0C5 dumper/pc1600-rom-dumper.bin
```

**On the PC-1600**, receive and auto-run it:

```
LOAD"COM1:",R
```

(Omit `,R` to just load without running, then start it later with
`CALL&C0C5`.)

### 3. Set up the serial port for receiving (before dumping)

The dumper program never configures `COM1:` itself — it expects the port
to already be set up from BASIC. On the PC-1600, before running the dump
menu's send option, type:

```
SETCOM"COM1:",9600,8,N,1,N,N
OUTSTAT"COM1:"
SNDSTAT"COM1:",28
RCVSTAT"COM1:",28
INIT"COM1:",4096
```

(`28` disables RTS/CTS hardware flow control, which does not work on many
USB-to-serial setups; `sde` 0.2.2 or newer paces the transfer itself. To use
hardware flow control anyway, pass `--flowcontrol` to `sde` and use `24`
instead. `OUTSTAT` with no parameter enables dynamic RTS/DTR flow control.)

### 4. Run the dumper and capture each page

Running `pc1600-rom-dumper.bin` (`CALL&C0C5` if not auto-run) shows a menu:

- **`1` = SWEEP** — walks every known ROM bank, showing a short hex sample
  on the LCD per bank. Useful to eyeball a bank before committing to a
  full dump; doesn't touch the serial port at all.
- **`2` = DUMP+SEND** — pages in each of the 7 confirmed-ROM banks in turn
  (the ones listed under Contents above), shows a 16-bit checksum for the
  page, then waits for a keypress: any key sends the full 16KB page over
  `COM1:`; `S` skips it without sending (useful to avoid re-sending a page
  already captured correctly).
- **`3` = LH5803 ROM** — dumps the LH-5803's own private 16KB ROM via the
  `CALLH` bridge (see Contents above). Runs a quick self-test first (fetches
  the same byte twice and checks the two agree) before showing a checksum;
  the checksum pass itself takes appreciably longer than the Z-80-side
  banks (~30 seconds observed — 16384 individual CPU-to-CPU round-trips),
  which is expected. Same keypress convention as option 2.

**Before pressing a key to send a page (option 2 or 3)**, start a receiver
on the PC:

```
sde get --device pc1600 --raw <filename>.bin
```

`--raw` is the important flag here — it captures the byte stream verbatim,
with no header parsing or content detection, ending the transfer on an
idle-timeout (500ms of no new bytes for the PC-1600). This matches how the
dumper sends: a plain 16384-byte stream, no header. Use the filename hint
the PC-1600's screen shows for that page (matches the `.BIN` names in this
directory) so the receiver output stays self-documenting.

Start the receiver, *then* press the key on the PC-1600 to send — the
PC-1600 doesn't buffer, so the receiver has to already be listening.

Repeat for all pages (7 from option 2, 1 from option 3). `sde get
--raw` prints a 16-bit sum (mod 65536) of the received bytes on completion;
compare it against the checksum the PC-1600 showed on screen for that page
before sending. They should match — if they don't, the transfer is suspect
and worth re-doing that page.

### Hardware notes

- The PC-1600's serial port is 5V TTL via a 15-pin connector; a USB/UART
  adapter is needed to talk to it from a modern PC (see
  SharpDataExchange's `docs/HardwareNotes.md` for wiring detail — this isn't specific
  to the dumper).
- A CE-1600P printer/plotter should be attached for Bank 4/5 to show real
  content instead of open bus (`FF` fill) — those banks are the CE-1600P's
  own ROM, only visible with the peripheral connected.
- The LH5803 dump (option 3) needs no peripheral attached and doesn't touch
  Z-80 bank switching at all — it's a completely separate mechanism (the
  `CALLH` cross-CPU bridge) from options 1/2.
