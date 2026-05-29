pipeline {

agent any

tools {
    maven 'maven3'
    jdk 'jdk21'
}

environment {

    AWS_ACCOUNT_ID = '811345154438'
    AWS_REGION = 'us-east-1'

    ECR_REPO = 'project-1'

    IMAGE_TAG = "${BUILD_NUMBER}"

    IMAGE_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}"
}

stages {

    stage('Checkout Code') {

        steps {

            git branch: 'main',
            credentialsId: 'github-token',
            url: 'https://github.com/vinaykumar485/project-1.git'
        }
    }

    stage('Build Maven') {

        steps {

            sh 'mvn clean package'
        }
    }

    stage('Run Tests') {

        steps {

            sh 'mvn test'
        }
    }

    stage('Build Docker Image') {

        steps {

            sh """
            docker build -t ${IMAGE_URI} .
            """
        }
    }

    stage('Login To AWS ECR') {

        steps {

            sh """
            aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
            """
        }
    }

    stage('Push Docker Image') {

        steps {

            sh """
            docker push ${IMAGE_URI}
            """
        }
    }

    stage('Update deployment.yaml') {

        steps {

            sh """
            sed -i 's|image:.*|image: ${IMAGE_URI}|g' k8s/deployment.yaml
            """
        }
    }

    stage('Push Updated YAML To GitHub') {

        steps {

            withCredentials([usernamePassword(
                credentialsId: 'github-token',
                usernameVariable: 'GIT_USERNAME',
                passwordVariable: 'GIT_PASSWORD'
            )]) {

                sh """
                git config user.email "jenkins@example.com"

                git config user.name "jenkins"

                git add k8s/deployment.yaml

                git commit -m "Updated image to ${IMAGE_TAG}"

                git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/vinaykumar485/project-1.git HEAD:main
                """
            }
        }
    }
}

post {

    success {

        echo 'Pipeline executed successfully'
    }

    failure {

        echo 'Pipeline failed'
    }
}

}

