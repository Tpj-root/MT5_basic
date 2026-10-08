# Add "Open in CMD" to the right-click menu (Windows 10)

## Method 1: Registry file (easiest)

1. Open **Notepad** and paste this:

```reg
Windows Registry Editor Version 5.00

; Right-click on empty space inside a folder
[HKEY_CLASSES_ROOT\Directory\Background\shell\OpenCmdHere]
@="Open in CMD"
"Icon"="cmd.exe"

[HKEY_CLASSES_ROOT\Directory\Background\shell\OpenCmdHere\command]
@="cmd.exe /s /k pushd \"%V\""

; Right-click on a folder itself
[HKEY_CLASSES_ROOT\Directory\shell\OpenCmdHere]
@="Open in CMD"
"Icon"="cmd.exe"

[HKEY_CLASSES_ROOT\Directory\shell\OpenCmdHere\command]
@="cmd.exe /s /k pushd \"%V\""

; Right-click on a drive (C:, D: ...)
[HKEY_CLASSES_ROOT\Drive\shell\OpenCmdHere]
@="Open in CMD"
"Icon"="cmd.exe"

[HKEY_CLASSES_ROOT\Drive\shell\OpenCmdHere\command]
@="cmd.exe /s /k pushd \"%V\""
```

2. Click **File → Save As**. Set **Save as type** to **All Files**, and name it `add_open_cmd.reg`.
3. Double-click the file, click **Yes**, then **OK**.
4. Right-click inside any folder. You should see **Open in CMD**. It works immediately, with no restart needed.

## Method 2: Open CMD as Administrator

This is useful for NSSM and service commands. Add it to the same file, or make a second `.reg` file:

```reg
Windows Registry Editor Version 5.00

[HKEY_CLASSES_ROOT\Directory\Background\shell\OpenCmdAdmin]
@="Open CMD as Administrator"
"Icon"="cmd.exe"

[HKEY_CLASSES_ROOT\Directory\Background\shell\OpenCmdAdmin\command]
@="powershell.exe -WindowStyle Hidden -Command \"Start-Process cmd.exe -ArgumentList '/s /k pushd \\\"%V\\\"' -Verb RunAs\""
```

Windows shows a UAC prompt each time you click it.

## Method 3: No registry (built-in trick)

Click the **address bar** in File Explorer, type `cmd`, and press Enter. CMD opens in that folder.

## Remove it later

Save this as `remove_open_cmd.reg` and double-click it:

```reg
Windows Registry Editor Version 5.00

[-HKEY_CLASSES_ROOT\Directory\Background\shell\OpenCmdHere]
[-HKEY_CLASSES_ROOT\Directory\shell\OpenCmdHere]
[-HKEY_CLASSES_ROOT\Drive\shell\OpenCmdHere]
[-HKEY_CLASSES_ROOT\Directory\Background\shell\OpenCmdAdmin]
```

## Notes

- Editing the registry needs an administrator account. If you get an "access denied" error, right-click the `.reg` file and choose **Merge**, or run it from an Administrator account.
- On Windows 10, holding **Shift** while you right-click already shows **Open PowerShell window here**. CMD is no longer in that menu, which is why this entry is needed.
- If you ever move to Windows 11, the entry appears under **Show more options**.