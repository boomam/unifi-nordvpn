# NordVPN "NordLynx" (Wireguard) client connection from UDM
Process for setting up NordVPN NordLynx (Wireguard protocol) on UDM/UDM-SE, to allow local network routing through NordVPN.

## Pre-Requisities
* Ubiquiti UniFi Dream Machine/Dream Machine SE.
* A seperate VLAN for your VPN traffic (optional, but recommended).
* NordVPN Subscription.
* A linux VM, or linux machine.

## Forewarning/Unknowns
One unknown right now is how often the keys need to be exported from the config via a linux system, or if the connection stays active, then resets the keys on the next connection.  
Until this is known, it is possible that the connection will need configuring, with data from a linux system every so often.  
&nbsp;  

## Step-by-step
### Step 1 - Get Wireguard Config from linux machine
From your linux machine, install the following packages    
```
apt update
apt install -y wireguard wireguard-tools curl jq net-tools
```
Next, install the linux version of the NordVPN client  
_Latest steps from NordVPN themselves, are located [here](https://support.nordvpn.com/hc/en-us/articles/20196094470929-Installing-NordVPN-on-Linux-distributions)_  
```
sh <(curl -sSf https://downloads.nordcdn.com/apps/linux/install.sh)
```
Get Access Token to login to NordVPN (without a GUI).  
Latest steps from NordVPN themselves, are located[here](https://support.nordvpn.com/hc/en-us/articles/20226600447633-How-to-log-in-to-NordVPN-on-Linux-devices-without-a-GUI)  
Once you have the token, copy it, then continue.  
&nbsp;  
Enter the token into the command below, replacing <token>  
```
nordvpn login --token <token>
```
Change Protocol to NordLynx (Wireguard)  
```
nordvpn set technology nordlynx
```
Configure server/region to connect too  
```
nordvpn c us123
```
Syntax here being `nordvpn c <country-code><server number>`  
For example, `nordvpn vpn c us123`.  
_you can also use specify the country, such as `us`.  
You can get the server number from [here](https://nordvpn.com/servers/tools/)  
&nbsp;  

Next, get the config from the local wireguard config.  
```
wg showconf nordlynx
wg show
ip addr show
```
You should get outputs similar to the below - 

`wg showconf nordlynx`  
```
[Interface]
ListenPort = 53656
FwMark = 1xe1h1
PrivateKey = lghfklhjl356mmjhfgjgfjfg

[Peer]
PublicKey = lghfklhjl356mmjhfgjgfjfg
AllowedIPs = 0.0.0.0/0
Endpoint = 1.2.3.4:1234
PersistentKeepalive = 25
```
&nbsp;  
`wg show`  
```
interface: nordlynx
  public key: lghfklhjl356mmjhfgjgfjfg
  private key: (hidden)
  listening port: 1234
  fwmark: 1xe1h1

peer: lghfklhjl356mmjhfgjgfjfg
  endpoint: 1.2.3.4:1234
  allowed ips: 0.0.0.0/0
  latest handshake: 31 seconds ago
  transfer: 54.56 KiB received, 50.97 KiB sent
  persistent keepalive: every 25 seconds
```
&nbsp;  
`ip addr show`  
```
5: nordlynx: <POINTOPOINT,UP,LOWER_UP> mtu 1420 qdisc noqueue state UNKNOWN group default qlen 1000
    link/none 
    inet 10.1.0.1/32 scope global nordlynx
       valid_lft forever preferred_lft forever
```
Save this information, as we will be using it in Unifi.


### Step 2 - Configure UniFi to make the connection
1. Login to the Unifi Network Console, then browse to `Settings`->`VPN`->`VPN Client`->`Create New`. 
2. Enter a descriptive name, for example `Nord VPN US 123`.
3. Change the `Setup` radio button to `Manual`.
4. Under here, we will be entering the following settings -  
  a. `Private Key` -> `Private Key`  
  b. `Endpoint IP` -> `Server Address IP/Hostname`  
  c. `Endpoint Port` -> `Server Address Port`  
  d. `Public Key` -> `Public Server Key`  
  e. `Interface Address` (ip addr show) -> `Tunnel IP`  
5. Click `Apply Changes`  

From here you should see the top right of the config screen change from 'not established', to 'connecting', to 'connected'.  
&nbsp;   

### Step 3 - Configure routing
1. Next, in the UniFi console, browse to `Settings`->`Routing`->`Policy-Based Routes`->`Create Entry`.  
2. Enter a descriptive name, for example 'VPN net to NordVPN'.  
3. Change the `What to Route?` option to `All Traffic`.  
4. Click `Select Device or Network`, then tick the boxes for the networks (or devices) you want to use NordVPN, then click `Save` once done.  
5. On the drop down next to `Interface`, select your newly created VPN client profile.  
6. Click `Add Entry` at the bottom.  
7. You're all done!  

### Step 4 - Possible improvements - 
1. Replacing 'Endpoint IP' with the hostname (not IP) of the VPN server, taken from NordVPNs docs, [here](https://nordvpn.com/servers/tools/).  
2. Ascertain lifetime of public/private keys.  

