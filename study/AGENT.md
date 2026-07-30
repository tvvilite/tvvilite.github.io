# Проект: Plants vs. Zombies (Mini HTML5)

Этот файл содержит исходный код простой браузерной версии популярной игры. 

## Механика игры:
* **Сетка:** 5 рядов и 8 колонок.
* **Солнце (☀️):** Ресурс для постройки. Вы получаете его автоматически со временем (в этой версии оно просто копится).
* **Растение (🌱):** Стоит 50 солнц. Стреляет горошинами (🟢). Чтобы посадить растение, кликните по свободной клетке на зеленом поле.
* **Зомби (🧟):** Идут справа налево. Если зомби дойдет до левого края, игра окончена. Горошины наносят им урон.

## Исходный код (index.html)

Скопируйте приведенный ниже код, сохраните его в файл с расширением `.html` и откройте в браузере.

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Растения против Зомби - HTML5 Мини-игра</title>
    <style>
        body {
            margin: 0;
            padding: 20px;
            background-color: #222;
            color: #fff;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        h1 {
            margin: 0 0 10px 0;
        }
        #game-container {
            position: relative;
        }
        canvas {
            background-color: #7CB342; /* Цвет газона */
            border: 5px solid #33691E;
            box-shadow: 0 10px 30px rgba(0,0,0,0.5);
            cursor: pointer;
        }
        #ui-bar {
            width: 800px;
            display: flex;
            justify-content: space-between;
            background: #558B2F;
            padding: 10px 20px;
            box-sizing: border-box;
            border: 5px solid #33691E;
            border-bottom: none;
            font-size: 24px;
            font-weight: bold;
        }
        #game-over {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: rgba(0, 0, 0, 0.8);
            padding: 30px;
            border-radius: 10px;
            text-align: center;
            display: none;
        }
        button {
            margin-top: 15px;
            padding: 10px 20px;
            font-size: 18px;
            cursor: pointer;
            background: #4CAF50;
            color: white;
            border: none;
            border-radius: 5px;
        }
        button:hover {
            background: #45a049;
        }
    </style>
</head>
<body>

    <h1>🌱 Растения против Зомби 🧟</h1>
    
    <div id="game-container">
        <div id="ui-bar">
            <span>Солнце: <span id="sun-count">50</span> ☀️</span>
            <span>Цена растения: 50 ☀️</span>
        </div>
        <canvas id="gameCanvas" width="800" height="500"></canvas>
        <div id="game-over">
            <h2>ЗОМБИ СЪЕЛИ ВАШИ МОЗГИ!</h2>
            <button onclick="location.reload()">Играть снова</button>
        </div>
    </div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const sunCountElement = document.getElementById('sun-count');
        const gameOverScreen = document.getElementById('game-over');

        // Настройки сетки
        const cellSize = 100;
        const rows = 5;
        const cols = 8;

        // Игровые переменные
        let sun = 50;
        let frame = 0;
        let gameOver = false;
        let score = 0;

        const plants = [];
        const zombies = [];
        const projectiles = [];

        // Класс Растения (Горохострел)
        class Plant {
            constructor(x, y) {
                this.x = x;
                this.y = y;
                this.width = cellSize;
                this.height = cellSize;
                this.health = 100;
                this.timer = 0;
            }
            draw() {
                ctx.fillStyle = '#00FF00';
                ctx.font = '50px Arial';
                ctx.fillText('🌱', this.x + 20, this.y + 70);
            }
            update() {
                this.timer++;
                // Выстрел каждые 100 кадров (примерно раз в 1.5 сек)
                if (this.timer % 100 === 0) {
                    projectiles.push(new Projectile(this.x + 50, this.y + 35));
                }
            }
        }

        // Класс Зомби
        class Zombie {
            constructor(y) {
                this.x = canvas.width;
                this.y = y;
                this.width = cellSize;
                this.height = cellSize;
                this.speed = Math.random() * 0.3 + 0.2; // Случайная скорость
                this.health = 100;
            }
            draw() {
                ctx.font = '50px Arial';
                ctx.fillText('🧟', this.x + 20, this.y + 70);
                
                // Полоска здоровья
                ctx.fillStyle = 'red';
                ctx.fillRect(this.x + 20, this.y + 10, 50, 5);
                ctx.fillStyle = 'green';
                ctx.fillRect(this.x + 20, this.y + 10, 50 * (this.health / 100), 5);
            }
            update() {
                this.x -= this.speed;
            }
        }

        // Класс Снаряда (Горошина)
        class Projectile {
            constructor(x, y) {
                this.x = x;
                this.y = y;
                this.speed = 5;
                this.power = 25; // Урон
                this.radius = 10;
            }
            draw() {
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fillStyle = '#76FF03';
                ctx.fill();
                ctx.strokeStyle = '#33691E';
                ctx.stroke();
                ctx.closePath();
            }
            update() {
                this.x += this.speed;
            }
        }

        // Рисуем сетку газона (шахматный порядок для красоты)
        function drawGrid() {
            for (let y = 0; y < canvas.height; y += cellSize) {
                for (let x = 0; x < canvas.width; x += cellSize) {
                    if ((x / cellSize + y / cellSize) % 2 === 0) {
                        ctx.fillStyle = 'rgba(0, 0, 0, 0.1)';
                        ctx.fillRect(x, y, cellSize, cellSize);
                    }
                }
            }
        }

        // Управление ресурсами
        function handleSun() {
            if (frame % 300 === 0) { // Каждые 5 секунд даем солнце
                sun += 25;
                sunCountElement.innerText = sun;
            }
        }

        // Посадка растений по клику
        canvas.addEventListener('click', function(e) {
            if (gameOver) return;
            const rect = canvas.getBoundingClientRect();
            const mouseX = e.clientX - rect.left;
            const mouseY = e.clientY - rect.top;

            const gridX = Math.floor(mouseX / cellSize) * cellSize;
            const gridY = Math.floor(mouseY / cellSize) * cellSize;

            // Проверка, есть ли уже растение
            let isOccupied = plants.some(p => p.x === gridX && p.y === gridY);

            if (!isOccupied && sun >= 50) {
                plants.push(new Plant(gridX, gridY));
                sun -= 50;
                sunCountElement.innerText = sun;
            }
        });

        // Основной игровой цикл
        function animate() {
            if (gameOver) {
                gameOverScreen.style.display = 'block';
                return;
            }

            ctx.clearRect(0, 0, canvas.width, canvas.height);
            drawGrid();

            // Спавн зомби
            if (frame % 400 === 0) {
                let randomRow = Math.floor(Math.random() * rows) * cellSize;
                zombies.push(new Zombie(randomRow));
            }

            handleSun();

            // Отрисовка и обновление растений
            plants.forEach((plant, index) => {
                plant.update();
                plant.draw();
            });

            // Отрисовка и обновление снарядов
            for (let i = projectiles.length - 1; i >= 0; i--) {
                let p = projectiles[i];
                p.update();
                p.draw();

                // Удаляем снаряд, если он улетел за экран
                if (p.x > canvas.width) {
                    projectiles.splice(i, 1);
                }
            }

            // Отрисовка и обновление зомби, обработка столкновений
            for (let i = zombies.length - 1; i >= 0; i--) {
                let z = zombies[i];
                z.update();
                z.draw();

                // Проигрыш, если зомби дошел до левого края
                if (z.x < 0) {
                    gameOver = true;
                }

                // Столкновение снаряда с зомби
                for (let j = projectiles.length - 1; j >= 0; j--) {
                    let p = projectiles[j];
                    if (p.x > z.x && p.x < z.x + 50 && p.y > z.y && p.y < z.y + cellSize) {
                        z.health -= p.power;
                        projectiles.splice(j, 1);
                    }
                }

                // Столкновение зомби с растением (зомби ест растение)
                for (let k = plants.length - 1; k >= 0; k--) {
                    let plant = plants[k];
                    if (z.x < plant.x + cellSize && z.x + 50 > plant.x && z.y === plant.y) {
                        z.x = plant.x + cellSize; // Зомби останавливается
                        plant.health -= 0.5; // Урон растению
                        if (plant.health <= 0) {
                            plants.splice(k, 1);
                        }
                    }
                }

                // Смерть зомби
                if (z.health <= 0) {
                    zombies.splice(i, 1);
                    score += 10;
                }
            }

            frame++;
            requestAnimationFrame(animate);
        }

        // Запуск игры
        animate();
    </script>
</body>
</html>