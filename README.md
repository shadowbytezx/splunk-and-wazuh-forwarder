## SIEM Integration: Wazuh → Splunk Forwarding

Centralized Wazuh's alert pipeline into Splunk using the Splunk Universal Forwarder, demonstrating multi-tool SIEM correlation rather than running each platform in isolation.

### Architecture

Wazuh Server (pogi, 10.0.3.3)
└── /var/ossec/logs/alerts/alerts.json
└── Splunk Universal Forwarder
└── tcpout → 10.0.3.4:9997
└── Splunk Enterprise Indexer (attacker, 10.0.3.4)


### Result
All Wazuh alerts — from both the Linux server and the Windows/Sysmon endpoint — are now searchable in Splunk using SPL, alongside any future data sources added to the same indexer.

```spl
index=main sourcetype=_json rule.level>=10
```
→ 112 high-severity events across both agents in a single query.

```spl
index=main sourcetype=_json | stats count by rule.description
```
→ Full breakdown of every alert type generated across the entire project, in one table.

### Troubleshooting Notes
- The Universal Forwarder requires the launching user to match ownership via `sudo` — running it as a separate dedicated service account fought against Splunk's own auto-chown behavior and had to be abandoned in favor of running it as the primary user.
- A stale `/opt/splunkforwarder/etc/passwd` entry from earlier failed attempts silently blocked a `user-seed.conf`-based credential reset; deleting it was required before the seed would apply.
- `inputs.conf` is not created automatically — the CLI's `add monitor` command can fail silently under certain auth states, so verifying/writing the config file directly was a faster, more reliable path than continuing to retry the CLI.

**Lesson:** when a CLI tool's interactive commands keep failing in ways that don't make sense, check whether its state lives in a plain config file you can read and edit directly — bypassing the tool's own command layer can isolate whether the problem is the tool's logic or its underlying state.
