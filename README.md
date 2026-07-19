# WPA/WPA2 Handshake Capture & Offline Cracking Audit

Project Type: Wireless Network Security Audit

Environment: Isolated Virtual Lab (Oracle VirtualBox + Kali Linux)

Objective: Demonstrate the vulnerability of weak WPA/WPA2 Pre-Shared Keys (PSK) through 4-way handshake capture and offline dictionary analysis.

# Table of Contents

Lab Environment Setup

Phase 1: Reconnaissance & Interface Preparation

Phase 2: Targeted Packet Capture

Phase 3: Handshake Interception (Deauthentication)

Phase 4: Protocol Analysis (Wireshark)

Phase 5: Offline Password Auditing

Conclusion & Key Takeaways

Disclaimer

# Lab Environment Setup
To ensure a safe and legal testing environment without risking interference with production networks, a localized virtual lab was constructed:

Hypervisor: Oracle VirtualBox
Guest OS: Kali Linux (Penetration Testing Distribution)
Access Point: A dedicated, isolated physical router.
Target SSID: Test-1
Target Password: 1234567890 (Intentionally weak password chosen to demonstrate the speed of dictionary attacks).

# Phase 1: Reconnaissance & Interface Preparation
Before capturing traffic, the wireless interface must be isolated from background processes and switched into a listening state.

1.1 Interface Verification
Scanned the system to identify the available wireless interfaces.

bash

```iwconfig```
Identified the primary wireless interface as wlan0.

1.2 Process Interference Mitigation
Killed conflicting background processes (such as NetworkManager) to prevent them from interrupting the wireless card during the attack.

bash

```sudo airmon-ng check kill```

1.3 Enabling Monitor Mode
Switched the wlan0 interface from Managed Mode to Monitor Mode, allowing it to listen to all wireless frames in the air, not just those directed to it.

bash

```sudo airmon-ng start wlan0```

Interface was successfully renamed to wlan0mon.

1.4 Verification
Confirmed that the interface was actively running in Monitor Mode.

bash

```sudo airmon-ng```


# Phase 2: Targeted Packet Capture
2.1 Network Reconnaissance
Scanned the surrounding airspace to identify access points, their channels, and connected clients.

bash

```sudo airodump-ng wlan0mon```

2.2 Target Identification
From the scan results, located the target router Test-1. Opened a text file (text.txt) to securely log the following target variables:

BSSID (MAC address of the router)
Channel
Station MAC (Connected client MAC addresses)
2.3 Locking onto the Target
Executed a targeted capture to save only the traffic specific to Test-1 on its specific channel. The captured data was written to a file prefixed with datafile.

bash

```sudo airodump-ng -w datafile -c [channel_number] --bssid [Test-1_BSSID] wlan0mon```

(Note: The terminal was left open to continuously listen for the handshake).


# Phase 3: Handshake Interception (Deauthentication)
To capture the WPA/WPA2 4-way handshake, a connected client must re-authenticate with the router. A deauthentication frame was sent to force this process.

3.1 Executing the Deauth Attack
Opened a new terminal and broadcasted deauth frames to the target router.

bash

```sudo aireplay-ng --deauth 0 -a [Test-1_BSSID] wlan0mon```

Command Breakdown:

--deauth 0: The 0 signifies a continuous, unlimited stream of deauthentication packets. (If replaced with a number like 10, it would send exactly 10 packets and stop).
-a [BSSID]: Specifies the target Access Point.
3.2 Capturing the Handshake
To expedite the process rather than waiting for a legitimate user to reconnect, a personal mobile device was connected to the Test-1 network.

Result: The airodump-ng terminal immediately displayed a WPA handshake: [BSSID] confirmation message at the top of the screen. The capture was stopped via Ctrl+C.


# Phase 4: Protocol Analysis (Wireshark)
Before attempting to crack the password, the captured file was analyzed to verify the integrity of the handshake.

bash

```wireshark datafile-01.cap```

Inside Wireshark, applied the display filter eapol to isolate the 4-way handshake frames (Messages 1-4), verifying that the ANonce, SNonce, and MICs were successfully captured.


# Phase 5: Offline Password Auditing
With the handshake securely captured in datafile-01.cap, an offline dictionary attack was initiated using the Aircrack-ng suite against the renowned rockyou.txt wordlist.

bash

```sudo aircrack-ng datafile-01.cap -w /usr/share/wordlists/rockyou.txt```

Results:

The tool began computing the PBKDF2-SHA1 hashes and comparing the resulting MICs against the captured handshake.
Time to Crack: Approximately 30 seconds.
Recovered Key: 1234567890
Note: At the time of recovery, the terminal indicated that the full rockyou.txt wordlist had roughly 1 hour and 48 minutes of passwords remaining to test, proving that weak passwords are found at the very beginning of such lists.


# Conclusion & Key Takeaways :
This project practically demonstrates why WPA/WPA2-PSK networks are vulnerable to offline attacks. The security of the network relies entirely on the entropy of the password. Because the password 1234567890 is highly common, it was cracked in 30 seconds.

Defensive Remediations :

Password Complexity: WPA2 passwords must be long (16+ characters), complex and avoid common dictionary words to survive offline brute-force attacks.
Upgrade to WPA3: WPA3 replaces the traditional 4-way handshake with SAE (Simultaneous Authentication of Equals), which is specifically designed to be immune to offline dictionary attacks, even if the password is weak.

⚠️ Disclaimer ⚠️
This documentation details a cybersecurity experiment performed in a strictly controlled, isolated lab environment using personally owned hardware. Executing deauthentication attacks or capturing handshakes on networks you do not own or have explicit authorization to test is illegal. This project was conducted solely for educational purposes and CV portfolio building.
