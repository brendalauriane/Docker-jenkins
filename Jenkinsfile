pipeline {
    agent any

    environment {
        // Variables demandées par le prof
        CONTAINER_ID   = ''
        SUM_PY_PATH    = "${WORKSPACE}\\sum.py"
        DIR_PATH       = "${WORKSPACE}"
        TEST_FILE_PATH = "${WORKSPACE}\\test_variables.txt"

        // AJOUT MINIMAL NECESSAIRE (car tu l'utilises dans Run)
        // Sans ça, docker run -d %IMAGE_NAME% échoue et CONTAINER_ID devient null
        IMAGE_NAME     = 'sum-python-app'
    }

    stages {

        stage('Build') {
            steps {
                // Build de l'image Docker depuis le Dockerfile dans DIR_PATH
                // (On garde ta ligne, mais on pourrait aussi utiliser %IMAGE_NAME%)
                bat "docker build -t %IMAGE_NAME% \"%DIR_PATH%\""
            }
        }

        stage('Run') {
    steps {
        script {

            // Lancement du conteneur en mode détaché (-d)
            def output = bat(
                script: "docker run -d sum-python-app",
                returnStdout: true
            )

            // Découper la sortie en lignes
            def lines = output.split(/\r?\n/)

            // Supprimer les lignes vides (Windows ajoute parfois des lignes vides)
            lines = lines.findAll { it.trim() }

            // Récupérer la dernière ligne = ID du conteneur
            env.CONTAINER_ID = lines[-1].trim()

            echo "ID DU CONTENEUR = ${env.CONTAINER_ID}"
        }
    }
}

        stage('Test') {
            steps {
                script {
                    // 1) Lire le fichier test_variables.txt et le découper en lignes
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

                        // Récupérer la dernière ligne non vide (plus robuste sur Windows)
                        def outLines = output.split(/\r?\n/).findAll { it.trim() }
                        def result = outLines[-1].trim().toFloat()

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

    // Étape 5 : Section post (nettoyage) — toujours stopper et supprimer
    post {
        always {
            script {
                echo "Arrêt et suppression du conteneur Docker..."

                // Vérification : si Run a échoué, CONTAINER_ID peut être vide
                if (env.CONTAINER_ID != null && env.CONTAINER_ID.trim() != '') {
                    // Stoppe le conteneur
                    bat "docker stop ${env.CONTAINER_ID}"

                    // Supprime le conteneur
                    bat "docker rm ${env.CONTAINER_ID}"

                    echo "Le conteneur ${env.CONTAINER_ID} a été arrêté et supprimé avec succès."
                } else {
                    echo "Aucun conteneur à supprimer (le pipeline s'est peut-être arrêté avant Run)."
                }
            }
        }
    }
}