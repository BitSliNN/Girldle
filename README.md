<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Girldle</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Arial', sans-serif;
            background-color: #f5f5f5;
            padding: 20px;
        }

        .container {
            max-width: 1100px;
            margin: 0 auto;
            background: white;
            padding: 30px;
            border-radius: 10px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }

        h1 {
            color: #d91e5b;
            text-align: center;
            font-size: 3em;
            letter-spacing: 3px;
            margin-bottom: 10px;
        }

        h2 {
            color: #d91e5b;
            text-align: center;
            font-size: 1.2em;
            margin-bottom: 10px;
        }

        .guesses-counter {
            text-align: center;
            color: #666;
            margin-bottom: 30px;
        }

        .search-container {
            position: relative;
            margin-bottom: 30px;
        }

        #searchInput {
            width: 100%;
            padding: 15px;
            font-size: 16px;
            border: 2px solid #ddd;
            border-radius: 5px;
            outline: none;
        }

        #searchInput:focus {
            border-color: #d91e5b;
        }

        .dropdown {
            position: absolute;
            top: 100%;
            left: 0;
            right: 0;
            background: white;
            border: 2px solid #ddd;
            border-top: none;
            max-height: 300px;
            overflow-y: auto;
            z-index: 1000;
            display: none;
        }

        .dropdown.show {
            display: block;
        }

        .dropdown-item {
            padding: 12px 15px;
            cursor: pointer;
            border-bottom: 1px solid #eee;
        }

        .dropdown-item:hover {
            background-color: #f0f0f0;
        }

        .guesses-table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 20px;
        }

        .guesses-table th {
            background-color: #d91e5b;
            color: white;
            padding: 12px;
            text-align: center;
            font-weight: bold;
            font-size: 0.9em;
        }

        .guesses-table td {
            padding: 12px;
            text-align: center;
            border: 1px solid #ddd;
            font-weight: 500;
            background-color: #f0f0f0;
            font-size: 0.85em;
        }

        .correct {
            background-color: #6aaa64 !important;
            color: white;
        }

        .partial {
            background-color: #c9b458 !important;
            color: white;
        }

        .arrow {
            font-size: 1.2em;
            font-weight: bold;
        }

        .message {
            text-align: center;
            margin-top: 20px;
            font-size: 1.2em;
            font-weight: bold;
            color: #d91e5b;
        }

        .success {
            color: #6aaa64;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>GIRLDLE</h1>
        <h2>GUESSING GAME</h2>
        <div class="guesses-counter">
            <span id="guessCount">0</span> of <span id="maxGuesses">8</span> guesses
        </div>

        <div class="search-container">
            <input type="text" id="searchInput" placeholder="Search for a person..." autocomplete="off">
            <div id="dropdown" class="dropdown"></div>
        </div>

        <table class="guesses-table" id="guessesTable">
            <thead>
                <tr>
                    <th>Name</th>
                    <th>Birth Year</th>
                    <th>Height</th>
                    <th>Gender</th>
                    <th>Website</th>
                    <th>Ethnicity</th>
                    <th>Status</th>
                </tr>
            </thead>
            <tbody id="guessesBody">
            </tbody>
        </table>

        <div id="message" class="message"></div>
    </div>

    <script>
        // ===== CUSTOMIZE YOUR DATA HERE =====
        
        // Helper function to parse height string to cm
        function parseHeight(heightStr) {
            const match = heightStr.match(/\((\d+)\s*cm\)/);
            return match ? parseInt(match[1]) : 0;
        }

        // Helper function to normalize ethnicity strings for comparison
        function normalizeEthnicity(ethnicity) {
            return ethnicity.toLowerCase().split(',').map(e => e.trim());
        }

        // Check if ethnicities partially match
        function checkEthnicityMatch(guessEthnicity, correctEthnicity) {
            const guessEthnicities = normalizeEthnicity(guessEthnicity);
            const correctEthnicities = normalizeEthnicity(correctEthnicity);
            
            // Check for exact match
            if (guessEthnicity === correctEthnicity) {
                return 'correct';
            }
            
            // Check for partial match
            const hasPartialMatch = guessEthnicities.some(ge => 
                correctEthnicities.some(ce => ce === ge)
            );
            
            return hasPartialMatch ? 'partial' : 'incorrect';
        }

        // All persons data
        const allPersons = [
            { name: "Abella Danger", cat1: "1995", cat2: "5 ft 4 in (163 cm)", cat3: "Female", cat4: "OnlyFans", cat5: "Ukrainian", cat6: "Retired" },
            { name: "Adriana Chechik", cat1: "1991", cat2: "5 ft 2 in (157 cm)", cat3: "Female", cat4: "OnlyFans", cat5: "Russian, Serbian", cat6: "Active" },
            { name: "Angela White", cat1: "1985", cat2: "5 ft 3 in (160 cm)", cat3: "Female", cat4: "angelawhite.com", cat5: "Australian", cat6: "Active" },
            { name: "Ariella Ferrera", cat1: "1979", cat2: "5 ft 6 in (168 cm)", cat3: "Female", cat4: "ariellaferrera.com", cat5: "Colombian", cat6: "Active" },
            { name: "Asa Akira", cat1: "1986", cat2: "5 ft 2 in (157 cm)", cat3: "Female", cat4: "asaakira.com", cat5: "Japanese", cat6: "Retired" },
            { name: "Autumn Falls", cat1: "2000", cat2: "5 ft 3 in (160 cm)", cat3: "Female", cat4: "OnlyFans", cat5: "Puerto Rican", cat6: "Retired" },
            { name: "Ava Addams", cat1: "1979", cat2: "5 ft 3 in (160 cm)", cat3: "Female", cat4: "avaaddams.com", cat5: "French, Italian", cat6: "Active" },
            { name: "Brandi Love", cat1: "1973", cat2: "5 ft 7 in (170 cm)", cat3: "Female", cat4: "brandilove.com", cat5: "Caucasian", cat6: "Active" },
            { name: "Bree Olson", cat1: "1986", cat2: "5 ft 3 in (160 cm)", cat3: "Female", cat4: "OnlyFans", cat5: "Ukrainian", cat6: "Retired" },
            { name: "Bridgette B", cat1: "1983", cat2: "5 ft 8 in (173 cm)", cat3: "Female", cat4: "bridgettebspain.com", cat5: "Spanish", cat6: "Active" },
            { name: "Britney Amber", cat1: "1986", cat2: "5 ft 5 in (165 cm)", cat3: "Female", cat4: "britneyamber.com", cat5: "Caucasian", cat6: "Active" },
            { name: "Brittany Andrews", cat1: "1973", cat2: "5 ft 6 in (168 cm)", cat3: "Female", cat4: "brittanyandrews.com", cat5: "Caucasian", cat6: "Active" },
            { name: "Carmella Bing", cat1: "1981", cat2: "5 ft 10 in (178 cm)", cat3: "Female", cat4: "OnlyFans", cat5: "Italian", cat6: "Retired" },
            { name: "Cherie DeVille", cat1: "1978", cat2: "5 ft 5 in (165 cm)", cat3: "Female", cat4: "cheriedeville.com", cat5: "French", cat6: "Active" },
            { name: "Christy Mack", cat1: "1991", cat2: "5 ft 5 in (165 cm)", cat3: "Female", cat4: "OnlyFans", cat5: "Caucasian", cat6: "Retired" },
            { name: "Dillion Carter", cat1: "1991", cat2: "5 ft 5 in (165 cm)", cat3: "Female", cat4: "OnlyFans", cat5: "Caucasian", cat6: "Semi-retired" },
            { name: "Dillion Harper", cat1: "1991", cat2: "5 ft 5 in (165 cm)", cat3: "Female", cat4: "OnlyFans", cat5: "Caucasian", cat6: "Semi-retired" },
            { name: "Elsa Jean", cat1: "1996", cat2: "5 ft 3 in (160 cm)", cat3: "Female", cat4: "OnlyFans", cat5: "Caucasian", cat6: "Retired" },
            { name: "Emily Willis", cat1: "1998", cat2: "5 ft 5 in (165 cm)", cat3: "Female", cat4: "OnlyFans", cat5: "Argentinian", cat6: "Retired" },
            { name: "Eva Elfie", cat1: "2000", cat2: "5 ft 5 in (165 cm)", cat3: "Female", cat4: "OnlyFans", cat5: "Russian", cat6: "Active" },
            { name: "Eva Notty", cat1: "1982", cat2: "5 ft 7 in (170 cm)", cat3: "Female", cat4: "evanotty.com", cat5: "Puerto Rican", cat6: "Active" },
            { name: "Francesca Le", cat1: "1970", cat2: "5 ft 2 in (157 cm)", cat3: "Female", cat4: "francescalesxxx.com", cat5: "Latina", cat6: "Active" },
            { name: "Gina Gerson", cat1: "1991", cat2: "5 ft 4 in (163 cm)", cat3: "Female", cat4: "OnlyFans", cat5: "Russian", cat6: "Semi-retired" },
            { name: "Jada Stevens", cat1: "1988", cat2: "5 ft 3 in (160 cm)", cat3: "Female", cat4: "jadastevens.com", cat5: "Caucasian", cat6: "Semi-retired" },
            { name: "Jenna Haze", cat1: "1982", cat2: "5 ft 3 in (160 cm)", cat3: "Female", cat4: "jennahaze.com", cat5: "Spanish, German", cat6: "Retired" },
            { name: "Jenna Jameson", cat1: "1974", cat2: "5 ft 7 in (170 cm)", cat3: "Female", cat4: "justjennajameson.com", cat5: "Italian", cat6: "Retired" },
            { name: "Jenna Presley", cat1: "1987", cat2: "5 ft 1 in (155 cm)", cat3: "Female", cat4: "OnlyFans", cat5: "Caucasian", cat6: "Retired" },
            { name: "Jessa Rhodes", cat1: "1993", cat2: "5 ft 6 in (168 cm)", cat3: "Female", cat4: "OnlyFans", cat5: "Caucasian", cat6: "Active" },
            { name: "Julianna Vega", cat1: "1989", cat2: "5 ft 5 in (165 cm)", cat3: "Female", cat4: "OnlyFans", cat5: "Dominican", cat6: "Active" },
            { name: "Kagney Linn Karter", cat1: "1987", cat2: "5 ft 7 in (170 cm)", cat3: "Female", cat4: "kagneylinnkarter.com", cat5: "German", cat6: "Retired" },
            { name: "Kissa Sins", cat1: "1987", cat2: "5 ft 4 in (163 cm)", cat3: "Female", cat4: "OnlyFans", cat5: "American", cat6: "Active" },
            { name: "Lana Rhoades", cat1: "1996", cat2: "5 ft 3 in (160 cm)", cat3: "Female", cat4: "OnlyFans", cat5: "Slovenian", cat6: "Retired" },
            { name: "Lela Star", cat1: "1985", cat2: "5 ft 2 in (157 cm)", cat3: "Female", cat4: "OnlyFans", cat5: "Cuban", cat6: "Active" },
            { name: "Lena Paul", cat1: "1993", cat2: "5 ft 4 in (163 cm)", cat3: "Female", cat4: "lenapaul.com", cat5: "Caucasian", cat6: "Active" },
            { name: "Lexi Belle", cat1: "1987", cat2: "5 ft 3 in (160 cm)", cat3: "Female", cat4: "lexibelle.com", cat5: "French, Canadian", cat6: "Retired" },
            { name: "Lily Lane", cat1: "1991", cat2: "5 ft 5 in (165 cm)", cat3: "Female", cat4: "OnlyFans", cat5: "Caucasian", cat6: "Active" },
            { name: "Lisa Ann", cat1: "1972", cat2: "5 ft 2 in (157 cm)", cat3: "Female", cat4: "theofficiallisaann.com", cat5: "Italian", cat6: "Retired" },
            { name: "Maddy O'Reilly", cat1: "1990", cat2: "5 ft 4 in (163 cm)", cat3: "Female", cat4: "maddyoreilly.com", cat5: "Irish", cat6: "Active" },
            { name: "Madison Ivy", cat1: "1989", cat2: "4 ft 11 in (150 cm)", cat3: "Female", cat4: "madisonivy.com", cat5: "German", cat6: "Semi-retired" },
            { name: "Mia Malkova", cat1: "1992", cat2: "5 ft 7 in (170 cm)", cat3: "Female", cat4: "OnlyFans", cat5: "German, Irish", cat6: "Active" },
            { name: "Mia Khalifa", cat1: "1993", cat2: "5 ft 2 in (157 cm)", cat3: "Female", cat4: "OnlyFans", cat5: "Lebanese", cat6: "Retired" },
            { name: "Nicole Aniston", cat1: "1987", cat2: "5 ft 5 in (165 cm)", cat3: "Female", cat4: "nicoleaniston.com", cat5: "German, Greek", cat6: "Active" },
            { name: "Nikki Benz", cat1: "1986", cat2: "5 ft 5 in (165 cm)", cat3: "Female", cat4: "nikkibenz.com", cat5: "Ukrainian", cat6: "Retired" },
            { name: "Phoenix Marie", cat1: "1981", cat2: "5 ft 8 in (173 cm)", cat3: "Female", cat4: "phoenixmarie.com", cat5: "German, Italian", cat6: "Active" },
            { name: "Rachel Starr", cat1: "1983", cat2: "5 ft 5 in (165 cm)", cat3: "Female", cat4: "rachelstarr.com", cat5: "Caucasian", cat6: "Active" },
            { name: "Rebecca Linares", cat1: "1983", cat2: "5 ft 3 in (160 cm)", cat3: "Female", cat4: "OnlyFans", cat5: "Spanish", cat6: "Retired" },
            { name: "Riley Reid", cat1: "1991", cat2: "5 ft 4 in (163 cm)", cat3: "Female", cat4: "rileyreid.com", cat5: "Dominican, Puerto Rican, Irish", cat6: "Retired" },
            { name: "Romi Rain", cat1: "1988", cat2: "5 ft 6 in (168 cm)", cat3: "Female", cat4: "romirain.com", cat5: "Italian", cat6: "Active" },
            { name: "Sasha Grey", cat1: "1988", cat2: "5 ft 6 in (168 cm)", cat3: "Female", cat4: "sashagrey.com", cat5: "Irish, Polish", cat6: "Retired" },
            { name: "Siri", cat1: "1988", cat2: "5 ft 9 in (175 cm)", cat3: "Female", cat4: "siripornstar.com", cat5: "Swedish, English", cat6: "Retired" },
            { name: "Sophie Dee", cat1: "1984", cat2: "5 ft 4 in (163 cm)", cat3: "Female", cat4: "OnlyFans", cat5: "Welsh", cat6: "Retired" },
            { name: "Stormy Daniels", cat1: "1979", cat2: "5 ft 7 in (170 cm)", cat3: "Female", cat4: "stormydaniels.com", cat5: "Cajun", cat6: "Semi-retired" },
            { name: "Tori Black", cat1: "1988", cat2: "5 ft 9 in (175 cm)", cat3: "Female", cat4: "toriblack.com", cat5: "German, Italian", cat6: "Retired" },
            { name: "Violet Myers", cat1: "1997", cat2: "5 ft 3 in (160 cm)", cat3: "Female", cat4: "OnlyFans", cat5: "Mexican, Turkish", cat6: "Active" },
            { name: "Violet Starr", cat1: "1996", cat2: "5 ft 5 in (165 cm)", cat3: "Female", cat4: "OnlyFans", cat5: "Cuban, Spanish", cat6: "Active" }
        ];

        // Define daily answers - each answer is the NAME of the person from the array above
        const dailyAnswers = [
            { date: "2025-11-20", answer: "Abella Danger" },
            { date: "2025-11-21", answer: "Mia Khalifa" },
            { date: "2025-11-22", answer: "Riley Reid" },
            { date: "2025-11-23", answer: "Lana Rhoades" },
            { date: "2025-11-24", answer: "Angela White" },
            { date: "2025-11-25", answer: "Adriana Chechik" },
            { date: "2025-11-26", answer: "Mia Malkova" },
            { date: "2025-11-27", answer: "Elsa Jean" },
            { date: "2025-11-28", answer: "Sasha Grey" },
            { date: "2025-11-29", answer: "Brandi Love" },
            { date: "2025-11-30", answer: "Lisa Ann" },
            { date: "2025-12-01", answer: "Jenna Jameson" },
            { date: "2025-12-02", answer: "Emily Willis" },
            { date: "2025-12-03", answer: "Autumn Falls" },
            { date: "2025-12-04", answer: "Violet Myers" },
            { date: "2025-12-05", answer: "Asa Akira" },
            { date: "2025-12-06", answer: "Nicole Aniston" },
            { date: "2025-12-07", answer: "Violet Starr" },
            { date: "2025-12-08", answer: "Stormy Daniels" },
            { date: "2025-12-09", answer: "Eva Elfie" },
            { date: "2025-12-10", answer: "Bree Olson" },
            { date: "2025-12-11", answer: "Jessa Rhodes" },
            { date: "2025-12-12", answer: "Dillion Harper" },
            { date: "2025-12-13", answer: "Gina Gerson" },
            { date: "2025-12-14", answer: "Cherie DeVille" },
            { date: "2025-12-15", answer: "Brittany Andrews" },
            { date: "2025-12-16", answer: "Ariella Ferrera" },
            { date: "2025-12-17", answer: "Ava Addams" },
            { date: "2025-12-18", answer: "Bridgette B" },
            { date: "2025-12-19", answer: "Britney Amber" },
            { date: "2025-12-20", answer: "Carmella Bing" },
            { date: "2025-12-21", answer: "Christy Mack" },
            { date: "2025-12-22", answer: "Dillion Carter" },
            { date: "2025-12-23", answer: "Eva Notty" },
            { date: "2025-12-24", answer: "Francesca Le" },
            { date: "2025-12-25", answer: "Jada Stevens" },
            { date: "2025-12-26", answer: "Jenna Haze" },
            { date: "2025-12-27", answer: "Jenna Presley" },
            { date: "2025-12-28", answer: "Julianna Vega" },
            { date: "2025-12-29", answer: "Kagney Linn Karter" },
            { date: "2025-12-30", answer: "Kissa Sins" },
            { date: "2025-12-31", answer: "Lela Star" },
            { date: "2026-01-01", answer: "Lena Paul" },
            { date: "2026-01-02", answer: "Lexi Belle" },
            { date: "2026-01-03", answer: "Lily Lane" },
            { date: "2026-01-04", answer: "Maddy O'Reilly" },
            { date: "2026-01-05", answer: "Madison Ivy" },
            { date: "2026-01-06", answer: "Nikki Benz" },
            { date: "2026-01-07", answer: "Phoenix Marie" },
            { date: "2026-01-08", answer: "Rachel Starr" },
            { date: "2026-01-09", answer: "Rebecca Linares" },
            { date: "2026-01-10", answer: "Romi Rain" },
            { date: "2026-01-11", answer: "Siri" },
            { date: "2026-01-12", answer: "Sophie Dee" },
            { date: "2026-01-13", answer: "Tori Black" },
        ];

        const maxGuesses = 8;
        // ===== END CUSTOMIZATION =====

        let guessCount = 0;
        let gameOver = false;
        let correctAnswer = null;
        let guessedNames = [];

        // Get today's date in YYYY-MM-DD format
        function getTodayDate() {
            const today = new Date();
            const year = today.getFullYear();
            const month = String(today.getMonth() + 1).padStart(2, '0');
            const day = String(today.getDate()).padStart(2, '0');
            return `${year}-${month}-${day}`;
        }

        // Get storage key for today
        function getStorageKey() {
            return `girldle_game_${getTodayDate()}`;
        }

        // Save game state
        function saveGameState() {
            const gameState = {
                guessCount: guessCount,
                gameOver: gameOver,
                guessedNames: guessedNames,
                guesses: Array.from(guessesBody.children).map(row => ({
                    name: row.cells[0].textContent,
                    birthYear: row.cells[1].innerHTML,
                    height: row.cells[2].innerHTML,
                    gender: row.cells[3].textContent,
                    website: row.cells[4].textContent,
                    ethnicity: row.cells[5].textContent,
                    status: row.cells[6].textContent,
                    cells: [
                        row.cells[0].className,
                        row.cells[1].className,
                        row.cells[2].className,
                        row.cells[3].className,
                        row.cells[4].className,
                        row.cells[5].className,
                        row.cells[6].className
                    ]
                })).reverse()
            };
            localStorage.setItem(getStorageKey(), JSON.stringify(gameState));
        }

        // Load game state
        function loadGameState() {
            const savedState = localStorage.getItem(getStorageKey());
            if (savedState) {
                const gameState = JSON.parse(savedState);
                guessCount = gameState.guessCount;
                gameOver = gameState.gameOver;
                guessedNames = gameState.guessedNames || [];
                guessCountDisplay.textContent = guessCount;

                // Restore guesses
                gameState.guesses.forEach(guess => {
                    const row = document.createElement('tr');
                    
                    ['name', 'birthYear', 'height', 'gender', 'website', 'ethnicity', 'status'].forEach((field, index) => {
                        const cell = document.createElement('td');
                        if (field === 'name' || field === 'gender' || field === 'website' || field === 'ethnicity' || field === 'status') {
                            cell.textContent = guess[field];
                        } else {
                            cell.innerHTML = guess[field];
                        }
                        cell.className = guess.cells[index];
                        row.appendChild(cell);
                    });
                    
                    guessesBody.appendChild(row);
                });

                // Check if game was won or lost
                if (gameOver) {
                    const lastGuess = gameState.guesses[gameState.guesses.length - 1];
                    if (lastGuess.name === correctAnswer.name) {
                        message.textContent = '🎉 Congratulations! You guessed correctly!';
                        message.className = 'message success';
                    } else {
                        message.textContent = `Game Over! The answer was ${correctAnswer.name}`;
                        message.className = 'message';
                    }
                }
            }
        }

        // Find today's answer
        function getTodayAnswer() {
            const todayDate = getTodayDate();
            const todayData = dailyAnswers.find(item => item.date === todayDate);
            
            if (todayData) {
                const person = allPersons.find(p => p.name === todayData.answer);
                return person || allPersons[0];
            }
            
            return allPersons[0];
        }

        correctAnswer = getTodayAnswer();

        const searchInput = document.getElementById('searchInput');
        const dropdown = document.getElementById('dropdown');
        const guessesBody = document.getElementById('guessesBody');
        const message = document.getElementById('message');
        const guessCountDisplay = document.getElementById('guessCount');

        // Load saved game state on page load
        loadGameState();

        searchInput.addEventListener('input', function() {
            const searchTerm = this.value.toLowerCase();
            dropdown.innerHTML = '';
            
            if (searchTerm.length === 0) {
                dropdown.classList.remove('show');
                return;
            }

            const filtered = allPersons.filter(p => 
                p.name.toLowerCase().includes(searchTerm) && !guessedNames.includes(p.name)
            );

            if (filtered.length > 0) {
                filtered.forEach(person => {
                    const div = document.createElement('div');
                    div.className = 'dropdown-item';
                    div.textContent = person.name;
                    div.onclick = () => selectPerson(person);
                    dropdown.appendChild(div);
                });
                dropdown.classList.add('show');
            } else {
                dropdown.classList.remove('show');
            }
        });

        document.addEventListener('click', function(e) {
            if (!searchInput.contains(e.target) && !dropdown.contains(e.target)) {
                dropdown.classList.remove('show');
            }
        });

        function selectPerson(person) {
            if (gameOver) return;
            
            // Check if already guessed
            if (guessedNames.includes(person.name)) {
                return;
            }

            searchInput.value = '';
            dropdown.classList.remove('show');
            
            guessCount++;
            guessCountDisplay.textContent = guessCount;
            guessedNames.push(person.name);

            const row = document.createElement('tr');
            
            // Name cell
            const nameCell = document.createElement('td');
            nameCell.textContent = person.name;
            if (person.name === correctAnswer.name) {
                nameCell.className = 'correct';
            }
            row.appendChild(nameCell);

            // Birth Year cell (with arrow if not correct)
            const birthYearCell = document.createElement('td');
            const personYear = parseInt(person.cat1);
            const correctYear = parseInt(correctAnswer.cat1);
            
            if (personYear === correctYear) {
                birthYearCell.textContent = person.cat1;
                birthYearCell.className = 'correct';
            } else {
                const arrow = personYear < correctYear ? '↑' : '↓';
                birthYearCell.innerHTML = `${person.cat1} <span class="arrow">${arrow}</span>`;
            }
            row.appendChild(birthYearCell);

            // Height cell (with arrow if not correct)
            const heightCell = document.createElement('td');
            const personHeight = parseHeight(person.cat2);
            const correctHeight = parseHeight(correctAnswer.cat2);
            
            if (personHeight === correctHeight) {
                heightCell.textContent = person.cat2;
                heightCell.className = 'correct';
            } else {
                const arrow = personHeight < correctHeight ? '↑' : '↓';
                heightCell.innerHTML = `${person.cat2} <span class="arrow">${arrow}</span>`;
            }
            row.appendChild(heightCell);

            // Gender cell
            const genderCell = document.createElement('td');
            genderCell.textContent = person.cat3;
            if (person.cat3 === correctAnswer.cat3) {
                genderCell.className = 'correct';
            }
            row.appendChild(genderCell);

            // Website cell
            const websiteCell = document.createElement('td');
            websiteCell.textContent = person.cat4;
            if (person.cat4 === correctAnswer.cat4) {
                websiteCell.className = 'correct';
            }
            row.appendChild(websiteCell);

            // Ethnicity cell (with partial match support)
            const ethnicityCell = document.createElement('td');
            ethnicityCell.textContent = person.cat5;
            const ethnicityMatch = checkEthnicityMatch(person.cat5, correctAnswer.cat5);
            if (ethnicityMatch === 'correct') {
                ethnicityCell.className = 'correct';
            } else if (ethnicityMatch === 'partial') {
                ethnicityCell.className = 'partial';
            }
            row.appendChild(ethnicityCell);

            // Status cell
            const statusCell = document.createElement('td');
            statusCell.textContent = person.cat6;
            if (person.cat6 === correctAnswer.cat6) {
                statusCell.className = 'correct';
            }
            row.appendChild(statusCell);

            guessesBody.insertBefore(row, guessesBody.firstChild);

            // Check win condition
            if (person.name === correctAnswer.name) {
                message.textContent = '🎉 Congratulations! You guessed correctly!';
                message.className = 'message success';
                gameOver = true;
            } else if (guessCount >= maxGuesses) {
                message.textContent = `Game Over! The answer was ${correctAnswer.name}`;
                message.className = 'message';
                gameOver = true;
            }

            // Save game state after each guess
            saveGameState();
        }
    </script>
</body>
</html>
