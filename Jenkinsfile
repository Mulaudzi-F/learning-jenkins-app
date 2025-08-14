pipeline {
    agent any

    environment{

            NETLIFY_SITE_ID ='8e22def3-c517-446a-80a0-fd5ade55b2da'
            NETLIFY_AUTH_TOKEN = credentials('netlify-token')
           
    }

    stages {

        
        stage('Build') {
            agent{
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "Small change"
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
        

        stage('Run Tests'){

         parallel {

                
            stage(' unit Test') {

                agent{
                    docker{
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

        stage('stage test') {

            agent{
                docker{
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true 
                   
                }
            }
            steps {
                sh '''
                npm i serve
                node_modules/.bin/serve -s build &
                sleep 10
                npx playwright test --reporter=html
                '''
            } 

            post {
        always {
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'playwright local HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        }
     }
        }
            }
        }


        stage('Deploy staging') {
            agent{
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli@20.1.1 node-jq
                    node_modules/.bin/netlify --version
                    echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status 
                    node_modules/.bin/netlify deploy --dir=build
                    node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
                    
                '''
                script{

                env.STAGING_URL = sh(script: "node_modules/.bin/node-jq -r 'deploy_url' deploy-output.json", returnStout:true) 
            }
            } 

            
        } 

         stage('staging E2E') {

            agent{
                docker{
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true 
                   
                }
            }

            environment{
                CI_ENVIRONMENT_URL= "${env.STAGING_URL}"
                 } 

            steps {
                sh '''
                npx playwright test --reporter=html
                '''
            } 

            post {
            always {
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'playwright staging E2E HTML Report', reportTitles: '', useWrapperFileDirectly: true])
            }
         }
        }

        stage('Approval'){
            steps{
                input 'Do you wish to deploy to production'
            }
        }
         stage('Deploy to production') {
            agent{
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status 
                    node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }


    

     stage('Prod E2E') {

            agent{
                docker{
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true 
                   
                }
            }

            environment{
                CI_ENVIRONMENT_URL= 'celebrated-strudel-db4eb7.netlify.app'
                 } 

            steps {
                sh '''
                npx playwright test --reporter=html
                '''
            } 

            post {
            always {
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'playwright Prod E2E HTML Report', reportTitles: '', useWrapperFileDirectly: true])
            }
         }
        }

    
    }

}
