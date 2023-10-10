pipeline {
    agent any
    environment {
        S3_BUCKET = 'sathis.backup'
        S3_PATH = 'backup'
    }
    
    stages {
        stage('Build and Run Containers') {
            steps {
                script {
                    // Build and run containers
                    sh 'docker run -itd --name sathis -p "8088:80" httpd'
                    sh 'docker run -itd --name web ubuntu'
                }
            }
        }
        stage('Save Images and Backup to S3') {
            steps {
                script {
                    DOCKER_IMAGES = sh(script: 'docker image ls --format "{{.Repository}}"', returnStdout: true).trim()
                    def imagesArray = DOCKER_IMAGES.split('\n')

                    imagesArray.each { image ->
                            sh "docker save -o ${image}.tar ${image}"
                            sh "aws s3 cp ${image}.tar s3://${S3_BUCKET}/${S3_PATH}/${image}.tar"
                            sh "rm ${image}.tar"
                            sh "docker rmi -f ${image}"
                    }
                }
            }
        }
        stage('Remove Files Older Than 7 Days') {
            steps {
                script {
                    // Remove files older than 7 days using find command
                    sh 'find /var/lib/jenkins -type f -mtime +7 -user root -exec rm {} +'
                    echo "7 days older files are deleted"
                }
            }
        }
        stage('Remove Files more than 30Mb') {
            steps {
                script {
                    // Remove files more than 50Mb using find command
                    sh 'find /var/lib/jenkins -type f -size +30M -exec rm {} +'
                    echo "more than 30Mb files are deleted"
                    }
            }
        }
    }
}
