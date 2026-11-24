# Scratchpad

Quick capture for thoughts, questions, and ideas that aren't organized yet.

## New Notes

<!-- Add new notes below this line. -->

## Unsorted Ideas

- 

## Questions to Revisit

- 
| rex field=Triggers max_match=0 "<(?<TriggerType>\w+Trigger)[\s>]"
| rex field=Triggers "<StartBoundary>(?<StartBoundary>[^<]+)</StartBoundary>"
| rex field=Triggers "<Interval>(?<RepeatInterval>[^<]+)</Interval>"
| rex field=Triggers "<(?<CalUnit>Days|Weeks)Interval>(?<CalInterval>\d+)</"
| eval Interval=coalesce(RepeatInterval, if(isnotnull(CalInterval), "every " . CalInterval . " " . lower(CalUnit), null()))
| eval TriggerType=mvjoin(TriggerType, ", ")
| fillnull value="none" Interval TriggerType StartBoundary
| fields - RepeatInterval CalUnit CalInterval
