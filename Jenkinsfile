pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/grozadanut/moqui-framework'
            }
        }
        stage('Start opensearch') {
            steps {
                sh '''
                cd docker
                ./clean.sh
                docker compose -f opensearch-compose.yml up -d
                '''
            }
        }
        stage('Prepare runtime') {
            steps {
                sh '''
                ./gradlew getComponent -Pcomponent=MarbleERP
                ./gradlew getComponent -Pcomponent=PopCommerce
                ./gradlew getComponent -Pcomponent=HiveMind
                ./gradlew getComponent -Pcomponent=mantle-udm
                ./gradlew getComponent -Pcomponent=mantle-usl
                ./gradlew getComponent -Pcomponent=PopRestStore
                ./gradlew getComponent -Pcomponent=moqui-linic-legacy
                ./gradlew gitp
                '''
            }
        }
        stage('Load demo data') {
            steps {
                sh '''
                ./gradlew cleanDb cleanLog cleanSessions cleanTempDir cleanOther
                ./gradlew load
                '''
            }
        }
        stage('Run Tests') {
            steps {
                sh "./gradlew test"
            }
        }
    }
    post {
        always {
            junit 'framework/build/test-results/**/*.xml'
            junit 'runtime/component/**/build/test-results/**/*.xml'

            sh '''
            cd docker
            docker compose -f opensearch-compose.yml stop
            '''
        }
    	failure {
    		script {
    			def jobName = env.JOB_NAME
    			def buildNumber = env.BUILD_NUMBER
    			def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
    			def commitMessage = sh(script: "git log -1 --pretty=%B", returnStdout: true).trim()

    			def body = """
    			<p>${jobName} - Build ${buildNumber}</p>
    			<p>Pipeline Status: ${pipelineStatus.toUpperCase()}</p>
    			<p>Commit: ${commitMessage}</p>
    			<a href="https://ci.flexbiz.ro/blue/organizations/jenkins/${jobName}/detail/${jobName}/${buildNumber}/pipeline">Check the console output.</a>
    			"""

    			emailext(
    				subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
    				body: body,
    				to: 'danut@flexbiz.ro',
    				from: 'info@flexbiz.ro',
    				replyTo: 'info@flexbiz.ro',
    				mimeType: 'text/html'
    			)
    		}
    	}
    }
}