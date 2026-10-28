
<img src="/images/Chapter4.png" width="680">

# *~De Re PlayNET*<br>Chapter 4: Session Establishment and the Client/Server Protocol

Once the client has successfully connected to a PlayNET server, its dialer code passes control to the inner application engine, known as the FSM, which takes over communication. All client knowledge of the communication protocol used to interact with the server and deliver the features of the PlayNET service is contained within the FSM. That interaction begins with the establishment of a session between the FSM and the server, allowing them to sustain a reliable exchange of messages for the duration of the user's call.

This chapter details the design of the protocol and explains how it is used on the wire to establish and maintain those sessions.

## Chapter Contents

* [4.1 A PlayNET Protocol Primer](#41-a-playnet-protocol-primer)
  * [4.1.1 Saying Hello](#411-saying-hello)
  * [4.1.2 Authenticating the User](#412-authenticating-the-user)
  * [4.1.3 Main Menu and Section Module Transitions](#413-main-menu-and-section-module-transitions)
  * [4.1.4 Cross-Section Module Transitions](#414-cross-section-module-transitions)
* [4.2 PlayNET Protocol Reference](#42-playnet-protocol-reference)
  * [4.2.1 Framing Layer](#421-framing-layer)
  * [4.2.2 Transport Layer](#422-transport-layer)
  * [4.2.3 Session Layer](#423-session-layer)
  * [4.2.4 Session Message Dictionary](#424-session-message-dictionary)
    * [4.2.4.1 MMM Module Messages](#4241-mmm-module-messages)
    * [4.2.4.2 ACC Module Messages](#4242-acc-module-messages)
    * [4.2.4.3 BBB Module Messages](#4243-bbb-module-messages)
    * [4.2.4.4 DLD Module Messages](#4244-dld-module-messages)
    * [4.2.4.5 LIB Module Messages](#4245-lib-module-messages)
    * [4.2.4.6 STP Module Messages](#4246-stp-module-messages)
    * [4.2.4.7 TLK Module Messages](#4247-tlk-module-messages)
    * [4.2.4.8 Message Master List](#4248-message-master-list)

<br>

## 4.1 A PlayNET Protocol Primer

Although PlayNET's protocol is relatively simple and straightforward by modern standards, in its day it was quite sophisticated compared to its 8-bit home computing telecommunications cousins, such as Kermit and CompuServe B. Sophisticated enough, in fact, that it might appear to be an unapproachable subject to readers unfamiliar with such things.

In this section, we provide a bit of tutorial runway leading up to the reference material, helping those readers appreciate some of the underlying communication challenges involved and the solutions the PlayNET engineers developed to address them.

---

### 4.1.1 Saying Hello

Once the client's modem has established a connection to the service, as explained in the previous chapter, it is ready to interact with the PlayNET servers.

That conversation begins with a message the client sends to the server. If you were able to read the bytes going out over the telephone line from a freshly booted client, and were not afraid of hexadecimal numbers, that first message would always look exactly like this:

|First|...|...|...|...|...|...|...|...|Last|
|---|---|---|---|---|---|---|---|---|---|
|11|44|F1|4F|7F|7F|23|04|00|0D|

This, in effect, is the client's "Hello" message. It may strike you as odd that the greeting requires 10 bytes of data when a greeting from me to you in an English pub would require only the five letters "H E L L O". There are a few reasons for this, and, in fact, looks are a little deceiving in this case.

To begin with, communication among machines requires special measures to preserve information integrity and ensure greater resilience than is typically needed in conversations among people. Even in a noisy pub, you and I would have the benefit of multiple modes of communication, such as facial expressions and body language. But in the early- to mid-1980s, the communication options available to a Commodore 64 attempting to reach an online service on the other side of the country, or the world, were much more limited. For the vast majority of users, that amounted to a 300-bit-per-second serial connection that depended on the noisy and sometimes unreliable public telephone network.

So, for the machine conversation to maintain integrity, there must be a way for each side to detect errors in the words being exchanged. The PlayNET protocol accounts for this with a mechanism that allows one side of the conversation to recognize when any bytes from the other side have become corrupted along the way. But of course, nothing comes for free, and the price paid for that benefit is a few bytes of each message dedicated to that purpose.

We can identify those special bytes by redrawing the first diagram in a way that distinguishes the bytes used for ensuring integrity from those carrying the actual content of the conversation:

|Role|First|...|...|...|...|...|...|...|...|Last|
|---|---|---|---|---|---|---|---|---|---|---|
|Content|||||7F|7F|23|04|00||
|Integrity|11|44|F1|4F||||||0D|

In the new Integrity role, we've dedicated several initial bytes to helping identify errors in the remainder of the message, along with one trailing byte to unambiguously mark its end. The remainder is considered message Content.

Technically, the 4 bytes are a specially encoded form of a 16-bit error-detection code known as a CRC-16 checksum. The single 0D byte is a carriage-return character that marks the end of the message's stream of bytes. More will be said about these protocol elements in the protocol reference section later.

And there you have it. Where we would say "H E L L O", the machines could say "7F 7F 23 04 00". The same number of characters, so we're now even, right?

Well, not quite, because we're not yet accounting for the resilience requirement. What if the error-detection mechanism actually found a problem with that message? How would the receiver signal to the sender that something was wrong?

The receiver could put the conversation on hold as soon as a confusing word was heard, immediately replying to the sender with the machine equivalent of "H U H ?". But then what would that do to the pace of the conversation? Either the sender would have to wait a period of time after each message was sent to see if a HUH? response followed, or there would have to be a protocol requiring the receiver to respond to each message with either a HUH? or an OK. The first choice would introduce built-in lag. The second would make the conversation synchronized, effectively making it as slow as the slowest participant at any given moment, while requiring more words in total for the same information exchange. Either way, the conversation would become much less efficient.

By way of comparison, imagine you and I are back in the pub, and I'm trying to tell you a brilliantly clever joke. But the room is loud, and we're sitting across the table from each other. I could tell it a couple of words at a time and then wait for you to acknowledge that you heard them. But since brevity is the soul of wit, that would make it nearly impossible to tell a good pub joke. Alternatively, I could wait for you to say "uh-huh" after every couple of words, but that would surely annoy everyone else at the table.

Maybe a better protocol would be one we're already naturally familiar with. I would continue telling the joke until, at some point, I heard you say something like "I got it up to the bit about your proctologist", at which point I would know exactly which parts to repeat.

What would that look like in terms of our machine protocol? For each participant, we could assign a number that increments each time they send a message and include that number with every message they send. Then we could also include the number of the most recent message from the other participant that was successfully received (along with all preceding messages). Each participant could then check this second number upon receipt of a new message to determine whether any of their previous messages needed to be repeated.

That resilience mechanism might then look like this in the message format:

|Role|First|...|...|...|...|...|...|...|...|Last|
|---|---|---|---|---|---|---|---|---|---|---|
|Content|||||||23|04|00||
|Resiliency|||||7F|7F|||||
|Integrity|11|44|F1|4F||||||0D|

Now, in the new Resiliency role, we've dedicated two numbers to track the sender's incrementing message number and the number of the most recent message successfully received from its peer.

The message content saying "hello" is now down to three bytes. Could we do even better?

Of course we could, and at this point we'll stop torturing you with analogies. (At least we don't torture you with our pub jokes.)

Let's transform that last diagram once again to more accurately reflect the protocol's design by representing the data as a more conventional protocol data unit (PDU):

|PDU Layer|First|...|...|...|...|...|...|...|...|Last|OSI Layer Equivalent|
|---|---|---|---|---|---|---|---|---|---|---|---|
|Session|||||||||||Application|
|Transport (Upper)|||||||23|04|00||Transport|
|Transport (Lower)|||||7F|7F|||||Transport|
|Framing|11|44|F1|4F||||||0D|Data Link|

This four-layer model represents the structure of the PDUs exchanged between clients and the server. The layers are mapped to their approximate OSI equivalents.

Viewed another way:

|PDU Byte|Protocol Layer|Purpose|
|---|---|---|
|**11 44 F1 4F** 7F 7F 23 04 00 0D|Framing|CRC-16/ARC, packed-nibble encoded|
|11 44 F1 4F **7F** 7F 23 04 00 0D|Transport|Message sequence number|
|11 44 F1 4F 7F **7F** 23 04 00 0D|Transport|Peer message acknowledgement number|
|11 44 F1 4F 7F 7F **23** 04 00 0D|Transport|Command; $23 = Open|
|11 44 F1 4F 7F 7F 23 **04 00** 0D|Transport|Transport parameters|
|11 44 F1 4F 7F 7F 23 04 00 0D|Session|(None)|
|11 44 F1 4F 7F 7F 23 04 00 **0D**|Framing|Terminator|

In this case, the message contains command $23, which we call the "Open" message, as in opening a new session. It has two transport-level parameters, $04 and $00. We're not sure what they represent, only that they appear exclusively in Open messages and are hard-coded in the client, so their values never vary.

The message sequence number and acknowledgment number are both $7F, their hard-coded initial value.

The CRC is computed over the transport- and framing-layer bytes. Since those bytes are always the same for the client's initial Open message, the resulting CRC is always the same as well.

This is why the client's first message to the server is always the same sequence of bytes.

For its part, the server then follows up with its own greeting:

#### Wire View
|First|...|...|...|...|...|...|Last|
|---|---|---|---|---|---|---|---|
|F1|43|11|41|7F|7F|24|0D|

That is,

|PDU Byte|Protocol Layer|Purpose|
|---|---|---|
|**F1 43 11 41** 7F 7F 24 0D|Framing|CRC-16/ARC, packed-nibble encoded|
|F1 43 11 41 **7F** 7F 24 0D|Transport|Message sequence number|
|F1 43 11 41 7F **7F** 24 0D|Transport|Peer message acknowledgement number|
|F1 43 11 41 7F 7F **24** 0D|Transport|Command; $24 = Ack|
|F1 43 11 41 7F 7F 24 0D|Session|(None)|
|F1 43 11 41 7F 7F 24 **0D**|Framing|Terminator|

But it is a lukewarm response. Command $24 is a general-purpose acknowledgment that will become fairly routine as the session proceeds. It's the equivalent of the server responding to the client's "Hello there!" with a "Right."

At this point in the PlayNET client's workflow, we have actually accomplished a lot. The client has successfully navigated the telephone network to establish a reliable connection to the PlayNET service and is now on speaking terms with an actual server. A working session has been established.

With that out of the way, the client's next step is to usher its user, Jenny, toward the service's content. But before she can be taken to the fun stuff, there is some "paperwork" we have to take care of.

---

### 4.1.2 Authenticating the User

Each PlayNET customer had a prearranged account number and password before their first sign-on to the service.

After the server responds to the Open message sent earlier, the client begins the authentication workflow by displaying this screen:

<img src="/images/mmm_account.png">

The user is then expected to enter their assigned account number. The client enforces formatting rules for the account number that are not evident on this screen. It must consist of five consecutive digits, followed by an asterisk ("*") [technically, a few other choices are also allowed], followed by five more consecutive digits, for a total of 11 characters (`NNNNN*NNNNN`). That also happens to be the the length limit of the input field.

Jenny carefully enters her account number, like so:

<img src="/images/mmm_account2.png">

The client independently validates the account number's format before forwarding it to the server. If the account number entered is invalid, the user will see this message:

<img src="/images/mmm_account3.png">

Note that this is the first easily observable evidence of the existence of the finite state machine (FSM) engine underlying the application. While the well-formedness checks are performed by native code, the prompt text, the input field overlay, and the field’s characteristics are orchestrated by the FSM. This offers a hint of the strategic purpose behind the FSM technology, which will be described later.

Next, Jenny is prompted to enter the password for her account:

<img src="/images/mmm_password.png">

Unlike the account number, the password is not subject to any client-side well-formedness checks, but it is limited to 10 characters, as you can see here:

<img src="/images/mmm_password2.png">

The characters are echoed in cleartext instead of being masked or hidden. (A relic of a simpler time.)

When RETURN is pressed, the client displays the following message:

<img src="/images/mmm_validation.png">

Then we are introduced to a new kind of protocol message that we call "Data". Now that the transport layer has been established, Data PDUs will take over as the most commonly used message type. Here's what they look like:

#### Wire View
|1st||||||||||||||||||||||||||||||
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|B1|45|C1|40|10|7F|20|44|44|30|30|38|36|37|35|33|30|39|30|69|68|61|74|65|73|63|68|6F|6F|0D|

#### PDU View
|PDU Layer|CRC1|CRC2|CRC3|CRC4|Seq|Ack|Cmd||||||||||||||||||||||||
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|Session||||||||44|44|30|30|38|36|37|35|33|30|39|30|69|68|61|74|65|73|63|68|6F|6F||
|Transport|||||10|7F|20<br>Data||||||||||||||||||||||||
|Framing|B1|45|C1|40||||||||||||||||||||||||||0D|

Data messages carry data in the Session layer, something we have not encountered until now. From this point forward, they will be very common.

The session payload of every Data message begins with a two-letter message ID. In this case, it is "DD". The message ID determines the semantics of the message, including how its contents should be parsed.

The purpose of the DD message is to request that the server validate a user's credentials. The payload contains two concatenated fields. The first is a fixed-length, 10-character account number, and the second is a variable-length password.

#### Session Payload View
|Purpose|1st||||||||||||||||||||||
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|Message ID|44<br>"D"|44<br>"D"|||||||||||||||||||||
|Account number|||30<br>"0"|30<br>"0"|38<br>"8"|36<br>"6"|37<br>"7"|35<br>"5"|33<br>"3"|30<br>"0"|39<br>"9"|30<br>"0"||||||||||||
|Password|||||||||||||69<br>"i"|68<br>"h"|61<br>"a"|74<br>"t"|65<br>"e"|73<br>"s"|63<br>"c"|68<br>"h"|6F<br>"o"|6F<br>"o"|

If the server is unable to validate the credentials, it responds with its own "DK" Data message, like this:

#### PDU View
|PDU Layer|CRC1|CRC2|CRC3|CRC4|Seq|Ack|Cmd||||
|---|---|---|---|---|---|---|---|---|---|---|
|Session||||||||44<br>"D"|4B<br>"K"||
|Transport|||||10|10|20<br>Data||||
|Framing|F1|4E|B1|47||||||0D|

That signals to the client that validation has failed. The client then displays the following screen, giving the user a chance to re-enter both values:

<img src="/images/mmm_invalid.png">

Otherwise, the server returns a "DO" Data message:

#### PDU View
|PDU Layer|CRC1|CRC2|CRC3|CRC4|Seq|Ack|Cmd||||
|---|---|---|---|---|---|---|---|---|---|---|
|Session||||||||44<br>"D"|4F<br>"O"||
|Transport|||||10|10|20<br>Data||||
|Framing|31|4D|B1|46||||||0D|

That tells the client that the user is authorized to access the PlayNET service's main menu, the springboard for all of the service's features.

---

### 4.1.3 Main Menu and Section Module Transitions

Before we conclude the primer and proceed to the protocol reference, it will be worthwhile to explain how a menu section module is invoked at the protocol level. After all, within the PlayNET client, the loadable modules encapsulate the various features, user interfaces, and user experiences of the service, and at the core of the system lies what is essentially a purpose-built, *protocol-directed*, bytecode-driven module transition and execution engine.

Our user Jenny should now see a beautifully rendered screen that looks much like this:

<img src="/images/mmm_screen3.png">

Static images don't really do it justice, as the colors of the letters of the "PlayNET" graphic in the center and the cursor box shift several times per second for a nice effect.

The rather cryptic advice at the bottom, "Use keys surrounding 'S' to make selection", is telling the user that there is a 1:1 relationship between the keyboard keys around the 'S' key and the menu section areas around the center PlayNET box. So, for example, pressing the 'C' key moves the cursor box to the "Account & Billing Information" section, like so:

<img src="/images/mmm_screen4.png">

It's noteworthy that moving the cursor around involves no new messages from the client. That work is handled locally by the FSM. But pressing RETURN on the new menu section does result in a flurry of new activity on the wire:

First, the client sends this message to the server:

#### PDU View
|PDU Layer|CRC1|CRC2|CRC3|CRC4|Seq|Ack|Cmd||||
|---|---|---|---|---|---|---|---|---|---|---|
|Session||||||||4D<br>"M"|41<br>"A"||
|Transport|||||11|10|20<br>Data||||
|Framing|61|49|01|4C||||||0D|

The server responds with:

#### PDU View
|PDU Layer|CRC1|CRC2|CRC3|CRC4|Seq|Ack|Cmd||||
|---|---|---|---|---|---|---|---|---|---|---|
|Session||||||||4D<br>"M"|43<br>"C"||
|Transport|||||11|11|20<br>Data||||
|Framing|51|44|81|4C||||||0D|

The client then sends:

#### PDU View
|PDU Layer|CRC1|CRC2|CRC3|CRC4|Seq|Ack|Cmd||||
|---|---|---|---|---|---|---|---|---|---|---|
|Session||||||||53<br>"S"|53<br>"S"||
|Transport|||||12|11|20<br>Data||||
|Framing|31|48|C1|40||||||0D|

Finally, the server replies with:

#### PDU View
|PDU Layer|CRC1|CRC2|CRC3|CRC4|Seq|Ack|Cmd||||
|---|---|---|---|---|---|---|---|---|---|---|
|Session||||||||53<br>"S"|53<br>"S"||
|Transport|||||12|12|20<br>Data||||
|Framing|71|4C|C1|40||||||0D|

Then the screen background and border turn red, a sign that a module transition has begun.

<img src="/images/basic_red.png">

A few seconds later, the ACC module's load screen is displayed:

<img src="/images/acc_load.png">

The screen will be static except for a countdown field that displays "1:16" on this screen. That number is PlayNET's equivalent of a progress bar as it counts down to 0:00. It is intended to be close to real clock time, but it's not really. Much like a modern progress bar, it may frequently leaving you holding your breath.

At the end of the countdown, the screen changes to the main menu of the ACC module:

<img src="/images/acc_screen.png">

Here you see another menu, but rendered as text instead of graphics. This is the normal style of menu presentation when you're in the menu section modules. Your "cursor" is the red line in the middle. You then scroll using the arrow keys, which shifts the entire menu up or down appropriately.

Following the new display there's a new message emitted from the client, after which the wire goes quiet again:

#### PDU View
|PDU Layer|CRC1|CRC2|CRC3|CRC4|Seq|Ack|Cmd||||
|---|---|---|---|---|---|---|---|---|---|---|
|Session||||||||53<br>"S"|47<br>"G"||
|Transport|||||13|12|20<br>Data||||
|Framing|B1|43|F1|4D||||||0D|

The unpleasant fact is that this signals to the server that usage metering should resume for your account. The good news is that the preceding "SS" message told the server to suspend metering in anticipation of the impending (and sometimes excruciatingly slow) module load. So when you saw the countdown clock begin, you could go make yourself a sandwich, confident that you weren't paying anything extra for it.

To recap, here is a summary of the recent interactions:

#### Wire Interactions
|Dir|Message ID|Meaning (Inferred)|
|---|---|---|
|C→S|MA|Request to transition specifically to the ACC module|
|S→C|MC|Granting permission for that module transition|
|C→S|SS|Request to suspend usage metering|
|S→C|SS|Acknowledging suspend of usage metering|
|C→S|SG|Request to resume usage metering|

> **Intuitive Insights**: The precise semantics of the two-letter Data messages are inferred and therefore highly speculative. This is probably the least well-grounded aspect of the project. All we know for certain is the behavior we can observe, which is then spiced up a bit with compelling interpretations of the message ID letters themselves.
>
>For instance, we know from observation that the "MA" message is routinely sent from the client to the server to begin the transition from the MMM (main menu) module to the ACC (accounting and billing) module. So it's tempting to presume that MA is a mnemonic for MMM-to-ACC. Likewise, perhaps the C in MC means Confirmation. Or it could have no particular meaning at all and was simply the next available letter in the M-family during the design process.
>
>SS is another tricky one. C→S SS is always observed to be paired with S→C SS during normal operation, and they are typically followed by a C→S SG at some point. Since the client's SS is seen to precede long-running client operations outside the control of the user, and a subsequent SG is seen when that operation is done, it's reasonable to presume that it is related to usage metering and an honorable attempt by the service to avoid unfair charges. So maybe SS means Service Suspend. But what about SG? We can't know for sure without access to the authentic service design.
>
>Just be aware that it would be unwise to place too much confidence in the inferred names or semantics of these messages beyond the state transitions they are observed to trigger.

---

### 4.1.4 Cross-Section Module Transitions

Once a user has transitioned from the main menu to a menu section's module, there is no way to return to the top-level menu screen again without restarting the client. The main menu is designed to be a one-time launchpad into the other menu section modules.

This may seem surprising and a bit troubling at first until you realize that each main menu section has its own top-level menu with a choice labeled "Return to Main Menu selections". It won't return you to the main menu screen (it's module is now a skeleton of its former self), but it will present you with a menu that effectively gives you the same module navigation options.

For example, let's say that after Jenny has spent some time looking at her account information (maybe she was concerned that she had already spent too much time online this month!), she decides she wants to go to the main chat reception room to see if any of her friends are there.

She performs this by scrolling down to the bottom option:

<img src="/images/acc_screen2.png">

Then she presses F1, the common "select" key.

At that point, she is presented with a sub-menu of different options for travel:

<img src="/images/acc_screen3.png">

These are the different menu section modules that she has access to from within the current module. She wants to go to the one named "People, Games and More" to enter the main chat reception room, so she uses the arrow keys to scroll up one position to it:

<img src="/images/acc_screen4.png">

And presses F1 again.

The next screen appears:

<img src="/images/acc_screen5.png">

And at that point, the wire goes active with a new message:

#### PDU View
|PDU Layer|CRC1|CRC2|CRC3|CRC4|Seq|Ack|Cmd|||||||||
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|Session||||||||4D<br>"M"|58<br>"X"|54<br>"T"|53<br>"S"|30<br>"0"|30<br>"0"|34<br>"4"|
|Transport|||||1B|1A|20<br>Data|||||||||
|Framing|41|44|61|44|||||||||||0D|

There are a couple of things to point out here.

First, the menu navigation steps she was performing did not involve any communication with the server. That's unlike how many other services of the time worked, where the server would be needed to provide the content of every new menu screen interactively. This was a core feature of PlayNET.

Second, this message is sort of similar to the MA message used previously to load the ACC module from the main menu module, but this one comes with a larger payload. The bytes following MX are inferred to contain another kind of module identifier, in which "T" is an internal alias for the "TLK" module known to the catalog system. [We have not yet firmly established what the "S004" characters represent, but there is some evidence that it may be a tag meaningful to some kind of offline content management system.]

The message then participates in another module transition sequence, as before:

#### Wire Interactions
|Dir|Message ID|Meaning (Inferred)|
|---|---|---|
|C→S|MX|Request to transition to a payload-specified module|
|S→C|MC|Granting permission for that module transition|
|C→S|SS|Request to suspend usage metering|
|S→C|SS|Acknowledging suspend of usage metering|

Once again, we see the red screen signaling a module transition, followed by the loading screen for the TLK module:

<img src="/images/basic_red.png">

<img src="/images/tlk_load.png">

After loading, the client sends the expected SG message but then it is followed by a new one, MR:

#### Wire Interactions
|Dir|Message ID|Meaning (Inferred)|
|---|---|---|
|C→S|SG|Request to resume usage metering|
|C→S|MR|Enter the main chat reception room|

At this point, we have demonstrated the mechanics of cross-menu-section transition, but the TLK module has some special features that are worth exploring a bit further.

The TLK module is the gateway to a couple of the major features of PlayNET: chat rooms and games. They are related in a sense because the rooms not only help users coordinate gameplay but also provide the medium for taunting opponents and celebrating victories.

In response to the MR message, the server responds with several additional messages:

#### Wire Interactions
|Dir|Message ID|Meaning (Inferred)|Payload|
|---|---|---|---|
|S→C|CM|Room name|"Reception 1"|
|S→C|CA|Room occupant list add|1, "Jack"|
|S→C|CA|Room occupant list add|2, "Diane"|
|S→C|CE|Room occupant list end|3, "Jenny"|

Then the room chat screen is displayed:

<img src="/images/tlk_screen.png">

Much to Jenny's delight, she sees that her good friend Diane is online along with Diane's boyfriend Jack. She types a greeting and presses RETURN:

<img src="/images/tlk_screen2.png">

Her message is forwarded to the other participants in the room, as evidenced by the chat log on her screen, and is soon followed by return greetings from her online pals:

<img src="/images/tlk_screen3.png">

On the wire, the exchange looks like this:

#### Wire Interactions
|Dir|Message ID|Meaning (Inferred)|Payload|
|---|---|---|---|
|C→S|AA|Send online message|"hi guys!"|
|S→C|AA|Receive online message|3, "hi guys!"|
|S→C|AA|Receive online message|2, "HEY GF!!!"|
|S→C|AA|Receive online message|1, "hubba hubba"|
|S→C|AA|Receive online message|2, "eyes on me Jacky boy ;)"|

Jenny then proposes a game with her friend and begins to set it up from the menu:

#### Wire Interactions
|Dir|Message ID|Meaning (Inferred)|Payload|
|---|---|---|---|
|C→S|AA|Send online message|"come on Di,lets go play HANGMAN lol"|
|S→C|AA|Send online message|3, "come on Di,lets go play HANGMAN lol"|

<img src="/images/tlk_screen4.png">

This concludes the protocol introduction. The next section provides a more comprehensive examination of the entire protocol set.

## 4.2 PlayNET Protocol Reference

This section describes the wire format of the PlayNET protocol messages.

As revealed in the preceding primer, every message exchanged between the client and the server represents a three-tiered stack of data encapsulations.

At the top of the stack is the session layer. Data at this layer are consumed directly by the application logic at either endpoint, having been successfully delivered and validated by the underlying layers.

The session layer is encapsulated by the transport layer, which is responsible for ensuring that messages within a session are delivered in the correct order. It is also responsible for routing transport-layer data to the appropriate handler.

The transport layer is then encapsulated by the bottommost framing layer. This layer is responsible for detecting and discarding messages that may have been corrupted during their journey through the noisy and unreliable public switched telephone network. It also ensures that the data transmitted over the wire are compatible with systems on the network that might otherwise misinterpret them.

Each of these layers is described in more detail below.

---

### 4.2.1 Framing Layer

The basic PlayNET protocol data unit (PDU) found on the wire is the frame. Its function is roughly equivalent to that of a Data Link layer frame in more modern protocols based on the Open Systems Interconnection (OSI) model.

It looks like this:


```
C0 C1 C2 C3 <body ...> $0D
|           |          |
|           |          +-- terminator ($0D)
|           +------------- body: 3+ bytes (the PDU transport layer)
+------------------------- 4-byte packed-nibble CRC of the body
```

- A frame has no start marker. They are consecutive $0D-terminated units on the wire. The receiver confirms the integrity of the frame by recomputing the CRC of the transport-layer body. Any frame which fails its CRC check is discarded.
- The frame is delimited at the end by the first *unescaped* $0D, aka terminator.
- There is no length field. The decoded transport-layer body length = frame length - 4 (CRC) - 1 ($0D), before escape expansion.
- The minimum frame length is 8 bytes: 4 (CRC) + 3-byte minimum transport-layer body + $0D.

#### Byte Transparency and the In-Band Escape

An in-band escape mechanism ensures that no byte of the transport-layer body collides with the $0D terminator.

Typical bodies already avoid terminator conflict by design:
- Sequence and acknowledgment numbers stay in the range $10-$7F.
- Command bytes are $20-$26.
- Message IDs and text are printable ASCII.

The escape is therefore exercised almost exclusively by the binary file transfer data. Any byte value may nonetheless appear in a body once escaped.

The encoding mechanism is simple. For each byte B in the set { $00, $0D, $0E, $5D, $8D, $8E, $FF }, it is transcribed as { $5D, XOR(B, $55) }:
- $00 -> $5D $55  ; avoids transmitting NULLs
- $0D -> $5D $58  ; avoids collision with framing terminator
- $0E -> $5D $5B  ; avoids ASCII Shift Out (SO) control character
- $5D -> $5D $08  ; avoids the escape byte itself
- $8D -> $5D $D8  ; avoids high-bit framing terminator
- $8E -> $5D $DB  ; avoids high-bit ASCII Shift Out (SO) control character
- $FF -> $5D $AA  ; avoids Telnet IAC

The decoding is equally simple. For each occurance of $5D, apply XOR(B, $55) to the next byte B read.

#### Cyclic Redundancy Check (CRC)

The 4 CRC bytes authenticate the de-escaped transport-layer body. The CRC field itself is never escaped. The algorithm used is CRC-16/ARC:
- Right-shifting, LSB first.
- Polynomial = $A001 (the bit-reversed form of $8005).
- Initial value = $0000, no finaly XOR, no final reflection.

Here is the C language code equivalent:

```
uint16_t crc16(const uint8_t *body, uint16_t len) {
    uint16_t crc = 0;
    for (uint16_t i = 0; i < len; i++) {
        uint8_t b = body[i];
        for (int j = 0; j < 8; j++) {
            if ((b ^ (uint8_t)crc) & 1) crc = (crc >> 1) ^ 0xA001;
            else crc = crc >> 1;
            b >>= 1;
        }
    }
    return crc;
}
```

To prevent the un-ecaped bytes of the 16-bit CRC value from colliding with the terminator ($0D) or escape ($5D) values, a special packed-nibble encoding is applied, resulting in a 4-byte sequence where each byte falls in the range $01-$4F:

```
hi = crc >> 8;
lo = crc & $FF;
C0 = (hi & 0xF0) | 0x01;  /* high nibble of hi byte */
C1 = (hi & 0x0F) | 0x40;  /* low nibble of hi byte */
C2 = (lo & 0xF0) | 0x01;  /* high nibble of lo byte */
C3 = (lo & 0x0F) | 0x40;  /* low nibble of lo byte */
```

Decoding inverts this:

```
uint16_t decode_crc(const uint8_t w[4]) {
    uint8_t hi = (w[0] & 0xF0) | (w[1] & 0x0F);
    uint8_t lo = (w[2] & 0xF0) | (w[3] & 0x0F);
    return ((uint16_t)hi << 8) | lo;
}
```

---

### 4.2.2 Transport Layer

The transport-layer PDU is carried within the body of the frame. It is responsible for tracking the message sequencing state of the endpoints and determining which of several transport-layer handlers should receive the data.

Mapped to a C language struct it looks like this:

```
struct transport_pdu {
   uint8_t seq;        /* body[0]: sender's sequence number for this PDU */
   uint8_t ack;        /* body[1]: cumulative acknowledgement number of the peer */
   uint8_t command;    /* body[2]: transport-layer command/handler */
   uint8_t payload[];  /* body[3..]: present only for command $20 */
};
```

- `seq` and `ack` are always present and in the range $10-$7F.
- `command` is always in the range $20-$26.
- If `command` is $20 (Data) then a variable-length payload containing session data will be present.

#### Sequence Numbers

Each endpoint maintains an outgoing sequence counter and tracks the latest sequence number reported by its peer. The value ranges from $10 to $7F, inclusive, wrapping around to $10 as needed.

A sender advances its sequence number by one for each frame of command type $20 (Data) or $25 (Null) that it sends. The sequence counters are initialized to $7F during session establishment and wrap around to $10 with the first command type $20 frame sent.

#### Acknowledgment Numbers

The `ack` value is always the sender's cumulative, in-order acknowledgment of its peer, i.e., the highest `seq` accepted from the peer with no preceding gaps. If the `ack` value fails to advance across successive PDUs, it means the sender is stalled, waiting for `ack` + 1.

#### Reliable Delivery

Each endpoint of the transport layer maintains both send and receive buffers, each with eight frame slots, for the purpose of recovering from data loss and corruption.

An endpoint may have at most eight of its own *sequence-advancing* frames (type $20 (Data) or $25 (Null)) unacknowledged by its peer. The `ack` value of an incoming frame frees the send buffer slots up to and including the acknowledged sequence number.

An out-of-order frame is retained in the receive buffer until the preceding gap is filled, while an unacknowledged outgoing frame remains in the send buffer until it is acknowledged. The receiver will not advance its `ack` value past a missing sequence. As the sender retransmits missing frames and the gaps in the receiver's buffer are filled, received frames are accepted in order, and the receiver advances its `ack` value accordingly. Thus, a single acknowledgment can acknowledge several frames at once.

After every eight frames accepted in order, a receiver sends a command type $24 (Ack) frame containing its cumulative `ack` value. This frame is not sequence-advancing, so it frees the peer's send window without consuming sequence space. To keep the peer's window open, an Ack frame containing an advanced `ack` value should be sent at least once for every eight frames received.

Retransmitted frames retain their original `seq` values but carry the latest `ack` value.

#### Transport Commands

In addition to the sequence and acknowledgement bytes, the transport layer also carries a command byte that indicates what kind of transport-layer PDU it is. That's used by the receiver to know which protocol handler the message should be routed to.

Here is the complete list of the transport command byte values along with their semantics.

|Command|Name<br>(Inferred)|Dir|Advances<br>Sequence?|Parameters|Role|
|---|---|---|---|---|---|
|$20|Data|C/S|Y||Carries session-layer data.|
|$21|Ping|C/S|N||A transport-level liveness check that can be originated by either endpoint. Receiver replies with $22.|
|$22|Pong|C/S|N||The expected response to $21.|
|$23|Open|C→S|N|$04, $00|Opens/resets a session. The meanings of its two parameters are not yet established.|
|$24|Ack|C/S|N||Explicit transport-layer acknowledgement. Directly acknowledges $23, and during normal operation is a periodic acknowledgement each endpoint emits after every 8 in-order frames (either direction).|
|$25|Null|C/S|Y||A session-layer non-data message that must be eventually acknowledged by the peer. Client periodically sends it when idle, so it may be used as a session keep-alive. Peer acknowledgement can be satisfied with another $25 response.|
|$26|Idle|C→S|N||A transport-layer idle indicator. Client periodically sends it when idle. Does not solicit a server response.|

---

### 4.2.3 Session Layer

A session-layer message is always encapsulated by a command type $20 (Data) transport-layer message, as its body.

It has a simple format. Invariably, it consists of two bytes comprising an ASCII message ID followed by zero or more bytes as parameters. Mapped to C it looks like this:

```
struct session_message {
   uint8_t id[2];         /* payload[0,1]: message ID */
   uint8_t parameters[];  /* payload[2..]: optional parameters */
};
```

Thanks to the transport and framing layers, these messages are delivered to the receiver in-order and free of data corruption errors. In a sense, that makes them similar to remote procedure calls (RPCs), where the ID acts as a function name and the parameters act as function arguments. In this case, however, there are no returned values.

The next section describes the set of supported session messages in greater detail.

---

### 4.2.4 Session Message Dictionary

There are 282 individual session-layer messages exchanged between the client and the server.  112 are client-to-server only, 143 are server-to-client only, and 27 are bi-directional.

Each module has its own particular messaging needs depending on the features it offers.  There are many messages that are used by all modules and several that are module-specific.

The sections below catalog the message IDs by module and directionality. Finally, there is a master list of messages indexed by name.

#### 4.2.4.1 MMM Module Messages

These messages are used by the Main Menu module.

IDs marked with an asterisk ("*") are unique to the module.

##### Client->Server

|Family|Message IDs|
|---|---|
| D | D1\* D2\* DD\* |
| G | GN |
| L | LO |
| M | MA MB MD MO\* MP\* |
| O | OR |
| P | PK |
| Q | QK |
| S | SS |
| X | XX |
| Z | ZA ZM ZN ZO ZY |

##### Server->Client

|Family|Message IDs|
|---|---|
| D | D! D1\* DA DK DO DQ DX DZ |
| G | GP |
| L | LO |
| M | MC |
| O | ON |
| P | PG |
| Q | QM |
| S | SE SM SS |
| X | XS XX XZ |
| Z | ZE ZH ZI ZM ZQ ZT ZZ |

#### 4.2.4.2 ACC Module Messages

These messages are used by the "Account & Billing Information" module.

IDs marked with an asterisk ("*") are unique to the module.

##### Client->Server

|Family|Message IDs|
|---|---|
| B | B1\* B2\* B3\* B4\* B5\* B6\* B7\* B8\* B9\* BA\* BB\* BG BI\* BJ\* BR\* BS\* BW\* BX\* BY\* BZ\* |
| E | EE EM EN |
| G | GN |
| L | LO |
| M | MB MD MF MX |
| O | OA OE OM ON OQ OR |
| P | PK |
| Q | QK |
| R | RM |
| S | SG SS |
| X | XX |
| Z | Z0 ZA ZC ZM ZN ZO ZY |

##### Server->Client

|Family|Message IDs|
|---|---|
| B | B!\* BC\* BD\* BE\* BF\* BH BK\* BL\* BM\* BN\* BO\* BP\* BQ\* BT\* BU\* BV\* BW\* BX\* |
| D | D! DA DK DQ DX DZ |
| E | E1 E2 EK EW |
| G | GP |
| L | LO |
| M | MC |
| O | OC OE OK OM ON OT OX |
| P | PG |
| Q | QM |
| R | RD RE RF RN RS |
| S | SS |
| X | XS XX |
| Z | ZC ZE ZH ZI ZM ZQ ZT ZZ |

#### 4.2.4.3 BBB Module Messages

These messages are used by the "Bulletin Boards & Special Interest Groups" module.

IDs marked with an asterisk ("*") are unique to the module.

##### Client->Server

|Family|Message IDs|
|---|---|
| E | EE EM EN |
| G | GN |
| H | H0\* H1\* H2\* H3\* H5\* H8\* H9\* HI\* HJ\* |
| L | LO |
| M | MX |
| O | OA OE OM ON OQ OR |
| P | PK |
| Q | QK |
| R | RM |
| S | SG SS |
| X | XX |

##### Server->Client

|Family|Message IDs|
|---|---|
| E | E1 E2 EK EW |
| G | GP |
| H | HA\* HB\* HC\* HD\* HE\* HF\* HG\* HH\* HK\* HR\* HX\* HY\* HZ\* |
| L | LO |
| M | MC |
| O | OC OE OK OM ON OT OX |
| P | PG |
| Q | QM |
| R | RD RE RF RN RS |
| S | SS |
| X | XS XX XZ |

#### 4.2.4.4 DLD Module Messages

These messages are used by the "Update PlayNET Disks" module.

IDs marked with an asterisk ("*") are unique to the module.

##### Client->Server

|Family|Message IDs|
|---|---|
| E | EE EG\* EM EN |
| F | FC\* FH\* FI\* FR\* |
| G | GN |
| L | LO |
| M | MA MB MD MX |
| O | OA OE OM ON OQ OR |
| P | PK |
| Q | QK |
| R | RM |
| S | SS |
| X | XX |

##### Server->Client

|Family|Message IDs|
|---|---|
| D | DA DK DO |
| E | E1 E2 EK EW |
| F | FA\* FB\* FE\* FG\* FH\* FI\* FJ\* FN\* FR\* FX FY\* |
| G | GP |
| L | LO |
| M | MC |
| O | OC OE OK OM ON OT OX |
| P | PG |
| Q | QM |
| R | RD RE RF RN RS |
| S | SE SS |
| X | XS XX XZ |

#### 4.2.4.5 LIB Module Messages

These messages are used by the "Software Delivery Service" module.

IDs marked with an asterisk ("*") are unique to the module.

##### Client->Server

|Family|Message IDs|
|---|---|
| E | EE EM EN |
| F | F1\* F2\* F3\* F4\* FK\* FO\* FX |
| G | GN |
| K | K0\* K1\* K2\* K3\* K4\* K5\* K8\* K9\* KI\* KJ\* KX\* |
| L | LO |
| M | MF ML\* MQ\* MX |
| O | OA OE OM ON OQ OR |
| P | PK |
| Q | QK |
| R | RM |
| S | SG SS |

##### Server->Client

|Family|Message IDs|
|---|---|
| E | E1 E2 EK EW |
| F | F5\* F6\* F7\* F8\* F9\* FD\* FK\* FO\* |
| G | GP |
| K | KA\* KB\* KC\* KD\* KE\* KF\* KG\* KH\* KK\* KR\* KX\* KY\* KZ\* |
| L | LO |
| M | MC |
| O | OC OE OK OM ON OT OX |
| P | PG |
| Q | QM |
| R | RD RE RF RN RS |
| S | SS |
| X | XS XX |

#### 4.2.4.6 STP Module Messages

These messages are used by the "PlayNET Shopping Center" module.

IDs marked with an asterisk ("*") are unique to the module.

##### Client->Server

|Family|Message IDs|
|---|---|
| E | EE EM EN |
| G | GN |
| L | LO |
| M | MX |
| O | OA OE OM ON OQ OR |
| P | PK |
| Q | QK |
| R | RM |
| S | SS |
| T | T1\* T2\* T3\* T4\* T5\* T6\* T7\* T9\* TI\* TJ\* TL\* TM\* |
| X | XX |
| Z | ZA ZC ZM ZN ZO ZY |

##### Server->Client

|Family|Message IDs|
|---|---|
| E | E1 E2 EK EW |
| G | GP |
| L | LO |
| M | MC |
| O | OC OE OK OM ON OT OX |
| P | PG |
| Q | QM |
| R | RD RE RF RN RS |
| S | SS |
| T | T0\* T5\* T6\* T7\* T8\* TA\* TB\* TC\* TD\* TE\* TF\* TG\* TK\* TX\* TZ\* |
| X | XX |
| Z | ZE ZH ZI ZM ZQ ZT ZZ |

#### 4.2.4.7 TLK Module Messages

These messages are used by the "People, Games & More!" module.

IDs marked with an asterisk ("*") are unique to the module.

##### Client->Server

|Family|Message IDs|
|---|---|
| A | AA\* |
| B | BG |
| C | CM\* CP\* |
| E | EE EM EN EQ\* |
| G | G!\* G4\* G8\* GB\* GI\* GN GU\* GY\* GZ\* |
| I | ID\* IG\* IX\* |
| J | J1\* JG\* |
| L | LB\* LC\* LG\* LN\* LO LQ\* |
| M | MA MB MD MF MR\* MS\* MX |
| O | OA OE OM ON OQ OR |
| P | PC\* PE\* PI\* PK PN\* PO\* PR\* PS\* PU\* PX\* |
| Q | QK QN\* QO\* QY\* |
| R | RM |
| S | SG SS |
| X | XX |
| Y | YN\* YQ\* |
| Z | Z0 ZA ZC ZK\* ZM ZN ZO ZY |

##### Server->Client

|Family|Message IDs|
|---|---|
| A | AA\* AB\* |
| B | BH |
| C | C1\* C2\* C3\* CA\* CB\* CE\* CL\* CM\* |
| D | D! DA DK DO DQ DX DZ |
| E | E1 E2 E3\* E4\* E5\* E6\* EK EW |
| G | G0\* G1\* G2\* G3\* G5\* G6\* G7\* G9\* GD\* GE\* GL\* GO\* GP GX\* |
| I | ID\* |
| J | J3\* JF\* JK\* JP\* |
| L | LD\* LE\* LF\* LH\* LM\* LO LP\* |
| M | MC |
| O | OC OE OK OM ON OT OX |
| P | PG PK PO\* |
| Q | QE\* QK QM QQ\* QT\* |
| R | RD RE RF RN RS |
| S | SB\* SC\* SE SM SN\* SP\* SS SX\* |
| X | XS XX XZ |
| Z | ZC ZE ZH ZI ZM ZQ ZT ZZ |

#### 4.2.4.8 Message Master List

All known session messages are listed here. The message data have been sourced from both static and runtime code analysis, and their purposes have been determined by observed [obs] or inferred [inf] effects. In many cases, the purpose has yet to be determined.

Client/server directionality is provided. `C/S` means a message is bi-directional.

Message arguments are represented by their C language equivalents.

| Code | Dir | Message arguments | Provenance | Purpose |
|---|---|---|---|---|
| `AA` | C/S | uint8 seat, char text[] | [obs] | People Connection public chat line (broadcast) |
| `AB` | S→C | uint8 idx, char text[]  (multi-field, schema not fully pinned) | [obs] | Structured multi-field People Connection record — dedicated parser $E287 stages index + text/name fields into $CE16/$CEB8 (purpose TBD) |
| `B!` | S→C | char field[] | [obs] | ACC record/field verb (purpose TBD) |
| `B1` | C→S | void | [obs] | ACC record/field verb (purpose TBD) |
| `B2` | C→S | void | [obs] | ACC record/field verb (purpose TBD) |
| `B3` | C→S | char field[] | [obs] | ACC record/field verb (purpose TBD) |
| `B4` | C→S | char field[] | [obs] | ACC record/field verb (purpose TBD) |
| `B5` | C→S | void | [obs] | ACC record/field verb (purpose TBD) |
| `B6` | C→S | record (multi-field) | [obs] | ACC record/field verb (purpose TBD) |
| `B7` | C→S | char field[] | [obs] | ACC record/field verb (purpose TBD) |
| `B8` | C→S | void | [obs] | ACC record/field verb (purpose TBD) |
| `B9` | C→S | void | [obs] | ACC record/field verb (purpose TBD) |
| `BA` | C→S | char field[] | [obs] | ACC record/field verb (purpose TBD) |
| `BB` | C→S | char field[] | [obs] | ACC record/field verb (purpose TBD) |
| `BC` | S→C | record (multi-field) | [obs] | ACC record/field verb (purpose TBD) |
| `BD` | S→C | void | [obs] | ACC record/field verb (purpose TBD) |
| `BE` | S→C | void | [obs] | ACC record/field verb (purpose TBD) |
| `BF` | S→C | void | [obs] | ACC record/field verb (purpose TBD) |
| `BG` | C→S | varies by module | [obs] | (purpose TBD) |
| `BH` | S→C | varies by module | [obs] | ACC record/field verb (purpose TBD) — payload differs by module: ACC=uint8 idx, TLK=record (multi-field) |
| `BI` | C→S | void | [obs] | ACC record/field verb (purpose TBD) |
| `BJ` | C→S | char field[] | [obs] | ACC record/field verb (purpose TBD) |
| `BK` | S→C | record (multi-field) | [obs] | ACC record/field verb (purpose TBD) |
| `BL` | S→C | record (multi-field) | [obs] | ACC record/field verb (purpose TBD) |
| `BM` | S→C | record (multi-field) | [obs] | ACC record/field verb (purpose TBD) |
| `BN` | S→C | void | [obs] | ACC record/field verb (purpose TBD) |
| `BO` | S→C | record (multi-field) | [obs] | ACC record/field verb (purpose TBD) |
| `BP` | S→C | record (multi-field) | [obs] | ACC record/field verb (purpose TBD) |
| `BQ` | S→C | record (multi-field) | [obs] | ACC record/field verb (purpose TBD) |
| `BR` | C→S | void | [obs] | ACC record/field verb (purpose TBD) |
| `BS` | C→S | char field[] | [obs] | ACC record/field verb (purpose TBD) |
| `BT` | S→C | record (multi-field) | [obs] | ACC record/field verb (purpose TBD) |
| `BU` | S→C | void | [obs] | ACC record/field verb (purpose TBD) |
| `BV` | S→C | char field[] | [obs] | ACC record/field verb (purpose TBD) |
| `BW` | C/S | char field[] | [obs] | ACC record/field verb (purpose TBD) |
| `BX` | C/S | varies by module | [obs] | ACC record/field verb (purpose TBD) |
| `BY` | C→S | void | [obs] | ACC record/field verb (purpose TBD) |
| `BZ` | C→S | void | [obs] | ACC record/field verb (purpose TBD) |
| `C1` | S→C | void | [obs] | Room-change REJECT — "engaged in game start-up" |
| `C2` | S→C | void | [obs] | Room-change REJECT — "room is filled to capacity" |
| `C3` | S→C | void | [obs] | Room-change REJECT — "public room cannot be private" |
| `CA` | S→C | uint8 seat, char name[] | [inf] | Room occupant ADD |
| `CB` | S→C | uint8 seat | [inf] | Room occupant LEAVE |
| `CE` | S→C | uint8 seat, char name[] | [inf] | Room occupant ENTRY / finalize → chat |
| `CL` | S→C | uint8 seat, char name[] | [inf] | Public-room occupant-add (loading) |
| `CM` | C/S | char name[] | [obs] | Create/enter a public room by name (→ MC/CA/CE) |
| `CP` | C→S | char name[] | [inf] | Create/enter a PRIVATE room by name |
| `D!` | S→C | varies by module | [obs] | Sign-on code (purpose TBD) — payload differs by module: ACC=uint8 idx, TLK=void |
| `D1` | C/S | char field[] | [obs] | Sign-on dialog #1 |
| `D2` | C→S | char field[] | [obs] | Sign-on dialog step |
| `DA` | S→C | varies by module | [obs] | Sign-on code (accepted at the sign-on wait) (purpose TBD) — payload differs by module: ACC=char field[], TLK=void, DLD=char field[] |
| `DD` | C→S | char acct[10], char pass[] | [obs] | Legacy sign-on login (account+access-code), → DO or DK/DX |
| `DK` | S→C | varies by module | [obs] | Reject: invalid account — payload differs by module: ACC=char field[], TLK=void, DLD=uint8 idx |
| `DO` | S→C | varies by module | [obs] | Display main menu — sign-on validation OK, proceed to main menu — payload differs by module: TLK=void, DLD=char field[] |
| `DQ` | S→C | void | [obs] | Sign-on code (purpose TBD) |
| `DX` | S→C | varies by module | [obs] | Reject: invalidated account — payload differs by module: ACC=char field[]; uint8 n; uint8 arr[n], TLK=void |
| `DZ` | S→C | varies by module | [obs] | Sign-on code (purpose TBD) — payload differs by module: ACC=char field[]; uint8 n; uint8 arr[n], TLK=void |
| `E1` | S→C | varies by module | [obs] | Event slot (purpose TBD) — payload differs by module: ACC=uint8 idx, TLK=void, DLD=char field[], BBB=char field[], LIB=void, STP=void |
| `E2` | S→C | varies by module | [obs] | Event slot (purpose TBD) — payload differs by module: ACC=void, TLK=void, DLD=void, BBB=char field[], LIB=void, STP=void |
| `E3` | S→C | void | [obs] | Event slot (purpose TBD) |
| `E4` | S→C | void | [obs] | Event slot (purpose TBD) |
| `E5` | S→C | void | [obs] | Event slot (purpose TBD) |
| `E6` | S→C | void | [obs] | Event slot (purpose TBD) |
| `EE` | C→S | char text[] | [inf] | E-mail compose BODY line |
| `EG` | C→S | void | [obs] | Download code (DLD) (purpose TBD) |
| `EK` | S→C | varies by module | [obs] | Inbox-listing entry — payload differs by module: ACC=void, TLK=record (multi-field), DLD=void, BBB=void, LIB=record (multi-field), STP=record (multi-field) |
| `EM` | C→S | char text[] | [inf] | E-mail compose HEADER |
| `EN` | C→S | char recipient[] | [inf] | E-mail compose RECIPIENT / answer |
| `EQ` | C→S | void | [obs] | Mail/event code (purpose TBD) |
| `EW` | S→C | varies by module | [obs] | Compose-done / end-of-listing — payload differs by module: ACC=char field[], TLK=void, DLD=char field[]; uint8 n; uint8 arr[n], BBB=void, LIB=void, STP=void |
| `F1` | C→S | uint8 data[] | [inf] | Download/upload data frame |
| `F2` | C→S | uint8 data[]  (escaped) | [obs] | Upload data block (PlayNET streams F2/F3/FX) |
| `F3` | C→S | uint8 data[]  (escaped) | [obs] | Upload data block |
| `F4` | C→S | char filename[], char uploader[] | [obs] | Upload request (open + identity) |
| `F5` | S→C | char field[] | [obs] | File-transfer control |
| `F6` | S→C | uint8 data[] | [inf] | Init download / "file located" |
| `F7` | S→C | uint8 data[] | [inf] | Download SEND block |
| `F8` | S→C | uint8 data[] | [inf] | Download SAVE block |
| `F9` | S→C | char field[] | [obs] | Download END |
| `FA` | S→C | record (multi-field) | [obs] | File-transfer code (DLD) (purpose TBD) |
| `FB` | S→C | void | [obs] | File-transfer code (DLD) (purpose TBD) |
| `FC` | C→S | char field[] | [obs] | File-transfer code (DLD) (purpose TBD) |
| `FD` | S→C | char field[] | [obs] | File-transfer verdict/metadata |
| `FE` | S→C | record (multi-field) | [obs] | File-transfer code (DLD) (purpose TBD) |
| `FG` | S→C | void | [obs] | File-transfer code (DLD) (purpose TBD) |
| `FH` | C/S | void | [obs] | File-transfer code (DLD) (purpose TBD) |
| `FI` | C/S | char plat[4], uint8 b1, uint8 b2, char cat[3] | [obs] | Update-Disks download HEADER record |
| `FJ` | S→C | uint8 data[] | [obs] | Update-Disks download DATA record |
| `FK` | C/S | void | [obs] | File-transfer keepalive (seen during upload/PFT waits) |
| `FN` | S→C | record (multi-field) | [obs] | File-transfer code (DLD) (purpose TBD) |
| `FO` | C/S | char field[] | [obs] | Start download (pull) |
| `FR` | C/S | char field[] | [obs] | File-transfer code (DLD) (purpose TBD) |
| `FX` | C/S | uint8 data[]  (escaped) | [obs] | Upload data block (final/variant) |
| `FY` | S→C | record (multi-field) | [obs] | Update-Disks download FINALIZE (gated by on-disk catalog count) |
| `G!` | C→S | void | [obs] | Game-ID / in-game action variant (purpose TBD) |
| `G0` | S→C | char field[] | [obs] | Game-ID / in-game action variant (purpose TBD) |
| `G1` | S→C | uint8 idx | [obs] | Game-ID / in-game action variant (purpose TBD) |
| `G2` | S→C | char field[] | [obs] | Game-ID / in-game action variant (purpose TBD) |
| `G3` | S→C | uint8 idx | [obs] | Game-ID / in-game action variant (purpose TBD) |
| `G4` | C→S | void | [obs] | Game-ID / in-game action variant (purpose TBD) |
| `G5` | S→C | uint8 idx | [obs] | Game-ID / in-game action variant (purpose TBD) |
| `G6` | S→C | uint8 idx | [obs] | Game-ID / in-game action variant (purpose TBD) |
| `G7` | S→C | uint8 idx | [obs] | Game-ID / in-game action variant (purpose TBD) |
| `G8` | C→S | void | [obs] | Game-ID / in-game action variant (purpose TBD) |
| `G9` | S→C | uint8 idx | [obs] | Game-ID / in-game action variant (purpose TBD) |
| `GB` | C→S | void | [obs] | Start a game |
| `GD` | S→C | void | [obs] | Game-ID / in-game action (purpose TBD) |
| `GE` | S→C | void | [obs] | Leave the game |
| `GI` | C→S | char field[] | [obs] | Request a game |
| `GL` | S→C | void | [obs] | Accept-ack |
| `GN` | C→S | varies by module | [obs] | Decline a game invitation |
| `GO` | S→C | uint8 n, uint8 arr[n] | [obs] | Prepare a game |
| `GP` | S→C | varies by module | [obs] | Invite a player to a game — payload differs by module: ACC=char field[], TLK=record (multi-field), DLD=char field[], BBB=void, LIB=void, STP=void |
| `GU` | C→S | uint8 seat | [inf] | Self-seat |
| `GX` | S→C | void | [obs] | In-game action (purpose TBD) |
| `GY` | C→S | char field[] | [obs] | Accept a game invitation |
| `GZ` | C→S | uint8 seat | [inf] | Partner-seat |
| `H0` | C→S | record (multi-field) | [obs] | BBB record/field verb (purpose TBD) |
| `H1` | C→S | void | [obs] | BBB record/field verb (purpose TBD) |
| `H2` | C→S | char field[] | [obs] | BBB record/field verb (purpose TBD) |
| `H3` | C→S | void | [obs] | BBB record/field verb (purpose TBD) |
| `H5` | C→S | record (multi-field) | [obs] | BBB record/field verb (purpose TBD) |
| `H8` | C→S | record (multi-field) | [obs] | BBB record/field verb (purpose TBD) |
| `H9` | C→S | record (multi-field) | [obs] | BBB record/field verb (purpose TBD) |
| `HA` | S→C | char field[] | [obs] | BBB record/field verb (purpose TBD) |
| `HB` | S→C | char field[] | [obs] | BBB record/field verb (purpose TBD) |
| `HC` | S→C | char field[] | [obs] | BBB record/field verb (purpose TBD) |
| `HD` | S→C | char field[] | [obs] | BBB record/field verb (purpose TBD) |
| `HE` | S→C | char field[] | [obs] | BBB record/field verb (purpose TBD) |
| `HF` | S→C | record (multi-field) | [obs] | BBB record/field verb (purpose TBD) |
| `HG` | S→C | char field[] | [obs] | BBB record/field verb (purpose TBD) |
| `HH` | S→C | void | [obs] | BBB record/field verb (purpose TBD) |
| `HI` | C→S | void | [obs] | BBB record/field verb (purpose TBD) |
| `HJ` | C→S | void | [obs] | BBB record/field verb (purpose TBD) |
| `HK` | S→C | void | [obs] | BBB record/field verb (purpose TBD) |
| `HR` | S→C | char field[] | [obs] | BBB record/field verb (purpose TBD) |
| `HX` | S→C | void | [obs] | BBB record/field verb (purpose TBD) |
| `HY` | S→C | char field[] | [obs] | BBB record/field verb (purpose TBD) |
| `HZ` | S→C | void | [obs] | BBB record/field verb (purpose TBD) |
| `ID` | C/S | char field[] | [obs] | Identifier |
| `IG` | C→S | record (multi-field) | [obs] | TLK code (purpose TBD) |
| `IX` | C→S | record (multi-field) | [obs] | TLK code (purpose TBD) |
| `J1` | C→S | record (multi-field) | [obs] | Suspend/journal code (TLK) (purpose TBD) |
| `J3` | S→C | uint8 idx | [obs] | Suspend/journal code (purpose TBD) |
| `JF` | S→C | uint8 idx | [obs] | Journal/suspend (purpose TBD) |
| `JG` | C→S | char field[] | [obs] | Suspend-save variant |
| `JK` | S→C | char field[] | [obs] | Decline / return |
| `JP` | S→C | record (multi-field) | [obs] | (purpose TBD) |
| `K0` | C→S | void | [inf] | Done / return (upload-done) |
| `K1` | C→S | uint8 obj[4] | [inf] | Select menu item (pick list item by object-ID) |
| `K2` | C→S | char field[] | [obs] | List/nav verb (purpose TBD) |
| `K3` | C→S | void | [obs] | List/nav verb (purpose TBD) |
| `K4` | C→S | uint8 obj[4] | [inf] | Download file |
| `K5` | C→S | record (multi-field) | [obs] | List/nav verb (purpose TBD) |
| `K8` | C→S | record (multi-field) | [obs] | List/nav verb (purpose TBD) |
| `K9` | C→S | void | [inf] | Done-viewing / return to list |
| `KA` | S→C | uint8 desc[9], uint8 sep, char text[28] | [obs] | File-list record (PlayNET §P35 record format) |
| `KB` | S→C | uint8 desc[9], uint8 sep, char text[28] | [obs] | File-list record |
| `KC` | S→C | char field[] | [obs] | LIB file-list record / browse-nav verb (purpose TBD) |
| `KD` | S→C | char field[] | [obs] | LIB file-list record / browse-nav verb (purpose TBD) |
| `KE` | S→C | char field[] | [obs] | LIB file-list record / browse-nav verb (purpose TBD) |
| `KF` | S→C | char field[] | [obs] | LIB file-list record / browse-nav verb (purpose TBD) |
| `KG` | S→C | char field[] | [obs] | LIB file-list record / browse-nav verb (purpose TBD) |
| `KH` | S→C | record (multi-field) | [obs] | LIB file-list record / browse-nav verb (purpose TBD) |
| `KI` | C→S | void | [obs] | List/nav verb (purpose TBD) |
| `KJ` | C→S | void | [obs] | List/nav verb (purpose TBD) |
| `KK` | S→C | char field[] | [obs] | LIB file-list record / browse-nav verb (purpose TBD) |
| `KR` | S→C | char field[] | [obs] | LIB file-list record / browse-nav verb (purpose TBD) |
| `KX` | C/S | void | [obs] | LIB file-list record / browse-nav verb (purpose TBD) |
| `KY` | S→C | char field[] | [obs] | LIB file-list record / browse-nav verb (purpose TBD) |
| `KZ` | S→C | void | [obs] | List terminator / "area empty" notice |
| `LB` | C→S | void | [obs] | Lobby/list verb (purpose TBD) |
| `LC` | C→S | void | [obs] | Lobby/list verb (purpose TBD) |
| `LD` | S→C | record (multi-field) | [obs] | Lobby/list verb (purpose TBD) |
| `LE` | S→C | char field[], uint8 n, uint8 arr[n] | [obs] | Lobby/list verb (purpose TBD) |
| `LF` | S→C | char field[], uint8 n, uint8 arr[n] | [obs] | Lobby/list verb (purpose TBD) |
| `LG` | C→S | void | [obs] | Lobby/list verb (purpose TBD) |
| `LH` | S→C | record (multi-field) | [obs] | Lobby/list verb (purpose TBD) |
| `LM` | S→C | record (multi-field) | [obs] | Lobby/list verb (purpose TBD) |
| `LN` | C→S | char field[] | [obs] | Lobby/list verb (purpose TBD) |
| `LO` | C/S | void | [obs] | Log-off / leave (back-out) |
| `LP` | S→C | char field[] | [obs] | Lobby/list verb (purpose TBD) |
| `LQ` | C→S | void | [obs] | Lobby/list verb (purpose TBD) |
| `MA` | C→S | void | [obs] | Enter ACC section |
| `MB` | C→S | void | [obs] | Section/profile code (purpose TBD) |
| `MC` | S→C | varies by module | [obs] | Menu-load ACK ("proceed into section") — payload differs by module: ACC=char field[], TLK=void, DLD=record (multi-field), BBB=char field[], LIB=void, STP=void |
| `MD` | C→S | void | [obs] | Enter DLD section |
| `MF` | C→S | void | [obs] | Enter file-transfer sub-function (Personal File Transfer) |
| `ML` | C→S | char field[] | [obs] | Menu-load request (PlayNET's observed section-entry is MO) |
| `MO` | C→S | void | [obs] | Section/menu-load request (enter a section from the menu) |
| `MP` | C→S | void | [obs] | Enter STP section |
| `MQ` | C→S | void | [obs] | Enter the Software Trading Post. Not to be confused with the STP section |
| `MR` | C→S | void | [obs] | Enter People Connection room (Lobby) → CM/CA/CE |
| `MS` | C→S | uint8 index | [inf] | Event-info / tell-me-more request |
| `MX` | C→S | char dest, char tag[4] | [obs] | Inter-section navigation to a target module (key-letter per cat.module) |
| `OA` | C→S | void | [inf] | OLM read-ack |
| `OC` | S→C | varies by module | [obs] | OLM code (purpose TBD) — payload differs by module: ACC=uint8 idx, TLK=void, DLD=char field[], BBB=void, LIB=void, STP=void |
| `OE` | C/S | char text[] | [inf] | OLM body line |
| `OK` | S→C | varies by module | [obs] | Ack — payload differs by module: ACC=record (multi-field), TLK=char field[], DLD=char field[], BBB=void, LIB=char field[], STP=char field[] |
| `OM` | C/S | char text[] | [inf] | OLM header line |
| `ON` | C/S | char target[] | [inf] | OLM start / target |
| `OQ` | C→S | varies by module | [obs] | OLM code (purpose TBD) |
| `OR` | C→S | varies by module | [obs] | OLM code (purpose TBD) |
| `OT` | S→C | varies by module | [obs] | OLM code (purpose TBD) — payload differs by module: ACC=void, TLK=char field[], DLD=void, BBB=char field[], LIB=char field[], STP=char field[] |
| `OX` | S→C | varies by module | [obs] | OLM code (purpose TBD) — payload differs by module: ACC=uint8 n; uint8 arr[n], TLK=void, DLD=char field[], BBB=void, LIB=void, STP=void |
| `PC` | C→S | void | [obs] | Cancel |
| `PE` | C→S | char field[] | [obs] | Find more |
| `PG` | S→C | varies by module | [obs] | Paging (purpose TBD) — payload differs by module: ACC=char field[], TLK=uint8 idx, DLD=uint8 idx, BBB=char field[], LIB=void, STP=void |
| `PI` | C→S | void | [obs] | Include me |
| `PK` | C/S | varies by module | [obs] | "not implemented" reply / paging |
| `PN` | C→S | char field[] | [obs] | Select partner |
| `PO` | C/S | void | [obs] | Matchmaking ack |
| `PR` | C→S | record (multi-field) | [obs] | Find partners |
| `PS` | C→S | void | [obs] | Status request |
| `PU` | C→S | void | [obs] | Exclude me |
| `PX` | C→S | void | [obs] | Paging (purpose TBD) |
| `QE` | S→C | char field[] | [obs] | Query code (purpose TBD) |
| `QK` | C/S | varies by module | [obs] | Query (ubiquitous) |
| `QM` | S→C | varies by module | [obs] | Query code (purpose TBD) — payload differs by module: ACC=record (multi-field), TLK=void, DLD=char field[], BBB=char field[], LIB=void, STP=void |
| `QN` | C→S | void | [obs] | Query code (purpose TBD) |
| `QO` | C→S | void | [obs] | Query code (purpose TBD) |
| `QQ` | S→C | char field[] | [obs] | Query code (purpose TBD) |
| `QT` | S→C | char field[] | [obs] | Query code (purpose TBD) |
| `QY` | C→S | void | [obs] | Query code (purpose TBD) |
| `RD` | S→C | varies by module | [obs] | Mail-waiting notification — payload differs by module: ACC=uint8 idx, TLK=void, DLD=void, BBB=void, LIB=void, STP=void |
| `RE` | S→C | varies by module | [obs] | Read-mail code (purpose TBD) — payload differs by module: ACC=void, TLK=char field[], DLD=uint8 idx, BBB=void, LIB=char field[], STP=char field[] |
| `RF` | S→C | varies by module | [obs] | End-of-mail footer — payload differs by module: ACC=char field[], TLK=char field[], DLD=uint8 idx, BBB=record (multi-field), LIB=char field[], STP=char field[] |
| `RM` | C→S | void | [inf] | Read-mail request |
| `RN` | S→C | varies by module | [obs] | Mail-start — payload differs by module: ACC=void, TLK=char field[], DLD=record (multi-field), BBB=char field[], LIB=char field[], STP=char field[] |
| `RS` | S→C | char line[] | [inf] | Mail text line (From/Date/body) |
| `SB` | S→C | char field[] | [obs] | Session code (purpose TBD) |
| `SC` | S→C | void | [obs] | Session code (purpose TBD) |
| `SE` | S→C | char field[] | [obs] | Session code (purpose TBD) |
| `SG` | C→S | void | [obs] | Advisory usage-metering RESUME (control returns to user) |
| `SM` | S→C | char field[] | [obs] | Session code (purpose TBD) |
| `SN` | S→C | record (multi-field) | [obs] | Session code (purpose TBD) |
| `SP` | S→C | void | [obs] | Session code (purpose TBD) |
| `SS` | C/S | void | [obs] | Advisory usage-metering SUSPEND + timing mark (enter blocked op) |
| `SX` | S→C | void | [obs] | Session code (purpose TBD) |
| `T0` | S→C | char field[] | [obs] | STP record/field verb (purpose TBD) |
| `T1` | C→S | void | [obs] | STP record/field verb (purpose TBD) |
| `T2` | C→S | void | [obs] | STP record/field verb (purpose TBD) |
| `T3` | C→S | record (multi-field) | [obs] | STP record/field verb (purpose TBD) |
| `T4` | C→S | record (multi-field) | [obs] | STP record/field verb (purpose TBD) |
| `T5` | C/S | record (multi-field) | [obs] | STP record/field verb (purpose TBD) |
| `T6` | C/S | record (multi-field) | [obs] | STP record/field verb (purpose TBD) |
| `T7` | C/S | record (multi-field) | [obs] | STP record/field verb (purpose TBD) |
| `T8` | S→C | char field[] | [obs] | STP record/field verb (purpose TBD) |
| `T9` | C→S | record (multi-field) | [obs] | STP record/field verb (purpose TBD) |
| `TA` | S→C | record (multi-field) | [obs] | STP record/field verb (purpose TBD) |
| `TB` | S→C | record (multi-field) | [obs] | STP record/field verb (purpose TBD) |
| `TC` | S→C | char field[] | [obs] | STP record/field verb (purpose TBD) |
| `TD` | S→C | char field[] | [obs] | STP record/field verb (purpose TBD) |
| `TE` | S→C | char field[] | [obs] | STP record/field verb (purpose TBD) |
| `TF` | S→C | char field[] | [obs] | STP record/field verb (purpose TBD) |
| `TG` | S→C | char field[] | [obs] | STP record/field verb (purpose TBD) |
| `TI` | C→S | record (multi-field) | [obs] | STP record/field verb (purpose TBD) |
| `TJ` | C→S | void | [obs] | STP record/field verb (purpose TBD) |
| `TK` | S→C | char field[] | [obs] | STP record/field verb (purpose TBD) |
| `TL` | C→S | void | [obs] | STP record/field verb (purpose TBD) |
| `TM` | C→S | void | [obs] | STP record/field verb (purpose TBD) |
| `TX` | S→C | void | [obs] | STP record/field verb (purpose TBD) |
| `TZ` | S→C | void | [obs] | STP record/field verb (purpose TBD) |
| `XS` | S→C | varies by module | [obs] | No-op/padding? (purpose TBD) — payload differs by module: ACC=void, TLK=char field[], DLD=uint8 n; uint8 arr[n], BBB=char field[], LIB=char field[] |
| `XX` | C/S | void | [inf] | Ubiquitous padding / no-op / generic |
| `XZ` | S→C | varies by module | [obs] | No-op/padding? (purpose TBD) — payload differs by module: TLK=void, DLD=char field[]; uint8 n; uint8 arr[n], BBB=void |
| `YN` | C→S | char field[] | [obs] | TLK code (purpose TBD) |
| `YQ` | C→S | void | [obs] | TLK code (purpose TBD) |
| `Z0` | C→S | varies by module | [obs] | Dialog code (purpose TBD) |
| `ZA` | C→S | void | [inf] | Confirm |
| `ZC` | C/S | varies by module | [obs] | Dialog code (purpose TBD) |
| `ZE` | S→C | varies by module | [obs] | Yes (defined, not seen on PlayNET wire) — payload differs by module: ACC=void, TLK=char field[], STP=char field[] |
| `ZH` | S→C | varies by module | [obs] | Dialog code (purpose TBD) — payload differs by module: ACC=void, TLK=char field[], STP=char field[] |
| `ZI` | S→C | varies by module | [obs] | Dialog code (purpose TBD) — payload differs by module: ACC=void, TLK=char field[], STP=char field[] |
| `ZK` | C→S | char field[] | [obs] | Dialog code (purpose TBD) |
| `ZM` | C/S | char field[] | [obs] | Create/echo dialog (drives form both ways) |
| `ZN` | C→S | void | [inf] | No (defined, not seen on PlayNET wire) |
| `ZO` | C→S | char body[] | [inf] | Form body |
| `ZQ` | S→C | char text[] | [inf] | Typed form entry (sign-on; see direction caveat) |
| `ZT` | S→C | char title[] | [inf] | Dialog title |
| `ZY` | C→S | void | [inf] | Yes/confirm variant |
| `ZZ` | S→C | varies by module | [obs] | Continue — payload differs by module: ACC=void, TLK=char field[], STP=char field[] |

---

In the [next chapter](/docs/chapter5.md) we'll dive even deeper into the PlayNET client to examine the engine at its core: the FSM.

