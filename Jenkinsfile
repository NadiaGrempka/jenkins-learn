pipeline {
    agent any

    environment {
        TARGET_DIR    = 'target'
        CLASS_DIR     = "${TARGET_DIR}/classes"
        TEST_DIR      = "${TARGET_DIR}/test-classes"
        REPORT_DIR    = "${TARGET_DIR}/reports"
        GIT_CRED_ID   = 'git-credentials-id'
        // API_KEY     = credentials('api-key-id')
    }

    stages {

    stage('Checkout') {
          steps {
            script {
            					def scmVars = checkout scm
                        echo "Checked out branch: ${scmVars.GIT_BRANCH}"
                    		}
          }
        }



        stage('Validation') {
        			steps {
        				script {
        					for (d in ['src/main/java','src/test/java','lib']) {
        						if (!fileExists(d)) {
        							error("❗️Brak katalogu ${d}")
                                }
                            }
                        }
                    }
                }


        stage('Build') {
            steps {
                sh '''
                  mkdir -p ${CLASS_DIR} ${TEST_DIR} ${REPORT_DIR}
                  javac -cp "lib/*" -d ${CLASS_DIR} $(find src/main/java -name '*.java')
                  javac -cp "lib/*:${CLASS_DIR}" -d ${TEST_DIR} $(find src/test/java -name '*.java')
                '''
            }
            post {
                success {
                    stash includes: "${CLASS_DIR}/**,${TEST_DIR}/**,lib/**",
                          name: 'classes-and-lib'
                }
            }
        }

        stage('Test') {
            steps {
                unstash 'classes-and-lib'
                script {
                    try {
                        sh '''
                          java -jar lib/junit-platform-console-standalone-*.jar \\
                            --class-path ${CLASS_DIR}:${TEST_DIR}:lib/* \\
                            --scan-class-path \\
                            --reports-dir=${REPORT_DIR} \\
                            --report-format=xml
                        '''
                    } finally {
                        junit "${REPORT_DIR}/*.xml"
                    }
                }
            }
        }

        stage('Package') {
            when { branch 'main' }
            steps {
                sh "jar cf app-${env.BUILD_ID}.jar -C ${CLASS_DIR} ."
            }
        }

        stage('Archive') {
            when { branch 'main' }
            steps {
                archiveArtifacts artifacts: "app-${env.BUILD_ID}.jar, ${REPORT_DIR}/*.xml",
                                 fingerprint: true
            }
        }
    }

    post {
        always {
            echo "Koniec potoku, status: ${currentBuild.currentResult}"
        }
    }
}
