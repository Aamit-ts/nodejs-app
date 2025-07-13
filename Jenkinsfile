pipeline {
    agent any

    tools {
        nodejs 'NodeJS-20'
    }

    environment {
        REMOTE_HOST = '16.16.218.78'
        REMOTE_USER = 'ec2-user'
        REMOTE_PATH = '/home/ec2-user/nodejs-app'
        SSH_CREDENTIALS = 'NodeServerSSHKey'
    }

    stages {

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Transfer to Remote Server') {
            steps {
                sshagent([env.SSH_CREDENTIALS]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no $REMOTE_USER@$REMOTE_HOST "mkdir -p $REMOTE_PATH"
                        rsync -avz --exclude=node_modules --exclude=.git ./ $REMOTE_USER@$REMOTE_HOST:$REMOTE_PATH/
                    """
                }
            }
        }

        stage('Install & Deploy using Local PM2') {
            steps {
                sshagent([env.SSH_CREDENTIALS]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no $REMOTE_USER@$REMOTE_HOST "
                            cd $REMOTE_PATH &&
                            npm install &&
                            npx pm2 start app.js --name my-app --update-env || npx pm2 restart my-app
                        "
                    """
                }
            }
        }
    }
}
