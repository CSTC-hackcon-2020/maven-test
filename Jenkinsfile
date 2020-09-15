pipeline{

      environment{
        ORIGIN_REPO =  'registry-vpc.cn-shanghai.aliyuncs.com/hackathonxx'
        IMAGE =  'maven-test'
        IMAGE_TAG =  sh(returnStdout: true,script: 'echo $branch-$buildnumber').trim()
        CLUSTER = 'https://kubernetes.default.svc.cluster.local:443'
      }

      agent{
        node{
          label 'slave-pipeline'
        }
      }

      stages{

        stage('Package'){
          steps{
              container("maven") {
                  sh "mvn package -B -DskipTests"
              }
          }
        }

        stage('Image Build And Publish'){
          steps{
              container("kaniko") {
                  sh "kaniko -f `pwd`/Dockerfile -c `pwd` --destination=${ORIGIN_REPO}/${IMAGE}:${IMAGE_TAG} --skip-tls-verify"
              }
          }
        }

        stage('Deploy to Kubernetes') {
          when { branch 'master' }
          steps {
            container('kubectl') {
              step([$class: 'KubernetesDeploy', authMethod: 'certs', apiServerUrl: '${CLUSTER}', credentialsId:'k8sCertAuth', config: 'deployment.yaml',variableState: 'ORIGIN_REPO,IMAGE,IMAGE_TAG'])
            }
          }
        }
      }
}
