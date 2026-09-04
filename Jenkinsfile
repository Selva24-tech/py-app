pipeline {
    agent any
    options {
        skipStagesAfterUnstable()
    }
    stages {
        stage('Build') {
            steps {
                sh 'python3 -m py_compile sources/add2vals.py sources/calc.py'
            }
        }
        stage('Test') {
            steps {
                sh 'python3 -m pytest --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }
        stage('Deliver') {
            steps {
                sh 'python3 -m PyInstaller --onefile sources/add2vals.py'
            }
            post {
                success {
                    archiveArtifacts 'dist/add2vals'
                }
            }
        }
        stage('Deploy to AWS') {
            steps {
                sshagent(credentials: ['sel2']) {
                    sh '''
                        echo "Copying binary to AWS EC2 instance..."
                        
                        # Copy the compiled binary via SCP to your EC2 instance
                        scp -o StrictHostKeyChecking=no dist/add2vals ec2-user@54.253.129.59:/tmp/add2vals
                        
                        # Move the binary to a system path and set permissions on the remote server
                        ssh -o StrictHostKeyChecking=no ec2-user@54.253.129.59 'sudo mv /tmp/add2vals /usr/local/bin/add2vals && sudo chmod +x /usr/local/bin/add2vals'
                        
                        echo "Deployment to AWS completed successfully!"
                    '''
                }
            }
        }
    }
}
