# 0G Validator monitoring guide with tenderduty

<img width="840" alt="tenderduty-01" src="https://github.com/qyeah98/0g/assets/99518712/13e2bda2-7964-483e-9530-6e1beb366d90">


## Set up monitoring system: Tenderduty
> [!TIP]
> **_1. Monitors a validator’s performance._**  
> The main purpose of tenderduty is to monitor the status of the consensus and alert if the validator is losing blocks.
> This can be based on consecutive failures or on a percentage of failures within the cutoff window.
> - Alert if jailed, tombstoned or inactive.
> - Alert destinations can be customised for each chain.
> - Monitors the health of nodes.
> - Provides a prometheus exporter for integration with other display systems.
> - Sends an alert to any of many destinations: Pagerduty, Discord, Telegram or Slack.
> - Dashboard to visualise the status.
> - Displays a table with the status of the validator and the node.
> - The last 512 blocks are displayed in the status grid.
> - Optionally displays a real-time log message stream with details of ongoing status checks.
> 
> **_2. Tenderduty_**  
> Tenderduty is a comprehensive monitoring tool for Tendermint networks.  
> Its main function is to alert a validator if it is missing blocks, and it has many other features.  
> V2 adds a web dashboard, a prometheus exporter, pagerduty and discord notifications, multi-chain support, more granular alerts and more alert types.  
> [Github Tenderduty](https://github.com/blockpane/tenderduty)

> [!CAUTION]
> This guide is supported for the 0G validator by following the [Moderator Daniel Moon guide](https://github.com/trusted-point/0g-tools).  
> Support OS is `Ubuntu22.04 LTS`.



## Update packages and Install dependencies
Run this command:
``` bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl build-essential git wget jq make gcc tmux pkg-config libssl-dev libleveldb-dev tar -y
```

## Install go
Run this command:
``` bash
ver="1.18.2"

cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"

sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"

rm "go$ver.linux-amd64.tar.gz"

echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
source ~/.bash_profile
go version
```

## Add user and install tenderduty
Run this command:
``` bash
sudo useradd -rs /bin/false tenderduty

cd $HOME
git clone https://github.com/blockpane/tenderduty
cd tenderduty

go install
cp example-config.yml config.yml
```

## Edit config.yml file 
Run this command:
``` bash
nano $HOME/tenderduty/config.yml
```

For simple monitoring, just change these in the config:
- Set project name : `0G`
- `chain_id: zgtendermint_16600-1`
- `valoper_address: <YOUR_VALOPER_ADDRESS>`   
  You can check your node valoper address by using `0gchaind keys show <YOUR_WALLET_NAME> --bech val -a`
- `url: https://rpc-testnet.0g.ai:443`
- `Discord enabled: yes`
- `Discord webhook: <YOUR_DOSCORD_WEBHOOK>`  
   How to get Discord Webhook [Discord: Intro to Webhooks](https://support.discord.com/hc/en-us/articles/228383668-Intro-to-Webhooks)

Example : config.yaml
``` yaml
---
# controls whether the dashboard is enabled.
enable_dashboard: yes
# What TCP port the dashboard will listen on. Only the port is controllable for now.
listen_port: 8888
# hide_logs is useful if the dashboard will be posted publicly. It disables the log feed,
# and obscures most node-related details. Be aware this isn't fully vetted for preventing
# info leaks about node names, etc.
hide_logs: no
# How long to wait before alerting that a node is down.
node_down_alert_minutes: 3
# Node Down alert Pagerduty Severity
node_down_alert_severity: critical

# Should the prometheus exporter be enabled?
prometheus_enabled: no
# What port should it listen on? For now only port is configurable.
prometheus_listen_port: 28866

# Global setting for pagerduty
pagerduty:
  # Should we use PD? Be aware that if this is set to no it overrides individual chain alerting settings.
  enabled: no
  # This is an API key, not oauth token, more details to follow, but check the v1 docs for more info
  api_key: aaaaaaaaaaaabbbbbbbbbbbbbcccccccccccc
  # Not currently used, but will be soon. This allows setting escalation priorities etc.
  default_severity: alert

# Discord settings
discord:
  # Alert to discord?
  enabled: yes # <============= Change to yes
  # The webhook is set by right-clicking on a channel, editing the settings, and configuring a webhook in the intergrations section.
  webhook: <YOUR_DOSCORD_WEBHOOK> # <========================== Edit

# Telegram settings
telegram:
  # Alert via telegram? Note: also supersedes chain-specific settings
  enabled: no
  # API key ... talk to @BotFather
  api_key: "5555555555:AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"
  # The group ID for the chat where messages will be sent. Google how to find this, will include better info later.
  channel: "-666666666"

# Slack settings
slack:
  # Send alerts to Slack?
  enabled: no
  # The webhook can be added in the Slack app directory.
  webhook: https://hooks.slack.com/services/xxxxxxxxxxxxxxxxxxxxxxxxxxx

# Healthcheck settings (dead man's switch)
healthcheck:
  # Send pings to determine if the monitor is running?
  enabled: no
  # URL to send pings to.
  ping_url: https://hc-ping.com/aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee
  # Rate in which pings are sent in seconds.
  ping_rate: 60

# The various chains to be monitored. Create a new entry for each chain. The name itself can be arbitrary, but a
# user-friendly name is recommended.
chains:
  # The user-friendly name that will be used for labels. Highly suggest wrapping in quotes.
  "0G":
    # chain_id is validated for a match when connecting to an RPC endpoint, also used as a label in several places.
    chain_id: zgtendermint_16600-1
    # Hooray, in v2 we derive the valcons from abci queries so you don't have to jump through hoops to figure out how
    # to convert ed25519 keys to the appropriate bech32 address.
    # Use valcons address if using ICS
    valoper_address:  <YOUR_VALOPER_ADDRESS> # <========== Set <YOUR_VALOPER_ADDRESS>
    # Should the monitor revert to using public API endpoints if all supplied RCP nodes fail?
    # This isn't always reliable, not all public nodes have websocket proxying setup correctly.
    public_fallback: no

    # Controls various alert settings for each chain.
    alerts:
      # If the chain stops seeing new blocks, should an alert be sent?
      stalled_enabled: yes
      # How long a halted chain takes in minutes to generate an alarm
      stalled_minutes: 1

      # Most basic alarm, you just missed x blocks ... would you like to know?
      consecutive_enabled: yes
      # How many missed blocks should trigger a notification?
      consecutive_missed: 1
      # Consecutive Missed alert Pagerduty Severity
      consecutive_priority: critical

      # For each chain there is a specific window of blocks and a percentage of missed blocks that will result in
      # a downtime jail infraction. Should an alert be sent if a certain percentage of this window is exceeded?
      percentage_enabled: no
      # What percentage should trigger the alert
      percentage_missed: 10
      # Percentage Missed alert Pagerduty Severity
      percentage_priority: warning

      # Should an alert be sent if the validator is not in the active set ie, jailed,
      # tombstoned, unbonding?
      alert_if_inactive: yes
      # Should an alert be sent if no RPC servers are responding? (Note this alarm is instantaneous with no delay)
      alert_if_no_servers: yes

      # for this *specific* chain it's possible to override alert settings. If the api_key or webhook addresses are empty,
      # the global settings will be used. Note, enabled must be set both globally and for each chain.

      # Chain specific setting for pagerduty
      pagerduty:
        enabled: yes
        api_key: "" # uses default if blank

      # Discord settings
      discord:
        enabled: yes
        webhook: "" # uses default if blank

      # Telegram settings
      telegram:
        enabled: yes
        api_key: "" # uses default if blank
        channel: "" # uses default if blank

      # Slack settings
      slack:
        enabled: yes
        webhook: "" # uses default if blank

    # This section covers our RPC providers. No LCD (aka REST) endpoints are used, only TM's RPC endpoints
    # Multiple hosts are encouraged, and will be tried sequentially until a working endpoint is discovered.
    nodes:
      # URL for the endpoint. Must include protocol://hostname:port
      - url: tcp://localhost:26657
        # Should we send an alert if this host isn't responding?
        alert_if_down: yes
      # repeat hosts for monitoring redundancy
      - url: https://rpc-testnet.0g.ai:443
        alert_if_down: no
```

## Create service file
Run this command:
``` bash
sudo tee /etc/systemd/system/tenderdutyd.service << EOF
[Unit]
Description=Tenderduty
After=network.target

[Service]
Type=simple
Restart=always
RestartSec=5
TimeoutSec=180

User=$USER
WorkingDirectory=$HOME/tenderduty
ExecStart=$(which tenderduty)

# there may be a large number of network connections if a lot of chains
LimitNOFILE=infinity

# extra process isolation
NoNewPrivileges=true
ProtectSystem=strict
RestrictSUIDSGID=true
LockPersonality=true
PrivateUsers=true
PrivateDevices=true
PrivateTmp=true

[Install]
WantedBy=multi-user.target
EOF
```

## Start tenderduty
Run this command:
``` bash
sudo systemctl daemon-reload
sudo systemctl enable tenderdutyd
sudo systemctl start tenderdutyd
```

## Status tenderduty
Run this command:
``` bash
sudo systemctl status tenderdutyd
```
Check if Tenderduty status is `active (running)`:  
----- Result -----
``` bash
● tenderdutyd.service - Tenderduty
     Loaded: loaded (/etc/systemd/system/tenderdutyd.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2024-05-01 05:38:07 UTC; 17h ago
   Main PID: 2247730 (tenderduty)
      Tasks: 15 (limit: 38237)
     Memory: 24.5M
        CPU: 25.202s
     CGroup: /system.slice/tenderdutyd.service
             └─2247730 /usr/local/bin/tenderduty
```


## Check tenderduty logs
Run this command:
``` bash
sudo journalctl -fu tenderdutyd
```
----- Result -----
``` bash
-- Logs begin at Sat 2024-03-16 23:10:29 UTC. --
May 01 23:30:42 0g-validator-newton tenderduty[530387]: 2024/05/01 23:30:42 tenderduty |  🟢 0G     node https://rpc-testnet.0g.ai:443 is healthy
May 01 23:31:42 0g-validator-newton tenderduty[530387]: 2024/05/01 23:31:42 tenderduty |  🟢 0G     node tcp://localhost:26657 is healthy
May 01 23:31:42 0g-validator-newton tenderduty[530387]: 2024/05/01 23:31:42 tenderduty |  🟢 0G     node https://rpc-testnet.0g.ai:443 is healthy
May 01 23:32:09 0g-validator-newton tenderduty[530387]: 2024/05/01 23:32:09 tenderduty |  🧊 zgtendermint_16600-1      block 836770
May 01 23:32:42 0g-validator-newton tenderduty[530387]: 2024/05/01 23:32:42 tenderduty |  🟢 0G     node tcp://localhost:26657 is healthy
May 01 23:32:42 0g-validator-newton tenderduty[530387]: 2024/05/01 23:32:42 tenderduty |  🟢 0G     node https://rpc-testnet.0g.ai:443 is healthy
May 01 23:33:42 0g-validator-newton tenderduty[530387]: 2024/05/01 23:33:42 tenderduty |  🟢 0G     node tcp://localhost:26657 is healthy
May 01 23:33:42 0g-validator-newton tenderduty[530387]: 2024/05/01 23:33:42 tenderduty |  🟢 0G     node https://rpc-testnet.0g.ai:443 is healthy
May 01 23:34:42 0g-validator-newton tenderduty[530387]: 2024/05/01 23:34:42 tenderduty |  🟢 0G     node tcp://localhost:26657 is healthy
May 01 23:34:42 0g-validator-newton tenderduty[530387]: 2024/05/01 23:34:42 tenderduty |  🟢 0G     node https://rpc-testnet.0g.ai:443 is healthy
```

> [!TIP]
> You can open dashboard on web browser by using tenderduty port and your server IP.  
> `http://<YOUR_SERVER_IP>:<PORT>`
> Default port on tenderduty is `8888`

Congratulations!  
You have set up a monitoring and alert system!!!
