# Adding Headers-Quotes-Line Feeds to Eventgen Events

James H Baxter  
Jan 17 2022  

Jupyter Notebook:
https://github.com/machinedatainsights/adding_headers_quotes_linefeeds_to_eventgen_events/blob/main/Add%20Headers%20-%20Quotes%20-%20Line%20Feeds%20to%20Eventgen%20Events.ipynb

PDF:
https://github.com/machinedatainsights/adding_headers_quotes_linefeeds_to_eventgen_events/blob/main/Adding%20Headers-Quotes-Line%20Feeds%20to%20Eventgen%20Events.pdf

## Introduction

This code was developed to modify events which were originally ingested via the Splunk Add-on for Linux and Unix and have been exported from a Splunk search so that they can be used with Splunk Eventgen to create artifical events of this type.  

When you export sample Linux/Unix OS-related events (cpu, disk, interfaces, etc.) from Splunk, you get events with just the data fields. What isn't obvious at first is that when these events were generated & sent to Splunk indexers by a forwarder, they originally included a header (for each event) and extra line feeds. When you see the events in a Splunk search, they only include the data fields - no header (vstat being the only exception I'm aware of) - but the fields have been properly parsed and identified (CPU & such) and appear in the left-hand 'Interesting Fields' list in Splunk (using Smart or Verbose Mode). An initial investigation of the props/transforms in the Add-on doesn't make it obvious where/how these fields were extracted or identified - but there are field aliases and evals for creating additional fields. It's only when you look at the scripts in /bin (cpu.sh, for example) that you see that each event was created with a smaller number of specific fields, and included a header. In many cases, there is also an extra line feed added. Doing a comparison of the various .sh files in /bin and another look through the props file and this all starts to make sense.  

So - after exporting some sample Linux/Unix events from Splunk, you have to edit those events to add the header and extra line feeds - recreating the events in the format they were in when sent to Splunk for indexing - before you can use them with Eventgen to create artifical events. Otherwise, the props/transforms from the Splunk Add-on for Linux/Unix will not properly parse the events.  

This code adds the headers and extra line feeds to events from a provided sample file, and creates a new output sample file (so that the origin file is not modified). It also adds quotes around the events, which may nor may not be needed but doesn't seem to hurt.

The other problem with these Linux/Unix events is that their timestamps (as they appear in Splunk searches) were created at index-time - the events themselves did not include a timestamp. When you export these events for use with Eventgen, they again do not include a timestamp. There are two ways of dealing with this situation that I'm aware of:

1. Use the mode = sample option in eventgen.con, which blasts all of the events in your sample file out at once; Splunk will give them an index-time timestamp - but they be all have the same timestamp per generation interval. Example:  

```text
[linux_os_events_10000.csv]
disabled = 0
mode = sample
timeField = _time
sampletype = csv
interval = 60
earliest = -60s
latest = now
```

2. My preference is to use mode = replay and the timeField = ```_time``` option, which requires you to include the ```_time``` field in the Splunk export:

```text
[linux_os_events_10000.csv]
disabled = 0
mode = replay
timeField = _time
sampletype = csv
interval = 60
earliest = -60s
latest = now
```

The search for exporting these events from Splunk is (substituting the correct index(es) for your environment):  

```text
index=linux_os
| reverse
| table index,host,source,sourcetype,_time,_raw

Exclude the _time column if you are not going to use this approach. Instead, use:

index=linux_os
| reverse
| table index,host,source,sourcetype,_raw

and remove the timeField = _time entry from the stanza.
```