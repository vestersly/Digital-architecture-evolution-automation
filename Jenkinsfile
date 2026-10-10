pipeline {
    agent any

    triggers {
        cron('H/15 * * * *')
    }

    environment {
        ALB_URL      = 'http://alb-three-tier-1243415748.us-east-1.elb.amazonaws.com'
        APP_IP       = '10.0.4.162'
        APP_PORT     = '8080'
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

        stage('Validate Peering Link: Private App-Server Route') {
            steps {
                echo "Testing private route across VPC Peering to ${APP_IP}:${APP_PORT}..."
                sh '''
                    PEER_RESPONSE=$(curl -s -f http://${APP_IP}:${APP_PORT}/)
                    echo "Private Peering Response: $PEER_RESPONSE"

                    echo "$PEER_RESPONSE" | grep -q '"status": "connected"' || {
                        echo "Direct App Peering check failed!"
                        exit 1
                    }
                    echo "Private VPC Peering connectivity verified successfully."
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
                    echo "All 3 tiers and VPC Peering verified end-to-end!"
                '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: All 3 tiers and VPC Peering are healthy.'
        }
        failure {
            echo 'ALERT: Validation check failed! Inspect Jenkins logs.'
        }
    }
}
