## Project Overview

This project conducts a comprehensive and operational security audit (Hardening Audit) on a Linux operating system (Kali Linux) to identify vulnerabilities, misconfigurations, and potential threats. The primary goal of this project is to enhance the system's security posture and provide recommendations for its hardening. Various tools, including Lynis and chkrootkit, were utilized in this audit.

## Tools Used

* **Kali Linux**: The operating system used for performing the audit and hosting security tools.
* **Lynis**: A powerful and flexible security auditing tool for Unix and Linux-based systems. Lynis performs deep scans to find security issues, misconfigurations, sensitive information, and more.
* **chkrootkit**: A script used to check for the presence of known rootkits on the system. This tool examines system files and looks for signs of infection.
* **Log Analysis**: Reviewing and analyzing system log files (such as `auth.log`) to identify failed login attempts and suspicious activities.

## Audit Findings

This section provides a summary of the significant findings obtained from running the auditing tools and analyzing the logs:

### Lynis Results (Excerpt from `audit_results.txt`)

The Lynis tool provided comprehensive reports on the system's security status. Some key highlights include:

* **Installed Packages**:
    * `inetd` package not found.
    * `xinetd` package is installed, but its status is `NOT ACTIVE`.
    * `rsh server` package is installed. (This could be a security risk as `rsh` is an insecure protocol).
    * `telnet server` package not found.
    * `NIS client` and `NIS server` are installed.
    * `TFTP client` and `TFTP server` are listed as `SUGGESTION`, indicating a need for further review or installation.

* **Banners and identification**:
    * `/etc/issue` file found.
    * `/etc/issue` contents marked as `WEAK`, suggesting that unnecessary or sensitive information might be displayed in the system's login banner.

* **System Information**:
    * Lynis Version: 3.1.4
    * Operating System: Kali Linux Rolling release
    * Kernel Version: 6.12.25
    * Hardware Platform: x86_64

### chkrootkit Results (Excerpt from `chkrootkit_results.txt`)

chkrootkit was executed to check for rootkits. The main results include:

* **System Tool Checks**: Many core tools such as `basename`, `chfn`, `chsh`, `cron`, `crontab`, `date`, `du`, `dirname`, `echo`, `egrep`, `env`, `find`, `grep`, `hdparm`, `su`, `ifconfig` were reported as `not infected`.
* **Sniffer Warning**:
    ```
    Checking `sniffer'...                                       WARNING

    WARNING: Output from ifpromisc:
    lo: not promisc and no packet sniffer sockets
    eth0: PACKET SNIFFER(/usr/sbin/NetworkManager[611])
    ```
    This warning indicates that `NetworkManager` is in `PACKET SNIFFER` mode on the `eth0` interface. While this might be normal, it requires investigation to ensure it's not a malicious process.

* **wted Warning**:
    ```
    Checking `wted'...                                          WARNING

    WARNING: output from chkwtmp:
    1 deletion(s) between Tue May  6 18:02:26 2025 and Wed May 14 04:48:32 2025
    1 deletion(s) between Sat May 17 06:30:56 2025 and Sat May 17 07:09:38 2025
    ```
    This warning indicates one or more deletions from the `wtmp` file (which logs user logins and logouts). This could be a sign of attempts to hide unauthorized activities and requires thorough investigation.

### Failed Login Attempts Analysis (Excerpt from `failed_logins.txt`)

An examination of `auth.log` shows multiple unsuccessful login attempts, primarily using the username `fakeuser`.

* **Failed Login Attempts for `dawn`**:
    * On May 18, 2025, several unsuccessful login attempts for user `dawn` from IP address `10.0.2.2` and ports `52904` and `52972` (via SSH) were recorded.

* **Failed Login Attempts for `invalid user fakeuser`**:
    * On June 9, 2025, repeated unsuccessful login attempts for an invalid user (`fakeuser`) from `::1` (localhost) and various ports (e.g., `45324`, `43856`) via SSH were recorded. This could indicate local brute-force attacks or penetration tests.

## Security Status Summary and Recommendations

Based on the audit results, the audited system has several weaknesses and security warnings that require attention:

* **Insecure Protocols**: The installation of `rsh server` is a serious concern. The RSH protocol is insecure and should be disabled or removed.
* **`xinetd` Configuration**: Although `xinetd` is installed, its `NOT ACTIVE` status should be verified. If services under `xinetd` are not needed, this status is good; otherwise, it should be activated and properly configured.
* **Login Banners**: The contents of `/etc/issue` should be reviewed, and any unnecessary information or software versions that could be exploited by attackers should be removed.
* **`chkrootkit` Warnings**:
    * The `PACKET SNIFFER` warning needs to be carefully investigated to ensure `NetworkManager` is legitimately performing this function and not a malicious process.
    * The deletion of entries from the `wtmp` file is a serious warning and should be investigated immediately. This could be a sign of attempts to hide unauthorized activities. Reviewing relevant log files (such as `/var/log/wtmp`) and enabling the Auditing System are recommended.
* **Failed Login Attempts**: The high number of unsuccessful login attempts, especially via SSH, indicates brute-force attacks or attempts at unauthorized access.
    * Implement account lockout policies after a specific number of failed attempts.
    * Use tools like `fail2ban` to block suspicious IP addresses.
    * Employ stronger authentication (e.g., SSH keys instead of passwords).
    * Change the default SSH port.

### General Hardening Recommendations:

* **Regular Updates**: Ensure the operating system, kernel, and all installed packages are regularly updated.
* **Remove Unnecessary Services**: Disable and remove any services or software packages that are not required.
* **Firewall Configuration**: Enable and properly configure a firewall (such as `ufw` or `iptables`) to restrict access.
* **User and Password Management**: Enforce strong password policies, lock unused accounts, and remove unnecessary users.
* **Log Monitoring**: Utilize centralized log collection tools and SIEM systems for continuous log monitoring and quick identification of threats.
* **Auditing System**: Enable and configure a system auditing tool (e.g., `auditd`) to record critical system activities.

---

