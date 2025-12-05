pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        // Check and install cqfd if not present
        stage('Install cqfd') {
            steps {
                sh '''
                    if ! command -v cqfd &> /dev/null; then
                        git clone https://github.com/savoirfairelinux/cqfd.git /tmp/cqfd
                        cd /tmp/cqfd
                        sudo make install
                    fi
                '''
            }
        }

        stage('Initialize cqfd') {
            steps {
                sh 'cqfd init'
            }
        }

        stage('Run vulnscout_ci') {
            steps {
                script {
                    def result = sh(script: 'cqfd -b vulnscout_ci', returnStatus: true)
                    if (result != 0) {
                        echo "vulnscout_ci detected CVEs (exit code: ${result})"
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
        }

        stage('Extract CVEs') {
            steps {
                sh '''
                    grep -Eo 'CVE-[0-9]{4}-[0-9]+' || true
                '''
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        failure {
            echo 'Build failed!'
        }
        unstable {
            echo 'CVEs detected - build marked as unstable'
        }
    }
}
