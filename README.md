<!DOCTYPE html>
<html lang="nl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SONG COLLECTIE</title>
    <style>
        /* Algemene styling */
        body, html {
            margin: 0;
            padding: 0;
            font-family: Arial, sans-serif;
            background-color: #f9f9f9;
        }
        /* Hoofding styling */
        header {
            position: fixed;
            top: 0;
            width: 100%;
            background-color: #fff;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
            padding: 10px;
            z-index: 10;
            text-align: center;
        }
        header img {
            width: 100%;
            max-width: 600px;
            height: auto;
        }
        .controls {
            margin-top: 10px;
            display: flex;
            justify-content: center;
            gap: 10px;
        }
        input[type="text"] {
            padding: 8px;
            width: 200px;
            border: 1px solid #ccc;
            border-radius: 5px;
        }
        button {
            padding: 8px 12px;
            border: none;
            background-color: #007bff;
            color: #fff;
            border-radius: 5px;
            cursor: pointer;
        }
        button:hover {
            background-color: #0056b3;
        }
        /* Alfabet-filter styling */
        .alphabet {
            display: flex;
            flex-wrap: wrap;
            justify-content: center;
            margin-top: 70px; /* om ruimte te maken voor de vaste header */
        }
        .alphabet button {
            background-color: transparent;
            color: #007bff;
            border: none;
            cursor: pointer;
            font-size: 18px;
            margin: 2px;
        }
        /* Liedjeslijst styling */
        #song-table {
            width: 100%;
            max-width: 600px;
            margin: 0 auto;
            padding-top: 20px;
        }
        #song-table th, #song-table td {
            padding: 10px;
            text-align: left;
            border-bottom: 1px solid #ddd;
        }
        /* Responsieve styling */
        @media (max-width: 600px) {
            header img {
                width: 100%;
            }
            input[type="text"], button {
                width: 100%;
                margin: 5px 0;
            }
        }
    </style>
    <script src="https://cdn.jsdelivr.net/npm/xlsx/dist/xlsx.full.min.js"></script>
    <script>
        let songData = [];
        let uitvoerderData = [];
        let titelData = [];

        // Laad het Excel-bestand bij het openen van de pagina
        window.onload = function() {
            loadExcelFile();
        };

        async function loadExcelFile() {
            try {
                console.log("Probeer het Excel-bestand te laden...");
                const response = await fetch('Songs.xlsx');
                const data = await response.arrayBuffer();
                const workbook = XLSX.read(data, { type: 'array' });

                // Controleer de structuur van de workbook
                console.log("Workbook geladen:", workbook);

                // Haal data uit kolommen A, B (gesorteerd op Uitvoerder) en D, E (gesorteerd op Titel)
                const sheet = workbook.Sheets[workbook.SheetNames[0]];
                uitvoerderData = XLSX.utils.sheet_to_json(sheet, { header: 1, range: "A1:B2753" }).slice(1);
                titelData = XLSX.utils.sheet_to_json(sheet, { header: 1, range: "D1:E2753" }).slice(1);

                // Controleer de ingelezen gegevens
                console.log("Uitvoerder Data:", uitvoerderData);
                console.log("Titel Data:", titelData);

                // Gebruik uitvoerderData als standaardlijst
                songData = uitvoerderData;
                displaySongs(songData);
            } catch (error) {
                console.error("Kan het bestand niet laden:", error);
            }
        }

        function displaySongs(data) {
            const tbody = document.querySelector('#song-table tbody');
            tbody.innerHTML = '';
            data.forEach(row => {
                const tr = document.createElement('tr');
                tr.innerHTML = `<td>${row[0] || ''}</td><td>${row[1] || ''}</td>`;
                tbody.appendChild(tr);
            });
        }

        // Zoek en filter functie
        function filterSongs() {
            const query = document.getElementById('search').value.toLowerCase();
            const filteredData = songData.filter(row => 
                (row[0] && row[0].toLowerCase().includes(query)) || 
                (row[1] && row[1].toLowerCase().includes(query))
            );
            console.log("Zoekterm:", query, "Gefilterde gegevens:", filteredData);
            displaySongs(filteredData);
        }

        // Sorteren op uitvoerder of titel
        function sortSongs(column) {
            songData.sort((a, b) => {
                if (!a[column]) return 1;
                if (!b[column]) return -1;
                return a[column].localeCompare(b[column]);
            });
            displaySongs(songData);
        }

        // Filter op alfabet
        function alphabetFilter(letter) {
            const filteredData = songData.filter(row => 
                (row[0] && row[0][0].toLowerCase() === letter) ||
                (row[1] && row[1][0].toLowerCase() === letter)
            );
            console.log("Gekozen letter:", letter, "Gefilterde gegevens:", filteredData);
            displaySongs(filteredData);
        }

        // Functie om over te schakelen tussen uitvoerderData en titelData
        function toggleSortOrder(order) {
            songData = (order === 'uitvoerder') ? uitvoerderData : titelData;
            console.log("Huidige lijst (op " + order + "):", songData);
            displaySongs(songData);
        }
    </script>
</head>
<body>
    <!-- Header met banner en zoekveld -->
    <header>
        <img src="Banner.jpg" alt="Banner">
        <div class="controls">
            <input type="text" id="search" placeholder="Zoek liedje..." oninput="filterSongs()">
            <button onclick="toggleSortOrder('uitvoerder')">Sorteer op Uitvoerder</button>
            <button onclick="toggleSortOrder('titel')">Sorteer op Titel</button>
        </div>
    </header>

    <!-- Alfabet filter -->
    <div class="alphabet">
        <!-- Genereer knoppen voor elke letter en # -->
        <script>
            const alphabet = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ#';
            alphabet.split('').forEach(letter => {
                document.write(`<button onclick="alphabetFilter('${letter.toLowerCase()}')">${letter}</button>`);
            });
        </script>
    </div>

    <!-- Tabel voor liedjeslijst -->
    <table id="song-table">
        <thead>
            <tr>
                <th>Uitvoerder</th>
                <th>Titel</th>
            </tr>
        </thead>
        <tbody>
            <!-- Dynamische inhoud komt hier -->
        </tbody>
    </table>
</body>
</html>
