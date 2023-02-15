# Command line process auditing

## What is command line process auditing?

Command Line Process Auditing adds additional information to event ID 4688 events (a new process has been created) showing the full audit information for command line processes.

![image](https://user-images.githubusercontent.com/88635951/218908151-9fbc0fb9-26de-4f53-bb31-2e6749ffb687.png)

If we take this example, without command line process auditing enabled, the 'Process Command Line' field, highlighted in red, is not available. We will still see the process name of 'C:\Windows\System32\wscript.exe' but not the detail of any command line arguments run with it.

## Why is command line process auditing important?

Command line process auditing is an extremely valuable piece of information that can be used to determine whether processes run are legitimate or malicious.

This is especially relevant with adversaries living off the land using legitimate tools maliciously. For instance, [Impacket](https://github.com/fortra/impacket) can use WmiPrvSE.exe to spawn cmd.exe processes, but so can legitimate applications. However, Impacket also has a very distinct command line syntax that is only exposed once auditing is enabled.

## How do I turn it on?

You can enable command line process auditing via [group policy](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/component-updates/command-line-process-auditing). Like any changes, it should be tested before being deployed widely.

## I don't have a SIEM, or it would cost too much money to ingest these events, is this still valuable?

Yes! Although it would be best to send logging data to a SIEM, it isn't a requirement. Even if you store command line process events locally on your devices it is a valuable forensic artifact. In the event of an incident, the logs can be retrieved from any devices of interest and analyzed by your incident response and forensic teams. It is recommended you configure your devices to store enough logging data locally to give you a useful amount of historical data needed during an incident.

## I have an EDR, is this still valuable?

Yes! Although process auditing is a key component of EDR products, it is still valuable to enable command line process auditing on your devices. This is important for a number of reasons:

- Threat actors are known to tamper with or disable EDR and other security tools
- Some devices may not be fully enrolled in your EDR product
- EDR products have different capabilities, which can change over time. Having auditing enabled on your devices separately gives you consistentcy to your available forensic data
- You may change your EDR product over time. Having auditing enabled on your devices separately gives you consistentcy to your available forensic data
- Some devices may be in airgapped networks or locations where the internet is not always available, so they aren't consistently sending data to your EDR

## Are there any downsides?

Enabling command line process auditing can expose sensitive information, such as usernames, passwords or other credentials. If this information is included in clear text in command line arguments, it will be logged to the event log and visible to anyone who has access to that data. It is recommended, of course, that credentials aren't sent in clear text in this way.

## Resources

[Microsoft Learn - Command line process auditing](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/component-updates/command-line-process-auditing)

[Defense Against the Lateral Arts: Detecting and Preventing Impacket’s Wmiexec](https://www.crowdstrike.com/blog/how-to-detect-and-prevent-impackets-wmiexec/)

[Defense Against the Lateral Arts: Detecting and Preventing Impacket’s Wmiexec](https://www.13cubed.com/downloads/impacket_exec_commands_cheat_sheet.pdf)

[Wanted: Process Command Lines](https://www.trustedsec.com/blog/wanted-process-command-lines/)

[Windows Event Logging and Forwarding](https://www.cyber.gov.au/acsc/view-all-content/publications/windows-event-logging-and-forwarding)

# Sysmon

## What is Sysmon?

Sysmon is one of the Sysinternals tools, developed and maintained by Microsoft. Sysmon adds additional, high fidelity logging data to the standard Windows events, and can include events such as file events, network connections and process creation events. These events are written to the 'Applications and Services Logs/Microsoft/Windows/Sysmon/Operational' log from Windows Vista onwards.

![image](https://user-images.githubusercontent.com/88635951/218924362-f34fa2f2-5891-42da-b466-b982ed02943b.png)

This picture shows an example of a process creation event in Sysmon.

## How do I install Sysmon?

Sysmon is very simple to install, you can install it manually on devices, or deploy it via script with Group Policy or SCCM, or similar tools.

When installing Sysmon, you provide a .xml file which tells Sysmon which events to capture.

![image](https://user-images.githubusercontent.com/88635951/218924206-5c8e9559-a11b-4f77-85f0-56b710600cf1.png)

This example shows Sysmon installed manually using an .xml config file.

## XML? That sounds hard.

It doesn't need to be, there are plenty of example .xml files available that have been created by community members and shared publicly.

- SwiftOnSecurity(https://github.com/SwiftOnSecurity/sysmon-config)
- Olaf Hartong(https://github.com/olafhartong/sysmon-modular)
- Florian Roth(https://github.com/Neo23x0/sysmon-config)

It is recommended to test these configurations in your environment before deploying widely to ensure they are compatible and understand the overhead on your devices. You may need to exclude tooling specific to your environment to reduce noise.

## Can I send this data to Sentinel or my SIEM?

Yes! That would be amazing - and in the event of an incident, the forensic team will love you for it. Depending on which SIEM product you use, how you get that data ingested will differ. You can use native Windows Event Forwarding, or your SIEM provider likely has native connectors or agents to collect that data. Keep in mind the cost of ingesting additional data of course.

## But I don't have a SIEM, does that matter?

No! Having Sysmon deployed and collecting data locally on devices is still valuable. It would be amazing to send this data to your SIEM, but if you can't, having it available locally on your devices is still an extremely valuable. Like all logs, it is recommended to configure Windows with enough disk space to retain some historical data. If data is only kept for a few hours because of Event Log size, the forensic value of that data reduces.

## I have an EDR, should I deploy Sysmon too?

Yes! As mentioned above, even though there is functionality cross over between EDR products and what Sysmon can log, maintaining Sysmon is valuable because:

- Threat actors are known to tamper with or disable EDR and other security tools
- Some devices may not be fully enrolled in your EDR product
- EDR products have different capabilities, which can change over time. Having Sysmon enabled on your devices separately gives you consistentcy to your available forensic data
- You may change your EDR product over time. Having Sysmon enabled on your devices separately gives you consistentcy to your available forensic data
- Some devices may be in airgapped networks or locations where the internet is not always available, so they aren't consistently sending data to your EDR
- Using an EDR you are at the mercy of what the vendor whats to log and alert on. While most EDR's are constantly updated with new alerts and detections, maintaining Sysmon is still valuable

Some customers choose to augment their EDR with Sysmon to reduce the cross over. Sysmon is not a replacement for your EDR or standard Windows event logs, it is a designed to compliment those.

## Can Sysmon prevent bad, or is it just for monitoring?

Sysmon v14 and newer can prevent executable files being written to specific folders, based on config. If you are new to Sysmon I would recommend first monitoring with it and getting experience with it before looking to enforce blocking actions.

Florian Roth has provided an [example of a blocking config](https://github.com/Neo23x0/sysmon-config/blob/master/sysmonconfig-export-block.xml) that prevents executable files being written to popular malware staging directories

## Resources

[Microsoft Learn - Sysmon](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)

[Florian Roth's Sysmon Configs](https://github.com/Neo23x0/sysmon-config)

[SwiftOnSecurity's Sysmon Config](https://github.com/SwiftOnSecurity/sysmon-config)

[Olaf Hartong's Sysmon Configs](https://github.com/olafhartong/sysmon-modular)

[Deploy Sysmon and collect additional data with Sentinel and the AMA agent](https://jeffreyappel.nl/deploy-sysmon-and-collect-data-with-sentinel-and-the-ama-agent/)



