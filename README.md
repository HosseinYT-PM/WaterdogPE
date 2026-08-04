# WaterdogPE Plugin Development Guide

This repository serves as a **template and comprehensive guide** for building plugins for **WaterdogPE** – a modern Minecraft: Bedrock Edition proxy server written in Java.  
Use this README to understand the plugin architecture, set up your development environment, and create your own plugins with ease.

---

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [Maven Configuration](#maven-configuration)
  - [Repository](#repository)
  - [Dependency](#dependency)
  - [Complete `pom.xml`](#complete-pomxml)
- [The Main Class](#the-main-class)
  - [Lifecycle Methods](#lifecycle-methods)
  - [Useful Inherited Methods](#useful-inherited-methods)
- [The `plugin.yml` Descriptor](#the-pluginyml-descriptor)
- [Working with the ProxyServer](#working-with-the-proxyserver)
  - [Common Operations](#common-operations)
  - [Manager Cheat Sheet](#manager-cheat-sheet)
- [Creating Commands](#creating-commands)
  - [Command Example](#command-example)
  - [CommandSettings Builder](#commandsettings-builder)
  - [Registering a Command](#registering-a-command)
- [Handling Events](#handling-events)
  - [Subscribing to an Event](#subscribing-to-an-event)
  - [Event Priority](#event-priority)
  - [Cancelling Events](#cancelling-events)
  - [Async Events](#async-events)
  - [Common Events](#common-events)
- [Scheduling Tasks](#scheduling-tasks)
- [Building and Deploying](#building-and-deploying)
- [Example Plugins](#example-plugins)
- [Resources & Further Reading](#resources--further-reading)
- [License](#license)

---

## Overview

WaterdogPE is a high‑performance proxy that allows you to manage multiple Bedrock servers behind a single IP address. Its plugin API gives you full control over:

- Player connections and authentication
- Server transfers and fallback logic
- Chat interception and formatting
- Custom commands and event handling
- Scheduling background tasks

This guide covers everything you need to get started with WaterdogPE plugin development.

---

## Prerequisites

Before you begin, ensure you have the following installed:

| Requirement | Version / Notes |
|-------------|-----------------|
| **Java JDK** | 17 or newer (WaterdogPE runs on Java 17) |
| **Maven**   | 3.6+ (or use the Maven wrapper) |
| **IDE**     | IntelliJ IDEA, Eclipse, or any IDE with Maven support |
| **Internet**| Maven will download dependencies from the WaterdogPE snapshot repository |

---

## Project Structure

A WaterdogPE plugin is a `.jar` file with a well-defined structure:

```
my-plugin/
├── pom.xml                          # Maven build file
└── src/
    └── main/
        ├── java/
        │   └── com/example/myplugin/
        │       └── MyPlugin.java    # Main class (extends Plugin)
        └── resources/
            └── plugin.yml            # Plugin descriptor
```

**Important:** The main class must be placed in the correct package and must extend `dev.waterdog.waterdogpe.plugin.Plugin`.

---

## Maven Configuration

### Repository

Add the WaterdogPE snapshot repository to your `pom.xml` so Maven can resolve the WaterdogPE API:

```xml
<repositories>
    <repository>
        <id>waterdog-snapshots</id>
        <url>https://repo.waterdog.dev/snapshots</url>
    </repository>
</repositories>
```

### Dependency

Declare the WaterdogPE API as a **provided** dependency (the proxy will supply it at runtime):

```xml
<dependencies>
    <dependency>
        <groupId>dev.waterdog.waterdogpe</groupId>
        <artifactId>waterdog</artifactId>
        <version>2.0.4-SNAPSHOT</version>
        <scope>provided</scope>
    </dependency>
</dependencies>
```

> **Note:** The exact version may change over time. Check the [repository](https://repo.waterdog.dev/snapshots/dev/waterdog/waterdogpe/waterdog/) for the latest snapshot.

### Complete `pom.xml`

Below is a fully configured `pom.xml` that you can copy and adapt:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>MyPlugin</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <repositories>
        <repository>
            <id>waterdog-snapshots</id>
            <url>https://repo.waterdog.dev/snapshots</url>
        </repository>
    </repositories>

    <dependencies>
        <dependency>
            <groupId>dev.waterdog.waterdogpe</groupId>
            <artifactId>waterdog</artifactId>
            <version>2.0.4-SNAPSHOT</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>

    <build>
        <finalName>${project.artifactId}</finalName>
        <plugins>
            <!-- Compiler plugin -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.11.0</version>
                <configuration>
                    <source>17</source>
                    <target>17</target>
                </configuration>
            </plugin>
            <!-- Shade plugin (optional, but useful if you have dependencies) -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.5.0</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

---

## The Main Class

Every plugin must have a main class that extends `dev.waterdog.waterdogpe.plugin.Plugin`.  
This class is the entry point for your plugin's lifecycle.

```java
package com.example.myplugin;

import dev.waterdog.waterdogpe.plugin.Plugin;

public class MyPlugin extends Plugin {

    @Override
    public void onEnable() {
        // Called when the proxy enables your plugin.
        // Register commands, subscribe to events, start tasks, etc.
        this.getLogger().info("MyPlugin has been enabled!");
    }

    @Override
    public void onDisable() {
        // Called on proxy shutdown or when the plugin is disabled.
        // Clean up: close connections, save data, cancel tasks.
        this.getLogger().info("MyPlugin has been disabled!");
    }
}
```

### Lifecycle Methods

| Method       | When It Runs                                          | Typical Use                                      |
|--------------|-------------------------------------------------------|--------------------------------------------------|
| `onStartup()`| After loading, before enabling (early in proxy start) | Load critical data; most plugins don't need this |
| `onEnable()` | **Required** – when proxy startup is complete         | Register commands/events, schedule tasks, read config |
| `onDisable()`| On proxy shutdown or plugin disable                   | Clean up resources, save data                    |

> **Important:** Always use `onEnable()` for setup – the constructor runs before the plugin is fully initialised, so `getProxy()` and `getLogger()` are not available yet.

### Useful Inherited Methods

| Method               | Returns                | Description                                         |
|----------------------|------------------------|-----------------------------------------------------|
| `getProxy()`         | `ProxyServer`          | Gateway to players, servers, scheduler, events, etc.|
| `getLogger()`        | `Logger`               | Logger prefixed with your plugin name               |
| `getDataFolder()`    | `File`                 | Plugin's data folder (`plugins/<name>/`)            |
| `getConfig()`        | `Configuration`        | Loads `config.yml` from data folder                 |
| `getName()`          | `String`               | Plugin name from `plugin.yml`                       |

---

## The `plugin.yml` Descriptor

Create `src/main/resources/plugin.yml` with the following content:

```yaml
name: MyPlugin
version: 1.0.0
author: YourName
main: com.example.myplugin.MyPlugin
```

### Required Fields

| Field   | Required | Description                                              |
|---------|----------|----------------------------------------------------------|
| `name`  | Yes      | Plugin name (used for data folder and logger prefix)     |
| `main`  | Yes      | Fully‑qualified main class name (package + class)        |
| `version`| Recommended | Version string                                        |
| `author`| Optional | Author name                                              |
| `depends`| Optional | List of plugin names that must load before yours         |

> WaterdogPE also accepts `waterdog.yml`, but `plugin.yml` is the conventional choice.

---

## Working with the ProxyServer

The `ProxyServer` is a singleton – your gateway to almost everything.

```java
ProxyServer proxy = this.getProxy();
// or from anywhere:
ProxyServer proxy = ProxyServer.getInstance();
```

### Common Operations

```java
ProxyServer proxy = this.getProxy();

// Players
ProxiedPlayer player = proxy.getPlayer("Steve");      // by name
ProxiedPlayer byId = proxy.getPlayer(uuid);            // by UUID
Map<String, ProxiedPlayer> online = proxy.getPlayers(); // all online

// Downstream Servers
ServerInfo lobby = proxy.getServerInfo("lobby1");
Collection<ServerInfo> servers = proxy.getServers();

// Core Managers
proxy.getCommandMap();      // Register/unregister commands
proxy.getEventManager();    // Subscribe to and call events
proxy.getScheduler();       // Run delayed/repeating tasks
proxy.getPluginManager();   // Look up other plugins
proxy.getConfiguration();   // Read config.yml values

// Logging
proxy.getLogger().info("Hello from the proxy!");
```

### Manager Cheat Sheet

| Manager            | Get It With               | Use It For                                      |
|--------------------|---------------------------|-------------------------------------------------|
| Command Map        | `proxy.getCommandMap()`   | Registering commands                            |
| Event Manager      | `proxy.getEventManager()` | Reacting to joins, chat, transfers, pings      |
| Scheduler          | `proxy.getScheduler()`    | Delayed, repeating, and async tasks            |
| Plugin Manager     | `proxy.getPluginManager()`| Looking up other loaded plugins                 |
| Configuration      | `proxy.getConfiguration()`| Reading the proxy's `config.yml`               |

---

## Creating Commands

Commands run on the proxy, not on the downstream server – perfect for `/server`, `/hub`, or `/staffchat`.

### Command Example

```java
package com.example.myplugin;

import dev.waterdog.waterdogpe.command.Command;
import dev.waterdog.waterdogpe.command.CommandSender;
import dev.waterdog.waterdogpe.command.CommandSettings;
import dev.waterdog.waterdogpe.player.ProxiedPlayer;

public class HubCommand extends Command {

    public HubCommand() {
        super("hub", CommandSettings.builder()
                .setDescription("Teleport to the hub")
                .setPermission("myplugin.command.hub")
                .setAliases("lobby", "leave")
                .build());
    }

    @Override
    public boolean onExecute(CommandSender sender, String alias, String[] args) {
        if (!sender.isPlayer()) {
            sender.sendMessage("§cOnly players can use this command.");
            return true;
        }

        ProxiedPlayer player = (ProxiedPlayer) sender;
        ServerInfo hub = player.getProxy().getServerInfo("lobby1");

        if (hub == null) {
            player.sendMessage("§cThe hub is not available right now.");
            return true;
        }

        player.connect(hub);
        return true;
    }
}
```

### CommandSettings Builder

| Method                  | Purpose                                          |
|-------------------------|--------------------------------------------------|
| `setDescription(String)`| Human‑readable description (shown in `/help`)    |
| `setUsageMessage(String)`| Usage hint shown when `onExecute` returns `false`|
| `setPermission(String)` | Permission the sender must have                  |
| `setPermissionMessage(String)`| Message shown when permission check fails |
| `setAliases(String...)` | Alternative command names                        |

### Registering a Command

```java
@Override
public void onEnable() {
    this.getProxy().getCommandMap().registerCommand(new HubCommand());
}
```

**Note:** `onExecute` return value:
- Return `true` if you handled the command (even with your own error message)
- Return `false` to show the command's usage message

---

## Handling Events

Events let your plugin react to proxy activity – player logins, chat messages, transfers, pings, etc.

### Subscribing to an Event

```java
import dev.waterdog.waterdogpe.event.defaults.PlayerChatEvent;
import dev.waterdog.waterdogpe.player.ProxiedPlayer;

@Override
public void onEnable() {
    this.getProxy().getEventManager()
        .subscribe(PlayerChatEvent.class, this::onChat);
}

private void onChat(PlayerChatEvent event) {
    ProxiedPlayer player = event.getPlayer();
    if (event.getMessage().contains("badword")) {
        event.setCancelled(true);
        player.sendMessage("§cWatch your language!");
    }
}
```

### Event Priority

Control handler order with `EventPriority`:

```java
import dev.waterdog.waterdogpe.event.EventPriority;

this.getProxy().getEventManager()
    .subscribe(PlayerChatEvent.class, this::onChat, EventPriority.HIGHEST);
```

Priorities (first to last): `LOWEST` → `LOW` → `NORMAL` (default) → `HIGH` → `HIGHEST`

### Cancelling Events

```java
event.setCancelled(true);
boolean cancelled = event.isCancelled();
```

> Only cancel events marked as cancellable in the event reference – calling `setCancelled` on a non‑cancellable event throws an exception.

### Async Events

Events annotated with `@AsyncEvent` run on a background thread pool:

```java
this.getProxy().getEventManager().callEvent(event)
    .whenComplete((completedEvent, error) -> {
        // Runs once every handler has finished
    });
```

### Common Events

| Event                      | Cancellable | Fires When                                               |
|----------------------------|-------------|----------------------------------------------------------|
| `ProxyStartEvent`          | –           | Proxy has started and bound its listener                 |
| `PreClientDataSetEvent`    | –           | Player login data is decoded (before player object exists)|
| `PlayerAuthenticatedEvent` | ✓           | `ProxiedPlayer` is created from login packet             |
| `PlayerLoginEvent`         | ✓           | Player is about to log in                                |
| `PlayerChatEvent`          | ✓           | Player sends a chat message                              |
| `PlayerTransferEvent`      | –           | Player is transferred to another server                  |

---

## Scheduling Tasks

The scheduler allows delayed, repeating, and async tasks.

```java
// Run after 20 ticks (1 second)
this.getProxy().getScheduler().scheduleDelayed(() -> {
    this.getLogger().info("Delayed task executed!");
}, 20);

// Run repeatedly every 100 ticks (5 seconds)
this.getProxy().getScheduler().scheduleRepeating(() -> {
    this.getLogger().info("Repeating task executed!");
}, 100);

// Run asynchronously
this.getProxy().getScheduler().scheduleAsync(() -> {
    // Heavy computation here
});
```

**Tick timing:** In WaterdogPE, one second equals 20 ticks.

---

## Building and Deploying

For detailed build instructions (including online and offline builds for Windows and Linux), please refer to the dedicated [BUILD.md](BUILD.md) file.
> **Building your plugin?** Check out [BUILD.md](BUILD.md) for step‑by‑step instructions on how to compile your plugin both online and offline on any platform.

---

## Example Plugins

For more examples, refer to the official [WaterdogPE Example‑plugins](https://github.com/WaterdogPE/Example-plugins) repository, which contains simple plugin implementations demonstrating various API features.

---

## Resources & Further Reading

- **Official Documentation:** [https://docs.waterdog.dev](https://docs.waterdog.dev)
- **Plugin API Introduction:** [https://docs.waterdog.dev/plugins/introduction](https://docs.waterdog.dev/plugins/introduction)
- **Commands Guide:** [https://docs.waterdog.dev/plugins/commands-guide](https://docs.waterdog.dev/plugins/commands-guide)
- **Events Guide:** [https://docs.waterdog.dev/plugins/events-guide](https://docs.waterdog.dev/plugins/events-guide)
- **Working with Players:** [https://docs.waterdog.dev/plugins/players-guide](https://docs.waterdog.dev/plugins/players-guide)
- **Fallback & Join Handler:** [https://docs.waterdog.dev/plugins/fallback-join-handler](https://docs.waterdog.dev/plugins/fallback-join-handler)
- **Scheduling Tasks:** [https://docs.waterdog.dev/plugins/scheduling-task](https://docs.waterdog.dev/plugins/scheduling-task)
- **GitHub Repository:** [https://github.com/WaterdogPE/WaterdogPE](https://github.com/WaterdogPE/WaterdogPE)

---

## License

WaterdogPE is licensed under the GNU General Public License, Version 2.0.  
This repository is provided as a template and guide; you are free to use and modify it for your own plugins.

---

*This guide is based on the official WaterdogPE Plugin API documentation. For the most up‑to‑date information, always refer to the official docs at [docs.waterdog.dev](https://docs.waterdog.dev).*

---
