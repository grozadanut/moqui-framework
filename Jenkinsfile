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
                docker compose -f opensearch-compose.yml stop
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
                junit 'framework/build/test-results/**/*.xml'
                junit 'runtime/component/**/build/test-results/**/*.xml'

                sh '''
                cd docker
                docker compose -f opensearch-compose.yml stop
                ./clean.sh
                '''
            }
        }
        stage('Deploy to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub',
                        usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh '''
                    echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin

                    ./gradlew cleanDb cleanLog cleanSessions cleanTempDir cleanOther
                    ./gradlew build
                    cd docker/simple
                    ./docker-build.sh ../.. $DOCKER_USERNAME/moqui:develop
                    docker push $DOCKER_USERNAME/moqui:develop
                    '''
                }
            }
        }
    }
}