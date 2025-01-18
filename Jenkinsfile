pipeline {
    agent {
        label 'debian-jdk21'
    }

    environment {
        LITTLEONI_MVN = credentials('littleoni-mvn')
    }

    stages {
        stage ('Get Branch') {
            steps {
                git branch: 'main', url: 'https://github.com/Escape-Systems/Cuneiform.git'
            }
        }
        stage('Apply Patches') {
            steps {
                sh 'git config --global user.name "Jenkins CI"'
                sh 'git config --global user.email "lostletters@t-ch.net"'
                sh './gradlew applyAllPatches'
                findBuildScans()
            }
        }

        parallel {
            stage('Generate Development Bundle') {
                steps {
                    sh './gradlew generateDevelopmentBundle '
                    findBuildScans()
                    archiveArtifacts artifacts: 'cuneiform-server/build/libs/paperweight-development-bundle-*.zip', followSymlinks: false
                }
            }
            stage('Generate Paperclip Jar') {
                steps {
                    sh './gradlew createMojmapPaperclipJar'
                    findBuildScans()
                    archiveArtifacts artifacts: 'cuneiform-server/build/libs/cuneiform-paperclip-*-mojmap.jar', followSymlinks: false
                }
            }
        }

        stage('Publish') {
            steps {
                sh './gradlew '+
                        '-PlittleOniSnapshotsPassword=$LITTLEONI_MVN_PSW ' +
                        '-PlittleOniSnapshotsUsername=$LITTLEONI_MVN_USR ' +
                        '-PpublishDevBundle '+
                        'publishAllPublicationsToLittleOniSnapshotsRepository'
                findBuildScans()
            }
        }
    }
}