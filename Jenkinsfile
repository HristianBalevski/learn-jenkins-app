pipeline {
    agent {
        docker {
            image 'node:22-alpine'
            args '--user=root -v $WORKSPACE:/var/jenkins_home/workspace/learn-jenkins-app'
        }
    }

    environment {
        CI = 'true'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Setup Node.js') {
            steps {
                sh 'node --version'
                sh 'npm --version'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm ci'
                sh 'npm install --save-dev @babel/plugin-proposal-private-property-in-object'
                sh 'npx playwright install --with-deps'
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'npm test -- --runInBand --detectOpenHandles'
            }
        }

        stage('E2E Tests') {
            steps {
                withDockerContainer(image: 'mcr.microsoft.com/playwright:v1.39.0-jammy') {
                    sh 'npx playwright test --reporter=html'
                }
            }
        }

        stage('Publish Test Reports') {
            steps {
                junit 'test-results/junit.xml'
                publishHTML(target: [
                    reportDir: 'playwright-report',
                    reportFiles: 'index.html',
                    reportName: 'Playwright Test Report'
                ])
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'build/**', fingerprint: true
            cleanWs()
        }
    }
}
