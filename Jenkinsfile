pipeline {
    agent any

    environment {
        // Configuración Nexus
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "localhost:8081"
        NEXUS_REPOSITORY = "RepositorioPrueba"
        NEXUS_CREDENTIAL_ID = "Nexus_Credential"
    }

    stages {
        stage('inicio') {
            steps {
                echo "Inicio"
            }
        }

        stage('Contruccion') {
            steps {
                echo "Construccion"
                bat 'mvn -B package'
            }
        }

        stage('Test') {
            steps {
                echo "Test"
                bat 'mvn clean verify'
            }
        }

        stage('SonarQube analysis') {
            environment {
                SCANNER_HOME = tool 'Sonarqube Scan'
            }
            steps {
                withSonarQubeEnv(credentialsId: 'SecretSonarQube', installationName: 'SonarQube') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=EjercicioM3-L6 \
                        -Dsonar.projectName=EjercicioM3-L6 \
                        -Dsonar.sources=src/ \
                        -Dsonar.java.binaries=target/classes/ \
                        -Dsonar.exclusions=src/test/java/****/*.java \
                        -Dsonar.projectVersion=${BUILD_NUMBER}-${GIT_COMMIT_SHORT}'''
                }
            }
        }

        stage("Publish to Local Nexus Repository Manager") {
            steps {
                script {
                    echo "Iniciando Nexus"
                    pom = readMavenPom file: "pom.xml"
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path
                    artifactExists = fileExists artifactPath
                    if (artifactExists) {
                        echo "***  NEXUS - Archivo: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}"
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        )
                    } else {
                        error "*** NEXUS - Archivo: ${artifactPath}, no fue encontrado"
                    }
                }
            }
        }
    }
    post {
        success {
            slackSend(channel: "#pruebasjenkins", token: "drmtM0r1aB8CgO0Sc4Fh0ZZN", color: "good", message: "funcionó")
        }
    }
}
