# Challenge Answers

## Basic Static Analysis
---

Q: What is the SHA256 hash of the sample?

A: 0C82E654C09C8FD9FDF4899718EFA37670974C9EEC5A8FC18A167F93CEA6EE83

---

Q: What architecture is this binary?

A: This is a 32-bit binary, as identified by PEView and/or PEStudio (among other tools).

---

Q: Are there any results from submitting the SHA256 hash to VirusTotal??

A: This can vary. Depending on the time that you are performing this challenge, there may be results.

---

Q: Describe the results of pulling the strings from this binary. Record and describe any strings that are potentially interesting. Can any interesting information be extracted from the strings?

A: The strings section of this challenge is more difficult than usual, because this malware sample appears to be a normal working program! The normal strings associated with PuTTY are present in the binary. Inspecting some of the strings that appear to be URLs reveals nothing of note as these URLs are standard to the normal PuTTY executable. Strings seems to be a dead end for this binary.

---

Q: Describe the results of inspecting the IAT for this binary. Are there any imports worth noting?

A: The same problem with pulling the strings from this binary is present when inspecting the IAT in PEView or PEStudio. There are imports present that deal with the Windows Registry that may be notable, but PuTTY's normal functions can also manipulate the registry. The IAT has plenty of imports to look at, but there is not enough information to make a determination yet.

---

Q: Is it likely that this binary is packed?

A: No, this binary is unlikely to be packed. There are no header sections that indicate a packing/unpacking stub and the Size of the Raw Data and Virtual Size of the headers are close values.

---
## Basic Dynamic Analysis


Q: Describe initial detonation. Are there any notable occurances at first detonation? Without internet simulation? With internet simulation?

A: Executing the program spawns PuTTY, which appears to be the normal program. If you look closely, it also spawns a blue window for a brief moment, which is in line with the scenario brief in the README.

---

Q: From the host-based indicators perspective, what is the main payload that is initiated at detonation? What tool can you use to identify this?

A: The blue window that appears momentarily is a powershell.exe window. Either by using that as a pivot point and filtering on "Process name contains powershell" or by examining the child processes that are spawned from putty.exe, you can find a child `powershell.exe` process spawned at detonation with putty.exe as its parent. When examining the powershell.exe process in Procmon, the arguments indicate that Powershell is executing a Base64 encoded and compressed string at detonation.

`Bonus:` If you base64 decode that string and then extract it using 7zip or the unzip utility on REMNux, the resulting stream can be written to an outfile. There, you can see the full text of the powershell reverse shell that is calling out to `bonus2.corporatebonusapplication.local`.

---

Q: What is the DNS record that is queried at detonation?

A: The DNS record is `bonus2.corporatebonusapplication.local`. This can be found in Wireshark by filtering for DNS records at detonation.

---

Q: What is the callback port number at detonation?

A: The port is 8443.

---

Q: What is the callback protocol at detonation?

A: The protocol is HTTPS/TLS. This can be identified in Wireshark by the initiation of a CLIENT HELLO message from the detonation to the specified domain.

---

Q: How can you use host-based telemetry to identify the DNS record, port, and protocol?

A: This can be accomplished by filtering on the name of the binary and adding an additional filter of "Operation contains TCP" in procmon.

---

Q: Attempt to get the binary to initiate a shell on the localhost. Does a shell spawn? What is needed for a shell to spawn?

The shell does not spawn without a proper TLS handshake, so using a basic ncat listener on port 443 does not initiate a shell. The syntax of the PowerShell reverse shell requires TLS to complete the network transaction, so even if you use the `hosts` file and open up a listener on port 443 to catch the incoming shell, you cannot coerce the binary to connect unless you can also provide a valid SSL certificate.

`Bonus:` the module used to spawn this reverse shell is available in Metasploit. Try to figure out which module is in use, bring a Kali machine into the lab, and catch the incoming shell!