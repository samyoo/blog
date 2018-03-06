
---
title: maven配置nexus本地仓库
date: 2018-03-06 11:29:44
tags: [java,maven,nexus]
categories: java
---

## 配置文件如下：

```
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
    <!-- localRepository
     | The path to the local repository maven will use to store artifacts.
     |
     | Default: ${user.home}/.m2/repository
    <localRepository>/path/to/local/repo</localRepository>
    -->

    <!-- interactiveMode
     | This will determine whether maven prompts you when it needs input. If set to false,
     | maven will use a sensible default value, perhaps based on some other setting, for
     | the parameter in question.
     |
     | Default: true
    <interactiveMode>true</interactiveMode>
    -->

    <!-- offline
     | Determines whether maven should attempt to connect to the network when executing a build.
     | This will have an effect on artifact downloads, artifact deployment, and others.
     |
     | Default: false
    <offline>false</offline>
    -->

    <pluginGroups>
    </pluginGroups>

    <proxies>
    </proxies>

    <servers>
        <server>
            <id>releases</id>
            <username>deployment</username>
            <password>12345678</password>
        </server>
        <server>
            <id>snapshots</id>
            <username>deployment</username>
            <password>12345678</password>
        </server>
    </servers>

    <mirrors>
        <mirror>
            <id>public-nexus</id>
            <name>Human Readable Name for this Mirror.</name>
            <url>http://192.168.60.30:8081/nexus/content/groups/public/</url>
            <mirrorOf>*</mirrorOf>
        </mirror>
    </mirrors>

    <profiles>
        <profile>
            <id>jdk18</id>
            <activation>
                <activeByDefault>true</activeByDefault>
                <jdk>1.8</jdk>
            </activation>
            <properties>
                <maven.compiler.source>1.8</maven.compiler.source>
                <maven.compiler.target>1.8</maven.compiler.target>
                <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
            </properties>
        </profile>
        <profile>
            <id>nexus-profile-iss</id>
            <repositories>
                <repository>
                    <id>nexus-iss</id>
                    <url>http://192.168.60.30:8081/nexus/content/groups/public/</url>
                    <releases>
                        <enabled>true</enabled>
                    </releases>
                    <snapshots>
                        <enabled>true</enabled>
                    </snapshots>
                </repository>
            </repositories>
            <pluginRepositories>
                <pluginRepository>
                    <id>nexus-iss</id>
                    <url>http://192.168.60.30:8081/nexus/content/groups/public/</url>
                    <releases>
                        <enabled>true</enabled>
                    </releases>
                    <snapshots>
                        <enabled>true</enabled>
                    </snapshots>
                </pluginRepository>
            </pluginRepositories>
        </profile>
    </profiles>

    <activeProfiles>
        <activeProfile>jdk18</activeProfile>
        <activeProfile>nexus-profile-iss</activeProfile>
    </activeProfiles>
</settings>

```