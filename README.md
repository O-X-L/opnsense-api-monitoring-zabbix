# OPNsense - Monitoring over API with Zabbix

<p align="center">
    <a title="Support this Project (Donate, Support-Licenses)" href="https://shop.oxl.app/collections/open-source">
        <img src="https://files.oxl.at/img/badge-oss-support.svg" alt="Support Badge (Donate, Support-Licenses)"/>
    </a>
</p>

This Zabbix template utilizes the built-in [HTTP_AGENT](https://www.zabbix.com/documentation/current/en/manual/config/items/itemtypes/http) to pull information from the OPNsense API and optimizes API-calls by utilizing [dependant-items](https://www.zabbix.com/documentation/current/en/manual/config/items/itemtypes/dependent_items).

Some checks may only work with OPNsense version >= 25.7

----

## Security

* Only connections over HTTPS
* SSL/TLS verification enabled (`verify_peer` & `verify_host`)
* Using Service-User with minimal privileges

----

## Template Content

* Items
  * Gateway Stati
    * Gateway Stati: OPNsense - Gateways High Delay
    * Gateway Stati: OPNsense - Gateways Offline
  * VIP Stati
    * VIP - HA Status
  * IPSec Phase-1 Stati
    * IPSec Phase-1 Count
  * IPSec Phase-2 Discovery
    * IPSec Phase-2 Status
      * IPSec Phase-2 Count
  * Routes
  * Service Stati
  * System Times

* Triggers
  * Gateways are offline
  * Services are inactive
  * Gateways have high delays
  * HA-State not as expected
  * IPSec Tunnels Phase-1 are offline
  * IPSec Tunnels Phase-2 are offline

----

## Setup

### OPNsense Service-User

* Create a new user at `System - Access - Users`

  <img src="https://raw.githubusercontent.com/O-X-L/opnsense-api-monitoring-zabbix/refs/heads/latest/docs/system-access-users.png" alt="OPNsense User Menu" width="300" />

* Give the user a name, password and the following permissions:
  * `Lobby: Dashboard`
  * `Diagnostics: Routing tables`
  * `System: Gateways` & `System: Gateway Groups` (*sadly, it seems there is no read-only option for the gateways*)
  * All with prefix `Status:` (*Could be limited*)

  <img src="https://raw.githubusercontent.com/O-X-L/opnsense-api-monitoring-zabbix/refs/heads/latest/docs/new-user.png" alt="OPNsense User creation" width="250" />

* Create and download an API-key for the user

  <img src="https://raw.githubusercontent.com/O-X-L/opnsense-api-monitoring-zabbix/refs/heads/latest/docs/api-key.png" alt="OPNsense User API-Key creation" width="400" />

* Test the access:

  If you see a response like this - you may have to increase the permissions: `{"status":403,"message":"Forbidden"}`

  ```bash
  OPN_FIREWALL="IP-or-DNS"
  OPN_API_KEY="YOUR-KEY"
  OPN_API_SECRET="YOUR-SECRET"
  
  echo "### TESTING SYSTEM-TIMES ###"
  curl -u "${OPN_API_KEY}:${OPN_API_SECRET}" -XGET "https://${OPN_FIREWALL}/api/diagnostics/system/system_time"

  echo "### TESTING ROUTES ###"
  curl -u "${OPN_API_KEY}:${OPN_API_SECRET}" -XGET "https://${OPN_FIREWALL}/api/diagnostics/interface/get_routes"
  
  echo "### TESTING VIRTUAL-IPs ###"
  curl -u "${OPN_API_KEY}:${OPN_API_SECRET}" -XGET "https://${OPN_FIREWALL}/api/diagnostics/interface/get_vip_status"

  echo "### TESTING IPSec ###"
  curl -u "${OPN_API_KEY}:${OPN_API_SECRET}" -XGET "https://${OPN_FIREWALL}/api/ipsec/sessions/search_phase1"
  curl -u "${OPN_API_KEY}:${OPN_API_SECRET}" -XGET "https://${OPN_FIREWALL}/api/ipsec/sessions/search_phase2"
  
  echo "### TESTING SERVICES ###"
  curl -u "${OPN_API_KEY}:${OPN_API_SECRET}" -XGET "https://${OPN_FIREWALL}/api/core/service/search"

  echo "### TESTING GATEWAYS ###"
  curl -u "${OPN_API_KEY}:${OPN_API_SECRET}" -XGET "https://${OPN_FIREWALL}/api/routing/settings/search_gateway"
  ```

----

### Network Access

The Zabbix-Server or -Proxy you choose to monitor the firewall-hosts with, has to be able to connect to the firewall's Web-UI.

----

### Zabbix Server

* Import the YAML Template into your Zabbix Server
* Link the template to a Firewall-Host
* Configure the required host-macros:
  * `{$OPN_FIREWALL}` => the IP or DNS of the firewall as configured for the Web-UI (*can include a port*)
  * `{$OPN_API_KEY}` => the generated API-key
  * `{$OPN_API_SECRET}` => the generated API-secret

* Configure optional host-macros:
  * `{$OPN_HA_STATE_EXPECT}` => the HA-state to be expected: 'standalone', 'primary' or 'secondary' (*default = standalone*)
  * `{$OPN_IPSEC_P1_EXPECT}` => number of IPSec-phase-1 connections to expect
  * `{$OPN_IPSEC_P2_EXPECT}` => number of IPSec-phase-2 connections to expect
