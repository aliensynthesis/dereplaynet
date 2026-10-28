
<img src="/images/Chapter3.png" width="680">

# *~De Re PlayNET*<br>Chapter 3: Bootstrapping and the Dialer

In the previous chapter we explored the internals of the application while it is at rest. Next, we'll examine how its state progresses from the time it is invoked until the moment a connection is first established with the online PlayNET service.

## Chapter Contents

* [3.1 Boot Sequence](#31-boot-sequence)
* [3.2 The Dialer](#32-the-dialer)

<br>

## 3.1 Boot Sequence

The "cold start" workflow of the PlayNET client application is described below.

1. The user inserts the SYSTEM disk into the drive and at the BASIC prompt enters `LOAD"*",8,1`.

   <img src="/images/basic_load.png">

2. The **PLAYNET** program is automatically loaded because it is the first entry in the DOS directory. It is then started by leveraging a widely used Commodore 64 trick that saves the user the trouble of entering a subsequent `RUN` command.
    - The keyboard buffer is preloaded with the character sequence `{RED}{CLR}SYS679{CR}`, which is automatically processed by the keyboard handler when the LOAD command returns to the READY prompt. In addition to the text and cursor color being set to red, the background color is also set to red, which is a technique to make the BASIC/KERNAL boot chatter hidden.

      <img src="/images/basic_red.png">

    - The `SYS679` command transfers control to an embedded code stub at $02A7, which again preloads the keyboard buffer, this time with the character sequence `RUN{CR}`, then sets up a KERNAL LOAD of the chained executable named LOADER.

3. The **LOADER** program is started and performs the following steps:
    - The KERNAL STOP vector (ISTOP) is patched to disable the RUN/STOP key.

      > **Intuitive Insights**: It's not clear why this locking operation is done here. RUN/STOP is not restored by this program or the chain-loaded LOADER;9. Our best guess is that it is to ensure completion of the memory bank-switching operations in LOADER;9, but why it's done here and not there is not understood.

    - The screen is cleared, and then the MMM.SPR file, containing the PlayNET logo sprite data, is loaded.
    - The border color is set, and the splash screen is drawn.

      <img src="/images/loader_screen.png">

    - An animation loop is entered, during which the program checks for a keypress on each cycle.
    - Upon detecting a keypress, or after approximately 11 seconds, the program launches the main menu module by storing "MMM" in the loader's module name buffer at $0335 and invoking `LOAD"LOADER;9",8,1`.

4. The **LOADER;9** program is started and continues:
    - Its BASIC stub calls `SYS2061` to initialize and start the Forth VM.
    - The Forth program changes the memory configuration to expose RAM beneath the I/O region, stores a verbatim copy of the VM at $D000, and then restores the previous memory configuration. The banked VM will be made accessible again later when needed for post-boot operations that require module decompression.
    - The screen color is set to red again to mask the screen RAM load operation below.

      <img src="/images/basic_red.png">

    - The disk channel is opened, and the program parses the MMM module's manifest file.
    - For each component file descriptor, the specified file's contents are read, decompressed if necessary, and placed in its target memory location.

      By design, the first component files of every manifest are the SCR, CLR and VIC data, which define the loading screen image of the module. It is therefore loaded immediately into screen RAM when the module starts to load, resulting in the image being displayed quickly. LOADER;9 logic then handles the countdown clock updates.

      <img src="/images/mmm_load.png">

    - The disk channel is closed, and the screen is cleared.
    - Control is passed to the BASIC code imported from the MMM.BAS.E.A1 component file.

MMM.BAS.E.A1 contains the largest collection of tokenized BASIC code of any file on the SYSTEM disk, comprising roughly 1,600 lines. Internally, the code can be divided into several functional areas. The most immediately relevant is the dialer feature, which accounts for approximately 500 lines of code. That is the next stop in the startup sequence.

## 3.2 The Dialer

The first phase of MMM.BAS.E.A1 execution is the dialer logic. It proceeds as follows:

1. Once again, the KERNAL STOP vector (ISTOP) is patched to disable the RUN/STOP key.

2. The C64 software UART on device 2 is opened for 300 baud communication.

3. The Hayes modem commands "+++" and "AT" are sent to device 2, with a small delay in between, in order to "wake" the modem.

4. The modem selection screen is drawn and the user is prompted to select the modem type.

   <img src="/images/dialer_modem.png">

This selection determines the values of a set of port and modem control variables, as follows:

|variable|description|
|---|---|
|DD|CIA #2 Port B DDR ($DD03); $02=PB1 outp(RTS), $04=PB2 outp(DTR), $20=PB5 outp|
|DL|line transition delay loop|
|DO|CIA #2 Port B ($DD01), released state|
|DP|CIA #2 Port B ($DD01), asserted state|
|CIA line-init|uses line init sequence|
|autodial|uses phone # prompt + ATD|

|TY|Modem|DD|DL|DO|DP|CIA line-init|autodial|
|---|---|---|---|---|---|---|---|
|1|VICMODEM, HES 1, etc. (default)|$06 (unused)|0|$FF (unused)|$FF (unused)|N|N|
|2|Commodore 1650, MasterModem|$06 (unused)|0|$FF (unused)|$FF (unused)|N|N|
|3|Commodore 1660 Modem 300|$26|5000|$DF|$DF|Y|N|
|4|Mighty Mo, HES 2A|$26|4000|$02|$04|Y|N|
|5|Westridge Auto-Modem, HES 2B|$06 (unused)|4000|$FF (unused)|$FF (unused)|N|N|
|6|Tele-Learning Modem|$06 (unused)|4000|$FF (unused)|$FF (unused)|N|N|
|7|Hayes-compatible RS-232 modem|$26|500|$80|$86|Y|Y|

5. If the autodial variable of the modem selected is Y then the dialer prompts the user for the number to dial.

   <img src="/images/dialer_auto.png">

6. For modems with special CIA line-init handling indicated, the CIA ports are exercised in the following sequence:
   - CIA #2 Port B DDR is asserted with the modem's DD value.
   - CIA #2 Port B is asserted with the modem's DP value.
   - If modem's DO is not equal to DP then CIA #2 Port B is asserted with the modem's DO value.
   - Handler spins until CIA #2 Port B bit 4 (DCD) is set (sort of...).
      > **Intuitive Insights**: The BASIC code of step #4 is dubious. It detokenizes to `86 IF PEEK(56577) AND 16 = 0 THEN 86`. But due to CBM BASIC operator precedence, `16 = 0` is evaluated before `AND`, so `IF` is always false. This smells like code leftover from prototyping or debugging and then forgotten about because it never worked correctly to begin with!

7. The dialer prompts the user for the network type.

   <img src="/images/dialer_network.png">

8. If the autodial variable of the modem selected is N then the dialer will display the following screen and prompt for a RETURN key:

   <img src="/images/dialer_manual.png">

   At this point the user is expected to manually place the call using a telephone and then press RETURN when the service's answering modem is heard.

9. The dialer will display the attempting to connect screen.

   <img src="/images/dialer_attempting.png">

   An F5 key input will abort the process.  Otherwise the behavior depends on the network type selected. If the user selected the direct dial option then the dialer waits for the modem to connect directly with the PlayNET service.

   If the network type was Telenet:
   - Once the modem is connected to Telenet, the dialer will loop sending periodic carriage-return characters looking for a "TERMINAL=" response from the Telenet PAD.
   - If it does not receive the response it will continue to loop until the user presses F5.
   - If it does receive the response then the dialer will attempt a connection to the service via the PAD command interface:
     - It will respond to the "TERMINAL=" prompt with `D1`, indicating a generic data terminal type, and wait for the "@" PAD prompt.
     - It then sets the connection's X.3 parameters with the PAD command `SET? 10:0,15:0,0:33,57:1,63:0` and again waits for the PAD prompt.
     - It sends the `CONNECT` command to host `51852E` and then expects the response "518 52E CONNECTED". If received then the link to the service has been established.
       > **Intuitive Insights**: "51852E" was an older-style Telenet NUA (Network User Address) even by 1980s standards, attributing genuine legacy street cred to the PlayNET service. 518 represented the telephone area code for the Albany/Troy region of New York, where the PlayNET servers resided.
       >
       > Telenet is a fascinating retro-topic in itself. The advanced hobbyist can find a full-featured Telenet PAD emulator and *Telnet* bridge at https://github.com/aliensynthesis/padawan-lite. It is very useful for projects of this kind. 

If there is a problem completing the connection to the service, then the following screen is shown and the user is given a chance to return to the modem selection screen.

<img src="/images/dialer_failed.png">

Otherwise, these are the final moments of the dialer's lifetime.
- The connection is allowed to settle for 0.75s before proceeding.
- The RUN/STOP key is re-enabled.
  > **Intuitive Insights**: This is the first point where RUN/STOP is re-enabled since it was originally disabled by the LOADER program. We guess that it was kept disabled throughout the dialer's BASIC code execution to prevent the situation where an accidental stop may have left the modem off-hook without the user being aware of it. That could potentially rack up expensive phone charges.
- To prepare for the next stage, the following beautiful screen is drawn using PRINT block art:

  <img src="/images/dialer_waiting.png">

- Finally, the Finite State Machine (FSM) is started up and prepared to take over the server communication role. One of its first tasks is to write the text "Waiting for telecommuncications connection with PlayNET" to the screen.

  Chapter 5 is dedicated to discussing the FSM in detail.

---

The client is now ready for session establishment, which we turn our attention to in the [next chapter](/docs/chapter4.md).

