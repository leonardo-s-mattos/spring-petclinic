#!/bin/env groovy

pipeline {
    agent any
    environment {
        GITHUB_CREDENTIALS = credentials('LEO_GITHUB_CREDENTIALS')
        IMAGE = "liatrio/petclinic-tomcat"
    }


    stages {
        stage('Build') {
            steps {
                echo 'Building......'
                checkout scm
                sh 'mvn -B -DskipTests clean install'
            }
        }

        stage('Quality & Security Control') {
            failFast true
            parallel {
                stage('Unit Test') {
                    steps {
                        echo 'Unit Testing.....'

                        sh 'mvn test'
                    }

                    post {
                        always {
                            junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml'
                        }
                    }
                }
                stage('Test coverage validation') {
                    steps {
                        sh "mvn verify cobertura:cobertura -amd -T 1.5C"
                        //cobertura()
                    }

                }
                stage('Quality Scan') {
                    steps {

                        echo 'Running quality scan with Sonar..... '
                        sh 'sleep 4'

                    }
                }
                stage('Security Fortify Scan') {
                    steps {
                        echo 'Running local fortify scan......'
                        sh 'sleep 2'

                    }
                }
                stage('Security Blackduck Scan') {
                    steps {
                        echo 'Running OSS Scan ..........'
                        sh 'sleep 5'

                    }
                }
            }
        }

        stage('Package') {
            when { anyOf { branch 'master'; branch 'PR-*' } }
            agent any
            steps {
                script {
                    sh 'mvn install'
                    if (env.BRANCH_NAME == 'master') {
                        pom = readMavenPom file: 'pom.xml'
                        TAG = pom.version
                    } else {
                        TAG = env.BRANCH_NAME
                    }
                    sh "docker build -t ${env.IMAGE}:${TAG} ."
                }
            }
        }

        stage('Deploy to Dev') {
            when { anyOf { branch 'master'; branch 'PR-*' } }
            steps {
                echo 'Deploying....'
                sh 'docker rm -f petclinic-tomcat-temp || true'
                sh "docker run -d -p 9966:8080 --name petclinic-tomcat-temp ${env.IMAGE}:${TAG}"
            }
            post {
                always {
                    echo 'Call health check endpoint....'
                    sh 'sleep 2'
                }
            }
        }
        stage('Validate Deploy to dev') {
            when { anyOf { branch 'master'; branch 'PR-*' } }
            failFast true
            parallel {
                stage('Healthcheck') {
                    steps {
                        echo 'Call health check endpoint....'
                        sh 'sleep 1'
                    }

                }

                stage('Run Smoke Tests on dev') {
                    steps {
                        echo 'Run Smoke tests on dev......'
                        sh "cd regression-suite"
                        sh "mvn clean -B test -DPETCLINIC_URL=http://dev-petclinic:8080/petclinic/"
                        echo "Should be accessible at http://localhost:18888/petclinic"
                    }
                }

                stage('API Service Test') {
                    steps {
                        echo 'Call API test with Postman....'
                        sh 'sleep 7'
                    }
                }
            }
        }

        stage('Deploy to Test') {
            when { anyOf { branch 'master' } }
            steps {
                echo 'Deploy to test environment....'
                sh 'sleep 6'
            }
        }

        stage('Validate Deploy to test') {
            when { anyOf { branch 'master' } }
            failFast true
            parallel {
                stage('Healthcheck test') {
                    steps {
                        echo 'Call health check endpoint....'
                        sh 'sleep 1'
                    }
                }

                stage('Run Smoke Tests on test') {
                    steps {
                        echo 'Run Smoke tests on test......'
                        sh 'sleep 4'
                    }
                }

            }
        }



        stage('Acceptance') {
            when { branch 'master' }
            failFast true
            parallel {

                stage('Run Functional Tests'){
                    steps {
                        echo 'Run functional tests........'
                        sh 'sleep 8'
                    }
                }
                stage('Run Performance Tests'){
                    steps {
                        echo 'Run performance tests with executePerformanceJMeterTests...........'
                        sh 'sleep 10'
                    }
                }


            }
        }

        stage('Deploy to prod?'){
            when {  branch 'master' }
            steps {
                input(
                        message: 'Do you want to publish a new version of the library in A?'
                )
            }
        }

        stage('Release Management & Audit') {
            when { branch 'master' }
            failFast true
            parallel {
                stage('Create Tag') {
                    steps {
                        echo 'Create GitHub Tag....  with...'
                        sh 'sleep 1'

                    }
                }

                stage('Create Change Management App Ticket') {
                    steps {
                        echo 'create change management ticket in ServiceNow....... '
                        sh 'sleep 4'
                    }
                }

                stage('Create release notes') {
                    steps {
                        echo 'create release notes....... '
                        sh 'sleep 4'
                    }
                }
            }
        }
        stage('Deploy to Prod - Pool A Blue') {
            when {  branch 'master'  }
            steps {
                echo 'Deploy to prod A Blue environment....'
                echo 'Validate Deploy with ......executeHealthCheck'
                sh 'sleep 4'
            }
        }

        stage('Run Smoke Tests on A Blue') {
            when {  branch 'master'  }
            steps {
                echo 'Run Smoke tests on prod A Blue......'
                sh 'sleep 2'
            }
        }

        stage('Deploy to Prod - Pool A Green') {
            when {  branch 'master'  }
            steps {
                echo 'Deploy to prod A Green environment....'
                echo 'Validate Deploy with ......executeHealthCheck'
                sh 'sleep 4'
            }
        }

        stage('Run Smoke Tests on A Green') {
            when {  branch 'master'  }
            steps {
                echo 'Run Smoke tests on prod A Green......'
                sh 'sleep 2'
            }
        }

        stage('Notify Build Status on A') {
            when {  branch 'master'  }
            steps {
                echo 'Notify Build Status on Slack.......'
                sh 'sleep 1'
            }

        }

        stage('Deploy to prod B?'){
            when {  branch 'master'  }
            steps {
                input(
                        message: 'Do you want to publish a new version of the library on B?'
                )
            }
        }

        stage('Deploy to Prod - Pool B Blue') {
            when {  branch 'master'  }
            steps {
                echo 'Deploy to prod A Blue environment....'
                echo 'Validate Deploy with ......executeHealthCheck'
                sh 'sleep 4'
            }
        }

        stage('Run Smoke Tests on B Blue') {
            when {  branch 'master'  }
            steps {
                echo 'Run Smoke tests on prod B Green......'
                sh 'sleep 2'
            }
        }

        stage('Deploy to Prod - Pool B Green') {
            when {  branch 'master'  }
            steps {
                echo 'Deploy to prod A Blue environment....'
                echo 'Validate Deploy with ......executeHealthCheck'
                sh 'sleep 4'
            }
        }

        stage('Run Smoke Tests on B Green') {
            when {  branch 'master'  }
            steps {
                echo 'Run Smoke tests on prod B Green......'
                sh 'sleep 2'
            }
        }

        stage('Notify Build Status on B') {
            when {  branch 'master'  }
            steps {
                echo 'Notify Build Status on Slack.......'
                sh 'sleep 1'
            }

        }

    }
}
