pipeline {
    agent any

    environment {
        DATA_PATH = ""  // Les fichiers de données sont à la racine
        MODEL_PATH = "models/"
        DOCKER_IMAGE_NAME = "mini-projet-model"
        DOCKER_REGISTRY = "yassindoghri"
        VENV_DIR = "venv"
        PYTHON_VERSION = "3.10"  // Assure la compatibilité avec TensorFlow
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/yassindoghriii/MLOPS.git'
            }
        }

        stage('Setup Python & Virtual Environment') {
            steps {
                script {
                    // Ajout de Python 3.10 et PostgreSQL à PATH
                    env.PATH = "/opt/homebrew/bin:/opt/homebrew/opt/python@${PYTHON_VERSION}/bin:${env.PATH}"
                }

                sh '''
                    # Vérifier la version de Python
                    python3 --version

                    # Créer un environnement virtuel
                    python3 -m venv ${VENV_DIR}

                    # Activer l'environnement et mettre à jour pip
                    source ${VENV_DIR}/bin/activate
                    python3 -m pip install --upgrade pip
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    source ${VENV_DIR}/bin/activate

                    # Installer TensorFlow en fonction de l'architecture du Mac
                    if [[ "$(uname -m)" == "arm64" ]]; then
                        python3 -m pip install tensorflow-macos==2.11.0 tensorflow-metal==0.8.0
                    else
                        python3 -m pip install tensorflow==2.11.0
                    fi

                    # Installer les autres dépendances
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
                    python3 -c "import tensorflow as tf; print('TensorFlow version:', tf.__version__)"
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
