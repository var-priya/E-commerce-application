@Library('Shared') _

pipeline {
    agent any

    environment {
        DOCKER_IMAGE_NAME = 'varpriya/easyshop-app'
        DOCKER_MIGRATION_IMAGE_NAME = 'varpriya/easyshop-migration'
        DOCKER_IMAGE_TAG = "${BUILD_NUMBER}"
        GITHUB_CREDENTIALS = credentials('github-credentials')
        GIT_BRANCH = "master"
        PROJECT_NAME = "E-commerce-application"
        GITHUB_REPO = "https://github.com/var-priya/E-commerce-application.git"
        PROJECT_KEY = "E-commerce-application"
        SONAR_SERVER = "sonar"
        ENVIRONMENT = "dev"
    }

    stages {

        stage('Cleanup Workspace') {
            steps {
                script {
                    clean_ws(
                        environment: env.ENVIRONMENT,
                        when: 'pre'
                    )
                }
            }
        }

        stage('Clone Repository') {
            steps {
                script {
                    checkoutCode(
                        url: env.GITHUB_REPO,
                        branchName: env.GIT_BRANCH,
                        credentialsId: 'github-credentials',
                        environment: env.ENVIRONMENT
                    )
                }
            }
        }

        stage('SonarQube Scan') {
            steps {
                script {
                    sonarScan(
                        projectKey: env.PROJECT_KEY,
                        branchName: env.GIT_BRANCH,
                        sonarServer: env.SONAR_SERVER,
                        environment: env.ENVIRONMENT
                    )
                }
            }
        }

        stage('Build Docker Images') {
            parallel {
                stage('Build Main App Image') {
                    steps {
                        script {
                            docker_build(
                                imageName: env.DOCKER_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG,
                                dockerfile: 'Dockerfile',
                                context: '.',
                                environment: env.ENVIRONMENT
                            )
                        }
                    }
                }

                stage('Build Migration Image') {
                    steps {
                        script {
                            docker_build(
                                imageName: env.DOCKER_MIGRATION_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG,
                                dockerfile: 'scripts/Dockerfile.migration',
                                context: '.',
                                environment: env.ENVIRONMENT
                            )
                        }
                    }
                }
            }
        }

        stage('Run Unit Tests') {
            steps {
                script {
                    run_tests(
                        environment: env.ENVIRONMENT
                    )
                }
            }
        }

        stage('Trivy Scan - Dev') {
            steps {
                script {
                    trivyScan(
                        imageName: env.DOCKER_IMAGE_NAME,
                        imageTag: env.DOCKER_IMAGE_TAG,
                        severity: 'CRITICAL,HIGH,MEDIUM',
                        ignoreUnfixed: true,
                        format: 'table',
                        environment: env.ENVIRONMENT
                    )
                }
            }
        }

        stage('Push Docker Images') {
            parallel {
                stage('Push Main App Image') {
                    steps {
                        script {
                            docker_push(
                                imageName: env.DOCKER_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG,
                                credentials: 'docker-hub-credentials',
                                environment: env.ENVIRONMENT
                            )
                        }
                    }
                }

                stage('Push Migration Image') {
                    steps {
                        script {
                            docker_push(
                                imageName: env.DOCKER_MIGRATION_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG,
                                credentials: 'docker-hub-credentials',
                                environment: env.ENVIRONMENT
                            )
                        }
                    }
                }
            }
        }

        stage('Update Kubernetes Manifests') {
            steps {
                script {
                    update_k8s_manifests(
                        imageTag: env.DOCKER_IMAGE_TAG,
                        manifestsPath: 'kubernetes',
                        gitCredentials: 'github-credentials',
                        gitUserName: 'varpriya',
                        gitUserEmail: 'varshneyp27@gmail.com',
                        environment: env.ENVIRONMENT
                    )
                }
            }
        }
    }

    post {
        success {
            echo " Pipeline completed successfully for ${env.ENVIRONMENT.toUpperCase()}."
        }
        failure {
            echo " Pipeline failed for ${env.ENVIRONMENT.toUpperCase()}."
        }
        always {
            echo " Final cleanup..."
            clean_ws(environment: env.ENVIRONMENT, when: 'post')
        }
    }
}

