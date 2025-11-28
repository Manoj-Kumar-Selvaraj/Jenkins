---

# 📌 **MAVEN FLASHCARDS — COMPLETE SET**

---

## 🟥 Flashcard 1 — What is Maven?

```
Maven = Build automation + Dependency management + Project lifecycle tool.
```

---

## 🟥 Flashcard 2 — Key Maven Terms

| Term      | Meaning                       |
| --------- | ----------------------------- |
| Lifecycle | Complete build flow           |
| Phase     | Step inside lifecycle         |
| Plugin    | Worker that performs a task   |
| Goal      | A specific task inside plugin |

---

## 🟥 Flashcard 3 — 3 Maven Lifecycles

```
1) clean → delete previous build
2) default → compile → test → package → install → deploy
3) site → generate documentation site
```

---

## 🟦 Flashcard 4 — Default Lifecycle Phases

```
validate → compile → test → package → verify → install → deploy
```

---

## 🟦 Flashcard 5 — Phase Meaning

| Phase    | What it does             |
| -------- | ------------------------ |
| validate | Checks structure         |
| compile  | Compile source code      |
| test     | Run tests                |
| package  | Create jar/war           |
| verify   | Run integration checks   |
| install  | Save to local repository |
| deploy   | Publish to remote repo   |

---

## 🟦 Flashcard 6 — SUPER IMPORTANT RULE

```
Running a later phase runs all earlier phases automatically.
Example: mvn package = validate+compile+test+package
```

---

## 🟩 Flashcard 7 — Top Commands

```
mvn clean
mvn compile
mvn test
mvn package
mvn install
mvn deploy
```

---

## 🟩 Flashcard 8 — Most Used CI Commands

```
mvn clean package -DskipTests
mvn clean install -U
mvn test -Dtest=ClassName
```

---

## 🟩 Flashcard 9 — Useful Options

| Option      | Meaning                |
| ----------- | ---------------------- |
| -DskipTests | Build without tests    |
| -Dtest=Name | Run only specific test |
| -X          | Debug output           |
| -q          | Quiet logs             |
| -U          | Force snapshot update  |
| -o          | Offline mode           |
| -T4         | Parallel build threads |

---

## 🟪 Flashcard 10 — pom.xml Core Tags

```
<groupId>com.company</groupId>
<artifactId>service</artifactId>
<version>1.0.0</version>
<packaging>jar|war|pom</packaging>
```

---

## 🟪 Flashcard 11 — Dependency Structure

```xml
<dependency>
  <groupId>org.example</groupId>
  <artifactId>libname</artifactId>
  <version>1.2.3</version>
  <scope>compile|test|runtime|provided</scope>
</dependency>
```

---

## 🟪 Flashcard 12 — Dependency Scope

| Scope    | Meaning                       |
| -------- | ----------------------------- |
| compile  | Default, needed everywhere    |
| test     | Used only while testing       |
| runtime  | Needed at runtime only        |
| provided | Available in server/container |
| system   | Local JAR (rare)              |

---

## 🟧 Flashcard 13 — settings.xml Purpose

```
Machine-level configs:
✓ private repo credentials
✓ proxy settings
✓ mirrors for fast download
✓ activate profiles
```

---

## 🟧 Flashcard 14 — settings.xml Structure

```xml
<servers>credentials</servers>
<mirrors>repo overrides</mirrors>
<profiles>runtime configs</profiles>
<activeProfiles>auto selection</activeProfiles>
```

---

## 🟨 Flashcard 15 — DependencyManagement

```
Controls dependency versions globally.
Child can use dependency WITHOUT writing version.
```

---

## 🟨 Flashcard 16 — Multi-Module Project Structure

```
Parent (packaging=pom)
 ├─ module-1
 ├─ module-2
 ├─ module-3
```

---

## 🟨 Flashcard 17 — Parent Definition in Child

```xml
<parent>
  <groupId>com.company</groupId>
  <artifactId>parent-pom</artifactId>
  <version>1.0.0</version>
</parent>
```

---

## 🟦 Flashcard 18 — Profiles

```
Different build behavior for dev/qa/prod.
Activated using: mvn install -Pdev
```

---

## 🟦 Flashcard 19 — Example Profile

```xml
<profile>
  <id>prod</id>
  <properties>
    <db.url>jdbc::prod</db.url>
  </properties>
</profile>
```

---

## 🟦 Flashcard 20 — Must Remember Interview Answers

```
POM defines HOW to build.
settings.xml defines WHERE from.
dependencyManagement controls versions globally.
Running mvn package runs all steps before package.
Parent POM centralizes config for child modules.
```
