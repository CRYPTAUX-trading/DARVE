<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Exemple de clic</title>
    <style>
        /* Style du bouton */
        button {
            padding: 10px 20px;
            font-size: 16px;
            cursor: pointer;
        }

        /* Style du texte qui apparaît */
        #message {
            margin-top: 20px;
            font-size: 24px;
            font-weight: bold;
            color: blue;
        }
    </style>
</head>
<body>

    <!-- Bouton -->
    <button onclick="afficherMessage()">CLIQUER</button>

    <!-- Zone où le message va apparaître -->
    <div id="message"></div>

    <script>
        // Fonction qui affiche le mot "NOA" lorsqu'on clique sur le bouton
        function afficherMessage() {
            document.getElementById("message").innerText = "NOA";
        }
    </script>

</body>
</html>

