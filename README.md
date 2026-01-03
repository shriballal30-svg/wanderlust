
---

# **Wanderlust Project Setup Guide**

## **1. Security Group Configuration**
Open these ports in your security group:
- **443** (HTTPS)
- **5173** (Vite/Dev Server)
- **80** (HTTP)
- **8080** (Jenkins)
- **9000** (SonarQube)
- **5000** (Registry/API)
- **22** (SSH)

**Note:** All we do this project in single server

---

## **2. System Update & Docker Installation**
```
apt update
apt install docker.io -y
apt install docker-compose -y
sudo usermod -aG docker $USER
```

---

## **3. Install Jenkins Setup**
```
Install Jenkins setup it
```

---

## **4. SonarQube in Docker Container Setup**
```
docker run -itd --name sonar-server -p 9000:9000 sonarqube:lts-community
```

---

## **5. Install Trivy**
```
Install trivy in using google command in EC2
It used to see the security of docker image, check different different command from in the google for trivy
eg: trivy image redis   # It scan this image
    trivy image nginx
```

---

## **6. Install Plugins in Jenkins**
```
1.SonarQube Scanner
2.Sonar Quality Gates
3.OWASP Dependency-check
4.Docker
5.Pipeline: Stage View
```

---

## **7. Integration Setup**

### **7.1 SonarQube to Jenkins**
```
go to → SonarQube → Administration → Configuration → Webhooks → create → name (any) → Jenkins_URL/sonarqube-webhook [eg (https://1.1.1.1:8080/sonarqube-webhook)] → create
                                                                                       
```

### **7.2 Jenkins to SonarQube**
```
First create token in SonarQube to connect Jenkins.

Go to SonarQube → Security → User → Administration-Token-three_dot_line_click → name token → expires → generate 


Jenkins → Manage Jenkins → System → Scroll & Add SonarQube → Name [any (eg. sonar)] → URL of SonarQube [eg. https://1.1.1.1:9000/] → add/create server authentication token → Jenkins click →[in Kind]secret text → paste sonar token in here [secret] → ID [any] → Add → now add server authentication token in there → save 
```

---

## **8. SonarQube Quality Gate Setup**
```
Jenkins → Manage Jenkins → Tools → Scroll & Add SonarQube scanner installation → name [any (eg. sonar)] → select version → save 
```

---

## **9. OWASP Dependency-Check Tool Setup**
```
Jenkins → Manage Jenkins → Tools → Scroll & Add Dependency-Check installation   → name [IMP (any)(eg. dc) cause we mention this name in our pipeline] → ☑️ install automatically → add installer → install from GitHub.com → Select any version → save      
```

---

## **10. Pipeline Configuration**
```
Jenkins → New Item → Name → Freestyle project → pull your git hub project URL → git hub webhook → paise your pipeline → save 


All setup done now we do pipeline
```

---

## **11. Pipeline Code**
```
pipeline {
    agent any
    environment {
        SONAR_HOME= tool "sonar"
    }
    stages {
        stage('code clone from github') {
            steps {
               git branch: 'main', url: 'https://github.com/shriballal30-svg/wanderlust.git'
            }
        }
         stage('sonarqube quality analysis') {
                steps {
                    withSonarQubeEnv("sonar"){
                        sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=wonderlust -Dsonar.projectKey=wonderlust"
                    }
                }
            }
         stage('OWAS dependency check ') {
                steps {
                  dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'dc'
                  dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                }
            }
          stage('sonar quality gate scan') {
                steps {
                    timeout(time: 4, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false
                    }
                }
            }
         stage('trivey systerm file scan') {
                steps {
                    sh 'trivy fs --format table -o trivy-fs-report.html .'
                }
            }
         stage('deploy usiing docker compose ') {
                steps {
                    sh "docker-compose up -d"
                }
            }
    }
}
```

---

## **12. Additional Setup Commands**
```
sudo apt-get update
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs
sudo apt-get install -y npm
node -v
npm -v

sudo systemctl enable docker
sudo systemctl start docker
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
sudo systemctl status docker
sudo usermod -aG docker $USER/ubuntu  #imp
sudo usermod -aG jenkins $USER/ubuntu
sudo usermod -aG docker jenkins     # imp
sudo systemctl restart jenkins
sudo systemctl reboot
```

---


