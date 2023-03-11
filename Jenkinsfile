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
        stage('Clone Repo') 
        {
            steps 
            {
                git branch: AppBranchName, credentialsId: GitHubCredentialsId, url: GitURl
            }
        }
        // stage('Git-secret Checking')
        // {
        //     steps
        //     {
        //         script
        //         {
        //             container('git-secret') 
        //             {
        //                 sh 'echo "Security check Staretd before build"'
        //                 echo "Checking for any secrets stored in the repo using git-secrets"
        //                 sh '''#!/bin/sh
        //                     git-secrets --scan -r $PWD
        //                     if [ "$?" -eq 1 ]; then
        //                         echo "CI Job Aborting as there is Secret Stored in the Repository."
        //                         exit 1
        //                     else 
        //                         echo "No Secret found in the code" 
        //                         echo "Proceed for next step....."                            
        //                     fi
        //                 '''
        //             }
        //         }
        //     }
        // }
        // stage('Trufflehog Scanning')
        // {
        //     steps
        //     {
        //         script
        //         {
        //             container('trufflehog') 
        //             {
        //                 sh 'echo "trufflehog scanning started Staretd before build"'
        //                 echo "Checking for any secrets stored in the repo using trufflehog"
        //                 sh 'trufflehog filesystem --directory=$AppName --debug --json > trufflehog_report.json'
        //                 sh 'cat trufflehog_report.json'
        //                 archiveArtifacts allowEmptyArchive: true, artifacts: 'trufflehog_report.json', onlyIfSuccessful: true
        //             }
        //         }
        //     }
        // }
        // stage('Lint dockerfile Hadolint') 
        // {
        //     steps 
        //     {
        //         script
        //         {
        //             container('hadolint') 
        //             {
        //                 sh 'ls -lrt '
        //                 sh 'hadolint $AppHome/Dockerfile'
        //                 sh '''#!/bin/sh
        //                     if [ "$?" -eq 1 ]; then
        //                         echo "CI Job Aborting as there is vulnerability in the Dockerfile. Please check hadolint.txt file"
        //                         hadolint $AppHome/Dockerfile | tee -a hadolint_lint.txt
        //                         cat hadolint_lint.txt
        //                         exit 1
        //                     else 
        //                         echo "No Error( Warning/Critical) and untrusted registries detected in the Dockerfile"    
        //                         echo "Proceed for next step....." 
        //                         hadolint $AppHome/Dockerfile | tee -a hadolint_lint.txt 
        //                         cat hadolint_lint.txt            
        //                     fi

        //                 '''
        //                 archiveArtifacts allowEmptyArchive: true, artifacts: 'hadolint_lint.txt', onlyIfSuccessful: false
        //             }
        //         }
        //     }
        // }
        stage('NPM Build') 
        {
            steps
            {
                script
                {   
                    container('node')
                    {
                      sh '''#!/bin/sh
                        cd $AppHome
                        rm yarn.lock package-lock.json
                        yarn install && yarn cache clean
                      '''
                    }
                }
                
            }
        }
        stage('NPM Unit Test')
        {
          steps
           {
            script
                {
                    container('node')
                    {
                        try
                        {
                            sh '''#!/bin/sh
                              cd $AppHome 
                              yarn add nyc --dev
                              npm run coverage
                            '''
                        }
                        catch (exc)
                        {
                            echo 'Tests failed'
                            currentBuild.result = 'UNSTABLE'
                        }
                    }
                }
            }
        }
        stage("SonarQube") 
        {
            steps 
            {
                container('sonar') 
                {
                    withSonarQubeEnv('sonarserver') 
                    {
                        echo 'Performing Sonar Scan'

                        sh "${SonarHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=$AppHome-NodeApp \
                        -Dsonar.sonar.projectName=$AppHome-NodeApp \
                        -Dsonar.language=js \
                        -Dsonar.sonar.projectAppVersion=1.0 \
                        -Dsonar.sonar.sourceEncoding=UTF-8 \
                        -Dsonar.sonar.host.url=http://sonar.nativek8s-test.com \
                        -Dsonar.login=squ_933cd1f17c6a926c61be916406cca1c5b08264f7 \
                        -Dsonar.sourceEncoding=UTF-8 \
                        -Dsonar.javascript.lcov.reportPaths=$AppHome/coverage/lcov.info \
                        -Dsonar.coverage.exclusions=$SonarCoverageExclusionsPath \
                        -Dsonar.exclusions=$SonarCodeExclusions"
                    }
                }
            }
        }
        stage("SonarQube Quality gate") 
        {
            steps 
            {
                script 
                {
                    def qualitygate = waitForQualityGate()
                    sleep(10)
                    if (qualitygate.status != "OK") 
                    {
                        waitForQualityGate abortPipeline: true
                    }
                }
            }
        }
        // stage('Dependency Check (SAST)') 
        // {
        //   steps 
        //     {   
                
        //         sh '''ls -lrt'''
        //         dependencyCheck additionalArguments: "--cveValidForHours 48 --disableYarnAudit --disableNodeAudit --disableNodeAudit --prettyPrint --scan $AppHome --format ALL ${fileExists('owasp-suppression.xml') ? "--suppression owasp-suppression.xml" : ""} --disableAssembly", odcInstallation: 'Dependency-check-v7.1.1'
        //         dependencyCheckPublisher pattern: ''
        //         archiveArtifacts allowEmptyArchive: true, artifacts: '**/dependency-check-report.html', onlyIfSuccessful: false
        //     }
        // }
        stage('Building Application Docker Image') 
        {
            steps 
            {   
                script
                {
                    container('docker') 
                    {
                        sh 'ls -lrt'
                        sh 'docker build --network=host -t $ImageRepoName:latest $AppHome/.'
                    }
                }
            }
        }
        // stage("Scanning Docker Image with Trivy")
        // {
        //     steps
        //     {
        //         container('trivy')
        //         {
        //             script 
        //             {
        //                 sh  'trivy image -f table -o trivy-results --exit-code 1 --severity CRITICAL,HIGH,MEDIUM,LOW $ImageRepoName:latest' 
        //                 archiveArtifacts allowEmptyArchive: true, artifacts: 'trivy-results', onlyIfSuccessful: false                       
        //             }
                    
        //         }
        //     }
        // }   
        stage('Push Docker Image to ECR')
        {        
            steps
            {
                script
                {
                    
                    container('mcds-client')
                    {
                        withAWS(credentials: 'AWS', region: Region)   // use aws as env
                        {
                            sh '''#!/bin/sh
                                aws ecr get-login-password --region '''+Region+''' | docker login --username AWS --password-stdin $ContainerImageRegistry 
                                if [[ $? == 0 ]]; then
                                    echo "ECR login success"
                                else
                                    echo "Unable to Login to ECR" 
                                    exit 1
                                fi
                                echo "Starting Docker Push" 2>&1 
                                docker tag '''+ImageRepoName+''':'latest' '''+ContainerImageRegistry+'''/'''+ImageRepoName+''':'''+AppVersion+'''
                                docker push '''+ContainerImageRegistry+'''/'''+ImageRepoName+''':'''+AppVersion+'''
                                if [ "$?" -eq 1 ]; then
                                    echo "Docker image Push to failed" 
                                    exit 1
                                else 
                                    echo "Docker Image pushed to ECR successfully" 
                                fi
                                docker tag '''+ContainerImageRegistry+'''/'''+ImageRepoName+''':'''+AppVersion+''' '''+ContainerImageRegistry+'''/'''+ImageRepoName+''':'latest'
                                docker push '''+ContainerImageRegistry+'''/'''+ImageRepoName+''':'latest'
                                if [ "$?" -eq 1 ]; then
                                    echo "Docker image Push to failed" 
                                    exit 1
                                else 
                                    echo "Docker Image pushed to ECR successfully" 
                                fi
                            '''
                        }
                    }              
                }
            }
        }
        stage('Building Helm Chart Values')
        {
            steps
            {
                container('yq')
                {      
                    sh '''#!/bin/sh
                        DIR="'''+HelmChartName+'''"
                        echo "'''+HelmChartName+'''"
                        if [ -d "$DIR" ]; then
                            ### Take action if $DIR exists ###
                            echo "Helm chart dir is avilable : ${DIR} proceding further"
                            export HELM_EXPERIMENTAL_OCI=1
                            ls -lrt "'''+HelmChartName+'''"
                            cd "'''+HelmChartName+'''"
                            yq e -i '.image.repository = "'''+ContainerImageRegistry+'''/'''+ImageRepoName+'''"' ./values.yaml
                            yq e -i '.image.tag = "'''+AppVersion+'''"' ./values.yaml
                            yq e -i '.replicaCount = "2"' ./values.yaml 
                            cat ./values.yaml
                            cd ../
                        else 
                            echo "Helm chart dir is not avilable"
                            currentBuild.result="FAILURE"
                        fi
                    '''                
                }
            }
        } 
        stage('Building helm Chart to Push Helm Chart to ECR')
        {
            steps
            {
                container('mcds-client')
                {
                    withAWS(credentials: 'AWS', region: Region) 
                    {
                        sh '''#!/bin/sh
                            ls -rlt
                            export HELM_EXPERIMENTAL_OCI=1
                            aws ecr get-login-password --region us-east-2 | helm registry login --username AWS --password-stdin '''+ContainerImageRegistry+'''
                            if [[ $? == 0 ]]; then
                                echo "ECR Helm login success"
                            else
                                echo "Unable to Login to ECR" 
                                exit 1
                            fi
                            helm version
                            kubectl version
                            export HELM_EXPERIMENTAL_OCI=1
                            helm template "'''+HelmChartName+'''"
                            helm lint "'''+HelmChartName+'''"
                            helm package "'''+HelmChartName+'''" --version '''+ChartVersion+'''
                            helm push "'''+HelmChartName+'''-'''+ChartVersion+'''".tgz oci://'''+ContainerImageRegistry+'''
                            if [[ $? == 0 ]]; then
                                echo "Helm Chart Successfully pushed to Help Repo"
                            else
                                echo "Unable to push Helm chart to ECR Helm Repo" 
                                exit 1
                            fi
                        ''' 
                    }
                }    
            }
        } 
    }
}
