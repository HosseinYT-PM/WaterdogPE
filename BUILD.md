# BUILD.md – Building WaterdogPE Plugins

This document explains how to build your WaterdogPE plugin from source using Maven, both **online** (with internet access) and **offline** (without internet access), on **Windows** and **Linux** systems.

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [Online Build](#online-build)
  - [Windows](#windows-online)
  - [Linux / macOS](#linux-online)
- [Offline Build](#offline-build)
  - [Preparing the Local Repository](#preparing-the-local-repository)
  - [Building Offline](#building-offline)
    - [Windows](#windows-offline)
    - [Linux / macOS](#linux-offline)
- [Troubleshooting](#troubleshooting)
- [Next Steps](#next-steps)

---

## Prerequisites

Before you begin, ensure the following are installed on your system:

| Requirement | Version / Notes |
|-------------|-----------------|
| **Java JDK** | 17 or later (set `JAVA_HOME` and add `bin` to `PATH`) |
| **Maven**    | 3.6+ (set `MAVEN_HOME` and add `bin` to `PATH`) |
| **Git** (optional) | To clone the repository, or you can download the source as a ZIP |

To verify installations, open a terminal (Command Prompt / PowerShell on Windows, bash on Linux) and run:

```bash
java -version
mvn -version
```

Both commands should show version information without errors.

---

## Online Build

This is the standard method – Maven will automatically download all required dependencies from the WaterdogPE snapshot repository.

### Windows (Online)

**Using Command Prompt (cmd):**

1. Open **Command Prompt** and navigate to your plugin project root (where `pom.xml` is located):
   ```cmd
   cd C:\path\to\your\plugin
   ```

2. Run Maven’s `package` goal:
   ```cmd
   mvn clean package
   ```

**Using PowerShell:**

1. Open **PowerShell** and navigate to your project:
   ```powershell
   cd C:\path\to\your\plugin
   ```

2. Execute the build:
   ```powershell
   mvn clean package
   ```

Maven will download the WaterdogPE API and its transitive dependencies from `https://repo.waterdog.dev/snapshots`.  
Once complete, the plugin `.jar` file will be located in the `target/` directory, named after your `<finalName>` or `<artifactId>` (e.g., `MyPlugin.jar`).

### Linux / macOS (Online)

1. Open a terminal and change to your project directory:
   ```bash
   cd /path/to/your/plugin
   ```

2. Build the plugin:
   ```bash
   mvn clean package
   ```

If you haven’t installed Maven globally, you can use the Maven wrapper (`mvnw`) if present in your project.

The resulting JAR will be in `target/`.

---

## Offline Build

Offline builds are useful when you are in an environment without internet access, or when you want to avoid repeated downloads. Maven can still build the project if all dependencies are already present in your local repository (`~/.m2/repository`).

### Preparing the Local Repository

Before you can build offline, you must have all required dependencies cached locally **at least once** while online.  

1. Perform an online build (as described above) on a machine with internet access. This will download everything into `~/.m2/repository`.
2. Copy the entire `~/.m2/repository` folder to the offline machine, placing it in the same location (`~/.m2/repository`) – or you can set a different local repository path using the `-Dmaven.repo.local` parameter.

**Alternative:** Manually download the WaterdogPE API JAR and POM from the snapshot repository and install them into your local repository using:
```bash
mvn install:install-file -Dfile=waterdog-2.0.4-SNAPSHOT.jar -DpomFile=waterdog-2.0.4-SNAPSHOT.pom
```
(You would need to download both `.pom` and `.jar` from https://repo.waterdog.dev/snapshots/dev/waterdog/waterdogpe/waterdog/2.0.4-SNAPSHOT/)

However, it's easier to just do one online build first.

### Building Offline

Once the local repository is populated, you can build offline using Maven's `--offline` flag.

#### Windows (Offline)

**Command Prompt:**
```cmd
cd C:\path\to\your\plugin
mvn clean package --offline
```

**PowerShell:**
```powershell
cd C:\path\to\your\plugin
mvn clean package --offline
```

#### Linux / macOS (Offline)
```bash
cd /path/to/your/plugin
mvn clean package --offline
```

If you need to use a custom local repository location, add:
```bash
mvn clean package --offline -Dmaven.repo.local=/path/to/your/repo
```

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| **"Cannot resolve dependencies"** | You are missing some dependencies in your local repository. Perform an online build first to download them, or check your internet connection. |
| **"JAVA_HOME is not set"** | On Windows, set `JAVA_HOME` to your JDK installation path and add `%JAVA_HOME%\bin` to `PATH`. On Linux, set `JAVA_HOME` in `~/.bashrc` or `~/.profile`. |
| **"Maven is not recognized"** | Ensure Maven is installed and its `bin` folder is added to your system `PATH`. |
| **"Offline build fails with missing plugin"** | Some Maven plugins (like `maven-compiler-plugin`) are also downloaded online. Run Maven online once to cache all plugins. |
| **Build produces no JAR** | Check that `pom.xml` has `<packaging>jar</packaging>` and that the `maven-compiler-plugin` and `maven-shade-plugin` (if used) are correctly configured. |
| **WaterdogPE version mismatch** | Ensure the version in your `pom.xml` matches one available in the repository. Update to the latest snapshot if needed. |

---

## Next Steps

After a successful build, your plugin JAR will be in the `target/` folder. Copy it to the `plugins/` directory of your WaterdogPE proxy installation and restart/reload the proxy.

For more details on plugin development, refer to the main [README.md](README.md).

---


