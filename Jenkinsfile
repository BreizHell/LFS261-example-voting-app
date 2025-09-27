pipeline {
    agent none

    stages{
        stage("Stages for the Vote service"){
            stages{
                stage('build'){
                    when {
                        changeset "**/vote/**"
                    }
                    agent{
                        docker{
                            image 'python:3.11-slim'
                            args '--user root'
                        }
                    }
                    steps{
                        echo 'Compiling vote app.' 
                        dir('vote'){
                            sh "pip install -r requirements.txt"
                        }
                    }
                }
                stage('test'){
                    when {
                        changeset "**/vote/**"
                    }
                    agent{
                        docker{
                            image 'python:3.11-slim'
                            args '--user root'
                        }
                    }
                    steps{
                        echo 'Running Unit Tests on vote app.' 
                        dir('vote'){
                            sh "pip install -r requirements.txt"
                            sh 'nosetests -v'
                        } 
                    }
                }

                stage('docker-image') {
                    agent any
                    when {
                        changeset "**/vote/**"
                    }
                    steps {
                        echo 'Building and pushing docker image'

                        script {
                            docker.withRegistry('https://index.docker.io/v1/', 'mydockerlogin') {
                                def img = docker.build(
                                    'breizhell/lfs-vote:latest',
                                    '--target dist ./vote'
                                )

                                img.push('latest')
                            }
                        }
                    }
                }
            }
        }

        stage("Stages for the Result service"){
            stages{
                stage("build") {
                    when {
                        changeset "**/result/**"
                    }
                    agent {
                        docker {
                            image 'node:22-alpine'
                        }
                    }
                    steps {
                        echo 'Compiling ...'
                        dir('result') {
                            sh 'npm install'
                        }
                    }
                }

                stage("test") {
                    when {
                        changeset "**/result/**"
                    }
                    agent {
                        docker {
                            image 'node:22-alpine'
                        }
                    }

                    steps {
                        echo 'Testing ...'
                        dir('result') {
                            sh 'npm install'
                            sh 'npm test'
                        }
                    }
                }

                stage('docker-image') {
                    agent any
                    when {
                        changeset "**/result/**"
                    }
                    steps {
                        echo 'Building and pushing docker image'

                        script {
                            docker.withRegistry('https://index.docker.io/v1/', 'mydockerlogin') {
                                def img = docker.build(
                                    'breizhell/lfs-result:latest',
                                    '--target dist ./result'
                                )

                                img.push('latest')
                            }
                        }
                    }
                }
  
            }
        }

        stage("Stages for the Worker service"){
            stages{
                stage("build"){
                    when {
                        changeset "**/worker/**"
                    }
                    agent {
                        docker {
                            image 'maven:3.9.8-sapmachine-21'
                            args '-v $HOME/.m2:/root/.m2'
                        }
                    }
                    steps{
                        echo 'Building worker app' 
                        dir('worker'){
                            sh 'mvn compile'
                        }
                    }
                }

                stage("test"){
                    when {
                        changeset "**/worker/**"
                    }
                    agent {
                        docker {
                            image 'maven:3.9.8-sapmachine-21'
                            args '-v $HOME/.m2:/root/.m2'
                        }
                    }
                    steps{
                        echo 'Running Unit Tests on worker app'
                        dir('worker') {
                            sh 'mvn clean test'
                        }
                    }
                }

                stage("package"){
                    when {
                        changeset "**/worker/**"
                    }
                    agent {
                        docker {
                            image 'maven:3.9.8-sapmachine-21'
                            args '-v $HOME/.m2:/root/.m2'
                        }
                    }

                    steps{
                        echo 'Packaging worker app into a .jar'
                        dir('worker') {
                            sh 'mvn package -DskipTests'
                            archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                        }
                    }
                }

                stage('docker-image') {
                    agent any
                    when {
                        changeset "**/worker/**"
                    }
                    steps {
                        echo 'Building and pushing docker image'
                        script {
                            docker.withRegistry('https://index.docker.io/v1/', 'mydockerlogin') {
                                def img = docker.build(
                                    'breizhell/lfs-worker:latest',
                                    './worker'
                                )
                                img.push('latest')
                            }
                        }
                    }
                }
            }
        }

        stage('Launching Sonarqube analysis') {
            agent any
            when {
                branch 'master'
            }
            environment {
                sonarpath = tool 'SonarScanner'
            }
            steps {
                withSonarQubeEnv('sonar-instavote') {
                    sh "${sonarpath}/bin/sonar-scanner -Dproject.settings=sonar-project.properties"
                }
            }
        }

        stage("Quality Gate") {
            agent any
            steps {
                withSonarQubeEnv('sonar-instavote') {
                    timeout(time: 2, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: true
                    }
                }
            }
        }

        stage('deploy') {
            agent any
            when {
                branch 'master'
            }
            steps {
                echo 'Deploying services with docker compose'
                sh 'docker compose up --help'
                sh 'docker compose up --detach'
            }
        }
    }

    post{
      always{
          echo 'This pipeline is completed.'
      }
    }
}