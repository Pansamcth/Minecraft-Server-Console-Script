# 📌 Minecraft Server Console Script

Scripts for managing **Minecraft Java Edition Servers** on both **Windows (.bat)** and **Linux (.sh)**.  
These scripts handle server startup, memory allocation, JVM optimizations, EULA acceptance, and automatic restarts in case of crashes or shutdowns.

---

## 🚀 Features

- ✅ **Automatic Java JDK 21 Setup (`JAVA_HOME`)**  
  Ensures the server always runs with **Java JDK 21**, providing improved performance, stability, and long-term compatibility.  
  - The script sets the `JAVA_HOME` variable and updates the `PATH`.  
  - Guarantees the correct Java version is used even if multiple versions are installed.  

- ✅ **Configurable Memory Heap (`Xms` / `Xmx`)**  
  - Allows you to define the **minimum (`-Xms`)** and **maximum (`-Xmx`)** heap size.  
  - Example: `Xms=4096M` / `Xmx=4096M` allocates 4 GB of RAM to the server.  
  - Prevents dynamic resizing overhead and stabilizes performance under load.  

- ✅ **Optimized JVM Flags (Aikar’s Flags)**  
  - Pre-configured for garbage collection (GC) optimization.  
  - Includes flags such as:  
    - `-XX:+UseG1GC` → G1 Garbage Collector, designed for low-latency large heaps  
    - `-XX:+AlwaysPreTouch` → Pre-allocates memory pages at startup  
    - `-XX:InitiatingHeapOccupancyPercent=15` → Starts GC early to reduce pause times  
    - `-XX:MaxGCPauseMillis=200` → Targets GC pauses under 200ms  
  - Reference: [Aikar’s Flags Guide](https://mcflags.emc.gs)  

- ✅ **Automatic EULA Acceptance**  
  - Updates `eula.txt` automatically (`eula=false` → `eula=true`).  
  - Ensures the server boots without manual intervention.  

- ✅ **Crash & Restart Handling**  
  - Runs inside a loop.  
  - If the server stops or crashes, the script waits 5 seconds and restarts it.  
  - Provides maximum uptime with zero manual restarts.  

---

## 🪟 Windows Script (start.bat)
```ruby
@title Server Console
@echo off
echo ------------------------------------------------------
echo                (%time%) Server is starting!         
echo ------------------------------------------------------

set JAVA_HOME=C:\Program Files\Java\jdk-21
set PATH=%JAVA_HOME%\bin;%PATH%

set Server_file=***Your Jar file***
set -Xms=4096M   # Initial Memory Allocation Pool
set -Xmx=4096M   # Maximum Memory Allocation Pool
# 1G = 1024M, 2G = 2048M, 4G = 4096M, 8G = 8192M, 16G = 16384M, 32G = 32768M
set Java_optioms=-XX:+AlwaysPreTouch -XX:+DisableExplicitGC -XX:+ParallelRefProcEnabled -XX:+PerfDisableSharedMem -XX:+UnlockExperimentalVMOptions -XX:+UseG1GC -XX:G1HeapRegionSize=8M -XX:G1HeapWastePercent=5 -XX:G1MaxNewSizePercent=40 -XX:G1MixedGCCountTarget=4 -XX:G1MixedGCLiveThresholdPercent=90 -XX:G1NewSizePercent=30 -XX:G1RSetUpdatingPauseTimePercent=5 -XX:G1ReservePercent=20 -XX:InitiatingHeapOccupancyPercent=15 -XX:MaxGCPauseMillis=200 -XX:MaxTenuringThreshold=1 -XX:SurvivorRatio=32 -Dusing.aikars.flags=https://mcflags.emc.gs -Daikars.new.flags=true

:StartServer
echo Starting the Minecraft server with the following options:
echo - Minimum Heap Size: %-Xms%
echo - Maximum Heap Size: %-Xmx%
echo ------------------------------------------------------
java -Xms%-Xms% -Xmx%-Xmx% %Java_optioms% -jar "%Server_file%" --nogui

cd C:\path\to\directory
powershell -Command "(Get-Content eula.txt) -replace 'eula=false', 'eula=true' | Set-Content eula.txt"

echo ------------------------------------------------------
echo                (%time%) Server restarting!          
echo ------------------------------------------------------
timeout 5
goto StartServer
```
> [!IMPORTANT]  
> Make sure you **update the `Server_file` variable** in the script to point to your actual JAR file.  
---
## 🐧 Linux Script (start.sh)
```ruby
#!/bin/bash
# Minecraft Server Launcher - Linux Edition
# chmod +x start.sh

# ==============================
# CONFIG
# ==============================
JAVA_HOME="/usr/lib/jvm/java-21-openjdk"    # Change to your Java path
PATH="$JAVA_HOME/bin:$PATH"     # Add Java to PATH

SERVER_FILE="***Your Jar file***"    # Change to your server .jar file
XMS="4096M"     # Initial Memory Allocation Pool
XMX="4096M"     # Maximum Memory Allocation Pool
# 1G = 1024M, 2G = 2048M, 4G = 4096M, 8G = 8192M, 16G = 16384M, 32G = 32768M

# Aikar’s Flags (optimized for MC server)
JAVA_FLAGS="-XX:+UseG1GC -XX:+ParallelRefProcEnabled -XX:MaxGCPauseMillis=200 -XX:+UnlockExperimentalVMOptions -XX:+DisableExplicitGC -XX:+AlwaysPreTouch -XX:G1NewSizePercent=30 -XX:G1MaxNewSizePercent=40 -XX:G1HeapRegionSize=8M -XX:G1ReservePercent=20 -XX:G1HeapWastePercent=5 -XX:G1MixedGCCountTarget=4 -XX:G1MixedGCLiveThresholdPercent=90 -XX:G1RSetUpdatingPauseTimePercent=5 -XX:InitiatingHeapOccupancyPercent=15 -XX:SurvivorRatio=32 -XX:MaxTenuringThreshold=1 -Daikars.new.flags=true"

# ==============================
# FUNCTIONS
# ==============================

banner() {
  echo -e "\e[36m"
  echo "======================================================"
  echo "   🚀 Minecraft Server Console | $(date +"%T")"
  echo "======================================================"
  echo -e "\e[0m"
}

start_server() {
  banner
  echo -e "\e[33mStarting server with:\e[0m"
  echo "- Min Heap: $XMS"
  echo "- Max Heap: $XMX"
  echo "------------------------------------------------------"
  java -Xms$XMS -Xmx$XMX $JAVA_FLAGS -jar "$SERVER_FILE" --nogui    # Start the server
}

accept_eula() {
  if [ -f "eula.txt" ]; then
    sed -i 's/eula=false/eula=true/g' eula.txt
  fi
}

# ==============================
# MAIN LOOP
# ==============================
while true
do
  start_server
  accept_eula

  echo -e "\e[31m------------------------------------------------------"
  echo "💀 Server crashed/stopped - restarting in 5s..."
  echo "------------------------------------------------------"
  sleep 5
done
```
> [!IMPORTANT]  
> Make sure you **update the `Server_file` variable** in the script to point to your actual JAR file.  
---

> [!WARNING]
> This script requires Java [JDK 21](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html) to function. The JAVA_HOME variable is set to the JDK 21 directory path, and this path is added to the PATH environment variable. This setup ensures that the Minecraft server uses JDK 21 to run, along with specific Java options configured in the script to enhance performance.

> [!TIP]
> ## Here are the server types that can be used in Minecraft (Java Edition):
> - **[Vanilla (Minecraft.net)](https://www.minecraft.net/en-us/download/server)** - Supports Datapacks from Mojang.
> - **[Spigot](https://getbukkit.org/download/spigot)** - Supports Plugins and is suitable for large servers.
> - **[Paper](https://papermc.io/downloads/paper)** - A fork of Spigot that enhances performance and supports Spigot plugins.
> - **[Purpur](https://purpurmc.org/downloads)** - A fork of Paper that is more flexible and customizable.
> - **[Forge](https://files.minecraftforge.net/net/minecraftforge/forge/)** - Designed for Mods, suitable for adding new content to the game.
> - **[Fabric](https://fabricmc.net/use/server/)** - A modding platform that emphasizes speed and flexibility.
