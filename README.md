# wireguard-cheat-sheet

# Down

## Down all
```shell
if [ -n "$(sudo wg show interfaces)" ]; then sudo wg show interfaces | xargs -n1 sudo wg-quick down; else echo "No WireGuard interfaces up, chill mal 😎"; fi
```




# Connect

```shell
sudo wg-quick up main_os-CH-AT-1
```



<br><br>

# Utils

## Retry
```
#!/bin/bash

# SERVER=wg-CH-CZ-1
# SERVER=main_os-CH-AT-1
# mSERVER=wg-IS-CZ-1

# Denmark (via Switzerland)
SERVER=wg-CH-DK-2

if [ -n "$(sudo wg show interfaces)" ]; then sudo wg show interfaces | xargs -n1 sudo wg-quick down; else echo "No WireGuard interfaces up, chill mal 😎"; fi

# Prüft kontinuierlich die Internetverbindung
while true; do
    # Teste mit einer HTTP-Anfrage, falls Ping nicht funktioniert
    if ! curl -s --max-time 5 http://google.com > /dev/null; then
        echo "Keine Internetverbindung. Starte WireGuard neu..."
        
        # WireGuard erst deaktivieren, wenn es läuft
        if sudo wg show main_os-CH-AT-1 &> /dev/null; then
            sudo wg-quick down $SERVER
        fi
        
        # Starte WireGuard neu
        sudo wg-quick up $SERVER
        
        # Warte 10 Sekunden nach Neustart, bevor neu geprüft wird
        sleep 10
    fi

    echo "Internet still alive"
    # Prüfe alle 10 Sekunden erneut
    sleep 2
done

```
