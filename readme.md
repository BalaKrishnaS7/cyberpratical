
# 🔍 Practical Report  
## **Analysis of Browser Session Restore Data on Linux Mint VM**

---

## **1. Aim**
To recover and analyze browser session restore data from a Linux Mint virtual machine in order to reconstruct **open tabs, browsing state, and unsubmitted form data** prior to shutdown.

---

## **2. Objectives**
- Identify Firefox session restore artifacts.
- Decode `.jsonlz4` files into readable JSON format.
- Extract information related to:
  - Open windows and tabs
  - Browsing history per tab
  - Active tab at the time of shutdown
  - Form data typed but not submitted

---

## **3. Tools & Environment**
| Component | Details |
|----------|--------|
| Operating System | Linux Mint (Virtual Machine) |
| Browser | Mozilla Firefox |
| Tools Used | Terminal, `lz4jsoncat`, `jq` |
| Evidence Directory | `~/.mozilla/firefox/<profile>/sessionstore-backups/` |

---

## **4. Theory / Background**
Firefox maintains **automatic session restore data** so tabs can be restored after a crash or restart. These store:

- URLs of open tabs  
- Navigation history for each tab  
- Active tab index  
- Scroll position  
- **Unsubmitted form values typed by the user**

Session restore files use the **compressed `.jsonlz4` format**, so they must be manually decoded.

| Filename | Meaning |
|----------|---------|
| `recovery.jsonlz4` | Most recent session |
| `recovery.baklz4` | Backup of recent session |
| `previous.jsonlz4` | Session prior to last restart |
| `upgrade.jsonlz4-YYYYMMDDHHMMSS` | Session saved during browser upgrade |

---

## **5. Procedure**

### **5.1 Simulating browsing activity**
1. Launched Firefox and opened multiple tabs.
2. Entered text into a webpage form without submitting it.
3. Simulated a crash to trigger session restore creation:
   ```bash
   pkill firefox
````
````
## **5.2 Locating session restore files**

```bash
cd ~/.mozilla/firefox
cd <profile>.default-release/sessionstore-backups
ls
```

### **5.3 Identify the latest session snapshot**

```bash
ls -lt
```

### **5.4 Decode the session file**

```bash
lz4jsoncat recovery.jsonlz4 | jq '.' > ~/Desktop/recovery_session.json
```

### **5.5 Extract browsing artifacts**

✔ URLs of open tabs

```bash
jq '.windows[].tabs[].entries[-1].url' ~/Desktop/recovery_session.json
```

✔ Active tab per window

```bash
jq '.windows[] | {selected_tab: .selected, tab_urls: [.tabs[].entries[-1].url]}' \
   ~/Desktop/recovery_session.json
```

✔ Recover unsubmitted form data

```bash
jq '.. | objects | select(has("formdata")) | {url: .url, formdata: .formdata}' \
   ~/Desktop/recovery_session.json
```

---

## **6. Observations**

| Evidence Type          | Findings (Example)   |
| ---------------------- | -------------------- |
| Number of windows      | 1                    |
| Total open tabs        | 5                    |
| Active tab at shutdown | Tab 2                |
| Websites open          | Extracted using `jq` |
| Unsubmitted form data  | Retrieval successful |

> Screenshots of terminal logs and JSON content can be inserted here.

---

## **7. Results**

The decoded session file provided:

* List of all URLs from open tabs
* Tab order and active tab data
* Full browsing history per tab
* **Unsubmitted form values typed before shutdown**

---

## **8. Conclusion**

Firefox session restore files contain detailed data useful for **digital forensics**, showing user activity prior to shutdown. Using `lz4jsoncat` and `jq` on Linux Mint, these artifacts can be decoded without specialized forensic tools.
This also highlights a **privacy risk** if unauthorized users gain access to browser profile folders.

---

## **9. References**

* Firefox Session Restore File Mechanics
* Linux Mint Tool Documentation
* `lz4jsoncat` — Mozilla LZ4 JSON Decoder
* `jq` — Command-line JSON processor

---


