pipeline {
    agent any
    tools {nodejs "node"}
    stage('GIT Login') {
            steps {
                git branch: 'main', credentialsId: 'github-credentials', url: 'https://github.com/Anie3142/Jide-x-Anie-Repo.git'
                }
            }
        }
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
                    sh "docker build -t anie3142/our-node-app:${IMAGE_NAME} ."
                    sh "echo ${PWD} | docker login -u ${USER} --password-stdin"
                    sh "docker push anie3142/our-node-app:${IMAGE_NAME}"
                }
            }
        }
        stage('deploy to EC2') {
            steps {
                script {
                   def shellCmd = "bash ./server-cmds.sh ${IMAGE_NAME}"
                   def ec2Instance = "ec2-user@52.91.90.72"

                   sshagent(['ec2-server-key']) {
                       sh "scp -o StrictHostKeyChecking=no server-cmds.sh ${ec2Instance}:/home/ec2-user"
                       sh "scp -o StrictHostKeyChecking=no docker-compose.yaml ${ec2Instance}:/home/ec2-user"
                       sh "ssh -o StrictHostKeyChecking=no ${ec2Instance} ${shellCmd}"
                   }     
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