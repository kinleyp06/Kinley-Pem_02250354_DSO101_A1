pipeline {
    agent any

    tools {
        nodejs 'NodeJS'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                url: 'https://github.com/kinleyp06/todo-app.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                dir('backend') {
                    bat 'npm install'
                }
            }
        }

        stage('Run Tests') {
            steps {
                dir('backend') {
                    bat 'npm test'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t kinleyp06/be-todo:latest ./backend'
            }
        }

        stage('Push Docker Image') {
            steps {
                bat 'docker push kinleyp06/be-todo:latest'
            }
        }
    }
}