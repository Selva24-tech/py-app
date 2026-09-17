pipeline {
    agent any
    stages {
        stage('Install Dependencies') {
            steps {
                sh '''
                    python3 -m venv venv
                    ./venv/bin/pip install --upgrade pip
                    ./venv/bin/pip install pytest pyinstaller
                '''
            }
        }
    
        stage('Build') {
            steps {
                sh 'python3 -m py_compile sources/add2vals.py sources/calc.py'
            }
        }
        
        stage('Test') {
            steps {
                sh './venv/bin/pytest --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }
        
        stage('Deliver') {
            steps {
                sh './venv/bin/pyinstaller --onefile sources/add2vals.py'
            }
            post {
                success {
                    archiveArtifacts 'dist/add2vals'
                }
            }
        }
        stage('Deploy to AWS') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'sla32', keyFileVariable: 'IDENTITY_FILE', usernameVariable: 'SSH_USER')]) {
                    sh '''
                        echo "Copying binary to AWS EC2 instance..."
                        
                        # Copy the compiled binary via SCP using the temporary identity file
                        scp -i "$IDENTITY_FILE" -o StrictHostKeyChecking=no dist/add2vals "$SSH_USER@13.210.12.187:/tmp/add2vals"
                        
                        # Move the binary to a system path and set permissions on the remote server
                        ssh -i "$IDENTITY_FILE" -o StrictHostKeyChecking=no "$SSH_USER@13.210.12.187" 'sudo mv /tmp/add2vals /usr/local/bin/add2vals && sudo chmod +x /usr/local/bin/add2vals'
                        
                        echo "Deployment to AWS completed successfully!"
                    '''
                }
            }
        }
    }
}
