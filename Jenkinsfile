pipeline {
  agent any
 
  tools {
  maven 'Maven3'
  }
  stages {
    stage ('checkout') {
      steps {
          checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: '1d91fb0b-6361-4e1b-8f15-0bee71b9f8ab', url: 'https://github.com/dvzlic/MyjenkinsRepo']])
      }
    }
    stage ('build') {
        steps {
             sh 'mvn clean install -f MyWebApp/pom.xml'
      }
    }
    stage ('code quality') {
        steps {
            withSonarQubeEnv('SonarQube') {
        sh 'mvn -f MyWebApp/pom.xml sonar:sonar'
        }
    }
        }
            stage ('JaCoCo') {
      steps {
      jacoco()
      }
    }
    stage ('Nexus Upload') {
      steps {
      nexusArtifactUploader(
      nexusVersion: 'nexus3',
      protocol: 'http',
      nexusUrl: 'ec2-3-136-87-180.us-east-2.compute.amazonaws.com:8081',
      groupId: 'com.dept.app',
      version: '1.0-SNAPSHOT',
      repository: 'maven-snapshots',
      credentialsId: '60df56b4-3be8-43d9-a53e-1efe4041b0f9',
      artifacts: [
      [artifactId: 'MyWebApp',
      classifier: '',
      file: 'MyWebApp/target/MyWebApp.war',
      type: 'war']
      ])
    }
}
 stage ('DEV Deploy') {
      steps {
      echo "deploying to DEV Env "
      deploy adapters: [tomcat9(credentialsId: 'e9c75ead-d44d-4b65-a678-39188fbc7b43', path: '', url: 'http://ec2-18-223-184-7.us-east-2.compute.amazonaws.com:8080')], contextPath: null, war: '**/*.war'
      }
    }
    stage ('Slack Notification') {
      steps {
        echo "deployed to DEV Env successfully"
        slackSend(channel:'e', message: "Job is successful, here is the info - Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
}
}
    stage ('DEV Approve') {
      steps {
      echo "Taking approval from DEV Manager for QA Deployment"
        timeout(time: 7, unit: 'DAYS') {
        input message: 'Do you want to deploy?', submitter: 'admin'
        }
      }
    }
     stage ('QA Deploy') {
      steps {
        echo "deploying to QA Env "
        deploy adapters: [tomcat9(credentialsId: 'e9c75ead-d44d-4b65-a678-39188fbc7b43', path: '', url: 'http://ec2-18-223-184-7.us-east-2.compute.amazonaws.com:8080')], contextPath: null, war: '**/*.war'
        }
    }
    stage ('QA Approve') {
      steps {
        echo "Taking approval from QA manager"
        timeout(time: 7, unit: 'DAYS') {
        input message: 'Do you want to proceed to PROD?', submitter: 'admin,manager_userid'
        }
      }
    }
    stage ('Slack Notification for QA Deploy') {
      steps {
        echo "deployed to QA Env successfully"
        slackSend(channel:'e', message: "Job is successful, here is the info - Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
      }
    }  
  }
}
