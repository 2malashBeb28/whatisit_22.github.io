<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>МУРИНО: Сон в красном грузовике</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            background-color: #111;
            color: #ddd;
            font-family: 'Courier New', monospace;
            overflow: hidden;
        }
        
        #game-container {
            position: relative;
            width: 800px;
            height: 600px;
            margin: 20px auto;
            border: 2px solid #d33;
        }
        
        #visual-novel {
            width: 100%;
            height: 100%;
            background-color: #222;
            position: relative;
        }
        
        #character-image {
            position: absolute;
            bottom: 150px;
            left: 50%;
            transform: translateX(-50%);
            max-height: 400px;
        }
        
        #text-box {
            position: absolute;
            bottom: 0;
            width: 100%;
            height: 140px;
            background-color: rgba(0, 0, 0, 0.8);
            padding: 10px;
            box-sizing: border-box;
        }
        
        #character-name {
            color: #d33;
            font-weight: bold;
            margin-bottom: 5px;
        }
        
        #dialogue-text {
            line-height: 1.4;
        }
        
        #choices {
            margin-top: 10px;
        }
        
        .choice-btn {
            background-color: #333;
            color: #ddd;
            border: 1px solid #d33;
            padding: 5px 10px;
            margin-right: 10px;
            cursor: pointer;
        }
        
        .choice-btn:hover {
            background-color: #d33;
        }
        
        #washing-game {
            display: none;
            width: 100%;
            height: 100%;
            background-color: #333;
            position: relative;
        }
        
        #progress-bar {
            width: 300px;
            height: 30px;
            background-color: #111;
            position: absolute;
            top: 20px;
            left: 50%;
            transform: translateX(-50%);
            border: 1px solid #d33;
        }
        
        #progress {
            height: 100%;
            width: 0%;
            background-color: #d33;
        }
        
        #dirt {
            position: absolute;
            width: 50px;
            height: 50px;
            background-color: #654321;
            border-radius: 50%;
            cursor: pointer;
        }
        
        #driving-game {
            display: none;
            width: 100%;
            height: 100%;
            background-color: #111;
            position: relative;
            overflow: hidden;
        }
        
        #road {
            position: absolute;
            width: 100%;
            height: 100%;
            background-color: #333;
            background-image: repeating-linear-gradient(
                0deg,
                transparent,
                transparent 49px,
                #fff 49px,
                #fff 50px
            );
        }
        
        #player-car {
            position: absolute;
            width: 60px;
            height: 100px;
            background-color: #d33;
            bottom: 50px;
            left: 370px;
        }
        
        .other-car {
            position: absolute;
            width: 60px;
            height: 100px;
            background-color: #369;
        }
        
        #game-over, #game-win {
            display: none;
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.8);
            z-index: 100;
            flex-direction: column;
            justify-content: center;
            align-items: center;
        }
        
        .ending-text {
            font-size: 24px;
            margin-bottom: 20px;
            color: #d33;
            text-align: center;
        }
        
        #restart-btn {
            background-color: #d33;
            color: #fff;
            border: none;
            padding: 10px 20px;
            cursor: pointer;
        }
    </style>
</head>
<body>
    <div id="game-container">
        <!-- Визуальная новелла -->
        <div id="visual-novel">
            <img id="character-image" src="" alt="Персонаж">
            <div id="text-box">
                <div id="character-name"></div>
                <div id="dialogue-text"></div>
                <div id="choices"></div>
            </div>
        </div>
        
        <!-- Мини-игра с мытьем -->
        <div id="washing-game">
            <div id="progress-bar">
                <div id="progress"></div>
            </div>
            <div id="dirt"></div>
        </div>
        
        <!-- Мини-игра с вождением -->
        <div id="driving-game">
            <div id="road"></div>
            <div id="player-car"></div>
        </div>
        
        <!-- Концовки -->
        <div id="game-over">
            <div class="ending-text">Илья убил Дашулю. Его посадили в тюрьму. В камере он нашел старую газету с заголовком: "Грузовик с говядиной так и не доехал до Казани..."</div>
            <button id="restart-btn">Начать сначала</button>
        </div>
        
        <div id="game-win">
            <div class="ending-text">Вы съели труп Картера. Вкус напоминал ту самую говядину из пропавшего грузовика. Дашуля улыбнулась: "Теперь мы часть одной тайны".</div>
            <button id="restart-btn">Начать сначала</button>
        </div>
    </div>

    <script>
        // Игровые переменные
        let currentScene = 0;
        let washingProgress = 0;
        let washingTimer;
        let dirtCount = 10;
        let carSpeed = 5;
        let otherCars = [];
        let carSpawnTimer;
        let score = 0;
        let keyState = {};
        
        // Диалоги и сцены
        const scenes = [
            // Сцена 0: Вступление
            {
                bg: "url('fura_background.jpg')",
                character: "ilya.png",
                name: "Илья",
                text: "Мурино. Чёртово Мурино. Я купил эту фуру, чтобы наконец свалить отсюда. Но она настояла ехать со мной...",
                choices: [
                    { text: "Продолжить", nextScene: 1 }
                ]
            },
            // Сцена 1: Диалог с Дашулей
            {
                bg: "url('fura_interior.jpg')",
                character: "dashulya.png",
                name: "Дашуля",
                text: "Ты вообще понимаешь, что без меня ты бы тут сдох от тоски? Вон даже вонь от тебя, как от пропавшей говядины...",
                choices: [
                    { text: "Промолчать", nextScene: 2 },
                    { text: "Ответить", nextScene: 3 }
                ]
            },
            // Сцена 2: Промолчал
            {
                bg: "url('fura_interior.jpg')",
                character: "ilya.png",
                name: "Илья",
                text: "...",
                choices: [
                    { text: "Дашуля продолжает", nextScene: 4 }
                ]
            },
            // Сцена 3: Ответил
            {
                bg: "url('fura_interior.jpg')",
                character: "ilya.png",
                name: "Илья",
                text: "Это ты воняешь, как та твоя идея с кражей грузовика!",
                choices: [
                    { text: "Дашуля реагирует", nextScene: 5 }
                ]
            },
            // Сцена 4: Переход к игре с мытьем
            {
                bg: "url('fura_interior.jpg')",
                character: "dashulya.png",
                name: "Дашуля",
                text: "Ладно, хватит дуться. Давай я тебя помою, а то воняешь, как будто ты тот самый пропавший грузовик с мясом.",
                choices: [
                    { text: "Начать мыть Илью", nextScene: "washing" }
                ]
            },
            // Сцена 5: Альтернативный переход
            {
                bg: "url('fura_interior.jpg')",
                character: "dashulya.png",
                name: "Дашуля",
                text: "Ой, да? Ну тогда я точно тебя мыть не буду! Хотя... может быть, если ты попросишь...",
                choices: [
                    { text: "Умолять Дашулю", nextScene: 6 },
                    { text: "Отказаться", nextScene: 7 }
                ]
            },
            // Сцена 6: Умолять
            {
                bg: "url('fura_interior.jpg')",
                character: "ilya.png",
                name: "Илья",
                text: "Ладно... пожалуйста...",
                choices: [
                    { text: "Дашуля соглашается", nextScene: "washing" }
                ]
            },
            // Сцена 7: Отказаться
            {
                bg: "url('fura_interior.jpg')",
                character: "dashulya.png",
                name: "Дашуля",
                text: "Ну и сиди вонючий! Я все равно найду тот грузовик с мясом...",
                choices: [
                    { text: "Продолжить", nextScene: "driving" }
                ]
            },
            // Сцена 8: После успешного мытья
            {
                bg: "url('fura_interior.jpg')",
                character: "ilya.png",
                name: "Илья",
                text: "Чёрт, теперь я пахну твоим этим дешёвым шампунем...",
                choices: [
                    { text: "Дашуля отвечает", nextScene: 9 }
                ]
            },
            // Сцена 9: Диалог после мытья
            {
                bg: "url('fura_interior.jpg')",
                character: "dashulya.png",
                name: "Дашуля",
                text: "Зато теперь ты не отпугнёшь Картера, когда мы его найдём...",
                choices: [
                    { text: "Кто такой Картер?", nextScene: 10 },
                    { text: "Мы его ищем?", nextScene: 11 }
                ]
            },
            // Сцена 10: Вопрос про Картера
            {
                bg: "url('fura_interior.jpg')",
                character: "dashulya.png",
                name: "Дашуля",
                text: "Ты серьёзно? Тот самый Картер, который украл грузовик с говядиной... и мою коллекцию видеокассет с 'Твин Пикс'...",
                choices: [
                    { text: "Продолжить", nextScene: 12 }
                ]
            },
            // Сцена 11: Вопрос про поиски
            {
                bg: "url('fura_interior.jpg')",
                character: "dashulya.png",
                name: "Дашуля",
                text: "Да, идиот! Весь этот рейс - чтобы найти его! Он где-то здесь, в этих лесах... и с моими кассетами!",
                choices: [
                    { text: "Продолжить", nextScene: 12 }
                ]
            },
            // Сцена 12: Переход к игре с вождением
            {
                bg: "url('fura_interior.jpg')",
                character: "ilya.png",
                name: "Илья",
                text: "Боже... ладно, поехали искать твоего Картера. Но если это опять твоя бредовая идея...",
                choices: [
                    { text: "Начать поиски", nextScene: "driving" }
                ]
            },
            // Сцена 13: После вождения (успех)
            {
                bg: "url('forest.jpg')",
                character: "dashulya.png",
                name: "Дашуля",
                text: "Смотри! Вон же его лагерь! И... о боже... он... он мёртв?",
                choices: [
                    { text: "Подойти ближе", nextScene: 14 },
                    { text: "Уехать", nextScene: 15 }
                ]
            },
            // Сцена 14: Финал с поеданием
            {
                bg: "url('forest.jpg')",
                character: "ilya.png",
                name: "Илья",
                text: "Он... он выглядит так, будто умер неделю назад. Но... откуда тогда этот запах жареного мяса?",
                choices: [
                    { text: "Попробовать мясо", nextScene: "win" },
                    { text: "Не трогать", nextScene: 16 }
                ]
            },
            // Сцена 15: Финал с уездом
            {
                bg: "url('fura_interior.jpg')",
                character: "dashulya.png",
                name: "Дашуля",
                text: "Ты серьёзно? Мы приехали за сотни километров, и теперь просто уедем?",
                choices: [
                    { text: "Дашуля злится", nextScene: "over" }
                ]
            },
            // Сцена 16: Альтернативный финал
            {
                bg: "url('forest.jpg')",
                character: "dashulya.png",
                name: "Дашуля",
                text: "Ты всегда такой трусливый! Я сама это сделаю!",
                choices: [
                    { text: "Попытаться остановить", nextScene: "over" },
                    { text: "Пусть делает", nextScene: "win" }
                ]
            }
        ];

        // Инициализация игры
        function initGame() {
            currentScene = 0;
            showScene(currentScene);
            
            // Настройка обработчиков клавиш для игры с вождением
            document.addEventListener('keydown', function(e) {
                keyState[e.key] = true;
            });
            
            document.addEventListener('keyup', function(e) {
                keyState[e.key] = false;
            });
            
            // Кнопка рестарта
            document.getElementById('restart-btn').addEventListener('click', function() {
                document.getElementById('game-over').style.display = 'none';
                document.getElementById('game-win').style.display = 'none';
                document.getElementById('visual-novel').style.display = 'block';
                document.getElementById('washing-game').style.display = 'none';
                document.getElementById('driving-game').style.display = 'none';
                currentScene = 0;
                showScene(currentScene);
            });
        }

        // Показать сцену
        function showScene(index) {
            if (index === "washing") {
                startWashingGame();
                return;
            }
            
            if (index === "driving") {
                startDrivingGame();
                return;
            }
            
            if (index === "over") {
                endGame(false);
                return;
            }
            
            if (index === "win") {
                endGame(true);
                return;
            }
            
            const scene = scenes[index];
            document.getElementById('visual-novel').style.backgroundImage = scene.bg;
            document.getElementById('character-image').src = scene.character;
            document.getElementById('character-name').textContent = scene.name;
            document.getElementById('dialogue-text').textContent = scene.text;
            
            const choicesContainer = document.getElementById('choices');
            choicesContainer.innerHTML = '';
            
            scene.choices.forEach(choice => {
                const button = document.createElement('button');
                button.className = 'choice-btn';
                button.textContent = choice.text;
                button.addEventListener('click', function() {
                    currentScene = choice.nextScene;
                    showScene(currentScene);
                });
                choicesContainer.appendChild(button);
            });
        }

        // Начать игру с мытьем
        function startWashingGame() {
            document.getElementById('visual-novel').style.display = 'none';
            document.getElementById('washing-game').style.display = 'block';
            
            washingProgress = 0;
            dirtCount = 10;
            updateProgressBar();
            
            // Создаем грязь
            createDirt();
            
            // Таймер для прогресса
            washingTimer = setInterval(function() {
                washingProgress -= 2;
                if (washingProgress < 0) washingProgress = 0;
                updateProgressBar();
                
                if (washingProgress >= 100) {
                    clearInterval(washingTimer);
                    setTimeout(function() {
                        document.getElementById('washing-game').style.display = 'none';
                        document.getElementById('visual-novel').style.display = 'block';
                        currentScene = 8;
                        showScene(currentScene);
                    }, 1000);
                }
            }, 200);
        }

        // Создать пятно грязи
        function createDirt() {
            if (dirtCount <= 0) return;
            
            const dirt = document.createElement('div');
            dirt.className = 'dirt';
            dirt.style.left = Math.random() * 700 + 'px';
            dirt.style.top = Math.random() * 400 + 100 + 'px';
            
            dirt.addEventListener('click', function() {
                washingProgress += 10;
                updateProgressBar();
                dirt.remove();
                dirtCount--;
                
                if (dirtCount > 0) {
                    setTimeout(createDirt, 500);
                }
            });
            
            document.getElementById('washing-game').appendChild(dirt);
        }

        // Обновить прогресс бар
        function updateProgressBar() {
            document.getElementById('progress').style.width = washingProgress + '%';
        }

        // Начать игру с вождением
        function startDrivingGame() {
            document.getElementById('visual-novel').style.display = 'none';
            document.getElementById('driving-game').style.display = 'block';
            
            // Очистить другие машины
            otherCars.forEach(car => car.remove());
            otherCars = [];
            score = 0;
            
            // Таймер для создания машин
            carSpawnTimer = setInterval(spawnCar, 2000);
            
            // Игровой цикл
            gameLoop();
        }

        // Создать машину
        function spawnCar() {
            const car = document.createElement('div');
            car.className = 'other-car';
            car.style.left = Math.random() * 680 + 'px';
            car.style.top = '-100px';
            document.getElementById('road').appendChild(car);
            otherCars.push(car);
        }

        // Игровой цикл
        function gameLoop() {
            // Движение игрока
            const playerCar = document.getElementById('player-car');
            if (keyState['ArrowLeft'] && parseInt(playerCar.style.left) > 20) {
                playerCar.style.left = parseInt(playerCar.style.left) - carSpeed + 'px';
            }
            if (keyState['ArrowRight'] && parseInt(playerCar.style.left) < 720) {
                playerCar.style.left = parseInt(playerCar.style.left) + carSpeed + 'px';
            }
            
            // Движение других машин и проверка столкновений
            for (let i = otherCars.length - 1; i >= 0; i--) {
                const car = otherCars[i];
                const top = parseInt(car.style.top);
                car.style.top = top + carSpeed + 'px';
                
                // Проверка столкновения
                if (checkCollision(playerCar, car)) {
                    endDrivingGame(false);
                    return;
                }
                
                // Удаление машин за экраном
                if (top > 600) {
                    car.remove();
                    otherCars.splice(i, 1);
                    score++;
                    
                    // Проверка на победу
                    if (score >= 10) {
                        endDrivingGame(true);
                        return;
                    }
                }
            }
            
            requestAnimationFrame(gameLoop);
        }

        // Проверка столкновений
        function checkCollision(rect1, rect2) {
            const rect1Left = parseInt(rect1.style.left);
            const rect1Top = parseInt(rect1.style.top);
            const rect2Left = parseInt(rect2.style.left);
            const rect2Top = parseInt(rect2.style.top);
            
            return !(
                rect1Left > rect2Left + 60 || 
                rect1Left + 60 < rect2Left || 
                rect1Top > rect2Top + 100 || 
                rect1Top + 100 < rect2Top
            );
        }

        // Завершить игру с вождением
        function endDrivingGame(success) {
            clearInterval(carSpawnTimer);
            
            if (success) {
                currentScene = 13;
            } else {
                currentScene = "over";
            }
            
            setTimeout(function() {
                document.getElementById('driving-game').style.display = 'none';
                document.getElementById('visual-novel').style.display = 'block';
                showScene(currentScene);
            }, 1000);
        }

        // Конец игры
        function endGame(win) {
            document.getElementById('visual-novel').style.display = 'none';
            document.getElementById('washing-game').style.display = 'none';
            document.getElementById('driving-game').style.display = 'none';
            
            if (win) {
                document.getElementById('game-win').style.display = 'flex';
            } else {
                document.getElementById('game-over').style.display = 'flex';
            }
        }

        // Запуск игры при загрузке страницы
        window.onload = initGame;
    </script>
</body>
</html>
