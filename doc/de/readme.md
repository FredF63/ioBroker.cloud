![Logo](media/cloud.png)
# ioBroker cloud Adapter
Dieser Adapter ermöglicht die Verbindung vom Internet über die ioBroker-Cloud zur lokalen Installation von ioBroker.

## Einstellungen
### APP-KEY
Um den Cloud-Adapter verwenden zu können, muss zunächst ein Account unter [https://iobroker.net](https://iobroker.net) erstellt werden.

![Intro](media/intro.png)

Über Edit Profile dann den APP-Key kopieren und hier eintragen.

![Einstellungen]/media/Einstellungen.PNG

### Instanz
Alle Anfragen des Cloud-Adapter werden an eine WEB-Instanz weitergeleitet. Der Benutzer muss hier die WEB-Instanz (meist web.0) angeben, die dem Benutzer angezeigt wird, wenn er sich auf der Website https://iobroker.net anmeldet.

## Erweiterte Einstellungen
### Verbindungs-Timeout(ms)
Erklärung fehlt

### Ping-Timeout(ms)
Erklärung fehlt

### Sprache
Bei Auswahl der Systemsprache werden die Smart-Namen von Geräten und Aufzählungen nicht übersetzt. Wenn eine Sprache angegeben ist, werden alle bekannten Namen in diese Sprache übersetzt.
Dies dient zum schnellen wechseln zwischen den Sprachen.

### Erlaube selbst-signierte Zertifikate
Bei Verwendung der iobroker.net Cloud bleibt dies deaktiviert. Diese Option wird nur bei Verwendung einer eigenen Cloud benötigt.

### Platziere Funktionsnamen vorne
Hiermit wird die Reihenfolge von Funktion und Rolle in selbst generierten Namen geändert:

- deaktiviert : "Raum Funktion", Beispiel: "Wohnzimmer Dimmer"
- aktiviert: "Funktion Raum", Beispiel: "Dimmer Wohnzimmer"

### Verbinde Raum und Funktion mit
Hier kann ein Wort definiert werden, das zwischen Funktion und Raum platziert wird. Mit eingesetzem "im" wird aus "Dimmer Wohnzimmer" "Dimmer im Wohnzimmer".

Dies wird jedoch nicht empfohlen, da die Erkennungs-Engine ein weiteres Wort analysieren muss und dies zu Missverständnissen führen kann.

### Ersetze in Namen
Erklärung fehlt

### Aus-Pegel für Schalter
Einige Gruppen haben unterschiedliche Geräten: z.B. Dimmer und Schalter. Diese können mit den Befehlen "EIN" und "AUS" oder auch mit Prozentwerten gesteuert werden. Wenn der Befehl "Setze auf 30%" lautet und der Aus-Pegel auf "30%" eingestellt ist, werden die Schalter eingeschaltet. Mit dem Befehl "Setze auf 25%" werden alle Schalter ausgeschaltet die Dimmer werden auf 25% gesetzt, bleiben also an.

Mit dem Befehl "AUS" speichert der Adapter außerdem den aktuellen Dimmerpegel, wenn der tatsächliche Wert über oder gleich "30%" ist .
Mit einem neuen "EIN"-Befehl schaltet der Adapter einen Dimmer nicht auf 100%, sondern auf den gespeicherten Pegel.

Beispiel:

- Angenommen, der Aus-Pegel steht auf 30%.
- Ein virtuelles Gerät "Licht" hat zwei physische Geräte: *Schalter* und *Dimmer*.
- Mit dem Befehl: "Setze das Licht auf 40%" wird dieser Wert gespeichert für den *Dimmer*, setzt den "Dimmer" auf diesen Wert und schaltet den  *Schalter* ein.
- Command: "turn the light off". The adapter will set the *dimmer* to 0% and will turn off the *switch*.
- Command: "turn on the light". *dimmer* => 40%, *switch* => ON.
- Command: "set the light to 20%". *dimmer* => 20%, *switch* => OFF. The value for dimmer will not be remembered, because it is bellow *OFF level*.
- Command: "turn on the light". *dimmer* => 40%, *switch* => ON.

### by ON
You can select the behaviour of ON command will come for the number state. The specific value can be selected or last non zero value will be used.

### Schreibe Antwort ins
For every command the text response will be generated. You can define here the Object ID , where this text must be written to. E.g. *sayit.0.tts.text*.

### Neustart bei Verbindunsabbruch
Erklärung fehlt

### Erlaube Zugriff auf Admin (nur pro)
Erklärung fehlt

### Eigene Einstellungen (nur pro)
Erklärung fehlt

## Services und IFTTT
### IFTTT
[Anleitung IFTTT](doc/ifttt.md)

### White list für Services
There is a possibility to send messages to cloud adapter.
If you call ```[POST]https://iobroker.net/service/custom_<NAME>/<user-app-key>``` und value as payload.

```
curl --data "myString" https://iobroker.net/service/custom_test/<user-app-key>
```

If you set in the settings the field "White list for services" the name *custom_test*, and call with "custom_test" as the service name, the state **cloud.0.services.custom_test** will be set to *myString*.

You may write "*" in white list and all services will be allowed.

From version 2.0.5 you can use GET request in form ```[GET]https://iobroker.net/service/custom_<NAME>/<user-app-key>/<data>``` to place the **\<data\>** into **cloud.0.services.custom_\<NAME\>**.

Here you can find instructions how to use it with [tasker](doc/tasker.md).

IFTTT service is allowed only if IFTTT key is set.

Reserved names are "ifttt", "text2command", "simpleApi", "swagger". These must be used without the ```"custom_"``` prefix.

### text2command
You may write "text2command" in white list, you can send POST request to ```https://iobroker.net/service/text2command/<user-app-key>``` to write data into *text2command.X.text* variable.

"X" can be defined in settings by the "Use text2command instance" option.

### simpleApi
*to do*

### Lock
Um die Sperren sperren zu können, muss der Status die Rolle "switch.lock" und native.LOCK_VALUE haben, um den Sperrstatus zu bestimmen.

```
Alexa, is "lock name" locked/unlocked
Alexa, lock the "lock name"
```

## How names will be generated
The adapter tries to generate virtual devices for smart home control (e.g. Amazon Alexa or Google Home).

The are two important enumerations for that: rooms and functions.

Rooms are like: living room, bath room, sleeping room.
Functions are like: light, blind, heating.

Following conditions must be met to get the state in the automatically generated list:

- the state must be in some "function" enumeration.
- the state must have role ("state", "switch" or "level.*", e.g. level.dimmer) if not directly included into "functions".
It can be that the channel is in the "functions", but state itself not.
- the state must be writable: common.write = true
- the state dimmer must have common.type as 'number'
- the state heating must have common.unit as '°C', '°F' or '°K' and common.type as 'number'

If the state is only in "functions" and not in any "room", the name of state will be used.

The state names will be generated from function and room. E.g. all *lights* in the *living room* will be collected in the virtual device *living room light*.
The user cannot change this name, because it is generated automatically.
But if the enumeration name changes, this name will be changed too. (e.g. function "light" changed to "lights", so the *living room light* will be changed to *living room lights*)

All the rules will be ignored if the state has common.smartName. In this case just the smart name will be used.

if *common.smartName* is **false**, the state or enumeration will not be included into the list generation.

The configuration dialog lets comfortable remove and add the single states to virtual groups or as single device.
![Configuration](img/configuration.png)

If the group has only one state it can be renamed, as for this the state's smartName will be used.
If the group has more than one state, the group must be renamed via the enumeration's names.

To create own groups the user can install "scenes" adapter or create "script" in Javascript adapter.

### Replaces
You can specify strings, that could be automatically replaced in the devices names. E.g if you set replaces to:
```.STATE,.LEVEL```, so all ".STATE" and ".LEVEL" will be deleted from names. Be careful with spaces.
If you will set ```.STATE, .LEVEL```, so ".STATE" and " .LEVEL" will be replaced and not ".LEVEL".

## Helper states
- **smart.lastObjectID**: This state will be set if only one device was controlled by home skill (alexa, google home).
- **smart.lastFunction**: Function name (if exists) for which last command was executed.
- **smart.lastRoom**:     Room name (if exists) for which last command was executed.
- **smart.lastCommand**:  Last executed command. Command can be: true(ON), false(OFF), number(%), -X(decrease at x), +X(increase at X)
- **smart.lastResponse**: Textual response on command. It can be sent to some text2speech (sayit) engine.



