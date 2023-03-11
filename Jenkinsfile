pipeline 
{ 
    environment 
    {     
      ContainerImageRegistry ="273381533537.dkr.ecr.us-east-2.amazonaws.com"
    }
    agent 
    {
       kubernetes {
      yaml '''
        apiAPP_VERSION: v1
        kind: Pod
        metadata:
          name: nginx-deployment
        spec:
          containers:
          - name: hadolint
            image: hadolint/hadolint:latest-debian
            resources:
              requests:
                memory: "100Mi"
                cpu: "50m"
              limits:
                memory: "300Mi"
                cpu: "200m"
            command:
            - cat
            tty: true
          - name: git-secret
            image: bitbucketpipelines/git-secrets-scan:latest
            resources:
              requests:
                memory: "100Mi"
                cpu: "50m"
              limits:
                memory: "300Mi"
                cpu: "200m"
            command:
            - cat
            tty: true
          - name: trufflehog
            image: trufflesecurity/trufflehog:latest
            resources:
              requests:
                memory: "100Mi"
                cpu: "50m"
              limits:
                memory: "300Mi"
                cpu: "200m"
            command:
            - cat
            tty: true
          - name: node
            #image: 273381533537.dkr.ecr.us-east-1.amazonaws.com/mcdscertimg-nodejs-12:latest
            image: node:19-alpine           
            tty: true
            command: ["/bin/sh"]
          - name: ecr
            image: amazon/aws-cli:latest
            resources:
              requests:
                memory: "100Mi"
                cpu: "50m"
              limits:
                memory: "300Mi"
                cpu: "200m"            
            command:
            - cat
            tty: true
          - name: sonar
            image: emeraldsquad/sonar-scanner:latest         
            command:
            - cat
            tty: true
          - name: helm
            image: alpine/helm:latest
            resources:
              requests:
                memory: "100Mi"
                cpu: "50m"
              limits:
                memory: "300Mi"
                cpu: "200m"            
            command:
            - cat
            tty: true
          - name: yq
            image: mikefarah/yq:latest
            command:
            - cat
            tty: true
          - name: mcds-client
            image: mikefarah/yq:latest          
            command:
            - cat
            tty: true
            volumeMounts:
              - mountPath: /var/run/docker.sock
                name: docker-sock
          - name: trivy
            image: aquasec/trivy:latest
            command:
            - cat
            tty: true
            volumeMounts:
              - mountPath: /var/run/docker.sock
                name: docker-sock
          - name: docker
            image: docker:latest
            resources:
              requests:
                memory: "100Mi"
                cpu: "50m"
              limits:
                memory: "300Mi"
                cpu: "200m"
            command:
            - cat
            tty: true
            volumeMounts:
              - mountPath: /var/run/docker.sock
                name: docker-sock
          volumes:
          - name: docker-sock
            hostPath:
              path: /var/run/docker.sock 
        '''
      }
    }
    stages 
    {       

        stage('Git-secret Checking')
        {
            steps
            {
                script
                {
                    container('git-secret') 
                    {
                        sh 'echo "Security check Staretd before build"'
                        echo "Checking for any secrets stored in the repo using git-secrets"
                        sh '''#!/bin/sh
                            git-secrets --scan -r $PWD
                            if [ "$?" -eq 1 ]; then
                                echo "CI Job Aborting as there is Secret Stored in the Repository."
                                exit 1
                            else 
                                echo "No Secret found in the code" 
                                echo "Proceed for next step....."                            
                            fi
                        '''
                    }
                }
            }
        }
    }
}