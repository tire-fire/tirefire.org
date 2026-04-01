+++
title = "Dynamic 1:1 NAT for pfSense"
date = "2026-03-31"
author = "tirefire"
cover = "img/tirefire.gif"
description = "Automatically configures 1:1 NAT based on DHCP-assigned WAN subnet"
+++

Automatically configures 1:1 NAT based on DHCP-assigned WAN subnet. Maps the LAN subnet to the external WAN /24 network.

## Installation

```bash
scp dynamic_1to1_nat.php root@<pfsense>:/usr/local/bin/
scp dynamic_1to1_nat.sh root@<pfsense>:/usr/local/etc/rc.d/
ssh root@<pfsense> "chmod +x /usr/local/bin/dynamic_1to1_nat.php /usr/local/etc/rc.d/dynamic_1to1_nat.sh"
```

## Configuration

Edit constants in `dynamic_1to1_nat.php`:

```php
define('WAN_INTERFACE', 'wan');           // WAN interface name
define('MAX_WAIT_SECS', 60);              // DHCP wait timeout
```

## Logs

```bash
tail -f /var/log/dynamic_1to1_nat.log
```

## Uninstallation

```bash
rm /usr/local/bin/dynamic_1to1_nat.php
rm /usr/local/etc/rc.d/dynamic_1to1_nat.sh
rm /var/log/dynamic_1to1_nat.log
```

Remove the 1:1 NAT rule from **Firewall > NAT > 1:1**.

## Files

### dynamic_1to1_nat.php

```php
#!/usr/local/bin/php-cgi -q
<?php
/*
 * dynamic_1to1_nat.php
 *
 * Automatically configures 1:1 NAT based on DHCP-assigned WAN subnet.
 * Maps LAN subnet to external WAN /24 network.
 *
 * Deploy to: /usr/local/bin/dynamic_1to1_nat.php
 * Run via rc.d script at boot
 */

require_once("config.inc");
require_once("util.inc");
require_once("filter.inc");
require_once("interfaces.inc");

define('WAN_INTERFACE', 'wan');
define('NAT_DESCRIPTION', 'Auto 1:1 NAT');
define('LOCK_FILE', '/tmp/dynamic_1to1_nat.lock');
define('LOG_FILE', '/var/log/dynamic_1to1_nat.log');
define('MAX_WAIT_SECS', 60);

function log_msg($msg) {
    $timestamp = date('Y-m-d H:i:s');
    $line = "[{$timestamp}] {$msg}\n";
    file_put_contents(LOG_FILE, $line, FILE_APPEND);
    echo $line;
}

function get_wan_ip() {
    $wan_if = get_real_interface(WAN_INTERFACE);
    if (empty($wan_if)) {
        return null;
    }
    return find_interface_ip($wan_if);
}

function get_wan_network($ip) {
    if (empty($ip)) {
        return null;
    }
    $octets = explode('.', $ip);
    if (count($octets) !== 4) {
        return null;
    }
    return "{$octets[0]}.{$octets[1]}.{$octets[2]}.0";
}

function find_existing_rule_index() {
    global $config;

    if (!isset($config['nat']['onetoone']) || !is_array($config['nat']['onetoone'])) {
        return -1;
    }

    foreach ($config['nat']['onetoone'] as $idx => $rule) {
        if (isset($rule['descr']) && $rule['descr'] === NAT_DESCRIPTION) {
            return $idx;
        }
    }

    return -1;
}

function create_nat_rule($external_network) {
    return array(
        'interface' => WAN_INTERFACE,
        'ipprotocol' => 'inet',
        'external' => $external_network,
        'source' => array(
            'network' => 'lan'
        ),
        'destination' => array(
            'any' => ''
        ),
        'descr' => NAT_DESCRIPTION,
        'natreflection' => 'disable'
    );
}

function update_nat_config($external_network) {
    global $config;

    if (!isset($config['nat'])) {
        $config['nat'] = array();
    }
    if (!isset($config['nat']['onetoone'])) {
        $config['nat']['onetoone'] = array();
    }

    $existing_idx = find_existing_rule_index();
    $new_rule = create_nat_rule($external_network);

    if ($existing_idx >= 0) {
        $old_external = $config['nat']['onetoone'][$existing_idx]['external'];
        if ($old_external === $external_network) {
            log_msg("Rule already exists with correct external network {$external_network}");
            return false;
        }
        log_msg("Updating existing rule: {$old_external} -> {$external_network}");
        $config['nat']['onetoone'][$existing_idx] = $new_rule;
    } else {
        log_msg("Creating new 1:1 NAT rule for {$external_network}");
        array_unshift($config['nat']['onetoone'], $new_rule);
    }

    return true;
}

function acquire_lock() {
    $fp = fopen(LOCK_FILE, 'c');
    if (!$fp) {
        return false;
    }
    if (!flock($fp, LOCK_EX | LOCK_NB)) {
        fclose($fp);
        return false;
    }
    return $fp;
}

function release_lock($fp) {
    if ($fp) {
        flock($fp, LOCK_UN);
        fclose($fp);
        @unlink(LOCK_FILE);
    }
}

function main() {
    global $config;

    log_msg("=== Starting dynamic 1:1 NAT configuration ===");

    $lock = acquire_lock();
    if (!$lock) {
        log_msg("Another instance is running, exiting");
        return 1;
    }

    log_msg("Waiting for WAN IP assignment (max " . MAX_WAIT_SECS . "s)...");
    $wan_ip = null;
    for ($i = 0; $i < MAX_WAIT_SECS; $i++) {
        $wan_ip = get_wan_ip();
        if (!empty($wan_ip)) {
            break;
        }
        sleep(1);
    }

    if (empty($wan_ip)) {
        log_msg("ERROR: Failed to get WAN IP after " . MAX_WAIT_SECS . " seconds");
        release_lock($lock);
        return 1;
    }

    log_msg("WAN IP: {$wan_ip}");

    $external_network = get_wan_network($wan_ip);
    if (empty($external_network)) {
        log_msg("ERROR: Could not determine external network from IP {$wan_ip}");
        release_lock($lock);
        return 1;
    }

    log_msg("External network: {$external_network}/24");

    $changed = update_nat_config($external_network);

    if (!isset($config['nat']['outbound']['mode']) || $config['nat']['outbound']['mode'] !== 'hybrid') {
        log_msg("Setting outbound NAT mode to hybrid");
        $config['nat']['outbound']['mode'] = 'hybrid';
        $changed = true;
    }

    if ($changed) {
        log_msg("Writing configuration...");
        write_config("Dynamic 1:1 NAT: LAN <-> " . $external_network . "/24");

        log_msg("Reloading filter rules...");
        filter_configure();

        log_msg("Configuration applied successfully");
    } else {
        log_msg("No changes needed");
    }

    release_lock($lock);
    log_msg("=== Completed ===");
    return 0;
}

exit(main());
?>
```

### dynamic_1to1_nat.sh

```bash
#!/bin/sh
#
# dynamic_1to1_nat.sh
#
# Wrapper script to trigger dynamic 1:1 NAT configuration
# Can be called from rc.d at boot or from rc.newwanip on DHCP renewal
#
# Deploy to: /usr/local/etc/rc.d/dynamic_1to1_nat.sh
# Make executable: chmod +x /usr/local/etc/rc.d/dynamic_1to1_nat.sh
#

SCRIPT="/usr/local/bin/dynamic_1to1_nat.php"
LOGFILE="/var/log/dynamic_1to1_nat.log"

log_event() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] [wrapper] $1" >> "$LOGFILE"
}

case "$1" in
    start|"")
        log_event "Triggered with arg: ${1:-none}"
        if [ -x "$SCRIPT" ]; then
            /usr/local/bin/php-cgi -q "$SCRIPT" 2>&1
            # Ensure WAN firewall rule allows inbound to NAT'd network
            easyrule pass wan any any 192.168.220.0/24 any >/dev/null 2>&1 || true
        else
            log_event "ERROR: Script not found or not executable: $SCRIPT"
            exit 1
        fi
        ;;
    stop)
        log_event "Stop called - nothing to do"
        ;;
    restart)
        log_event "Restart called - running script"
        if [ -x "$SCRIPT" ]; then
            /usr/local/bin/php-cgi -q "$SCRIPT" 2>&1
        fi
        ;;
    *)
        echo "Usage: $0 {start|stop|restart}"
        exit 1
        ;;
esac

exit 0
```
