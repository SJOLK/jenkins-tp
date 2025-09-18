pipeline {
    agent any 
    stages {
        stage('Hello') { 
            steps {
               echo 'Hello World'
            }
        }
        stage('Build') { 
            steps {
                sh ./build.sh
            }
        }
    }
}
