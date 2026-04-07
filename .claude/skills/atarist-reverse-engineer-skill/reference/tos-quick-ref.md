# Atari ST TOS Quick Reference

Condensed reference for system calls, vectors, and hardware used in Atari ST binary analysis.

---

## GEMDOS Functions (TRAP #1)

Call convention: push parameters right-to-left, push function number, `TRAP #1`, clean stack.

| Func# | Name | Parameters | Returns | Stack Clean |
|---|---|---|---|---|
| $00 | Pterm0 | (none) | never returns | — |
| $01 | Cconin | (none) | D0.L = char | 2 |
| $02 | Cconout | char.W | (void) | 4 |
| $03 | Cauxin | (none) | D0.L = char | 2 |
| $04 | Cauxout | char.W | (void) | 4 |
| $05 | Cprnout | char.W | D0.W = status | 4 |
| $06 | Crawio | char.W | D0.L = char or 0 | 4 |
| $07 | Crawcin | (none) | D0.L = char (no echo) | 2 |
| $08 | Cnecin | (none) | D0.L = char (no echo, waits) | 2 |
| $09 | Cconws | string.L | (void) | 6 |
| $0A | Cconrs | buffer.L | (void) | 6 |
| $0B | Cconis | (none) | D0.W = -1/0 | 2 |
| $0E | Dsetdrv | drive.W | D0.L = drive map | 4 |
| $19 | Dgetdrv | (none) | D0.W = drive# | 2 |
| $1A | Fsetdta | addr.L | (void) | 6 |
| $20 | Super | stack.L | D0.L = old SSP | 6 |
| $2F | Fgetdta | (none) | D0.L = DTA addr | 2 |
| $30 | Sversion | (none) | D0.W = version | 2 |
| $36 | Dfree | buf.L, drive.W | D0.L = error | 8 |
| $39 | Dcreate | path.L | D0.L = error | 6 |
| $3A | Ddelete | path.L | D0.L = error | 6 |
| $3B | Dsetpath | path.L | D0.L = error | 6 |
| $3C | Fcreate | name.L, attr.W | D0.W = handle | 8 |
| $3D | Fopen | name.L, mode.W | D0.W = handle | 8 |
| $3E | Fclose | handle.W | D0.W = error | 4 |
| $3F | Fread | handle.W, count.L, buf.L | D0.L = bytes read | 12 |
| $40 | Fwrite | handle.W, count.L, buf.L | D0.L = bytes written | 12 |
| $41 | Fdelete | name.L | D0.L = error | 6 |
| $42 | Fseek | offset.L, handle.W, mode.W | D0.L = position | 10 |
| $43 | Fattrib | name.L, flag.W, attr.W | D0.W = attr | 10 |
| $47 | Dgetpath | buf.L, drive.W | D0.W = error | 8 |
| $48 | Malloc | size.L | D0.L = addr | 6 |
| $49 | Mfree | addr.L | D0.L = error | 6 |
| $4A | Mshrink | 0.W, addr.L, size.L | D0.L = error | 12 |
| $4B | Pexec | mode.W, name.L, cmd.L, env.L | D0.L = error | 16 |
| $4C | Pterm | code.W | never returns | — |
| $4E | Fsfirst | name.L, attr.W | D0.L = error | 8 |
| $4F | Fsnext | (none) | D0.L = error | 2 |
| $56 | Frename | 0.W, old.L, new.L | D0.L = error | 12 |
| $57 | Fdatime | buf.L, handle.W, flag.W | D0.L = error | 10 |

### GEMDOS Parameter Values

**Fopen mode.W**: 0=read-only, 1=write-only, 2=read/write

**Fseek mode.W**: 0=from start, 1=from current position, 2=from end

**Pexec mode.W**: 0=load+go, 1=load (don't execute), 2=reserved, 3=load overlay, 4=go (execute already loaded), 5=create basepage only

**Fattrib flag.W**: 0=read attributes, 1=set attributes

### GEMDOS Error Codes
| Code | Meaning |
|---|---|
| -1 | General error |
| -33 | File not found |
| -34 | Path not found |
| -35 | Too many open files |
| -36 | Access denied |
| -39 | Insufficient memory |
| -46 | Invalid drive |

---

## BIOS Functions (TRAP #13)

Call convention: push parameters right-to-left, push function number (word), `TRAP #13`, clean stack.

**Packed PEA pattern**: `PEA $XXXXyyyy` pushes func# (high word) + first param (low word) in one instruction.

| Func# | Name | Parameters | Returns | Description |
|---|---|---|---|---|
| $00 | Getmpb | ptr.L | (void) | Get Memory Parameter Block |
| $01 | Bconstat | dev.W | D0.W = -1/0 | Check device input status |
| $02 | Bconin | dev.W | D0.L = char+scan | Read character from device |
| $03 | Bconout | dev.W, char.W | (void) | Write character to device |
| $04 | Rwabs | flag.W, buf.L, count.W, rec.W, dev.W | D0.L = err | Read/write disk sectors |
| $05 | Setexc | vec#.W, addr.L | D0.L = old addr | Set exception vector |
| $06 | Tickcal | (none) | D0.L = ms/tick | Timer calibration |
| $07 | Getbpb | dev.W | D0.L = BPB ptr | Get BIOS Parameter Block |
| $08 | Bcostat | dev.W | D0.W = -1/0 | Check device output status |
| $09 | Mediach | dev.W | D0.W = status | Check media change |
| $0A | Drvmap | (none) | D0.L = bitmap | Get connected drive bitmap |
| $0B | Kbshift | mode.W | D0.W = state | Get/set keyboard shift state |

### BIOS Device Numbers
| Dev# | Device |
|---|---|
| 0 | Printer (PRT:) |
| 1 | AUX/RS-232 (AUX:) |
| 2 | Console (CON:) |
| 3 | MIDI |
| 4 | Keyboard (IKBD) |
| 5 | Screen (raw) |

### Bconin Return Value (device 2 — console)
```
D0.L = $00 SS 00 AA
         │  │     └── ASCII code (low byte)
         │  └──────── Scancode (high byte of high word)
         └─────────── Always zero
```

### Kbshift Bits
| Bit | Key |
|---|---|
| 0 | Right Shift |
| 1 | Left Shift |
| 2 | Control |
| 3 | Alternate |
| 4 | Caps Lock |
| 5 | Right mouse button (via IKBD) |
| 6 | Left mouse button (via IKBD) |

---

## XBIOS Functions (TRAP #14)

Call convention: same as BIOS — push params right-to-left, push function number, `TRAP #14`, clean stack.

| Func# | Name | Key Parameters | Returns | Stack | Description |
|---|---|---|---|---|---|
| $00 | Initmous | type.W, param.L, vec.L | — | 12 | Initialize mouse (type: 0=disable, 1=relative, 2=absolute, 4=keycode) |
| $01 | Ssbrk | amount.W | D0.L = addr | 4 | Reserve memory at top of RAM |
| $02 | Physbase | — | D0.L = addr | 2 | Get physical screen base |
| $03 | Logbase | — | D0.L = addr | 2 | Get logical screen base |
| $04 | Getrez | — | D0.W = rez | 2 | Get resolution (0=low, 1=med, 2=high) |
| $05 | Setscreen | log.L, phys.L, rez.W | — | 12 | Set screen addresses/resolution (-1 = keep) |
| $06 | Setpalette | pal.L | — | 6 | Set all 16 palette registers |
| $07 | Setcolor | reg.W, col.W | D0.W = old | 6 | Set/read one palette register (-1 = read) |
| $08 | Floprd | buf.L, 0.L, dev.W, sect.W, trk.W, side.W, cnt.W | D0.L = err | 20 | Read floppy sectors |
| $09 | Flopwr | buf.L, 0.L, dev.W, sect.W, trk.W, side.W, cnt.W | D0.L = err | 20 | Write floppy sectors |
| $0A | Flopfmt | buf.L, 0.L, dev.W, spt.W, trk.W, side.W, interl.W, magic.L, fill.W | D0.L = err | 26 | Format floppy track |
| $0C | Midiws | cnt.W, buf.L | — | 8 | Write string to MIDI port |
| $0D | Mfpint | vec#.W, vector.L | — | 8 | Set MFP interrupt vector |
| $0E | Iorec | dev.W | D0.L = ptr | 4 | Get I/O buffer record (0=RS232, 1=kbd, 2=MIDI) |
| $0F | Rsconf | baud.W, flow.W, ucr.W, rsr.W, tsr.W, scr.W | D0.L = old | 14 | Configure RS-232 |
| $10 | Keytbl | unshift.L, shift.L, caps.L | D0.L = ptr | 14 | Get/set key translation tables (-1 = keep) |
| $11 | Random | — | D0.L = 24-bit | 2 | Get pseudo-random number |
| $12 | Protobt | buf.L, serial.L, type.W, exec.W | — | 14 | Create prototype boot sector |
| $13 | Flopver | buf.L, 0.L, dev.W, sect.W, trk.W, side.W, cnt.W | D0.L = err | 20 | Verify floppy sectors |
| $14 | Scrdmp | — | — | 2 | Trigger screen dump |
| $15 | Cursconf | func.W, rate.W | D0.W | 6 | Configure cursor (0=flash, 1=steady, 2=off, 3=on, 4=set rate, 5=get rate) |
| $16 | Settime | datetime.L | — | 6 | Set system date/time |
| $17 | Gettime | — | D0.L = datetime | 2 | Get system date/time |
| $18 | Bioskeys | — | — | 2 | Restore default keyboard tables |
| $19 | Ikbdws | cnt.W, buf.L | — | 8 | Write string to intelligent keyboard |
| $1A | Jdisint | intno.W | — | 4 | Disable MFP interrupt |
| $1B | Jenabint | intno.W | — | 4 | Enable MFP interrupt |
| $1C | Giaccess | val.W, reg.W | D0.W = val | 6 | Access YM2149 sound chip (reg bit 7: 0=read, 1=write) |
| $1D | Offgibit | bitno.W | — | 4 | Clear bit in YM2149 port A |
| $1E | Ongibit | bitno.W | — | 4 | Set bit in YM2149 port A |
| $1F | Xbtimer | timer.W, ctrl.W, data.W, vec.L | — | 12 | Set MFP timer (timer: 0=A, 1=B, 2=C, 3=D) |
| $20 | Dosound | cmdlist.L | — | 6 | Execute sound command list |
| $21 | Setprt | config.W | D0.W = old | 4 | Configure printer (-1 = read) |
| $22 | Kbdvbase | — | D0.L = ptr | 2 | Get keyboard vector table |
| $23 | Kbrate | delay.W, repeat.W | D0.L = old | 6 | Set key repeat rate (-1 = keep) |
| $24 | Prtblk | pblk.L | — | 6 | Print graphics block |
| $25 | Vsync | — | — | 2 | Wait for vertical blank |
| $26 | Supexec | func.L | D0 = func return | 6 | Execute function in supervisor mode |
| $27 | Puntaes | — | — | 2 | Expunge AES (reclaim memory) |

### XBIOS Gettime/Settime Format
```
D0.L = YYYYYYYMMMMMDDDDDHHHHHMMMMMMSSSSS (binary)
Bits 25-31: year - 1980    Bits 21-24: month (1-12)
Bits 16-20: day (1-31)     Bits 11-15: hour (0-23)
Bits 5-10:  minute (0-59)  Bits 0-4:   seconds/2 (0-29)
```

### XBIOS Rsconf Baud Rates
| Value | Baud | Value | Baud |
|---|---|---|---|
| 0 | 19200 | 8 | 600 |
| 1 | 9600 | 9 | 300 |
| 2 | 4800 | 10 | 200 |
| 4 | 2400 | 11 | 150 |
| 7 | 1200 | 12 | 134 |

### MFP Interrupt Numbers (for Jdisint/Jenabint/Mfpint)
| Int# | Source | Vector Addr |
|---|---|---|
| 0 | Centronics busy | $100 |
| 1 | RS-232 DCD | $104 |
| 2 | RS-232 CTS | $108 |
| 3 | Blitter done | $10C |
| 4 | Timer D (baud rate) | $110 |
| 5 | Timer C (200 Hz system) | $114 |
| 6 | Keyboard/MIDI ACIA | $118 |
| 7 | FDC/HDC done | $11C |
| 8 | Timer B (HBL count) | $120 |
| 9 | Transmit error | $124 |
| 10 | Transmit buffer empty | $128 |
| 11 | Receive error | $12C |
| 12 | Receive buffer full | $130 |
| 13 | Timer A (user) | $134 |
| 14 | RS-232 ring detect | $138 |
| 15 | Monochrome detect | $13C |

---

## Line-A Functions

Direct opcodes in code (not TRAP-based). The 68000 interprets these as "Line-A emulator" exceptions.

| Opcode | Function | Description |
|---|---|---|
| `$A000` | linea_init | Initialize Line-A; returns: D0/A0=variable base, A1=font headers, A2=routine ptrs |
| `$A001` | put_pixel | Draw pixel at PTSIN[0,1] with color INTIN[0] |
| `$A002` | get_pixel | Read pixel at PTSIN[0,1]; returns color in D0 |
| `$A003` | draw_line | Draw line from X1,Y1 to X2,Y2 using LNMASK, WMODE, COLBITn |
| `$A004` | horiz_line | Draw horizontal line (fast) using X1,X2,Y1 |
| `$A005` | filled_rect | Fill rectangle X1,Y1 to X2,Y2 using PATPTR, WMODE, COLBITn |
| `$A006` | filled_poly | Fill polygon (one scanline at a time) |
| `$A007` | bitblt | Bit block transfer — parameter block pointed to by A6 (see below) |
| `$A008` | textblt | Text block transfer — uses SRCX/SRCY, DESTX/DESTY, FBASE, STYLE |
| `$A009` | show_mouse | Show mouse cursor (decrements M_HID_CT) |
| `$A00A` | hide_mouse | Hide mouse cursor (increments M_HID_CT) |
| `$A00B` | transform_mouse | Change mouse cursor form (data pointed to by A0) |
| `$A00C` | undraw_sprite | Remove sprite (restore saved background) |
| `$A00D` | draw_sprite | Draw sprite (save background first) |
| `$A00E` | copy_raster | Copy raster form (like vro_cpyfm) |
| `$A00F` | seedfill | Flood fill from PTSIN[0,1] |

### Line-A Variable Block (returned by $A000 in A0)

After `$A000`, A0 points to offset +0. Variables are at both positive and negative offsets.

**Positive offsets — Graphics state:**

| Offset | Name | Size | Description |
|---|---|---|---|
| +$00 | PLANES | W | Number of bit planes |
| +$02 | WIDTH | W | Screen width in bytes (80 or 160) |
| +$04 | CONTRL | L | Pointer to VDI CONTRL array |
| +$08 | INTIN | L | Pointer to VDI INTIN array |
| +$0C | PTSIN | L | Pointer to VDI PTSIN array |
| +$10 | INTOUT | L | Pointer to VDI INTOUT array |
| +$14 | PTSOUT | L | Pointer to VDI PTSOUT array |
| +$18 | COLBIT0 | W | Color bit value for plane 0 |
| +$1A | COLBIT1 | W | Color bit value for plane 1 |
| +$1C | COLBIT2 | W | Color bit value for plane 2 |
| +$1E | COLBIT3 | W | Color bit value for plane 3 |
| +$20 | LSTLIN | W | Last line pixel flag (0=draw) |
| +$22 | LNMASK | W | Line style mask pattern |
| +$24 | WMODE | W | Writing mode (0=replace, 1=transparent, 2=XOR, 3=reverse transparent) |
| +$26 | X1 | W | Line/rect X1 coordinate |
| +$28 | Y1 | W | Line/rect Y1 coordinate |
| +$2A | X2 | W | Line/rect X2 coordinate |
| +$2C | Y2 | W | Line/rect Y2 coordinate |
| +$2E | PATPTR | L | Pointer to fill pattern buffer |
| +$32 | PATMSK | W | Fill pattern index mask (length-1) |
| +$34 | MFILL | W | Multi-plane fill (0=single plane) |
| +$36 | CLIP | W | Clipping flag (0=off, nonzero=on) |
| +$38 | XMINCL | W | Clip rectangle minimum X |
| +$3A | YMINCL | W | Clip rectangle minimum Y |
| +$3C | XMAXCL | W | Clip rectangle maximum X |
| +$3E | YMAXCL | W | Clip rectangle maximum Y |

**TextBlt variables (positive offsets, used by $A008):**

| Offset | Name | Size | Description |
|---|---|---|---|
| +$40 | XDDA | W | X DDA accumulator (init $8000) |
| +$42 | DDAINC | W | Fractional scaling increment |
| +$44 | SCALDIR | W | Scale direction (0=down, 1=up) |
| +$46 | MONO | W | Monospaced font flag |
| +$48 | SRCX | W | Source X in font form |
| +$4A | SRCY | W | Source Y in font form |
| +$4C | DESTX | W | Destination X on screen |
| +$4E | DESTY | W | Destination Y on screen |
| +$50 | DELX | W | Character width |
| +$52 | DELY | W | Character height |
| +$54 | FBASE | L | Pointer to font data |
| +$58 | FWIDTH | W | Font form width in bytes |
| +$5A | STYLE | W | Text effects: bit0=bold, 1=light, 2=italic, 3=underline, 4=outline |
| +$5C | LITEMASK | W | Lighten mask ($5555) |
| +$5E | SKEWMASK | W | Italic skew mask |
| +$60 | WEIGHT | W | Bold thicken width |
| +$62 | ROFF | W | Skew offset above baseline |
| +$64 | LOFF | W | Skew offset below baseline |
| +$66 | SCALE | W | Scaling flag (0=none) |
| +$68 | CHUP | W | Character rotation (0/900/1800/2700) |
| +$6A | TEXTFG | W | Text foreground color |
| +$6C | SCRTCHP | L | Pointer to text effects scratch buffers |
| +$70 | SCRPT2 | W | Offset to second scratch buffer |
| +$72 | TEXTBG | W | Text background color |
| +$74 | COPYTRAN | W | Raster copy mode (0=opaque, nonzero=transparent) |
| +$76 | SEEDABORT | L | Pointer to seedfill abort callback |

**Negative offsets — Display, font, cursor:**

| Offset | Name | Size | Description |
|---|---|---|---|
| -$02 | BYTES_LIN | W | Screen width in bytes |
| -$04 | V_REZ_VT | W | Vertical resolution in pixels |
| -$0A | V_OFF_AD | L | Pointer to font character offset table |
| -$0C | V_REZ_HZ | W | Horizontal resolution in pixels |
| -$0E | V_FNT_WD | W | Font form width in bytes |
| -$10 | V_FNT_ST | W | First ASCII code in font |
| -$12 | V_FNT_ND | W | Last ASCII code in font |
| -$16 | V_FNT_AD | L | Pointer to font data |
| -$17 | V_CUR_CT | B | Cursor flash countdown timer |
| -$18 | V_PERIOD | B | Cursor flash period (frames) |
| -$1A | V_CUR_XY+2 | W | Alpha cursor Y (cell position) |
| -$1C | V_CUR_XY | W | Alpha cursor X (cell position) |
| -$1E | V_CUR_OF | W | Byte offset screen base → cursor cell |
| -$22 | V_CUR_AD | L | Alpha cursor screen address |
| -$24 | V_COL_FG | W | Foreground color index |
| -$26 | V_COL_BG | W | Background color index |
| -$28 | V_CEL_WR | W | Bytes to next vertical cell row |
| -$2A | V_CEL_MY | W | Max cell Y (rows - 1) |
| -$2C | V_CEL_MX | W | Max cell X (columns - 1) |
| -$2E | V_CEL_HT | W | Cell height in pixels |

**Negative offsets — Mouse state:**

| Offset | Name | Size | Description |
|---|---|---|---|
| -$152 | MOUSE_FLAG | B | Mouse interrupt processing enabled |
| -$153 | CUR_FLAG | B | Draw mouse on VBL if nonzero |
| -$156 | CUR_Y | W | Mouse cursor Y |
| -$158 | CUR_X | W | Mouse cursor X |
| -$15A | V_HID_CNT | W | Alpha cursor hide depth |
| -$15C | CUR_MS_STAT | B | Mouse status: bits 0-1=buttons, 5=moved |
| -$254 | MOUSE_BT | W | Mouse button state (bit 0=left, bit 1=right) |
| -$256 | M_HID_CT | W | Mouse cursor hide depth counter |
| -$258 | GCURY | W | **Current mouse Y position** |
| -$25A | GCURX | W | **Current mouse X position** |
| -$34E | M_CDB_FG | W | Mouse foreground color |
| -$350 | M_CDB_BG | W | Mouse background color |
| -$352 | M_PLANES | W | Mouse draw mode (1=normal, -1=XOR) |
| -$354 | M_POS_HY | W | Mouse hotspot Y in 16x16 form |
| -$356 | M_POS_HX | W | Mouse hotspot X in 16x16 form |
| -$38A | CUR_FONT | L | Pointer to current font header |

---

## GEM AES Functions (TRAP #2, D0=200)

Call convention: `MOVE.L #aes_params,D1; MOVE.W #200,D0; TRAP #2` — no stack cleanup needed.

Parameter block: AES_Params → Control[5], Global[14], Int_In[16], Int_Out[7], Addr_In[3], Addr_Out[1].
Control[0]=function#, Int_Out[0]=return value.

| Func# | Name | Key Parameters | Description |
|---|---|---|---|
| 10 | appl_init | — | Register app; returns ap_id |
| 19 | appl_exit | — | Deregister app |
| 20 | evnt_keybd | — | Wait for key; Out[0]=key |
| 23 | evnt_mesag | Addr_In[0]=msg_buf | Wait for message |
| 25 | evnt_multi | In[0]=event_flags | Wait for multiple events |
| 30 | menu_bar | In[0]=show; Addr_In[0]=tree | Show/hide menu |
| 42 | objc_draw | In[0]=obj, In[1]=depth; Addr_In[0]=tree | Draw object tree |
| 50 | form_do | In[0]=start; Addr_In[0]=tree | Handle dialog |
| 52 | form_alert | In[0]=default; Addr_In[0]=string | Alert box |
| 54 | form_center | Addr_In[0]=tree | Center dialog |
| 77 | graf_handle | — | Get VDI handle |
| 78 | graf_mouse | In[0]=form | Set mouse shape |
| 100 | wind_create | In[0]=kind, In[1-4]=rect | Create window |
| 101 | wind_open | In[0]=handle, In[1-4]=rect | Open window |
| 102 | wind_close | In[0]=handle | Close window |
| 104 | wind_get | In[0]=handle, In[1]=field | Get attribute |
| 105 | wind_set | In[0]=handle, In[1]=field | Set attribute |
| 107 | wind_update | In[0]=flag(1-4) | Lock/unlock screen |
| 110 | rsrc_load | Addr_In[0]=filename | Load .RSC file |
| 112 | rsrc_gaddr | In[0]=type, In[1]=index | Get resource address |

**evnt_multi flags**: MU_KEYBD=1, MU_BUTTON=2, MU_M1=4, MU_M2=8, MU_MESAG=$10, MU_TIMER=$20

**Key messages**: MN_SELECTED=10, WM_REDRAW=20, WM_TOPPED=21, WM_CLOSED=22, WM_FULLED=23, WM_SIZED=27, WM_MOVED=28, AC_OPEN=40, AC_CLOSE=41, AP_TERM=50

For full GEM reference see `gem-quick-ref.md`.

---

## GEM VDI Functions (TRAP #2, D0=115)

Call convention: `MOVE.L #vdi_params,D1; MOVE.W #115,D0; TRAP #2` — no stack cleanup needed.

Parameter block: VDI_Params → contrl[12], intin[128], ptsin[128], intout[128], ptsout[128].
contrl[0]=opcode, contrl[5]=sub-opcode, contrl[6]=handle.

| Opcode | Name | Description |
|---|---|---|
| 1 | v_opnwk | Open physical workstation |
| 3 | v_clrwk | Clear workstation |
| 6 | v_pline | Draw polyline |
| 8 | v_gtext | Draw text |
| 9 | v_fillarea | Draw filled area |
| 12 | vst_height | Set text height (pixels) |
| 15 | vsl_type | Set line style (1-7) |
| 17 | vsl_color | Set line color |
| 22 | vst_color | Set text color |
| 23 | vsf_interior | Set fill type (0=hollow..4=user) |
| 25 | vsf_color | Set fill color |
| 32 | vswr_mode | Set writing mode (1=replace, 2=transparent, 3=XOR, 4=erase) |
| 100 | v_opnvwk | Open virtual workstation |
| 101 | v_clsvwk | Close virtual workstation |
| 106 | vst_effects | Set text effects (bit field) |
| 109 | vro_cpyfm | Raster copy opaque |
| 114 | vr_recfl | Fill rectangle |
| 125 | v_bar | Draw bar/filled rect |
| 129 | vs_clip | Set clipping (intin[0]=on/off) |
| 134 | vrt_cpyfm | Raster copy transparent |

For full GEM reference see `gem-quick-ref.md`.

---

## 68000 Exception Vector Table

Vectors occupy addresses $000-$3FF in RAM. Each is a 32-bit pointer.

| Address | Vector# | Exception |
|---|---|---|
| $00 | 0 | Reset: Initial SSP |
| $04 | 1 | Reset: Initial PC |
| $08 | 2 | **Bus Error** |
| $0C | 3 | **Address Error** |
| $10 | 4 | **Illegal Instruction** |
| $14 | 5 | **Division by Zero** |
| $18 | 6 | **CHK Instruction** |
| $1C | 7 | **TRAPV Instruction** |
| $20 | 8 | **Privilege Violation** |
| $24 | 9 | **Trace** |
| $28 | 10 | Line-A Emulator |
| $2C | 11 | Line-F Emulator |
| $60 | 24 | Spurious Interrupt |
| $64 | 25 | Level 1 Autovector (TT MFP) |
| $68 | 26 | Level 2 Autovector (HBL) |
| $6C | 27 | Level 3 Autovector |
| $70 | 28 | **Level 4 Autovector** (VBL) |
| $74 | 29 | Level 5 Autovector |
| $78 | 30 | Level 6 Autovector (MFP) |
| $7C | 31 | Level 7 Autovector (NMI) |
| $80 | 32 | TRAP #0 |
| $84 | 33 | **TRAP #1** (GEMDOS) |
| $88 | 34 | TRAP #2 (GEM/AES/VDI) |
| $B4 | 45 | **TRAP #13** (BIOS) |
| $B8 | 46 | **TRAP #14** (XBIOS) |

---

## Atari ST Memory Map (Key Locations)

| Address | Content |
|---|---|
| $000-$3FF | Exception vector table (256 vectors × 4 bytes) |
| $400-$5FF | System variables (TOS) |
| $800+ | TPA (Transient Program Area) — programs load here |
| $FA0000+ | I/O registers (ST MFP, ACIA, etc.) |
| $FF8200+ | Video controller (Shifter) registers |
| $FF8800+ | Sound chip (YM2149/PSG) registers |
| $FFFC00+ | Keyboard/MIDI ACIA registers |

### System Variables ($400-$512)

| Address | Name | Size | Purpose |
|---|---|---|---|
| $400 | etv_timer | L | Timer interrupt chain vector |
| $404 | etv_critic | L | Critical error chain vector |
| $408 | etv_term | L | Process terminate vector |
| $420 | memvalid | L | Memory validity marker ($752019F3) |
| $424 | memcntlr | W | Memory controller config |
| $426 | resvalid | L | Validates resvector |
| $42A | resvector | L | RESET bailout vector |
| $42E | phystop | L | Physical top of RAM |
| $432 | _membot | L | Bottom of available memory |
| $436 | _memtop | L | Top of available memory |
| $43A | memval2 | L | Validates memcntlr |
| $43E | flock | W | Floppy/DMA lock (non-zero = locked) |
| $440 | seekrate | W | Default floppy seek rate |
| $442 | _timr_ms | W | System timer calibration (ms) |
| $444 | _fverify | W | Non-zero: verify on floppy write |
| $446 | _bootdev | W | Boot device number |
| $448 | palmode | W | Non-zero: PAL mode |
| $44A | defshiftmd | W | Default video resolution |
| $44C | sshiftmd | W | Shadow of shiftmd register |
| $44E | _v_bas_ad | L | **Logical screen base address** |
| $452 | vblsem | W | VBL semaphore (mutex) |
| $454 | nvbls | W | Number of VBL handler slots |
| $456 | _vblqueue | L | Pointer to VBL handler array |
| $45A | colorptr | L | Pointer to palette setup (or NULL) |
| $45E | screenpt | L | Pointer to screen base setup (or NULL) |
| $462 | _vbclock | L | **VBL counter** (incremented each unblocked VBL) |
| $466 | _frclock | L | **Frame counter** (free-running, every VBL) |
| $46A | hdv_init | L | Hard disk init vector |
| $46E | swv_vec | L | Resolution change bailout vector |
| $472 | hdv_bpb | L | Disk "get BPB" vector |
| $476 | hdv_rw | L | Disk read/write vector |
| $47A | hdv_boot | L | Disk "get boot sector" vector |
| $47E | hdv_mediach | L | Disk media change vector |
| $482 | _cmdload | W | Non-zero: load COMMAND.COM |
| $484 | conterm | B | Console/keyboard config bits |
| $4A2 | savptr | L | Register save area pointer |
| $4A6 | _nflops | W | Number of floppy drives |
| $4BA | _hz_200 | L | **200 Hz raw system timer tick** |
| $4C2 | _drvbits | L | Bit vector of connected drives |
| $4C6 | _dskbufp | L | Pointer to common disk buffer |
| $4CE | _vbl_list | L | Initial VBL handler queue |
| $4EE | _dumpflg | W | Screen dump flag |
| $4F2 | _sysbase | L | Base of OS in ROM |
| $4F6 | _shell_p | L | Global shell info pointer |

### Video Shifter Registers ($FF8200+)

| Address | Name | R/W | Purpose |
|---|---|---|---|
| $FF8201 | dbaseh | R/W | Screen base address high byte |
| $FF8203 | dbasel | R/W | Screen base address mid byte |
| $FF8205 | vcounthi | R | **Display counter high** (current scan position) |
| $FF8207 | vcountmid | R | Display counter mid |
| $FF8209 | vcountlow | R | Display counter low |
| $FF820A | syncmode | R/W | Video sync mode |
| $FF8240-$FF825E | color0-15 | R/W | 16 palette registers (word each, $0RGB format) |
| $FF8260 | shiftmd | R/W | **Resolution** (0=low 320x200, 1=med 640x200, 2=high 640x400) |

### YM2149 Sound Chip ($FF8800)

Access: write register number to $FF8800, then read/write data at $FF8800/$FF8802.

| Reg# | Name | Bits | Description |
|---|---|---|---|
| 0 | tone_A_fine | 7-0 | Channel A tone period (fine) |
| 1 | tone_A_coarse | 3-0 | Channel A tone period (coarse) |
| 2 | tone_B_fine | 7-0 | Channel B tone period (fine) |
| 3 | tone_B_coarse | 3-0 | Channel B tone period (coarse) |
| 4 | tone_C_fine | 7-0 | Channel C tone period (fine) |
| 5 | tone_C_coarse | 3-0 | Channel C tone period (coarse) |
| 6 | noise_period | 4-0 | Noise generator period |
| 7 | **mixer** | 7-0 | **I/O and mixer control** (see below) |
| 8 | amp_A | 4-0 | Channel A amplitude (bit 4: 1=use envelope) |
| 9 | amp_B | 4-0 | Channel B amplitude (bit 4: 1=use envelope) |
| 10 | amp_C | 4-0 | Channel C amplitude (bit 4: 1=use envelope) |
| 11 | env_fine | 7-0 | Envelope period (fine) |
| 12 | env_coarse | 7-0 | Envelope period (coarse) |
| 13 | env_shape | 3-0 | Envelope shape (0-15) |
| 14 | **port_A** | 7-0 | **I/O port A** (directly accessible via Giaccess/Ongibit/Offgibit) |
| 15 | port_B | 7-0 | I/O port B (directly accessible via Giaccess) |

**Register 7 (mixer) bits**: bit0=tone A off, 1=tone B off, 2=tone C off, 3=noise A off, 4=noise B off, 5=noise C off, 6=port A direction, 7=port B direction

**Port A bits**: bit 0=disk side 0, 1=disk drive A select, 2=disk drive B select, 3=RS-232 RTS, 4=RS-232 DTR, 5=centronics strobe, 6=GPO

### MFP 68901 Registers ($FFFFFA00+)

The MFP handles interrupts, timers, and serial I/O. All registers are at odd addresses (byte-wide).

| Address | Name | Description |
|---|---|---|
| $FFFFFA01 | **GPIP** | General purpose I/O (bit 7=mono detect, bit 5=FDC, bit 4=keyboard ACIA) |
| $FFFFFA03 | AER | Active edge register (0=falling, 1=rising) |
| $FFFFFA05 | DDR | Data direction register (0=input, 1=output) |
| $FFFFFA07 | **IERA** | Interrupt enable A (bits 0-7 = ints 8-15) |
| $FFFFFA09 | **IERB** | Interrupt enable B (bits 0-7 = ints 0-7) |
| $FFFFFA0B | IPRA | Interrupt pending A |
| $FFFFFA0D | IPRB | Interrupt pending B |
| $FFFFFA0F | ISRA | Interrupt in-service A |
| $FFFFFA11 | ISRB | Interrupt in-service B |
| $FFFFFA13 | IMRA | Interrupt mask A |
| $FFFFFA15 | IMRB | Interrupt mask B |
| $FFFFFA17 | VR | Interrupt vector base (upper 4 bits) |
| $FFFFFA19 | **TACR** | Timer A control (prescaler/mode) |
| $FFFFFA1B | **TBCR** | Timer B control (prescaler/mode) |
| $FFFFFA1D | **TCDCR** | Timer C+D control (upper nibble=C, lower=D) |
| $FFFFFA1F | TADR | Timer A data register |
| $FFFFFA21 | TBDR | Timer B data register |
| $FFFFFA23 | TCDR | Timer C data register |
| $FFFFFA25 | TDDR | Timer D data register |
| $FFFFFA27 | SCR | Sync character register |
| $FFFFFA29 | UCR | USART control register |
| $FFFFFA2B | RSR | Receiver status register |
| $FFFFFA2D | TSR | Transmitter status register |
| $FFFFFA2F | UDR | USART data register |

**Timer control prescaler values**: 0=stopped, 1=÷4, 2=÷10, 3=÷16, 4=÷50, 5=÷64, 6=÷100, 7=÷200. Timer A/B also support event-count mode (bit 3=1).

**Timer usage**: A=user/application, B=HBL counting/raster effects, C=system 200Hz ($114), D=RS-232 baud rate

### Disk Controller ($FF8600+)

| Address | Name | Description |
|---|---|---|
| $FF8604 | diskctl | FDC/HDC data register (select via $FF8606) |
| $FF8606 | fifo | DMA mode/status register |
| $FF8609 | dmahigh | DMA base address high byte |
| $FF860B | dmamid | DMA base address mid byte |
| $FF860D | dmalow | DMA base address low byte |

**FDC register select** (written to $FF8606 before accessing $FF8604): $80=command, $82=track, $84=sector, $86=data

### Keyboard/MIDI ACIAs ($FFFFFC00+)

| Address | Name | Description |
|---|---|---|
| $FFFFFC00 | keyctl | Keyboard ACIA control/status |
| $FFFFFC02 | keybd | Keyboard ACIA data |
| $FFFFFC04 | midictl | MIDI ACIA control/status |
| $FFFFFC06 | midi | MIDI ACIA data |

### Atari ST Screen Formats

| Mode | Resolution | Colors | Bitplanes | Bytes/line | Total |
|---|---|---|---|---|---|
| 0 (Low) | 320×200 | 16 | 4 | 160 | 32,000 |
| 1 (Med) | 640×200 | 4 | 2 | 160 | 32,000 |
| 2 (High) | 640×400 | 2 | 1 | 80 | 32,000 |

**Bitplane interleaving** (Med/Low): Words alternate between planes.
- Low: W0P0, W0P1, W0P2, W0P3, W1P0, W1P1, W1P2, W1P3, ...
- Med: W0P0, W0P1, W1P0, W1P1, W2P0, W2P1, ...
- High: Simple linear — 1 bit per pixel, 8 pixels per byte

---

## Atari ST Keyboard Scancodes (Function Keys)

| Key | Scancode | Shift+ |
|---|---|---|
| F1 | $3B | $54 |
| F2 | $3C | $55 |
| F3 | $3D | $56 |
| F4 | $3E | $57 |
| F5 | $3F | $58 |
| F6 | $40 | $59 |
| F7 | $41 | $5A |
| F8 | $42 | $5B |
| F9 | $43 | $5C |
| F10 | $44 | $5D |
| Help | $62 | — |
| Undo | $61 | — |
| Cursor Up | $48 | — |
| Cursor Down | $50 | — |
| Cursor Left | $4B | — |
| Cursor Right | $4D | — |
| Return | $1C | — |
| Backspace | $0E | — |
| Delete | $53 | — |
| Tab | $0F | — |
| Escape | $01 | — |
| Space | $39 | — |

---

## DTA (Disk Transfer Area) Structure

Set by GEMDOS Fsetdta ($1A), filled by Fsfirst ($4E) and Fsnext ($4F).

| Offset | Size | Content |
|---|---|---|
| $00-$14 | 21 | Reserved (used by GEMDOS internally) |
| $15 | 1 | File attributes |
| $16 | 2 | Time (packed: hours×2048 + minutes×32 + seconds/2) |
| $18 | 2 | Date (packed: (year-1980)×512 + month×32 + day) |
| $1A | 4 | File size (bytes, big-endian longword) |
| $1E | 14 | Filename (8.3 format, null-terminated) |

### File Attribute Bits
| Bit | Attribute |
|---|---|
| 0 | Read-only |
| 1 | Hidden |
| 2 | System |
| 3 | Volume label |
| 4 | Subdirectory |
| 5 | Archive |

---

## Atari ST Executable (Basepage) Structure

When TOS loads a program, it creates a basepage at the start of the allocated memory:

| Offset | Size | Content |
|---|---|---|
| $00 | L | Pointer to start of TPA (= basepage address) |
| $04 | L | Pointer to end of TPA |
| $08 | L | **Pointer to TEXT segment** |
| $0C | L | TEXT segment size |
| $10 | L | **Pointer to DATA segment** |
| $14 | L | DATA segment size |
| $18 | L | **Pointer to BSS segment** |
| $1C | L | BSS segment size |
| $20 | L | Pointer to DTA (default = basepage+$80) |
| $24 | L | Pointer to parent's basepage |
| $2C | L | **Pointer to environment string** |
| $80 | 128 | Command line (first byte = length) |
