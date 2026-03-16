node {

    /* -------------------------------
       Global Variables
    --------------------------------*/

    def GIT_REPO           = "https://github.com/your-repo/java-app.git"
    def GIT_BRANCH         = "main"

    def MAVEN_TOOL         = "Maven3"
    def SONAR_SERVER       = "SonarQube"

    def SONAR_PROJECT_KEY  = "java-project"
    def SONAR_PROJECT_NAME = "java-project"

    def TRIVY_REPORT       = "reports/trivy-report.txt"
    def ZAP_REPORT         = "reports/zap-report.html"

    def ZAP_TARGET_URL     = "http://your-app-url"

    def SLACK_CHANNEL      = "#ci-operation-notifications"

    def mvnHome


    try {

        stage('Checkout Code') {

            git url: GIT_REPO, branch: GIT_BRANCH

        }


        stage('Initialize Tools') {

            mvnHome = tool MAVEN_TOOL

        }


        stage('Code Compilation') {

            sh "${mvnHome}/bin/mvn clean compile"

        }


        stage('Unit Testing') {

            sh "${mvnHome}/bin/mvn test"

        }


        stage('Static Code Analysis - SonarQube') {

            withSonarQubeEnv(SONAR_SERVER) {

                sh """
                ${mvnHome}/bin/mvn sonar:sonar \
                -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                -Dsonar.projectName=${SONAR_PROJECT_NAME}
                """

            }

        }


        stage('Dependency Scan - Trivy') {

            sh """
            mkdir -p reports
            trivy fs --format table -o ${TRIVY_REPORT} .
            """

        }


        stage('DAST Scan - OWASP ZAP') {

            sh """
            docker run -t owasp/zap2docker-stable zap-baseline.py \
            -t ${ZAP_TARGET_URL} \
            -r ${ZAP_REPORT}
            """

        }


        /* -------------------------------
           Success Handling
        --------------------------------*/

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

        /* -------------------------------
           Failure Handling
        --------------------------------*/

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

        /* -------------------------------
           Always Execute
        --------------------------------*/

        archiveArtifacts artifacts: 'reports/*', fingerprint: true

    }
}