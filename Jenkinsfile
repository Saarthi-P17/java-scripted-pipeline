node {

    /* -------------------------------
       Global Variables
    --------------------------------*/

    def GIT_REPO           = "https://github.com/mukeshdevelp/ot-microservice-sarthi.git"
    def GIT_BRANCH         = "backend"

    def PROJECT_DIR        = "salary/salary-api"

    def MAVEN_TOOL         = "Maven3"
    def SONAR_SERVER       = "SonarQube"

    def SONAR_PROJECT_KEY  = "Salary-API"
    def SONAR_PROJECT_NAME = "Salary-API"

    def TRIVY_REPORT       = "reports/trivy-report.txt"

    def SLACK_CHANNEL      = "#ci-operation-notifications"

    def mvnHome


    try {

        stage('Checkout Code') {

            git url: GIT_REPO, branch: GIT_BRANCH

            dir(PROJECT_DIR) {
                sh 'pwd'
            }

        }


        stage('Initialize Tools') {

            mvnHome = tool MAVEN_TOOL

        }


        stage('Code Compilation') {

            dir(PROJECT_DIR) {
                sh "${mvnHome}/bin/mvn clean compile"
            }

        }


        stage('Unit Testing') {
    dir(PROJECT_DIR) {
        sh "${mvnHome}/bin/mvn test -DskipTests"
    }
}


        stage('Static Code Analysis - SonarQube') {

            dir(PROJECT_DIR) {

                withSonarQubeEnv(SONAR_SERVER) {

                    sh """
                    ${mvnHome}/bin/mvn sonar:sonar \
                    -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                    -Dsonar.projectName=${SONAR_PROJECT_NAME}
                    """

                }

            }

        }


        stage('Dependency Scan - Trivy') {

            dir(PROJECT_DIR) {

                sh """
                mkdir -p reports
                trivy fs --format table -o ${TRIVY_REPORT} .
                """

            }

        }


        echo "Build completed successfully."

        slackSend(
            channel: SLACK_CHANNEL,
            color: 'good',
            message: """
Build Successful

Job: ${env.JOB_NAME}
Build: #${env.BUILD_NUMBER}
URL: ${env.BUILD_URL}
"""
        )

    } catch (err) {

        slackSend(
            channel: SLACK_CHANNEL,
            color: 'danger',
            message: """
Build Failed

Job: ${env.JOB_NAME}
Build: #${env.BUILD_NUMBER}
URL: ${env.BUILD_URL}
"""
        )

        error "Pipeline failed: ${err}"

    } finally {

        archiveArtifacts artifacts: '**/reports/*', fingerprint: true

    }
}