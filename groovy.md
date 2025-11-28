---

# 🧱 1. Declarative Pipeline – Master Syntax

A fully loaded template:

```groovy
pipeline {
    agent any

    options {
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
        disableConcurrentBuilds()
    }

    environment {
        APP_ENV = 'dev'
        VERSION = "1.0.${BUILD_NUMBER}"
    }

    parameters {
        string(name: 'BRANCH', defaultValue: 'main', description: 'Git branch to build')
        choice(name: 'ENV', choices: ['dev', 'qa', 'prod'], description: 'Target environment')
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run tests?')
    }

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    triggers {
        cron('H 2 * * *')
        pollSCM('H/5 * * * *')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: params.BRANCH, url: 'https://github.com/org/repo.git'
            }
        }

        stage('Build') {
            when {
                expression { params.ENV != 'prod' }
            }
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test') {
            when {
                expression { params.RUN_TESTS }
            }
            steps {
                sh 'mvn test'
                junit 'target/surefire-reports/*.xml'
            }
        }

        stage('Deploy') {
            when {
                anyOf {
                    branch 'main'
                    environment name: 'ENV', value: 'prod'
                }
            }
            steps {
                sh "./deploy.sh ${params.ENV}"
            }
        }
    }

    post {
        success {
            echo "Build #${BUILD_NUMBER} succeeded"
        }
        failure {
            echo "Build #${BUILD_NUMBER} failed"
        }
        always {
            cleanWs()
        }
    }
}
```

Now let’s decode **each piece of syntax**.

---

## 1.1 `pipeline { ... }`

* Top-level block – **must exist** for Declarative.
* Contains: `agent`, `options`, `environment`, `parameters`, `stages`, `post`, etc.

---

## 1.2 `agent` syntax

Where builds run.

```groovy
agent any
```

**Options:**

```groovy
agent any                    // any available node
agent none                   // no global agent – set per stage

agent { label 'docker' }     // node with specific label

agent { node { label 'maven' } }   // older style

agent {
    docker {
        image 'maven:3.8.8-eclipse-temurin-17'
        args '-v /root/.m2:/root/.m2'
    }
}
```

Per-stage agent:

```groovy
pipeline {
    agent none
    stages {
        stage('Build') {
            agent { label 'maven-node' }
            steps { sh 'mvn package' }
        }
    }
}
```

---

## 1.3 `options { ... }`

Pipeline-wide options.

```groovy
options {
    timeout(time: 30, unit: 'MINUTES')               // abort after 30m
    buildDiscarder(logRotator(numToKeepStr: '10'))   // keep last 10 builds
    disableConcurrentBuilds()                        // queue if running
    skipDefaultCheckout()                            // don't auto-checkout SCM
    timestamps()                                     // add timestamps to logs
}
```

---

## 1.4 `environment { ... }`

Environment variables visible in all stages.

```groovy
environment {
    APP_ENV = 'dev'
    VERSION = "1.0.${BUILD_NUMBER}"
    PATH = "$PATH:/opt/tools"
}
```

* Use `${VAR}` inside strings.
* Accessible in steps as:

  * Shell: `$APP_ENV`
  * Groovy: `env.APP_ENV`

---

## 1.5 `parameters { ... }`

Job input parameters.

```groovy
parameters {
    string(name: 'BRANCH', defaultValue: 'main', description: 'Git branch')
    choice(name: 'ENV', choices: ['dev', 'qa', 'prod'], description: 'Environment')
    booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run tests')
    text(name: 'NOTES', defaultValue: '', description: 'Release notes')
}
```

Use via:

```groovy
params.BRANCH
params.ENV
params.RUN_TESTS
```

---

## 1.6 `tools { ... }`

Auto-install configured tools (from Jenkins global config).

```groovy
tools {
    jdk 'jdk17'
    maven 'maven3'
    nodejs 'node18'
}
```

---

## 1.7 `triggers { ... }`

Auto build triggers.

```groovy
triggers {
    cron('H 2 * * *')         // nightly at around 2am
    pollSCM('H/5 * * * *')    // every 5 minutes check SCM
}
```

* Use either `cron` or `pollSCM`, or webhooks instead.

---

## 1.8 `stages { stage('Name') { ... } }`

Main work area.

```groovy
stages {
    stage('Build') {
        steps { ... }
    }
}
```

---

### 1.8.1 `steps { ... }`

Actual commands/operations.

Common steps:

```groovy
steps {
    echo "Hello Jenkins"                          // print message
    sh 'ls -l'                                    // run shell (Linux)
    bat 'dir'                                     // run cmd (Windows)

    git branch: 'main', url: 'https://github.com/org/repo.git'

    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
    junit 'target/surefire-reports/*.xml'
    stash name: 'jar', includes: 'target/*.jar'
    unstash 'jar'
}
```

---

### 1.8.2 `when { ... }` – conditional stage execution

```groovy
stage('Deploy') {
    when {
        branch 'main'
        environment name: 'ENV', value: 'prod'
    }
    steps { ... }
}
```

Useful variants:

```groovy
when {
    expression { params.RUN_TESTS }            // Groovy expression
}

when {
    anyOf {
        branch 'main'
        branch 'release/*'
    }
}

when {
    allOf {
        not { branch 'develop' }
        environment name: 'ENV', value: 'prod'
    }
}
```

---

### 1.8.3 `parallel { ... }`

Run blocks in parallel.

```groovy
stage('Tests') {
    parallel {
        stage('Unit Tests') {
            steps { sh 'mvn test' }
        }
        stage('API Tests') {
            steps { sh './run_api_tests.sh' }
        }
        stage('UI Tests') {
            steps { sh './run_ui_tests.sh' }
        }
    }
}
```

---

### 1.8.4 `matrix { ... }` (Declarative)

```groovy
stage('Matrix Build') {
    matrix {
        axes {
            axis {
                name 'OS'
                values 'linux', 'windows'
            }
            axis {
                name 'JDK'
                values '11', '17'
            }
        }
        stages {
            stage('Test') {
                steps {
                    echo "Testing on ${OS} with JDK ${JDK}"
                }
            }
        }
    }
}
```

---

## 1.9 `post { ... }` – actions after pipeline

```groovy
post {
    always {
        echo 'Pipeline finished (success or failure)'
    }
    success {
        echo 'Pipeline succeeded'
    }
    failure {
        echo 'Pipeline failed'
    }
    unstable {
        echo 'Tests unstable'
    }
    changed {
        echo 'Build result changed vs previous'
    }
}
```

You can put `post` inside `stage` too (for stage-specific post).

---

## 1.10 Common Utility Steps (in steps{})

```groovy
timeout(time: 10, unit: 'MINUTES') {
    sh './long_running.sh'
}

retry(3) {
    sh 'flaky_command'
}

input(message: 'Deploy to PROD?', ok: 'Proceed')

script {
    // escape to full Groovy here (for complex logic)
    def files = sh(script: 'ls *.jar', returnStdout: true).trim()
    echo "Found jars: ${files}"
}
```

---

# 🧱 2. Scripted Pipeline – Syntax Essentials

Scripted pipeline is full Groovy-based, more flexible.

### 2.1 Minimal scripted example

```groovy
node {
    stage('Checkout') {
        git 'https://github.com/org/repo.git'
    }

    stage('Build') {
        sh 'mvn clean package'
    }

    stage('Test') {
        sh 'mvn test'
    }

    stage('Deploy') {
        sh './deploy.sh'
    }
}
```

**Keywords:**

* `node { ... }` → allocate an executor/agent
* `stage('Name') { ... }` → visual step
* steps inside are Groovy body.

---

### 2.2 Parallel in scripted

```groovy
node {
    stage('Parallel Tests') {
        parallel(
            "Unit Tests": {
                sh 'mvn test'
            },
            "API Tests": {
                sh './run_api_tests.sh'
            }
        )
    }
}
```

---

### 2.3 try/catch/finally with scripted

```groovy
node {
    try {
        stage('Build') {
            sh 'mvn clean package'
        }
        stage('Test') {
            sh 'mvn test'
        }
    } catch (err) {
        echo "Build failed: ${err}"
        currentBuild.result = 'FAILURE'
        throw err
    } finally {
        echo 'Cleaning workspace...'
        deleteDir()
    }
}
```

---

### 2.4 Scripted environment

```groovy
node {
    env.APP_ENV = 'dev'
    echo "Env is ${env.APP_ENV}"
}
```

Environment variables via `env.VAR`.

---

# 🧱 3. Groovy Basics You Actually Need

You don’t need full Groovy, just these bits:

```groovy
def name = 'Manoj'
if (params.ENV == 'prod') {
    echo "Deploying to prod"
} else {
    echo "Deploying to ${params.ENV}"
}

def list = ['one', 'two', 'three']
list.each { item ->
    echo "Item: ${item}"
}

def map = [env: 'dev', version: '1.0']
echo map.env       // 'dev'
echo map['version']
```

Used mostly in `script { ... }` blocks inside Declarative.

---

# 🧱 4. Common Jenkinsfile Patterns (Cheat Sheet)

### 4.1 Maven build pattern

```groovy
stage('Build') {
    steps {
        sh 'mvn clean package -DskipTests'
        archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
    }
}
```

### 4.2 Node build pattern

```groovy
stage('Build UI') {
    steps {
        sh 'npm install'
        sh 'npm run build'
        archiveArtifacts artifacts: 'build/**/*', onlyIfSuccessful: true
    }
}
```

### 4.3 Python pattern

```groovy
stage('Build Python App') {
    steps {
        sh '''
          python3 -m venv venv
          . venv/bin/activate
          pip install -r requirements.txt
          zip -r app.zip .
        '''
        archiveArtifacts artifacts: 'app.zip'
    }
}
```

### 4.4 Basic deploy with SSH

```groovy
stage('Deploy') {
    steps {
        sshagent(['server-key']) {
            sh '''
              scp target/app.jar user@server:/opt/app/
              ssh user@server "systemctl restart app"
            '''
        }
    }
}
```

---

> `Cheat sheet` or `Interview questions`
