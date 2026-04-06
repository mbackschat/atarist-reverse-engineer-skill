# Atari ST Reverse-Engineering Skill

A Claude Code skill that guides the reverse-engineering of Atari ST (Motorola 68000) binaries, producing three deliverables:

- **`SOURCE.txt`** — Fully annotated 68000 disassembly listing
- **`ANALYSIS.md`** — Technical analysis document
- **`MANUAL.md`** — User manual (for interactive tools)

The methodology, prompts, Python tooling, and reference material are generalized to work with any Atari ST binary.

---

## Installation

Add the skill to your Claude Code project-scoped configuration:

```bash
claude mcp add-skill atari-st-reverse-engineer /path/to/atarist-reverse-engineer-skill/SKILL.md
```

Or add it manually to `.claude/settings.json` in your project:

```json
{
  "skills": [
    "/path/to/atarist-reverse-engineer-skill/SKILL.md"
  ]
}
```

Replace `/path/to/` with the actual location where you cloned this repo.

Once installed, the skill appears as the `/atari-st-reverse-engineer` slash command.

---

## Quick Start

### As a Claude Code skill

```
/atari-st-reverse-engineer path/to/BINARY.PRG
```

Claude will follow the 6-phase playbook automatically: scan the binary, identify sections, analyze code with parallel agents, build annotations, generate all three deliverables, and review them.

### Manual workflow

If you prefer step-by-step control, follow [plan.md](plan.md) — the full reverse-engineering playbook.

---

## What It Does

The skill orchestrates a complete reverse-engineering pipeline:

```
Binary File (.PRG / .TOS / .ACC / raw)
    │
    ├─ Phase 1: Setup ──────── Copy disassembler, install capstone via uv
    │
    ├─ Phase 2: Initial Scan ── Parse header, extract strings, find system calls,
    │                            map subroutines, identify code vs data regions
    │
    ├─ Phase 3: Deep Analysis ─ Launch parallel agents to analyze each section:
    │                            trace logic, identify algorithms, document
    │                            subroutine signatures (entry/exit registers)
    │
    ├─ Phase 4: Annotate ────── Build annotations.py with block comments
    │                            (routine headers) and inline comments
    │                            (per-instruction explanations), regenerate listing
    │
    ├─ Phase 5: Document ────── Write ANALYSIS.md (technical) and MANUAL.md (user)
    │                            from the analysis findings
    │
    ├─ Phase 6: Review ──────── Cross-check all deliverables against raw binary
    │                            with parallel reviewer agents
    │
    └─ Phase 7: Pseudocode ─── Write pseudocode for key algorithms and main logic,
                                 insert into ANALYSIS.md subsystem sections
```

---

## File Structure

```
atari-st-reverse-engineer-skill/
│
├── SKILL.md                           Skill definition (YAML frontmatter + instructions)
├── plan.md                            Step-by-step RE playbook (6 phases, 10 tips)
├── README.md                          This file
│
├── prompts/
│   ├── analysis-sections.md           9 template prompts for code section analysis
│   │                                  (entry point, I/O, exceptions, commands,
│   │                                   assembler, screen, keyboard, file I/O, debugger)
│   ├── annotation-guide.md            Style guide for writing 68000 annotations
│   │                                  (block comments, inline comments, 68000 tricks,
│   │                                   TOS call documentation, loop markers)
│   └── review-checklist.md            3 reviewer agent prompts for cross-checking
│                                      SOURCE.txt, ANALYSIS.md, and MANUAL.md
│
├── scripts/
│   ├── disasm_atari.py                Generalized 68000 disassembler & analyzer (955 lines)
│   │                                  Capstone-based with byte-pattern scanning for
│   │                                  TRAP/Line-A/BSR/RTS identification
│   ├── annotations_template.py        Starter annotation file with format examples
│   └── requirements.txt               Python dependency: capstone>=5.0.7
│
└── reference/
    ├── tos-quick-ref.md               Condensed TOS reference (327 lines):
    │                                  GEMDOS/BIOS/XBIOS/Line-A function tables,
    │                                  exception vectors, memory map, DTA layout,
    │                                  basepage structure, keyboard scancodes,
    │                                  screen formats, hardware registers
    ├── TOS.TXT                        Complete TOS system call reference (270 KB)
    ├── GEMDOS.TXT                     Full GEMDOS reference (file I/O, memory, console)
    ├── BIOS.TXT                       BIOS quick reference
    ├── BIOS_Calls _Trap_13.TXT        BIOS Trap #13 detailed reference
    ├── XBIOS.TXT                      XBIOS reference (screen, mouse, timers, sound)
    ├── AES.md                         AES (Application Environment Services) reference
    ├── AES_CALL.TXT                   AES function calls (detailed)
    ├── GDOS_INF.TXT                   GDOS (Graphical Device OS) reference
    ├── SALAD.TXT                      System Assembly Language documentation
    ├── 68000_Assembly_Language.txt     68000 instruction set reference
    ├── INTRO68K.txt                   Introduction to 68000 programming
    └── atari.s                        Atari ST hardware register definitions
                                       (display, DMA, sound, MFP, ACIA)
```

---

## The Disassembler

`scripts/disasm_atari.py` is a standalone Python tool that reads any Atari ST binary and produces an annotated assembly listing.

### Usage

```bash
# Initial reconnaissance — just show extracted strings
python disasm_atari.py BINARY.PRG --strings-only

# Full disassembly with default prefix
python disasm_atari.py BINARY.PRG --prefix TOOLNAME
```

### What it does automatically

- Parses the 28-byte Atari ST executable header ($601A magic)
- Extracts all printable strings with their code offsets
- Identifies **TRAP #1** (GEMDOS), **TRAP #13** (BIOS), **TRAP #14** (XBIOS) system calls by scanning backwards from each TRAP instruction to find the function number push
- Identifies **TRAP #2** (GEM AES/VDI) calls by detecting `MOVE.W #200,D0` (AES) or `MOVE.W #115,D0` (VDI), with best-effort function resolution from the parameter block
- Identifies **Line-A** graphics calls ($A000–$A00F)
- Maps all **BSR/JSR** call targets (subroutine entry points)
- Maps all **Bcc/BRA** branch targets
- Finds all **RTS/RTE** instructions (subroutine boundaries)
- Resolves **LEA xxx(PC)** string and data references
- Produces a Capstone-based disassembly with all the above as annotations

### What you add during analysis

- **`KNOWN_SUBS`** dict: map offsets to meaningful subroutine names
- **`SECTIONS`** list: define section boundaries with descriptions
- **Data regions**: mark font data, string tables, opcode tables
- **`annotations.py`**: block and inline comments (loaded from working directory)

### Built-in TOS databases

The disassembler includes complete function databases for:

| Database | Coverage |
|---|---|
| GEMDOS | 38 functions ($00–$57): file I/O, memory, process control, console |
| BIOS | 12 functions ($00–$0B): device I/O, vectors, keyboard |
| XBIOS | 32 functions ($00–$27): screen, mouse, timers, sound, serial, floppy |
| AES | 65+ functions (10–130): windows, menus, dialogs, forms, resources |
| VDI | 50+ functions (1–134): graphics, text, raster, input, attributes |
| Line-A | 16 functions ($A000–$A00F): graphics primitives, mouse cursor |

These automatically annotate every system call in the disassembly.

---

## The Annotation System

Annotations live in a separate `annotations.py` file that the disassembler imports at runtime, keeping analysis knowledge separate from the tool. See `scripts/annotations_template.py` for the format and `prompts/annotation-guide.md` for the style guide.

---

## Tips

1. **Start with strings** — `--strings-only` reveals the program's purpose in seconds
2. **System calls reveal structure** — clusters of file ops = file manager, Bconin = keyboard handler
3. **The main loop always exists** — find the `BRA.W` back to the top, that's your main loop
4. **A5 is usually the base register** — `LEA (PC),A5` at entry means all data is A5-relative
5. **Data looks like garbage** when disassembled — nonsensical instructions = font/string/table data
6. **PEA $XXXX is packed** — `PEA $000BFFFF` = Kbshift(-1), not a random address
7. **Low-memory writes = vectors** — anything writing to $00–$3FF is installing exception handlers
8. **ROL.L #8 cycles bytes** — the 68000 idiom for processing each byte of a longword
9. **Check the help text** — many Atari ST tools embed their entire help screen as a string block
10. **Write CONTEXT.md** — future you will thank present you
