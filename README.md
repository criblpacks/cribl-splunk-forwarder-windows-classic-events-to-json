# **Cribl Splunk Forwarder Windows Classic Events to JSON**
----

This pack is designed to transform Splunk Windows Classic events to JSON, reduce event sizes, be compliant with the Splunk Common Information Model (CIM) and maintain backwards compatibility with:

* Splunk Add-on for Microsoft Windows: https://splunkbase.splunk.com/app/742
* Splunk Common Information Model (CIM): https://splunkbase.splunk.com/app/1621

Reduction is diff

---
## **Requirements Section**

### **props.conf**

Add the following stanza to a **props.conf** file on the **Splunk search head**: `$SPLUNK_HOME/etc/system/local/props.conf` to override the Splunk Add-On settings for the Windows sources.  The props.conf can also be deployed in a new app using the Splunk deployment server.

```
[source::WinEventLog...]
KV_MODE = auto
priority = 1
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
## **Important - Ensure you follow the Lookup steps below to avoid any data loss on Pack updates!**

### **TEMPLATE_EventCode_Passthru.csv**
1. Some events contain Messages with code, like SQL, PowerShell, or other scripts in the Message and parsing is not necessary.
1. This lookup table is used to clean up whitespace to provide a small event reduction while allowing the event to passthru without JSON transformation to the Destination.
1. Follow the steps below to preserve your changes on any Pack updates
1. Open the **`TEMPLATE_EventCode_Passthru.csv`** in the Pack Knowledge.
1. Select the Text option by the Edit Mode.
1. Copy and paste the entire contents of the lookup table to memory or to a new **`EventCode_Passthru.csv`** text file.
1. Notice we removed the word **Template**.
1. Close the Pack lookup table.
1. In the **`EventCode_Passthru.csv`** text file, modify the content to suit your requirements and save this file locally.
1. Create a new lookup table within the Pack by selecting the New Lookup File button.  There are two options to choose from:
	1. Choose **Upload a New File** using the file you created.
	1. Select **Create with Text Editor** option, Filename: **`EventCode_Passthru.csv`** and paste the contents.
	1. Select **Save**
1. Under the **Pack Knowledge/Global Variables** adjust the value to: **`EventCode_Passthru.csv`**
1. By selecting Save, Commit & Deploy, you are creating a local copy of the Pipelines persist on Pack updates.

### **TEMPLATE_Message_EventCode_LogName.csv**
1. This lookup table allows you drop the Message field if it is not required.
1. Open the **`TEMPLATE_Message_EventCode_LogName.csv`** in the Pack Knowledge.
1. Select the Text option by the Edit Mode.
1. Copy and paste the entire contents of the lookup table to memory or to a new **`Message_EventCode_LogName.csv`** text file.
1. Notice we removed the word **Template**.
1. Close the Pack lookup table.
1. In the **`Message_EventCode_LogName.csv`** text file, modify the content to suit your requirements and save this file locally.
1. Create a new lookup table within the Pack by selecting the New Lookup File button.  There are two options to choose from:
	1. Choose **Upload a New File** using the file you created.
	1. Select **Create with Text Editor** option, Filename: **`Message_EventCode_LogName.csv`** and paste the contents.
	1. Select **Save**
1. Under the **Pack Knowledge/Global Variables** adjust the value to: **`EventCode_LogName.csv`**
1. By selecting Save, Commit & Deploy, you are creating a local copy of the Pipelines persist on Pack updates.

### **TEMPLATE_Drop_Fields.csv**
By default, all arrays are extracted to top level fields for CIM compliance mapping in Splunk.
This lookup table is used to drop unnecessary top level fields.
Simply add the EventCode to the "EventCode" column and comma separated field names that can be dropped to the "drops" column.
1. Follow the steps below to preserve your changes on any Pack updates
1. Open the **`TEMPLATE_Drop_Fields.csv`** in the Pack Knowledge.
1. Select the Text option by the Edit Mode.
1. Copy and paste the entire contents of the lookup table to memory or to a new **`Drop_Fields.csv`** text file.
1. Notice we removed the word **Template**.
1. Close the Pack lookup table.
1. In the **`Drop_Fields.csv`** text file, modify the content to suit your requirements and save this file locally.
1. Create a new lookup table within the Pack by selecting the New Lookup File button.  There are two options to choose from:
	1. Choose **Upload a New File** using the file you created.
	1. Select **Create with Text Editor** option, Filename: **`Drop_Fields.csv`** and paste the contents.
	1. Select **Save**
1. Under the **Pack Knowledge/Global Variables** adjust the value to: **`Drop_Fields.csv`**
1. By selecting Save, Commit & Deploy, you are creating a local copy of the Pipelines persist on Pack updates.

---
## **Release Notes**
---
**1.1.4** - 2022-03-31: Pull EventCode and SourceName to top level fields to ensure that signature_id is created.

**1.1.3** - 2022-03-31: The Search-time operation sequence of Splunk will perform the transforms before the KV_MODE.  Since the Windows TA isn't designed for JSON, the recommended approach is to send across the top level index-time fields.

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
parent_process service_path signature signature_message vendor_privilege Privileges
```

**1.1.2** - 2022-03-27: Added Tweaks for some EventCodes in Eval Functions 16-19.  Added Try/Catch to all Code functions to clean up Cribl logging.

**1.1.1** - 2022-01-27: Updated Function 32 to support Account_* for multivalued fields.

**1.1.0** - 2022-01-26: Brought back the multi-valued fields in _raw and as top level fields to get around the curly brace issue that Splunk has with search time extractions from JSON arrays.  That's why there is a props.conf file at the top, to auto-extract JSON, and the top level arrays ensure you get both an Account_Name{} field and an Account_Name.  Splunk gives you the ability to dorp curly braces using props.conf, but only with indexed time fields. Splunk should add support in the JSON_TRIM_BRACES_IN_ARRAY_NAMES option of props.conf to support search time extractions without curly braces. 

**1.0.1** - 2022-01-10: Fixed minor pack issue.

**1.0.0** - 2022-01-10: Added a code function before the initial Mask that maps the top level section such as Subject to the Key Names below it, like Account Name. This will make a key such as Subject_Account_Name.  Since the Keys are now unique, this also has a side benefit where arrays were no longer necessary to be extracted at the top level.

**0.9.0** - 2022-01-10: Initial release.


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
