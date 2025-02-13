pipeline {
    agent any

    environment {
        DATA_PATH = ""  // Les fichiers sont à la racine
        MODEL_PATH = "models/"
        DOCKER_IMAGE_NAME = "mini-projet-model"
        DOCKER_REGISTRY = "yassindoghri"  
    }

    stages {
        stage('Cloner le code') {
            steps {
                git branch: 'main', url: 'https://github.com/yassindoghriii/MLOPS.git'
            }
        }

        stage('Vérifier pg_config (PostgreSQL)') {
            steps {
                script {
                    def pg_config_exists = sh(script: "which pg_config", returnStatus: true) == 0
                    if (!pg_config_exists) {
                        echo "⚠️ pg_config introuvable. Installation de PostgreSQL via Homebrew..."
                        sh 'brew install postgresql'
                    }
                }
            }
        }

        stage('Vérifier les fichiers de données') {
            steps {
                script {
                    if (fileExists('train.csv') && fileExists('test.csv')) {
                        echo "✔️ Les fichiers de données existent."
                    } else {
                        error "❌ Les fichiers train.csv et test.csv sont manquants."
                    }
                }
            }
        }

        stage('Installer les dépendances') {
            steps {
                sh 'python3 -m pip install --upgrade pip'
                sh "sed -i '' 's/psycopg2/psycopg2-binary/g' requirements.txt" // Remplacement automatique
                sh 'python3 -m pip install --no-cache-dir -r requirements.txt || exit 1'
            }
        }

        stage('Prétraitement des données') {
            steps {
                sh 'python preprocessing.py'
            }
        }

        stage('Entraînement du modèle') {
            steps {
                sh 'python train.py'
            }
        }

        stage('Évaluation du modèle') {
            steps {
                sh 'python evaluate.py'
            }
        }

        stage('Construire l\'image Docker avec l\'API Flask') {
            steps {
                sh 'docker build -t $DOCKER_REGISTRY/$DOCKER_IMAGE_NAME:latest .'
            }
        }

        stage('Push l\'image Docker vers Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'yassin', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh "docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD"
                    sh "docker push $DOCKER_REGISTRY/$DOCKER_IMAGE_NAME:latest"
                }
            }
        }

        stage('Stockage des artefacts') {
            steps {
                archiveArtifacts artifacts: 'rf_model.pkl, dt_model.pkl, ann_model.pkl', fingerprint: true
            }
        }

        stage('Construire et Déployer avec Docker Compose') {
            steps {
                sh 'docker-compose up --build -d'
            }
        }

        stage('Vérifier les Conteneurs') {
            steps {
                sh 'docker ps'
            }
        }
    }

    post {
        success {
            echo "🎉 Pipeline terminé avec succès ! ✅"
        }
        failure {
            echo "🚨 Le pipeline a échoué ! Vérifie les logs Jenkins."
        }
    }
}
