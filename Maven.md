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

Reply **Gradle** to continue.🚀
