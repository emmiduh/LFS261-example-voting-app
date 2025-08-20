pipeline{
    agent none
    stages{
        stage("worker-build"){
            when{
                changeset "**/worker/**"
            }
            agent {
                docker{
                    image 'maven:3.8.7-openjdk-18-slim'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            steps{
                echo 'Compiling worker app..'
                dir('worker'){
                    sh 'mvn compile'
                }
            }
        }
        stage("worker-test"){
            when{
                changeset "**/worker/**"
            }
            agent {
                docker{
                    image 'maven:3.8.7-openjdk-18-slim'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            steps{
                echo 'Running unit tests on worker app..'
				dir('worker'){
					sh 'mvn clean test'
				}
            }
        }
        stage("worker-package"){
            when{ 
                changeset "**/worker/**"
            }
            agent {
                docker{
                    image 'maven:3.8.7-openjdk-18-slim'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
			steps{
                echo 'Packaging worker app into a jarfile'
				dir('worker'){
					sh 'mvn package -DskipTests'
		            archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
				}
            }
        }
        stage('worker-docker-package'){
            agent any
            when{
                changeset "**/worker/**"
                branch 'master'
            }
            steps{
                echo 'Packaging worker app with docker'
                script{
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') { 
                        def workerImage = docker.build("emmiduh93/worker:v${env.BUILD_ID}", "./ worker")
                        workerImage.push("${env.BRANCH_NAME}")
                        workerImage.push("latest")
                    }
                }
            }
        }
    

        stage('result-build'){
            agent{
                docker {
                    image 'node:24.5.0-trixie'
                }
            }
            when {
                changeset "**/result/**"
            }
            steps {
                echo 'Compiling result app..'
                dir('result'){
                    sh 'npm install'
                }
            }
        }
        
        stage('result-test'){
            agent{
                docker {
                    image 'node:24.5.0-trixie'
                }
            }
            when {
                changeset '**/result/**'
            }
            steps{
                echo 'Running Unit Tests on result app..'
                dir('result'){
                    sh '''npm install
                          npm test
                       '''
                }
            }
        }
        stage('result-docker-package') {
          agent any
          when {
            changeset '**/result/**'
            branch 'master'
          }
          steps {
            echo 'Packaging result app with docker'
            script {
              docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                def resultImage = docker.build("emmiduh93/result:v${env.BUILD_ID}", './result')
                resultImage.push()
                resultImage.push("${env.BRANCH_NAME}")
                resultImage.push('latest')
              }
            }
          }
        }

        stage('vote-build'){ 
            agent{
                docker{
                    image 'python:3.11-slim'
                    args '--user root'
                 }
            }
            when {
                 changeset '**/vote/**'
            }
            steps{ 
                echo 'Compiling vote app.' 
                dir('vote'){    
                    sh "pip install -r requirements.txt"
                } 
            } 
        } 
        stage('vote-test'){ 
           
            agent {
                docker{
                    image 'python:3.11-slim'
                    args '--user root'
                }
            }
            when {
                 changeset '**/vote/**'
            }
            steps{ 
                echo 'Running Unit Tests on vote app.' 
                dir('vote'){ 
                   
                        sh "pip install -r requirements.txt"
                        sh 'nosetests -v'
                        
                        
                } 
            } 
        }

        stage('vote integration'){ 
          agent any 
          when{ 
            changeset "**/vote/**" 
            branch 'master' 
          } 
          steps{ 
            echo 'Running Integration Tests on vote app' 
            dir('vote'){ 
                sh 'sh integration_test.sh' 
            } 
         } 
       }

        stage('vote-docker-package'){
            agent any
         
          steps{
            echo 'Packaging vote app with docker'
            script{
              docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                  // ./vote is the path to the Dockerfile that Jenkins will find from the Github repo
                  def voteImage = docker.build("emmiduh93/vote:v${env.BUILD_ID}", "./vote")
                  voteImage.push()
                  voteImage.push("${env.BRANCH_NAME}")
                  voteImage.push("latest")
              }
            }
          }
      }

      stage('deploy to dev'){
          agent any
          when{
            branch 'master'
          }
          steps{
            echo 'Deploy instavote app with docker compose'
            sh 'docker-compose up -d'
          }
      }


    }

    post{
        always{
            echo 'Building mono pipeline is completed..'
        }
    }
}
