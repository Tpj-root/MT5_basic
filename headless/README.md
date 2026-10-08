# MT5_basic



```
https://nssm.cc/download
https://nssm.cc/ci/nssm-2.24-101-g897c7ad.zip
https://github.com/KravitzMC/MT5-stealth-mode
```



# Manual headless setup (no scripts)

You do the same thing the scripts do: register MT5 as a Windows Service with NSSM.

## Step 1: Prepare the files

1. Install MT5 normally and run it once. Log in to your broker to confirm the login works, then close it.
2. Download NSSM from **nssm.cc** and unzip it.
3. Copy `win64\nssm.exe` to a simple folder such as `C:\MT5Service\`.
4. Create `C:\MT5Service\common.ini`:

```ini
[Common]
Login=12345678
Password=your_password
Server=YourBroker-Server

[StartUp]
Symbol=XAUUSD

[Experts]
AllowDllImport=1
Enabled=1
```

Use your broker's exact server name and symbol name. A path without spaces makes this much easier.

## Step 2: Register the service

1. Open Command Prompt **as Administrator**.
2. Go to the folder:

```bat
cd C:\MT5Service
```

3. Run the NSSM installer window:

```bat
nssm.exe install MT5Service
```

4. A window opens. Fill in the **Application** tab:
   - **Path:** `C:\Program Files\MetaTrader 5\terminal64.exe`
   - **Startup directory:** `C:\Program Files\MetaTrader 5`
   - **Arguments:** `/config:C:\MT5Service\common.ini /portable`

5. Open the **Log on** tab:
   - Select **Local System account**.
   - Tick **Allow service to interact with desktop**.

6. Open the **Details** tab:
   - **Startup type:** Automatic.
   - **Display name:** MT5Service.

7. Open the **Exit actions** tab and set **Restart** as the action on exit, with a delay of about 5000 ms.
8. Open the **I/O** tab and set **Output (stdout)** to `C:\MT5Service\mt5_out.log` and **Error (stderr)** to `C:\MT5Service\mt5_err.log`. These logs help when something fails.
9. Click **Install service**.

## Step 3: Start it

Close any MT5 windows that are already open, then run:

```bat
nssm.exe start MT5Service
```

You can also open `services.msc`, find **MT5Service**, and click **Start**.

## Step 4: Check it works

- In `services.msc`, the status should read **Running**.
- In Task Manager → **Details** tab, `terminal64.exe` should be listed with the user name **SYSTEM**. No window appears on your desktop, so it is headless.
- Run the Python buy script. If the account info prints, it works.

## Handy commands

```bat
nssm.exe status MT5Service
nssm.exe restart MT5Service
nssm.exe stop MT5Service
nssm.exe remove MT5Service confirm
nssm.exe edit MT5Service
```

`nssm.exe edit` reopens the settings window, so you can change anything later. Restart the service after editing `common.ini`.

## Option B: Task Scheduler (no NSSM)

1. Open **Task Scheduler** and choose **Create Task** (not "Basic Task").
2. **General** tab:
   - Select **Run whether user is logged on or not**.
   - Tick **Run with highest privileges**.
3. **Triggers** tab: **At startup**.
4. **Actions** tab:
   - **Program:** `C:\Program Files\MetaTrader 5\terminal64.exe`
   - **Arguments:** `/config:C:\MT5Service\common.ini /portable`
   - **Start in:** `C:\Program Files\MetaTrader 5`
5. **Settings** tab: tick **If the task fails, restart every 1 minute**.
6. Click OK and enter your Windows password when asked.

This also runs MT5 with no visible window, but it restarts less reliably after a crash than NSSM.

## Troubleshooting

- **Service starts, then stops:** check `mt5_err.log`. The most common causes are a wrong path or a typo in the arguments.
- **Not logging in:** test by running `terminal64.exe /config:C:\MT5Service\common.ini /portable` by hand in Command Prompt. This shows errors on screen.
- **Python can't connect:** the service runs as SYSTEM, so a Python script running as your user may fail to attach. Try `mt5.initialize(path=r"C:\Program Files\MetaTrader 5\terminal64.exe")`, or run Python as the same account. I couldn't test this on a real machine, so verify it on your setup.
- **Security:** `common.ini` holds your password in plain text. Restrict the folder to Administrators, and use a demo account first.