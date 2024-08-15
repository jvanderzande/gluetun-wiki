# FastestVPN

## TLDR

```sh
# OpenVPN
docker run -it --rm --cap-add=NET_ADMIN -e VPN_SERVICE_PROVIDER=fastestvpn \
-e OPENVPN_USER=abc -e OPENVPN_PASSWORD=abc \
-e SERVER_COUNTRIES=Netherlands qmcgaw/gluetun
```

```sh
# Wireguard
docker run -it --rm --cap-add=NET_ADMIN -e VPN_SERVICE_PROVIDER=fastestvpn \
-e VPN_TYPE=wireguard \
-e WIREGUARD_PRIVATE_KEY=wOEI9rqqbDwnN8/Bpp22sVz48T71vJ4fYmFWujulwUU= \
-e WIREGUARD_ADDRESSES="10.64.222.21/32" \
-e SERVER_COUNTRIES=Netherlands qmcgaw/gluetun
```

```yml
version: "3"
services:
  gluetun:
    image: qmcgaw/gluetun
    cap_add:
      - NET_ADMIN
    environment:
      - VPN_SERVICE_PROVIDER=fastestvpn
      - VPN_TYPE=wireguard
      - WIREGUARD_PRIVATE_KEY=wOEI9rqqbDwnN8/Bpp22sVz48T71vJ4fYmFWujulwUU=
      - WIREGUARD_ADDRESSES=10.64.222.21/32
      - SERVER_COUNTRIES=Netherlands
```

## Required environment variables

- `VPN_SERVICE_PROVIDER=fastestvpn`

### OpenVPN only

- `OPENVPN_USER`
- `OPENVPN_PASSWORD`

### Wireguard only

- `WIREGUARD_PRIVATE_KEY` is your 32 bytes key in base64 format. It corresponds to the `PrivateKey` field value in the Wireguard configuration file.
- `WIREGUARD_ADDRESSES` is the IP prefix to assign to the Wireguard interface, corresponding to the `Address` field value in the Wireguard configuration file.

## Optional environment variables

- `SERVER_COUNTRIES`: Comma separated list of countries
- `SERVER_CITIES`: Comma separated list of cities
- `SERVER_HOSTNAMES`: Comma separated list of server hostnames

## Servers

To see a list of servers available, [list the VPN servers with Gluetun](../servers.md#list-of-vpn-servers).

## How to obtain your WIREGUARD_PRIVATE_KEY

You could request FastestVPN support for your private_key.  
An other option is to use the FastestVPN windows client.
The Windows FastestVPN client generates an **FastestVPNWireGuard.conf** file in directory **%ProgramW6432%\FastestVPN\Resources\data** at the startup of an VPN connection.
The only challenge is that the file exists for a few seconds at startup of an VPN connection and is then  deleted. You need to be fast to copy it or use the below Windows batchfile to help you out.
Just copy&paste the below lines into **FastestVPN_Get_wireguard_config.bat** file, run it and follow the instructions.

```cmd
@echo off
setlocal EnableDelayedExpansion
REM This script will try to capture the generated wireguard config file and copy it before it gets deleted again by the window FastestVPN program.
REM It will also show you the docker environment variables required for gluetun.
echo =========================================================================================
echo Manual steps before this script can copy your Wireguard settings for FastestVPN:
echo =========================================================================================
echo  - First you need to start the FastestVPN Windows client
echo  - Ensure the protocol is set to wireguard in: Settings/VPN Protocol/Disable Auto and Select Wireguard
echo  - Press "any key" in this window so the batchfile will start monitoring for: %ProgramW6432%\FastestVPN\Resources\data\FastestVPNWireGuard.conf
echo  - Start a vpn connection in the FastestVPN program
echo  - The batchscript should detect the creation of %ProgramW6432%\FastestVPN\Resources\data\FastestVPNWireGuard.conf and copy it to this script directory
pause
if exist "%ProgramW6432%\FastestVPN\Resources\data" (
  echo Found FastestVPNWireGuard directory so start monitoring for FastestVPNWireGuard.conf .
) else (
  echo %ProgramW6432%\FastestVPN\Resources\data doesn't exists. Is the FasttestVPN Windows client installed?.
  goto Exitbatch
)

:CopyConfig
if exist "%ProgramW6432%\FastestVPN\Resources\data\FastestVPNWireGuard.conf" (
  copy "%ProgramW6432%\FastestVPN\Resources\data\FastestVPNWireGuard.conf" FastestVPNWireGuard.conf
  echo Copy FastestVPNWireGuard.conf ended with rc %errorlevel%
  goto ProcessConfig
) else (
  echo file doesn't exist yet: Start a FastestVPN wireguard connection.
)

if exist "FastestVPNWireGuard.conf" (
  echo ======  Debugging ================
  goto ProcessConfig
) else (
  echo file doesn't exist yet: Start a FastestVPN wireguard connection.
)
timeout /t 1 /nobreak > NUL
goto CopyConfig

:ProcessConfig
if exist "FastestVPNWireGuard.conf" (
  echo ---- Found Wireguard config: ------------------------------------------------------------
  type FastestVPNWireGuard.conf
  echo ---- End Found Wireguard config: --------------------------------------------------------
  echo .
  echo -----------------------------------------------------------------------------------------
  echo Use this data for your docker gluetun settings:
  echo -----------------------------------------------------------------------------------------
  echo     environment:
  echo       - VPN_SERVICE_PROVIDER=fastestvpn
  echo       - VPN_TYPE=wireguard
  echo       - SERVER_COUNTRIES=##YourCountryOfChoice##
  For /F "Delims=\= tokens=1*" %%A in ('Type "FastestVPNWireGuard.conf"^|FIND /I "PrivateKey = "') Do (
    set "pk=%%B"
    rem trim leading spaces
    for /F "tokens=* eol= " %%S in ("!pk!") do set "pk=%%S"
    echo       - WIREGUARD_PRIVATE_KEY=!pk!
  )
  For /F "Delims=\= tokens=1*" %%A in ('Type "FastestVPNWireGuard.conf"^|FIND /I "Address = "') Do (
    set "ip=%%B"
    rem trim leading spaces
    for /F "tokens=* eol= " %%S in ("!ip!") do set "ip=%%S"
    echo       - WIREGUARD_ADDRESSES=!ip!
  )

) else (
    echo file doesn't exist yet. Try this process again.
)
:Exitbatch
echo -- END --
pause
```

Ouput of the script:

```txt
c:\temp>FastestVPN_Get_wireguard_config.bat
=========================================================================================
Manual Steps before this script can copy your Wireguard settings for FastestVPN:
=========================================================================================
 - First you need to start the FastestVPN Windows client
 - Ensure the protocol is set to wireguard in: Settings/VPN Protocol/Disable Auto and Select Wireguard
 - Press "any key" in this window so the bat chfile will start monitoring for: C:\Program Files\FastestVPN\Resources\data\FastestVPNWireGuard.conf
 - Start a vpn connection in the FastestVPN program
 - The batchscript should detect the creation of C:\Program Files\FastestVPN\Resources\data\FastestVPNWireGuard.conf and copy it to this script directory
Press any key to continue . . . 
Found FastestVPNWireGuard directory so start monitoring for FastestVPNWireGuard.conf .
file doesn't exist yet: Start a FastestVPN wireguard connection.
======  Debugging ================
---- Found Wireguard config: ------------------------------------------------------------
[Interface]
PrivateKey = wOEI9rqqbDwnN8/Bpp22sVz48T71vJ4fYmFWujulwUU=
Address = 10.64.222.21/32
DNS = 10.8.8.8

[Peer]
PublicKey = 658QxufMbjOTmB61Z7f+c7Rjg7oqWLnepTalqBERjF0=
AllowedIPs = 0.0.0.0/0
Endpoint = nl-01.jumptoserver.com:51820
---- End Found Wireguard config: --------------------------------------------------------
.
-----------------------------------------------------------------------------------------
Use this data for your docker gluetun settings:
-----------------------------------------------------------------------------------------
    environment:
      - VPN_SERVICE_PROVIDER=fastestvpn
      - VPN_TYPE=wireguard
      - SERVER_COUNTRIES=##YourCountryOfChoice##
      - WIREGUARD_PRIVATE_KEY=wOEI9rqqbDwnN8/Bpp22sVz48T71vJ4fYmFWujulwUU=
      - WIREGUARD_ADDRESSES=10.64.222.21/32
-- END --
Press any key to continue . . . 
```
