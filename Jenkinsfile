pipeline {
    environment {
        DOCKER_ID = "alidoghan"
        MOVIE_IMAGE = "movie-service"
        CAST_IMAGE = "cast-service"
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }
    agent any
    stages {
        stage('Docker Build'){
            steps {
                script {
                    sh '''
                        docker build -t $DOCKER_ID/$MOVIE_IMAGE:$DOCKER_TAG ./movie-service
                        docker build -t $DOCKER_ID/$CAST_IMAGE:$DOCKER_TAG ./cast-service
                    '''
                }
            }
        }
        stage('Docker Push'){
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }
            steps {
                script {
                    sh '''
                        docker login -u $DOCKER_ID -p $DOCKER_PASS
                        docker push $DOCKER_ID/$MOVIE_IMAGE:$DOCKER_TAG
                        docker push $DOCKER_ID/$CAST_IMAGE:$DOCKER_TAG
                    '''
                }
            }
        }
        stage('Deployment in dev'){
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                        rm -Rf ~/.kube
                        mkdir ~/.kube
                        cat $KUBECONFIG > ~/.kube/config
                        cp charts/values-movie.yaml movie-values.yml
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" movie-values.yml
                        helm upgrade --install movie-app charts --values=movie-values.yml --set service.nodePort=30020 --namespace dev
                        cp charts/values-cast.yaml cast-values.yml
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" cast-values.yml
                        helm upgrade --install cast-app charts --values=cast-values.yml --set service.nodePort=30021 --namespace dev
                    '''
                }
            }
        }
        stage('Deployment in qa'){
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                        rm -Rf ~/.kube
                        mkdir ~/.kube
                        cat $KUBECONFIG > ~/.kube/config
                        cp charts/values-movie.yaml movie-values.yml
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" movie-values.yml
                        helm upgrade --install movie-app charts --values=movie-values.yml --set service.nodePort=30022 --namespace qa
                        cp charts/values-cast.yaml cast-values.yml
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" cast-values.yml
                        helm upgrade --install cast-app charts --values=cast-values.yml --set service.nodePort=30023 --namespace qa
                    '''
                }
            }
        }
        stage('Deployment in staging'){
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                        rm -Rf ~/.kube
                        mkdir ~/.kube
                        cat $KUBECONFIG > ~/.kube/config
                        cp charts/values-movie.yaml movie-values.yml
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" movie-values.yml
                        helm upgrade --install movie-app charts --values=movie-values.yml --set service.nodePort=30024 --namespace staging
                        cp charts/values-cast.yaml cast-values.yml
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" cast-values.yml
                        helm upgrade --install cast-app charts --values=cast-values.yml --set service.nodePort=30025 --namespace staging
                    '''
                }
            }
        }
        stage('Deployment in prod'){
            when {
                expression {
                    return env.GIT_BRANCH == 'origin/master' || env.GIT_BRANCH == 'master'
                }
            }
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                timeout(time: 15, unit: "MINUTES") {
                    input message: 'Do you want to deploy in production ?', ok: 'Yes'
                }
                script {
                    sh '''
                        rm -Rf ~/.kube
                        mkdir ~/.kube
                        cat $KUBECONFIG > ~/.kube/config
                        cp charts/values-movie.yaml movie-values.yml
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" movie-values.yml
                        helm upgrade --install movie-app charts --values=movie-values.yml --set service.nodePort=30026 --namespace prod
                        cp charts/values-cast.yaml cast-values.yml
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" cast-values.yml
                        helm upgrade --install cast-app charts --values=cast-values.yml --set service.nodePort=30027 --namespace prod
                    '''
                }
            }
        }
    }
    post {
        failure {
            mail to: "eng.alidoghan@gmail.com",
                 subject: "${env.JOB_NAME} - Build # ${env.BUILD_ID} has failed",
                 body: "Check console output at ${env.BUILD_URL}"
        }
    }
}
