pipeline {
  agent any
  environment
  {    //it was in every stage
    IMAGE_NAME = 'nexus.teamdigitale.test/kafka-consumer'
  }
  stages {
    stage('Build') {
      steps {
        script {
        slackSend (message: "BUILD START: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' CHECK THE RESULT ON: https://cd.daf.teamdigitale.it/blue/organizations/jenkins/CI-Kafka-Consumer/activity")                  
        sh 'COMMIT_ID=$(echo ${GIT_COMMIT} | cut -c 1-6); docker build . -t $IMAGE_NAME:$BUILD_NUMBER-$COMMIT_ID'
        }
      }
    }
    stage('Test') {
      steps { //sh' != sh'' only one sh command
      script {
        sh '''
	COMMIT_ID=$(echo ${GIT_COMMIT} | cut -c 1-6);
        CONTAINERID=$(docker run -d -p 3000:3000 $IMAGE_NAME:$BUILD_NUMBER-$COMMIT_ID);
        sleep 5s;
        docker stop ${CONTAINERID};
        docker rm ${CONTAINERID}
	'''
      }
    }
    }
    stage('Upload'){
      steps {
        script {
          if(env.BRANCH_NAME == 'citest'  || env.BRANCH_NAME == 'master' ){
            sh 'COMMIT_ID=$(echo ${GIT_COMMIT} | cut -c 1-6); docker push $IMAGE_NAME:$BUILD_NUMBER-$COMMIT_ID'
            sh 'COMMIT_ID=$(echo ${GIT_COMMIT} | cut -c 1-6); docker rmi $IMAGE_NAME:$BUILD_NUMBER-$COMMIT_ID'
          }
        }
      }
    }
    stage('Staging') {

      steps {
        script {
            if(env.BRANCH_NAME=='citest' || env.BRANCH_NAME == 'master'){
                //  sed "s#image: nexus.teamdigitale.test/kafka.*#image: nexus.teamdigitale.test/kafka-consumer:$BUILD_NUMBER-$COMMIT_ID#" kafka-consumer.yaml > kafka-consumer1.yaml
                sh ''' COMMIT_ID=$(echo ${GIT_COMMIT}|cut -c 1-6);
                cd kubernetes/test;
                sed "s#image: nexus.teamdigitale.test/kafka.*#image: nexus.teamdigitale.test/kafka-consumer:$BUILD_NUMBER-$COMMIT_ID#" kafka-consumer.yaml > kafka-consumer$BUILD_NUMBER.yaml;
                kubectl --kubeconfig=${JENKINS_HOME}/.kube/config.teamdigitale-staging delete -f kafka-consumer$BUILD_NUMBER.yaml || true;
                kubectl --kubeconfig=${JENKINS_HOME}/.kube/config.teamdigitale-staging create -f kafka-consumer$BUILD_NUMBER.yaml '''
                slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' https://cd.daf.teamdigitale.it/blue/organizations/jenkins/CI-Kafka-Consumer/activity")
          }
        }
      }
    }
  }
  post { 
        failure { 
            slackSend (color: '#ff0000', message: "FAIL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' https://cd.daf.teamdigitale.it/blue/organizations/jenkins/CI-Kafka-Consumer/activity")
        }
    }
}
