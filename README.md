
# 📶 Fixing “wlan0: No secrets were provided” on Radxa (NetworkManager)

A practical, terminal-based guide to diagnosing and fixing the common Linux Wi-Fi error:

> **`wlan0: No secrets were provided`**

This issue typically occurs when **NetworkManager has a broken or incomplete Wi-Fi connection profile**—most often missing credentials—rather than a driver or service failure.

Tested on ARM boards (Radxa Rock 5C), but applicable to most Linux systems using NetworkManager.

---

## 🧠 What’s Actually Going Wrong?

* NetworkManager **is running**
* Wi-Fi hardware **is detected**
* Scanning for networks **works**
* Connection fails because:

  * A saved Wi-Fi profile exists **without a password**
  * NetworkManager keeps reusing that broken profile

The fix is to **delete the bad profile and reconnect with explicit credentials**.

---

## ✅ Prerequisites

* Linux system using **NetworkManager**
* Terminal access
* `nmcli` available (comes with NetworkManager)

---

## 🔍 Step-by-Step Fix

### Step 1️⃣ Check Wi-Fi device state

Run:

```bash
nmcli device status
```

Expected output looks like:

```
DEVICE   TYPE   STATE        CONNECTION
wlan0    wifi   disconnected --
```

States like `disconnected`, `connecting`, or `failed` are fine — we just want to confirm the device exists.

---

### Step 2️⃣ List saved Wi-Fi connections

This error often comes from a **saved profile with no password**.

```bash
nmcli connection show
```

If you see your Wi-Fi SSID listed, delete it:

```bash
nmcli connection delete "YourSSID"
```

> ⚠️ Even if you think it’s correct — delete it.
> We want a clean slate.

---

### Step 3️⃣ Reconnect *with* password (important)

Scan for available networks:

```bash
nmcli dev wifi list
```

Then connect explicitly, providing the password:

```bash
nmcli dev wifi connect "YourSSID" password "YourPassword"
```

> ⚠️ Quotes matter if your SSID or password contains spaces or symbols.

This creates a **new, valid connection profile** with stored credentials.

---

### Step 4️⃣ If it still says “No secrets were provided”

Confirm how NetworkManager sees the network:

```bash
nmcli -f IN-USE,SSID,SECURITY,SIGNAL dev wifi list
```

Check:

* **SSID**
* **SECURITY** column (e.g. `WPA2`, `WPA3`, `--`)

If the network uses **WPA3**, older NetworkManager versions may struggle.

---

## 🧩 Common Causes on ARM Boards (Radxa, SBCs)

* Headless setup → password never prompted
* Auto-generated Wi-Fi profile missing secrets
* Switching between WPA2 and WPA3
* Reusing a corrupted connection profile

---

## 🧠 Mental Model (Very Important)

* **Devices** → hardware (`wlan0`)
* **Connections** → saved profiles (SSID + security + password)

Most Wi-Fi issues are **connection problems**, not device problems.

If scanning works but connecting fails → **delete the connection profile**.

---

## 🛠️ Quick Recovery Cheat Sheet

```bash
nmcli connection show
nmcli connection delete "BadSSID"
nmcli dev wifi connect "SSID" password "YourPassword"
```

---

## ✅ Status

Once the above steps work:

* Wi-Fi driver ✔
* Firmware ✔
* NetworkManager ✔
* Credentials ✔

You’re good to go 📶

---

