def gv

pipeline {
    agent any
    tools {
        maven 'maven-3.9'
    }

    stages {

        stage("increment version") {
            steps {
                script {
                    echo "incrementing app version..."
                    sh 'mvn build-helper:parse-version versions:set \
                        -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
                        versions:commit'
                    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
                    def version = matcher[0][1]
                    env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                }
            }
        }

        stage("build app") {
            steps {
                script {
                    echo "Building the application..."
                    sh 'mvn clean package'
                }
            }
        }

        stage("build image") {
            steps {
                script {
                    echo "building docker image"
                    withCredentials([usernamePassword(credentialsId:'docker-hub-repo', passwordVariable: 'PASS', usernameVariable:'USER')]) {
                        sh "docker build -t lsb22/demo-app:${IMAGE_NAME} ."
                        sh 'echo $PASS | docker login -u $USER --password-stdin'
                        sh "docker push lsb22/demo-app:${IMAGE_NAME}"
                    }
                }
            }
        }

        stage("deploy") {
            steps {
                script {
                    echo "Deploying the application"
                }
            }
        }

        stage("commit version update") {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId:'github-credentials', passwordVariable: 'PASS', usernameVariable:'USER')]) {
                        sh 'git config --global user.email "jenkinsuser22@gmail.com"'
                        sh 'git config --global user.name "jenkinsuser22"'
                        sh "git remote set-url origin https://${USER}:${PASS}@github.com/lsb22/java-maven-app.git"
                        sh 'git add .'
                        sh 'git commit -m "CI: version bump"'
                        sh 'git push origin HEAD:jenkins-jobs'
                    }
                }
            }
        }
    }
}