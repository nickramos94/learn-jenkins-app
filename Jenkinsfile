pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '44884f8c-33a7-4a72-8d43-5944f3882bc2'
        NETLIFY_AUTH_TOKEN = credentials('netlify_token')
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci 
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Run tests'){
            parallel {
                stage('unit tests'){
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true                    
                        }
                    }
                    steps {
                        sh '''
                            test -f build/index.html
                            npm test
                        '''
                    }

                    post {
                        always {
                            junit 'jest-results/junit.xml'

                        }
                    }
                }

                stage('E2E'){ // Tests that are run against a locally built website
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.48.1-noble'
                            reuseNode true                    
                        }
                    }
                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local Report', reportTitles: '', useWrapperFileDirectly: true])                        
                        }
                        
                    }
                }
                stage('Prod E2E'){ //Tests that are run against the production environment on Netify
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.48.1-noble'
                            reuseNode true                    
                        }
                    }

                    steps {
                        sh '''
                            npx playwright test --reporter=html
                        '''
                    }

                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E', reportTitles: '', useWrapperFileDirectly: true])                        
                        }
                        
                    }

                    environment {
                        CI_ENVIRONMENT_URL = 'https://relaxed-beijinho-372ccb.netlify.app' // This variable tells plyawright where to perform the tests on
                    }
                }
            }
        }

        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Deploying to production site id: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status 
                    node_modules/.bin/netlify deploy --dir=build --prod 
                '''
            }
        }    
    }
}
