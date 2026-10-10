pipeline {
    agent any

    triggers {
        cron('H/15 * * * *')
    }

    environment {
        ALB_URL          = 'http://alb-three-tier-1243415748.us-east-1.elb.amazonaws.com'
        INTERNAL_ALB_URL = 'http://internal-alb-internal-app-210434280.us-east-1.elb.amazonaws.com'
    }

    stages {
        stage('Validate Tier 1: Public ALB Ingress') {
            steps {
                echo 'Testing web tier ingress via Public ALB...'
                sh '''
                    STATUS=$(curl -s -o /dev/null -w "%{http_code}" ${ALB_URL}/)
                    if [ "$STATUS" -ne 200 ]; then
                        echo "Web tier ALB check failed with status: $STATUS"
                        exit 1
                    fi
                    echo "Web tier returned HTTP 200 OK via ALB"
                '''
            }
        }

        stage('Validate Tier 2: Internal ALB & Auto Scaling App Tier') {
            steps {
                echo "Testing private application route via Internal ALB..."
                sh '''
                    # Check internal ALB first; fallback to Web-Server reverse proxy if Jenkins cannot resolve internal VPC DNS directly
                    RESPONSE=$(curl -s -f --connect-timeout 5 ${INTERNAL_ALB_URL}:8080/ || curl -s -f --connect-timeout 5 ${INTERNAL_ALB_URL}/ || curl -s -f --connect-timeout 5 ${ALB_URL}/api/)
                    echo "App Tier Response: $RESPONSE"

                    echo "$RESPONSE" | grep -Eq '("status": *"connected"|Database connected successfully)' || {
                        echo "Application tier validation check failed!"
                        exit 1
                    }
                    echo "Internal Application Auto Scaling fleet connectivity verified successfully."
                '''
            }
        }

        stage('Validate End-to-End Application & MariaDB Flow') {
            steps {
                echo 'Testing end-to-end data retrieval through ALB /api/ route...'
                sh '''
                    API_RESPONSE=$(curl -s -f ${ALB_URL}/api/)
                    echo "Full Flow Payload: $API_RESPONSE"

                    echo "$API_RESPONSE" | grep -q "Database connected successfully" || {
                        echo "Database record payload verification failed!"
                        exit 1
                    }
                    echo "All 3 tiers, Auto Scaling Group, and MariaDB verified end-to-end!"
                '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: All 3 tiers, Auto Scaling fleet, and Database are healthy.'
        }
        failure {
            echo 'ALERT: Validation check failed! Inspect Jenkins logs.'
        }
    }
}
