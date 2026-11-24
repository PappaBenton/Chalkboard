# Scratchpad

Quick capture for thoughts, questions, and ideas that aren't organized yet.

## New Notes

<!-- Add new notes below this line. -->

## Unsorted Ideas

- 

## Questions to Revisit

- 

| eval DelTime=if(EventCode==4699, _time, null())
| eval ChgTime=if(EventCode==4702, _time, null())
| eventstats max(DelTime) as LastDeleted max(ChgTime) as LastChanged count(ChgTime) as ChangeEvents by host Task_Name
| where EventCode!=4699
| eval TaskStatus=case(
    isnotnull(LastDeleted) AND LastDeleted>=_time, "Deleted " . strftime(LastDeleted, "%Y-%m-%d %T"),
    true(), "No 4699 in search window - assume still registered")
| eval LaterChange=if(isnotnull(LastChanged) AND LastChanged>_time, "Modified " . strftime(LastChanged, "%Y-%m-%d %T"), "none")
| fields - DelTime ChgTime LastDeleted LastChanged

| eval ScriptText=if(DecodedCommand!="N/A", DecodedCommand, Arguments)
| rex field=ScriptText max_match=0 "(?i)(?<RegChanges>\b(?:(?:Set|New)-ItemProperty|reg(?:\.exe)?\s+add)\s[^;\r\n|&]+)"
| eval PolicyKeyTouched=if(match(ScriptText, "(?i)\\\\Policies\\\\"), "yes", "no")
| fillnull value="none" RegChanges
| fields - ScriptText

| table _time host Account_Name EventCode Task_Name Command Arguments DecodedCommand RegChanges PolicyKeyTouched TriggerType StartBoundary Interval TaskStatus LaterChange ChangeEvents
