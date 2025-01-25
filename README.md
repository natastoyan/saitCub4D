<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>4D Искажённый Куб</title>
    <style>
        body {
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            background-color: #000;
            perspective: 1000px; /* Начальная перспектива */
            overflow: hidden;
            user-select: none; /* Запрет выделения текста */
        }

        .scene {
            width: 200px;
            height: 200px;
            position: relative;
            transform-style: preserve-3d;
            margin-bottom: 40px; /* Увеличиваем отступ снизу */
            transition: transform 0.3s ease-out; /* Плавное изменение расстояния */
        }

        .cube {
            width: 100%;
            height: 100%;
            position: absolute;
            transform-style: preserve-3d;
            transform: translateZ(0); /* Начальное положение камеры */
        }

        .face {
            position: absolute;
            width: 200px;
            height: 200px;
            background-color: rgba(255, 255, 255, 0.1);
            border: 2px solid rgba(255, 255, 255, 0.5);
            display: flex;
            justify-content: center;
            align-items: center;
            user-select: none; /* Запрет выделения текста */
            transition: opacity 0.5s ease-out, background-color 0.5s ease-out, transform 0.5s ease-out; /* Плавное изменение масштаба */
        }

        .front  { transform: translateZ(100px); }
        .back   { transform: rotateY(180deg) translateZ(100px); }
        .right  { transform: rotateY(90deg) translateZ(100px); }
        .left   { transform: rotateY(-90deg) translateZ(100px); }
        .top    { transform: rotateX(90deg) translateZ(100px); }
        .bottom { transform: rotateX(-90deg) translateZ(100px); }

        .controls {
            display: flex;
            gap: 10px;
            margin-top: 20px;
        }

        .controls button {
            padding: 10px 20px;
            font-size: 16px;
            cursor: pointer;
            background-color: #333;
            border: none;
            color: white;
            border-radius: 5px;
            transition: background-color 0.3s ease;
        }

        .controls button:hover {
            background-color: #555;
        }
    </style>
</head>
<body>
    <div class="scene">
        <div class="cube">
            <div class="face front"></div>
            <div class="face back"></div>
            <div class="face right"></div>
            <div class="face left"></div>
            <div class="face top"></div>
            <div class="face bottom"></div>
        </div>
    </div>

    <div class="controls">
        <button onclick="toggle4DEffect()">4D Эффект</button>
        <button onclick="toggleColorChange()">Случайный цвет</button>
        <button onclick="toggleOpacityChange()">Прозрачность</button>
        <button onclick="resetCube()">Сброс</button>
    </div>

    <script>
        const scene = document.querySelector('.scene');
        const cube = document.querySelector('.cube');
        const faces = document.querySelectorAll('.face');
        let isDragging = false;
        let startX, startY;
        let rotationX = 0, rotationY = 0;
        let targetRotationX = 0, targetRotationY = 0;
        let velocityX = 0, velocityY = 0;
        const friction = 0.92; // Коэффициент трения для замедления
        const sensitivity = 0.1; // Уменьшенная чувствительность вращения
        const smoothness = 0.1; // Увеличенная плавность вращения
        let is4DEffectEnabled = false; // Флаг для 4D-эффекта
        let isColorChangeEnabled = false; // Флаг для изменения цвета
        let isOpacityChangeEnabled = false; // Флаг для изменения прозрачности
        let cameraDistance = 0; // Начальное расстояние камеры

        // Обработчик начала перетаскивания (мышь)
        cube.addEventListener('mousedown', (e) => {
            isDragging = true;
            startX = e.clientX;
            startY = e.clientY;
        });

        // Обработчик движения мыши
        document.addEventListener('mousemove', (e) => {
            if (!isDragging) return;

            const deltaX = e.clientX - startX;
            const deltaY = e.clientY - startY;

            // Вычисляем скорость вращения на основе движения мыши
            velocityX = deltaY * sensitivity;
            velocityY = deltaX * sensitivity;

            // Обновляем целевые углы поворота
            targetRotationX -= velocityX;
            targetRotationY += velocityY;

            // Обновляем начальные координаты
            startX = e.clientX;
            startY = e.clientY;
        });

        // Обработчик отпускания мыши
        document.addEventListener('mouseup', () => {
            isDragging = false;
        });

        // Обработчик начала перетаскивания (сенсор)
        cube.addEventListener('touchstart', (e) => {
            isDragging = true;
            const touch = e.touches[0]; // Получаем первое касание
            startX = touch.clientX;
            startY = touch.clientY;
        });

        // Обработчик движения пальца (сенсор)
        document.addEventListener('touchmove', (e) => {
            if (!isDragging) return;

            const touch = e.touches[0]; // Получаем первое касание
            const deltaX = touch.clientX - startX;
            const deltaY = touch.clientY - startY;

            // Вычисляем скорость вращения на основе движения пальца
            velocityX = deltaY * sensitivity;
            velocityY = deltaX * sensitivity;

            // Обновляем целевые углы поворота
            targetRotationX -= velocityX;
            targetRotationY += velocityY;

            // Обновляем начальные координаты
            startX = touch.clientX;
            startY = touch.clientY;
        });

        // Обработчик окончания касания (сенсор)
        document.addEventListener('touchend', () => {
            isDragging = false;
        });

        // Обработчик колеса мыши для изменения расстояния камеры
        document.addEventListener('wheel', (e) => {
            e.preventDefault(); // Отключаем стандартное поведение прокрутки
            const delta = e.deltaY * -0.5; // Инвертируем значение для интуитивного управления

            // Ограничиваем минимальное и максимальное расстояние камеры
            cameraDistance = Math.max(-500, Math.min(500, cameraDistance + delta));

            // Применяем новое расстояние камеры
            scene.style.transform = `translateZ(${cameraDistance}px)`;
        }, { passive: false });

        // Функция для плавного вращения и замедления
        function animate() {
            if (!isDragging) {
                // Замедляем скорость вращения
                velocityX *= friction;
                velocityY *= friction;

                // Обновляем целевые углы поворота
                targetRotationX -= velocityX;
                targetRotationY += velocityY;
            }

            // Плавное "догоняющее" вращение
            rotationX += (targetRotationX - rotationX) * smoothness;
            rotationY += (targetRotationY - rotationY) * smoothness;

            // Применяем поворот к кубу
            cube.style.transform = `rotateX(${rotationX}deg) rotateY(${rotationY}deg)`;

            // Добавляем 4D искажение: изменение масштаба
            if (is4DEffectEnabled) {
                faces.forEach(face => {
                    // Медленное изменение масштаба
                    const scale = 1 + Math.sin(rotationX * Math.PI / 180) * 0.05; // Уменьшенный масштаб (0.05 вместо 0.1)
                    face.style.transform = `${window.getComputedStyle(face).transform} scale(${scale})`;
                });
            }

            // Добавляем плавное изменение цвета
            if (isColorChangeEnabled) {
                faces.forEach(face => {
                    const hue = (rotationX + rotationY) % 360; // Цветовой тон
                    face.style.backgroundColor = `hsla(${hue}, 100%, 50%, 0.2)`;
                });
            }

            // Добавляем изменение прозрачности в зависимости от вращения
            if (isOpacityChangeEnabled) {
                faces.forEach(face => {
                    const opacity = Math.abs(Math.sin((rotationX + rotationY) * Math.PI / 180)); // Прозрачность на основе вращения
                    face.style.opacity = opacity;
                });
            }

            // Запрашиваем следующий кадр анимации
            requestAnimationFrame(animate);
        }

        animate();

        // Функция для включения/выключения 4D-эффекта
        function toggle4DEffect() {
            is4DEffectEnabled = !is4DEffectEnabled;
        }

        // Функция для включения/выключения изменения цвета
        function toggleColorChange() {
            isColorChangeEnabled = !isColorChangeEnabled;
            if (!isColorChangeEnabled) {
                // Возвращаем стандартный цвет, если изменение цвета отключено
                faces.forEach(face => {
                    face.style.backgroundColor = `rgba(255, 255, 255, 0.1)`;
                });
            }
        }

        // Функция для включения/выключения изменения прозрачности
        function toggleOpacityChange() {
            isOpacityChangeEnabled = !isOpacityChangeEnabled;
            if (!isOpacityChangeEnabled) {
                // Возвращаем стандартную прозрачность, если изменение прозрачности отключено
                faces.forEach(face => {
                    face.style.opacity = 1;
                });
            }
        }

        // Функция для сброса куба в исходное состояние
        function resetCube() {
            // Сбрасываем вращение
            rotationX = 0;
            rotationY = 0;
            targetRotationX = 0;
            targetRotationY = 0;
            velocityX = 0;
            velocityY = 0;

            // Сбрасываем эффекты
            is4DEffectEnabled = false;
            isColorChangeEnabled = false;
            isOpacityChangeEnabled = false;

            // Возвращаем стандартные значения
            cube.style.transform = `rotateX(0deg) rotateY(0deg)`;
            faces.forEach(face => {
                face.style.transform = ''; // Сбрасываем масштаб
                face.style.backgroundColor = `rgba(255, 255, 255, 0.1)`; // Возвращаем стандартный цвет
                face.style.opacity = 1; // Возвращаем стандартную прозрачность
            });

            // Сбрасываем расстояние камеры
            cameraDistance = 0;
            scene.style.transform = `translateZ(${cameraDistance}px)`;
        }
    </script>
</body>
</html>
