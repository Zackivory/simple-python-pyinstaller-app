pipeline {
    agent any
    options {
        skipStagesAfterUnstable()
    }

    environment {
        PATH = "/opt/anaconda3/bin:$PATH"
    }

    stages {
        stage('Setup Conda Environment') {
            steps {
                // Set up or update a Conda environment
                sh '''
                    conda create --yes --name myenv python=3.8
                    conda activate myenv
                    pip install pytest pyinstaller
                '''
                // Use environment variables to make sure conda is initialized in each shell step
                script {
                    env.PATH = "/opt/anaconda3/envs/myenv/bin:$PATH"
                }
            }
        }
        stage('Build') {
            steps {
                sh 'python -m py_compile sources/add2vals.py sources/calc.py'
                stash(name: 'compiled-results', includes: 'sources/*.py*')
            }
        }
        stage('Test') {
            steps {
                sh '''
                    conda activate myenv
                    pytest --junit-xml=test-reports/results.xml sources/test_calc.py
                '''
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }
        stage('Deliver') {
            steps {
                sh '''
                    conda activate myenv
                    pyinstaller --onefile sources/add2vals.py
                '''
            }
            post {
                success {
                    archiveArtifacts 'dist/add2vals'
                }
            }
        }
    }
}