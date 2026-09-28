# Jenkins with Build Tool (Maven) — Virtual Lab Experiment 6

A minimal Java project that Jenkins builds automatically with Maven whenever code is pushed to GitHub.

```
jenkins-lab/
├── src/
│   └── HelloWorld.java
├── pom.xml
├── Jenkinsfile        (optional: same build as a pipeline job)
└── .gitignore
```

## 1. Install tools (Ubuntu)

```bash
sudo apt update
sudo apt install openjdk-17-jdk maven git -y
java -version && mvn -version
```

Jenkins is not in Ubuntu's default repos, so add the Jenkins repo first:

```bash
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update && sudo apt install jenkins -y
sudo systemctl enable --now jenkins
sudo systemctl status jenkins
```

> Current Jenkins releases need Java 17 or 21 — Java 11 from the lab sheet will not start Jenkins.

Open `http://localhost:8080`, unlock with
`sudo cat /var/lib/jenkins/secrets/initialAdminPassword`, and install suggested plugins.

## 2. Build locally first

```bash
mvn clean package
java -jar target/jenkins-lab-1.0.jar
# Hello from Jenkins CI Pipeline!
```

## 3. Push to GitHub

Create an empty repo named `jenkins-lab` on GitHub, then:

```bash
git init
git add .
git commit -m "Initial Jenkins Maven Project"
git branch -M main
git remote add origin https://github.com/<username>/jenkins-lab.git
git push -u origin main
```

## 4. Configure the Jenkins job

1. **Manage Jenkins → Tools → Maven installations** → Add Maven, name it `Maven`, tick *Install automatically* (or point MAVEN_HOME to `/usr/share/maven`).
2. **New Item** → name `jenkins-lab` → **Freestyle project** → OK.
3. **Source Code Management** → Git → `https://github.com/<username>/jenkins-lab.git`, branch `*/main`.
4. **Build Triggers** (this is what makes it *automatic*):
   - **Poll SCM** with schedule `H/2 * * * *` — Jenkins checks GitHub every ~2 minutes. Works on localhost.
   - or **GitHub hook trigger for GITScm polling** — needs a GitHub webhook to `http://<public-jenkins-url>/github-webhook/`; localhost is not reachable from GitHub, so use ngrok or Poll SCM.
5. **Build Steps** → *Invoke top-level Maven targets* → Maven version `Maven`, Goals `clean package`.
6. Save → **Build Now** → open the build → **Console Output**. Expected: `BUILD SUCCESS`.

To prove auto-triggering: edit the message in `HelloWorld.java`, commit and push — a new build starts on its own within the polling interval.

## Pipeline job (optional)

New Item → **Pipeline** → *Pipeline script from SCM* → Git → same URL → Script Path `Jenkinsfile`.

## Note on pom.xml

The lab sheet's pom has no Java version set, so `maven-compiler-plugin 3.8.1` defaults to Java 1.6, which JDK 11+ rejects ("Source option 6 is no longer supported"). This pom sets `maven.compiler.source/target` to 11 so the build succeeds, and adds a jar manifest so the jar is runnable.
# vle-6-jenkins
