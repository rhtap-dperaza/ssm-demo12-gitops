pipeline {
    agent {
        kubernetes {
            yaml """
              apiVersion: v1
              kind: Pod
              spec:
                containers:
                - name: 'runner'
                  image: 'quay.io/redhat-appstudio/rhtap-task-runner:latest'
                  securityContext:
                    privileged: true
            """
        }
        
    }
    environment {
        HOME = "${WORKSPACE}"
        DOCKER_CONFIG = "${WORKSPACE}/.docker"
        ROX_API_TOKEN = credentials('ROX_API_TOKEN')
        ROX_CENTRAL_ENDPOINT = credentials('ROX_CENTRAL_ENDPOINT')
        GITOPS_AUTH_PASSWORD = credentials('GITOPS_AUTH_PASSWORD')
        GITOPS_AUTH_USERNAME = credentials('GITOPS_AUTH_USERNAME')
        QUAY_IO_CREDS = credentials('QUAY_IO_CREDS')
        COSIGN_SECRET_PASSWORD = credentials('COSIGN_SECRET_PASSWORD')
        COSIGN_SECRET_KEY = credentials('COSIGN_SECRET_KEY')
        COSIGN_PUBLIC_KEY = credentials('COSIGN_PUBLIC_KEY')
        QUAY_IO_CREDS = credentials('QUAY_IO_CREDS')
        IMAGE_REGISTRY_USER = credentials('IMAGE_REGISTRY_USER')
        IMAGE_REGISTRY_PASSWORD = credentials('IMAGE_REGISTRY_PASSWORD')
        TUF_MIRROR = credentials('TUF_MIRROR')
        REKOR_HOST = credentials('REKOR_HOST')
        TRUSTIFICATION_BOMBASTIC_API_URL = credentials('TRUSTIFICATION_BOMBASTIC_API_URL')
        TRUSTIFICATION_OIDC_ISSUER_URL = credentials('TRUSTIFICATION_OIDC_ISSUER_URL')
        TRUSTIFICATION_OIDC_CLIENT_ID = credentials('TRUSTIFICATION_OIDC_CLIENT_ID')
        TRUSTIFICATION_OIDC_CLIENT_SECRET = credentials('TRUSTIFICATION_OIDC_CLIENT_SECRET')
        TRUSTIFICATION_SUPPORTED_CYCLONEDX_VERSION = credentials('TRUSTIFICATION_SUPPORTED_CYCLONEDX_VERSION')
    }
    stages {
        stage('init') {
            steps {
                container('runner') {
                    sh '''
                        cp -R /work/* .
                        env
                        git config --global --add safe.directory $WORKSPACE
                        echo running init
                        ./rhtap/init.sh
                    '''
                }
            }
        }

        stage('ec verify') {
            steps {
                container('runner') {
                    sh '''
                        echo "gathering deploy images"
                        ./rhtap/gather-deploy-images.sh
                        echo "verifying enterprise contract"
                        ./rhtap/verify-enterprise-contract.sh
                    '''
                }
            }
        }

        stage('sbom upload') {
            steps {
                container('runner') {
                    sh '''
                        chmod +x ./rhtap/gather-images-to-upload-sbom.sh
                        echo "gathering sbom images"
                        ./rhtap/gather-images-to-upload-sbom.sh
                        echo "downloading sbom images"
                        ./rhtap/download-sbom-from-url-in-attestation.sh
                        echo "uploading sbom images to tpa"
                        ./rhtap/upload-sbom-to-trustification.sh
                    '''
                }
            }
        }
    }
}
