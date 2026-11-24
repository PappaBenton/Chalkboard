# Scratchpad

Quick capture for thoughts, questions, and ideas that aren't organized yet.

## New Notes

<!-- Add new notes below this line. -->

## Unsorted Ideas

- 

## Questions to Revisit

- 
| rex field=CommandLine "(?i)\s[-/]e[a-z]*\s+\"?(?<EncodedCmd>[A-Za-z0-9+/=]{20,})"
| eval b64=split(replace(EncodedCmd, "[^A-Za-z0-9+/]", ""), "")
| eval sextet=mvmap(b64, len(mvindex(split("#ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/", b64), 0)) - 1)
| eval bitstr=mvjoin(mvmap(sextet, (floor(sextet/32)%2) . (floor(sextet/16)%2) . (floor(sextet/8)%2) . (floor(sextet/4)%2) . (floor(sextet/2)%2) . (sextet%2)), "")
| rex field=bitstr max_match=0 "(?<octet>[01]{8})"
| eval byte=mvfilter(mvmap(octet, tonumber(octet, 2)) != 0)
| eval DecodedCmd=urldecode(mvjoin(mvmap(byte, "%" . printf("%02X", byte)), ""))
| fillnull value="N/A" DecodedCmd
| fields - b64 sextet bitstr octet byte
