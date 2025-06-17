# **Cribl Splunk Forwarder Windows Classic Events to JSON**
----

This pack is designed to transform Splunk Windows Classic events to JSON and reduce event sizes.

---
## **Requirements Section**

Install or upload the following App on the **Splunk search head** to ensure all fields are properly extracted.

* `$SPLUNK_HOME/etc/apps/ZZ_Splunk_Windows_TA_Cribl`

```
ZZZ_Splunk_Windows_TA_Cribl
├── default
│   ├── app.conf
│   ├── props.conf
├── metadata
│   └── default.meta

--------------------
app.conf
[install]
is_configured = 0

[ui]
is_visible = 0
label = ZZZ_Splunk_Windows_TA_Cribl

[launcher]
author = Your Name
description = 
version = 1.2.2

--------------------
props.conf
[source::XmlWinEventLog...]
KV_MODE = auto
priority = 1

[source::WinEventLog...]
KV_MODE = auto
priority = 1

--------------------
default.meta
[]
export=system
access = read : [ * ], write : [ admin, power ]

--------------------
```

```
______________________________________________________________________________
|        Search Time          |                                              |
|      Operation Order        |             Operation name                   |            
| --------------------------- | -------------------------------------------- |
|                           1 | Role-based field filtering                   |
|                           2 | Inline field extraction (no field transform) |
|                           3 | Field extraction that uses a field transform |
|                           4 | Automatic key-value field extraction         |
|                           5 | Field aliasing                               |
|                           6 | Calculated fields                            |
|                           7 | Lookups                                      |
|                           8 | Event types                                  |
|                           9 | Tags                                         |
------------------------------------------------------------------------------
```

Per EventCode, the following fields are preserved at the top level to ensure that transforms continue to work as expected:

```
_raw _time cribl_pipe cribl_breaker index source sourcetype host hf punct date_* time*pos* before*
Account_Domain Account_Name Caller_Computer_Name Caller_Domain Caller_Logon_ID Caller_Machine_Name
Caller_User_Name Change_Type Client_Address Client_Domain Client_Logon_ID Client_Machine_Name Client_User_Name
ComputerName Creator_Process_Name Description Domain EventData_Xml EventID EventRecordID FileName File_Name File_Path
Group_Domain Group_Type_Change Image_File_Name IpAddress IpPort KeyFilePath LogFileCleared_Xml
LogonType Logon_Account Logon_ID Logon_account MemberName Member_ID Member_Name Message New_Account_Name
New_Domain New_Process_Name ObjectName Object_Name Primary_Domain Primary_User_Name PrivilegeList
Process_Command_Line RenderingInfo_Xml Security_ID Source_Network_Address Source_Workstation SubStatus
SubjectDomainName SubjectLogonId SubjectUserName Supplied_Realm_Name System_Props_Xml TargetDomainName
TargetProcessName TargetServerName TargetUserName Target_Account_ID Target_Account_Name Target_Domain
Target_Process_Name Target_Server_Name Target_User_Name TokenElevationType Token_Elevation_Type User
UserData_Xml User_ID User_Name Workstation WorkstationName Workstation_Name new_process nt_host param1
parent_process process_name service_path signature signature_message vendor_privilege Privileges Computer
```

### **Reloading Splunk configurations**
1. Restart Splunk is the easiest option but people might not want to restart the Search Head.
1. Reload Search Head Config Files via Search: In Splunk run a search ending with **`| extract reload=t`**
1. Reload Search Head Config Files via URL: In Splunk change the URL of **`http://yoursplunksearchhead/en-US/debug/refresh`**, hit return and select the refresh button. Might take a little bit to complete.
1. Clear Search Head Cache via URL: In Splunk change the URL of **`http://yoursplunksearchhead/en-US/_bump`**, hit return and select the refresh button.

---
## **Using The Pack**
To use this Pack, follow these steps:

1. Install the pack
2. Create a Route/Filter to send all Windows Classic Events through the Pack


---
## **Release Notes**
---
**1.2.6** - 2025-06-17: Fixed release date for 1.2.5 (Thanks Matthew!)

**1.2.5** - 2025-06-07: Added Try/Catch to Long Key Names Code Function

**1.2.4** - 2025-01-07: Syntax fix.

**1.2.3** - 2025-01-06: Pack hygiene, clean up some samples, delete extra pipelines, etc.

**1.2.2** - 2024-07-03: Two functions, Code to create the object.  Still has some nesting and will need to be adjusted based on props/transforms to deal with the multiple AccountName and other fields that have more than one value.

**1.2.1** - 2024-03-27: Fields changed in final Eval to keep EventID in _raw because tstats was having issues searching by EventID as indexed field.  Other option could be to just create a fields.conf entry to handle any indexed fields passed from Cribl. Adjusted filter in Minimum Conversion Route to also skip another EventCode.

**1.2.0** - 2023-11-20: Adjust README and note the search-time reverse order of precedence.

**1.1.9** - 2023-06-19: Fix XML_Messages pipeline for Classic Events, add Code Function to all pipelines to remove duplicate keys that exist in _raw when they are also in top level fields.

**1.1.8** - 2023-05-05: Sample cleanup.

**1.1.7** - 2023-05-05: Additional whitespace cleanup again.

**1.1.6** - 2023-05-05: Fix Powershell whitespace cleanup.

**1.1.5** - 2023-04-20: Separate the Pack Routes for PowerShell, XML in Message, and the rest of the events into their own Routes and removed all Final Flags from the pipelines. Reduced the lookup tables to a single allow list by EventCode.

**1.1.4** - 2023-03-31: Pull EventCode and SourceName to top level fields to ensure that signature_id, src and dst fields are created.

**1.1.3** - 2023-03-31: The Search-time operation sequence of Splunk will perform the transforms before the KV_MODE.  Since the Windows TA isn't designed for JSON, the recommended approach is to send across the top level index-time fields.

```
______________________________________________________________________________
|        Search Time          |                                              |
|      Operation Order        |             Operation name                   |            
| --------------------------- | -------------------------------------------- |
|                           1 | Role-based field filtering                   |
|                           2 | Inline field extraction (no field transform) |
|                           3 | Field extraction that uses a field transform |
|                           4 | Automatic key-value field extraction         |
|                           5 | Field aliasing                               |
|                           6 | Calculated fields                            |
|                           7 | Lookups                                      |
|                           8 | Event types                                  |
|                           9 | Tags                                         |
------------------------------------------------------------------------------
```
Per EventCode, the following fields are preserved at the top level to ensure that transforms continue to work as expected:
```
_raw* _time* cribl_pipe* cribl_breaker* index source sourcetype host hf* punct* date_* time*pos* before* Account_Domain* Account_Name* Caller_Computer_Name* Caller_Domain* Caller_Logon_ID* Caller_Machine_Name* Caller_User_Name* Change_Type* Client_Address* Client_Domain* Client_Logon_ID* Client_Machine_Name* Client_User_Name* ComputerName* Creator_Process_Name* Description* Domain* EventData_Xml* EventID* EventRecordID* FileName* File_Name* File_Path* Group_Domain* Group_Type_Change* Image_File_Name* IpAddress* IpPort* KeyFilePath* LogFileCleared_Xml* LogonType* Logon_Account* Logon_ID* Logon_account* MemberName* Member_ID* Member_Name* New_Account_Name* New_Domain* New_Process_Name* ObjectName* Object_Name* Primary_Domain* Primary_User_Name* PrivilegeList* Process_Command_Line* RenderingInfo_Xml* Security_ID* Source_Network_Address* Source_Workstation* SubStatus* SubjectDomainName* SubjectLogonId* SubjectUserName* Supplied_Realm_Name* System_Props_Xml* TargetDomainName* TargetProcessName* TargetServerName* TargetUserName* Target_Account_ID* Target_Account_Name* Target_Domain* Target_Process_Name* Target_Server_Name* Target_User_Name* TokenElevationType* Token_Elevation_Type* User* UserData_Xml* User_ID* User_Name* Workstation* WorkstationName* Workstation_Name* new_process* nt_host* param1* parent_process* service_path* signature* signature_message* vendor_privilege* Privileges* EventCode SourceName
```

**1.1.2** - 2023-03-27: Added Tweaks for some EventCodes in Eval Functions 16-19.  Added Try/Catch to all Code functions to clean up Cribl logging.

**1.1.1** - 2023-01-27: Updated Function 32 to support Account_* for multivalued fields.

**1.1.0** - 2023-01-26: Brought back the multi-valued fields in _raw and as top level fields to get around the curly brace issue that Splunk has with search time extractions from JSON arrays.  That's why there is a props.conf file at the top, to auto-extract JSON, and the top level arrays ensure you get both an Account_Name{} field and an Account_Name.  Splunk gives you the ability to dorp curly braces using props.conf, but only with indexed time fields. Splunk should add support in the JSON_TRIM_BRACES_IN_ARRAY_NAMES option of props.conf to support search time extractions without curly braces. 

**1.0.1** - 2023-01-10: Fixed minor pack issue.

**1.0.0** - 2023-01-10: Added a code function before the initial Mask that maps the top level section such as Subject to the Key Names below it, like Account Name. This will make a key such as Subject_Account_Name.  Since the Keys are now unique, this also has a side benefit where arrays were no longer necessary to be extracted at the top level.

**0.9.0** - 2023-01-10: Initial release.


## **Contributing to the Pack**
---
Discuss this pack on our Community Slack channel [#packs](https://cribl-community.slack.com/archives/C021UP7ETM3).

## **Contact**
---
* Email: <david@cribl.io>
* Slack: [david (cribl.io)](https://cribl-community.slack.com/team/U01C35EMQ01)

## **License**
---
This Pack uses the following license: [`Apache 2.0`](https://github.com/criblio/appscope/blob/master/LICENSE).
