🛡️ Windows CLI Basics – TryHackMe
Module 5 · Operating System Basics


📅 DATE
2 May 2026 · Day 20


📚 WHAT I LEARNED
Windows has two CLI environments — Command Prompt (cmd) for legacy commands and PowerShell for modern scripting

The Windows file system uses backslashes (\) and drive letters like C:\ to navigate directories
Many Windows CLI commands mirror Linux concepts but use different syntax and naming conventions

PowerShell uses cmdlets — structured commands like Get-Process — making it more powerful than cmd for administration

The CLI is essential for remote administration, scripting, and investigating Windows systems during incidents


🛠️ WHAT I DID

Completed the Windows CLI Basics room in full

Practised navigating the file system and managing files using both cmd and PowerShell

Ran commands to inspect system information, processes, and network configuration


💻 COMMANDS EXPLORED

Command	What it does
cd	Changes the current directory
dir	Lists files and folders in the current directory
mkdir	Creates a new directory
del	Deletes a file
copy	Copies a file to another location
move	Moves or renames a file
type	Displays the contents of a text file (like cat on Linux)
cls	Clears the terminal screen
ipconfig	Displays network configuration including IP address
ping	Tests connectivity to another host or IP address
tasklist	Lists all currently running processes
taskkill	Terminates a running process by name or PID
systeminfo	Displays detailed system and OS information
netstat	Shows active network connections and listening ports
help	Lists available commands or explains a specific command



🔐 WHY IT MATTERS

Attackers frequently use the Windows CLI to move through systems — defenders need to know the same tools to detect and respond

PowerShell is heavily used in both legitimate administration and malicious scripts — understanding it is critical for any Windows security role


❓ ONE THING I DIDN'T FULLY UNDERSTAND

Everything was clear
