# Промпт-инженерия для Курицы-Существа

## Базовые промпты

### 1. Простая мультяшная курица
**Positive prompt:**
```
cartoon chicken, simple design, bright colors, friendly expression, standing pose
```
**Negative prompt:**
```
realistic, photograph, complex background, text, watermark
```

### 2. Фэнтези курица-воин
**Positive prompt:**
```
fantasy chicken warrior, armor pieces, heroic pose, feathered helmet, sword, epic lighting, digital painting style, detailed plumage, golden and red colors
```
**Negative prompt:**
```
photo, modern, guns, mechanical, robot, blurry, bad anatomy
```

### 3. Магическая курица-маг
**Positive prompt:**
```
magical chicken wizard, flowing robes, glowing staff, mystical aura, spell effects, purple and blue color scheme, fantasy art, detailed feathers with magical sparkles
```
**Negative prompt:**
```
realistic chicken, farm, mundane, no magic, photographic, low quality
```

### 4. Киберпанк курица
**Positive prompt:**
```
cyberpunk chicken, neon feathers, glowing cybernetic implants, futuristic city background, holographic wings, LED eyes, synthwave colors, digital art
```
**Negative prompt:**
```
medieval, fantasy, natural, organic only, no technology, rustic
```

### 5. Стимпанк курица-изобретатель
**Positive prompt:**
```
steampunk chicken inventor, brass goggles, mechanical wings, gears and cogs, Victorian style, copper and bronze colors, workshop background, detailed mechanical parts
```
**Negative prompt:**
```
modern, digital, clean, minimalist, no mechanical parts
```

## Продвинутые техники промптинга

### Весовые коэффициенты (Attention/Emphasis)
```
(chicken:1.2), (cute:0.8), ((fluffy feathers:1.5))
```
- Одинарные скобки () = умножение веса на 1.1
- Двойные скобки (()) = умножение на 1.21
- Число после двоеточия = точный вес

### Смешивание стилей
```
[cartoon|realistic] chicken - чередование между стилями
(cartoon:0.7) (realistic:0.3) chicken - взвешенное смешивание
```

### Поэтапная генерация
```
[chicken:dragon:0.5] - начать с курицы, перейти к дракону на 50% шагов
```

## Упражнения по промптингу

### Задание 1: Эмоциональный спектр
Создайте 5 изображений курицы с разными эмоциями:
1. Happy/cheerful chicken
2. Angry/fierce chicken
3. Sad/melancholic chicken
4. Surprised/shocked chicken
5. Wise/contemplative chicken

### Задание 2: Стилевое разнообразие
Сгенерируйте курицу в разных художественных стилях:
1. Studio Ghibli style chicken
2. Disney Pixar style chicken
3. Tim Burton style dark chicken
4. Watercolor painting chicken
5. Low poly 3D style chicken

### Задание 3: Окружение и контекст
Поместите курицу в различные сеттинги:
1. Chicken in enchanted forest
2. Chicken in space station
3. Chicken in underwater kingdom
4. Chicken in volcanic lair
5. Chicken in cozy library

### Задание 4: Гибридизация
Создайте курицу-гибрид с другими существами:
1. Chicken-dragon hybrid
2. Chicken-phoenix fusion
3. Chicken-unicorn creature
4. Chicken-griffin mix
5. Chicken-peacock blend

## Чек-лист качественного промпта

- [ ] Основной объект четко определен
- [ ] Указан художественный стиль
- [ ] Описана цветовая схема
- [ ] Добавлены детали (глаза, перья, поза)
- [ ] Указано освещение (если важно)
- [ ] Определен фон или его отсутствие
- [ ] Negative prompt исключает нежелательные элементы
- [ ] Использованы весовые коэффициенты для важных деталей

## Метрики для оценки результатов

1. **Соответствие промпту** (1-10)
2. **Художественное качество** (1-10)
3. **Оригинальность** (1-10)
4. **Техническое исполнение** (1-10)
5. **Эмоциональное воздействие** (1-10)

## Домашнее задание

1. Создайте "Character Sheet" курицы-существа:
   - Front view
   - Side view
   - Back view
   - 3/4 view
   - Close-up головы
   - Детали: крылья, лапы, хвост

2. Разработайте 3 уникальных варианта курицы-существа:
   - Молодая курица (chick/juvenile)
   - Взрослая курица (adult)
   - Легендарная курица (legendary/boss)

3. Создайте "эволюционную цепочку":
   - Обычная курица → Магическая курица → Божественная курица-феникс

Запишите все использованные промпты, параметры и seed для воспроизводимости!