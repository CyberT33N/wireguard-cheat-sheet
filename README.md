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

v2:
```
#!/bin/bash

# --- Konfiguration ---
# Wähle den zu verwendenden WireGuard Server/Interface-Namen
# Der Name sollte dem Dateinamen in /etc/wireguard/ entsprechen (ohne .conf)
# Beispiel: wg0, wg-CH-CZ-1, main_os-CH-AT-1, wg-IS-CZ-1
# Aktuelle Auswahl:
SERVER="wg-CH-DK-2" # Stelle sicher, dass /etc/wireguard/wg-CH-DK-2.conf existiert

# --- Funktion zur Deaktivierung ---
# Stoppt eine spezifische WireGuard-Schnittstelle, aber nur, wenn sie gerade läuft.
# Verwendet 'wg show <interface>' um zu prüfen und 'wg-quick down <interface>' zum Stoppen.
# Argument $1: Der Name der WireGuard-Schnittstelle (z.B. wg0, wg-ch-dk-2)
stop_wireguard_if_running() {
  local interface_name="$1"
  if [ -z "$interface_name" ]; then
    echo "Fehler: Kein Schnittstellenname an stop_wireguard_if_running übergeben." >&2
    return 1 # Fehler signalisieren
  fi

  echo "Prüfe Status der WireGuard-Schnittstelle '$interface_name'..."
  # Prüfen, ob die Schnittstelle mit 'wg show' angezeigt wird (indikativ dafür, dass sie konfiguriert/aktiv ist)
  if sudo wg show "$interface_name" &> /dev/null; then
    echo "Schnittstelle '$interface_name' scheint aktiv/konfiguriert zu sein. Versuche sie zu stoppen..."
    if sudo wg-quick down "$interface_name"; then
      echo "Schnittstelle '$interface_name' erfolgreich via wg-quick down gestoppt."
    else
      # wg-quick down kann fehlschlagen, wenn die Schnittstelle nicht via wg-quick gestartet wurde
      # oder bereits unten ist. Wir geben eine Warnung aus.
      echo "Warnung: 'sudo wg-quick down $interface_name' ist fehlgeschlagen oder hatte keine Wirkung (war evtl. schon unten?)."
    fi
  else
    echo "Schnittstelle '$interface_name' ist nicht aktiv oder nicht konfiguriert (laut 'wg show')."
  fi
}

# --- Skriptstart ---
echo "Skript gestartet. Konfigurierter Server/Schnittstelle: $SERVER"

# Stelle sicher, dass die konfigurierte Schnittstelle zu Beginn nicht läuft.
# (Hinweis: Das ursprüngliche Skript hat *alle* laufenden WG-Interfaces gestoppt.)
echo "Führe initiale Bereinigung für '$SERVER' durch..."
stop_wireguard_if_running "$SERVER"

echo "Starte Überwachungsschleife für die Internetverbindung..."
# Prüft kontinuierlich die Internetverbindung
while true; do
    # Teste mit einer HTTP-Anfrage, falls Ping nicht funktioniert
    # Option: -I <interface> um über spezifisches Interface zu testen? Kann komplex sein.
    echo "Prüfe Internetverbindung..."
    if ! curl -s --max-time 5 http://google.com > /dev/null; then
        echo "Keine Internetverbindung erkannt!"

        # WireGuard neu starten
        echo "Versuche WireGuard ($SERVER) neu zu starten..."

        # Schritt 1: Interface stoppen, falls es läuft (via Funktion)
        # Kommentar: via get erst deaktivieren wenn es läuft -> wird hier durch die Funktion erledigt.
        stop_wireguard_if_running "$SERVER"

        # Schritt 2: Starte WireGuard neu
        echo "Starte WireGuard-Schnittstelle '$SERVER'..."
        if sudo wg-quick up "$SERVER"; then
          echo "WireGuard '$SERVER' erfolgreich gestartet."
        else
          echo "FEHLER beim Starten von WireGuard '$SERVER'. Prüfe Konfiguration und Logs."
          # Optional: Hier könnte man eine Fehlerbehandlung einfügen
          # (z.B. Skript beenden, anderen Server versuchen, länger warten)
        fi

        # Schritt 3: Warte nach Neustartversuch, bevor neu geprüft wird
        echo "Warte 10 Sekunden nach Neustartversuch..."
        sleep 10
    else
        # echo "Internetverbindung ist aktiv." # Kann zu viel Output sein, bei Bedarf einkommentieren
        : # Mache nichts, wenn die Verbindung okay ist (Platzhalter für Lesbarkeit)
    fi

    # Prüfe alle 2 Sekunden erneut
    # echo "Warte 2 Sekunden bis zur nächsten Prüfung..." # Optionaler Log
    sleep 2
done

echo "Skript beendet (sollte in der Endlosschleife eigentlich nicht passieren)."
```



v1
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
