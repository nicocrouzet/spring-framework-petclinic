pipeline {
    agent any
    tools {
        maven "maven 3.6"
    }
     environment {
        NEXUS_ARTIFACT_VERSION= "${env.BUILD_NUMBER}"
    }

    options {
        parallelsAlwaysFailFast()
    }
    stages {
        stage('Non-Parallel Stage') {
            steps {
                echo 'This stage will be executed first.'
            }
        }
        stage('Parallel Stage') {
            parallel {
                   stage('Checkstyle') {
                        steps{
                            // Run the maven build with checkstyle
                            echo "remove checkstyle because too long"
//                            sh "mvn clean package checkstyle:checkstyle"
                         }
                     }
                    stage('Sonarqube') {
                        steps {
                            withSonarQubeEnv('SonarQube') {
//                            sh "mvn  clean package sonar:sonar -Dsonar.host_url=$SONAR_HOST_URL "
                            }
                         }
                    }
            }
        }
        stage('Publish in Nexus') {
            steps {
                nexusPublisher nexusInstanceId: 'Nexus',
                nexusRepositoryId: 'releases',
                packages: [[$class: 'MavenPackage',
                mavenAssetList: [[classifier: '', extension: '', filePath: 'target/petclinic.war']], mavenCoordinate: [artifactId: 'spring-framework-petclinic', groupId: 'org.springframework.samples', packaging: 'war', version: NEXUS_ARTIFACT_VERSION]]]
            }
        }
        stage('Build image') {
            steps{
                script{
                    def customImage = docker.build("petclinic-project")
                }
            }
        }
//        stage('Run Test image non parallel') {
//            steps{
//                echo "no run image as it does not work"
////                   value = "docker ps --all --quiet --filter=name=petclinic-test"
////                    def val= sh (returnStdout: true, script: command)
////                  //def value = "docker ps --all --quiet --filter=name='petclinic-test'".execute()
////                  //echo "value = $value.text"
////                  if ($val.text)
////                  {
////                    def stop_container="docker stop petclinic-test".execute()
////                    def rm_container="docker rm petclinic-test".execute()
////                  }
//               }
////               sh 'docker run -d --name petclinic-test -p 8090:8080 petclinic-project'
//            }
//        }

        stage('Run Test image Parallel 1') {
            parallel {
                   stage('Run 1') {
                        steps{
                            // Run the maven build with checkstyle
                            echo "Run parallel 1"
                            value = sh "docker ps --all --quiet --filter=name='petclinic-test'"
                            echo "value = $value.text"
                            if ($val.text)
                            {
                                sh "docker stop petclinic-test && docker rm petclinic-test"
                                sh 'docker run -d --name petclinic-test -p 8090:8080 petclinic-project'
                            }
                         }
                     }
                   stage('Run 2') {
                        steps{
                            // Run the maven build with checkstyle
                            echo "Run parallel 2"
                            sh "docker stop petclinic-nico && docker rm petclinic-nico"
                            sh 'docker run -d --name petclinic-nico -p 8190:8080 petclinic-project'
                         }
                     }
                   stage('Run 3') {
                        steps{
                            // Run the maven build with checkstyle
                            echo "Run parallel 3"
                            sh "docker stop petclinic-uat && docker rm petclinic-uat"
                            sh 'docker run -d --name petclinic-uat -p 8290:8080 petclinic-project'
                         }
                   }
            }
        }
    }
}
