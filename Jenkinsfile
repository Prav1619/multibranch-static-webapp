pipeline {
    agent any

    environment {
        // UPDATE THESE THREE WITH YOUR ACTUAL PUBLIC EC2 IPs
        DEV_EC2  = "54.210.218.241"
        QA_EC2   = "98.81.77.187"
        PROD_EC2 = "18.208.142.185"
    }

    stages {
        
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Static Website') {
            steps {
                sh """
                rm -rf build
                mkdir build
                cp -r src/* build/
                """
            }
        }

        stage('Generate Config.js') {
            steps {
                script {
                    def envFile = ""

                    if (env.BRANCH_NAME == "dev")  envFile = "env/dev.env"
                    if (env.BRANCH_NAME == "qa")   envFile = "env/qa.env"
                    if (env.BRANCH_NAME == "main") envFile = "env/prod.env"

                    if (envFile != "") {
                        sh """
                        echo 'window.APP_CONFIG = {' > build/config.js
                        grep -v '^#' ${envFile} | sed "s/^/  /" | sed "s/=/: '/" | sed "s/\$/',/" >> build/config.js
                        echo '};' >> build/config.js
                        """
                    }
                }
            }
        }

        stage('Deploy to DEV') {
            when { branch 'dev' }
            steps {
                sh """
                scp -o StrictHostKeyChecking=no -r build/* ec2-user@${DEV_EC2}:/var/www/html/
                ssh -o StrictHostKeyChecking=no ec2-user@${DEV_EC2} 'sudo systemctl restart nginx'
                """
            }
        }

        stage('Deploy to QA') {
            when { branch 'qa' }
            steps {
                sh """
                scp -o StrictHostKeyChecking=no -r build/* ec2-user@${QA_EC2}:/var/www/html/
                ssh -o StrictHostKeyChecking=no ec2-user@${QA_EC2} 'sudo systemctl restart nginx'
                """
            }
        }

        stage('Approval for PROD') {
            when { branch 'main' }
            steps {
                script {
                    timeout(time: 10, unit: 'MINUTES') {
                        input message: "Approve PROD Deployment?"
                    }
                }
            }
        }

        stage('Deploy to PROD') {
            when { branch 'main' }
            steps {
                sh """
                scp -o StrictHostKeyChecking=no -r build/* ec2-user@${PROD_EC2}:/var/www/html/
                ssh -o StrictHostKeyChecking=no ec2-user@${PROD_EC2} 'sudo systemctl restart nginx'
                """
            }
        }

        stage('Feature Branch CI Only') {
            when { branch pattern: "feature/.*" }
            steps {
                echo "Feature branch → CI only (no deployment)"
            }
        }
    }

    post {
        always {
            echo "Pipeline finished for branch: ${env.BRANCH_NAME}"
        }
    }
}
