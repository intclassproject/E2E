import groovy.json.JsonSlurper
import hudson.model.*
def path_dev_json_file
def path_prod_json_file
def services
def docker_prod_repo = 'devopsint/prod'
def underscore = '_'
def colons = ':'
def template_ip = '_IP'
def hosts_file_name = 'hosts'
def playbook_file = 'main.yml'
pipeline{
    options{
        timeout(30)
    }
    agent{
        label 'master'
    }
    stages{
        stage('checkout'){
            steps{
                script{
                    dir('Dev'){
                        deleteDir()
                        checkout([$class: 'GitSCM', branches: [[name: 'Dev']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'git-cred-id', url: "https://github.com/intclassproject/Release.git"]]])
                        path_dev_json_file = sh(script: "pwd", returnStdout: true).trim() + '/' + 'Dev' + '.json'


                    }
                    dir('Prod'){
                        deleteDir()
                        checkout([$class: 'GitSCM', branches: [[name: 'Prod']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'git-cred-id', url: "https://github.com/intclassproject/Release.git"]]])
                        path_prod_json_file = sh(script: "pwd", returnStdout: true).trim() + '/' + 'Prod' + '.json'
                    }
                    dir('Infrastructure'){
                        deleteDir()
                        checkout([$class: 'GitSCM', branches: [[name: 'Dev']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'git-cred-id', url: "https://github.com/intclassproject/Infrastructure.git"]]])

                            }
                    }
                }
            }
        stage('Deploy services'){
            steps{
                script{
                    dir('Infrastructure'){
                        dir('ansible') {
                            services = Return_Json_From_File("$path_dev_json_file").release.services.keySet()
                            servers = Return_Json_From_File("$path_dev_json_file")["release"]["services"]["$params.triggered_by"]["servers"]
                            for (server in servers){
                                sh "cp $hosts_file_name ${hosts_file_name}.bak"
                                sh "sed -i 's/\\$params.triggered_by$template_ip\\>/$server/' $hosts_file_name"
                                sh """
                                            ansible-playbook -vvvvv -i hosts  -e "service=$params.triggered_by image_name=$params.Image_version"  $playbook_file
                               """
                                sh "mv ${hosts_file_name}.bak $hosts_file_name "
                            }
                            for (service in services){
                                servers = Return_Json_From_File("$path_dev_json_file")["release"]["services"]["$service"]["servers"]
                                for (server in servers){
                                    sh "cp $hosts_file_name ${hosts_file_name}.bak"
                                   if(service != params.triggered_by){
                                        stable_version = Return_Json_From_File("$path_prod_json_file")["release"]["services"]["$service"]["version"]
                                        image_name = docker_prod_repo + colons + service + underscore + stable_version
                                        sh "sed -i 's/\\$service$template_ip\\>/$server/' $hosts_file_name"
                                       try{
                                           sh """
                                            ansible-playbook -vvvvv -i hosts  -e "service=$service image_name=$image_name"  $playbook_file
                                        """
                                       }
                                       catch (exception){
                                           println("Deploy services is failed")
                                       }

                                   }
                                    sh "mv ${hosts_file_name}.bak $hosts_file_name "
                                }

                            }


                        }
                    }

                }
            }
        }
//        stage('QA tests'){
//            steps{
//                script{
//
//                }
//            }
//        }
        }
    }
def Return_Json_From_File(file_name){
    return new JsonSlurper().parse(new File(file_name))
}