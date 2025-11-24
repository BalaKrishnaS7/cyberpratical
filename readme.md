

---

````md
# 🔍 Practical Report  
## **Analysis of Browser Session Restore Data on Linux Mint VM**

---

## **1. Aim**
To recover and analyze browser session restore data from a Linux Mint virtual machine in order to reconstruct **open tabs, browsing state, and unsent form data** prior to shutdown.

---

## **2. Objective**
- Identify Firefox session restore files.
- Decode `.jsonlz4` session data to readable JSON.
- Extract information about:
  - Open windows and tabs
  - Browser history per tab
  - Active tab at shutdown
  - Form data that was typed but not submitted

---

## **3. Tools & Environment**
| Component | Details |
|---------|--------|
| OS | Linux Mint (VM) |
| Browser | Mozilla Firefox |
| Tools Used | Terminal, `lz4jsoncat`, `jq` |
| Evidence Location | `~/.mozilla/firefox/<profile>/sessionstore-backups/` |

---

## **4. Theory / Background**
Firefox automatically creates session restore snapshots so that tabs can be restored after a crash or restart. These artifacts store:

- Tabs and URLs open at the time of shutdown  
- Navigation history for each tab  
- Scroll position & selected tab  
- **Unsubmitted form values typed by the user**

These snapshots are stored in **compressed `.jsonlz4` files**, which must be decoded for forensic analysis.

| Filename | Meaning |
|----------|---------|
| `recovery.jsonlz4` | Most recent session (written during last run) |
| `recovery.baklz4` | Backup of recent session |
| `previous.jsonlz4` | Session before last shutdown |
| `upgrade.jsonlz4-YYYYMMDDHHMMSS` | Session saved during browser upgrade |

---

## **5. Procedure**

### **5.1 Simulating browsing session**
1. Launched Firefox and opened multiple tabs.
2. Entered text into a webpage form (but did **not** submit it).
3. Simulated browser crash to generate session restore data:
   ```bash
   pkill firefox
````

### **5.2 Locating session restore files**

```bash
cd ~/.mozilla/firefox
cd <profile>.default-release/sessionstore-backups
ls
```

### **5.3 Identifying the most recent session file**

```bash
ls -lt
```

`recovery.jsonlz4` appeared as the latest modified file.

### **5.4 Decoding the session file**

```bash
lz4jsoncat recovery.jsonlz4 | jq '.' > ~/Desktop/recovery_session.json
```

### **5.5 Extracting browsing data**

✔ **List URLs of open tabs**

```bash
jq '.windows[].tabs[].entries[-1].url' ~/Desktop/recovery_session.json
```

✔ **Check active tab in each window**

```bash
jq '.windows[] | {selected_tab: .selected, tab_urls: [.tabs[].entries[-1].url]}' \
   ~/Desktop/recovery_session.json
```

✔ **Recover unsent form data**

```bash
jq '.. | objects | select(has("formdata")) |
   {url: .url, formdata: .formdata}' \
   ~/Desktop/recovery_session.json
```

---

## **6. Observations**

| Evidence Type             | Findings                                  |
| ------------------------- | ----------------------------------------- |
| Number of browser windows | *Example: 1*                              |
| Total open tabs           | *Example: 5*                              |
| Active tab at shutdown    | *Example: Tab 2*                          |
| Websites open             | *Extracted using jq*                      |
| Unsubmitted form data     | *Recovered text from login / search form* |

> Screenshots of terminal commands and JSON output may be added here.

---

## **7. Results**

From the decoded session file, the following were successfully recovered:

* List of URLs from all open tabs
* Order and structure of tabs across windows
* Active tab state before shutdown
* Complete browsing history from each tab
* Unsubmitted form values typed by the user

This confirms that Firefox session restore data can be used to reconstruct **detailed user activity** before shutdown.

---

## **8. Conclusion**

Firefox’s `.jsonlz4` session snapshot files contain highly informative data about user browsing activity including **URLs, tab history, and form inputs**. Using Linux Mint tools like `lz4jsoncat` and `jq`, it is possible to decode and analyze these files without requiring forensic suites. This technique is valuable for **digital forensics investigations**, but also demonstrates a **privacy risk** if unauthorized individuals gain access to browser profile directories.

---

## **9. References**

* Firefox Session Restore Mechanism
* Linux Mint Terminal Tools Documentation
* `lz4jsoncat` — Mozilla LZ4 JSON Decoder
* `jq` — JSON Parser & Formatter

---

### ✔ Report Completed

```

---

If you want, I can also create:

📌 **PPT slides**  
📌 **Flowchart of procedure**  
📌 **Viva exam questions & answers**

Just tell me what you need next. 🚀
```
