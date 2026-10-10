pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    triggers {
        cron('H/15 * * * *')
    }

    environment {
        // Public Application Load Balancer
        ALB_URL = 'http://alb-three-tier-1243415748.us-east-1.elb.amazonaws.com'

        // Internal Application Load Balancer - Port 80
        INTERNAL_ALB_URL = 'http://internal-alb-internal-app-210434280.us-east-1.elb.amazonaws.com'
    }

    stages {

        stage('Validate Tier 1: Public ALB Ingress') {
            steps {
                echo 'DAEA TEST 01: Validating public Application Load Balancer...'

                sh '''
                    set -eu

                    STATUS=$(curl --silent --show-error \
                        --output /dev/null \
                        --write-out "%{http_code}" \
                        --connect-timeout 5 \
                        --max-time 15 \
                        "${ALB_URL}/")

                    echo "Public ALB HTTP Status: $STATUS"

                    if [ "$STATUS" != "200" ]; then
                        echo "FAILED: Public ALB did not return HTTP 200."
                        exit 1
                    fi

                    echo "PASSED: Public ALB connectivity verified."
                '''
            }
        }

        stage('Validate Tier 2: Internal ALB Connectivity') {
            steps {
                echo 'DAEA TEST 02: Validating internal Application Load Balancer on port 80...'

                sh '''
                    set -eu

                    RESPONSE=$(curl --silent --show-error --fail \
                        --connect-timeout 5 \
                        --max-time 15 \
                        "${INTERNAL_ALB_URL}/")

                    echo "Internal ALB Response: $RESPONSE"

                    echo "$RESPONSE" | grep -Eq \
                        '("status": *"connected"|Database connected successfully)' || {
                        echo "FAILED: Internal ALB returned an unexpected application response."
                        exit 1
                    }

                    echo "PASSED: Internal ALB application connectivity verified."
                '''
            }
        }

        stage('Validate Tier 3: Application and Database') {
            steps {
                echo 'DAEA TEST 03: Validating application-reported MariaDB connectivity...'

                sh '''
                    set -eu

                    RESPONSE=$(curl --silent --show-error --fail \
                        --connect-timeout 5 \
                        --max-time 15 \
                        "${INTERNAL_ALB_URL}/")

                    echo "Application Database Response: $RESPONSE"

                    echo "$RESPONSE" | grep -Eq \
                        '"status": *"connected"' || {
                        echo "FAILED: Application did not report database connectivity."
                        exit 1
                    }

                    echo "$RESPONSE" | grep -q \
                        'Database connected successfully' || {
                        echo "FAILED: Expected database record was not returned."
                        exit 1
                    }

                    echo "PASSED: Application reported successful MariaDB connectivity."
                '''
            }
        }

        stage('Validate End-to-End Application Flow') {
            steps {
                echo 'DAEA TEST 04: Validating public ALB to application and database flow...'

                sh '''
                    set -eu

                    API_RESPONSE=$(curl --silent --show-error --fail \
                        --connect-timeout 5 \
                        --max-time 15 \
                        "${ALB_URL}/api/")

                    echo "End-to-End Application Response: $API_RESPONSE"

                    echo "$API_RESPONSE" | grep -Eq \
                        '"status": *"connected"' || {
                        echo "FAILED: End-to-end application connectivity check failed."
                        exit 1
                    }

                    echo "$API_RESPONSE" | grep -q \
                        'Database connected successfully' || {
                        echo "FAILED: End-to-end database record validation failed."
                        exit 1
                    }

                    echo "PASSED: End-to-end application and database flow verified."
                '''
            }
        }
    }

    post {

        success {
            echo '''
            ==========================================
                 DAEA INFRASTRUCTURE VALIDATION
            ==========================================

            STATUS: SUCCESS

            Public ALB:             PASSED
            Internal ALB:           PASSED
            Application Response:   PASSED
            Database Response:      PASSED
            End-to-End Flow:        PASSED

            Architecture Stage: 4
            Validation Result: SUCCESS

            ==========================================
            '''
        }

        failure {
            echo '''
            ==========================================
                 DAEA INFRASTRUCTURE ALERT
            ==========================================

            STATUS: FAILURE

            One or more infrastructure validation
            stages failed.

            Review the Jenkins console output to
            identify the affected infrastructure
            component.

            ==========================================
            '''
        }

        always {
            echo "DAEA validation completed. Build: ${env.BUILD_NUMBER}"
        }
    }
}
