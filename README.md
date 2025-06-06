# Austin LimaCharlie Workshop

This guide will provide you with a step-by-step of all the commands we will use throughout this workshop. Please reference it as we move forward. If you have questions, feel free to ask your group moderator.

> [!TIP] 
> **Terms Used in this Workshop**
> * **Sensor** - A tool that collects and acts on data across endpoints, networks, and cloud environments to provide full visibility and security
> * **Endpoint Agent** - A lightweight sensor installed on an endpoint that collects security telemetry and enables real-time monitoring and response.
> * **Adapter** - Flexible data ingestion mechanisms for both on-premise and cloud environments
> * **Extension** -  A handy add-on that boosts the platform’s features by hooking into third-party tools and custom setups, and once created, any subscribing organization can use them across their environments.
> * **Organization** - A tenant within LimaCharlie
> * **OID** - The organizations unique ID
> * **SID** - The sensor's unique ID

## Setting Up LimaCharlie

In this lab we will be configuring a LimaCharlie organization, loading in sample data, building detection rules, and utilizing response actions to create an alert when a detection is made.

### Tools We'll Use

- [LimaCharlie](https://app.limacharlie.io)
- [LimaCharlie Docs](https://docs.limacharlie.io/docs)
- [SOCRadar](https://socradar.io/)

### Resources to Download
- [Workshop Files - updateme](https://github.com/lc-cbot/austin-workshop/workshop_files.zip) 
   - Extract the archive after downloading. You should end up with five files, shown here.
      ![Archive Contents](/img/archive_contents.png)
- [LimaCharlie Adapter](https://docs.limacharlie.io/docs/adapter-deployment#:~:text=pre%2Ddefined%20format.-,Adapter%20Binaries,-Software%2Dbased%2C%20or) 
   - Download the adapter specific to your system.
> [!IMPORTANT]
> Make sure you save these files to a convient location as you'll need them throughout the workshop


### Lab 1: Configuring LimaCharlie and Ingesting Logs

To begin, we will need to create an account within LimaCharlie and ingest the first set of logs for this workshop. We will use the community tier of LimaCharlie for this workshop, which will be yours to keep for free after the workshop. The community edition is limited to two sensors per organization and two organizations (four sensors in total).

1. [Create a free LimaCharlie account](https://free.limacharlie.io) (please select Austin Workshop for the first option, feel free to use a burner email)
2. Once you have an organization setup in LimaCharlie, log in to your LimaCharlie account and click on your organization's name
3. First, we need to create an installation key that will be used to authenticate the sensor that will be used to ingest the lab data. To access your installation keys, click on "Sensors" and then select "Installation Keys"
4. Create a new installation key by clicking the "Create Installation Key" button
5. Add a description that will allow you to associate the installation key with the intended purpose
6. The installation key allows for sensors to automatically be tagged when they authenticate to LimaCharlie. For this workshop, use the tag "lab-data" and then click "Create"
> [!TIP]
> Tags can be used for a variety of purposes, including classifying endpoints, automating detection and response actions, creating powerful workflows, and triggering automations
7. Get the new installation key's GUID by clicking the copy icon in "Adapter Key" column for the installation key you created. 
> [!IMPORTANT] 
> This will be used in the next few steps, so it's recommended to put this into a text file to make referencing it easier
8. You'll also need your organizations ID (OID), so copy it from the URL and add it to your text file
> [!TIP] 
> The OID is also available by viewing a sensor's details, but grabbing it from the URL is simpler. 
9. Next, on the sidebar, click "Sensors" and then click on "External Adapters" 
> [!NOTE]
> LimaCharlie offers multiple ways to ingest logs into the platform. For this lab, we're going to read in the file from your system using a local adapter. To simplify the management of local adapters, the External Adapters menu allows you to centralize management of all your on-premise adapters. 
10. Click "Add External Adapter"
11. Give your adapter a descriptive name.
12. Open the LimaCharlie Austin Workshop folder you extracted earlier and open lc-adapter.yaml
13. Change the ```YOUR_OID``` field to the OID you saved in step 8
14. Change the ```YOUR_INSTALL_KEY``` field to the installation key you saved in step 6
15. Change the ```YOUR_FILE_PATH``` field to point to the "sample_data_1.json" file you extracted in the [Resources to Download](#resources-to-download) section
16. Switch back to your web browser where you were creating the external adapter within LimaCharlie
16. Copy and paste the entire configuratin into the "External Adapter Definition" area in your web browser and then click "Create"
17. Copy the GUID into your text editor. You will use this to configure the adapter you downloaded in the [Resources to Download](#resources-to-download) section
18. Open a command prompt on your system and change to the directory where you extracted the archive to
19. Execute the adapter setting the type as ```cloud```, then pass the parameter ```conf_guid``` and paste in the GUID value from the External Adapter configuration we created in step 17 along with the ```oid``` parameter and OID from step 8. The output should look like the following. If you do not see the "BLAHBLAH" line, then check your file_path and make sure it points to correct file and location
20. Switch back to your web browser. On the side menu, click "Sensors" if it's not already open and then select the sensor you created by clicking on the sensor with your hostname. If you cannot find the sensor you created, wait a minute and refresh the page. Verify your sensor shows as online
21. On the sidebar, click on "Query Console"
22. Enter the following query and then click "Submit"
   - ```-2h | * | * | event/PROCESS_ID == 666 and event/PARENT_PROCESS_ID == 31337```
23. Verify data is returned. The results show the logs you ingested in step 19. If you do not see logs after a few minutes, feel free to ask your moderator for assistance. 

:tada: Lab 1 is now complete! 

### Lab 2: Configuring Add-ons

Add-ons, also known as extensions, extend LimaCharlie's capabilities and allow you to add functionality such as managed rulesets, API connections, investigative tools, and threat intelligence feeds. While many add-ons are provided by LimaCharlie, users are encouraged to develop their own and make them available to other users through the add-ons marketplace.  

1. If you are not already logged in to your LimaCharlie account, log in
2. Click "Add-Ons", located at the top-right of the LimaCharlie UI. Feel free to explore the add-ons that are available
>[!NOTE] 
> Some add-ons may have a cost associated with them. This cost is not included in the free tier. 
3. Search for the ```ext-lookup-manager``` add-on and click on it
4. The screen displays information about the extension, the developer, as well as any associated cost. To enable an add-on click on "Subscribe". Add-ons are subscribed on a per-organization basis, so only the organization displayed is subscribed to the add-on
5. On the sidebar, click "All Add-ons" to go back to viewing all available add-ons
6. Using the same process as before, subscribe your organization to the following add-ons. This will prove your organization with a base set of detections:
   - ```ext-sigma```
   - ```ext-snapattack```



### Lab 3: Ingesting Lookups

For this lab, we'll be configuring the Lookup Manager that was enabled in lab 2, which will allow us import lookup lists. These lists can contain various types of information such as malicious hashes and IPs and can be used in detection rules.

### Lab 4: Configuring Outputs

## Writing Detection and Response Rules from Scratch

### Lab 5: Writing a Detection and Response Rule for Known Malicious IPs

### Lab 6: Writing a Detection and Response Rule for Known Malicious Hashes


1. Log into your LimaCharlie account and go to the menu on the left and navigate to "Automation -> D&R Rules"

2. Click the "+ New Rule" button to create a new detection rule

3. Name the rule "First Detection - Test"

4. In the "Detect" window copy and past this YAML detection code:

```yaml
event: DNS_REQUEST
op: is
path: event/DOMAIN_NAME
value: superevildomain.com
```
In the "Response" window enter this YAML response code:

```yaml
- action: report
  name: DNS Hit superevildomain.com
```

Save the detection and ensure it is enabled. 

5. Go to your Windows Virtual Machine in VirtualBox and open a browser and visit the superevildomain.com 

6. After a minute go to your LimaCharlie account and navigate to "Detections" on the left hand navigation. You should see an alert for your new detection rule.

![VM Setup](/img/18_first_detection_alert.png) 

## Lab 4: Adding Windows Event Logs to Our Telemetry

So now we will create a more useful rule for Windows, we will use our EDR agent to detect if Windows Defender is disabled on a system. However, we will first need to add Windows Event Logs (WEL) to the telemetry we collect from the system. 

1. In LimaCharlie on the left side navigation menu go to "Sensors -> Artifact Collection" and click on the "+ Add Artifact Collection Rule" button and give it the name of "windows-logs"

![VM Setup](/img/19_artifact_collection_rule.png) 

2. In the patterns field we will add several Windows Event Logs that our EDR sensor will send to LimaCharlie which we can then write detections for. Enter each of these in the pattern field:

```yaml
wel://application:*
```

```yaml
wel://security:*
```

```yaml
wel://system:*
```

```yaml
wel://Microsoft-windows-Windows Defender/Operational:*
```

Set the "Platform(s)" field to "windows"  and save your Artifact Collection Rule

![VM Setup](/img/20_artifact_rules.png) 

3. Now we will write a new rule that will detect when Windows Defender is disabled, go back to your D&R Rules (Automation -> D&R Rules) and click "+ New Rule" and enter this YAML for the detection logic: 

```yaml
event: WEL
op: is
path: event/EVENT/System/EventID
value: '5001'
```

Then enter this YAML for a basic reporting response: 

```yaml
- action: report
  name: Windows Defender Malware Disabled

```

Save your new detection. 

4. Go into your Windows Virtual Machine in VirtualBox and disable Windows Defender (search Defender in the search box in the main nav)

## Lab 5: Setting up YARA Scans

We will setup a single YARA rule that will run on the endpoint at regular intervals. 

1. In your LimaCharlie account go to "Automation -> YARA Rules" and click on the "+ Add Yara Rule" button

2. You can write your own YARA rule, or leverage rules built by the community, in this exercise we will use [Florian Roth's God Mode YARA rule](https://github.com/Neo23x0/god-mode-rules/blob/master/godmode.yar).

3. In the name field enter "God Mode" and then copy and paste the YARA rule:

```yara

/*
      _____        __  __  ___        __      
     / ___/__  ___/ / /  |/  /__  ___/ /__    
    / (_ / _ \/ _  / / /|_/ / _ \/ _  / -_)   
    \___/\___/\_,_/_/_/__/_/\___/\_,_/\__/    
     \ \/ / _ | / _ \/ _ |   / _ \__ __/ /__  
      \  / __ |/ , _/ __ |  / , _/ // / / -_) 
      /_/_/ |_/_/|_/_/ |_| /_/|_|\_,_/_/\__/  
   
   Florian Roth - v0.8.1 August 2024 - Merry Christmas!

   The 'God Mode Rule' is a proof-of-concept YARA rule designed to 
   identify a wide range of security threats. It includes detections for 
   Mimikatz usage, Metasploit Meterpreter payloads, PowerShell obfuscation 
   and encoded payloads, various malware indicators, and specific hacking 
   tools. This rule also targets ransomware behaviors, such as 
   shadow copy deletion commands, and patterns indicative of crypto mining. 
   It's further enhanced to detect obfuscation techniques and signs of 
   advanced persistent threats (APTs), including unique strings from 
   well-known hacking tools and frameworks. 
*/

rule IDDQD_God_Mode_Rule {
   meta:
      description = "Detects a wide array of cyber threats, from malware and ransomware to advanced persistent threats (APTs)"
      author = "Florian Roth"
      reference = "Internal Research - get a god mode rule set with THOR by Nextron Systems"
      date = "2019-05-15"
      modified = "2024-01-12"
      score = 60
   strings:
      $ = "sekurlsa::logonpasswords" ascii wide nocase           /* Mimikatz Command */
      $ = "ERROR kuhl" wide xor                                  /* Mimikatz Error */
      $ = " -w hidden " ascii wide nocase                        /* Power Shell Params */
      $ = "Koadic." ascii                                        /* Koadic Framework */
      $ = "ReflectiveLoader" fullword ascii wide xor             /* Generic - Common Export Name */
      $ = "%s as %s\\%s: %d" ascii xor                           /* CobaltStrike indicator */
      $ = "[System.Convert]::FromBase64String(" ascii            /* PowerShell - Base64 encoded payload */
      $ = "/meterpreter/" ascii xor                              /* Metasploit Framework - Meterpreter */
      $ = / -[eE][decoman]{0,41} ['"]?(JAB|SUVYI|aWV4I|SQBFAFgA|aQBlAHgA|cgBlAG)/ ascii wide  /* PowerShell encoded code */
      $ = /  (sEt|SEt|SeT|sET|seT)  / ascii wide                 /* Casing Obfuscation */
      $ = ");iex " nocase ascii wide                             /* PowerShell - compact code */ 
      $ = "Nir Sofer" fullword wide                              /* Hack Tool Producer */
      $ = "impacket." ascii                                      /* Impacket Library */
      $ = /\[[\+\-!E]\] (exploit|target|vulnerab|shell|inject)/ nocase  /* Hack Tool Output Pattern */
      $ = "0000FEEDACDC}" ascii wide                             /* Squiblydoo - Class ID */
      $ = "vssadmin delete shadows" ascii nocase                 /* Shadow Copy Deletion via vssadmin - often used in ransomware */
      $ = ".exe delete shadows" ascii nocase                     /* Shadow Copy Deletion via vssadmin - often used in ransomware */
      $ = " shadowcopy delete" ascii wide nocase                 /* Shadow Copy Deletion via WMIC - often used in ransomware */
      $ = " delete catalog -quiet" ascii wide nocase             /* Shadow Copy Deletion via wbadmin - often used in ransomware */
      $ = "stratum+tcp://" ascii wide                            /* Stratum Address - used in Crypto Miners */
      $ = /\\(Debug|Release)\\(Key[lL]og|[Ii]nject|Steal|By[Pp]ass|Amsi|Dropper|Loader|CVE\-)/  /* Typical PDB strings found in malware or hack tools */
      $ = /(Dropper|Bypass|Injection|Potato)\.pdb/ nocase        /* Typical PDP strings found in hack tools */
      $ = "Mozilla/5.0" xor(0x01-0xff) ascii wide                /* XORed Mozilla user agent - often found in implants */
      $ = "amsi.dllATVSH" ascii xor                              /* Havoc C2 */
      $ = "BeaconJitter" xor                                     /* Sliver */
      $ = "main.Merlin" ascii fullword                           /* Merlin C2 */
      $ = "\x48\x83\xec\x50\x4d\x63\x68\x3c\x48\x89\x4d\x10" xor /* Brute Ratel C4 */
      $ = "}{0}\"-f " ascii wide                                 /* PowerShell obfuscation - format string */
      $ = "HISTORY=/dev/null" ascii                              /* Linux HISTORY tampering - found in many samples */
      $ = " /tmp/x;" ascii                                       /* Often used in malicious linux scripts */
      $ = /comsvcs(\.dll)?[, ]{1,2}(MiniDump|#24)/               /* Process dumping method using comsvcs.dll's MiniDump */
      $ = "AmsiScanBuffer" base64 base64wide                     /* AMSI Bypass */
      $ = "AmsiScanBuffer" xor(0x01-0xff)                        /* AMSI Bypass */
      $ = "%%%%%%%%%%%######%%%#%%####%  &%%**#" ascii wide xor  /* SeatBelt */
   condition:
      1 of them
}

```
4. Next we will add the rule to a scanner. In LimaCharlie navigate to "Automation -> YARA Scanners" and click the "+ Add YARA Scanner" 

5. In the drop down you should see the God Mode YARA rule we just created select it. For platforms select "windows" and "linux" and click "Save".

![YARA SCAN](/img/21_yara_scan.png) 


## Lab 6: Recess!
