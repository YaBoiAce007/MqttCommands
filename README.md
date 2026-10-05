# Mosquitto MQTT Broker — Local Network Setup

This guide shows how to stop the **Mosquitto Windows Service** that normally listens on `localhost`, start Mosquitto manually on your PC's **LAN IP**, and test MQTT communication.

## 1. Stop the current Mosquitto Windows Service

Open **PowerShell as Administrator**:

```powershell
Stop-Service mosquitto
```

### What does this do?

The Mosquitto Windows Service automatically runs `mosquitto.exe` in the background.

In the current setup, this service listens on:

```text
127.0.0.1:1883
```

`127.0.0.1` means **localhost — this computer itself**.

Therefore, devices such as an ESP32 cannot connect to this listener through the PC's LAN IP.

The command:

```powershell
Stop-Service mosquitto
```

stops that Windows Service and frees port `1883`.

---

## 2. Check that port 1883 is free

Run:

```powershell
netstat -ano | findstr :1883
```

Ideally, nothing should be returned.

If you see something like:

```text
TCP    127.0.0.1:1883    ...    LISTENING
```

then something is still using port `1883`.

---

## 3. Create a Mosquitto configuration file

Create a temporary configuration file:

```powershell
notepad C:\mosquitto-test.conf
```

Put the following inside:

```text
listener 1883 192.168.29.210
allow_anonymous true
```

Save and close the file.

### What these settings mean

```text
listener 1883 192.168.29.210
```

Tells Mosquitto:

> Listen for MQTT connections on port `1883` using the PC's LAN IP `192.168.29.210`.

This allows devices on the same network, such as your ESP32, to connect to the broker.

```text
allow_anonymous true
```

Allows clients to connect without a username and password.

This is convenient for local testing and your prototype, but it should **not** be used when exposing an MQTT broker to the public Internet.

---

## 4. Start Mosquitto manually

Run:

```powershell
mosquitto -c C:\mosquitto-test.conf -v
```

You should see output indicating that Mosquitto is opening the listener on port `1883`.

Keep this PowerShell window open.

### Flags used here

| Flag | Meaning | Example |
|---|---|---|
| `-c` | **Config file** — tells Mosquitto which configuration to use | `-c C:\mosquitto-test.conf` |
| `-v` | **Verbose** — shows detailed information while Mosquitto is running | `-v` |

So:

```powershell
mosquitto -c C:\mosquitto-test.conf -v
```

basically means:

> Start Mosquitto using this configuration file and show me detailed output.

---

# 5. Test MQTT subscription

Open a **second PowerShell window**.

Run:

```powershell
mosquitto_sub -h 192.168.29.210 -p 1883 -t iv/bubble
```

This subscribes to the MQTT topic:

```text
iv/bubble
```

The terminal will appear to do nothing. That's normal.

It is simply **waiting for a message**.

### Flags used here

| Flag | Meaning | Example |
|---|---|---|
| `-h` | **Host** — the MQTT broker's IP address | `-h 192.168.29.210` |
| `-p` | **Port** — the MQTT broker's port | `-p 1883` |
| `-t` | **Topic** — the MQTT channel to subscribe to | `-t iv/bubble` |

So:

```powershell
mosquitto_sub -h 192.168.29.210 -p 1883 -t iv/bubble
```

means:

> Connect to the MQTT broker at `192.168.29.210:1883` and listen for messages on `iv/bubble`.

---

# 6. Publish a test message

Open a **third PowerShell window**.

Run:

```powershell
mosquitto_pub -h 192.168.29.210 -p 1883 -t iv/bubble -m "TEST"
```

The subscriber from Step 5 should display:

```text
TEST
```

### Additional flag: `-m`

| Flag | Meaning | Example |
|---|---|---|
| `-m` | **Message** — the data you want to publish | `-m "TEST"` |

So:

```powershell
-m "TEST"
```

means:

> Publish the message `TEST`.

---

# 7. MQTT flow

At this point, the communication looks like:

```text
                MQTT Broker
          192.168.29.210:1883
                    │
          ┌─────────┴─────────┐
          │                   │
     Subscriber            Publisher
     mosquitto_sub         mosquitto_pub
          │                   │
          │    iv/bubble      │
          └───────────────────┘
```

For your IV-bubble project, the ESP32 can eventually replace the publisher:

```text
ESP32
  │
  │  "BUBBLE DETECTED"
  ▼
MQTT Broker
  │
  │  iv/bubble
  ▼
Server / Application
```

---

# 8. Stop the manually started broker

Go back to the PowerShell window where this is running:

```powershell
mosquitto -c C:\mosquitto-test.conf -v
```

Press:

```text
Ctrl + C
```

This terminates the **manually started Mosquitto broker**.

---

# 9. Start the Windows Service again

If you want to return to the original Windows Service:

```powershell
Start-Service mosquitto
```

The Windows Service will start Mosquitto again using its configured settings.

In your original setup, that service listens on:

```text
127.0.0.1:1883
```

which means **localhost only**.

---

# Quick Flag Reference

```text
mosquitto
    -c    Config file
    -v    Verbose / detailed output

mosquitto_sub
    -h    Host / broker IP
    -p    Port
    -t    Topic

mosquitto_pub
    -h    Host / broker IP
    -p    Port
    -t    Topic
    -m    Message
```

## Complete command sequence

### Stop Windows Service

```powershell
Stop-Service mosquitto
```

### Check port

```powershell
netstat -ano | findstr :1883
```

### Create configuration

```powershell
notepad C:\mosquitto-test.conf
```

Configuration:

```text
listener 1883 192.168.29.210
allow_anonymous true
```

### Start manual broker

```powershell
mosquitto -c C:\mosquitto-test.conf -v
```

### Subscribe

```powershell
mosquitto_sub -h 192.168.29.210 -p 1883 -t iv/bubble
```

### Publish

```powershell
mosquitto_pub -h 192.168.29.210 -p 1883 -t iv/bubble -m "TEST"
```

### Stop manual broker

```text
Ctrl + C
```

### Start Windows Service again

```powershell
Start-Service mosquitto
```
