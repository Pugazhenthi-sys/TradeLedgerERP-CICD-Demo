# TradeLedgerERP - Jenkins - Docker - Kubernetes CI/CD Demo

Pipeline: GitHub -> Jenkins -> Maven -> Docker image -> Docker Hub -> Kubernetes (Docker Desktop)

## Project files
| File | Purpose |
|------|---------|
| App.java | Sample Java app (prints a message every 5 s) |
| pom.xml | Maven build -> target/simple-java-app-1.0.jar |
| Dockerfile | Packages the jar into a container image |
| deployment.yaml | Kubernetes Deployment (2 replicas) + NodePort Service (30008) |
| Jenkinsfile | The CI/CD pipeline |

## 1. Prerequisites (Windows machine)
- JDK 17, Maven, Git (all on PATH)
- Docker Desktop with Kubernetes enabled (Settings > Kubernetes > Enable > Apply & Restart)
- Jenkins running on the same machine
- A GitHub account (public repo) and a Docker Hub account

Check: `java -version`, `mvn -v`, `git --version`, `docker version`, `kubectl get nodes`

## 2. Replace placeholders
- Already set: image `pugazhmsp/rice-traders-app:latest`, repo `Pugazhenthi-sys/TradeLedgerERP-CICD-Demo`

## 3. Test locally
    mvn clean package
    docker build -t pugazhmsp/rice-traders-app:latest .
    docker run --rm pugazhmsp/rice-traders-app:latest

## 4. Push to GitHub
Repo: https://github.com/Pugazhenthi-sys/TradeLedgerERP-CICD-Demo (public), then:

    git init
    git add .
    git commit -m "Initial Java CI/CD commit"
    git branch -M main
    git remote add origin https://github.com/Pugazhenthi-sys/TradeLedgerERP-CICD-Demo.git
    git push -u origin main

## 5. Configure Jenkins
1. Manage Jenkins > Plugins > Available plugins. Install: Docker, Docker Pipeline, Kubernetes CLI. Restart Jenkins if asked.
2. Manage Jenkins > Credentials > System > Global credentials (unrestricted) > Add Credentials:
   - Kind: Username with password
   - Scope: Global
   - Username: pugazhmsp
   - Password: your Docker Hub password (or access token - recommended)
   - ID: `dockerhub-creds`
   - Description: DockerHub Login
3. Dashboard > New Item > name `Java-CICD` > Pipeline > OK.
4. Pipeline section > Definition: **Pipeline script from SCM**
   - SCM: Git
   - Repository URL: https://github.com/Pugazhenthi-sys/TradeLedgerERP-CICD-Demo.git
   - Branch: `*/main`
   - Script Path: `Jenkinsfile`
   (Alternative: choose "Pipeline script" and paste the contents of Jenkinsfile.)
5. Save > Build Now. Watch Console Output until `Finished: SUCCESS`.

## 6. Verify
    docker images
    kubectl get deployments
    kubectl get pods
    kubectl get svc
    kubectl logs -l app=java-app --tail=5

Expected logs: "Hello from Java CI/CD Pipeline!"

## Troubleshooting
- `docker`/`kubectl`/`mvn` not recognized in Jenkins: Jenkins service has a different PATH. Add the tools to the SYSTEM PATH and restart the Jenkins service.
- kubectl cannot connect from Jenkins: if Jenkins runs as a Windows service, it uses a different user profile and cannot see your `%USERPROFILE%\.kube\config`. Either run Jenkins as your user (`java -jar jenkins.war`) or set the service to log on as your account.
- `no main manifest attribute`: pom.xml must include the maven-jar-plugin with mainClass (already included here).
- ImagePullBackOff: the image name in deployment.yaml doesn't match what was pushed, or the Docker Hub repo is private.
- Docker push denied: wrong credentials, or the image name doesn't start with your Docker Hub username.
