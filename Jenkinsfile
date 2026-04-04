pipeline {
    agent any

    environment {
        CONTAINER_ID   = ''
        SUM_PY_PATH    = "${WORKSPACE}\\sum.py"
        DIR_PATH       = "${WORKSPACE}"
        TEST_FILE_PATH = "${WORKSPACE}\\test_variables.txt"
    }

    stages {
        stage('Build') {
            steps {
                bat "docker build -t sum-python-app \"%DIR_PATH%\""
            }
        }

        stage('Run') {
            steps {
                script {
                    // Le @echo off empêche Windows de polluer le texte
                    // Le .trim() retire les sauts de ligne vides
                    def output = bat(script: "@echo off\ndocker run -d %IMAGE_NAME%", returnStdout: true).trim()
                    def lines = output.split(/\r?\n/)
                    
                    // On prend la dernière ligne qui est le vrai ID du conteneur
                    env.CONTAINER_ID = lines[-1].trim()
                    echo "ID DU CONTENEUR = ${env.CONTAINER_ID}"
                }
            }
        }


stage('Test') {
    steps {
        script {
            // 1) Lire le fichier test_variables.txt (TEST_FILE_PATH) et le découper en lignes
            def testLines = readFile(env.TEST_FILE_PATH).split(/\r?\n/)

            // 2) Pour chaque ligne : exécuter sum.py dans le conteneur et comparer au résultat attendu
            for (line in testLines) {
                line = line.trim()
                if (line == '') continue  // ignore les lignes vides

                // Chaque ligne contient : arg1 arg2 expectedSum
                def vars = line.split(/\s+/)
                def arg1 = vars[0]
                def arg2 = vars[1]
                def expectedSum = vars[2].toFloat()

                // Exécuter sum.py dans le conteneur Docker avec docker exec
                def output = bat(
                    script: "docker exec ${env.CONTAINER_ID} python /app/sum.py ${arg1} ${arg2}",
                    returnStdout: true
                )

                // Récupérer la dernière ligne de sortie (le print final du script)
                def result = output.split(/\r?\n/)[-1].trim().toFloat()

                // Comparer résultat obtenu vs attendu
                if (result == expectedSum) {
                    echo "SUCCÈS : ${arg1} + ${arg2} = ${result} (attendu: ${expectedSum})"
                } else {
                    error "ERREUR : ${arg1} + ${arg2} = ${result} (attendu: ${expectedSum})"
                        }
                     }
                }
            }
        }
    }

    //  Étape 5 : Section post de netoyage 
    post {
        always {
            script {
                echo "Arrêt et suppression du conteneur Docker..."
                
                // On vérifie d'abord que l'ID du conteneur n'est pas vide
                // (au cas où le pipeline aurait planté à l'étape Build, avant de créer le conteneur)
                if (env.CONTAINER_ID != null && env.CONTAINER_ID.trim() != '') {
                    // Stoppe le conteneur
                    bat "docker stop ${env.CONTAINER_ID}"
                    
                    // Supprime le conteneur
                    bat "docker rm ${env.CONTAINER_ID}"
                    
                    echo "Le conteneur ${env.CONTAINER_ID} a été arrêté et supprimé avec succès."
                } else {
                    echo "Aucun conteneur à supprimer (le pipeline s'est peut-être arrêté avant)."
                }
            }
        }
    }
}