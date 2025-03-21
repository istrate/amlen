// By default we build on the oldest supported distro (centos7)
// Jenkinsfiles for other distros (with associated build container Dockerfiles
// are in server_build/buildcontainer
//
pipeline {
  agent {
    kubernetes {
      label 'amlen-centos7-build-pod'
      yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: amlen-centos7-build
    image: quay.io/amlen/amlen-builder-centos7:latest
    command:
    - cat
    tty: true
    resources:
      limits:
        memory: "4Gi"
        cpu: "2"
      requests:
        memory: "4Gi"
        cpu: "2"
  - name: jnlp
    volumeMounts:
    - name: volume-known-hosts
      mountPath: /home/jenkins/.ssh
  volumes:
  - name: volume-known-hosts
    configMap:
      name: known-hosts
"""
    }
  }
    stages {
        stage('Build') {
            steps {
                container('amlen-centos7-build') {
                   sh 'pwd && free -m && cd server_build && bash buildcontainer/build.sh'
                }
            }
        }
        stage('Deploy') {
            steps {
                container('jnlp') {
                    sshagent ( ['projects-storage.eclipse.org-bot-ssh']) {
                        sh '''
                            pwd
                            ls
                            ls ..
                            echo ${GIT_BRANCH}
                            #ssh -o BatchMode=yes genie.amlen@projects-storage.eclipse.org rm -rf /home/data/httpd/download.eclipse.org/projectname/snapshots
                            ssh -o BatchMode=yes genie.amlen@projects-storage.eclipse.org mkdir -p /home/data/httpd/download.eclipse.org/amlen/snapshots/${GIT_BRANCH}/centos7/
                            scp -o BatchMode=yes -r rpms/*.tar.gz rpms/*rpm genie.amlen@projects-storage.eclipse.org:/home/data/httpd/download.eclipse.org/amlen/snapshots/${GIT_BRANCH}/centos7/
                        '''
                    }
                }
            }
        }
    }
    post {
        // send a mail on unsuccessful and fixed builds
        unsuccessful { // means unstable || failure || aborted
            emailext subject: 'Build $BUILD_STATUS $PROJECT_NAME #$BUILD_NUMBER!', 
            body: '''Check console output at $BUILD_URL to view the results.''',
            recipientProviders: [culprits(), requestor()], 
            to: 'levell@uk.ibm.com'
        }
        fixed { // back to normal
            emailext subject: 'Build $BUILD_STATUS $PROJECT_NAME #$BUILD_NUMBER!', 
            body: '''Check console output at $BUILD_URL to view the results.''',
            recipientProviders: [culprits(), requestor()], 
            to: 'levell@uk.ibm.com'
        }
    }
}
