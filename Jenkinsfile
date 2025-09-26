pipeline {
    agent any

    environment {
        // NETLIFY_SITE_ID = '8e22def3-c517-446a-80a0-fd5ade55b2da'
        // NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        AWS_DEFAULT_REGION = 'us-east-1'
    }

    stages { 

         stage('Deploy to AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    reuseNode true
                    args '-u root  --entrypoint=""'
                }
            }

            

            steps {
                withCredentials([usernamePassword(credentialsId: 'my aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        yum install jq -y
                        LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq ".taskDefinition.revision")
                        echo "Latest task definition revision is $LATEST_TD_REVISION"
                        aws ecs update-service --cluster Learn-JenkinsApp-clusture-prod  --service LearnJenkinsApp-TaskDefinition-Prod-service-z3zelp5n --task-definition LearnJenkinsApp-TaskDefinition-Prod:${LATEST_TD_REVISION}
                    '''
                }
            }
        }


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

       
        // stage('Tests') {
        //     parallel {
        //         stage('Unit tests') {
        //             agent {
        //                 docker {
        //                     image 'node:18-alpine'
        //                     reuseNode true
        //                 }
        //             }
        //             steps {
        //                 sh '''
        //                     npm test
        //                 '''
        //             }
        //             post {
        //                 always {
        //                     junit 'jest-results/junit.xml'
        //                 }
        //             }
        //         }

        //         stage('E2E') {
        //             agent {
        //                 docker {
        //                     image 'my-playwright'
        //                     reuseNode true
        //                 }
        //             }
        //             steps {
        //                 sh '''
        //                     serve -s build &
        //                     sleep 10
        //                     npx playwright test --reporter=html
        //                 '''
        //             }
        //             post {
        //                 always {
        //                     publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Local E2E'])
        //                 }
        //             }
        //         }
        //     }
        // }

        // stage('Deploy staging') {
        //     agent {
        //         docker {
        //             image 'my-playwright'
        //             reuseNode true
        //         }
        //     }
        //     environment {
        //         CI_ENVIRONMENT_URL = 'STAGING_URL_TO_BE_SET'
        //     }
        //     steps {
        //         sh '''
        //             netlify --version
        //             echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
        //             netlify status
        //             netlify deploy --dir=build --json > deploy-output.json
        //             CI_ENVIRONMENT_URL=$(jq -r '.deploy_url' deploy-output.json)
        //             npx playwright test --reporter=html
        //         '''
        //     }
        //     post {
        //         always {
        //             publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E'])
        //         }
        //     }
        // }

        // stage('Deploy prod') {
        //     agent {
        //         docker {
        //             image 'my-playwright'
        //             reuseNode true
        //         }
        //     }
        //     environment {
        //         CI_ENVIRONMENT_URL = 'YOUR NETLIFY SITE URL'
        //     }
        //     steps {
        //         sh '''
        //             node --version
        //             netlify --version
        //             echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
        //             netlify status
        //             netlify deploy --dir=build --prod
        //             npx playwright test --reporter=html
        //         '''
        //     }
        //     post {
        //         always {
        //             publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod E2E'])
        //         }
        //     }
        // }
    }
}

