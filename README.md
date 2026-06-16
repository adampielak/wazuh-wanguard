# wazuh-wanguard

![Wazuh](https://img.shields.io/badge/wazuh-4.x-005571)
![Type](https://img.shields.io/badge/type-active%20response-orange)
![License](https://img.shields.io/badge/license-MIT-green)

Wazuh active-response integration with [Andrisoft Wanguard](https://www.andrisoft.com/software/wanguard) — automatically trigger DDoS mitigation in Wanguard when Wazuh detects a volumetric attack or anomalous traffic event.

## How it works

```
Wazuh alert (rule match)
        │
        ▼
active-response script
        │
        ▼
Wanguard REST API → BGP blackhole / traffic scrubbing
```

When a configured Wazuh rule fires, the active-response script calls the Wanguard API to block or divert the offending IP through BGP or RTBH (Remote Triggered Black Hole).

## Install

```bash
# Copy active-response script
cp wanguard-ar.sh /var/ossec/active-response/bin/
chmod 750 /var/ossec/active-response/bin/wanguard-ar.sh
chown root:wazuh /var/ossec/active-response/bin/wanguard-ar.sh
```

Add to `/var/ossec/etc/ossec.conf`:

```xml
<ossec_config>
  <command>
    <name>wanguard-block</name>
    <executable>wanguard-ar.sh</executable>
    <timeout_allowed>yes</timeout_allowed>
  </command>

  <active-response>
    <command>wanguard-block</command>
    <location>local</location>
    <rules_id>100200</rules_id>
    <timeout>600</timeout>
  </active-response>
</ossec_config>
```

Configure Wanguard API endpoint and credentials in the script header:

```bash
WANGUARD_URL="https://wanguard.example.com/api"
WANGUARD_USER="api-user"
WANGUARD_PASS="api-password"
```

Restart manager:

```bash
systemctl restart wazuh-manager
```

## Requirements

- Wazuh 4.x manager
- Andrisoft Wanguard with REST API enabled
- `curl` on the manager host
