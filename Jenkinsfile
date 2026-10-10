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
                withCredentials([sshUserPrivateKey(credentialsId: 'Sla', keyFileVariable: 'IDENTITY_FILE', usernameVariable: 'SSH_USER')]) {
                    sh '''
                        echo "Configuring private key permissions..."
                        chmod 600 "$IDENTITY_FILE"
                        
                        echo "Copying binary to AWS EC2 instance..."
                        scp -i "$IDENTITY_FILE" -o StrictHostKeyChecking=no dist/add2vals "$SSH_USER@47.129.103.40:/tmp/add2vals"
                        
                        echo "Moving binary to system path and setting executable permissions..."
                        ssh -i "$IDENTITY_FILE" -o StrictHostKeyChecking=no "$SSH_USER@47.129.103.40" "sudo mv /tmp/add2vals /usr/local/bin/add2vals && sudo chmod +x /usr/local/bin/add2vals"
                        
                        echo "Deployment to AWS completed successfully!"
                    '''
                }
            }
        }
    }
}
