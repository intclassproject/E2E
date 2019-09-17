pipeline{
    options{
        timeout(30)
    }
    agent{
        label 'slave'
    }
    stages{
        stage('checkout'){
            steps{
                script{
                    dir('Dev'){
                        deleteDir()
                        checkout([$class: 'GitSCM', branches: [[name: 'Dev']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'git-cred-id', url: "https://github.com/intclassproject/Release.git"]]])

                    }
                    dir('Infrastructure'){
                        deleteDir()


                    }
                }
            }
        }
    }
}