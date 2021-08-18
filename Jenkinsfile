// Uses Declarative syntax to run commands inside a container.
pipeline {
    agent {
        kubernetes {
            // Rather than inline YAML, in a multibranch Pipeline you could use: yamlFile 'jenkins-pod.yaml'
            // Or, to avoid YAML:
            // containerTemplate {
            //     name 'shell'
            //     image 'ubuntu'
            //     command 'sleep'
            //     args 'infinity'
            // }
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: shell
    image: taylorsilva/carvel-apps
    env:
    - name: KBLD_REGISTRY_HOSTNAME_0
      value: "harbor.tools.azure.nvcodes.net"
    - name: KBLD_REGISTRY_USERNAME_0
      valueFrom:
        secretKeyRef:
          name: bookinfo-secrets
          key: registry-user
    - name: KBLD_REGISTRY_PASSWORD_0
      valueFrom:
        secretKeyRef:
          name: bookinfo-secrets
          key: registry-password
    command:
    - sleep
    args:
    - infinity
'''
            // Can also wrap individual steps:
            // container('shell') {
            //     sh 'hostname'
            // }
            defaultContainer 'shell'
        }
    }
    stages {
        stage('checkout'){
            environment { 
                GIT_AUTH = credentials('f09786ed-8c24-4d1a-a768-f0b5266383be') 
            }
            steps{
                container('jnlp'){
                    sh('''
                        git config --global credential.helper "!f() { echo username=\\$GIT_AUTH_USR; echo password=\\$GIT_AUTH_PSW; }; f"
                        git config --global user.email "nitashav@vmware.com"
                        git config --global user.name "nitashav-vmw"
                        git clone https://github.com/nitashav-vmw/fiserv-esf.git
                    ''')
                }
            }
        }
        stage('generate-image-reference') {
            steps {
              sh('''
              mkdir .imgpkg
              kbld -f . --imgpkg-lock-output ./.imgpkg/images.yml
              ls . 
              cat ./.imgpkg/images.yml
              ''') 
             
            }
        }
        stage('esf-bundle-push'){
            steps{
                sh ('''
                echo build number: ${BUILD_NUMBER}
                imgpkg push -b harbor.tools.azure.nvcodes.net/esf-release/esf:v${BUILD_NUMBER} -f . --registry-username "${KBLD_REGISTRY_USERNAME_0}" --registry-password "${KBLD_REGISTRY_PASSWORD_0}"
                ''')
            }
        }
    }
}
