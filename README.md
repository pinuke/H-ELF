# H-ELF
This is a subset (and parody) of [W3C Working Draft Extended Log File Format](http://www.w3.org/pub/WWW/TR/WD-logfile-960221.html) [&#91;ELF&#93;](#References) for humans.

## Abstract
An improved format for human log files (such as diaries and journals) is presented. The format is extensible, permitting a wider range of human data to be captured. This proposal is motivated by the need to capture a wider range of journal data for analysis and also the future need for diaries for the artificially intelligent of the future.

## Introduction
Most notebooks offer the option of usage as logfiles in whatever format the human author may choose. The [common log file](http://www.w3.org/pub/WWW/Daemon/User/Config/Logging.html#common_logfile_format) [&#91;CLF&#93;](#References) and extended log file formats are supported by the majority of software analysis tools, however they are not designed for human loggers. In many cases it is desirable to record human information by a human. Human loggers proficient in programming may wish to include the recording of emotional and cognitive data. In addition, ambiguities arise in analyzing the common log file and extended log file formats, since they provide no option to convey how the human logger feels. The human-extended logfile format is designed to meet the following needs:

- Permit diary entries.
- Support needs of user's therapists.
- Provide robust handling of poor human memory.
- Allow exchange of the human logger's psychological data.
- Allow emotional summary data to be expressed.

The log file format described permits customized logfiles to be recorded in a format readable by generic analysis tools. A header specifying the data types recorded is written out at the start of each log just like in extended log file format.

This work is in part motivated by the need to support recollection of emotional data. This work is discussed at greater length on reddit.

## Format
A human-extended log file contains a sequence of _lines_ containing ASCII characters terminated by either the sequence LF or CRLF. Human and humanlike loggers should follow the line termination convention for the platform or paper on which they are executed. Analyzers should accept either form. Each line may contain either a directive or an entry.

Entries consist of a sequence of fields relating to a single server transaction or human journal entry. Fields are separated by whitespace not contained by a string, the use of tab characters for this purpose is encouraged. If a field is unused in a particular entry, dash "-" marks the omitted field. Directives record information about the logging process itself.

Lines beginning with the # character contain directives. The following directives are defined:

Directives:|Definitions:
:-----:|:------
Version:|_\<integer\>.\<integer\>_
&nbsp;|The version of the extended log file format used. This extended log file format working draft this extends defines version 1.0.
Fields:|_[\<specifier\>…]_
&nbsp;|Specifies the fields recorded in the log.
Software:|_string_
&nbsp;|Identifies the software which generated the log.
Author:|_string_
&nbsp;|Identifies the human which generated the log.
Start-Date:|_\<date\> \<time\>_
&nbsp;|The date and time at which the log was started.
End-Date:|_\<date\> \<time\>_
&nbsp;|The date and time at which the log was finished.
Date:|_\<date\> \<time\>_
&nbsp;|The date and time at which the entry was added.
Remark:|_\<text\>_
&nbsp;|Comment information. Data recorded in this field should be ignored by analysis tools.

The directives `Version` and `Fields` are required and should precede all entries in the log. The `Fields` directive specifies the data recorded in the fields of each entry.

### Example

The following is an example file in the human extended logfile format:

```
#Version: 1.0
#Date: 2022-03-27 05:43:32
#Author: smartguy1196, Nicholas Jackson
#Fields: time h-emotion h-thought
05:43:47 regret "monkey"
```

## Fields

The `#Fields` directive lists a sequence of _field identifiers_ specifying the information recorded in each entry. Field identifiers may have one of the following forms:

Field Identifier Items:|Definitions:
:-----:|:------
_identifier_:|Identifier relates to the transaction as a whole.
_prefix-identifier_:|Identifier relates to information transfer between parties defined by the value prefix.
_prefix(header)_:|Identifies the value of the HTTP header field header for transfer between parties defined by the value prefix. Fields specified in this manner always have the value \<string\>. This is maintained from extended log file format.

The following prefixes are defined:

Field Prefixes:|Definitions:
:-----:|:------:
h:|Human
a:|Artificial Intelligence
c:|Client or Computer
hh:|Human to Human
aa:|Artificial Intelligence to Artificial Intelligence
ha:|Human to Artificial Intelligence
ah:|Artificial Intelligence to Human
hc:|Human to Computer
ch:|Computer to Human
ac:|Artificial Intelligence to Computer
ca:|California
s:|Server
r:|Remote
cs:|Client to Server.
sc:|Server to Client.
sr:|Server to Remote Server, this prefix is used by proxies.
rs:|Remote Server to Server, this prefix is used by proxies.
x:|Application specific identifier.

The identifier `h-emotion` thus refers to the emotion expressed by the human while `a(Referer)` refers to the `referer`: field of the reply by an artificial intelligence. The identifier `a-ip` refers to the artificial intelligence's ip address.

## Identifiers.
The following identifiers do not require a prefix

Field Identifiers:|Definitions:
:-----:|:------
date:|Date at which transaction completed, field has type \<date\>
time:|Time at which transaction completed, field has type \<time\>
time-taken:|Time taken for transaction to complete in seconds, field has type \<fixed\>
bytes:|bytes transferred, field has type \<integer\>
cached:|Records whether a cache hit occurred, field has type <integer> 0 indicates a cache miss.

The following identifiers require a prefix

Field Identifiers:|Definitions:
:-----:|:------
ip:|IP address and port, field has type \<address\>
dns:|DNS name, field has type \<name\>
status:|Status code, field has type \<integer\>
comment:|Comment returned with status code, field has type \<text\>
method:|Method, field has type \<name\>
uri:|URI, field has type \<uri\>
uri-stem:|Stem portion alone of URI (omitting query), field has type \<uri\>
uri-query:|Query portion alone of URI, field has type \<uri\>

## Special fields for log summaries.

Analysis tools may generate log summaries. A log summary entry begins with a count specifying the number of times a particular event occurred. For example a therapist may be interested in a count of the number of thoughts of a particular memory with a given `h-emotion` field, but not be interested in recording information about individual requests such as the IP address.

The following field is mandatory and must precede all others:

Field Identifiers:|Definitions:
:-----:|:------:
count:|The number of entries for which the listed data, field has type \<integer\>

The following fields may be used in place of time to allow aggregation of log file entries over intervals of time.

Field Identifiers:|Definitions:
:-----:|:------:
time-from:|Time at which sampling began, field has type \<time\>
time-to:|Time at which sampling ended, field has type \<time\>
interval:|Time over which sampling occurred in seconds, field has type \<integer\>

### Entries

This section describes the data formats for log file field entries. These formats are chosen so as to avoid ambiguity, minimize the difficulty of generation and parsing and provide for human readability.

Each logfile entry consists of a sequence of fields separated by whitespace and terminated by a CR or CRLF sequence. The meanings of the fields are defined by a preceding #Fields directive. If a field is omitted for a particular entry a single dash "-" is substituted.

Log file parsers should be tolerant of errors. If an entry contains corrupt data or is terminated unexpectedly the parser should resynchronize using the end of line marker and continue to parse the following entries. Entries must not contain any ASCII control characters.

```
<entry> = *<field> <end-of-line>

<field> = <integer> | <fixed> | <uri> | <date> | <time> | <string>

<digit = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9"
```

### Integer

```
<integer> = 1*<digit>
```

Integers are represented as a sequence of digits.

### Fixed Format Float

```
<fixed> = 1*<digit> [. *<digit>]
```

### URI

A URI as specified by [RFC1738](http://ds.internic.net/rfc/rfc1738.txt), relative URIs are specified by [RFC1808](http://ds.internic.net/rfc/rfc1808.txt). URIs cannot by definition include whitespace or ASCII control characters. Consequently no ambiguity arises from their use.

### Date

```
<date>  = 4<digit> "-" 2<digit> "-" 2<digit>
```

Dates are recorded in the format `YYYY-MM-DD` where YYYY, MM and DD stand for the numeric year, month and day respectively. All dates are specified in GMT. This format is chosen to assist collation using sort.

### Time

```
<time>  = 2<digit> ":" 2<digit> [":" 2<digit> ["." *<digit>]
```

Times are recorded in the form HH:MM, HH:MM:SS or HH:MM:SS.S where HH is the hour in 24 hour format, MM is minutes and SS is seconds. All times are specified in GMT.

### String

```
<string>        = '"' *<schar> '"'

<schar> = xchar | '"' '"'
```

Strings are output in quoted form. If a string contains a quotation character the character is repeated. This format is unambiguous since fields are by definition separated by whitespace. The dash character must not be used as an abbreviation for an empty string.

No mechanism for incorporating control characters is defined.

### Text

```
<text>  =  <char>*
```

The text field is used only by directives.

### Name

```
<name>  =  <alpha> [ "." *<alpha> ]
```

DNS name.

### Address

```
<name>  =  <integer> [ "." *<integer> ] [ ":" <integer> ]
```

Numeric IP address and optional port specifier.

## Acknowledgments
Robert Thau provided useful advice and some code. John Mallery and Roger Hurwitz helped develop many of the ideas.

## References

Reference Titles:|Citations:
:-----:|:------
\[CLF\]|[_The Common Logfile Format_](http://www.w3.org/pub/WWW/Daemon/User/Config/Logging.html#common_logfile_format), A. Luotonen. World Wide Web Consortium, July 1995, ht<span>tp://</span>www.<span>w3.</span>org/pub/WWW/Daemon/User/Config/Logging.html
\[ELF\]|[_Extended Log File Format_](http://www.w3.org/pub/WWW/TR/WD-logfile-960221.html), P. M. Hallam-Baker, B. Behlendorf. World Wide Web Consortium, 23 March 1996. This edition of the Extended Log File Format Working Draft is ht<span>tp://</span>www.<span>w3.</span>org/pub/WWW/TR/WD-logfile-960221.html. The [latest edition of Extended Log File Format](https://www.w3.org/TR/WD-logfile.html) is available at ht<span>tp://</span>www.<span>w3.</span>org/TR/WD-logfile.html
