pipeline {
    // This tells Jenkins to run the pipeline on any available agent (your local host).
    agent any 

    stages {
        stage('Build') {
             // Run this stage inside a Docker container
            agent {
                docker {
                    image 'node:25-alpine'
                    // Run container as root so NPM install never fails
                    // This prevents permission problems inside local Jenkins
                    args '-u root'
                    reuseNode true
                }
            }
            steps {
                sh'''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''  
            }
        }
        stage('Test'){
            agent {
                docker {
                    image 'node:25-alpine'
                    // Run container as root so NPM install/ci never fails
                    // This prevents permission problems inside local Jenkins
                    args '-u root'
                    reuseNode true
                }
            }
            steps{
                sh'''
                    echo "test stage"
                    test -f build/index.html
                    npm test
                '''

            }
        }
        stage('E2E'){
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.56.1-noble'
                    args '-u root'
                    // Run container as root so NPM install/ci never fails
                    // This prevents permission problems inside local Jenkins
                    reuseNode true
                }
            }
            steps{
                sh'''
                    # Playwright Docker images already come with browser dependencies preinstalled
                    # You only need to install the browser binaries (which npx playwright install does)
                    npx playwright install  # --with-deps

                    npm install serve
                    node_modules/.bin/serve -s build &
                    sleep 10
                    npx playwright test
                '''

            }

        }
    }
    post{
        always {
            junit 'jest-results/junit.xml'
        }
    }

}
