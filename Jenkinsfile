pipeline {
    agent any
    tools {nodejs "node"}
    stages {
        stage('increment version') {
            steps {
                script {
                    dir("app") {
                        npm version minor
                        def packageJson = readJSON file: 'package.json'
                        def version = packageJson.version
                        env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                    }
                }
            }
        }
        stage('Run tests') {
            steps {
               script {
                    dir("app") {
                        sh "npm install"
                        sh "npm run test"
                    } 
               }
            }
        }
        stage('Build and Push docker image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'USER', passwordVariable: 'PWD')]){
                    sh "docker build -t anie3142/my-node-app:${IMAGE_NAME} ."
                    sh "echo ${PWD} | docker login -u ${USER} --password-stdin"
                    sh "docker push anie3142/my-node-app:${IMAGE_NAME}"
                }
            }
        }
        stage('commit version update') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'github-credentials', usernameVariable: 'USER', passwordVariable: 'PWD')]) {
                        sh 'git config --global user.email "aniebiet.ccie@gmail.com"'
                        sh 'git config --global user.name "Anie3142"'

                        sh "git remote set-url origin https://${USER}:${PWD}@github.com/Anie3142/Jide-x-Anie-Repo.git"
                        sh 'git add .'
                        sh 'git commit -m "ci: version bump"'
                        sh 'git push origin HEAD:jenkins-jobs'
                    }
                }
            }
        }
    }
}