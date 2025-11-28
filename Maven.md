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

