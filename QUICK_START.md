# Quick Start Guide
## Get Ready for the Phoebus Developers Meeting and Codeathon

This guide will help you prepare for the Phoebus Tools and Services session at the 2026 EPICS Developers Meeting and Codeathon.

---

## Before You Arrive

### 1. Set Up Your Development Environment

#### Install Java Development Kit (JDK)
- **Required:** JDK 21 or later (JDK 25 recommended for current Phoebus upgrade work)
- **Download:** https://adoptium.net/ (Eclipse Temurin)
- **Verify installation:**
  ```bash
  java -version
  javac -version
  ```

#### Install Maven
- **Required:** Maven 3.6 or later
- **Download:** https://maven.apache.org/download.cgi
- **Verify installation:**
  ```bash
  mvn -version
  ```

#### Install Git
- **Required:** Latest stable version
- **Download:** https://git-scm.com/downloads
- **Configure:**
  ```bash
  git config --global user.name "Your Name"
  git config --global user.email "your.email@example.com"
  ```

#### Choose Your IDE
Pick one:
- **Eclipse:** https://www.eclipse.org/downloads/
- **IntelliJ IDEA:** https://www.jetbrains.com/idea/download/
- **VS Code:** https://code.visualstudio.com/ (with Java extensions)

#### Install Docker (Optional but Recommended)
- **For services development and testing**
- **Download:** https://www.docker.com/products/docker-desktop

---

### 2. Create GitHub Account
- Sign up at https://github.com if you don't have an account
- Set up SSH keys or personal access token for authentication
- Fork repositories you'll be working on (optional, can do on-site)

---

### 3. Clone Repositories (Optional Pre-work)

You can clone these ahead of time to save bandwidth at the venue:

```bash
# Create a workspace directory
mkdir -p ~/epics-codeathon-2026
cd ~/epics-codeathon-2026

# Clone Phoebus
git clone https://github.com/ControlSystemStudio/phoebus.git

# Clone Phoebus-Olog
git clone https://github.com/Olog/phoebus-olog.git

# Clone ChannelFinder Service
git clone https://github.com/ChannelFinder/ChannelFinderService.git

# Clone EPICS Archiver Appliance
git clone https://github.com/shroffk/epicsarchiverap.git

# Clone EPICS Core Java
git clone https://github.com/shroffk/epicsCoreJava.git
```

---

### 4. Build Phoebus (Optional Pre-work)

Test that you can build Phoebus:

```bash
cd phoebus
mvn clean verify -DskipTests
```

This may take 10-20 minutes on first run. If successful, you'll see:
```
[INFO] BUILD SUCCESS
```

---

### 5. Review Project Lists

Browse the project lists to get familiar with what's available:
- `projects/README.md` - Development projects
- `documentation-tasks/README.md` - Documentation tasks

Think about which areas interest you most.

---

### 6. Join Communication Channels

- **Matrix Chat:** Join [#codeathon-oct26:epics-controls.org](https://matrix.to/#/#codeathon-oct26:epics-controls.org)
- **Install Matrix client:** Element (https://element.io/) or use web interface
- Introduce yourself in the chat before the event!

---

### 7. Familiarize Yourself with EPICS

If you're new to EPICS:
- Visit https://epics-controls.org/
- Read the overview documentation
- Understand basic concepts: IOC, PV, Channel Access, PV Access

If you're new to Phoebus:
- Watch introduction videos if available
- Read user documentation at https://github.com/ControlSystemStudio/phoebus
- Download and run a pre-built version

---

### 8. Prepare Your Laptop

- **Ensure your laptop is charged and you have a charger**
- **US Power:** Bring a US plug adapter if needed (NEMA 5-15, 120V, 60Hz)
- **WiFi:** Check event communications for Brookhaven guest Wi-Fi details before arrival
- **Backup:** Commit any local work before traveling
- **Free up disk space:** Need at least 10 GB free
- **Update your OS and software**
