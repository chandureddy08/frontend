pipeline {
    agent {
        label 'AGENT-1'
    }
    options{
        timeout(time: 10, unit: 'MINUTES')
        disableConcurrentBuilds()
        ansiColor('xterm')
    }
    environment {
        appVersion = '' // variable declaration
        nexusUrl = 'nexus.chandureddy.online:8081'
        region = 'us-east-1'
        account_id = '339713021737'
    }
    stages {
        stage('Read the version') {
            steps {
                script {
                    def packageJson = readJSON file: 'package.json'
                    appVersion = packageJson.version
                    echo "Application version: ${appVersion}"
                }
            }
        }
        stage('Docker build') {
            steps {
                script {
                    sh """
                        aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${account_id}.dkr.ecr.${region}.amazonaws.com

                        docker build -t ${account_id}.dkr.ecr.${region}.amazonaws.com/expense-frontend:${appVersion} .

                        docker push ${account_id}.dkr.ecr.${region}.amazonaws.com/expense-frontend:${appVersion}
                    """
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    sh """
                        aws eks update-kubeconfig --region ${region} --name expense-dev
                        cd helm
                        sed -i 's/IMAGE_VERSION/${appVersion}/g' values.yaml

                        if helm ls --all --short | grep -q frontend; then
                            helm upgrade frontend .
                        else
                            helm install frontend .
                        fi
                    """
                }
            }
        }
        // Uncomment these stages if needed
        /*
        stage('Build') {
            steps {
                sh """
                    zip -q -r frontend-${appVersion}.zip * -x Jenkinsfile -x frontend-${appVersion}.zip
                    ls -ltr
                """
            }
        }
        stage('Nexus artifact upload') {
            steps {
                script {
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: "${nexusUrl}",
                        groupId: 'com.expense',
                        version: "${appVersion}",
                        repository: "frontend",
                        credentialsId: 'nexus-auth',
                        artifacts: [
                            [
                                artifactId: "frontend",
                                classifier: '',
                                file: "frontend-" + "${appVersion}" + '.zip',
                                type: 'zip'
                            ]
                        ]
                    )
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    def params = [
                        string(name: 'appVersion', value: "${appVersion}")
                    ]
                    build job: 'frontend-deploy', parameters: params, wait: false
                }
            }
        }
        */
    }
    post { 
        always { 
            echo 'I will always say Hello again!'
            deleteDir()
        }
        success { 
            echo 'I will run when the pipeline is successful'
        }
        failure { 
            echo 'I will run when the pipeline fails'
        }
    }
}
