
<img src="/images/Chapter2.png" width="680">

# *~De Re PlayNET*<br>Chapter 2: The Static View

In this chapter, we explore the structure and internals of the PlayNET client files that were distributed to its customers.

Here we will focus on the first floppy disk of the three-disk set, labeled "SYSTEM". The other two disks, labeled "GAMES 1" and "GAMES 2", will be the focus of Chapter 7.

Most modern software products require some initial installation process on the host computer before the application can be used. Even in the 1980s, some software products were distributed in a form that required an initial step of creating a working disk to prepare them for use.

PlayNET was not like that. It was shipped ready to go and required only that its bootstrap program be invoked to begin using it. In practical terms, the files on the disk represent a snapshot of the initial state of the functioning application before it is loaded into memory. That makes them the natural starting point for understanding its behavior.

## Chapter Contents

* [2.1 Distribution Media](#21-distribution-media)
  * [2.2 The "SYSTEM" Disk](#22-the-system-disk)
    * [2.2.1 File Naming Scheme](#221-file-naming-scheme)
    * [2.2.2 File Categories (Inferrred)](#222-file-categories-inferred)
      * [2.2.2.1 Bootstrap Files](#2221-bootstrap-files)
      * [2.2.2.2 Catalog Metadata Files](#2222-catalog-metadata-files)
      * [2.2.2.3 Module Manifest Files](#2223-module-manifest-files)
      * [2.2.2.4 Module Component Files](#2224-module-component-files)
      * [2.2.2.5 Unknown Files](#2225-unknown-files)
    * [2.2.3 Disk Catalog and Loader Modules](#223-disk-catalog-and-loader-modules)
      * [2.2.3.1 Common Kinds of Module Component Files (Inferred)](#2231-common-kinds-of-module-component-files-inferred)
      * [2.2.3.2 Module Manifests](#2232-module-manifests)
        * [2.2.3.2.1 MMM Module Manifest](#22321-mmm-module-manifest)
        * [2.2.3.2.2 ACC Module Manifest](#22322-acc-module-manifest)
        * [2.2.3.2.3 BBB Module Manifest](#22323-bbb-module-manifest)
        * [2.2.3.2.4 DLD Module Manifest](#22324-dld-module-manifest)
        * [2.2.3.2.5 LIB Module Manifest](#22325-lib-module-manifest)
        * [2.2.3.2.6 STP Module Manifest](#22326-stp-module-manifest)
        * [2.2.3.2.7 TLK Module Manifest](#22327-tlk-module-manifest)

<br>

## 2.1 Distribution Media

The PlayNET client software was distributed as 3-disk set.

<img src="/images/PlayNETDisks.jpg"> <img src="/images/PlayNETDiskBack.jpg">

Each disk in the set studied had a small paper label affixed to the upper-left corner of its front face. On the first disk, the label was white with red lettering and bore three lines of text, reading from top to bottom: "PlayNET™ ★", "SYSTEM", and "© PLAYNET, INC. 1984".

The other two disks had red labels with white lettering. Their text was identical to that of the first disk, except that "SYSTEM" was replaced by "GAMES 1" and "GAMES 2", respectively.

The physical disks were all black with uncrimped jacket folds. The front-side hub ring was glossy black. Ink-stamped on the right-side jacket fold on the back of all three was the mark "LNE85115582".

> **Intuitive Insights**: "LNE" is probably the manufacturer's identity but no information could be found about it. The "85" is possibly the year of manufacture, where the "115" represents the day (April 25), and "582" a batch number.

Next we examine the contents of the disks.

*Note: Unfortunately, although the project was able to acquire the authentic disk set described above, the data was not recoverable. The project was able to acquire copies of the first two disks (SYSTEM and GAMES 1) that was part of a COMAL Users Group archive. We have not been able to locate a copy of the GAMES 2 disk however.*

## 2.2 The "SYSTEM" Disk

This disk contains the main client application files, including the seven core loadable modules and the catalog metadata files. When a game on another disk is run, the user is prompted to insert this disk back into the drive when the game is finished.

The DOS directory header of the disk reads: `0 "NEW FSM SYSTEM  " NF 2A`

From this we can derive a few tidbits of information:
- The 16-character padded volume label is `NEW FSM SYSTEM  `. "FSM" is an abbreviation for "finite-state machine", which is a key concept of the software's architecture. We will cover the FSM in much greater detail later. A "new" FSM of course implies a previous one. This is consistent with the name of one of the files, `FSM2.MLP.E.A1`, suggesting a prior version might have been named "FSM1", "FSM", etc.
- The disk ID is `NF`, presumed to be an abbreviation for "new FSM".
- The volume is of course the standard Commodore 5.25-inch disk DOS format "2A".

> **Intuitive Insights**: The author is puzzled by the volume label. It seems that a more appropriate use of that data real estate would've been something along the lines of "PLAYNET SYSTEM" or "PLAYNET(TM) V2", etc. Something more consumable by the general public. This is a smell that calls into question the disk image's authenticity as an actual production release version. Maybe it's from an early access/beta testing release instead. Or, maybe it was simply an entertaining "engineering escape" from PlayNET devs!
 
Here are the names of the 125 files of the disk sorted alphabetically:

```
ACC             ACC.BAS.E.A1     ACC.CLR.E.A1     ACC.DO.E.A1    ACC.JMP.E.A1     ACCLEX;7.2.E.A1
ACC.PTR.E.A1    ACC.SCR.E.A1     ACC.SEND.E.A1    ACC.TBL.E.A1   ACC.VIC.A1       BBB
BB.BAS.E.A1     BBB.CLR.E.A1     BBB.PTR.E.A1     BBB.SCR.E.A1   BBB.TBL.E.A1     BBB.VIC.A1
BB.DO.E.A1      BB.JMP.E.A1      BBRINT;2.A1      BB.SEND.E.A1   BBSLEX;7.1.E.A1  CAT.ACC
CAT.BBB         CAT.DISK         CAT.DLD          CAT.FILE       CAT.GAME         CAT.ID
CAT.LIB         CAT.MMM          CAT.MODULE       CAT.PROGRAM    CAT.STP          CAT.TLK
DLD             DLD.4000.E.A1    DLD.ADDRESS.E.A1 DLD.BAS.E.A1   DLD.CLR.E.A1     DLD.DUMMY.1.A1
DLD.DUMMY.2.A1  DLD.JMPTBL.E.A1  DLDLEX;7.1.E.A1  DLD.PTR.E.A1   DLD.SCR.E.A1     DLD.SNDTAB.E.A1
DLD.TBL.E.A1    DLD.VIC.A1       DOINIT.E.A1      EDC.MLP.E.A1   FDEBUG.MLP.E.A1  FSM2.MLP.E.A1
GENRINT;1.A1    GTRINT;4.A1      KERNAL;2.E.A1    LIB            LIB.BAS.E.A1     LIB.CLR.E.A1
LIB.DO.E.A1     LIB.EFT1.E.A1    LIB.EFT2.E.A1    LIB.JMP.E.A1   LIBLEX;7.0.E.A1  LIB.PTR.E.A1
LIB.SCR.E.A1    LIB.SEND.E.A1    LIB.TBL.E.A1     LIB.VIC.A1     LOADER           LOADER;9
MMM             MMM;3/6.PTR.E.A1 MMM;3/6.TBL.E.A1 MMM.BAS.E.A1   MMM.CLR.E.A1     MMM.DO.E.A1
MMM.DUMMY.1.A1  MMM.DUMMY.2.A1   MMM.DUMMY.3.A1   MMM.DUMMY.4.A1 MMM.GENERAL.E.A1 MMM.JMP.E.A1
MMMLEX;7.0.E.A1 MMMRINT;5.A1     MMM.SCR.E.A1     MMM.SEND.E.A1  MMMSNDRTN.E.A1   MMM.SPR
MMM.VIC.A1      PLAYNET          STP              STP.BAS.E.A1   STP.CLR.E.A1     STP.JMP.MLP.E.A1
STPLEX;7.0.E.A1 STP.LO.MLP.E.A1  STP.PTR.E.A1     STP.SCR.E.A1   STP.SEND.E.A1    STP.TBL.E.A1
STP.VIC.A1      TALK.V5.E.A1     TLK              TLK.CLR.E.A1   TLK.DIS.E.A1     TLK.DO.E.A1
TLK.DUMMY.1.A1  TLK.DUMMY.2.A1   TLK.DUMMY.3.A1   TLK.GS.E.A1    TLK.JMP.E.A1     TLK.PTR.E.A1
TLK.SCR.E.A1    TLK.SEND.E.A1    TLK.TBL.E.A1     TLK.VIC.A1     XHCOMM;7.1.E.A1  XLCOMM;7.1.E.A1
XTHLEX;7.0.E.A1 XTLLEX;7.0.E.A1  ZBBMLP;10.E.A1   ZEDITH;10.E.A1 ZEDITL;10.E.A1
```

All of the files are type PRG except for seven which are type SEQ: `ACC`, `BBB`, `DLD`, `LIB`, `MMM`, `STP`, and `TLK`.

At first glance, the list looks like a jumbled mess of letters, numbers and punctuation. But there is a method to it, and with but a few curious exceptions it is remarkebly well-organized. The purpose of each file is explained in the following sections.

### 2.2.1 File Naming Scheme

When you study the names of the files for awhile, a few patterns begin to emerge:
- 101 file names end with ".A1".
- 81 of those end with "E.A1". They coincide with all compressed files.
- 7 names are only 3 characters in length. They coincide with the files of type SEQ.
- 81 names begin with one of those seven 3-character ones.
- Most of the names begin with one of the 3-characters prefixes, end with one of the "A1" suffixes, and share a number of inner period-delimited name elements, such as ".BAS.", ".DO.", ".JMP.", and several others.

These traits are indicitive of a fairly sophisticated *semantics-based* code and data classification system, and that in fact is the case. However, this system does not depend on consistency of file naming to function. The names of the files typically reflect their roles via conventions but they are not strictly followed. For example, the files named MMM.DO.E.A1 and STP.LO.MLP.E.A1 have the same functional role within their respective modules, but one name has the inner element .DO. while the other has .LO.MLP.

Instead of roles being determined by names, they are implied by their memory load locations as specified by their uncompressed PRG headers, or in some cases their non-standard locations are known implicitly by other associated module files, as is the case with MMM.DO.E.A1.

> **Intuitive Insights**: The author spent some time investigating these naming conventions and what they could mean in terms of revealing the software tools used by the PlayNET engineers. It hints at a level of sophistication beyond what was customery for 8-bit ecosystems at the time.
>
> For example, consider the file named "ACCLEX;7.2.E.A1". A character sequence that stands out is "7.2.E.A1". Offhand that looks like a revision tag from a source code control system, where the version tree may have been something like A1->E->2->7 for a file named ACCLEX.
>
> There is evidence from outside sources that PlayNET used Stratus computers for their servers. In the 1980s it was not uncommon for developers to use the same machines for software development that were used for production services, in an A/B switching mode for example, due to the higher cost in those days of running dedicated development machines. So it's not far-fetched to think that the files that made up the client floppy disk BOM may have been version-controlled using a system available for Stratus. Two candidates that come to mind are SCCS and RCS, but "7.2.E.A1" is not a valid revision tag for either of those systems.
>
> Then there is the semicolon to consider. We know that ".E.A1" is a common pattern among the file names with 81 instances of it. If we eliminate that substring from the name we are left with "ACCLEX;7.2". Then applying the same filter to "BBSLEX;7.1.E.A1" leaves "BBSLEX;7.1". As if 7.2 and 7.1 are semicolon-delimited major.minor version tags.
>
> We know that there was a semicolon-delimited file versioning system in wide use at that time by the DEC VMS operating system. But it only supported a single integer in its tag, not a dot-delimited one. So the file "LOADER;9" would've been a valid VMS file name, but "BBSLEX;7.1" would not have been. And then there's the file named "MMM;3/6.PTR.E.A1" to contend with!
>
> So in conclusion, we have no idea what, if any, revision control system may have been used to generate the revision tag info, presuming that's what it is. That seems to leave the following three possibilities: 1) the developers used some obscure 3rd-party product that is long-forgotten, 2) they used a customized management system of their own invention, probably also long-forgotten, or 3) they simply used manually-assigned tags that made sense to the few people who actually needed to know what they meant.

### 2.2.2 File Categories (Inferred)

As a result of the naming scheme and the semantics they reveal, the project categorizes the files according to six 6 logical groupings, all inferred from context. That is, there is no evidence that PlayNET engineers ever used the same categorizations, but they are consistent with analysis and observation.

|Category|Count|Purpose|
|---|---|---|
|Bootstrap|3|Files that participate in application loading|
|Catalog Metadata|13|Metadata files of the disk catalog system|
|Module Manifest|7|Files that describe the component files of a loader module|
|Module Component|93|Component files of a loader module|
|Unknown|9|Unknown|
|(TOTAL)|125| |

Each is described in detail below.

#### 2.2.2.1 Bootstrap Files

There are 3 files that stage the runtime so that it's ready to begin communicating with the service.  They chain-load in the following order: PLAYNET -> LOADER -> LOADER;9.

A more detailed description of the boot sequence can be found in Chapter 3.

|Name|Purpose|Type|Disk Size (Blocks)|Size (Bytes)|Load Address|Contents|
|---|---|---|---|---|---|---|
|PLAYNET|First directory entry/boot file|PRG|3|564|$00C6|native code + low-memory RAM image|
|LOADER|Bootstrap loader|PRG|7|1566|$0B01|tokenized BASIC|
|LOADER;9|Module loader|PRG|17|4083|$0800|tokenized BASIC stub + FIG-Forth ITC|

- **PLAYNET**: This executable exists solely to perform a one-shot handoff from the disk's boot sector to the next executable in the load chain, LOADER. Being the first directory entry, it is the file that is loaded when the user enters the BASIC command `LOAD"*",8,1`. It is not a normal program, but rather a low-memory RAM image that LOAD injects across the zero page, the stack, and page 2 of memory to inject a classic "autostart/autoboot" hack. The result is the user (or boot loader/cartridge) need only type the LOAD command to kick things off, no subsequent RUN command is needed.

- **LOADER**: This executable is a BASIC program that displays the colorful and animated "Welcome" screen and waits for a keypress. On keypress or timeout it will proceed to LOADing the next executable in the bootstrap chain, LOADER;9.

  <img src="/images/loader_screen.png">

- **LOADER;9**: This curiously named executable is the module loader, a key component of the application's runtime system. When triggered, it loads a specified module manifest file that contains a list of descriptors for the files that comprise the module. It then loads each of those component files, in order, writing them to memory according to the information in the component file's header.

  What makes this executable surprisingly unique among the others is that at its heart lies a FIG-Forth indirect-threaded code (ITC) virtual machine. We were able to decode a 75-colon-word program that revealed file-loading machinery as well as a canonical-Huffman decompressor compatible with the compression used by the distribution's files. Since a scan of the other code-bearing files revealed no additional decompression logic, we reasonably concluded that the role of LOADER;9 is not limited to bootstrapping but also extends into subsequent application phase transitions. This was later confirmed by observation.

> **Intuitive Insights**: After carefully comparing the executable with several Forth systems available to Commodore developers in the 1983-84 era, we have concluded that it is almost certainly a SAVETURNKEY program built by the C64-FORTH-79 compiler by Gregg Harris/Performance Micro Products.

#### 2.2.2.2 Catalog Metadata Files

There are 13 files associated with the disk catalog system organized into 4 groups according to role:
- Master files
- Per-disk files
- Per-module files
- Per-game files

The design of the disk catalog system is explained in more detail in section 2.2.3.

##### Master Files

The master files reside on the SYSTEM disk.

|Name|Purpose|Type|Disk Size (Blocks)|Size (Bytes)|Load Address|Contents|
|---|---|---|---|---|---|---|
|CAT.DISK|Catalog's per-disk free block counts|PRG|1|20|DLD module's working buffers|record-oriented data|
|CAT.GAME|Game registry|PRG|2|290|$8000|record-oriented data|
|CAT.ID|Catalog's identity|PRG|1|6|DLD module's working buffers|record-oriented data|
|CAT.MODULE|Module registry|PRG|1|239|$1000 (via $A800*)|record-oriented data|
|CAT.PROGRAM|Downloaded files registry|PRG|1|3|DLD module's working buffers|record-oriented data|

\* = temporary staging location to avoid clobbering the loader

- **CAT.DISK**:

  The data in this file is record-oriented with the following structure:

  Format: `[LA][RC]{[RL=$01][TD]}{[RL=$04][DI][BC]$00}...`
  - LA = load address (2 bytes LE) [Note: effectively ignored by DLD loader]
  - RC = record count (1 byte)
  - RL = record length (1 byte)
  - TD = top disk index (1 byte)
  - DI = disk index (1 byte)
  - BC = block count (2 bytes LE)

- **CAT.GAME**:

  The data in this file is record-oriented with the following structure:

  Format: `[LA][RC]{[RL][GK][PT][GI][PC][DN]}...`
  - LA = load address (2 bytes LE)
  - RC = record count (1 byte)
  - RL = record length (1 byte)
  - GK = game key (1 byte)
  - PT = product tag (4 bytes ASCII, e.g. "G010")
  - GI = game ID (3 bytes ASCII, e.g. "BCK")
  - PC = player count (3 bytes, 1st = min, 2nd = max, 3rd = default)
  - DN = display name (variable, 1st = PETSCII, rest = Screen Codes)

- **CAT.ID**:

  The data in this file is record-oriented with the following structure:

  Format: `[LA][RC]{?}...`
  - LA = load address (2 bytes LE) [Note: effectively ignored by DLD loader]
  - RC = record count (1 byte)
  - ?

- **CAT.MODULE**:

  The data in this file is record-oriented with the following structure:

  Format: `[LA][RC]{[RL][MK][PT][MI]$00[SK][DN]}...`
  - LA = load address (2 bytes LE)
  - RC = record count (1 byte)
  - RL = record length (1 byte)
  - MK = module key (1 byte)
  - PT = product tag (4 bytes ASCII, e.g. "S004")
  - MI = program ID (3 bytes ASCII, e.g. "MMM")
  - SK = selector key (1 byte ASCII)
  - HI = home indicator (1 byte, $01 if MI == "MMM" else $00) [Note: not well understood]
  - DN = display name (variable, 1st = PETSCII, rest = C64 Screen Codes)

- **CAT.PROGRAM**:

  The data in this file is record-oriented with the following structure:

  Format: `[LA][RC]{[PN][TC][DI][U1][U2]}...`
  - LA = load address (2 bytes LE) [Note: effectively ignored by DLD loader]
  - RC = record count (1 byte)
  - PN = program name (5 bytes)
  - TC = type code (3 bytes, e.g. "PRG")
  - DI = disk index (1 byte)
  - U1 = unknown1 (1 byte, de-biased: wire value - $10))
  - U2 = unknown2 (1 byte, de-biased: wire value - $10))

##### Per-Disk Files

These files reside on every disk of the set managed by the catalog system. They contain an inventory of the managed files on that disk.

|Name|Purpose|Type|Disk Size (Blocks)|Size (Bytes)|Load Address|Contents|
|---|---|---|---|---|---|---|
|CAT.FILE|Disk's file inventory|PRG|8|1999|DLD module's working buffers|record-oriented data|

- **CAT.FILE**:

  The data in this file is record-oriented with the following structure:

  Format: `[LA][RC]{[SC][FT][BC][FN]}...`
  - LA = load address (2 bytes LE) [Note: effectively ignored by DLD loader]
  - RC = record count (1 byte)
  - SC = share count, # of modules that load this file (1 byte)
  - FT = file type (1 byte ASCII, e.g. 'P' = PRG, 'S' = SEQ, etc.)
  - BC = block count (3 bytes ASCII decimal digits, e.g. "014")
  - FN = file name (variable bytes C64 Screen Codes)

##### Per-Module Files

Each module has its own catalog inventory of files associated with it. Note that this is independent of the module manifest files used by LOADER;9.

|Name|Purpose|Type|Disk Size (Blocks)|Size (Bytes)|Load Address|Contents|
|---|---|---|---|---|---|---|
|CAT.ACC|ACC module catalog|PRG|1|199|DLD module's working buffers|record-oriented data|
|CAT.BBB|BBB module catalog|PRG|1|251|DLD module's working buffers|record-oriented data|
|CAT.DLD|DLD module catalog|PRG|2|265|DLD module's working buffers|record-oriented data|
|CAT.LIB|LIB module catalog|PRG|2|291|DLD module's working buffers|record-oriented data|
|CAT.MMM|MMM module catalog|PRG|2|479|DLD module's working buffers|record-oriented data|
|CAT.STP|STP module catalog|PRG|2|263|DLD module's working buffers|record-oriented data|
|CAT.TLK|TLK module catalog|PRG|2|336|DLD module's working buffers|record-oriented data|

- **CAT.ACC**, **CAT.BBB**, **CAT.DLD**, **CAT.LIB**, **CAT.MMM**, **CAT.STP**, **CAT.TLK**:

  The data in thes files is record-oriented with the following structure:

  Format: `[LA][RC]{[FT][BC][FN]}...`
  - LA = load address (2 bytes LE)
  - RC = record count (1 byte)
  - FT = file type (1 byte ASCII, e.g. 'P' = PRG, 'S' = SEQ, etc.)
  - BC = block count (3 bytes ASCII decimal digits, e.g. "014")
  - FN = file name (variable bytes C64 Screen Codes)

##### Per-Game Files

There are no games files on this disk.

#### 2.2.2.3 Module Manifest Files

These files are read and parsed by the module loader executable (LOADER;9). They contain descriptors that instruct the loader how to map the contents of a module's component files into memory. There is one manifest file per module, and only one is ever loaded during a loader invocation.

|Name|Purpose|Type|Disk Size (Blocks)|Size (Bytes)|Content|
|---|---|---|---|---|---|
|MMM|MMM module manifest|SEQ|2|345|record-oriented data|
|ACC|ACC module manifest|SEQ|1|174|record-oriented data|
|BBB|BBB module manifest|SEQ|1|217|record-oriented data|
|DLD|DLD module manifest|SEQ|1|199|record-oriented data|
|LIB|LIB module manifest|SEQ|1|251|record-oriented data|
|STP|STP module manifest|SEQ|1|229|record-oriented data|
|TLK|TLK module manifest|SEQ|1|239|record-oriented data|

- **MMM**, **ACC**, **BBB**, **DLD**, **LIB**, **STP**, **TLK**:

  Each file has two parts, a module descriptor followed by a set of component file descriptors, one for each file loaded.

  Module descriptor format: This 13-byte sequence is a bit novel in its structure.
  - The first 2 bytes are a `$40|nibble`-encoded value that is the module's estimated load time in seconds. This seeds the loader screen's countdown counter.
  - The next 4 bytes are also a `$40|nibble`-encoded value, as 2 pairs, that provides the 16-bit screen RAM address for the displayed timer. This allows customization of its location around the per-section background image.
  - The next 3 bytes are the ASCII-encoded countdown value initially painted.
  - The next 1 byte is an ASCII space character delimiter.
  - The next 3 bytes are the ASCII-encoded name of the module.
  - This is followed by a carriage return terminator.

  For example, the module descriptor `FOLJ@E151 MMM` decodes as follows:
  - Estimated load time = {F O} -> {$46 $4F} -> $6F, or 111 seconds.
  - Screen RAM address = {L J} {@ E} -> {$4C $4A} {$40 $45}, or $05CA (little endian).
  - The initial painted countdown value = {1 5 1} -> 151 -> "1:51".
  - The module name = "MMM".

  File descriptor format: `[EF][FN]<CR>`
  - EF = encoding flag (1 byte ASCII): `A` = raw, `B` = canonical-Huffman compression
  - FN = file name (variable bytes ASCII)
  - <CR> = carriage return

  The last file descriptor is always `AEND<CR>`, the end-of-list sentinel value.

The contents of each of these files are described in the Module Manifests section below.

#### 2.2.2.4 Module Component Files

These files are used to configure the memory regions of the Finite State Machine (FSM) for a particular module.  They are detailed in the descriptions of their associated modules below.

#### 2.2.2.5 Unknown Files

There are some files whose function is not known because they could not be correlated with any other existing files.

|Name|Purpose|Type|Disk Size (Blocks)|Size (Bytes)|Load Address|Content|
|---|---|---|---|---|---|---|
|DLD.DUMMY.1.A1|Unknown|PRG|4|1000|N/A|random data|
|DLD.DUMMY.2.A1|Unknown|PRG|4|1000|N/A|random data|
|MMM.DUMMY.1.A1|Unknown|PRG|4|1000|N/A|random data|
|MMM.DUMMY.2.A1|Unknown|PRG|4|1000|N/A|random data|
|MMM.DUMMY.3.A1|Unknown|PRG|4|1000|N/A|random data|
|MMM.DUMMY.4.A1|Unknown|PRG|4|1000|N/A|random data|
|TLK.DUMMY.1.A1|Unknown|PRG|4|1000|N/A|random data|
|TLK.DUMMY.2.A1|Unknown|PRG|4|1000|N/A|random data|
|TLK.DUMMY.3.A1|Unknown|PRG|4|1000|N/A|random data|

> **Intuitive Insights**: These 9 files are a mystery. There is no logic on the disk that reads them. They are all composed of identical, fairly high-quality random-like data (entropy = 7.76 bits/byte, index of coincedence = 0.0041, bit balance = 50.21%). We have two competing hypotheses:
>
> Machine intelligence: These are placeholders reserving disk blocks for future expansion of their respective modules via the disk update feature. They are allocated in convenient 1000-byte chunks and contain good random data to resist compression.
>
> Human intelligence: Interesting idea, but 1) why the per-module budget, why not simply "DUMMY.1", "DUMMY.2", etc.?, 2) 4 disk blocks is 1024 bytes not 1000, why the gap?, and 3) why care about resistence to disk compression when (unlike decompression) there's no application feature that applies it? More likely they are more "engineering escape" artifacts like the quirky volume name. Maybe residual data from a test of the disk update feature that accidentally leaked back into their BOM. Possibly further evidence the files are from an alpha/beta/pre-release distribution.
>
> Neither explanation is truly satisfying. Your guess is as good as ours.

### 2.2.3 Disk Catalog and Loader Modules

The PlayNET client uses two overlapping systems to account for the various files associated with it.

The first is a disk catalog feature that uses a set of metadata files (CAT.*) to track the current contents of the product files across the disk set. Management of this catalog is the sole responsibility of the DLD module, which is accessible via the "Update PlayNET Disks" section of the Main Menu. This module is capable of downloading new or updated PlayNET product files to the disk collection.

The second is a set of manifest files (the seven SEQ files) that track which product files are associated with each module. These files are used exclusively by the LOADER;9 module loader, which does not refer to the catalog files to perform its function. However, the manifest files are themselves cataloged and managed by the DLD module, just like the other product files.

The module manifest and component files are described below.

### 2.2.3.1 Common Kinds of Module Component Files (Inferred)

There are certain kinds of files that are common across the modules and share certain traits. They are listed in the following table:

|Kind|Purpose|Type|Disk Size (Blocks)|Uncompressed Size (Bytes)|Load Address|Contents|
|---|---|---|---|---|---|---|
|BAS|Section Logic|PRG|variable|variable|$0801 (via $A001*)|tokenized BASIC|
|CLR|Color RAM Image|PRG|1 or 2 depending on compressibility|1000|$D800|data|
|DO|FSM Segment Handlers|PRG|variable|variable|variable|native code|
|JMP|FSM Subroutine Dispatch Table|PRG|variable|variable|$1394 (via $AB94*)|data|
|LEX|FSM Selector Dispatch Table|PRG|variable|variable|$C800|data|
|PTR|FSM Base Table|PRG|1|28|$F46C|data|
|RINT|Raster Interrupt Table|PRG|1|80|$C7B0|data|
|SCR|Screen RAM Image|PRG|2 or 3 depending on compressibility|1000-1024 (if sprites)|$0400|data|
|SEND|FSM Send Code Table|PRG|variable|variable|$BDD0|data|
|TBL|FSM Arrangement|PRG|variable|variable|$1300 (via $AB00*)|FSM bytecode|
|VIC|VIC-II Register Block|PRG|1|47|$D000|data|

\* = temporary staging location to avoid clobbering the loader

### 2.2.3.2 Module Manifests 

There are seven manifests for the modules associated with the operation of the client runtime.

### 2.2.3.2.1 MMM Module Manifest

The MMM module contains logic and data for the display and functionality of the service dialer, including session establishment and authentication, and the main menu.

<img src="/images/mmm_load.png"> <img src="/images/mmm_screen1.png"> <img src="/images/mmm_account.png"> <img src="/images/mmm_screen3.png">

|Module Descriptor|Load Time (Seconds)|Screen RAM Address|Initial String|Module Name|
|---|---|---|---|---|
|FOLJ@E151 MMM|111|$05CA|"1:51"|"MMM"|

|Order|Component Descriptor|File Name|Kind/Purpose|Disk Size (Blocks)|Uncompressed Size (Bytes)|Load Address|Contents|
|---|---|---|---|---|---|---|---|
|1|BMMM.SCR.E.A1|MMM.SCR.E.A1|SCR|(default)|(default)|(default)|(default)|
|2|BMMM.CLR.E.A1|MMM.CLR.E.A1|CLR|(default)|(default)|(default)|(default)|
|3|AMMM.VIC.A1|MMM.VIC.A1|VIC|(default)|(default)|(default)|(default)|
|4|BMMM.GENERAL.E.A1|MMM.GENERAL.E.A1|resident menu section manager|6|1396|$EE00|native code|
|5|BEDC.MLP.E.A1|EDC.MLP.E.A1|transport layer logic|6|1482|$B800|native code|
|6|BDOINIT.E.A1|DOINIT.E.A1|frame builder logic|1|104|$9A00|native code|
|7|BKERNAL;2.E.A1|KERNAL;2.E.A1|KERNAL vector patch|1|127|$FF81|data|
|8|BXLCOMM;7.1.E.A1|XLCOMM;7.1.E.A1|FSM interface|5|1024|$C400|native code|
|9|BXHCOMM;7.1.E.A1|XHCOMM;7.1.E.A1|FSM handlers|14|3584|$E000|native code|
|10|AMMMRINT;5.A1|MMMRINT;5.A1|RINT|(default)|(default)|(default)|(default)|
|11|BFSM2.MLP.E.A1|FSM2.MLP.E.A1|FSM core|11|2938|$F400|native code|
|12|BMMM.DO.E.A1|MMM.DO.E.A1|DO|1|50|$6000|(default)|
|13|AMMM.SPR|MMM.SPR|PlayNET logo sprites|3|576|$0800 (via $A000*)|data|
|14|BMMM.SEND.E.A1|MMM.SEND.E.A1|SEND|1|125|(default)|(default)|
|15|BMMMSNDRTN.E.A1|MMMSNDRTN.E.A1|send routines|1|239|$BF10|code|
|16|BMMMLEX;7.0.E.A1|MMMLEX;7.0.E.A1|LEX|2|303|(default)|(default)|
|17|ACAT.MODULE|CAT.MODULE|module catalog|1|237|$1000 (via $A800*)|data|
|18|BMMM;3/6.TBL.E.A1|MMM;3/6.TBL.E.A1|TBL|8|2553|$4000|(default)|
|19|BMMM;3/6.PTR.E.A1|MMM;3/6.PTR.E.A1|PTR|(default)|(default)|(default)|(default)|
|20|BMMM.JMP.E.A1|MMM.JMP.E.A1|JMP|1|28|$4094|(default)|
|21|BFDEBUG.MLP.E.A1|FDEBUG.MLP.E.A1|FSM debug overlay|1|80|$9D00|native code|
|22|BMMM.BAS.E.A1|MMM.BAS.E.A1|dialer + section logic|25|7846|$1301 (via $AB01*)|tokenized BASIC|

\* = temporary staging location to avoid clobbering the loader

Notes:
- DOINIT.E.A1, MMM.DO.E.A1, MMMLEX;7.0.E.A1, MMM;3/6.PTR.E.A1, MMM.JMP.E.A1, and FDEBUG.MLP.E.A1 have negative compression but it does not cross a block boundary.

### 2.2.3.2.2 ACC Module Manifest

The ACC module contains logic and data for the main menu's "Account & Billing Information" section.

<img src="/images/acc_load.png"> <img src="/images/acc_screen.png">

|Module Descriptor|Load Time (Seconds)|Screen RAM Address|Initial String|Module Name|
|---|---|---|---|---|
|FBMH@D138 ACC|96|$04D8|"1:38"|"ACC"|

|Order|Component Descriptor|File Name|Kind/Purpose|Disk Size (Blocks)|Uncompressed Size (Bytes)|Load Address|Contents|
|---|---|---|---|---|---|---|---|
|1|BACC.SCR.E.A1|ACC.SCR.E.A1|SCR|(default)|(default)|(default)|(default)|
|2|BACC.CLR.E.A1|ACC.CLR.E.A1|CLR|(default)|(default)|(default)|(default)|
|3|AACC.VIC.A1|ACC.VIC.A1|VIC|(default)|(default)|(default)|(default)|
|4|AGENRINT;1.A1|GENRINT;1.A1|RINT|(default)|(default)|(default)|(default)|
|5|BACC.SEND.E.A1|ACC.SEND.E.A1|SEND|1|261|(default)|(default)|
|6|BACC.DO.E.A1|ACC.DO.E.A1|DO|1|175|$C000|(default)|
|7|BACCLEX;7.2.E.A1|ACCLEX;7.2.E.A1|LEX|2|443|(default)|(default)|
|8|BACC.TBL.E.A1|ACC.TBL.E.A1|TBL|52|19331|(default)|(default)|
|9|BACC.PTR.E.A1|ACC.PTR.E.A1|PTR|(default)|(default)|(default)|(default)|
|10|BACC.JMP.E.A1|ACC.JMP.E.A1|JMP|1|60|(default)|(default)|
|11|BACC.BAS.E.A1|ACC.BAS.E.A1|BAS|3|753|(default)|(default)|

Notes:
- ACC.DO.E.A1, ACC.PTR.E.A1, and ACC.JMP.E.A1 have negative compression but it does not cross a block boundary.

### 2.2.3.2.3 BBB Module Manifest

The BBB module contains logic and data for the main menu's "Bulletin Boards & Special Interest Groups" section.

<img src="/images/bbb_load.png"> <img src="/images/bbb_screen.png">

|Module Descriptor|Load Time (Seconds)|Screen RAM Address|Initial String|Module Name|
|---|---|---|---|---|
|ELFM@F132 BBB|92|$066D|"1:32"|"BBB"|

|Order|Component Descriptor|File Name|Kind/Purpose|Disk Size (Blocks)|Uncompressed Size (Bytes)|Load Address|Contents|
|---|---|---|---|---|---|---|---|
|1|BBBB.SCR.E.A1|BBB.SCR.E.A1|SCR|(default)|(default)|(default)|(default)|
|2|BBBB.CLR.E.A1|BBB.CLR.E.A1|CLR|(default)|(default)|(default)|(default)|
|3|ABBB.VIC.A1|BBB.VIC.A1|VIC|(default)|(default)|(default)|(default)|
|4|BBBSLEX;7.1.E.A1|BBSLEX;7.1.E.A1|LEX|2|395|(default)|(default)|
|5|BBB.SEND.E.A1|BB.SEND.E.A1|SEND|1|234|(default)|(default)|
|6|BBB.DO.E.A1|BB.DO.E.A1|DO|8|1968|$8C00|(default)|
|7|BZBBMLP;10.E.A1|ZBBMLP;10.E.A1|text editor core|6|1423|$8000|native code|
|8|BZEDITL;10.E.A1|ZEDITL;10.E.A1|text editor forms|5|1261|$8700|native code|
|9|BZEDITH;10.E.A1|ZEDITH;10.E.A1|text editor commands|4|1003|$C000|native code|
|10|ABBRINT;2.A1|BBRINT;2.A1|RINT|(default)|(default)|(default)|(default)|
|11|BBBB.TBL.E.A1|BBB.TBL.E.A1|TBL|34|12120|(default)|(default)|
|12|BBBB.PTR.E.A1|BBB.PTR.E.A1|PTR|(default)|(default)|(default)|(default)|
|13|BBB.JMP.E.A1|BB.JMP.E.A1|JMP|2|286|(default)|(default)|
|14|BBB.BAS.E.A1|BB.BAS.E.A1|BAS|3|653|(default)|(default)|

Notes:
- BBSLEX;7.1.E.A1, BBB.PTR.E.A1, BB.JMP.E.A1 have negative compression but it does not cross a block boundary.

### 2.2.3.2.4 DLD Module Manifest

The DLD module contains logic and data for the main menu's "Update PlayNET Disks" section.

<img src="/images/dld_load.png"> <img src="/images/dld_screen1.png"> <img src="/images/dld_screen2.png">

|Module Descriptor|Load Time (Seconds)|Screen RAM Address|Initial String|Module Name|
|---|---|---|---|---|
|EFIB@F126 DLD|86|$0692|"1:26"|"DLD"|

|Order|Component Descriptor|File Name|Kind/Purpose|Disk Size (Blocks)|Uncompressed Size (Bytes)|Load Address|Contents|
|---|---|---|---|---|---|---|---|
|1|BDLD.SCR.E.A1|DLD.SCR.E.A1|SCR|(default)|(default)|(default)|(default)|
|2|BDLD.CLR.E.A1|DLD.CLR.E.A1|CLR|(default)|(default)|(default)|(default)|
|3|ADLD.VIC.A1|DLD.VIC.A1|VIC|(default)|(default)|(default)|(default)|
|4|AGENRINT;1.A1|GENRINT;1.A1|RINT|(default)|(default)|(default)|(default)|
|5|BDLD.SNDTAB.E.A1|DLD.SNDTAB.E.A1|SEND|1|260|(default)|(default)|
|6|BDLDLEX;7.1.E.A1|DLDLEX;7.1.E.A1|LEX|2|383|(default)|(default)|
|7|BDLD.4000.E.A1|DLD.4000.E.A1|download engine|17|4817|$4000|code|
|8|BDLD.ADDRESS.E.A1|DLD.ADDRESS.E.A1|download buffer map|1|60|$5400|data|
|9|BDLD.TBL.E.A1|DLD.TBL.E.A1|TBL|31|10959|(default)|(default)|
|10|BDLD.PTR.E.A1|DLD.PTR.E.A1|PTR|(default)|(default)|(default)|(default)|
|11|BDLD.JMPTBL.E.A1|DLD.JMPTBL.E.A1|JMP|1|200|(default)|(default)|
|12|BDLD.BAS.E.A1|DLD.BAS.E.A1|BAS|5|1212|(default)|(default)|

Notes:
- DLD.ADDRESS.E.A1, DLD.PTR.E.A1 and DLD.JMPTBL.E.A1 have negative compression but it does not cross a block boundary.

### 2.2.3.2.5 LIB Module Manifest

The LIB module contains logic and data for the main menu's "Software Delivery Service" section.

<img src="/images/lib_load.png"> <img src="/images/lib_screen.png">

|Module Descriptor|Load Time (Seconds)|Screen RAM Address|Initial String|Module Name|
|---|---|---|---|---|
|H@BH@D208 LIB|128|$0428|"2:08"|"LIB"|

|Order|Component Descriptor|File Name|Kind/Purpose|Disk Size (Blocks)|Uncompressed Size (Bytes)|Load Address|Contents|
|---|---|---|---|---|---|---|---|
|1|BLIB.SCR.E.A1|LIB.SCR.E.A1|SCR|(default)|(default)|(default)|(default)|
|2|BLIB.CLR.E.A1|LIB.CLR.E.A1|CLR|(default)|(default)|(default)|(default)|
|3|ALIB.VIC.A1|LIB.VIC.A1|VIC|(default)|(default)|(default)|(default)|
|4|BLIBLEX;7.0.E.A1|LIBLEX;7.0.E.A1|LEX|2|431|(default)|(default)|
|5|BLIB.SEND.E.A1|LIB.SEND.E.A1|SEND|2|287|(default)|(default)|
|6|BLIB.DO.E.A1|LIB.DO.E.A1|DO|8|2043|$8C00|(default)|
|7|BLIB.EFT1.E.A1|LIB.EFT1.E.A1|engine-side handlers|3|508|$5E00|native code|
|8|BLIB.EFT2.E.A1|LIB.EFT2.E.A1|disk-side handlers|5|1227|$9800|native code|
|9|BZBBMLP;10.E.A1|ZBBMLP;10.E.A1|text editor core|6|1423|$8000|native code|
|10|BZEDITL;10.E.A1|ZEDITL;10.E.A1|text editor forms|5|1261|$8700|native code|
|11|BZEDITH;10.E.A1|ZEDITH;10.E.A1|test editor commands|4|1003|$C000|native code|
|12|ABBRINT;2.A1|BBRINT;2.A1|RINT|(default)|(default)|(default)|(default)|
|13|BLIB.TBL.E.A1|LIB.TBL.E.A1|TBL|52|18753|(default)|(default)|
|14|BLIB.PTR.E.A1|LIB.PTR.E.A1|PTR|(default)|(default)|(default)|(default)|
|15|BLIB.JMP.E.A1|LIB.JMP.E.A1|JMP|2|344|(default)|(default)|
|16|BLIB.BAS.E.A1|LIB.BAS.E.A1|BAS|6|1738|(default)|(default)|

Notes:
- LIBLEX;7.0.E.A1, and LIB.SEND.E.A1, LIB.PTR.E.A1, and LIB.JMP.E.A1 have negative compression but it does not cross a block boundary. However, negative compression of LIB.EFT1.E.A1 *does* cost a disk block.

### 2.2.3.2.6 STP Module Manifest

The STP module contains logic and data for the main menu's "PlayNET Shopping Center" section.

<img src="/images/stp_load.png"> <img src="/images/stp_screen.png">

|Module Descriptor|Load Time (Seconds)|Screen RAM Address|Initial String|Module Name|
|---|---|---|---|---|
|E@JF@E120 STP|80|$05A6|"1:20"|"STP"|

|Order|Component Descriptor|File Name|Kind/Purpose|Disk Size (Blocks)|Uncompressed Size (Bytes)|Load Address|Contents|
|---|---|---|---|---|---|---|---|
|1|BSTP.SCR.E.A1|STP.SCR.E.A1|SCR|(default)|(default)|(default)|(default)|
|2|BSTP.CLR.E.A1|STP.CLR.E.A1|CLR|(default)|(default)|(default)|(default)|
|3|ASTP.VIC.A1|STP.VIC.A1|VIC|(default)|(default)|(default)|(default)|
|4|BSTPLEX;7.0.E.A1|STPLEX;7.0.E.A1|LEX|3|495|(default)|(default)|
|5|BSTP.SEND.E.A1|STP.SEND.E.A1|SEND|1|253|(default)|(default)|
|6|BSTP.LO.MLP.E.A1|STP.LO.MLP.E.A1|DO|7|1936|$8C00|(default)|
|7|BZBBMLP;10.E.A1|ZBBMLP;10.E.A1|text editor core|6|1423|$8000|native code|
|8|BZEDITL;10.E.A1|ZEDITL;10.E.A1|text editor forms|5|1261|$8700|native code|
|9|BZEDITH;10.E.A1|ZEDITH;10.E.A1|text editor commands|4|1003|$C000|native code|
|10|ABBRINT;2.A1|BBRINT;2.A1|RINT|(default)|(default)|(default)|(default)|
|11|BSTP.TBL.E.A1|STP.TBL.E.A1|TBL|26|9252|(default)|(default)|
|12|BSTP.PTR.E.A1|STP.PTR.E.A1|PTR|(default)|(default)|(default)|(default)|
|13|BSTP.JMP.MLP.E.A1|STP.JMP.MLP.E.A1|JMP|1|292|(default)|(default)|
|14|BSTP.BAS.E.A1|STP.BAS.E.A1|BAS|3|654|(default)|(default)|

Notes:
- STPLEX;7.0.E.A1 and STP.PTR.E.A1 have negative compression but it does not cross a block boundary.

### 2.2.3.2.7 TLK Module Manifest

The TLK module contains logic and data for the main menu's "People, Games & More!" section.

<img src="/images/tlk_load.png"> <img src="/images/tlk_screen.png">

|Module Descriptor|Load Time (Seconds)|Screen RAM Address|Initial String|Module Name|
|---|---|---|---|---|
|GHHB@G200 TLK|120|$0782|"2:00"|"TLK"|

|Order|Component Descriptor|File Name|Kind/Purpose|Disk Size (Blocks)|Uncompressed Size (Bytes)|Load Address|Contents|
|---|---|---|---|---|---|---|---|
|1|BTLK.SCR.E.A1|TLK.SCR.E.A1|SCR|(default)|(default)|(default)|(default)|
|2|BTLK.CLR.E.A1|TLK.CLR.E.A1|CLR|(default)|(default)|(default)|(default)|
|3|ATLK.VIC.A1|TLK.VIC.A1|VIC|(default)|(default)|(default)|(default)|
|4|AGTRINT;4.A1|GTRINT;4.A1|RINT|(default)|(default)|(default)|(default)|
|5|ACAT.GAME|CAT.GAME|game catalog|2|288|$8000|data|
|6|ACAT.MODULE|CAT.MODULE|module catalog|1|237|$1000 (via $A800*)|data|
|7|BTLK.DO.E.A1|TLK.DO.E.A1|DO|5|1317|$6900|(default)|
|8|BTLK.DIS.E.A1|TLK.DIS.E.A1|room display handlers|4|900|$7600|native code|
|9|BTLK.GS.E.A1|TLK.GS.E.A1|game selection handlers|3|603|$7100|native code|
|10|BTLK.SEND.E.A1|TLK.SEND.E.A1|SEND|2|312|(default)|(default)|
|11|BXTLLEX;7.0.E.A1|XTLLEX;7.0.E.A1|LEX|1|207|$C180)|(default)|
|12|BXTHLEX;7.0.E.A1|XTHLEX;7.0.E.A1|LEX|2|480|(default)|(default)|
|13|BTLK.TBL.E.A1|TLK.TBL.E.A1|TBL|58|20975|(default)|(default)|
|14|BTLK.PTR.E.A1|TLK.PTR.E.A1|PTR|(default)|(default)|(default)|(default)|
|15|BTLK.JMP.E.A1|TLK.JMP.E.A1|JMP|1|180|(default)|(default)|
|16|BTALK.V5.E.A1|TALK.V5.E.A1|BAS|3|763|(default)|(default)|

\* = temporary staging location to avoid clobbering the loader

Notes:
- TLK.GS.E.A1, XTHLEX;7.0.E.A1, and TLK.PTR.E.A1 have negative compression but it does not cross a block boundary.

---

Now that we've seen what the PlayNET client looks like at rest, in the [next chapter](/docs/chapter3.md) we'll examine it as it comes to life.

