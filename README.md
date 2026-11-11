# EaseFilter Process Filter Driver SDK

[![Platform](https://img.shields.io/badge/Platform-Windows-blue.svg)]()
[![Language](https://img.shields.io/badge/Language-C%23%20%7C%20C++%20%7C%20Python%20%7C%20Rust%20%7C%20Java-success.svg)]()
[![License](https://img.shields.io/badge/License-Commercial-orange.svg)]()

## 🛡️ Overview

The **EaseFilter Process Filter Driver SDK** is a robust kernel-mode development kit designed for Windows process monitoring and protection. Operating deep within the Windows executive layer, it intercepts process and thread operations—such as creation, termination, and handle requests—before they reach their intended target.

This SDK empowers developers to build enterprise-grade security, auditing, and continuous monitoring applications. It is ideal for system administrators and security engineers who need to block malware, audit application usage, and sandbox untrusted executables in real-time.

---

## 🏗️ Architecture

The EaseFilter ecosystem relies on a highly efficient client-server architecture bridging kernel space and user space:

1. **Kernel-Mode Component (`EaseFlt.sys`)**: The core filter driver residing in the Windows kernel. It intercepts system calls related to processes and threads via native Windows kernel callbacks (e.g., `PsSetCreateProcessNotifyRoutineEx`).
2. **User-Mode Wrapper (`FilterAPI.dll`)**: A dynamic link library handling high-speed communication between your user-mode application and the kernel driver.
3. **Managed Control Layer (`EaseFilter.FilterControl`)**: A .NET wrapper available via NuGet, allowing seamless integration for C#, F#, and VB.NET applications.

![EaseFilter Process SDK Architecture](https://www.easefilter.com/images/process-filter-driver-architecture.jpg)

---

## ✨ Core Capabilities

* **Process and Thread Monitoring**: Real-time callback notifications for process creation (capturing PID, executable path, command-line arguments), termination, and thread tracking.
* **Active Protection and Blocking**: Definitively allow or deny execution at the kernel level. Prevent untrusted binaries from running before the OS allocates memory.
* **Process-Based File Access Control**: Integrate with the File System Filter Driver to sandbox applications (e.g., grant `notepad.exe` read-only access to system folders, or block specific processes from extracting data).
* **Dynamic Process Evaluation**: Inspect process events programmatically and block them dynamically in real-time by modifying the event's `ReturnStatus`.

---

## 🚀 Getting Started

### 1. Basic Initialization & Monitoring (C#)

Define a rule, configure control flags, and start monitoring specific directories.

```csharp
using System;
using EaseFilter.FilterControl;

namespace ProcessMonitorApp {
    class Program {
        static FilterControl filterControl = new FilterControl();

        static void Main(string[] args) {
            string lastError = string.Empty;
            string licenseKey = "Email us to request a trial key: info@easefilter.com";
            
            FilterAPI.FilterType filterType = FilterAPI.FilterType.PROCESS_FILTER;
            
            try {
                // Setup the filter rule targeting an untrusted folder
                ProcessFilterRule rule = new ProcessFilterRule();
                rule.ProcessNameFilterMask = "c:\\untrusted_folder\\*";
                
                // Set flags: Block execution AND request notifications
                rule.ControlFlag = FilterAPI.ProcessControlFlag.DENY_NEW_PROCESS_CREATION | 
                                   FilterAPI.ProcessControlFlag.PROCESS_CREATION_NOTIFICATION;
                
                filterControl.OnProcessCreation += FilterControl_OnProcessCreation;
                filterControl.AddProcessFilterRule(rule);
                
                if (!filterControl.StartFilter(filterType, 5, 10, licenseKey, ref lastError)) {
                    Console.WriteLine("Failed to start filter: " + lastError);
                    return;
                }
                
                Console.WriteLine("Driver started successfully. Press any key to exit.");
                Console.ReadKey();
            } finally {
                filterControl.StopFilter();
            }
        }

        private static void FilterControl_OnProcessCreation(object sender, ProcessEventArgs e) {
            Console.WriteLine($"[BLOCKED] Attempted launch: {e.ImageFileName} by PID {e.ParentProcessId}");
        }
    }
}
```

---

## 💡 Common Use Cases

### Application Sandboxing (Preventing Child Processes)
Vulnerable applications like Office or PDF readers are often exploited to launch malicious scripts. Sandbox them to prevent child process creation.

```csharp
ProcessFilterRule sandboxRule = new ProcessFilterRule();
sandboxRule.ProcessNameFilterMask = "*WINWORD.EXE";

// Deny child processes and enable logging
sandboxRule.ControlFlag = FilterAPI.ProcessControlFlag.DENY_CHILD_PROCESS_CREATION | 
                          FilterAPI.ProcessControlFlag.PROCESS_CREATION_NOTIFICATION;

filterControl.AddProcessFilterRule(sandboxRule);
filterControl.OnProcessCreation += (sender, e) => {
    Console.WriteLine($"[SANDBOX BLOCK] PID {e.ParentProcessId} attempted to launch {e.ImageFileName}");
};
```

### Endpoint Agent Protection (Anti-Kill)
Protect your own security software or critical services from being terminated by malware or unauthorized users.

```csharp
ProcessFilterRule protectionRule = new ProcessFilterRule();
protectionRule.ProcessNameFilterMask = "*MySecureAgent.exe";

// Request a synchronous callback BEFORE the process is terminated
protectionRule.ControlFlag = FilterAPI.ProcessControlFlag.PROCESS_PRE_TERMINATION_REQUEST;

filterControl.AddProcessFilterRule(protectionRule);
// You can then handle the pre-termination event to deny the request
```

---

## A C# process monitoring and protection example.
With the process monitoring and protection example, it enables your application to prevent the untrusted executable binaries ( malwares) from being launched, protect your data being damaged by the untrusted processes. It also enables your application to get the callback notification for the process/thread creation or termination, from the new process information you can get the parent process Id and thread Id of the new created process, you also can get the exact file name that is used to open the executable file and the command line that is used to execute the process if it is available.

![Process monitoring and protection](https://www.easefilter.com/images/processScreenshot.png)

[Read more about process filter example](https://www.easefilter.com/Forums_Files/Process-Monitor.htm)


## EaseFilter File System Filter Driver SDK Reference
| Product Name | Description |
| --- | --- |
| [Cloud File System SDK](https://www.easefilter.com/cloud/cloud-file-system-sdk.htm) | EaseFilter Cloud File System SDK Introduction. |
| [CloudTier Storage Tiering SDK](https://www.easefilter.com/cloud/storage-tiering-sdk.htm) | EaseFilter Storage Tiering Filter Driver SDK Introduction. |
| [File Monitor SDK](https://www.easefilter.com/kb/file-monitor-filter-driver-sdk.htm) | EaseFilter File Monitor Filter Driver SDK Introduction. |
| [File Control SDK](https://www.easefilter.com/kb/file-control-file-security-sdk.htm) | EaseFilter File Control Filter Driver SDK Introduction. |
| [File Encryption SDK](https://www.easefilter.com/kb/transparent-file-encryption-filter-driver-sdk.htm) | EaseFilter Transparent File Encryption Filter Driver SDK Introduction. |
| [Registry Filter SDK](https://www.easefilter.com/kb/registry-filter-drive-sdk.htm) | EaseFilter Registry Filter Driver SDK Introduction. |
| [Process Filter SDK](https://www.easefilter.com/kb/process-filter-driver-sdk.htm) | EaseFilter Process Filter Driver SDK Introduction. |
| [EaseFilter SDK Programming](https://www.easefilter.com/kb/programming.htm) | EaseFilter Filter Driver SDK Programming. |

## EaseFilter SDK Sample Projects
| Sample Project | Description |
| --- | --- |
| [CloudTier Storage Tiering Demo](https://www.easefilter.com/cloud/cloudtier-storage-tiering-demo.htm) | A HSM File System Filter Driver Demo. |
| [CloudTier S3 Tiering Demo](https://www.easefilter.com/cloud/cloudtier-s3-intelligent-tiering-demo.htm) | CloudTier S3 Intelligent Tiering Demo. |
| [Cloud File DR S3 Demo](https://www.easefilter.com/cloud/cloud-file-dr-demo.htm) | Cloud File DR S3 Demo. |
| [Amazon S3 File Explorer Demo](https://www.easefilter.com/cloud/s3-browser-demo.htm) | Amazon S3 File Explorer Demo. |
| [Auto File DRM Encryption](https://www.easefilter.com/kb/auto-file-drm-encryption-tool.htm) | Auto file encryption with DRM data embedded. |
| [Transparent File Encrypt](https://www.easefilter.com/kb/AutoFileEncryption.htm) | Transparent on access file encryption. |
| [Secure File Sharing with DRM](https://www.easefilter.com/kb/DRM_Secure_File_Sharing.htm) | Secure encrypted file sharing with digital rights management. |
| [File Monitor Example](https://www.easefilter.com/kb/file-monitor-demo.htm) | Monitor file system I/O in real time, tracking file changes. |
| [File Protector Example](https://www.easefilter.com/kb/file-protector-demo.htm) | Prevent sensitive files from being accessed by unauthorized users or processes. |
| [FolderLocker Example](https://www.easefilter.com/kb/FolderLocker.htm) | Lock file automatically in a FolderLocker. |
| [Process Monitor](https://www.easefilter.com/kb/Process-Monitor.htm) | Monitor the process creation and termination, block unauthorized process running. |
| [Registry Monitor](https://www.easefilter.com/kb/RegMon.htm) | Monitor the Registry activities, block the modification of the Registry keys. |
| [Secure Sandbox Example](https://www.easefilter.com/kb/Secure-Sandbox.htm) |A secure sandbox example, block the processes accessing the files out of the box. |
| [FileSystemWatcher Example](https://www.easefilter.com/kb/FileSystemWatcher.htm) | File system watcher, logging the file I/O events. |
| [ZeroTrust Example](https://www.easefilter.com/kb/zero-trust-file-access-control-demo.htm) | Zero trust file access control with encryption feature. |

## Filter Driver Reference

* [Understand MiniFilter Driver](https://www.easefilter.com/kb/understand-minifilter.htm)
* [Understand File I/O](https://www.easefilter.com/kb/File_IO.htm)
* [Understand I/O Request Packets(IRPs)](https://www.easefilter.com/kb/understand-irps.htm)
* [Filter Driver Developer Guide](https://www.easefilter.com/kb/DeveloperGuide.htm)
* [MiniFilter Filter Driver Framework](https://www.easefilter.com/kb/minifilter-framework.htm)
* [Isolation Filter Driver](https://www.easefilter.com/kb/Isolation_Filter_Driver.htm)

## License

This repository contains two distinct licensing categories:

1. **Wrapper/Implementation Code (MIT License):** 
   The source code, scripts, and configuration files provided directly in this repository are licensed under the [MIT License](LICENSE). You are free to modify, distribute, and use this specific integration code.

2. **EaseFilter SDK (Commercial License):**
   This project requires the EaseFilter SDK (including its binaries, filter drivers, and core libraries) to function. The EaseFilter SDK is **proprietary commercial software** and is strictly excluded from the MIT License.   
   To use the EaseFilter SDK for commercial purposes or in a production environment, you must obtain a valid commercial license directly from [EaseFilter](https://www.easefilter.com/license.pdf).

## Support
If you have questions or need help, please contact support@easefilter.com 

[Home](https://www.easefilter.com/) | [Solution](https://www.easefilter.com/solutions.htm) | [Download](https://www.easefilter.com/download.htm) | [Demos](https://www.easefilter.com/online-fileio-test.aspx) | [Blog](https://blog.easefilter.com/) | [Programming](https://www.easefilter.com/kb/programming.htm)
