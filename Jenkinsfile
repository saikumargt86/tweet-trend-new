pipeline {
    agent {
        node{
            label 'maven-server'
        }
    }
enviroment {
    PATH = "/opt/apache-maven-3.9.5/bin:$PATH"
}
    stages {
        stage('build') {
            steps {
                sh 'mvn clean deploy'
            }
        }
    }
}
