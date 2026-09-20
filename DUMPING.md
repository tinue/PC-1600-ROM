# How to dump the ROM of your PC-1600

This guide walks you through reading the ROM out of your own Sharp PC-1600 and
saving it as files on your computer. It takes about 30 minutes.

You will end up with eight files of 16 KB each.

The dumps in [`dumps/new/`](dumps/new) come from the **newer** ROM version,
which is the one in nearly every PC-1600. An **older** version exists, but it
is rare. It is in [`dumps/old/`](dumps/old) (dumped twice, files identical). To find out which version you have, type this on
the PC-1600:

```
PRINT PEEK #(0,&7FFF)
```

- `4` or `5`: the newer version. Your files should be identical to the ones in
  [`dumps/new/`](dumps/new).
- `130`: the older, rare version. Your files should be compared with
  [`dumps/old/`](dumps/old), and please share them if they differ.

## What you need

- A Sharp PC-1600 with fresh batteries or its power adapter
- A USB-to-serial cable that is already wired up for the PC-1600's 15-pin
  serial connector
- A Mac, Linux or Windows computer
- Optional: a CE-1600P printer/plotter, connected to the PC-1600. Without it,
  two of the eight files (the peripheral ROM) contain only `FF` bytes (see [Notes](#notes)).

## 1. Download the two programs

On your computer:

1. **`sde`**, the transfer tool: go to the
   [SharpDataExchange releases page](https://github.com/tinue/SharpDataExchange/releases),
   download the archive for your system, and unpack it. Put `bin/sde`
   somewhere on your `PATH`, or simply run it from where you unpacked it.
2. **The dumper program**: download
   [`pc1600-rom-dumper.bin`](https://github.com/tinue/PC-1600-ROM/raw/main/dumper/pc1600-rom-dumper.bin)
   and remember where you saved it.

Then open a terminal (Command Prompt or PowerShell on Windows) in the folder
where you want the dumped files to end up.

## 2. Connect the PC-1600

1. Switch the PC-1600 off.
2. Plug the serial cable into the PC-1600's serial connector.
3. Plug the USB end into your computer.
4. Switch the PC-1600 on.

## 3. Set up the PC-1600

**On the PC-1600**, switch to **PRO mode**. The `NEW` command below only works
in PRO mode. Then reserve some memory for the dumper, and set up the serial
port by typing these lines:

```
NEW"S0:",&B00
SETCOM"COM1:",9600,8,N,1,N,N
OUTSTAT"COM1:"
SNDSTAT"COM1:",24
RCVSTAT"COM1:",24
INIT"COM1:",8192
```

## 4. Load the dumper onto the PC-1600

**On the PC-1600**, get ready to receive the program:

```
BLOAD"COM1:"
```

**On your computer**, send the program (adjust the path to where you saved the
file). Do this right after pressing Enter on the PC-1600:

```
sde put --device pc1600 --start-address 0xC0C5 --run-address 0xC0C5 pc1600-rom-dumper.bin -v
```

When the transfer is done, the dumper starts by itself.

> If `sde` complains that it cannot find the serial port, add
> `--port <name>` to the command. The name looks like `/dev/cu.usbserial-XXXX`
> on a Mac, `/dev/ttyUSB0` on Linux, or `COM3` on Windows.

## 5. Dump the pages

After loading, the dumper shows this menu on the PC-1600:

| Key | What it does |
|---|---|
| `1` | Sweep: shows a small sample of every bank on the display. Nothing is sent. Just for looking. |
| `2` | Dump and send the 7 pages of the main ROM and the CE-1600P |
| `3` | Dump and send the LH-5803 co-processor ROM (takes about 30 seconds before it asks to send) |

> **➡️ Now press `2` on the PC-1600.**
> The other options do not dump the main ROM. Option `3` comes at the very end.

The PC-1600 now works through the pages one by one. For each page, it shows the **file name** to use (second
line) and a **checksum** `CHK=` (third line), and then asks
`PRESS KEY TO SEND`. ***Remember or write down the checksum***; you will compare it in a moment.

For each page, do these steps in this order:

1. **On your computer**, start the receiver, using the file name shown on the
   PC-1600's display (for example `PC1600-P0-B0.BIN`):

   ```
   sde get --device pc1600 --raw PC1600-P0-B0.BIN
   ```

   The receiver now waits.
2. **On the PC-1600**, press any key. The page is sent (about 20 seconds).
   Pressing `S` instead skips the page, for example one you already have.
3. **On your computer**, `sde` finishes by itself and prints the number of
   bytes and a **16-bit sum**. The byte count must be `16384`, and the sum must
   be **the same as the checksum on the PC-1600's display**.
   If they differ, dump that page again (see below).
4. The PC-1600 now moves on to the next page. Repeat from step 1.

After the seventh page, the PC-1600 shows `DONE - BASIC RESTORED` and returns
to BASIC. To get the last file, start the dumper again by typing `CALL&C0C5`, press
`3`, and repeat the same steps for `PC1600-LH5803-C000-FFFF.BIN`.

To repeat a single page, start the dumper again, press `2`, and press `S` for
every page you do not want to send again.

## 6. Done

You now have eight `.BIN` files, each exactly 16384 bytes. You can check them
against the MD5 list in the [README](README.md#verification) (for example
`md5 file.BIN` on a Mac, `md5sum file.BIN` on Linux,
`certutil -hashfile file.BIN MD5` on Windows). With the newer
version (`4` or `5` above), all checksums should match. If a file differs, dump
it a second time: if you get the same result twice, the dump is good, and you
probably have the rare older version.

## Notes

- **Nothing on the PC-1600 is changed.** The dumper only reads the ROM. It
  lives in RAM, so `NEW` or a reset removes it.
- **Without the CE-1600P attached**, the two pages that belong to it
  (`PC1600-P1-B4-CE1600P.BIN` and `PC1600-P1-B5-CE1600P-OR-F.BIN`, kept in
  `dumps/peripherals/`) come out
  filled with `FF`. That is normal.
- **`ERROR 142` on the PC-1600.** Most likely one of the serial setup commands
  from step 3 went wrong, or the PC-1600 was reset in the meantime. Type the
  setup commands again (`SETCOM`, `OUTSTAT`, `SNDSTAT`, `RCVSTAT`, `INIT`) and
  retry.
- **Start the receiver first.** The PC-1600 does not buffer: if `sde get` is
  not already waiting when you press the key, data is lost.
- **Stopping the dumper.** It has no quit key. To abort a send, press `BREAK`
  on the PC-1600; it shows `SEND ERROR - ABORTED` and returns to BASIC. The
  same happens if the receiver on your computer is not answering.
- **Something not working?** See the [Troubleshooting](https://github.com/tinue/SharpDataExchange#troubleshooting)
  section of the `sde` documentation. The cable wiring is described in its
  [`docs/HardwareNotes.md`](https://github.com/tinue/SharpDataExchange/blob/main/docs/HardwareNotes.md).
