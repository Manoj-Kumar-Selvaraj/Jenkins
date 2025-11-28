---

# 🔥 Let’s Start with **MAVEN** (Technology 1)

---

## 🟢 What is Maven?

> Maven is a build automation & dependency management tool mainly for Java applications.

---

## 🟡 1. Key Files in Maven

### **1️⃣ pom.xml** → The heart of Maven

Defines:

✔ Project metadata
✔ Dependencies
✔ Plugins
✔ Build instructions
✔ Versioning

Example:

```xml
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.demo</groupId>
  <artifactId>myapp</artifactId>
  <version>1.0.0</version>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
      <version>3.2.1</version>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.11.0</version>
        <configuration>
          <source>17</source>
          <target>17</target>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>
```

---

### **2️⃣ settings.xml**

Used for:

| Purpose                        | Examples                   |
| ------------------------------ | -------------------------- |
| Private repository credentials | Nexus / Artifactory        |
| Proxy config                   | Corporate networks         |
| Repository mirrors             | Faster dependency download |

Location:

```
~/.m2/settings.xml
```

Sample:

```xml
<settings>
  <mirrors>
    <mirror>
      <id>central-mirror</id>
      <url>https://nexus.local/repository/maven-public/</url>
      <mirrorOf>*</mirrorOf>
    </mirror>
  </mirrors>
</settings>
```

---

## 🟣 2. Maven Lifecycle

Maven runs in phases (CI/CD uses these phases heavily)

| Phase        | What it does                        |
| ------------ | ----------------------------------- |
| **validate** | Check project structure             |
| **compile**  | Compiles code                       |
| **test**     | Runs unit tests                     |
| **package**  | Creates JAR/WAR                     |
| **verify**   | Integrations tests                  |
| **install**  | Places artifact in local repository |
| **deploy**   | Publish artifact to remote repo     |

---

## 🟠 3. Maven Commands (Most used)

| Action                   | Command                         |
| ------------------------ | ------------------------------- |
| Build and test           | `mvn clean install`             |
| Build skip tests         | `mvn clean package -DskipTests` |
| Run tests only           | `mvn test`                      |
| Deploy artifact to Nexus | `mvn deploy`                    |
| Download dependency tree | `mvn dependency:tree`           |

These are used in **Jenkins build stage**.

---

## 🟢 Maven in CI/CD Pipeline (Jenkinsfile Example)

```groovy
stage('Build') {
  steps {
    sh "mvn clean package -DskipTests"
  }
}

stage('Publish') {
  steps {
    sh "mvn deploy"
  }
}
```

This builds → creates artifact → pushes to repo.

---

Now we go **Maven to Master level**.
We will drill deeper into **settings.xml**, **pom.xml** and **all real-time Maven commands used in CI/CD**.

We will go in this order:

### 🔥 Maven Deep Dive Roadmap

1. **settings.xml (complete coverage)**
2. **pom.xml (full anatomy + plugins + profiles + inheritance)**
3. **Maven lifecycle + phases + goals**
4. **All Maven commands for CI/CD, debugging, optimization**
5. **Real deployment scenarios (Nexus, Artifactory, Private Repo)**

We start with **settings.xml first** as you asked.

---

# 🟢 1. **settings.xml — Deep Mastery**

`settings.xml` is used for **environment-specific configuration**, NOT project-specific.

📌 pom.xml = Project scope
📌 settings.xml = Machine/User scope

---

## 📍 Location of settings.xml

| OS          | Path                               |
| ----------- | ---------------------------------- |
| Linux / Mac | `~/.m2/settings.xml`               |
| Windows     | `C:\Users\<User>\.m2\settings.xml` |

---

## 🔥 Why settings.xml Exists?

| Without settings.xml           | With settings.xml                 |
| ------------------------------ | --------------------------------- |
| Maven uses default repo (slow) | Can set mirrors for faster builds |
| No proxy support               | Can add proxy config              |
| No private repo auth           | Can store credentials             |
| Can't change repository policy | We can override remote repos      |

---

## 🔥 settings.xml Structure

```xml
<settings>
   <localRepository>/path/to/.m2/repo</localRepository>
   <mirrors>...</mirrors>
   <servers>...</servers>
   <proxies>...</proxies>
   <profiles>...</profiles>
   <activeProfiles>...</activeProfiles>
</settings>
```

### Let's learn each part in depth 👇

---

### 1️⃣ localRepository

Used to change where Maven stores dependencies.

```xml
<localRepository>/opt/maven-repo</localRepository>
```

Useful when attaching large SSD/NAS for performance.

---

### 2️⃣ servers → **credentials storage**

Used to store authentication for **private repos** (Nexus/Artifactory/ECR).

```xml
<servers>
  <server>
    <id>nexus-releases</id>
    <username>deploy_user</username>
    <password>securePass</password>
  </server>
</servers>
```

Then in `pom.xml` you reference:

```xml
<distributionManagement>
  <repository>
    <id>nexus-releases</id>
    <url>https://nexus.company.com/repository/releases/</url>
  </repository>
</distributionManagement>
```

🔥 **id must match** — interview question!

---

### 3️⃣ mirrors → Override default repo

To replace Maven Central with internal mirror (common in companies).

```xml
<mirrors>
  <mirror>
    <id>corp-mirror</id>
    <mirrorOf>*</mirrorOf>
    <url>https://nexus.company.com/repo/public/</url>
  </mirror>
</mirrors>
```

✔ Faster download
✔ Full control of dependencies
✔ Can block banned libs (security)

---

### 4️⃣ proxies (corporate networks)

```xml
<proxies>
  <proxy>
    <active>true</active>
    <protocol>http</protocol>
    <host>10.10.1.200</host>
    <port>8080</port>
    <username>proxyUser</username>
    <password>proxyPass</password>
  </proxy>
</proxies>
```

---

### 5️⃣ profiles → activate custom build environments (POWERFUL)

Example: Switch repo based on environment

```xml
<profiles>
  <profile>
    <id>dev</id>
    <repositories>
      <repository>
        <id>dev-repo</id>
        <url>http://nexus.dev/repo</url>
      </repository>
    </repositories>
  </profile>

  <profile>
    <id>prod</id>
    <repositories>
      <repository>
        <id>prod-repo</id>
        <url>http://nexus.prod/repo</url>
      </repository>
    </repositories>
  </profile>
</profiles>
```

Activate:

```xml
<activeProfiles>
  <activeProfile>prod</activeProfile>
</activeProfiles>
```

OR via command:

```
mvn clean install -Pdev
```

---

# 🏆 settings.xml Interview Killer Lines

📌 *settings.xml controls how Maven behaves in an environment.*
📌 *pom.xml controls how the project is built.*
📌 *Mirror + Profile + Servers = real enterprise setup.*

Best one-liner:

> "**settings.xml is where we secure access to private repos, set mirrors for performance, define proxies, and activate custom build profiles.**"

---

### 🚀 **POM.XML — Deep Dive (MASTER-Level)**

This is where we become **deployment-grade strong** in Maven.
We will break `pom.xml` into every meaningful section used in real enterprise CI/CD.

---

## 🔥 What is `pom.xml`?

> **POM (Project Object Model)** tells Maven how to *build*, *test*, *package*, *version*, and *deploy* a project.
> It is the **heart** of Maven.

### Interview One-Liner

> *If Maven is a machine, `pom.xml` is the blueprint.*

---

# 🟢 Core Structure of pom.xml

```xml
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.company</groupId>
  <artifactId>inventory-service</artifactId>
  <version>1.0.0</version>
  <packaging>jar</packaging>
</project>
```

---

## 1️⃣ **groupId / artifactId / version** → (GAV Coordinates)

| Tag        | Meaning                                |
| ---------- | -------------------------------------- |
| groupId    | Organization or Project Namespace      |
| artifactId | Name of the application module         |
| version    | Build Release ID (snapshot or release) |

📌 These three uniquely identify ANY Java artifact.

### Real Example:

```
com.company.billing:payment-api:2.4.1-SNAPSHOT
```

---

## 2️⃣ **packaging Types**

| packaging   | Output                         |
| ----------- | ------------------------------ |
| jar         | Standard Java output           |
| war         | Web application (Tomcat/JBoss) |
| ear         | Enterprise archive (old J2EE)  |
| pom         | Parent/inheritance module      |
| spring-boot | Bundles embedded server        |

---

## 3️⃣ Dependencies (MOST IMPORTANT)

```xml
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>3.2.1</version>
    <scope>compile</scope>
  </dependency>
</dependencies>
```

### Supported `scope`

| Scope             | Included at Runtime? | Included in Jar/WAR? | Used For                         |
| ----------------- | -------------------- | -------------------- | -------------------------------- |
| compile (default) | ✔                    | ✔                    | Most dependencies                |
| provided          | ✖                    | ✖                    | Servlet containers handle this   |
| runtime           | ✔                    | ✖                    | MySQL/JDBC drivers               |
| test              | ✖ (prod)             | ✖                    | JUnit, Mockito                   |
| system            | ✔                    | ✔                    | Local manually-added JARs (rare) |

---

## 4️⃣ Build Section (Plugins + Execution = CI/CD Power)

```xml
<build>
  <plugins>

    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-compiler-plugin</artifactId>
      <version>3.11.0</version>
      <configuration>
        <source>17</source>
        <target>17</target>
      </configuration>
    </plugin>

  </plugins>
</build>
```

### Common Build Plugins

| Plugin                   | Purpose              |
| ------------------------ | -------------------- |
| maven-compiler-plugin    | Java compile version |
| maven-surefire-plugin    | Unit test execution  |
| maven-jar-plugin         | JAR build config     |
| maven-war-plugin         | WAR packaging        |
| spring-boot-maven-plugin | Boot runnable jars   |
| maven-deploy-plugin      | Deploy to repo       |

---

## 5️⃣ Profiles (for different environments)

Multiple env → same code → different behavior

```xml
<profiles>
  <profile>
    <id>prod</id>
    <properties>
      <db.url>jdbc:mysql://prod/db</db.url>
    </properties>
  </profile>

  <profile>
    <id>dev</id>
    <properties>
      <db.url>jdbc:mysql://dev/db</db.url>
    </properties>
  </profile>
</profiles>
```

Activate via:

```
mvn clean install -Pprod
```

🔥 Interview Line:

> *Profiles allow environment-based builds without modifying code.*

---

## 6️⃣ Parent POM + Multi Module Projects

Large enterprise uses **parent + children POMs**.

```xml
<parent>
  <groupId>com.company.parent</groupId>
  <artifactId>platform-pom</artifactId>
  <version>1.0</version>
</parent>
```

Benefits:

| Benefit                                | Why used                    |
| -------------------------------------- | --------------------------- |
| Centralized version control            | Same lib version everywhere |
| Common plugins for all modules         | No duplication              |
| Single command to build whole monorepo | `mvn install` builds all    |

---

## 7️⃣ Dependency Management (For version consistency)

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
      <version>2.15.2</version>
    </dependency>
  </dependencies>
</dependencyManagement>
```

Children only write:

```
<dependency>
  <groupId>...</groupId>
  <artifactId>...</artifactId>
</dependency>
```

Version auto-inherited.

---

# 🟣 Full Real-Time Enterprise POM Example

```xml
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.company.app</groupId>
  <artifactId>order-service</artifactId>
  <version>3.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <properties>
    <java.version>17</java.version>
    <spring.version>3.2.0</spring.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-j</artifactId>
      <scope>runtime</scope>
    </dependency>
    <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
</project>
```

---

### 🏆 Interview Killer One-Liners for POM.XML

| Use this & interviewer will remember you                                              |
| ------------------------------------------------------------------------------------- |
| *POM is the DNA of a project — it defines version, build, dependencies & deployment.* |
| *settings.xml decides where to fetch; pom.xml decides what to build.*                 |
| *Profiles + DependencyManagement + Plugins = Enterprise Maven.*                       |

---

## 0️⃣ Tiny XML Primer (very important)

A POM is just **XML**.

* A **tag** looks like: `<groupId> ... </groupId>`
* An **element** = opening tag + content + closing tag
* An **attribute** is inside a tag:
  Example: `<project xmlns="...">` → `xmlns` is an attribute
* Tags are **case-sensitive**: `<groupId>` ≠ `<GroupId>`

---

## 1️⃣ A Simple POM We’ll Explain Fully

Here is a small but real `pom.xml`:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>demo-app</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>demo-app</name>
    <description>Demo application</description>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>3.2.1</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.11.0</version>
                <configuration>
                    <source>17</source>
                    <target>17</target>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

Now let’s go through it **piece by piece**.

---

## 2️⃣ The `<project ...>` Tag and Its Attributes

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">
```

* `<project`

  * This is the **root element** of the POM.
  * Every POM **must** start with `<project ...>` and end with `</project>`.

* `xmlns="http://maven.apache.org/POM/4.0.0"`

  * `xmlns` = **XML Namespace**.
  * It tells tools: *“This XML follows Maven POM 4.0.0 format.”*
  * You don’t change this; for modern POMs it’s almost always this value.

* `xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"`

  * Declares another namespace with the prefix `xsi`.
  * `xsi` = XML Schema Instance.
  * This is standard XML stuff so we can use attributes like `xsi:schemaLocation`.

* `xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd"`

  * This says: *“For documents in the Maven POM namespace, use this XSD file to validate.”*
  * First URL = namespace, second URL = XSD file (schema definition).
  * Again, you rarely modify this in normal usage.

So that whole line basically means:

> “This is a Maven POM file in version 4.0.0 format, and here’s the schema to validate it.”

The closing tag appears at the end:

```xml
</project>
```

That closes the **entire POM**.

---

## 3️⃣ `<modelVersion>`

```xml
<modelVersion>4.0.0</modelVersion>
```

* `modelVersion`

  * This tells Maven what **POM model version** is used.
  * For Maven 2+ this is almost always `4.0.0`.
  * You **don’t invent your own** here; you just keep `4.0.0`.

---

## 4️⃣ GAV: `<groupId>`, `<artifactId>`, `<version>`

These three uniquely identify your project/artifact in repositories.

```xml
<groupId>com.example</groupId>
<artifactId>demo-app</artifactId>
<version>1.0.0-SNAPSHOT</version>
```

### `<groupId>com.example</groupId>`

* `groupId`

  * A kind of **namespace / package** for your project.
  * Usually reverse domain style: `com.company`, `org.myorg.app`
  * Used to avoid naming conflicts between different organizations.

### `<artifactId>demo-app</artifactId>`

* `artifactId`

  * The actual **name of the artifact** (the JAR/WAR name).
  * When Maven builds, you normally get something like:
    `demo-app-1.0.0-SNAPSHOT.jar`

### `<version>1.0.0-SNAPSHOT</version>`

* `version`

  * The specific **version** of this build.
  * Common pattern: `major.minor.patch`

    * `1.0.0`, `1.0.1`, `2.1.3`, etc.
  * `-SNAPSHOT` means **“in development / not a final release”**.

    * Release versions usually drop `-SNAPSHOT` → `1.0.0`.

Together, these three form:

```text
com.example : demo-app : 1.0.0-SNAPSHOT
```

This is how dependencies are identified.

---

## 5️⃣ `<packaging>`

```xml
<packaging>jar</packaging>
```

* `packaging`

  * Tells Maven **what kind of file to build**.
  * Common values:

    * `jar` → normal library or backend app
    * `war` → web app for servlet containers
    * `pom` → parent/inherited project
    * `ear` → enterprise application
  * If not specified, default is `jar`.

---

## 6️⃣ `<name>` and `<description>`

```xml
<name>demo-app</name>
<description>Demo application</description>
```

* `<name>`

  * Human-friendly name of your project.
  * Shown in some tools (like IDEs, repositories).

* `<description>`

  * Short description of what this project does.
  * Optional but nice for documentation.

These don’t affect the build logic; they are metadata.

---

## 7️⃣ `<properties>` Block

```xml
<properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
</properties>
```

* `<properties>`

  * A container for **custom values** that can be reused elsewhere in the POM.

* `<maven.compiler.source>17</maven.compiler.source>`

  * This is a **property** that tells Maven:
    *“Compile the source code as Java 17 syntax.”*
  * It is often used by compiler plugin.

* `<maven.compiler.target>17</maven.compiler.target>`

  * This tells Maven:
    *“Generate bytecode compatible with Java 17 JVM.”*

You can reference **any property** using `${property.name}` syntax.

Example:

```xml
<properties>
    <java.version>17</java.version>
</properties>

<dependency>
    <groupId>something</groupId>
    <artifactId>lib</artifactId>
    <version>${java.version}</version>
</dependency>
```

Here `${java.version}` will be replaced with `17`.

---

## 8️⃣ `<dependencies>` and `<dependency>` Blocks

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <version>3.2.1</version>
    </dependency>
</dependencies>
```

* `<dependencies>`

  * A **list** container holding multiple `<dependency>` items.

* `<dependency>`

  * One single dependency definition.

Inside each `<dependency>`:

* `<groupId>org.springframework.boot</groupId>`

  * Same idea as your project’s groupId, but this time it’s the **library’s** namespace.

* `<artifactId>spring-boot-starter-web</artifactId>`

  * Name of the **library** you want to use.

* `<version>3.2.1</version>`

  * Specific version of that library.

Optionally, you may see `<scope>`:

```xml
<scope>test</scope>
```

* `scope` controls **where / when** the dependency is used:

  * `compile` (default) – needed for compiling & running
  * `test` – only used in test code
  * `runtime` – needed only when running, not compiling
  * `provided` – available in server/container (e.g. Tomcat provides it)

---

## 9️⃣ `<build>`, `<plugins>`, `<plugin>`, `<configuration>`

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.11.0</version>
            <configuration>
                <source>17</source>
                <target>17</target>
            </configuration>
        </plugin>
    </plugins>
</build>
```

Let’s break each tag:

### `<build>`

* This block contains **build-related customization**.
* Here you specify plugins and sometimes output dirs, filters, etc.

### `<plugins>`

* Container holding multiple `<plugin>` entries.

### `<plugin>`

This is a **Maven plugin**, which is like an extension that adds tasks to Maven.

Inside `<plugin>`:

* `<groupId>org.apache.maven.plugins</groupId>`

  * The group under which Maven plugins are published.
  * Maven’s own official plugins are usually under `org.apache.maven.plugins`.

* `<artifactId>maven-compiler-plugin</artifactId>`

  * The specific plugin name.
  * `maven-compiler-plugin` = plugin that compiles Java code.

* `<version>3.11.0</version>`

  * Version of the plugin you want to use.

* `<configuration> ... </configuration>`

  * Plugin-specific settings (not general Maven syntax; the plugin defines these).
  * For compiler plugin:

    * `<source>17</source>` = Java source level
    * `<target>17</target>` = generated bytecode target level

So when Maven runs the **compile phase**, it uses this plugin with this configuration.

---

## 🔟 Closing `</project>`

```xml
</project>
```

This closes the root `<project>` tag.
Everything between `<project ...>` and `</project>` is your POM.

---

## 1️⃣1️⃣ How to Think About It (Mental Model)

* **Root (`<project>`)** → “This is a Maven project.”
* **GAV (groupId, artifactId, version)** → “Who am I?”
* **packaging** → “What kind of file do I produce?”
* **dependencies** → “What other libraries do I need?”
* **build/plugins** → “What extra tools/steps should Maven use to build?”
* **properties** → “Variables / constants I can reuse.”
* **settings.xml vs pom.xml**

  * `settings.xml` = about **your machine / environment**
  * `pom.xml` = about **this project**

---

## 📌 What Are Maven Profiles?

Profiles allow you to **change Maven behavior/environment without modifying code**.
You can have:

* different database URLs
* different dependencies
* different build configs
* different packaging types
* different deployment repos

Based on environment: **dev / qa / prod**

You activate it via:

```
mvn clean install -Pdev
```

or permanently inside `settings.xml`.

---

# 🔥 Full Example POM with Profiles (We Will Break Down Entirely)

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example.app</groupId>
    <artifactId>profile-demo</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>

    <properties>
        <spring.version>3.2.1</spring.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>${spring.version}</version>
        </dependency>
    </dependencies>

    <profiles>

        <profile>
            <id>dev</id>
            <properties>
                <db.url>jdbc:mysql://dev-server:3306/devdb</db.url>
                <log.level>DEBUG</log.level>
            </properties>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
        </profile>

        <profile>
            <id>prod</id>
            <properties>
                <db.url>jdbc:mysql://prod-server:3306/proddb</db.url>
                <log.level>ERROR</log.level>
            </properties>
            <build>
                <plugins>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-surefire-plugin</artifactId>
                        <version>3.1.2</version>
                        <configuration>
                            <skipTests>false</skipTests>
                        </configuration>
                    </plugin>
                </plugins>
            </build>
        </profile>

    </profiles>

</project>
```

---

# 🔥 Now we break down the new things inside this POM

## 🟢 `<profiles>` (container tag)

```
<profiles>
    ...profiles inside...
</profiles>
```

* `<profiles>` is the **parent tag**
* It means **multiple profiles are defined inside**
* Nothing executes here — it just groups them

---

## 🟡 `<profile>` (one environment definition)

```
<profile>
   ...
</profile>
```

* **One single profile block**
* You can have as many `<profile>` entries as you want
* Each profile changes Maven build behavior differently

---

## 🟣 `<id>` — **Profile Name** (VERY IMPORTANT)

```
<id>dev</id>
```

* The name of the profile
* You use this to activate it
* Command: `mvn package -Pdev`

Profile 2:

```
<id>prod</id>
```

Command: `mvn clean install -Pprod`

⚠ `id` must be unique for each profile.

---

## 🔵 `<properties>` inside a profile

```
<properties>
    <db.url>jdbc:mysql://dev-server/devdb</db.url>
    <log.level>DEBUG</log.level>
</properties>
```

* Defines **environment-specific values**
* These variables can be used anywhere in POM using `${db.url}`
* dev profile uses DEBUG logs
* prod profile uses ERROR logs

So properties change per environment.

---

## 🔥 `<activation>` (Automatic profile selection)

```
<activation>
    <activeByDefault>true</activeByDefault>
</activation>
```

* This means:
  **If no profile is selected → use this one automatically**
* Usually `dev` is default
* prod must be chosen only when needed:

```
mvn package -Pprod
```

🚀 Interview line:

> *Activation decides when a profile wakes up.*

---

## 🟠 `<build>` inside a profile

```
<build>
    <plugins>
        <plugin>
            <artifactId>maven-surefire-plugin</artifactId>
```

This means:

🔹 Only when **prod profile** is active →
Maven will use this **test execution plugin**

So *build behavior changes based on environment*.

In dev profile we don’t specify `<build>` → so default config runs.

---

# 🧠 Why are profiles so powerful?

| Feature                | Benefit                            |
| ---------------------- | ---------------------------------- |
| Different DB config    | dev/test/prod DB URLs              |
| Different log level    | DEBUG in dev, ERROR in prod        |
| Different plugins      | Run tests in prod, skip in dev     |
| Different repositories | Local repo for dev, Nexus for prod |
| Different packaging    | JAR for dev, WAR for prod          |

Using profiles, **one codebase → many build behaviors**.

---

# 🏆 Interview-grade one-liners

Use these and interviewers will nod:

| Line                                                                       |
| -------------------------------------------------------------------------- |
| *Profiles helps Maven behave differently without changing code.*           |
| *We use `-Pdev` or `-Pprod` to activate different build environments.*     |
| *activation makes profiles auto-selectable, otherwise they are manual.*    |
| *Profiles can change properties, dependencies, plugins or even packaging.* |

---

# 🔥 Why Multi-Module Project Exists?

Instead of having 10 separate Maven projects (billing, order, user, payment), we create:

```
Root Parent (contains build rules, versions, plugin configs)
 ├── service-user
 ├── service-order
 ├── service-payment
 └── service-inventory
```

Child modules **inherit**:

✔ dependency versions
✔ plugins
✔ build configuration
✔ profiles
✔ java version
✔ common rules

---

# 🟢 Structure of a Multi-Module Maven Project

```
parent-project/                  (root)
│── pom.xml  <-- Parent POM (packaging = pom)
├── service-user/
│   └── pom.xml  <-- Child 1
├── service-order/
│   └── pom.xml  <-- Child 2
└── service-payment/
    └── pom.xml  <-- Child 3
```

---

# 🔥 FULL EXAMPLE (We Will Explain LINE-BY-LINE)

## **Parent POM (Root pom.xml)**

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.company.app</groupId>
    <artifactId>parent-project</artifactId>
    <version>1.0.0</version>
    <packaging>pom</packaging>
```

---

### BREAKDOWN:

| Tag                          | meaning                                                                                              |
| ---------------------------- | ---------------------------------------------------------------------------------------------------- |
| `<project>`                  | Root element of POM                                                                                  |
| `<groupId>`                  | Global namespace of company/application                                                              |
| `<artifactId>`               | Name of main parent project                                                                          |
| `<version>`                  | Version for the parent BOM                                                                           |
| `<packaging>pom</packaging>` | 🟥 MOST IMPORTANT — this means *this project will NOT create JAR/WAR* → it is ONLY a parent template |

Without `packaging=pom`, it CANNOT be a parent POM.

🔥 Interview Statement:

> Parent POM must always use packaging type `pom`.

---

Next part inside parent:

```xml
<modules>
    <module>service-user</module>
    <module>service-order</module>
    <module>service-payment</module>
</modules>
```

### BREAKDOWN:

| Tag                                | Meaning                                                     |
| ---------------------------------- | ----------------------------------------------------------- |
| `<modules>`                        | List of child projects this parent controls                 |
| `<module>service-user</module>`    | Looks for folder `service-user/` containing a child pom.xml |
| `<module>service-order</module>`   | Another child                                               |
| `<module>service-payment</module>` | Another child                                               |

---

### DependencyManagement inside Parent

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>3.2.1</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

Meaning:

* Child modules can use `spring-boot-starter-web` **without specifying version**.
* Version is controlled from one place → here.

So upgrading Spring Boot is 1-line change.

---

### Build plugins inside parent

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.11.0</version>
            <configuration>
                <source>17</source>
                <target>17</target>
            </configuration>
        </plugin>
    </plugins>
</build>
```

Meaning:

| Behavior                             | Result              |
| ------------------------------------ | ------------------- |
| All child modules inherit Java 17    | No duplicate config |
| Same compiler plugin for all modules | Consistent build    |

---

✔ Parent POM DONE.
Next — we explain CHILD POM line-by-line.

---

## **Child Module pom.xml (Example: service-user)**

```xml
<project>
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.company.app</groupId>
        <artifactId>parent-project</artifactId>
        <version>1.0.0</version>
        <relativePath>../pom.xml</relativePath>
    </parent>

    <artifactId>service-user</artifactId>
```

### BREAKDOWN:

| Tag                       | Meaning                                           |
| ------------------------- | ------------------------------------------------- |
| `<parent>` block          | Connects this child to parent POM                 |
| `<groupId>` inside parent | Must match parent                                 |
| `<artifactId>`            | Must match parent project artifactId              |
| `<version>`               | Parent version                                    |
| `<relativePath>`          | Where to find parent pom (default ../ is correct) |

🔥 MOST IMPORTANT:

### Child does NOT need to define groupId

because it inherits it from the parent automatically.

Child only defines:

```
<artifactId>service-user</artifactId>
```

So full artifact becomes:

```
com.company.app : service-user : 1.0.0
```

---

### Dependencies inside child

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

### NOTE 💥 — No `<version>` tag

Because version was declared in `dependencyManagement` of parent.

Child inherits version automatically.

---

# 🔥 ONE COMMAND BUILDS EVERYTHING

From root parent folder:

```
mvn clean install
```

All modules get built in order:

```
service-user → service-order → service-payment
```

No need to go into each folder.

---

# 🏆 Interview Killer Lines

| Use any of these                                                                |
| ------------------------------------------------------------------------------- |
| Multi-module = One parent defines rules, many children inherit them.            |
| Parent POM uses `packaging=pom` because it exists only to configure, not build. |
| Child modules don't need versions — inherited from dependencyManagement.        |
| One `mvn install` builds all microservices together.                            |
| Parent POM gives standardization & prevents duplication across modules.         |

---
