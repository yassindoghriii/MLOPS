pipeline {
    agent any

    environment {
        DATA_PATH = ""  // Data files are in the root directory
        MODEL_PATH = "models/"
        DOCKER_IMAGE_NAME = "mini-projet-model"
        DOCKER_REGISTRY = "yassindoghri"
        VENV_DIR = "venv"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/yassindoghriii/MLOPS.git'
            }
        }

        stage('Setup Environment & Dependencies') {
            steps {
                script {
                    // Ensure PostgreSQL's pg_config is in PATH
                    env.PATH = "/opt/homebrew/bin:${env.PATH}"
                }
                
                sh '''
                    # Create virtual environment
                    python3 -m venv ${VENV_DIR}
                    
                    # Activate environment and install dependencies
                    source ${VENV_DIR}/bin/activate
                    python3 -m pip install --upgrade pip
                    python3 -m pip install --no-cache-dir -r requirements.txt
                '''
            }
        }

        stage('Check Environment & Dependencies') {
            steps {
                sh '''
                    source ${VENV_DIR}/bin/activate
                    python3 --version
                    which python3
                    pip list
                    which pg_config || echo "⚠️ pg_config not found!"
                    pg_config --version || echo "⚠️ pg_config cannot run!"
                '''
            }
        }

        stage('Check Data Files') {
            steps {
                script {
                    if (fileExists('train.csv') && fileExists('test.csv')) {
                        echo "✔️ Data files exist."
                    } else {
                        error "❌ Missing train.csv and/or test.csv."
                    }
                }
            }
        }

        stage('Preprocess Data') {
            steps {
                sh 'source ${VENV_DIR}/bin/activate && python3 preprocessing.py'
            }
        }

        stage('Train Model') {
            steps {
                sh 'source ${VENV_DIR}/bin/activate && python3 train.py'
            }
        }

        stage('Evaluate Model') {
            steps {
                sh 'source ${VENV_DIR}/bin/activate && python3 evaluate.py'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t ${DOCKER_REGISTRY}/${DOCKER_IMAGE_NAME}:latest .
                '''
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'yassin', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh '''
                        docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}
                        docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE_NAME}:latest
                    '''
                }
            }
        }

        stage('Store Model Artifacts') {
            steps {
                archiveArtifacts artifacts: 'rf_model.pkl, dt_model.pkl, ann_model.pkl', fingerprint: true
            }
        }

        stage('Deploy with Docker Compose') {
            steps {
                sh 'docker-compose up --build -d'
            }
        }

        stage('Check Running Containers') {
            steps {
                sh 'docker ps'
            }
        }
    }

    post {
        success {
            echo "🎉 Pipeline completed successfully! ✅"
        }
        failure {
            echo "🚨 Pipeline failed! Check Jenkins logs."
        }
    }
}
