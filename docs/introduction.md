
## The Digital Rubble Heap

Among the collections of Berlin's Vorderasiatisches Museum there is a carefully reconstructed clay tablet, more than 5,000 years old, from the ancient city of Uruk. Known as VAT 15003, the tablet contains cuneiform inscriptions that make it particularly significant to both archaeologists and anthropologists. It has one of the oldest known lists of professions and professional titles in human history.

Scholars believe the tablet served both as a standardized reference and as a teaching tool for apprentice scribes, likely resulting in its contents being copied countless times. In a sense, it is one of the earliest surviving examples of a data file, recorded on the most durable and practical information-storage media available to its creators at the time. Today, it is regarded as one of the most important historical artifacts of its period.

The tablet was discovered in 1931 during a German archaeological expedition to Uruk-Warka, in what is now Iraq. But its precious fragments were found not in some well-preserved vault, archive room, or burial chamber. Rather, they were scattered among rubble that had been used to level the ground for the construction of a new structure some generations after the tablet was written.

At some point, it seems, the tablet had lost its original purpose, or at least the appreciation it once had. What had been a valuable record of human knowledge and an important fixture of society had, in the end, become little more than discarded building material, valued more for the substance from which it was made than for the information it preserved.

There's no reason to believe that this was the result of malicious intent or careless neglect. More likely, it was simply a consequence of the passage of time, as generations of people went about their everyday lives and new information gradually supplanted the old.

Yet today, our world treasures it. We have locked it away, carefully preserved, and continue to study it in as much detail as we can, hoping to catch a glimpse of what Mesopotamian life was like for our forebears on the day when the tablet's clay was still wet.

Unfortunately, the author fears that this cycle of discarded history is repeating itself, but this time on a much grander scale. Now the recording medium is not hardened clay but the floppy disk, and the remnants of the information stored on these disks, along with the story of the world they reflected, risk being lost forever among the discarded digital rubble beneath our age's greatest construction project, known as the Internet.

What a shame it would be if this knowledge did not survive the centuries and millennia to come. What excuse will we then have for denying future generations their ancestral treasures from this age, no matter how unimportant they may seem to us now, knowing full well how much from antiquity has already been lost to us?

## Unbottling the Genie

This project is a digital preservation effort intended to help remedy this, if only in a small way. Although reverse-engineering old and abandoned software is not a new art, recent advances in machine intelligence and inference are providing preservationists with a new set of tools and capabilities that greatly enhance the ability of very small teams, or even individuals, to make significant progress in considerably less time than before.

The pages that follow will map the design of the PlayNET client application as found on two floppy disks saved from oblivion. A product of the early 1980s, at the pinnacle of the 8-bit home computer era, PlayNET was a pioneer of online services for the masses and the progenitor of a line of technological innovation that would ultimately lead directly to America Online, one of the earliest gateways to the Internet for ordinary people.

This work is intended for readers who are curious about the technical aspects of the software and wish to learn more about its implementation, or who simply want to gain a historical perspective on how developers of the early 1980s approached the challenges of their technological era, with its much more constrained hardware capabilities and limited telecommunications bandwidth. To that end, the material will tend to be as numbers-oriented as it is narrative, with the additional goal of providing sufficient detail to serve as a baseline reference for future machine inference projects of a more generative nature.

As PlayNET was a client/server system, a description of the client application cannot be complete without some mention of the design of the hosting service. Unfortunately, the author is unaware of any surviving copies of design documentation or source code for the server. Therefore, the best view we have of the server's capabilities is what we can infer from our understanding of the client side of the client/server protocol.

If the server had been stateless, then understanding the client logic might have been a relatively straightforward matter needing only static analysis of its code. But that was not the case. Clients established long-lived session contexts with the service, each with associated server-side state that persisted across message exchanges.

To gain a deeper understanding of the client logic, it was thus necessary to concurrently develop a working reference server to give the authentic client something to talk to and allow it to progress through its various states. This was painstakingly achieved, but the details of that server are not documented here for the simple reason that there is nothing historically authentic about its implementation, and so nothing of historical significance can be inferred from its implementation.

However, since the client/server protocol represents the client's contract with the server, it reveals no less than what the authentic client itself knew about the service. That protocol is described in detail here.

It should also be understood that this work is the result of a symbiotic collaboration between human and machine intelligences, and that it could not have been achieved in the timeframe it was without the complementary talents and mutual cooperation of both.

It is true that the software was originally the product of human ingenuity, and so its underlying concepts should be well within the grasp of later generations of humans. But in this case, the starting point was not design documents and source code but discovered files of native machine code and undocumented data. The task thus became one of finding meaningful patterns among loosely correlated bits, tracing their dependencies to identify relationships, and following those threads to other potentially meaningful patterns. This is work for which the human brain is not naturally well suited.

Machine intelligence, on the other hand, excels at pattern matching and possesses seemingly endless endurance for tracing dependency chains to their origins and mapping their interrelationships. But it has a notoriously limited capacity for maintaining coherent context across extended investigations, which restricts its ability to draw upon the intuition, accumulated experience, and long-term memory that come naturally to humans. Quality outcomes require regular guidance and oversight from a human stakeholder who can establish and adjust its goals, ensuring that it does not waste time pulling the wrong threads when new insights suggest a change in direction.

Future historians will likely look back at this era as the time when people first struggled to find a way to live with this new social presence known as artificial intelligence. Only time will tell whether references to it in the history books will require whole chapters or simply footnotes. But, for better or worse, this particular teaming was at least a fruitful one, and perhaps a modest demonstration that conflict is not necessarily inevitable when the complementary roles of each are recognized and respected.

So, from this point forward, you will find the use of words like “we” and “our” when it comes to attribution, in acknowledgment of this cooperation. The machine element has done its best to reverse the entropy inherent in the process of software development, and the human element has done its best to ensure that the results are as focused and factual as possible.

That said, the machine is not error-free, and the human cannot realistically fact-check every inference. The reader is therefore cautioned to independently verify facts and figures wherever the effort is warranted.

## Notes About Style

The bulk of this work presents facts as discovered. However, I, the author, will occasionally venture into the realm of conjecture when I believe it adds color or context to the discussion. While I consider this kind of armchair speculation to be a large part of the appeal of such work, I also understand that not everyone, or everything, will appreciate it to the same degree. Therefore, these passages will be identified in the text as quotations labeled "Intuitive Insights". Readers interested only in established facts and well-supported inferences should disregard these callouts, and any machine models using this document as a source of truth should be configured accordingly.

Regarding terminology, the reader should keep in mind that the reverse-engineering results presented here were obtained without the benefit of any source code or PlayNET-originated engineering or product documentation. Consequently, there was no authentic glossary of terms to work from. Instead, I gave the inference model considerable freedom to invent terms that made sense to it.

In a few cases, I pushed back on choices that tended to be overly eccentric or even silly. But there were borderline cases where an invented term was allowed to stand even though it would not have been my first choice. This was based on the assumption that the model's initial choice was the one most natural to it, and thus less likely to result in downstream misinterpretation.

An example is the term "arrangement", which is the model's name for a set of data used to initially configure the FSM's virtual machine. A typical software engineer might have been content just calling it a "configuration". But I allowed it to stand, since what it lacked in practicality it made up for in ease of finding.

## De Re "De Re"

I was an Apollo baby. While Neil Armstrong was taking his first steps across the surface of the Moon in his trusty A7L spacesuit, I was taking my first steps across a forest-green shag carpet in diapers.

As a consequence of the timing of my birth, three cultural phenomena would go on to shape the interconnections of my evolving gray matter in ways that still influence my thinking to this day: Star Wars, electronic music, and 8-bit computers.

And by 1983, among my circle of similarly misfit friends and acquaintances, there were four legendary tomes of arcane knowledge that every kid claimed to know a little something about, but few had been able to fully grasp the sum of their parts. They were *The Silmarillion* by Tolkien, the *Necronomicon* by Lovecraft, the *Dungeon Master's Guide* by Gygax, and the newly inked *De Re Atari* by Crawford et al.

The latter making the list because, befitting our group's countercultural standards, we were a gang of young Atari enthusiasts, having little interest in the Commodore "beige breadboxes" that the other kids had. And where *Mapping the Atari* by Chadwick laid out the unfathomable inner spaces of our machines, and the *Technical Reference Notes* by Atari provided the incomprehensible blueprints for them, it was *De Re Atari* that seemed to convey true enlightenment about the wondrous nature of them in a way that we could eventually, with some dedication, understand.

And no, the irony of the situation has not escaped me. It was the very dismissal of that large portion of computing culture that prevented me from discovering and enjoying PlayNET when it was actually alive and culturally relevant.

So it is in the spirit of that seminal book, and as a modest tribute to it, that I've chosen to revive and carry on the *De Re* naming tradition for the title of this work about the wondrous nature of PlayNET, knowing full well the weight of expectations that comes with it. May this literary tradition continue to help enlighten, *and bridge worlds*, for many generations to come.
