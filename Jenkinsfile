def publish_flag = false;
pipeline {
    agent any
    environment{
        DOCKER_HUB_USER = credentials('DOCKER_HUB_USER')
        DOCKER_HUB_PWD = credentials('DOCKER_HUB_PWD')
        
        SSH_USER = credentials('SSH_USER')
        SSH_HOST = credentials('SSH_HOST')
    }

    stages {
        stage('get_tag') {
            steps {
                script {
                    git 'https://github.com/DiemTrinh1513636/nest_jenkin'
                    env.IMAGE_VERSION = sh(returnStdout: true, script: "git tag --contains | head -1").trim()
                    env.IMAGE_NAME =  "$DOCKER_HUB_USER/nest_jenkin:$IMAGE_VERSION"
                }
            }
        }
        stage('publish') {
            when {
                allOf {
                    branch 'master'
                    expression {
                        env.IMAGE_VERSION != ""
                    }
                }
            }
            steps {
                echo "Publish image: build image and push to docker hub"
                git 'https://github.com/DiemTrinh1513636/nest_jenkin'
                sh "docker build -t $IMAGE_NAME ."
                sh "docker login -u $DOCKER_HUB_USER -p $DOCKER_HUB_PWD"
                sh "docker push $IMAGE_NAME"
                script { publish_flag = true }
            }
        }
        stage('deploy') {
            when {
                allOf {
                    branch 'master'
                    expression {
                        publish_flag == true
                    }
                }
            }
            steps {
                echo "Deploy: ssh to server, pull image and run container"
                sshagent(credentials:['jenkin_user']){
                    sh "ssh $SSH_USER@$SSH_HOST 'docker images && docker pull $IMAGE_NAME && docker stop nest_jenkin_app || true && docker rm nest_jenkin_app || true && docker run --name nest_jenkin_app -p 3003:3000 -d $IMAGE_NAME'"
                }
            }
        }
    }
}
