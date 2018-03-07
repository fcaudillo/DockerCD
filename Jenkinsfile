pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Building...'
                dir ('android/'){
                    sh 'echo sdk.dir=$ANDROID_HOME > local.properties'
                    sh 'yes | $ANDROID_HOME/tools/bin/sdkmanager --licenses'
                    sh 'cp -R $ANDROID_HOME/licenses licenses/'
                    sh './gradlew build clean'
                }
            }
        }
        stage('Generate Apk') {
            steps {
                dir ('android/'){
                    sh './gradlew assembleDebug'
                }
            }
        }
        stage('Unit test'){
            steps {
                dir ('android/'){
                    sh './gradlew test'
                }
            }
        }
        stage('Expresso test') {
            agent {
                docker {
                    image 'butomo1989/docker-android-x86-7.1.1'
                    args '--privileged -p 6080:6080 -p 5554:5554 -p 5555:5555 -e "DEVICE=Samsung Galaxy S6"'
                }
            }
            steps {
                docker.image('butomo1989/docker-android-x86-7.1.1').run('--privileged -p 6080:6080 -p 5554:5554 -p 5555:5555 --env "DEVICE=Samsung Galaxy S6"');
                sh '$ANDROID_HOME/platform-tools/adb kill-server'
                sh '$ANDROID_HOME/platform-tools/adb start-server'
                sh '$ANDROID_HOME/platform-tools/adb wait-for-device shell \'while [[ -z $(getprop sys.boot_completed) ]]; do sleep 1; done\''
                dir ('android/'){
                    sh './gradlew connectedAndroidTest'
                }
            }
        }
        stage('Publish') {
            steps {
                sh 'curl "https://dashboard.applivery.com/api/builds" \
                    -X POST \
                    -H "Authorization:42c00e9014a6d943c1e9fe75ed0050de253a3620" \
                    -F app="5a906745ccc70a0d57513329" \
                    -F versionName="Test version name" \
                    -F notes="Bug fixing" \
                    -F notify="true" \
                    -F os="android" \
                    -F tags="tag1" \
                    -F package=@"$WORKSPACE/android/app/build/outputs/apk/debug/app-debug.apk"'
            }
        }
    }
    post { 
        always { 
            deleteDir()
            sh 'docker rm -f $(docker ps -aq)'
        }
        success {
            echo 'I succeeeded!'
        }
        unstable {
            echo 'I am unstable :/'
        }
        failure {
            echo 'I failed :('
        }
        changed {
            echo 'Things were different before...'
        }
    }
}