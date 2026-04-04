Parfait. Voici la version propre, commentée, sans emoji, adaptée pour le rendu de TP, avec une séparation claire de chaque étape du projet :

pipeline {

    // ==============================================
    // Étape 1 : Définition de l'agent et environnement
    // ==============================================
    agent any

    environment {
        // Chemin vers le fichier sum.py sur la machine locale
        SUM_PY_PATH    = "D:\\Documents\\Docker-jenkins\\sum.py"

        // Chemin vers le dossier contenant le Dockerfile
        DIR_PATH       = "D:\\Documents\\Docker-jenkins"

        // Chemin vers le fichier de variables de test
        TEST_FILE_PATH = "D:\\Documents\\Docker-jenkins\\test_variables.txt"

        /*
          Note importante : CONTAINER_ID n'est PAS déclaré ici.
          Bug Jenkins Windows : toute variable déclarée dans la section environment
          devient implicitement en lecture seule et ne peut pas être
          modifiée ensuite dans un bloc script.
        */
    }


    stages {

        // ==============================================
        // Étape 2 : Stage de Construction (Build)
        // ==============================================
        stage('Build') {
            steps {
                // Construit l'image Docker à partir du Dockerfile
                bat "docker build -t sum-python-app \"%DIR_PATH%\""
            }
        }


        // ==============================================
        // Étape 3 : Stage d'Exécution (Run)
        // ==============================================
        stage('Run') {
            steps {
                script {
                    // Supprime le conteneur s'il existait déjà d'une exécution précédente
                    bat "docker rm -f sum-container || exit /b 0"

                    /*
                      Démarre le conteneur en arrière plan.
                      On utilise un nom fixe au lieu de récupérer l'ID dynamique
                      pour contourner le bug de returnStdout cassé sur la commande bat Windows.
                      Ceci fonctionne exactement de la même manière qu'un ID.
                    */
                    bat "docker run -d --name sum-container sum-python-app"

                    echo "Conteneur lancé avec succès"
                }
            }
        }


        // ==============================================
        // Étape 4 : Stage de Test
        // ==============================================
        stage('Test') {
            steps {
                script {
                    // Lit le fichier de test et découpe en lignes
                    def testLines = readFile(env.TEST_FILE_PATH).split(/\r?\n/)

                    // Pour chaque ligne de test
                    for (line in testLines) {
                        line = line.trim()

                        // Ignore les lignes vides ou mal formées
                        if (line == '' || line.split(/\s+/).size() < 3) continue

                        // Découpe la ligne en arg1, arg2, somme attendue
                        def vars = line.split(/\s+/)
                        def arg1 = vars[0]
                        def arg2 = vars[1]
                        def expectedSum = vars[2].toFloat()

                        // Exécute sum.py à l'intérieur du conteneur Docker
                        def output = bat(
                            script: "docker exec sum-container python /app/sum.py ${arg1} ${arg2}",
                            returnStdout: true
                        )

                        // Récupère la dernière ligne non vide = résultat du script
                        def outLines = output.split(/\r?\n/).findAll { it.trim() }
                        def result = outLines[-1].trim().toFloat()

                        // Comparaison avec une petite tolérance pour les floats
                        if (Math.abs(result - expectedSum) < 0.0001) {
                            echo "SUCCÈS : ${arg1} + ${arg2} = ${result}"
                        } else {
                            error "ERREUR : ${arg1} + ${arg2} = ${result} (attendu: ${expectedSum})"
                        }
                    }
                }
            }
        }

    }


    // ==============================================
    // Étape 5 : Nettoyage systématique
    // ==============================================
    post {
        always {
            script {
                echo "Nettoyage du conteneur Docker..."

                // Arrête et supprime le conteneur dans TOUS les cas
                // Même si le pipeline a échoué à n'importe quelle étape
                bat "docker rm -f sum-container"

                echo "Conteneur arrêté et supprimé"
            }
        }
    }

}