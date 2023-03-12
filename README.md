# Kostal-Plenticore-NodeRed
Node-Red Flow for reading and writing Modbus-Register

**Achtung! Alls auf eigene Gefahr. Mit Zugriff auf den Kostal Wechselrichter können Schäden nicht ausgeschlossen werden.**

Natürlich übernehme ich keine Haftung für Schäden, die der Einsatz meines Node-Red Flows verursachen sollte.
Jeder, der dies nutzt, sollte wissen, was er damit anstellen kann!

*Achtung: der Flow enthält noch keine Fehlerbehandlung. d.h. Werte werden "as is" übertragen und werden nicht auf Plausibilität geprüft. Auch andere potentielle Fehler werden nicht abgefangen.

# Grundvoraussetzungen

* Kostal Plenticore Plus 10 (ohne Batteriespeicher macht vieles hier keinen Sinn), ab Firmware 1.56
* Freigeschaltete "Externe Batteriesteuerung"
* Node-Red v3 (könnte auch mit v2 laufen, ist aber entwickelt und getestet mit v2)
* MQTT-Server
* InfluxDB-Server (optional)
* installierte Node-Red Nodes: node-red-contrib-modbus, node-red-contrib-influxdb (optional)

# Erläuterung

Der Flow greift auf die Modbus-Schnittstelle des Kostal Plenticore zu und liest zunächst die entsprechenden Register lt. Dokumentation von Kostal aus. Hierzu werden die jeweiligen Register entweder alle 60 Minuten (für Statistiken und andere informative Informationen) oder alle 2 Sekunden (für die Live-Werte) abgefragt und in jeweils einzelne MQTT-Topics (unterhalb devices/Kostal) verpackt an einen MQTT-Broker verschickt.

Der Flow erwartet unterhalb des MQTT-Topics devices/Kostal folgende Topics mit validen Werten:
* batteryChargePowerDCAbsolute
* batteryMaxChargePowerLimit
* batteryMaxDischargePowerLimit
* minimumSoc
* maximumSoc

Diese Werte können einzeln gesendet werden (wenn nur der minimumSoC beeinflusst werden soll) oder alle fünf auf einmal gesendet werden. Wenn ein Wert gesendet wurde, wird die Automatik gestartet und jeder mit dem entsprechenden MQTT-Topic geschickte Wert wird an den entsprechenden Modbus Register geschickt. Während des Intervalls nicht empfangene Topics werden ignoriert, also nicht an den Wechselrichter gesendet.

Sobald innerhalb 30 Sekunden keine Werte mehr gesendet werden, wird die Automatik gestoppt und es werden auch keine Modbus-Register geschrieben. Der Kostal Plenticore wird nun wieder in den "Automatik-Modus" wechseln.

_Hinweis: Die Logik geht von einem gesetzten Intervall von 60 Sekunden bei der Einstellung im Batterie-Menü des Wechselricthers unter "Timeout ext. Batteriesteuerung [s]" aus. Wenn der Timeout kürzer eingestellt wurde, muss natürlich auch der Flow entsprechend angepasst werden.

# Konfiguration

Anzupassen sind natürlich die IP-Adressen in den nodes für Modbus und MQTT, sowie optional bei der InfluxDB. Natürlich können andere Register mit ausgelesen werden oder geschrieben werden. Die MQTT-Topics sind in den einzelnen "function"-Topics schnell änderbar.
