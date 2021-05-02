 def getDockerTag(){
        def tag = sh script: 'git rev-parse HEAD', returnStdout: true
        return tag
        }
        

pipeline{
        agent any  
        environment{
	    Docker_tag = getDockerTag()
        }
        
        stages{

            stage('Clone Sources')
                {
              steps{
                  git url : 'https://github.com/ravipratap520/sample.git'
                    }
                 }


              stage('Quality Gate Statuc Check'){

               agent {
                docker {
                image 'maven'
                args '-v $HOME/.m2:/root/.m2'
                }
            }
                  steps{
                      script{
                      withSonarQubeEnv('sonarserver') { 
                      sh "mvn sonar:sonar"
                       }
                      timeout(time: 1, unit: 'HOURS') {
                      def qg = waitForQualityGate()
                      if (qg.status != 'OK') {
                           error "Pipeline aborted due to quality gate failure: ${qg.status}"
                      }
                    }
		    sh "mvn clean install"
                  }
                }  
              }

              stage('Publish over SSH'){
			           steps{
                   sshagent(['deploy_user']){
                     scp webapp/target/*.war ec2-user@13.233.190.102://opt//playbooks/
                   }
                 			        }
		        }
    
		 
		        stage('ansible playbook'){
			           steps{
				                 ansiblePlaybook credentialsId: 'private-key', become: true, installation: 'ansible', inventory: 'hosts', playbook: 'tomcat.yml'
			        }
		        }
		
	
		
               }
	       
	       
	       
	      
    
}