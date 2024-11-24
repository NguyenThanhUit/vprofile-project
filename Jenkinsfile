pipeline {
    agent any
    tools {
        maven "MAVEN"
        jdk "OracleJDK8"
    }
    environment {
        KUBECONFIG = "/var/lib/jenkins/.kube/config" 
        registry = "nguyenthanh91ndu/test"
        registryCredential = 'dockerhub-credentials'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'cicd-kube', url: 'https://github.com/NguyenThanhUit/vprofile-project.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${registry}:${BUILD_NUMBER}")
                }
            }
        }

        stage('Deploy Docker Image') {
            steps {
                script {
                    docker.withRegistry('', registryCredential) {
                        dockerImage.push("${BUILD_NUMBER}")
                        dockerImage.push('latest')
                    }
                }
            }
        }

        stage('Remove Unused Docker Image') {
            steps {
                sh "docker rmi ${registry}:${BUILD_NUMBER}"
            }
        }

        stage('Build & SonarQube Analysis') {
            environment {
                scannerHome = tool 'sonar'
            }
            steps {
                withSonarQubeEnv('My SonarQube Server') {
                    sh 'mvn clean package sonar:sonar'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                withSonarQubeEnv('My SonarQube Server') {
                    sh '''${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=vprofile \
                        -Dsonar.projectName=vprofile-repo \
                        -Dsonar.projectVersion=1.0 \
                        -Dsonar.sources=src/ \
                        -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                        -Dsonar.junit.reportsPath=target/surefire-reports/ \
                        -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                        -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }

                timeout(time: 4, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false
                }
            }
        }

        stage('Setup Kubernetes Config') {
            steps {
                script {
                    sh '''
                        sudo mkdir -p /var/lib/jenkins/.kube
                        sudo cp /home/nguyenthanh/.kube/config /var/lib/jenkins/.kube/config
                        sudo chown -R jenkins:jenkins /var/lib/jenkins/.kube
                    '''
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                script {
                    sh "kubectl apply -f /home/nguyenthanh/kube/vprofile-project-cicd-kube/kubernetes/db/vprodb.yml"
                    sh "kubectl apply -f /home/nguyenthanh/kube/vprofile-project-cicd-kube/kubernetes/db/vprodbdep.yml"
                    sh "kubectl apply -f /home/nguyenthanh/kube/vprofile-project-cicd-kube/kubernetes/memcache/mc.yml"
                    sh "kubectl apply -f /home/nguyenthanh/kube/vprofile-project-cicd-kube/kubernetes/memcache/mcdep.yml"
                    sh "kubectl apply -f /home/nguyenthanh/kube/vprofile-project-cicd-kube/kubernetes/vpro-app/vproapp-service.yml"
                    sh "kubectl apply -f /home/nguyenthanh/kube/vprofile-project-cicd-kube/kubernetes/vpro-app/vproappdep.yml"
                    sh "kubectl apply -f /home/nguyenthanh/kube/vprofile-project-cicd-kube/kubernetes/tomapp/vproapp-nodeport-service.yml"
                    sh "kubectl apply -f /home/nguyenthanh/kube/vprofile-project-cicd-kube/kubernetes/tomapp/vproapp.yml"
                    sh "kubectl apply -f /home/nguyenthanh/kube/vprofile-project-cicd-kube/kubernetes/tomapp/vproappdep.yml"
                    sh "kubectl apply -f /home/nguyenthanh/kube/vprofile-project-cicd-kube/kubernetes/app.yml"
                }
            }
        }
    }
}
