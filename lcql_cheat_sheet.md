# LCQL Query Options

**Format:**  TIMEFRAME \| SENSORS \| EVENTS \| FILTER \| PROJECTION *(optional)*   
**Example:**  -24h \| plat == windows \| \* \| event/\* contains 'psexec'   

| **Section** | **Description** | 
| :----- | :----- |
| TIMEFRAME | Time range the query applies to. Can be either relative such as -1h or -30m or absolute such as 2022-01-22 10:00:00 to 2022-01-25 14:00:00 |
| SENSORS   | The set of sensors to query. Can be either \* for all sensors, a list of space separated SIDs like *111-... 222-... 333-...* or a [SENSOR SELECTOR](https://docs.limacharlie.io/v1/docs/sensors-reference-sensor-selector-expressions) like *plat == windows* |
| EVENTS | The list of events to include in the query, space separated like NEW_PROCESS DNS_REQUEST or \* to go over all event types.         |    |
| FILTER | The actual query filter. The filters are a series of statements combined using *and/or* that can be associated with parenthesis (()). String literals, when used, can be double-quoted to be case insensitive or single-quoted to be case sensitive. Selectors behave like D&R rules, for example: event/FILE_PATH |    |
| PROJECTION          | ***Optional***.  A list of fields to extract from the results with a possible alias, like: event/FILE_PATH as path event/USER_NAME AS user_name event/COMMAND_LINE. The Projection can also support a grouping functionality by adding GROUP BY(field1 field2 ...) at the end of the projection statement.         |    |

|**Operator** | **Example** |
| :----------- | :----------- |
| is | event/FILE_PATH is "c:\\windows\\calc.exe" |
| == | event/FILE_PATH == "c:\\windows\\calc.exe" |
| is not | event/FILE_IS_SIGNED is not 0 |
| !=| event/FILE_IS_SIGNED != 0 |
| contains | event/FILE_PATH contains 'evil' |
| not contains | event/FILE_PATH not contains 'system32' |
| matches | event/FILE_PATH matches ".\*system\[0-9a-z\].\*" |
| not matches | event/FILE_PATH not matches ".\*system\[0-9a-z\].\*" |
| starts with | event/FILE_PATH starts with "c:\\windows" |
| not starts with | event/FILE_PATH not starts with "c:\\windows" |
| ends with | event/FILE_PATH ends with '.exe' |
| not ends with | event/FILE_PATH not ends with '.exe' |
| cidr | event/NETWORK_CONNECTIONS/IP_ADDRESS cidr "10.1.0.0/16" |
| is lower than | event/NETWORK_CONNECTIONS/PORT is lower than 1024 |
| is greater than | event/NETWORK_CONNECTIONS/PORT is greater than 1024 |
| is platform | is platform "windows" |
| is not platform | is not platform "linux"|
| is tagged | is tagged "vip" |
| is not tagged | is not tagged "vip" |
| is public address | event/NETWORK_CONNECTIONS/IP_ADDRESS is public address |
| is private address | event/NETWORK_CONNECTIONS/IP_ADDRESS is private address |
| scope | event/NETWORK_CONNECTIONS scope (event/IP_ADDRESS is public address and event/PORT is 443) |
| with child / with descendant / with events | event/FILE_PATH contains "evil" with child (event/COMMAND_LINE contains "powershell") |  


# CLI Query Components
**Installation:** pip install limacharlie  
**Execution:** limacharlie query  
**Example:** limacharlie query --query "-24h \| plat == windows \| DNS_REQUEST \| event/DOMAIN_NAME contains 'google' \| event/DOMAIN_NAME as domain COUNT(event) as count GROUP BY(domain)"   
| **Option**   | **Description** | 
| ------------ | --------------- |
| --query      | Query to execute in LimaCharlie |
| --limit-event | Limit the number of events evaluated to approximately this number |
| --limit-eval | Limit the number of rule evaluations to *approximately* this number |
| --dry-run    | If set, the request will be simulated and the maximum number of evaluations expected will be returned | 
| --pretty     | Output json in pretty format (in single-query mode) |
| --format     | Print format for interactive mode |
| --out-file   | *In interactive mode*, output log to this file |

<table>
<colgroup>
<col style="width: 42%" />
<col style="width: 57%" />
</colgroup>
<thead>
<tr class="header">
<th colspan="2"><h1>LCQL Examples</h1></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td colspan="2"><strong>General Queries</strong></td>
</tr>
<tr class="even">
<td>Search <em>all</em> event types across <em>all</em> Windows systems
for a particular string showing up in <em>any</em> field</td>
<td>-24h | plat == windows | * | event/* contains 'psexec'</td>
</tr>
<tr class="odd">
<td colspan="2"><strong>Network Telemetry</strong></td>
</tr>
<tr class="even">
<td><strong>Domain Count:</strong> Show all domains resolved by Windows
hosts that contain "google" in the last 10 minutes and the number of
times each was resolved.</td>
<td>-10m | plat == windows | DNS_REQUEST | event/DOMAIN_NAME contains
'google' | event/DOMAIN_NAME as domain COUNT(event) as count GROUP
BY(domain)</td>
</tr>
<tr class="odd">
<td><strong>Domain Prevalence:</strong> Show all domains resolved by
Windows hosts that contain "google" in the last 10 minutes and the
number of unique sensor that has resolved them.</td>
<td>-10m | plat == windows | DNS_REQUEST | event/DOMAIN_NAME contains
'google' | event/DOMAIN_NAME as domain COUNT_UNIQUE(routing/sid) as
count GROUP BY(domain)</td>
</tr>
<tr class="even">
<td colspan="2"><strong>Process Activity</strong></td>
</tr>
<tr class="odd">
<td><strong>Unsigned Binaries:</strong> Show all unsigned binaries on
Windows hosts with the full file path, hash, and file name, grouped by
the path, hash, and filename for the past 24 hours</td>
<td>-24h | plat == windows | CODE_IDENTITY |
event/SIGNATURE/FILE_IS_SIGNED == 0 | event/FILE_PATH as Path event/HASH
as Hash event/ORIGINAL_FILE_NAME as OriginalFileName COUNT_UNIQUE(Hash)
as Count GROUP BY(Path Hash OriginalFileName)</td>
</tr>
<tr class="even">
<td><strong>Process Command Line Args:</strong> Show process arguments
on Windows hosts with the command line, full file path, and hostname for
the past hour</td>
<td>-1h | plat == windows | NEW_PROCESS EXISTING_PROCESS |
event/COMMAND_LINE contains "psexec" | event/FILE_PATH as path
event/COMMAND_LINE as cli routing/hostname as host</td>
</tr>
<tr class="odd">
<td><strong>Stack Children by Parent:</strong> Show all processes on
Windows hosts that were spawned by “cmd.exe” and group them by the
parent process and child process over the past 12 hours</td>
<td>-12h | plat == windows | NEW_PROCESS | event/PARENT/FILE_PATH
contains "cmd.exe" | event/PARENT/FILE_PATH as parent event/FILE_PATH as
child COUNT_UNIQUE(event) as count GROUP BY(parent child)</td>
</tr>
<tr class="even">
<td colspan="2"><strong>Windows Event Log (WEL)</strong></td>
</tr>
<tr class="odd">
<td><strong>%COMSPEC% in Service Path:</strong> Show services that were
installed to Windows hosts that contain “COMSPEC” over the past 12
hours</td>
<td>-12h | plat == windows | WEL | event/EVENT/System/EventID == "7045"
and event/EVENT/EventData/ImagePath contains "COMSPEC"</td>
</tr>
<tr class="even">
<td><strong>Overpass-the-Hash:</strong> Show potential overpass-the-hash
attacks on Windows hosts for the past 12 hours</td>
<td>-12h | plat == windows | WEL | event/EVENT/System/EventID == "4624"
and event/EVENT/EventData/LogonType == "9" and
event/EVENT/EventData/AuthenticationPackageName == "Negotiate" and
event/EVENT/EventData/LogonProcess == "seclogo"</td>
</tr>
<tr class="odd">
<td><p><strong>Taskkill from a Non-System Account:</strong> Show
“taskkill” executions from non-system accounts on Windows hosts over the
past 12 hours.</p>
<p><em>Note: Requires process auditing to be enabled</em></p></td>
<td>-12h | plat == windows | WEL | event/EVENT/System/EventID == "4688"
and event/EVENT/EventData/NewProcessName contains "taskkill" and
event/EVENT/EventData/SubjectUserName not ends with "!"</td>
</tr>
<tr class="even">
<td><strong>Logons by Specific LogonType:</strong> Show remote
interactive logons to Windows hosts for the past 24 hours</td>
<td>-24h | plat == windows | WEL | event/EVENT/System/EventID == "4624"
AND event/EVENT/EventData/LogonType == "10"</td>
</tr>
<tr class="odd">
<td><strong>Stack/Count All LogonTypes by User:</strong> Show logons for
Windows hosts grouped by the username and the logon type for the past 24
hours</td>
<td>-24h | plat == windows | WEL | event/EVENT/System/EventID == "4624"
| event/EVENT/EventData/LogonType AS LogonType
event/EVENT/EventData/TargetUserName as UserName COUNT_UNIQUE(event) as
Count GROUP BY(UserName LogonType)</td>
</tr>
<tr class="even">
<td><strong>Failed Logons:</strong> Show all failed logons to Windows
hosts for the past hour</td>
<td>-1h | plat==windows | WEL | event/EVENT/System/EventID == "4625" |
event/EVENT/EventData/IpAddress as SrcIP event/EVENT/EventData/LogonType
as LogonType event/EVENT/EventData/TargetUserName as Username
event/EVENT/EventData/WorkstationName as SrcHostname</td>
</tr>
<tr class="odd">
<td colspan="2"><strong>GitHub Telemetry</strong></td>
</tr>
<tr class="even">
<td><strong>GitHub Protected Branch Override:</strong> Show all the
GitHub branch protection override (force pushing to repo without all
approvals) in the past 12h that came from a user outside the United
States, with the repo, user and number of infractions</td>
<td>-12h | plat == github | protected_branch.policy_override |
event/public_repo is false and event/actor_location/country_code is not
"us" | event/repo as repo event/actor as actor COUNT(event) as count
GROUP BY(repo actor)</td>
</tr>
</tbody>
</table>
