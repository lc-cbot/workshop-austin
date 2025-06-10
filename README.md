# Austin LimaCharlie Workshop

- [Austin LimaCharlie Workshop](#austin-limacharlie-workshop)
  - [Setting Up LimaCharlie](#setting-up-limacharlie)
    - [Resources We'll Use](#resources-well-use)
    - [Resources to Download](#resources-to-download)
    - [Lab 1: Configuring LimaCharlie and Ingesting Logs](#lab-1-configuring-limacharlie-and-ingesting-logs)
    - [Lab 2: Configuring Add-ons](#lab-2-configuring-add-ons)
    - [Lab 3: Ingesting Lookups](#lab-3-ingesting-lookups)
    - [Lab 4: Configuring Outputs](#lab-4-configuring-outputs)
  - [Writing Detection and Response Rules from Scratch](#writing-detection-and-response-rules-from-scratch)
    - [Lab 5: Writing a Detection and Response Rule for Known Malicious IPs](#lab-5-writing-a-detection-and-response-rule-for-known-malicious-ips)
    - [Lab 6: Writing a Detection and Response Rule for Known Malicious Hashes](#lab-6-writing-a-detection-and-response-rule-for-known-malicious-hashes)


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

### Resources We'll Use

- [LimaCharlie](https://app.limacharlie.io)
- [LimaCharlie Docs](https://docs.limacharlie.io/docs)
- [SOCRadar](https://socradar.io/)
- [LCQL Cheat Sheet](https://github.com/lc-cbot/workshop-austin/blob/main/lcql_cheat_sheet.md)

### Resources to Download
- [Workshop Files](https://github.com/lc-cbot/workshop-austin/blob/main/workshop_files.zip) 
   - Extract the archive after downloading. You should end up with four files, shown here.  
      ![Archive Contents](/img/archive_contents.png)  
- [LimaCharlie Adapter](https://docs.limacharlie.io/docs/adapter-deployment#:~:text=pre%2Ddefined%20format.-,Adapter%20Binaries,-Software%2Dbased%2C%20or) 
   - Download the adapter specific to your system.
- [Austin Workshop Secrets](https://docs.google.com/spreadsheets/d/1pCDzULuTO1EtKKHOel3RyD04GHmLh4ZTjsL5Edha-10/edit?usp=sharing)
  - This contains the URLs and API keys you'll utilize for the workshop. These credentials will only be valid for a short time (< 1 week)
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

12. Copy and paste the following YAML into the "External Adapter Definition" box:
```
sensor_type: file
file:
  client_options:
    identity:
      installation_key: YOUR_INSTALLATION_KEY
      oid: YOUR_OID
    backfill: true
    inactivity_threshold: 0
    no_follow: false
    platform: lc_event
    sensor_seed_key: austin_workshop
    serialize_files: false
  file_path: YOUR_FILE_PATH\\small_sample-1.jsonl
```
13. Change the ```YOUR_OID``` field to the OID you saved in step 8

14. Change the ```YOUR_INSTALL_KEY``` field to the installation key you saved in step 6

15. Change the ```YOUR_FILE_PATH``` field to point to the ```small_sample-1.jsonl``` file you extracted in the [Resources to Download](#resources-to-download) section

16. Switch back to your web browser where you were creating the external adapter within LimaCharlie

16. Copy and paste the entire configuratin into the "External Adapter Definition" area in your web browser and then click "Create"

17. Copy the GUID into your text editor. You will use this to configure the adapter you downloaded in the [Resources to Download](#resources-to-download) section

18. Open a command prompt on your system and change to the directory where you extracted the archive to

19. Execute the adapter setting the type as ```cloud```, then pass the parameter ```conf_guid``` and paste in the GUID value from the External Adapter configuration we created in step 17 along with the ```oid``` parameter and OID from step 8. The output should look like the following. If you do not see the "BLAHBLAH" line, then check your file_path and make sure it points to correct file and location

20. Switch back to your web browser. On the side menu, click "Sensors" if it's not already open and then select the sensor you created by clicking on the sensor with your hostname. If you cannot find the sensor you created, wait a minute and refresh the page. Verify your sensor shows as online

21. On the sidebar, click on "Query Console"

22. Enter the following query and then click "Submit"
```
-2h | * | NETWORK_CONNECTIONS | event/NETWORK_ACTIVITY/?/SOURCE/IP_ADDRESS is public address
```

23. Verify data is returned. The results show the logs you ingested in step 19. If you do not see logs after a few minutes, feel free to ask your moderator for assistance. 

:tada: Lab 1 is now complete! 

### Lab 2: Configuring Add-ons

Add-ons, also known as extensions, extend LimaCharlie's capabilities and allow you to add functionality such as managed rulesets, API connections, investigative tools, and threat intelligence feeds. While many add-ons are provided by LimaCharlie, users are encouraged to develop their own and make them available to other users through the add-ons marketplace.  

1. If you are not already logged in to your LimaCharlie account, log in

2. Click "Add-Ons", located at the top-right of the LimaCharlie UI. Feel free to explore the add-ons that are available
> [!NOTE] 
> Some add-ons may have a cost associated with them. This cost is not included in the free tier. 

3. Search for the ```ext-lookup-manager``` add-on and click on it

4. The screen displays information about the extension, the developer, as well as any associated cost. To enable an add-on click on "Subscribe". Add-ons are subscribed on a per-organization basis, so only the organization displayed is subscribed to the add-on

5. On the sidebar, click "All Add-ons" to go back to viewing all available add-ons

6. Using the same process as before, subscribe your organization to the following add-ons. This will provide your organization with a base set of detections:
   - ```ext-sigma```
   - ```ext-snapattack```

:tada: Lab 2 is now finished. On to lab 3!

### Lab 3: Ingesting Lookups

Lookups are lists that can contain various types of information such as malicious hashes and IPs and can be used in detection rules. These can be manually added, or they can be managed via the lookup manager to simplify adding and maintaining them. The lookup manager will automatically synchronize the list every 24 hours, or the lookups can be manually synchronized by clicking the "Manual Sync" button within the lookup manager extension. 

The lookup manager utilizes [Authenticated Resource Locator (ARL)](https://docs.limacharlie.io/docs/reference-authentication-resource-locator) strings to define where to get the lookup from. ARLs can point to standard websites, websites with authentication, or even specialized resources such as Google Cloud Storage and GitHub repos. For this lab, we'll be configuring the Lookup Manager that was enabled in lab 2, which will allow us easily import and manage lookup lists from an unauthenticated URL.

1. If you are not already logged in to your LimaCharlie account, log in

2. On the sidebar menu, click on "Extensions" and then click "Lookup Manager"
> [!NOTE]
> If you do not see the Lookup Manager under Extensions, ensure you subscribed to it in lab 2

3. Click "Add New Lookup Configuration"

4. Give the lookup a descriptive name that you will reference later
> [!TIP]
> This name will be how you reference the lookup within rules, so make sure to name it something memorable

5. Copy and paste the first ARL from the workshop secrets list you were provided. If you do not have this list, please reach out to your coordinator
   - If you are working on this outside of a LimaCharlie workshop, please feel free to use the following ARLs that are similar to the SOCRadar lists used during the workshop:
      - Malicious IPs: ```[github,romainmarcoux/malicious-ip/full-40k.txt]```
      - Malicous Hashes: ```[github,romainmarcoux/malicious-hash/full-hash-sha256-aa.txt]```

6. For the format, select "newline" since each hash can be found on a separate line
> [!TIP]
> To view the additional formats supported by LimaCharlie, visit [Lookups](https://docs.limacharlie.io/docs/lookups)

7. Click "Save" to save the lookup. 

8. In order to sync the lookup without waiting for the 24 hour period, click on "Manual Sync" at the top-right corner of the UI

9. If everything was done correctly, a message will pop up in the bottom-right corner of the UI that says the sync was successful. If there is an error, review your configuration and attempt again. If you still encounter issues, please reach out to the workshop coordinator for assistance

10. Repeat steps 3 through 8 for the remaining lookups

:tada: Lab 3 is finished and you now have working lookups that will automatically syncronize every 24 hours! Head over to lab 4 to configure outputs. 

### Lab 4: Configuring Outputs

LimaCharlie output can be used for many different tasks. Outputs can send events to a third-party location for long-term (> 1 year) retention, trigger webhooks, send notifications, etc. 

1. You know the drill at this point - if you are not already logged in to your LimaCharlie account, log in

2. On the sidebar, click "Outputs"

3. Click "Add Output"

4. Since we only want to send to our output on specific detections, select the "Tailored" output stream
> [!TIP]
> LimaCharlie's data pipeline allows you to output as much or as little data as desired. This can take the form of raw events, detections, audit logs, etc. For more information, please see [Outputs](https://docs.limacharlie.io/docs/outputs) in the LimaCharlie documentation

5. Choose the "Slack" output destination
> [!TIP]
> The output destination selection screen shows all of the output destinations supported by LimaCharlie. In addition to the vendor-specific destinations such as Tines, generic destinations are also supported via webhook and SMTP. This allows LimaCharlie to easily integrate with ticketing systems such as ServiceNow, ZenDesk, etc. For more information, please see [Output Destinations](https://docs.limacharlie.io/docs/output-destinations)

6. Give your destination a memorable name. You will need to reference this name during D&R rule creation

7. Set your API token to the token found in the workshop secrets file
> [!NOTE]
> If doing this outside of a physical workshop, you will need to create a free Slack workspace and follow the instructions located here to create the API token: [Getting a Slack API Token](https://api.slack.com/tutorials/tracks/getting-a-token)

8. Set your channel to ```#austin-workshop``` 

9. Click "Advanced Options" to display additional options for configuring the output

10. In the "Message Text" box, put your name and then click "Save Output"

:tada: Your LimaCharlie environment is now set up! Next, it's time to create some detection and response rules using all the work you've done thus far.

## Writing [Detection and Response Rules](https://docs.limacharlie.io/docs/detection-and-response) from Scratch

LimaCharlie allows users to create rules to detect activiy and then response actions to do something with that detection. These responses can range from a generating a simple alert to complex, multi-step processes intended to automate IR tasks such as isolating hosts, dumping memory, running playbooks. Additionally, these reponse actions can trigger actions in third-party applications such as Tines to enhance response activities. 

All rules are written in [YAML](https://yaml.org) and are made up of two descriptors, detection and response. 
> [!TIP] 
> Within the LimaCharlie UI, these are broken into separate text boxes, so when copying and pasting from the workshop files and LimaCharlie documentation, you will copy the section beginning with ```# Detection``` into the detection box and the section beginning with ```# Response``` into the response box.

### Lab 5: Writing a Detection and Response Rule for Known Malicious IPs

Rules can either be created completely from scratch or a basic rule can automatically be generated from an event. Events can be found directly from a sensor or from the LimaCharlie Query Console. For this lab, we'll utilize a LCQL query to find an event representative of what we want to detect on, and then use that rule generate our starter rule that we'll then expand on.

Currently, the Query Console is only available within the old UI, so if you have changed to the new UI, please click the "Go back to default theme" button at the top of the UI. 

1. Click "Query Console" from on the sidebar

2. Enter the following query and then slick save: 
```
-2h | * | NETWORK_CONNECTIONS | event/NETWORK_ACTIVITY/?/SOURCE/IP_ADDRESS is public address
```
> [!TIP]
> Save queries to save time and avoid having to remember the query every time you want to look for events or detections

3. Click "Save New Query" button to go to the save options

4. Give the query a descriptive name, and then click "Save" 

5. Click on the "Saved" tab to view the query you just created. Since this query is already loaded, just click on the "Query" tab to go back to your query.

6. Click Submit to query the data you loaded during lab 1. If you do not see any results, ensure you correctly entered the query. If you need assistance, please feel free to ask.

7. These events show processes making connections to internal and external IPs. To create the base detection rule, click "Start a D&R rule" at the bottom-right of the screen
> [!TIP] 
> You can download the results of the query in json or csv format

1. A basic detection rule is created and the event you used as the source is shown at the bottom of the screen

2. Copy and paste the following YAML into the "Detect" box, replacing the ```IP_LOOKUP_NAME``` field below with the IP lookup you created in [Lab 3: Ingesting Lookups](#lab-3-ingesting-lookups)
``` 
event: NETWORK_CONNECTIONS
op: lookup
path: event/NETWORK_ACTIVITY/?/SOURCE/IP_ADDRESS
resource: hive://lookup/IP_LOOKUP_NAME 
```

> [!TIP]
> **Explanation of what is going on here**  
> The event type we want to detect on: ```event: NETWORK_CONNECTIONS```  
> The operator that determines how the rules are interpreted, here it's a lookup: ```op: lookup```   
> The path within the event we want to look at for detection: ```path: event/NETWORK_ACTIVITY/?/SOURCE/IP_ADDRESS```  
> Finally, the lookup we want to compare the "path" against: ```resource: hive://lookup/IP_LOOKUP_NAME```

10. Copy and paste the following YAML into the "Respond" box, replacing the ```SLACK_OUTPUT_NAME``` with the name of the slack output you created in [Lab 4: Configuring Outputs](#lab-4-configuring-outputs):
```
- action: report
  name: WORKSHOP - Attempted connection to known malicious IP - {{ .routing.hostname }}
- action: output
  name: SLACK_OUTPUT_NAME
```
> [!IMPORTANT]
> Make sure your indentations are correct. YAML is an indentation-based language, so incorrect indentation will cause errors

> [!TIP]
> **Explanation of what is going on here**  
> The action we want to occur, in this case "report," which generates a detection: ```- action: report```
> The name we want our detection to display. This also includes a variable from the event's routing information to dynamically include the hostname in the detection name: ```   name: WORKSHOP - Attempted connection to known malicious IP - {{ .routing.hostname }}```
> The second action tells LimaCharlie to send the detection to an output: ```- action: output```
> Finally, the second "name" tells LimaCharlie which output to use: ```  name: SLACK_OUTPUT_NAME```

11. After you have entered the YAML in both text boxes, click "Create" to create your rule.

12. Open the external adapter you created in [Lab 1: Configuring LimaCharlie and Ingesting Logs](#lab-1-configuring-limacharlie-and-ingesting-logs) and change the ```file_path``` to point to the file ```sample_logs-1.jsonl```. The adapter will automatically restart and read in the new file

13. If you are attending an in-person LimaCharlie workshop, join the #austin-workshop channel in the testing Slack workspace at: [LC Testing Slack Workspace](https://join.slack.com/t/lc-testing/shared_invite/zt-372kgle8m-RITMnHuhoso1Gz8e~ExzFA)
   - If you are working on this workshop on your own, you will need to utilize your own Slack workspace

14. You should see message in the Slack channel with your name and the event information. If you do not see this after a few minutes, please verify your YAML was correctly created and the Slack output is correctly configured. If you need assistance, please don't hesitate to ask.

:tada: You've completed the first D&R rule lab! In the next lab, you'll take what you've learned and create your own rule.

### Lab 6: Writing a Detection and Response Rule for Known Malicious Hashes

Using the information you learned in this workshop, create a D&R rule that alerts on a known-bad hash using the following information:
* First sample log file: ```small_sample-2.jsonl```
> [!TIP]
> Reconfigure the external adapter to point to this file
* LCQL Query: ```-2h | * | NEW_PROCESS | event/HASH != ''```
   * Pick any of the NEW_PROCESS events to create a D&R rule from
> [!TIP]
> For help with LCQL, check out the [LCQL Cheat Sheet](https://github.com/lc-cbot/workshop-austin/blob/main/lcql_cheat_sheet.md)
* Match the hash contained at the following path against the hash lookup created in [Lab 3: Ingesting Lookups](#lab-3-ingesting-lookups)
   * ```event/HASH```
* Create a report action with a name of your choosing
   * Extra credit: Include the ```file_path``` field in the name
* Create another action to output the detection to the Slack output you previously created
* Use the ```sample_logs-2.jsonl``` file to test your newly created rule





