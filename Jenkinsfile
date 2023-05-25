
pipeline {
    agent any

    environment {
        //PREVIOUS_BUILDN = "${BUILD_NUMBER.toInteger() - 1}"
        PREVIOUS_BUILDN = "0"
    }

    stages {

        stage('Parallel stage to fetch code and file verification'){
            parallel{
                stage('Fetch code from Computers Repo') {
                    steps {
                        echo '---------------STARTING PIPELINE---------------------'
                        echo '---------------FETCHING CODE FROM DEV BRANCH---------'
                        git branch: 'main', url: 'https://github.com/VallDev/ComputersDeploy.git'
                    }
                }

                stage('File verification') {
                    steps {
                        echo '----------------FILE .tar VERIFICATION---------------'
                        sh '''
                            TAR_DIR=./computers-go.tar
                            if [ -e $TAR_DIR ]; then
                                rm ./computers-go.tar
                                echo "-------------.tar FILE HAS BEEN SUCCESSFULLY DELETED-"
                            else
                                echo "-------------.tar FILE DOES NOT EXIST--------------"
                            fi
                        '''
                    }
                }
            }
        }

        stage('Building Go app and Docker image') {
            steps {
                echo '---------------BUILDING DOCKER IMAGE-----------------'
                sh 'docker build --network=host -t computers-go:${BUILD_NUMBER} .'
            }
        }

        stage('Saving Image') {
            steps {
                echo '---------------SAVING IMAGE IN .tar FILE-------------'
                sh 'docker save computers-go -o computers-go.tar'
            }
        }

        stage('Deploy in Configuration') {
            steps {
                echo '---------------DEPLOYING APP-------------------------'
                sh 'scp -v -o StrictHostKeyChecking=no computers-go.tar ubuntu@44.201.78.229:/home/ubuntu/ImagesToSend'
                echo '-------------SAVING NUMBER OF CURRENT BUILD----------'
                sh  "ssh ubuntu@44.201.78.229 'cd ImagesToSend && echo ${BUILD_NUMBER} > current-build-number' "
            }
        }


        /*stage('Rebuilding Image, stopping and deleting previous image'){
            parallel{
                stage('Stopping and Deleting previous Image') {
                    steps {
                        //echo '-------------GETTING NUMBER OF IMAGE TAG--------'
                        //sh " ssh andres@192.168.10 'cd ImagesToRun && ls -la && PREVIOUS_BUILDN=\$(cat build-number) && echo \$PREVIOUS_BUILDN\' "
                        echo '-------------STOPING SERVICE OF PREVIOUS IMAGE--'
                        sh  " ssh andres@192.168.0.10 'cd ImagesToRun && docker stop \$(docker ps -q --filter ancestor=computers-go:\$(cat build-number))'" 
                        echo '-------------DELETING PREVIOUS IMAGE------------'
                        sh  "ssh andres@192.168.0.10 'cd ImagesToRun && docker image rmi computers-go:\$(cat build-number)' " 
                     }
                }

                stage('Rebuilding the image from .tar format') {
                    steps {
                        echo '----------------CHANGING .tar FILE TO A DOCKER IMAGE--'
                        sh "ssh andres@192.168.0.10 'cd ImagesToRun && docker load -i computers-go.tar'"
                    }       
                }
            }
        }*/

        /*stage('Runing Docker Image and Saving Tag') {
            steps {
                echo '---------------RUNING DOCKER IMAGE-------------------'
                sh "ssh andres@192.168.0.10 'cd ImagesToRun && docker run -d -p 8080:8080 -t computers-go:${BUILD_NUMBER}'"
                echo '-------------SAVING NUMBER OF IMAGE TAG----------'
                sh  "ssh andres@192.168.0.10 'cd ImagesToRun && echo ${BUILD_NUMBER} > build-number' " 

            }
        }*/
        
        stage('Runing 3 scripts to deploy and run Docker Images'){
            parallel{
                stage('Runing script of Back1') {
                    steps {
                        echo '---------------STARTING SCRIPT BACK1-------------------'
                        sh "ssh ubuntu@44.201.78.229 'cd ImagesToSend && bash back1-script.sh'"
                    }
                }

                stage('Runing script of Back2') {
                    steps {
                        echo '---------------STARTING SCRIPT BACK1-------------------'
                        sh "ssh ubuntu@44.201.78.229 'cd ImagesToSend && bash back2-script.sh'"
                    }
                }

                stage('Runing script of Back3') {
                    steps {
                        echo '---------------STARTING SCRIPT BACK1-------------------'
                        sh "ssh ubuntu@44.201.78.229 'cd ImagesToSend && bash back3-script.sh'"
                    }
                }
            }

    }

    post{
        always{
            echo '-------------SENDING MESSAGE TO DISCORD CHANNEL ANDRES'                                                                                                                                             
            discordSend description: 'Computers API Project by Andrés', footer: "Build Number:${BUILD_NUMBER}", link: env.BUILD_URL, result: currentBuild.currentResult, title: JOB_NAME, thumbnail:'https://upload.wikimedia.org/wikipedia/commons/thumb/d/d7/Desktop_computer_clipart_-_Yellow_theme.svg/220px-Desktop_computer_clipart_-_Yellow_theme.svg.png' , webhookURL: 'https://discord.com/api/webhooks/1111022539993522296/Dyulm13hj0Clo0EBGxKK08Pzglal8GmARld80rXc-opc9O-jC_w_A74Q_rS3QbjtfUjU'
            echo '---------------FINISHING PIPELINE--------------------'
        }
    }
}
