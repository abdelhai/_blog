---
title: "Why would you want more than machine language?"
date: 2020-06-07T11:54:33+02:00
draft: false
---

***(a short history of the birth of assembly)***


The use of assembly language and an assembler was an idea that evolved at the dawn of early digital *stored program computers*, in the decade following the Second World War. [Richard Hamming](https://en.wikipedia.org/wiki/Richard_Hamming) reports an estimate that the use of assembly language represented a 2x improvement on programmer productivity [[p. 31]](#hamming).

Nonetheless, it took awhile to catch on amongst experienced programmers, many of whom continued to program in the earlier *machine code*.  Hamming reports of these programmers dismissing assembly as 'sissy stuff' and there are legends of towering figures in computing being quite dismissive of it. This piece is a short historical survey of the early development behind initial assembly languages and samples some of these early reactions to it.

## Programming Before Assembly

At the dawn of assembly, the model of computing which was taking shape was that of the *stored program computer*. The ideas for the *stored program computer* sprang out of the [ENIAC Project](https://en.wikipedia.org/wiki/ENIAC) and its shortcomings.  One of the early shortcomings of the ENIAC was that it required significant hardware modifications each time a new program was run [[p.75]](#campbell).  [John von Neumann](https://en.wikipedia.org/wiki/John_von_Neumann) did consulting work on this project and outlined the ideas for improvement from this collaboration with the ENIAC team in the [initial report on the EDVAC](http://web.eah-jena.de/~kleine/history/machines/VonNeumann-1stDraftReportEDVAC.pdf), though there are questions about who should get credit for the ideas [[p. 24]](#hamming)[[p. 77]](#campbell).

One of the most important of these ideas was to organize the computer such that the program being executed is itself also stored in the computer's memory--this way, the computer itself did not need to be tediously reconfigured for each new program [[p. 75-76]](#campbell).

## What is Assembly?

Stored program computers understand and can execute *machine code*, code expressed in numerical form (many of the earliest computers in pure binary). Below is a very short snippet of a more modern version of such machine code:

`6689C3`

From a human's perspective, it's not very intuitive, and if you are not incredibly immersed in a specific set of machine code, it is  *meaningless* (it is also not meant to be the hex code for a [seaish looking blue](https://www.color-hex.com/color/6689c3)). 

In contrast, the below code is something a human with knowledge of English can get some type of idea about, even with little background.

`mov bx, ax`

The second snippet is assembly code, which is read by an *assembler* and translated into the first snippet for execution by a computer.

In *Assemblers and Loaders*, an assembler is described as [[p. 1]](#saloman):
> ... a translator that translates source instructions (in symbolic language) into target instructions (in machine language), on a one to one basis.


For early *stored program computers*, humans had to *first do this translation* themselves *before* they could execute the programs. The idea behind assembly was to *use the machine itself* to translate between a more programmer friendly notation and the code the machine could understand, because computers are great at manipulating symbols at a high speed and precision [[p. 168]](#campbell). [Richard Hamming](https://en.wikipedia.org/wiki/Richard_Hamming) reports an estimate that the use of assembly language represented a 2x improvement on programmer productivity [[p. 31]](#hamming).



## The Development of Assembly on Early Stored Program Computers

Inspired by von Neumann and the ideas in the EDVAC report, two British research camps started working on stored program computers at the end of the 1940s, leading to some of the earliest assemblers. 

[Andrew Booth](https://en.wikipedia.org/wiki/Andrew_Donald_Booth) led the ARC project out of Birbeck College, with [Kathleen Booth](https://en.wikipedia.org/wiki/Kathleen_Booth) (born as Britten) as an assistant — Kathleen is credited with creating an assembler for the 'ARC2'. Meanwhile, [Maurice Wilkes](https://en.wikipedia.org/wiki/Maurice_Wilkes) led the EDSAC project out of Cambridge with David Wheeler as an assistant — Wheeler is credited with creating the EDSAC's assembler [[p. 81]](#campbell). 

The tedium of translating human symbolic understandings of programs into machine-readable programs led to the early assemblers.

### The ARC Project & Kathleen Booth

Kathleen was working with Andrew on the ARC, a specialized computer for calculating Fourier synteses 12-24x faster than a research student could do using traditional methods [[p. 102]](https://www.ams.org/journals/mcom/1954-08-046/S0025-5718-54-99336-9/S0025-5718-54-99336-9.pdf). After visiting von Neumann in Princeton, they constructed the ARC2, which was a ['stored program computer'](https://www.i-programmer.info/history/people/1253-andrew-booth.html). 

She and her husband released a few publications about this machine in 1947, one of which is [*General Considerations in the Design of an All Purpose Electronic Digital Computer*](http://www.mt-archive.info/Booth-1947.pdf). The second, *Coding For A.R.C*, is believed to contain her [assembly language for the ARC2](https://hackaday.com/2018/08/21/kathleen-booth-assembling-early-computers-while-inventing-assembly/). There is little documented about this second report and, as far as we know, there is no digital copy of *Coding For A.R.C.*. Birbeck College only goes as far as to say she ['developed a very early assembly language'](https://www.dcs.bbk.ac.uk/about/history/).

### EDSAC & David Wheeler

The story of David Wheeler and the EDSAC is more widely documented.

As it goes, Maurice Wilkes led the EDSAC project (after visiting the United States and learning about the EDVAC) to launch a stored program computer at Cambridge University. The EDSAC became operational in May 1949 [[85]](#campbell). During this project Wilkes noticed that 'assembling' *is something computers are well equipped to do;* he put David Wheeler on this job for a doctorate project [[168-169]](#campbell). 

The result of this project was the *Initial Orders* program, completed in May 1949, which would translate more human friendly punched codes into binary, and was loaded into memory as a ['bootstrap program'](https://www.cl.cam.ac.uk/~mr10/Edsac/edsacposter.pdf).

Wikipedia cites the earlier source crediting Kathleen Booth as the inventor of assembly. Her *Coding for A.R.C.* piece was released in 1947, while Wheeler's Initial orders came online later in 1949. He is credited by the IEEE Computer Society as having created the first ['assembly language'](https://www.computer.org/profiles/david-wheeler). As far as we are aware, there is no digital copy of *Coding for A.R.C.*, and it is something we would love to see come online.

### IBMs Early Commercial Assemblers

In the early 1950s, IBM's first commercial computers also had assemblers running. These assemblers took another step forwards from the early academics who developed assemblers. The earlier assemblers used more human friendly symbols for 'operations', but computer addresses were still fixed in a program's code. This had the downside that programmers who encountered and corrected bugs needed to re-assign a chain of subsequent addresses by hand or use an alternate method and end up with 'spaghetti code' [[p. 25]](#hamming).

With IBM's assemblers, *symbolic* addresses were introduced, where the assembler did the work of address assignment for the programmer [[p. 116]]((https://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=4640454)). Both IBM's first commercial scientific ([the 701](https://en.wikipedia.org/wiki/IBM_701)) and business ([the 650](https://en.wikipedia.org/wiki/IBM_650)) computers had assembly languages with symbolic addresses. For the 701, Nathaniel Rochester developed the [*Symbolic Assembly Program (SAP) in 1953](https://ia600101.us.archive.org/25/items/symbolic-programming/Image111817152723.facing_text.pdf). For the 650, Stan Poley [developed SOAP in 1955](http://www.columbia.edu/cu/computinghistory/650.html), which improved the performance of programs by the intelligent assignment of addresses [[p. 16]](#saloman).


## Early Responses to Assembly

Though most of the following perspectives are largely anecdotal, they suggest there is evidence that using *assembly language*, or anything higher than machine code, took time to catch on among practicing programmers and faced tremendous skepticism, even from earlier figures of our story, like Von Neumann (credit to [this presentation](http://worrydream.com/dbx/) for the sources).

[John Lee](http://ei.cs.vt.edu/~history/VonNeumann.html) reports anecdotes from von Neumann himself questioning the use of anything higher than machine code :

> In the 1950's von Neumann ... was confronted with the FORTRAN concept; John Backus remembered von Neumann being unimpressed and that he asked "why would you want more than machine language?" ...

> ... Donald Gillies, one of von Neumann's students at Princeton, and later a faculty member at the University of Illinois, recalled in the mid-1970's that the graduates students were being "used" to hand assemble programs into binary for their early machine (probably the IAS machine). He took time out to build an assembler, but when von Neumann found out about he was very angry, saying (paraphrased), "It is a waste of a valuable scientific computing instrument to use it to do clerical work."

Richard Hamming, writing about programming on the IBM 701 (the same machine Rochester developed the assembler for) [[p. 25]](#hamming):

> I once spent a full year, with the help of a lady programmer from Bell Telephone Libraries, on one big problem coding in absolute binary for the IBM 701, which used all the 32K registers then available. After that experience I vowed never again would I ask anyone to do such labor. Having heard about a symbolic system from Poughkeepsie, IBM, I ask[ed] her to send for it and to use it on the next problem...As I expected, she reported it was much easier...

> So we told everyone about the new method, meaning about 100 people ...

> ... To my knowledge only one person —yes, only one—of all 100 showed any interest!



He also writes (most probably about [Rochester's system for the 701](#ibms-early-commercial-assemblers)) [[p. 26]](#hamming):

> Finally a more complete, and more useful, Symbolic Assembly Program (SAP) was devised after more years than you are apt to believe ... most programmers continued their heroic absolute binary programming. At the time SAP first appeared I would guess about 1% of the older programmers were interested in it—using SAP was "sissy stuff" and a real programmer would not stoop to wasting machine capacity to do the assembly.

As Rochester developed SAP out of Poughkeepsie for IBM, it raises the question if Hamming's two anecdotes were referring to one and the same event, and the earlier 'symbolic system' was also Rochester's assembler. 

Nonetheless, Hamming estimates an initial interest level of about 1% amongst experienced programmers, despite his report of an estimated 2x productivity improvement.

## Conclusion

The anecdotal evidence suggests assembly language was far from an overnight success, despite the advantages it offered. Nonetheless zooming forwards, it's clear that the idea behind assembly took hold. At a more abstract level, the idea of '*buffering the user from the machine itself*', most likely using the computers themselves to do so, has carried forwards with many new layers added on after assembly; Hamming cites a report that assembly and technologies after have boosted programmer productivity by a factor of 90 over a 30 year period [[p. 26, 31]](#hamming).


## Continued

If you read this and have better knowledge than we do, we would love to see:

1. A copy of *'Programming for A.R.C.'* online. 
2. More studies (esp. quantitative) or anecdotes on the adoption of assembly language.
3. Your story or opinion on why programmers initially kept programming in machine code.
4. Reports on the increase in programmer productivity since the internet.

Please share with us!

## References Cited Multiple Times

<a id="campbell"></a>

* Campbell-Kelly, Martin, William Aspray, Nathan Ensmenger and Jeffrey Yost. 2014. *Computer: A History of the Information Machine.*  Westview Press.

<a id="hamming"></a>

* Hamming, Richard. 2005. *The Art and Science of Doing Engineering: Learning to Learn*  Gordon and Breach Science Publishers.

<a id="salomon"></a>

* Salomon, David. 1993. *Assemblers and Loaders*. Ellis Horwood Ltd. http://www.davidsalomon.name/assem.advertis/asl.pdf