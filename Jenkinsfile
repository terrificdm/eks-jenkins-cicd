pipeline {
    agent {
        label 'jenkins-jenkins-agent'
    }
    stages {
        stage('Get Source') {
            steps {
                echo "1.Clone Repo Stage"
                git credentialsId: 'GitHubKey', url: 'git@github.com:terrificdm/poc-app-repo.git'
                script {
                    build_tag = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
                    repo_name = '091166060467.dkr.ecr.us-east-1.amazonaws.com'
                    app_name = 'poc-app-image'
                }
            }
        }
        stage('Build Image') {
            steps {
                echo "2.Build Docker Image Stage"
                sh "docker build --network host -t ${repo_name}/${app_name}:latest ."
                sh "docker tag ${repo_name}/${app_name}:latest ${repo_name}/${app_name}:${build_tag}"
            }

        }
        stage('Push Image') {
            steps {
                echo "3.Push Docker Image Stage"
                withDockerRegistry(credentialsId: 'ecr:us-east-1:AWSKey', url: 'https://091166060467.dkr.ecr.us-east-1.amazonaws.com/poc-app-image') {
                    sh "docker push ${repo_name}/${app_name}:latest"
                    sh "docker push ${repo_name}/${app_name}:${build_tag}"
                }
            }
        }
        stage('Deploy App') {
            steps {
                echo "4.Deploy Stage"
                sh "sed -i 's/<REPO_NAME>/${repo_name}/' deployfile/app.yaml"
                sh "sed -i 's/<APP_NAME>/${app_name}/' deployfile/app.yaml"
                sh "sed -i 's/<BUILD_TAG>/${build_tag}/' deployfile/app.yaml"
                sh "kubectl apply -f deployfile/app.yaml"
            }
        }
    }
}
