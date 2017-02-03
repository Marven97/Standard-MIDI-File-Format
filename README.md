
<HTML>
<HEAD>
<META NAME="description" CONTENT="MIDI File Format Spec. 1.1+">
<META NAME="keywords" CONTENT="MIDI, file, format, specification, updated">
<TITLE>Standard MIDI file format, updated</TITLE>
</HEAD>
<BODY BGCOLOR="#FFE4B5">
<CENTER><H1>Standard MIDI-File Format</H1></CENTER>
<CENTER><H2>Abstract.</H2>
A detailed Specification of the Standard MIDI file format
<H1><A NAME="CONTENTS">Table of Contents</A></H1>
</CENTER>
<P ALIGN=LEFT><UL>
<LI><A HREF="#BM0_">0 - Introduction</A>
<LI><A HREF="#BM1_">1 - Sequences, Tracks, Chunks: File Block Structure</A>
<UL>
<LI><A HREF="#BM1_1">1.1 - Variable Length Quantity</A>
<LI><A HREF="#BM1_2">1.2 - Files</A>
<LI><A HREF="#BM1_3">1.3 - Chunks</A>
<LI><A HREF="#BM1_4">1.4 - Chunk Types</A>
</UL>
<LI><A HREF="#BM2_">2 - Chunk Descriptions</A>
<UL>
<LI><A HREF="#BM2_1">2.1 - Header Chunks</A>
<LI><A HREF="#BM2_2">2.2 - MIDI File Formats 0,1 and 2</A>
<LI><A HREF="#BM2_3">2.3 - Track Chunks</A>
</UL>
<LI><A HREF="#BM3_">3 - Meta-Events</A>
<UL>
<LI><A HREF="#BM3_1">3.1 - Meta Event Definitions</A>
</UL>
<LI><A HREF="#BMA1_">Appendix 1</A> - MIDI Messages
<UL>
<LI><A HREF="#BMA1_1">Appendix 1.1</A> - Table of Major MIDI Messages
<LI><A HREF="#BMA1_2">Appendix 1.2</A> - Table of MIDI Controller Messages (Data Bytes)
<LI><A HREF="#BMA1_3">Appendix 1.3</A> - Table of MIDI Note Numbers
<LI><A HREF="#BMA1_4">Appendix 1.4</A> - General MIDI Instrument Patch Map
<LI><A HREF="#BMA1_5">Appendix 1.5</A> - General MIDI Percussion Key Map
</UL>
<LI><A HREF="#BMA2_">Appendix 2</A> - Program Fragments and Example MIDI Files
</UL>
<P>
<H3>Acknowledgement:</H3>
This document was originally distributed in text format by The International
MIDI Association. I have updated it and added new Appendices.<BR>
&copy; Copyright 1999 David Back.<BR>
EMail: <A HREF="mailto:david@csw2.co.uk">david@csw2.co.uk</A><BR>
Web: <A HREF="http://www.csw2.co.uk/">http://www.csw2.co.uk</A><BR>
This document may be freely copied in whole or in part provided the copy contains this
Acknowledgement.
<P>
<H2><A NAME="BM0_">0 - Introduction</A></H2>
<P>This document details the structure of MIDI Files. The
purpose of MIDI Files is to provide a way of interchanging
time-stamped MIDI data between different programs on the same or
different computers. One of the primary design goals is compact
representation, which makes it very appropriate for disk-based
file format, but which might make it inappropriate for storing in
memory for quick access by a sequencer program.

<P>MIDI Files contain one or more MIDI streams, with time
information for each event. Song, sequence, and track structures,
tempo and time signature information, are all supported. Track
names and other descriptive information may be stored with the
MIDI data. This format supports multiple tracks and multiple
sequences so that if the user of a program which supports
multiple tracks intends to move a file to another one, this
format can allow that to happen.

<P>The specification defines the 8-bit binary data stream used in the
file. The data can be stored in a binary file, nibbilized,
7-bit-ized for efficient MIDI transmission, converted to Hex
ASCII, or translated symbolically to a printable text file. This
spec addresses what's in the 8-bit stream. It does not address
how a MIDI File will be transmitted over MIDI.
<P><A HREF="#CONTENTS">Back to contents</A></P>
<H2><A NAME="BM1_">1 - Sequences, Tracks, Chunks: File Block Structure</A></H2>

<P>In this document, bit 0 means the least significant bit of a
byte, and bit 7 is the most significant.

<H3><A NAME="BM1_1">1.1 - Variable Length Quantity</A></H3>
<P>Some numbers in MIDI Files are represented in a form called
VARIABLE-LENGTH QUANTITY. These numbers are represented 7 bits
per byte, most significant bits first. All bytes except the last
have bit 7 set, and the last byte has bit 7 clear. If the number
is between 0 and 127, it is thus represented exactly as one byte.

<P><B>Some examples of numbers represented as variable-length quantities:</B>
<P>
<TABLE BORDER=1>
<TR><TD>00000000 </TD><TD>00</TD></TR>
<TR><TD>00000040 </TD><TD>40</TD></TR>
<TR><TD>0000007F </TD><TD>7F</TD></TR>
<TR><TD>00000080 </TD><TD>81 00</TD></TR>
<TR><TD>00002000 </TD><TD>C0 00</TD></TR>
<TR><TD>00003FFF </TD><TD>FF 7F</TD></TR>
<TR><TD>00004000 </TD><TD>81 80 00</TD></TR>
<TR><TD>00100000 </TD><TD>C0 80 00</TD></TR>
<TR><TD>001FFFFF </TD><TD>FF FF 7F</TD></TR>
<TR><TD>00200000 </TD><TD>81 80 80 00</TD></TR>
<TR><TD>08000000 </TD><TD>C0 80 80 00</TD></TR>
<TR><TD>0FFFFFFF </TD><TD>FF FF FF 7F</TD></TR>
</TABLE>
<P>The largest number which is allowed is 0FFFFFFF so that the
variable-length representations must fit in 32 bits in a routine
to write variable-length numbers. Theoretically, larger numbers
are possible, but 2 x 10<SUP>8</SUP> 96ths of a beat at a fast tempo of 500
beats per minute is four days, long enough for any delta-time!

<H3><A NAME="BM1_2">1.2 - Files</A></H3>
To any file system, a MIDI File is simply a series of
8-bit bytes. On the Macintosh, this byte stream is stored in the
data fork of a file (with file type 'MIDI'), or on the Clipboard
(with data type 'MIDI'). Most other computers store 8-bit byte
streams in files.

<H3><A NAME="BM1_3">1.3 - Chunks</A></H3>
MIDI Files are made up of -chunks-. Each chunk has a
4-character type and a 32-bit length, which is the number of
bytes in the chunk. This structure allows future chunk types to
be designed which may be easily be ignored if encountered by a
program written before the chunk type is introduced. Your
programs should EXPECT alien chunks and treat them as if they
weren't there.

<P>Each chunk begins with a 4-character ASCII type. It is
followed by a 32-bit length, most significant byte first (a
length of 6 is stored as 00 00 00 06). This length refers to the
number of bytes of data which follow: the eight bytes of type and
length are not included. Therefore, a chunk with a length of 6
would actually occupy 14 bytes in the disk file.

<P>This chunk architecture is similar to that used by Electronic
Arts' IFF format, and the chunks described herein could easily be
placed in an IFF file. The MIDI File itself is not an IFF file:
it contains no nested chunks, and chunks are not constrained to
be an even number of bytes long. Converting it to an IFF file is
as easy as padding odd length chunks, and sticking the whole
thing inside a FORM chunk.

<H3><A NAME="BM1_4">1.4 - Chunk Types</A></H3>
<P>MIDI Files contain two types of chunks: header chunks and
track chunks. A -header- chunk provides a minimal amount of
information pertaining to the entire MIDI file. A -track- chunk
contains a sequential stream of MIDI data which may contain
information for up to 16 MIDI channels. The concepts of multiple
tracks, multiple MIDI outputs, patterns, sequences, and songs may
all be implemented using several track chunks.

<P>A MIDI File always starts with a header chunk, and is followed
by one or more track chunks.

<P>MThd &lt;length of header data&gt;<BR>
 &lt;header data&gt;<BR>
 MTrk &lt;length of track data&gt;<BR>
 &lt;track data&gt;<BR>
 MTrk &lt;length of track data&gt;<BR>
 &lt;track data&gt;<BR>
 . . .
<P><A HREF="#CONTENTS">Back to contents</A></P>
<H2><A NAME="BM2_">2 - Chunk Descriptions</A></H2>

<H3><A NAME="BM2_1">2.1 - Header Chunks</A></H3>
The header chunk at the beginning of the file
specifies some basic information about the data in the file.
Here's the syntax of the complete chunk:

<P>&lt;Header Chunk&gt; = &lt;chunk
type&gt;&lt;length&gt;&lt;format&gt;&lt;ntrks&gt;&lt;division&gt;

<P>As described above, &lt;chunk type&gt; is the four ASCII
characters 'MThd'; &lt;length&gt; is a 32-bit representation of
the number 6 (high byte first).

<P>The data section contains three 16-bit words, stored
most-significant byte first.

<P>The first word, &lt;format&gt;, specifies the overall
organisation of the file. Only three values of &lt;format&gt; are
specified:

<P>0-the file contains a single multi-channel track<BR>
1-the file contains one or more simultaneous tracks (or MIDI outputs) of a sequence<BR>
2-the file contains one or more sequentially independent single-track patterns

<P>More information about these formats is provided below.

<P>The next word, &lt;ntrks&gt;, is the number of track chunks in
the file. It will always be 1 for a format 0 file.

<P>The third word, &lt;division&gt;, specifies the meaning of the
delta-times. It has two formats, one for metrical time, and one
for time-code-based time:
<TABLE BORDER=1>
<TR><TH>bit 15 </TH><TH>bits 14 thru 8</TH><TH>bits 7 thru 0</TH></TR>
<TR><TD>0 </TD><TD COLSPAN=2>ticks per quarter-note</TD></TR>
<TR><TD>1 </TD><TD>negative SMPTE format </TD><TD>ticks per frame</TD></TR>
</TABLE>
<P>If bit 15 of &lt;division&gt; is zero, the bits 14 thru 0
represent the number of delta time &quot;ticks&quot; which make
up a quarter-note. For instance, if division is 96, then a time
interval of an eighth-note between two events in the file would
be 48.

<P>If bit 15 of &lt;division&gt; is a one, delta times in a file
correspond to subdivisions of a second, in a way consistent with
SMPTE and MIDI Time Code. Bits 14 thru 8 contain one of the four
values -24, -25, -29, or -30, corresponding to the four standard
SMPTE and MIDI Time Code formats (-29 corresponds to 30 drop
frame), and represents the number of frames per second. These
negative numbers are stored in two's compliment form. The second
byte (stored positive) is the resolution within a frame: typical
values may be 4 (MIDI Time Code resolution), 8, 10, 80 (bit
resolution), or 100. This stream allows exact specifications of
time-code-based tracks, but also allows millisecond-based tracks
by specifying 25 frames/sec and a resolution of 40 units per
frame. If the events in a file are stored with a bit resolution
of thirty-frame time code, the division word would be E250 hex.

<H3><A NAME="BM2_2">2.2 - MIDI File Formats 0,1 and 2</A></H3>
A Format 0 file has a header chunk
followed by one track chunk. It is the most interchangeable
representation of data. It is very useful for a simple
single-track player in a program which needs to make synthesisers
make sounds, but which is primarily concerned with something
else such as mixers or sound effect boxes. It is very desirable
to be able to produce such a format, even if your program is
track-based, in order to work with these simple programs.

<P>A Format 1 or 2 file has a header chunk followed by one or
more track chunks. programs which support several simultaneous
tracks should be able to save and read data in format 1, a
vertically one dimensional form, that is, as a collection of
tracks. Programs which support several independent patterns
should be able to save and read data in format 2, a horizontally
one dimensional form. Providing these minimum capabilities will
ensure maximum interchangeability.

<P>In a MIDI system with a computer and a SMPTE synchroniser
which uses Song Pointer and Timing Clock, tempo maps (which
describe the tempo throughout the track, and may also include
time signature information, so that the bar number may be
derived) are generally created on the computer. To use them with
the synchroniser, it is necessary to transfer them from the
computer. To make it easy for the synchroniser to extract this
data from a MIDI File, tempo information should always be stored
in the first MTrk chunk. For a format 0 file, the tempo will be
scattered through the track and the tempo map reader should
ignore the intervening events; for a format 1 file, the tempo map
must be stored as the first track. It is polite to a tempo map
reader to offer your user the ability to make a format 0 file
with just the tempo, unless you can use format 1.

<P>All MIDI Files should specify tempo and time signature. If
they don't, the time signature is assumed to be 4/4, and the
tempo 120 beats per minute. In format 0, these meta-events should
occur at least at the beginning of the single multi-channel
track. In format 1, these meta-events should be contained in the
first track. In format 2, each of the temporally independent
patterns should contain at least initial time signature and tempo
information.

<P>Format IDs to support other structures may be defined in
the future. A program encountering an unknown format ID may still
read other MTrk chunks it finds from the file, as format 1 or 2,
if its user can make sense of them and arrange them into some
other structure if appropriate. Also, more parameters may be
added to the MThd chunk in the future: it is important to read
and honour the length, even if it is longer than 6.

<H3><A NAME="BM2_3">2.3 - Track Chunks</A></H3>
The track chunks (type MTrk) are where actual
song data is stored. Each track chunk is simply a stream of MIDI
events (and non-MIDI events), preceded by delta-time values. The
format for Track Chunks (described below) is exactly the same for
all three formats (0, 1, and 2: see &quot;Header Chunk&quot;
above) of MIDI Files.

<P>Here is the syntax of an MTrk chunk (the + means &quot;one or
more&quot;: at least one MTrk event must be present):

<P>&lt;Track Chunk&gt; = &lt;chunk type&gt;&lt;length&gt;&lt;MTrk
event&gt;+

<P>The syntax of an MTrk event is very simple:

<P>&lt;MTrk event&gt; = &lt;delta-time&gt;&lt;event&gt;

<P>&lt;delta-time&gt; is stored as a variable-length quantity. It
represents the amount of time before the following event. If the
first event in a track occurs at the very beginning of a track,
or if two events occur simultaneously, a delta-time of zero is
used. Delta-times are always present. (Not storing delta-times of
0 requires at least two bytes for any other value, and most
delta-times aren't zero.) Delta-time is in some fraction of a
beat (or a second, for recording a track with SMPTE times), as
specified in the header chunk.

<P>&lt;event&gt; = &lt;MIDI event&gt; | &lt;sysex event&gt; |
&lt;meta-event&gt;

<P>&lt;MIDI event&gt; is any MIDI channel message
 <A HREF="#BMA1_">See Appendix 1 - MIDI Messages.</A> Running status
is used: status bytes of MIDI channel messages may be omitted if
the preceding event is a MIDI channel message with the same
status. The first event in each MTrk chunk must specify status.
Delta-time is not considered an event itself: it is an integral
part of the syntax for an MTrk event. Notice that running status
occurs across delta-times.

<P>&lt;sysex event&gt; is used to specify a MIDI system exclusive
message, either as one unit or in packets, or as an
&quot;escape&quot; to specify any arbitrary bytes to be
transmitted. <A HREF="#BMA1_">See Appendix 1 - MIDI Messages.</A> A normal
complete system exclusive message is stored in a MIDI File in this way:

<P>F0 &lt;length&gt; &lt;bytes to be transmitted after F0&gt;

<P>The length is stored as a variable-length quantity. It
specifies the number of bytes which follow it, not including the
F0 or the length itself. For instance, the transmitted message F0
43 12 00 07 F7 would be stored in a MIDI File as F0 05 43 12 00
07 F7. It is required to include the F7 at the end so that the
reader of the MIDI File knows that it has read the entire
message.

<P>Another form of sysex event is provided which does not imply
that an F0 should be transmitted. This may be used as an
&quot;escape&quot; to provide for the transmission of things
which would not otherwise be legal, including system realtime
messages, song pointer or select, MIDI Time Code, etc. This uses
the F7 code:

<P>F7 &lt;length&gt; &lt;all bytes to be transmitted&gt;

<P>Unfortunately, some synthesiser manufacturers specify that
their system exclusive messages are to be transmitted as little
packets. Each packet is only part of an entire syntactical system
exclusive message, but the times they are transmitted are
important. Examples of this are the bytes sent in a CZ patch
dump, or the FB-01's &quot;system exclusive mode&quot; in which
microtonal data can be transmitted. The F0 and F7 sysex events
may be used together to break up syntactically complete system
exclusive messages into timed packets.

<P>An F0 sysex event is used for the first packet in a series --
it is a message in which the F0 should be transmitted. An F7
sysex event is used for the remainder of the packets, which do
not begin with F0. (Of course, the F7 is not considered part of
the system exclusive message).

<P>A syntactic system exclusive message must always end with an
F7, even if the real-life device didn't send one, so that you
know when you've reached the end of an entire sysex message
without looking ahead to the next event in the MIDI File. If it's
stored in one complete F0 sysex event, the last byte must be an
F7. There also must not be any transmittable MIDI events in
between the packets of a multi-packet system exclusive message.
This principle is illustrated in the paragraph below.

<P>Here is a MIDI File of a multi-packet system exclusive
message: suppose the bytes F0 43 12 00 were to be sent, followed
by a 200-tick delay, followed by the bytes 43 12 00 43 12 00,
followed by a 100-tick delay, followed by the bytes 43 12 00 F7,
this would be in the MIDI File:
<TABLE BORDER=1>
<TR><TD>F0 03 43 12 00</TD><TD>&nbsp;</TD></TR>
<TR><TD>81 48 </TD><TD>200-tick delta time</TD></TR>
<TR><TD>F7 06 43 12 00 43 12 00</TD><TD>&nbsp;</TD></TR>
<TR><TD>64 </TD><TD>100-tick delta time</TD></TR>
<TR><TD>F7 04 43 12 00 F7</TD><TD>&nbsp;</TD></TR>
</TABLE>
<P>When reading a MIDI File, and an F7 sysex event is encountered
without a preceding F0 sysex event to start a multi-packet system
exclusive message sequence, it should be presumed that the F7
event is being used as an &quot;escape&quot;. In this case, it is
not necessary that it end with an F7, unless it is desired that
the F7 be transmitted.

<P>&lt;meta-event&gt; specifies non-MIDI information useful to
this format or to sequencers, with this syntax:

<P>FF &lt;type&gt; &lt;length&gt; &lt;bytes&gt;

<P>All meta-events begin with FF, then have an event type byte
(which is always less than 128), and then have the length of the
data stored as a variable-length quantity, and then the data
itself. If there is no data, the length is 0. As with chunks,
future meta-events may be designed which may not be known to
existing programs, so programs must properly ignore meta-events
which they do not recognise, and indeed should expect to see
them. Programs must never ignore the length of a meta-event which
they do not recognise, and they shouldn't be surprised if it's
bigger than expected. If so, they must ignore everything past
what they know about. However, they must not add anything of
their own to the end of the meta- event. Sysex events and meta
events cancel any running status which was in effect. Running
status does not apply to and may not be used for these messages.
<P><A HREF="#CONTENTS">Back to contents</A></P>
<H2><A NAME="BM3_">3 - Meta-Events</A></H2>

<P>A few meta-events are defined herein. It is not required for
every program to support every meta-event.

<P>In the syntax descriptions for each of the meta-events a set
of conventions is used to describe parameters of the events. The
FF which begins each event, the type of each event, and the
lengths of events which do not have a variable amount of data are
given directly in hexadecimal. A notation such as dd or se, which
consists of two lower-case letters, mnemonically represents an
8-bit value. Four identical lower-case letters such as wwww
mnemonically refer to a 16-bit value, stored
most-significant-byte first. Six identical lower-case letters
such as tttttt refer to a 24-bit value, stored
most-significant-byte first. The notation len refers to the length
portion of the meta-event syntax, that is, a number, stored as a
variable- length quantity, which specifies how many bytes
(possibly text) data were just specified by the length.

<P>In general, meta-events in a track which occur at the same
time may occur in any order. If a copyright event is used, it
should be placed as early as possible in the file, so it will be
noticed easily. Sequence Number and Sequence/Track Name events,
if present, must appear at time 0. An end-of- track event must
occur as the last event in the track.

<H3><A NAME="BM3_1">3.1 - Meta-Event Definitions</A></H3>
<P><B>FF 00 02 Sequence Number</B><BR>
This optional event, which must occur
at the beginning of a track, before any nonzero delta-times, and
before any transmittable MIDI events, specifies the number of a
sequence. In a format 2 MIDI File, it is used to identify each
&quot;pattern&quot; so that a &quot;song&quot; sequence using the
Cue message can refer to the patterns. If the ID numbers are
omitted, the sequences' locations in order in the file are used
as defaults. In a format 0 or 1 MIDI File, which only contain one
sequence, this number should be contained in the first (or only)
track. If transfer of several multitrack sequences is required,
this must be done as a group of format 1 files, each with a
different sequence number.

<P><B>FF 01 len text Text Event</B><BR>
Any amount of text describing
anything. It is a good idea to put a text event right at the
beginning of a track, with the name of the track, a description
of its intended orchestration, and any other information which
the user wants to put there. Text events may also occur at other
times in a track, to be used as lyrics, or descriptions of cue
points. The text in this event should be printable ASCII
characters for maximum interchange. However, other character
codes using the high-order bit may be used for interchange of
files between different programs on the same computer which
supports an extended character set. Programs on a computer which
does not support non-ASCII characters should ignore those
characters.

<P>Meta-event types 01 through 0F are reserved for various types
of text events, each of which meets the specification of text
events (above) but is used for a different purpose:

<P><B>FF 02 len text Copyright Notice</B><BR>
Contains a copyright notice as
printable ASCII text. The notice should contain the characters
(C), the year of the copyright, and the owner of the copyright.
If several pieces of music are in the same MIDI File, all of the
copyright notices should be placed together in this event so that
it will be at the beginning of the file. This event should be the
first event in the track chunk, at time 0.

<P><B>FF 03 len text Sequence/Track Name</B><BR>
If in a format 0 track, or
the first track in a format 1 file, the name of the sequence.
Otherwise, the name of the track.

<P><B>FF 04 len text Instrument Name</B><BR>
A description of the type of
instrumentation to be used in that track. May be used with the
MIDI Prefix meta-event to specify which MIDI channel the
description applies to, or the channel may be specified as text
in the event itself.

<P><B>FF 05 len text Lyric</B><BR>
A lyric to be sung. Generally, each
syllable will be a separate lyric event which begins at the
event's time.

<P><B>FF 06 len text Marker</B><BR>
Normally in a format 0 track, or the
first track in a format 1 file. The name of that point in the
sequence, such as a rehearsal letter or section name (&quot;First
Verse&quot;, etc.)

<P><B>FF 07 len text Cue Point</B><BR>
A description of something happening
on a film or video screen or stage at that point in the musical
score (&quot;Car crashes into house&quot;, &quot;curtain
opens&quot;, &quot;she slaps his face&quot;, etc.)

<P><B>FF 20 01 cc MIDI Channel Prefix</B><BR>
The MIDI channel (0-15)
contained in this event may be used to associate a MIDI channel
with all events which follow, including System exclusive and
meta-events. This channel is &quot;effective&quot; until the next
normal MIDI event (which contains a channel) or the next MIDI
Channel Prefix meta-event. If MIDI channels refer to
&quot;tracks&quot;, this message may be put into a format 0 file,
keeping their non-MIDI data associated with a track. This
capability is also present in Yamaha's ESEQ file format.

<P><B>FF 2F 00 End of Track</B><BR>
This event is not optional. It is
included so that an exact ending point may be specified for the
track, so that an exact length is defined, which is necessary for tracks
which are looped or concatenated.

<P><B>FF 51 03 tttttt Set Tempo (in microseconds per MIDI quarter-note)</B><BR>
This event indicates a tempo change. Another way of
putting &quot;microseconds per quarter-note&quot; is &quot;24ths
of a microsecond per MIDI clock&quot;. Representing tempos as
time per beat instead of beat per time allows absolutely exact
long-term synchronisation with a time-based sync protocol such as
SMPTE time code or MIDI time code. The amount of accuracy
provided by this tempo resolution allows a four-minute piece at
120 beats per minute to be accurate within 500 usec at the end of
the piece. Ideally, these events should only occur where MIDI
clocks would be located -- this convention is intended to
guarantee, or at least increase the likelihood, of compatibility
with other synchronisation devices so that a time signature/tempo
map stored in this format may easily be transferred to another
device.

<P><B>FF 54 05 hr mn se fr ff SMPTE Offset</B><BR>
This event, if present,
designates the SMPTE time at which the track chunk is supposed to
start. It should be present at the beginning of the track, that
is, before any nonzero delta-times, and before any transmittable
MIDI events. the hour must be encoded with the SMPTE format, just
as it is in MIDI Time Code. In a format 1 file, the SMPTE Offset
must be stored with the tempo map, and has no meaning in any of
the other tracks. The ff field contains fractional frames, in
100ths of a frame, even in SMPTE-based tracks which specify a
different frame subdivision for delta-times.

<P><B>FF 58 04 nn dd cc bb Time Signature</B><BR>
The time signature is
expressed as four numbers. nn and dd represent the numerator and
denominator of the time signature as it would be notated. The
denominator is a negative power of two: 2 represents a
quarter-note, 3 represents an eighth-note, etc. The cc parameter
expresses the number of MIDI clocks in a metronome click. The bb
parameter expresses the number of notated 32nd-notes in a MIDI
quarter-note (24 MIDI clocks). This was added because there are
already multiple programs which allow a user to specify that what
MIDI thinks of as a quarter-note (24 clocks) is to be notated as,
or related to in terms of, something else.

<P>Therefore, the complete event for 6/8 time, where the
metronome clicks every three eighth-notes, but there are 24
clocks per quarter-note, 72 to the bar, would be (in hex):

<P>FF 58 04 06 03 24 08

<P>That is, 6/8 time (8 is 2 to the 3rd power, so this is 06 03),
36 MIDI clocks per dotted-quarter (24 hex!), and eight notated
32nd-notes per quarter-note.

<P><B>FF 59 02 sf mi Key Signature</B><BR>
sf = -7: 7 flats<BR>
sf = -1: 1 flat<BR>
sf = 0: key of C<BR>
sf = 1: 1 sharp<BR>
sf = 7: 7 sharps<BR>
<P>
mi = 0: major key<BR>
mi = 1: minor key<BR>

<P><B>FF 7F len data Sequencer Specific Meta-Event</B><BR>
Special requirements for particular sequencers may use this event type:
the first byte or bytes of data is a manufacturer ID (these are
one byte, or if the first byte is 00, three bytes). As with MIDI
System Exclusive, manufacturers who define something using this
meta-event should publish it so that others may be used by a
sequencer which elects to use this as its only file format;
sequencers with their established feature-specific formats should
probably stick to the standard features when using this format.
<P>
<A HREF="#BMA2_">See Appendix 2 - Program Fragments and Example MIDI Files</A> for an
example midi file.
<P><A HREF="#CONTENTS">Back to contents</A></P>
<HR>
<CENTER><H2><A NAME="BMA1_">Appendix 1 - MIDI Messages</A></H2></CENTER>
<P ALIGN=LEFT>
A MIDI message is made up of an eight-bit status byte which is generally followed
by one or two data bytes. There are a number of different types of MIDI messages.
At the highest level, MIDI messages are classified as being either Channel Messages
or System Messages. Channel messages are those which apply to a specific Channel, and
the Channel number is included in the status byte for these messages. System messages
are not Channel specific, and no Channel number is indicated in their status bytes. 
<P>
Channel Messages may be further classified as being either Channel Voice Messages,
or Mode Messages. Channel Voice Messages carry musical performance data, and these
messages comprise most of the traffic in a typical MIDI data stream. Channel Mode
messages affect the way a receiving instrument will respond to the Channel Voice
messages.
<P>
MIDI System Messages are classified as being System Common Messages, System Real Time
Messages, or System Exclusive Messages. System Common messages are intended for all
receivers in the system. System Real Time messages are used for synchronisation between
clock-based MIDI components. System Exclusive messages include a Manufacturer's
Identification (ID) code, and are used to transfer any number of data bytes in a format
specified by the referenced manufacturer.
<P>
<CENTER><H3><A NAME="BMA1_1">Appendix 1.1 - Table of Major MIDI Messages</A></H3></CENTER>
<P ALIGN=LEFT><TABLE BORDER=1>
<TR><TH COLSPAN=3>Channel Voice Messages</TH></TR>
<TR><TH>Status<BR>D7----D0<BR>nnnn is the MIDI channel no.</TH><TH>Data Byte(s)<BR>D7----D0</TH><TH>Description</TH></TR>
<TR><TD VALIGN=TOP>1000nnnn</TD><TD VALIGN=TOP>0kkkkkkk<BR>0vvvvvvv</TD><TD>Note Off event.<BR>
This message is sent when a note is released (ended).<BR>
(kkkkkkk) is the key (note) number.<BR>
(vvvvvvv) is the velocity.</TD></TR>
<TR><TD VALIGN=TOP>1001nnnn</TD><TD VALIGN=TOP>0kkkkkkk<BR>0vvvvvvv</TD><TD>Note On event.<BR>
This message is sent when a note is depressed (start).<BR>
(kkkkkkk) is the key (note) number.<BR>
(vvvvvvv) is the velocity.</TD></TR>
<TR><TD VALIGN=TOP>1010nnnn</TD><TD VALIGN=TOP>0kkkkkkk<BR>0vvvvvvv</TD><TD>Polyphonic Key Pressure (Aftertouch).<BR>
This message is most often sent by pressing down on the key after it &quot;bottoms out&quot;.<BR>
(kkkkkkk) is the key (note) number.<BR>
(vvvvvvv) is the pressure value.</TD></TR>
<TR><TD VALIGN=TOP>1011nnnn</TD><TD VALIGN=TOP>0ccccccc<BR>0vvvvvvv</TD><TD>Control Change.<BR>
This message is sent when a controller value changes.  Controllers include devices
such as pedals and levers. Certain controller numbers are reserved
for specific purposes. See Channel Mode Messages.<BR>
(ccccccc) is the controller number.<BR>
(vvvvvvv) is the new value.</TD></TR>
<TR><TD VALIGN=TOP>1100nnnn</TD><TD VALIGN=TOP>0ppppppp</TD><TD>Program Change.<BR>
This message sent when the patch number changes.<BR>
(ppppppp) is the new program number.</TD></TR>
<TR><TD VALIGN=TOP>1101nnnn</TD><TD VALIGN=TOP>0vvvvvvv</TD><TD>Channel Pressure (After-touch).<BR>
This message is most often sent by pressing down on the key after it &quot;bottoms out&quot;.
This message is different from polyphonic after-touch. Use this message to send the single greatest
pressure value (of all the current depressed keys).<BR>
(vvvvvvv) is the pressure value.</TD></TR>
<TR><TD VALIGN=TOP>1110nnnn</TD><TD VALIGN=TOP>0lllllll<BR>0mmmmmmm</TD><TD>Pitch Wheel Change.<BR>
This message is sent to indicate a change in the pitch wheel.  The pitch wheel is measured by a
fourteen bit value. Centre (no pitch change) is 2000H.  Sensitivity is a function of the transmitter.<BR>
(lllllll) are the least significant 7 bits.<BR>
(mmmmmmm) are the most significant 7 bits.</TD></TR>
<TR><TH COLSPAN=3>Channel Mode Messages  (See also Control Change, above)</TH></TR>
<TR><TD VALIGN=TOP>1011nnnn</TD><TD VALIGN=TOP>0ccccccc<BR>0vvvvvvv</TD><TD>Channel Mode Messages.<BR>
This the same code as the Control Change (above), but implements Mode control by using reserved
controller numbers.  The numbers are:<BR>
Local Control.<BR>
When Local Control is Off, all devices on a given channel will respond only to data received
over MIDI.  Played data, etc. will be ignored.  Local Control On restores the functions of the
normal controllers.<BR>
c = 122, v = 0: Local Control Off<BR>
c = 122, v = 127: Local Control On<BR>
<P>All Notes Off.<BR>
When an All Notes Off is received all oscillators will turn off.<BR>
c = 123, v = 0: All Notes Off<BR>
c = 124, v = 0: Omni Mode Off<BR>
c = 125, v = 0: Omni Mode On<BR>
c = 126, v = M: Mono Mode On (Poly Off) where M is the number of channels (Omni Off) or 0 (Omni On)<BR>
c = 127, v = 0: Poly Mode On (Mono Off)
(Note: These four messages also cause All Notes Off)</TD></TR>
<TR><TH COLSPAN=3>System Common Messages</TH></TR>
<TR><TD VALIGN=TOP>11110000</TD><TD VALIGN=TOP>0iiiiiii<BR>0ddddddd<BR>..<BR>..<BR>0ddddddd<BR>11110111</TD><TD VALIGN=TOP>System Exclusive.<BR>
This message makes up for all that MIDI doesn't support. (iiiiiii) is usually a seven-bit Manufacturer's I.D. code.
If the synthesiser recognises the I.D. code as its own, it will listen to the rest of
the message (ddddddd).  Otherwise, the message will be ignored.  System Exclusive
is used to send bulk dumps such as patch parameters and other non-spec data.
(Note: Real-Time messages ONLY may be interleaved with a System Exclusive.)
This message also is used for extensions called Universal Exclusive Messages.</TD></TR>
<TR><TD>11110001</TD><TD>&nbsp;</TD><TD>Undefined.</TD></TR>
<TR><TD VALIGN=TOP>11110010</TD><TD VALIGN=TOP>0lllllll<BR>0mmmmmmm</TD><TD>Song Position Pointer.<BR>
This is an internal 14 bit register that holds the number of MIDI beats (1 beat=
six MIDI clocks) since the start of the song.  l is the LSB, m the MSB.</TD></TR>
<TR><TD VALIGN=TOP>11110011</TD><TD VALIGN=TOP>0sssssss</TD><TD>Song Select.<BR>
The Song Select specifies which sequence or song is to be played.</TD></TR>
<TR><TD>11110100</TD><TD>&nbsp;</TD><TD>Undefined.</TD></TR>
<TR><TD>11110101</TD><TD>&nbsp;</TD><TD>Undefined.</TD></TR>
<TR><TD VALIGN=TOP>11110110</TD><TD VALIGN=TOP>&nbsp;</TD><TD>Tune Request.<BR>
Upon receiving a Tune Request, all analog synthesisers should tune their oscillators.</TD></TR>
<TR><TD VALIGN=TOP>11110111</TD><TD VALIGN=TOP>&nbsp;</TD><TD>End of Exclusive.<BR>
Used to terminate a System Exclusive dump (see above).</TD></TR>
<TR><TH COLSPAN=3>System Real-Time Messages</TH></TR>
<TR><TD VALIGN=TOP>11111000</TD><TD VALIGN=TOP>&nbsp;</TD><TD>Timing Clock.<BR>
Sent 24 times per quarter note when synchronisation is required.</TD></TR>
<TR><TD VALIGN=TOP>11111001</TD><TD VALIGN=TOP>&nbsp;</TD><TD>Undefined.</TD></TR>
<TR><TD VALIGN=TOP>11111010</TD><TD VALIGN=TOP>&nbsp;</TD><TD>Start.<BR>
Start the current sequence playing. (This message will be followed with Timing Clocks).</TD></TR>
<TR><TD VALIGN=TOP>11111011</TD><TD VALIGN=TOP>&nbsp;</TD><TD>Continue.<BR>
Continue at the point the sequence was Stopped.</TD></TR>
<TR><TD VALIGN=TOP>11111100</TD><TD VALIGN=TOP>&nbsp;</TD><TD>Stop.<BR>
Stop the current sequence.</TD></TR>
<TR><TD VALIGN=TOP>11111101</TD><TD VALIGN=TOP>&nbsp;</TD><TD>Undefined.<BR></TD></TR>
<TR><TD VALIGN=TOP>11111110</TD><TD VALIGN=TOP>&nbsp;</TD><TD>Active Sensing.<BR>
Use of this message is optional. When initially sent, the receiver will expect
to receive another Active Sensing message each 300ms (max), or it will be assume
that the connection has been terminated. At termination, the receiver will turn off
all voices and return to normal (non-active sensing) operation.</TD></TR>
<TR><TD VALIGN=TOP>11111111</TD><TD VALIGN=TOP>&nbsp;</TD><TD>Reset.<BR>
Reset all receivers in the system to power-up status. This should be used
sparingly, preferably under manual control. In particular, it should not
be sent on power-up.<BR>In a MIDI file this is used as an escape to introduce
&lt;meta events&gt;.</TD></TR>
</TABLE>
<P><A HREF="#CONTENTS">Back to contents</A></P>
<HR>
<CENTER><H3><A NAME="BMA1_2">Appendix 1.2 - Table of MIDI Controller Messages (Data Bytes)</A></H3></CENTER>
<P ALIGN=LEFT>
The following table lists the MIDI Controller messages in numerical (binary) order.<P>
<TABLE BORDER=2>
<TR><TH COLSPAN=3>2nd Byte Value</TH><TH>Function</TH><TH COLSPAN=2>3rd Byte</TH></TR>
<TR><TH>Binary</TH><TH>Hex</TH><TH>Dec</TH><TH>&nbsp;</TH><TH>Value</TH><TH>Use</TH></TR>
<TR><TD>00000000</TD><TD>00</TD><TD>0</TD><TD>Bank Select</TD><TD>0-127</TD><TD>MSB</TD></TR>
<TR><TD>00000001</TD><TD>01</TD><TD>1</TD><TD>* Modulation wheel</TD><TD>0-127</TD><TD>MSB</TD></TR>
<TR><TD>00000010</TD><TD>02</TD><TD>2</TD><TD>Breath control</TD><TD>0-127</TD><TD>MSB</TD></TR>
<TR><TD>00000011</TD><TD>03</TD><TD>3</TD><TD>Undefined</TD><TD>0-127</TD><TD>MSB</TD></TR>
<TR><TD>00000100</TD><TD>04</TD><TD>4</TD><TD>Foot controller</TD><TD>0-127</TD><TD>MSB</TD></TR>
<TR><TD>00000101</TD><TD>05</TD><TD>5</TD><TD>Portamento time</TD><TD>0-127</TD><TD>MSB</TD></TR>
<TR><TD>00000110</TD><TD>06</TD><TD>6</TD><TD>Data Entry</TD><TD>0-127</TD><TD>MSB</TD></TR>
<TR><TD>00000111</TD><TD>07</TD><TD>7</TD><TD>* Channel Volume (formerly Main Volume)</TD><TD>0-127</TD><TD>MSB</TD></TR>
<TR><TD>00001000</TD><TD>08</TD><TD>8</TD><TD>Balance</TD><TD>0-127</TD><TD>MSB</TD></TR>
<TR><TD>00001001</TD><TD>09</TD><TD>9</TD><TD>Undefined</TD><TD>0-127</TD><TD>MSB</TD></TR>
<TR><TD>00001010</TD><TD>0A</TD><TD>10</TD><TD>* Pan</TD><TD>0-127</TD><TD>MSB</TD></TR>
<TR><TD>00001011</TD><TD>0B</TD><TD>11</TD><TD>* Expression Controller</TD><TD>0-127</TD><TD>MSB</TD></TR>
<TR><TD>00001100</TD><TD>0C</TD><TD>12</TD><TD>Effect control 1</TD><TD>0-127</TD><TD>MSB</TD></TR>
<TR><TD>00001101</TD><TD>0D</TD><TD>13</TD><TD>Effect control 2</TD><TD>0-127</TD><TD>MSB</TD></TR>
<TR><TD>00001110</TD><TD>0E</TD><TD>14</TD><TD>Undefined</TD><TD>0-127</TD><TD>MSB</TD></TR>
<TR><TD>00001111</TD><TD>0F</TD><TD>15</TD><TD>Undefined</TD><TD>0-127</TD><TD>MSB</TD></TR>
<TR><TD>00010000</TD><TD>10</TD><TD>16</TD><TD>General Purpose Controller #1</TD><TD>0-127</TD><TD>MSB</TD></TR>
<TR><TD>00010001</TD><TD>11</TD><TD>17</TD><TD>General Purpose Controller #2</TD><TD>0-127</TD><TD>MSB</TD></TR>
<TR><TD>00010010</TD><TD>12</TD><TD>18</TD><TD>General Purpose Controller #3</TD><TD>0-127</TD><TD>MSB</TD></TR>
<TR><TD>00010011</TD><TD>13</TD><TD>19</TD><TD>General Purpose Controller #4</TD><TD>0-127</TD><TD>MSB</TD></TR>
<TR><TD>00010100</TD><TD>14</TD><TD>20</TD><TD>Undefined</TD><TD>0-127</TD><TD>MSB</TD></TR>
<TR><TD>00010101</TD><TD>15</TD><TD>21</TD><TD>Undefined</TD><TD>0-127</TD><TD>MSB</TD></TR>
<TR><TD>00010110</TD><TD>16</TD><TD>22</TD><TD>Undefined</TD><TD>0-127</TD><TD>MSB</TD></TR>
<TR><TD>00010111</TD><TD>17</TD><TD>23</TD><TD>Undefined</TD><TD>0-127</TD><TD>MSB</TD></TR>
<TR><TD>00011000</TD><TD>18</TD><TD>24</TD><TD>Undefined</TD><TD>0-127</TD><TD>MSB</TD></TR>
<TR><TD>00011001</TD><TD>19</TD><TD>25</TD><TD>Undefined</TD><TD>0-127</TD><TD>MSB</TD></TR>
<TR><TD>00011010</TD><TD>1A</TD><TD>26</TD><TD>Undefined</TD><TD>0-127</TD><TD>MSB</TD></TR>
<TR><TD>00011011</TD><TD>1B</TD><TD>27</TD><TD>Undefined</TD><TD>0-127</TD><TD>MSB</TD></TR>
<TR><TD>00011100</TD><TD>1C</TD><TD>28</TD><TD>Undefined</TD><TD>0-127</TD><TD>MSB</TD></TR>
<TR><TD>00011101</TD><TD>1D</TD><TD>29</TD><TD>Undefined</TD><TD>0-127</TD><TD>MSB</TD></TR>
<TR><TD>00011110</TD><TD>1E</TD><TD>30</TD><TD>Undefined</TD><TD>0-127</TD><TD>MSB</TD></TR>
<TR><TD>00011111</TD><TD>1F</TD><TD>31</TD><TD>Undefined</TD><TD>0-127</TD><TD>MSB</TD></TR>
<TR><TD>00100000</TD><TD>20</TD><TD>32</TD><TD>Bank Select</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>00100001</TD><TD>21</TD><TD>33</TD><TD>Modulation wheel</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>00100010</TD><TD>22</TD><TD>34</TD><TD>Breath control</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>00100011</TD><TD>23</TD><TD>35</TD><TD>Undefined</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>00100100</TD><TD>24</TD><TD>36</TD><TD>Foot controller</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>00100101</TD><TD>25</TD><TD>37</TD><TD>Portamento time</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>00100110</TD><TD>26</TD><TD>38</TD><TD>Data entry</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>00100111</TD><TD>27</TD><TD>39</TD><TD>Channel Volume (formerly Main Volume)</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>00101000</TD><TD>28</TD><TD>40</TD><TD>Balance</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>00101001</TD><TD>29</TD><TD>41</TD><TD>Undefined</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>00101010</TD><TD>2A</TD><TD>42</TD><TD>Pan</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>00101011</TD><TD>2B</TD><TD>43</TD><TD>Expression Controller</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>00101100</TD><TD>2C</TD><TD>44</TD><TD>Effect control 1</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>00101101</TD><TD>2D</TD><TD>45</TD><TD>Effect control 2</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>00101110</TD><TD>2E</TD><TD>46</TD><TD>Undefined</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>00101111</TD><TD>2F</TD><TD>47</TD><TD>Undefined</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>00110000</TD><TD>30</TD><TD>48</TD><TD>General Purpose Controller #1</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>00110001</TD><TD>31</TD><TD>49</TD><TD>General Purpose Controller #2</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>00110010</TD><TD>32</TD><TD>50</TD><TD>General Purpose Controller #3</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>00110011</TD><TD>33</TD><TD>51</TD><TD>General Purpose Controller #4</TD><TD>0-127</TD><TD>LSB</TD></TR> 
<TR><TD>00110100</TD><TD>34</TD><TD>52</TD><TD>Undefined</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>00110101</TD><TD>35</TD><TD>53</TD><TD>Undefined</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>00110110</TD><TD>36</TD><TD>54</TD><TD>Undefined</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>00110111</TD><TD>37</TD><TD>55</TD><TD>Undefined</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>00111000</TD><TD>38</TD><TD>56</TD><TD>Undefined</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>00111001</TD><TD>39</TD><TD>57</TD><TD>Undefined</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>00111010</TD><TD>3A</TD><TD>58</TD><TD>Undefined</TD><TD>0-127</TD><TD>LSB</TD></TR> 
<TR><TD>00111011</TD><TD>3B</TD><TD>59</TD><TD>Undefined</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>00111100</TD><TD>3C</TD><TD>60</TD><TD>Undefined</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>00111101</TD><TD>3D</TD><TD>61</TD><TD>Undefined</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>00111110</TD><TD>3E</TD><TD>62</TD><TD>Undefined</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>00111111</TD><TD>3F</TD><TD>63</TD><TD>Undefined</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>01000000</TD><TD>40</TD><TD>64</TD><TD>* Damper pedal on/off (Sustain)</TD><TD>&lt;63=off</TD><TD>&gt;64=on</TD></TR>
<TR><TD>01000001</TD><TD>41</TD><TD>65</TD><TD>Portamento on/off</TD><TD>&lt;63=off</TD><TD>&gt;64=on</TD></TR>
<TR><TD>01000010</TD><TD>42</TD><TD>66</TD><TD>Sustenuto on/off</TD><TD>&lt;63=off</TD><TD>&gt;64=on</TD></TR>
<TR><TD>01000011</TD><TD>43</TD><TD>67</TD><TD>Soft pedal on/off</TD><TD>&lt;63=off</TD><TD>&gt;64=on</TD></TR>
<TR><TD>01000100</TD><TD>44</TD><TD>68</TD><TD>Legato Footswitch</TD><TD>&lt;63=off</TD><TD>&gt;64=on</TD></TR>
<TR><TD>01000101</TD><TD>45</TD><TD>69</TD><TD>Hold 2</TD><TD>&lt;63=off</TD><TD>&gt;64=on</TD></TR>
<TR><TD>01000110</TD><TD>46</TD><TD>70</TD><TD>Sound Controller 1 (Sound Variation)</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>01000111</TD><TD>47</TD><TD>71</TD><TD>Sound Controller 2 (Timbre)</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>01001000</TD><TD>48</TD><TD>72</TD><TD>Sound Controller 3 (Release Time)</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>01001001</TD><TD>49</TD><TD>73</TD><TD>Sound Controller 4 (Attack Time)</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>01001010</TD><TD>4A</TD><TD>74</TD><TD>Sound Controller 5 (Brightness)</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>01001011</TD><TD>4B</TD><TD>75</TD><TD>Sound Controller 6</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>01001100</TD><TD>4C</TD><TD>76</TD><TD>Sound Controller 7</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>01001101</TD><TD>4D</TD><TD>77</TD><TD>Sound Controller 8</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>01001110</TD><TD>4E</TD><TD>78</TD><TD>Sound Controller 9</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>01001111</TD><TD>4F</TD><TD>79</TD><TD>Sound Controller 10</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>01010000</TD><TD>50</TD><TD>80</TD><TD>General Purpose Controller #5</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>01010001</TD><TD>51</TD><TD>81</TD><TD>General Purpose Controller #6</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>01010010</TD><TD>52</TD><TD>82</TD><TD>General Purpose Controller #7</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>01010011</TD><TD>53</TD><TD>83</TD><TD>General Purpose Controller #8</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>01010100</TD><TD>54</TD><TD>84</TD><TD>Portamento Control</TD><TD>0-127</TD><TD>Source Note</TD></TR>
<TR><TD>01010101</TD><TD>55</TD><TD>85</TD><TD>Undefined</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>01010110</TD><TD>56</TD><TD>86</TD><TD>Undefined</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>01010111</TD><TD>57</TD><TD>87</TD><TD>Undefined</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>01011000</TD><TD>58</TD><TD>88</TD><TD>Undefined</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>01011001</TD><TD>59</TD><TD>89</TD><TD>Undefined</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>01011010</TD><TD>5A</TD><TD>90</TD><TD>Undefined</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>01011011</TD><TD>5B</TD><TD>91</TD><TD>Effects 1 Depth</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>01011100</TD><TD>5C</TD><TD>92</TD><TD>Effects 2 Depth</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>01011101</TD><TD>5D</TD><TD>93</TD><TD>Effects 3 Depth</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>01011110</TD><TD>5E</TD><TD>94</TD><TD>Effects 4 Depth</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>01011111</TD><TD>5F</TD><TD>95</TD><TD>Effects 5 Depth</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD COLSPAN=6>&nbsp;</TD></TR>
<TR><TD>01100000</TD><TD>60</TD><TD>96</TD><TD>Data entry +1</TD><TD>N/A</TD><TD>&nbsp;</TD></TR>
<TR><TD>01100001</TD><TD>61</TD><TD>97</TD><TD>Data entry -1</TD><TD>N/A</TD><TD>&nbsp;</TD></TR>
<TR><TD>01100010</TD><TD>62</TD><TD>98</TD><TD>Non-Registered Parameter Number LSB</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>01100011</TD><TD>63</TD><TD>99</TD><TD>Non-Registered Parameter Number MSB</TD><TD>0-127</TD><TD>MSB</TD></TR>
<TR><TD>01100100</TD><TD>64</TD><TD>100</TD><TD>* Registered Parameter Number LSB</TD><TD>0-127</TD><TD>LSB</TD></TR>
<TR><TD>01100101</TD><TD>65</TD><TD>101</TD><TD>* Registered Parameter Number MSB</TD><TD>0-127</TD><TD>MSB</TD></TR>
<TR><TD>01100110</TD><TD>66</TD><TD>102</TD><TD>Undefined</TD><TD>?</TD><TD>&nbsp;</TD></TR>
<TR><TD>01100111</TD><TD>67</TD><TD>103</TD><TD>Undefined</TD><TD>?</TD><TD>&nbsp;</TD></TR>
<TR><TD>01101000</TD><TD>68</TD><TD>104</TD><TD>Undefined</TD><TD>?</TD><TD>&nbsp;</TD></TR>
<TR><TD>01101001</TD><TD>69</TD><TD>105</TD><TD>Undefined</TD><TD>?</TD><TD>&nbsp;</TD></TR>
<TR><TD>01101010</TD><TD>6A</TD><TD>106</TD><TD>Undefined</TD><TD>?</TD><TD>&nbsp;</TD></TR>
<TR><TD>01101011</TD><TD>6B</TD><TD>107</TD><TD>Undefined</TD><TD>?</TD><TD>&nbsp;</TD></TR>
<TR><TD>01101100</TD><TD>6C</TD><TD>108</TD><TD>Undefined</TD><TD>?</TD><TD>&nbsp;</TD></TR>
<TR><TD>01101101</TD><TD>6D</TD><TD>109</TD><TD>Undefined</TD><TD>?</TD><TD>&nbsp;</TD></TR>
<TR><TD>01101110</TD><TD>6E</TD><TD>110</TD><TD>Undefined</TD><TD>?</TD><TD>&nbsp;</TD></TR>
<TR><TD>01101111</TD><TD>6F</TD><TD>111</TD><TD>Undefined</TD><TD>?</TD><TD>&nbsp;</TD></TR>
<TR><TD>01110000</TD><TD>70</TD><TD>112</TD><TD>Undefined</TD><TD>?</TD><TD>&nbsp;</TD></TR>
<TR><TD>01110001</TD><TD>71</TD><TD>113</TD><TD>Undefined</TD><TD>?</TD><TD>&nbsp;</TD></TR>
<TR><TD>01110010</TD><TD>72</TD><TD>114</TD><TD>Undefined</TD><TD>?</TD><TD>&nbsp;</TD></TR>
<TR><TD>01110011</TD><TD>73</TD><TD>115</TD><TD>Undefined</TD><TD>?</TD><TD>&nbsp;</TD></TR>
<TR><TD>01110100</TD><TD>74</TD><TD>116</TD><TD>Undefined</TD><TD>?</TD><TD>&nbsp;</TD></TR>
<TR><TD>01110101</TD><TD>75</TD><TD>117</TD><TD>Undefined</TD><TD>?</TD><TD>&nbsp;</TD></TR>
<TR><TD>01110110</TD><TD>76</TD><TD>118</TD><TD>Undefined</TD><TD>?</TD><TD>&nbsp;</TD></TR>
<TR><TD>01110111</TD><TD>77</TD><TD>119</TD><TD>Undefined</TD><TD>?</TD><TD>&nbsp;</TD></TR>
<TR><TD COLSPAN=6>&nbsp;</TD></TR>
<TR><TD>01111000</TD><TD>78</TD><TD>120</TD><TD>All Sound Off</TD><TD>0</TD><TD>&nbsp;</TD></TR>
<TR><TD>01111001</TD><TD>79</TD><TD>121</TD><TD>* Reset All Controllers</TD><TD>0</TD><TD>&nbsp;</TD></TR>
<TR><TD>01111010</TD><TD>7A</TD><TD>122</TD><TD>Local control on/off</TD><TD>0=off</TD><TD>127=on</TD></TR>   
<TR><TD>01111011</TD><TD>7B</TD><TD>123</TD><TD>* All notes off</TD><TD>0</TD><TD>&nbsp;</TD></TR>
<TR><TD>01111100</TD><TD>7C</TD><TD>124</TD><TD>Omni mode off (+ all notes off)</TD><TD>0</TD><TD>&nbsp;</TD></TR>
<TR><TD>01111101</TD><TD>7D</TD><TD>125</TD><TD>Omni mode on (+ all notes off)</TD><TD>0</TD><TD>&nbsp;</TD></TR>
<TR><TD>01111110</TD><TD>7E</TD><TD>126</TD><TD>Poly mode on/off (+ all notes off)</TD><TD>**</TD><TD>&nbsp;</TD></TR>
<TR><TD>01111111</TD><TD>7F</TD><TD>127</TD><TD>Poly mode on (incl mono=off +all notes off)</TD><TD>0</TD><TD>&nbsp;</TD></TR>
</TABLE>
* Equipment must respond in order to comply with General MIDI level 1.<BR>
** This equals the number of channels, or zero if the number of channels
equals the number of voices in the receiver.
<P><A HREF="#CONTENTS">Back to contents</A></P>
<HR>
<CENTER><H3><A NAME="BMA1_3">Appendix 1.3 - Table of MIDI Note Numbers</A></H3></CENTER>
<P ALIGN=LEFT>
This table lists all MIDI Note Numbers by octave.
<P>
<I>The absolute octave number designations are based on Middle C = C4,
which is an arbitrary but widely used assignment.</I>
<P>
<TABLE BORDER=1>
<TR><TH>Octave #</TH><TH COLSPAN=12>Note Numbers</TH></TR>
<TR><TH>&nbsp;</TH><TH>C</TH><TH>C#</TH><TH>D</TH><TH>D#</TH><TH>E</TH><TH>F</TH>
<TH>F#</TH><TH>G</TH><TH>G#</TH><TH>A</TH><TH>A#</TH><TH>B</TH></TR>
<TR><TD>-1</TD><TD>0</TD><TD>1</TD><TD>2</TD><TD>3</TD><TD>4</TD><TD>5</TD><TD>6</TD><TD>7</TD><TD>8</TD><TD>9</TD><TD>10</TD><TD>11</TD></TR>
<TR><TD>0</TD><TD>12</TD><TD>13</TD><TD>14</TD><TD>15</TD><TD>16</TD><TD>17</TD><TD>18</TD><TD>19</TD><TD>20</TD><TD>21</TD><TD>22</TD><TD>23</TD></TR>
<TR><TD>1</TD><TD>24</TD><TD>25</TD><TD>26</TD><TD>27</TD><TD>28</TD><TD>29</TD><TD>30</TD><TD>31</TD><TD>32</TD><TD>33</TD><TD>34</TD><TD>35</TD></TR>
<TR><TD>2</TD><TD>36</TD><TD>37</TD><TD>38</TD><TD>39</TD><TD>40</TD><TD>41</TD><TD>42</TD><TD>43</TD><TD>44</TD><TD>45</TD><TD>46</TD><TD>47</TD></TR>
<TR><TD>3</TD><TD>48</TD><TD>49</TD><TD>50</TD><TD>51</TD><TD>52</TD><TD>53</TD><TD>54</TD><TD>55</TD><TD>56</TD><TD>57</TD><TD>58</TD><TD>59</TD></TR>
<TR><TD>4</TD><TD>60</TD><TD>61</TD><TD>62</TD><TD>63</TD><TD>64</TD><TD>65</TD><TD>66</TD><TD>67</TD><TD>68</TD><TD>69</TD><TD>70</TD><TD>71</TD></TR>
<TR><TD>5</TD><TD>72</TD><TD>73</TD><TD>74</TD><TD>75</TD><TD>76</TD><TD>77</TD><TD>78</TD><TD>79</TD><TD>80</TD><TD>81</TD><TD>82</TD><TD>83</TD></TR>
<TR><TD>6</TD><TD>84</TD><TD>85</TD><TD>86</TD><TD>87</TD><TD>88</TD><TD>89</TD><TD>90</TD><TD>91</TD><TD>92</TD><TD>93</TD><TD>94</TD><TD>95</TD></TR>
<TR><TD>7</TD><TD>96</TD><TD>97</TD><TD>98</TD><TD>99</TD><TD>100</TD><TD>101</TD><TD>102</TD><TD>103</TD><TD>104</TD><TD>105</TD><TD>106</TD><TD>107</TD></TR>
<TR><TD>8</TD><TD>108</TD><TD>109</TD><TD>110</TD><TD>111</TD><TD>112</TD><TD>113</TD><TD>114</TD><TD>115</TD><TD>116</TD><TD>117</TD><TD>118</TD><TD>119</TD></TR>
<TR><TD>9</TD><TD>120</TD><TD>121</TD><TD>122</TD><TD>123</TD><TD>124</TD><TD>125</TD><TD>126</TD><TD>127</TD><TD>&nbsp;</TD><TD>&nbsp;</TD><TD>&nbsp;</TD><TD>&nbsp;</TD></TR>
</TABLE>
<P><A HREF="#CONTENTS">Back to contents</A></P>
<HR>
<CENTER><H3><A NAME="BMA1_4">Appendix 1.4 - General MIDI Instrument Patch Map</A></H3></CENTER>
<P ALIGN=LEFT><UL>
<LI>The names of the instruments indicate what sort of sound will
be heard when that instrument number (MIDI Program Change or &quot;PC#&quot;)
is selected on the GM synthesizer.
<LI>These sounds are the same for all MIDI Channels except Channel
10, which has only percussion sounds and some sound &quot;effects&quot;.
(See <A HREF="#BMA1_5">Appendix 1.5</A> - General MIDI Percussion Key Map)
</UL>
<H3>GM Instrument Families </H3>
<P>
The General MIDI instrument sounds are grouped by families. In
each family are 8 specific instruments.
<P>
<TABLE BORDER=3>
<TR><TH>PC#</TH><TH>Family</TH><TH>PC#</TH><TH>Family</TH></TR>
<TR><TD>1-8</TD><TD>Piano</TD><TD>65-72</TD><TD>Reed</TD></TR>
<TR><TD>9-16</TD><TD>Chromatic Percussion</TD><TD>73-80</TD><TD>Pipe</TD></TR>
<TR><TD>17-24</TD><TD>Organ</TD><TD>81-88</TD><TD>Synth Lead</TD></TR>
<TR><TD>25-32</TD><TD>Guitar</TD><TD>89-96</TD><TD>Synth Pad</TD></TR>
<TR><TD>33-40</TD><TD>Bass</TD><TD>97-104</TD><TD>Synth Effects</TD></TR>
<TR><TD>41-48</TD><TD>Strings</TD><TD>105-112</TD><TD>Ethnic</TD></TR>
<TR><TD>49-56</TD><TD>Ensemble</TD><TD>113-120</TD><TD>Percussive</TD></TR>
<TR><TD>57-64</TD><TD>Brass</TD><TD>121-128</TD><TD>Sound Effects</TD></TR>
</TABLE>
<H3>GM Instrument Patch Map</H3>
<P>
<B>Note:</B> While GM does not define the actual characteristics
of any sounds, the names in parentheses after each of the synth
leads, pads, and sound effects are, in particular, intended only
as guides.
<P>
<TABLE BORDER=3>
<TR><TH>PC#</TH><TH>Instrument</TH><TH>PC#</TH><TH>Instrument</TH></TR>
<TR><TD>1.</TD><TD>Acoustic Grand Piano</TD><TD>65.</TD><TD>Soprano Sax</TD></TR>
<TR><TD>2.</TD><TD>Bright Acoustic Piano</TD><TD>66.</TD><TD>Alto Sax</TD></TR>
<TR><TD>3.</TD><TD>Electric Grand Piano</TD><TD>67.</TD><TD>Tenor Sax</TD></TR>
<TR><TD>4.</TD><TD>Honky-tonk Piano</TD><TD>68.</TD><TD>Baritone Sax</TD></TR>
<TR><TD>5.</TD><TD>Electric Piano 1 (Rhodes Piano)</TD><TD>69.</TD><TD>Oboe</TD></TR>
<TR><TD>6.</TD><TD>Electric Piano 2 (Chorused Piano)</TD><TD>70.</TD><TD>English Horn</TD></TR>
<TR><TD>7.</TD><TD>Harpsichord</TD><TD>71.</TD><TD>Bassoon</TD></TR>
<TR><TD>8.</TD><TD>Clavinet</TD><TD>72.</TD><TD>Clarinet</TD></TR>
<TR><TD>9.</TD><TD>Celesta</TD><TD>73.</TD><TD>Piccolo</TD></TR>
<TR><TD>10.</TD><TD>Glockenspiel</TD><TD>74.</TD><TD>Flute</TD></TR>
<TR><TD>11.</TD><TD>Music Box</TD><TD>75.</TD><TD>Recorder</TD></TR>
<TR><TD>12.</TD><TD>Vibraphone</TD><TD>76.</TD><TD>Pan Flute</TD></TR>
<TR><TD>13.</TD><TD>Marimba</TD><TD>77.</TD><TD>Blown Bottle</TD></TR>
<TR><TD>14.</TD><TD>Xylophone</TD><TD>78.</TD><TD>Shakuhachi</TD></TR>
<TR><TD>15.</TD><TD>Tubular Bells</TD><TD>79.</TD><TD>Whistle</TD></TR>
<TR><TD>16.</TD><TD>Dulcimer (Santur)</TD><TD>80.</TD><TD>Ocarina</TD></TR>
<TR><TD>17.</TD><TD>Drawbar Organ (Hammond)</TD><TD>81.</TD><TD>Lead 1 (square wave)</TD></TR>
<TR><TD>18.</TD><TD>Percussive Organ</TD><TD>82.</TD><TD>Lead 2 (sawtooth wave)</TD></TR>
<TR><TD>19.</TD><TD>Rock Organ</TD><TD>83.</TD><TD>Lead 3 (calliope)</TD></TR>
<TR><TD>20.</TD><TD>Church Organ</TD><TD>84.</TD><TD>Lead 4 (chiffer)</TD></TR>
<TR><TD>21.</TD><TD>Reed Organ</TD><TD>85.</TD><TD>Lead 5 (charang)</TD></TR>
<TR><TD>22.</TD><TD>Accordion (French)</TD><TD>86.</TD><TD>Lead 6 (voice solo)</TD></TR>
<TR><TD>23.</TD><TD>Harmonica</TD><TD>87.</TD><TD>Lead 7 (fifths)</TD></TR>
<TR><TD>24.</TD><TD>Tango Accordion (Band neon)</TD><TD>88.</TD><TD>Lead 8 (bass + lead)</TD></TR>
<TR><TD>25.</TD><TD>Acoustic Guitar (nylon)</TD><TD>89.</TD><TD>Pad 1 (new age Fantasia)</TD></TR>
<TR><TD>26.</TD><TD>Acoustic Guitar (steel)</TD><TD>90.</TD><TD>Pad 2 (warm)</TD></TR>
<TR><TD>27.</TD><TD>Electric Guitar (jazz)</TD><TD>91.</TD><TD>Pad 3 (polysynth)</TD></TR>
<TR><TD>28.</TD><TD>Electric Guitar (clean)</TD><TD>92.</TD><TD>Pad 4 (choir space voice)</TD></TR>
<TR><TD>29.</TD><TD>Electric Guitar (muted)</TD><TD>93.</TD><TD>Pad 5 (bowed glass)</TD></TR>
<TR><TD>30.</TD><TD>Overdriven Guitar</TD><TD>94.</TD><TD>Pad 6 (metallic pro)</TD></TR>
<TR><TD>31.</TD><TD>Distortion Guitar</TD><TD>95.</TD><TD>Pad 7 (halo)</TD></TR>
<TR><TD>32.</TD><TD>Guitar harmonics</TD><TD>96.</TD><TD>Pad 8 (sweep)</TD></TR>
<TR><TD>33.</TD><TD>Acoustic Bass</TD><TD>97.</TD><TD>FX 1 (rain)</TD></TR>
<TR><TD>34.</TD><TD>Electric Bass (fingered)</TD><TD>98.</TD><TD>FX 2 (soundtrack)</TD></TR>
<TR><TD>35.</TD><TD>Electric Bass (picked)</TD><TD>99.</TD><TD>FX 3 (crystal)</TD></TR>
<TR><TD>36.</TD><TD>Fretless Bass</TD><TD>100.</TD><TD>FX 4 (atmosphere)</TD></TR>
<TR><TD>37.</TD><TD>Slap Bass 1</TD><TD>101.</TD><TD>FX 5 (brightness)</TD></TR>
<TR><TD>38.</TD><TD>Slap Bass 2</TD><TD>102.</TD><TD>FX 6 (goblins)</TD></TR>
<TR><TD>39.</TD><TD>Synth Bass 1</TD><TD>103.</TD><TD>FX 7 (echoes, drops)</TD></TR>
<TR><TD>40.</TD><TD>Synth Bass 2</TD><TD>104.</TD><TD>FX 8 (sci-fi, star theme)</TD></TR>
<TR><TD>41.</TD><TD>Violin</TD><TD>105.</TD><TD>Sitar</TD></TR>
<TR><TD>42.</TD><TD>Viola</TD><TD>106.</TD><TD>Banjo</TD></TR>
<TR><TD>43.</TD><TD>Cello</TD><TD>107.</TD><TD>Shamisen</TD></TR>
<TR><TD>44.</TD><TD>Contrabass</TD><TD>108.</TD><TD>Koto</TD></TR>
<TR><TD>45.</TD><TD>Tremolo Strings</TD><TD>109.</TD><TD>Kalimba</TD></TR>
<TR><TD>46.</TD><TD>Pizzicato Strings</TD><TD>110.</TD><TD>Bag pipe</TD></TR>
<TR><TD>47.</TD><TD>Orchestral Harp</TD><TD>111.</TD><TD>Fiddle</TD></TR>
<TR><TD>48.</TD><TD>Timpani</TD><TD>112.</TD><TD>Shanai</TD></TR>
<TR><TD>49.</TD><TD>String Ensemble 1 (strings)</TD><TD>113.</TD><TD>Tinkle Bell</TD></TR>
<TR><TD>50.</TD><TD>String Ensemble 2 (slow strings)</TD><TD>114.</TD><TD>Agogo</TD></TR>
<TR><TD>51.</TD><TD>SynthStrings 1</TD><TD>115.</TD><TD>Steel Drums</TD></TR>
<TR><TD>52.</TD><TD>SynthStrings 2</TD><TD>116.</TD><TD>Woodblock</TD></TR>
<TR><TD>53.</TD><TD>Choir Aahs</TD><TD>117.</TD><TD>Taiko Drum</TD></TR>
<TR><TD>54.</TD><TD>Voice Oohs</TD><TD>118.</TD><TD>Melodic Tom</TD></TR>
<TR><TD>55.</TD><TD>Synth Voice</TD><TD>119.</TD><TD>Synth Drum</TD></TR>
<TR><TD>56.</TD><TD>Orchestra Hit</TD><TD>120.</TD><TD>Reverse Cymbal</TD></TR>
<TR><TD>57.</TD><TD>Trumpet</TD><TD>121.</TD><TD>Guitar Fret Noise</TD></TR>
<TR><TD>58.</TD><TD>Trombone</TD><TD>122.</TD><TD>Breath Noise</TD></TR>
<TR><TD>59.</TD><TD>Tuba</TD><TD>123.</TD><TD>Seashore</TD></TR>
<TR><TD>60.</TD><TD>Muted Trumpet</TD><TD>124.</TD><TD>Bird Tweet</TD></TR>
<TR><TD>61.</TD><TD>French Horn</TD><TD>125.</TD><TD>Telephone Ring</TD></TR>
<TR><TD>62.</TD><TD>Brass Section</TD><TD>126.</TD><TD>Helicopter</TD></TR>
<TR><TD>63.</TD><TD>SynthBrass 1</TD><TD>127.</TD><TD>Applause</TD></TR>
<TR><TD>64.</TD><TD>SynthBrass 2</TD><TD>128.</TD><TD>Gunshot</TD></TR>
</TABLE>
<P><A HREF="#CONTENTS">Back to contents</A></P>
<HR>
<CENTER><H3><A NAME="BMA1_5">Appendix 1.5 - General MIDI Percussion Key Map</A></H3></CENTER>
<P ALIGN=LEFT>
On MIDI Channel 10, each MIDI Note number ("Key#") corresponds
to a different drum sound, as shown below.
GM-compatible instruments must have the sounds on the keys
shown here. While many current instruments also have additional
sounds above or below the range show here, and may even have
additional "kits" with variations of these sounds, only these
sounds are supported by General MIDI.
<P>
<TABLE BORDER=2>
<TR><TH>Key#</TH><TH>Note</TH><TH>Drum Sound</TH><TH>Key#</TH><TH>Note</TH><TH>Drum Sound</TH></TR>
<TR><TD>35</TD><TD>B1</TD><TD>Acoustic Bass Drum</TD><TD>59</TD><TD>B3</TD><TD>Ride Cymbal 2</TD></TR>
<TR><TD>36</TD><TD>C2</TD><TD>Bass Drum 1</TD><TD>60</TD><TD>C4</TD><TD>Hi Bongo</TD></TR>
<TR><TD>37</TD><TD>C#2</TD><TD>Side Stick</TD><TD>61</TD><TD>C#4</TD><TD>Low Bongo</TD></TR>
<TR><TD>38</TD><TD>D2</TD><TD>Acoustic Snare</TD><TD>62</TD><TD>D4</TD><TD>Mute Hi Conga</TD></TR>
<TR><TD>39</TD><TD>D#2</TD><TD>Hand Clap</TD><TD>63</TD><TD>D#4</TD><TD>Open Hi Conga</TD></TR>
<TR><TD>40</TD><TD>E2</TD><TD>Electric Snare</TD><TD>64</TD><TD>E4</TD><TD>Low Conga</TD></TR>
<TR><TD>41</TD><TD>F2</TD><TD>Low Floor Tom</TD><TD>65</TD><TD>F4</TD><TD>High Timbale</TD></TR>
<TR><TD>42</TD><TD>F#2</TD><TD>Closed Hi Hat</TD><TD>66</TD><TD>F#4</TD><TD>Low Timbale</TD></TR>
<TR><TD>43</TD><TD>G2</TD><TD>High Floor Tom</TD><TD>67</TD><TD>G4</TD><TD>High Agogo</TD></TR>
<TR><TD>44</TD><TD>G#2</TD><TD>Pedal Hi-Hat</TD><TD>68</TD><TD>G#4</TD><TD>Low Agogo</TD></TR>
<TR><TD>45</TD><TD>A2</TD><TD>Low Tom</TD><TD>69</TD><TD>A4</TD><TD>Cabasa</TD></TR>
<TR><TD>46</TD><TD>A#2</TD><TD>Open Hi-Hat</TD><TD>70</TD><TD>A#4</TD><TD>Maracas</TD></TR>
<TR><TD>47</TD><TD>B2</TD><TD>Low-Mid Tom</TD><TD>71</TD><TD>B4</TD><TD>Short Whistle</TD></TR>
<TR><TD>48</TD><TD>C3</TD><TD>Hi Mid Tom</TD><TD>72</TD><TD>C5</TD><TD>Long Whistle</TD></TR>
<TR><TD>49</TD><TD>C#3</TD><TD>Crash Cymbal 1</TD><TD>73</TD><TD>C#5</TD><TD>Short Guiro</TD></TR>
<TR><TD>50</TD><TD>D3</TD><TD>High Tom</TD><TD>74</TD><TD>D5</TD><TD>Long Guiro</TD></TR>
<TR><TD>51</TD><TD>D#3</TD><TD>Ride Cymbal 1</TD><TD>75</TD><TD>D#5</TD><TD>Claves</TD></TR>
<TR><TD>52</TD><TD>E3</TD><TD>Chinese Cymbal</TD><TD>76</TD><TD>E5</TD><TD>Hi Wood Block</TD></TR>
<TR><TD>53</TD><TD>F3</TD><TD>Ride Bell</TD><TD>77</TD><TD>F5</TD><TD>Low Wood Block</TD></TR>
<TR><TD>54</TD><TD>F#3</TD><TD>Tambourine</TD><TD>78</TD><TD>F#5</TD><TD>Mute Cuica</TD></TR>
<TR><TD>55</TD><TD>G3</TD><TD>Splash Cymbal</TD><TD>79</TD><TD>G5</TD><TD>Open Cuica</TD></TR>
<TR><TD>56</TD><TD>G#3</TD><TD>Cowbell</TD><TD>80</TD><TD>G#5</TD><TD>Mute Triangle</TD></TR>
<TR><TD>57</TD><TD>A3</TD><TD>Crash Cymbal 2</TD><TD>81</TD><TD>A5</TD><TD>Open Triangle</TD></TR>
<TR><TD>58</TD><TD>A#3</TD><TD>Vibraslap</TD><TD>&nbsp;</TD><TD>&nbsp;</TD><TD>&nbsp;</TD></TR>
</TABLE>
<P><A HREF="#CONTENTS">Back to contents</A></P>
<HR>
<CENTER><H2><A NAME="BMA2_">Appendix 2 - Program Fragments and Example MIDI Files</A></H2></CENTER>
<P ALIGN=LEFT>
Here are some of the routines to read and write
variable-length numbers in MIDI Files. These routines are in C,
and use getc and putc, which read and write single 8-bit
characters from/to the files infile and outfile.
<P>
WriteVarLen(value)<BR>
register long value;<BR>
{<BR>
register long buffer;<BR>
<P>
buffer = value &amp; 0x7f;<BR>
while((value &gt;&gt;= 7) &gt; 0)<BR>
&nbsp;{<BR>
&nbsp;buffer &lt;&lt;= 8;<BR>
&nbsp;buffer |= 0x80;<BR>
&nbsp;buffer += (value &amp;0x7f);<BR>
&nbsp;}

<P>while (TRUE)<BR>
&nbsp;{<BR>
&nbsp;putc(buffer,outfile);<BR>
&nbsp;if(buffer &amp; 0x80) buffer &gt;&gt;= 8;<BR>
&nbsp;else<BR>
&nbsp;break;<BR>
&nbsp;}<BR>
}
<P>
doubleword ReadVarLen()<BR>
{<BR>
register doubleword value;<BR>
register byte c;
<P>
if((value = getc(infile)) &amp; 0x80)<BR>
&nbsp;{<BR>
&nbsp;value &amp;= 0x7f;<BR>
&nbsp;do<BR>
&nbsp;&nbsp;{<BR>
&nbsp;&nbsp;value = (value &lt;&lt; 7) + ((c = getc(infile))) &amp; 0x7f);<BR>
&nbsp;&nbsp;} while (c &amp; 0x80);<BR>
&nbsp;}<BR>
return(value);<BR>
}
<P>As an example, MIDI Files for the following excerpt are shown
below. First, a format 0 file is shown, with all information
intermingled; then, a format 1 file is shown with all data
separated into four tracks: one for tempo and time signature, and
three for the notes. A resolution of 96 &quot;ticks&quot; per
quarter note is used. A time signature of 4/4 and a tempo of 120,
though implied, are explicitly stated.<BR>
![](http://www.music.mcgill.ca/~ich/classes/mumt306/test.gif)<BR>
The contents of the MIDI stream represented by this example
are broken down here:
<TABLE BORDER=1>
<TR><TH>Delta-Time<BR> (decimal)</TH><TH>Event-Code<BR> (hex)</TH><TH>Other Bytes<BR> (decimal)</TH><TH>Comment</TH></TR>
<TR><TD>0 </TD><TD>FF 58 </TD><TD>04 04 02 24 08 </TD><TD>4 bytes; 4/4 time; 24 MIDI clocks/click, 8 32nd notes/ 24 MIDI clocks (24 MIDI clocks = 1 crotchet = 1 beat)</TD></TR>
<TR><TD>0 </TD><TD>FF 51 </TD><TD>03 500000 </TD><TD>3 bytes: 500,000 usec/ quarter note = 120 beats/minute</TD></TR>
<TR><TD>0 </TD><TD>C0 </TD><TD>5 </TD><TD>Ch.1 Program Change 5 = GM Patch 6 = Electric Piano 2</TD></TR>
<TR><TD>0 </TD><TD>C1 </TD><TD>46 </TD><TD>Ch.2 Program Change 46 = GM Patch 47 = Harp</TD></TR>
<TR><TD>0 </TD><TD>C2 </TD><TD>70 </TD><TD>Ch.3 Program Change 70 = GM Patch 71 = Bassoon</TD></TR>
<TR><TD>0 </TD><TD>92 </TD><TD>48 96 </TD><TD>Ch.3 Note On C3, forte</TD></TR>
<TR><TD>0 </TD><TD>92 </TD><TD>60 96 </TD><TD>Ch.3 Note On C4, forte</TD></TR>
<TR><TD>96 </TD><TD>91 </TD><TD>67 64 </TD><TD>Ch.2 Note On G4, mezzo-forte</TD></TR>
<TR><TD>96 </TD><TD>90 </TD><TD>76 32 </TD><TD>Ch.1 Note On E5, piano</TD></TR>
<TR><TD>192 </TD><TD>82 </TD><TD>48 64 </TD><TD>Ch.3 Note Off C3, standard</TD></TR>
<TR><TD>0 </TD><TD>82 </TD><TD>60 64 </TD><TD>Ch.3 Note Off C4, standard</TD></TR>
<TR><TD>0 </TD><TD>81 </TD><TD>67 64 </TD><TD>Ch.2 Note Off G4, standard</TD></TR>
<TR><TD>0 </TD><TD>80 </TD><TD>76 64 </TD><TD>Ch.1 Note Off E5, standard</TD></TR>
<TR><TD>0 </TD><TD>FF 2F </TD><TD>00 </TD><TD>Track End</TD></TR>
</TABLE>
The entire format 0 MIDI file contents in hex follow. First, the header chunk:
<TABLE BORDER=1>
<TR><TD>4D 54 68 64 </TD><TD>MThd</TD></TR>
<TR><TD>00 00 00 06 </TD><TD>chunk length</TD></TR>
<TR><TD>00 00 </TD><TD>format 0</TD></TR>
<TR><TD>00 01 </TD><TD>one track</TD></TR>
<TR><TD>00 60 </TD><TD>96 per quarter-note</TD></TR>
</TABLE>
Then the track chunk. Its header followed by the events (notice the running status is used in places):
<TABLE BORDER=1>
<TR><TD>4D 54 72 6B </TD><TD>MTrk</TD></TR>
<TR><TD> 00 00 00 3B </TD><TD>chunk length (59)</TD></TR>
</TABLE>
<TABLE BORDER=1>
<TR><TH>Delta-Time </TH><TH>Event </TH><TH>Comments</TH></TR>
<TR><TD>00 </TD><TD>FF 58 04 04 02 18 08 </TD><TD>time signature</TD></TR>
<TR><TD>00 </TD><TD>FF 51 03 07 A1 20 </TD><TD>tempo</TD></TR>
<TR><TD>00 </TD><TD>C0 05 </TD><TD>&nbsp;</TD></TR>
<TR><TD>00 </TD><TD>C1 2E</TD><TD>&nbsp;</TD></TR>
<TR><TD>00 </TD><TD>C2 46</TD><TD>&nbsp;</TD></TR>
<TR><TD>00 </TD><TD>92 30 60</TD><TD>&nbsp;</TD></TR>
<TR><TD>00 </TD><TD>3C 60 </TD><TD>running status</TD></TR>
<TR><TD>60 </TD><TD>91 43 40</TD><TD>&nbsp;</TD></TR>
<TR><TD>60 </TD><TD>90 4C 20</TD><TD>&nbsp;</TD></TR>
<TR><TD>81 40 </TD><TD>82 30 40 </TD><TD>two-byte delta-time</TD></TR>
<TR><TD>00 </TD><TD>3C 40 </TD><TD>running status</TD></TR>
<TR><TD>00 </TD><TD>81 43 40</TD><TD>&nbsp;</TD></TR>
<TR><TD>00 </TD><TD>80 4C 40</TD><TD>&nbsp;</TD></TR>
<TR><TD>00 </TD><TD>FF 2F 00 </TD><TD>end of track</TD></TR>
</TABLE>
A format 1 representation of the file is slightly different.
Its header chunk:
<TABLE BORDER=1>
<TR><TD>4D 54 68 64 </TD><TD>MThd</TD></TR>
<TR><TD>00 00 00 06 </TD><TD>chunk length</TD></TR>
<TR><TD>00 01 </TD><TD>format 1</TD></TR>
<TR><TD>00 04 </TD><TD>four tracks</TD></TR>
<TR><TD>00 60 </TD><TD>96 per quarter note</TD></TR>
</TABLE>
First, the track chunk for the time signature/tempo track. Its
header, followed by the events:
<TABLE BORDER=1>
<TR><TD>4D 54 72 6B </TD><TD>MTrk</TD></TR>
<TR><TD>00 00 00 14 </TD><TD>chunk length (20)</TD></TR>
</TABLE>
<TABLE BORDER=1>
<TR><TH>Delta-Time </TH><TH>Event </TH><TH>Comments</TH></TR>
<TR><TD>00 </TD><TD>FF 58 04 04 02 18 08 </TD><TD>time signature</TD></TR>
<TR><TD>00 </TD><TD>FF 51 03 07 A1 20 </TD><TD>tempo</TD></TR>
<TR><TD>83 00 </TD><TD>FF 2F 00 </TD><TD>end of track</TD></TR>
</TABLE>
Then, the track chunk for the first music track. The MIDI
convention for note on/off running status is used in this
example:
<TABLE BORDER=1>
<TR><TD>4D 54 72 6B </TD><TD>MTrk</TD></TR>
<TR><TD>00 00 00 10 </TD><TD>chunk length (16)</TD></TR>
</TABLE>
<TABLE BORDER=1>
<TR><TH>Delta-Time </TH><TH>Event </TH><TH>Comments</TH></TR>
<TR><TD>00 </TD><TD>C0 05</TD><TD>&nbsp;</TD></TR>
<TR><TD>81 40 </TD><TD>90 4C 20</TD><TD>&nbsp;</TD></TR>
<TR><TD>81 40 </TD><TD>4C 00 </TD><TD>Running status: note on, vel=0</TD></TR>
<TR><TD>00 FF 2F 00</TD><TD>&nbsp;</TD><TD>&nbsp;</TD></TR>
</TABLE>
Then, the track chunk for the second music track:
<TABLE BORDER=1>
<TR><TD>4D 54 72 6B </TD><TD>MTrk</TD></TR>
<TR><TD>00 00 00 0F </TD><TD>chunk length (15)</TD></TR>
</TABLE>
<TABLE BORDER=1>
<TR><TH>Delta-Time </TH><TH>Event </TH><TH>Comments</TH></TR>
<TR><TD>00 </TD><TD>C1 2E</TD><TD>&nbsp;</TD></TR>
<TR><TD>60 </TD><TD>91 43 40</TD><TD>&nbsp;</TD></TR>
<TR><TD>82 20 </TD><TD>43 00 </TD><TD>running status</TD></TR>
<TR><TD>00 </TD><TD>FF 2F 00 </TD><TD>end of track</TD></TR>
</TABLE>
Then, the track chunk for the third music track:
<TABLE BORDER=1>
<TR><TD>4D 54 72 6B </TD><TD>MTrk</TD></TR>
<TR><TD>00 00 00 15 </TD><TD>chunk length (21)</TD></TR>
</TABLE>
<TABLE BORDER=1>
<TR><TH>Delta-Time </TH><TH>Event </TH><TH>Comments</TH></TR>
<TR><TD>00 </TD><TD>C2 46</TD><TD>&nbsp;</TD></TR>
<TR><TD>00 </TD><TD>92 30 60</TD><TD>&nbsp;</TD></TR>
<TR><TD>00 </TD><TD>3C 60 </TD><TD>running status</TD></TR>
<TR><TD>83 00 </TD><TD>30 00 </TD><TD>two-byte delta-time, running status</TD></TR>
<TR><TD>00 </TD><TD>3C 00 </TD><TD>running status</TD></TR>
<TR><TD> 00 </TD><TD>FF 2F 00 </TD><TD>end of track</TD></TR>
</TABLE>
<P><A HREF="#CONTENTS">Back to contents</A></P>
END.
</body>
</html>
