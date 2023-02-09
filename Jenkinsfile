/**
* Declarative pipeline
* https://jenkins.io/doc/book/pipeline
*/

def scan_type
def target
pipeline {
    agent any
    parameters {
        choice  choices: ["Baseline", "APIS", "Full"],
            description: 'Type of scan that is going to perform inside the container',
            name: 'SCAN_TYPE'

        string defaultValue: "10.0.0.0",
            description: 'Target IP to scan (Don\'t add the protocol or the port)',
            name: 'TARGET'

        booleanParam defaultValue: true,
            description: 'Parameter to know if a report should be generated.',
            name: 'GENERATE_REPORT'
    }
    stages {
        stage('Pipeline Info') {
            steps {
                script {
                    echo "<--Parameter Initialization-->"
                    echo """
                    The current parameters are:
                        Scan Type: ${params.SCAN_TYPE}
                        Target: ${params.TARGET}
                        Generate report: ${params.GENERATE_REPORT}
                    """
                }
            }
        }

        stage('Setting up OWASP ZAP docker container') {
            steps {
                script {
                    /*echo "Pulling up last OWASP ZAP container --> Start"
                    sh 'docker pull owasp/zap2docker-stable'
                    echo "Pulling up last VMS container --> End"
                    echo "Starting container --> Start"
                    sh """
                    */
                    docker run -dt --name owasp \
                    owasp/zap2docker-stable \
                    /bin/bash
                    """
                }
            }
        }


        stage('Prepare wrk directory') {
            when {
                environment name : 'GENERATE_REPORT', value: 'true'
            }
            steps {
                script {
                    sh """
                        docker exec owasp \
                        mkdir /zap/wrk
                    """
                }
            }
        }


        stage('Scanning target on owasp container') {
            steps {
                script {
                    scan_type = "${params.SCAN_TYPE}"
                    echo "----> scan_type: $scan_type"
                    target = "https://${params.TARGET}:8080"
                    if(scan_type == "Baseline"){
                        sh """
                            docker exec owasp \
                            zap-baseline.py \
                            -t $target \
                            -x report.xml \
                            -I
                        """
                    }
                    else if(scan_type == "APIS"){
                        sh """
                            docker exec owasp \
                            zap-api-scan.py \
                            -t $target \
                            -x report.xml \
                            -I
                        """
                    }
                    else if(scan_type == "Full"){
                        sh """
                            docker exec owasp \
                            zap-full-scan.py \
                            -t $target \
                            -x report.xml \
                            -I
                        """
                        //-x report-$(date +%d-%b-%Y).xml
                    }
                    else{
                        echo "Something went wrong..."
                    }
                }
            }
        }
        stage('Copy Report to Workspace'){
            steps {
                script {
                    sh '''
                        docker cp owasp:/zap/wrk/report.xml ${WORKSPACE}/report.xml
                    '''
                }
            }
        }
    }
    post {
        always {
            echo "Removing container"
            sh '''
                docker stop owasp
                docker rm owasp
            '''
        }
    }
}