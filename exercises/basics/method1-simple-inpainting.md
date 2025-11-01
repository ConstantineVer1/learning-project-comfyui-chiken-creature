# Практическое упражнение: Simple Inpainting Method

## Цель
Научиться базовому методу создания гибрида через inpainting - самый простой, но эффективный способ.

## Что мы будем делать
1. Генерировать курицу в полный рост
2. Вырезать голову из фото человека через SAM
3. Создать маску области головы курицы
4. Объединить человеческую голову с телом курицы
5. Сгладить переход через inpainting

---

## Подготовка

### Шаг 1: Установите необходимые custom nodes

Через ComfyUI Manager найдите и установите:

1. **ComfyUI-Impact-Pack**
   - Основной пакет для SAM сегментации
   - Содержит: SAMLoader (Impact), SAMDetectorCombined, BBoxDetectorCombined, SEGSToImageList, MaskToSEGS
   - URL: https://github.com/ltdrdata/ComfyUI-Impact-Pack

2. **ComfyUI-Impact-Subpack** (ОБЯЗАТЕЛЬНО!)
   - Дополнительные детекторы для Impact Pack
   - Содержит: UltralyticsDetectorProvider, предобученные YOLO модели
   - URL: https://github.com/ltdrdata/ComfyUI-Impact-Subpack
   - ВАЖНО: Устанавливается отдельно от Impact-Pack!

3. **WAS Node Suite**
   - Утилиты для работы с масками и изображениями
   - Содержит: ColorCorrect, ImageLevels, MaskBlur, ImageToMask
   - URL: https://github.com/WASasquatch/was-node-suite-comfyui

4. **ComfyUI_Comfyroll_CustomNodes** (опционально)
   - Дополнительные утилиты с масками
   - Содержит: CR Image Mask, CR Apply Mask

### Шаг 2: Скачайте модели

#### 1. Stable Diffusion 1.5 (для генерации курицы)
```
Модель: v1-5-pruned-emaonly.ckpt
URL: https://huggingface.co/runwayml/stable-diffusion-v1-5/resolve/main/v1-5-pruned-emaonly.ckpt
Папка: ComfyUI/models/checkpoints/
Размер: ~4 GB
Используется: Stage 1 - генерация базовой курицы
```

#### 2. SD 1.5 Inpainting Model (для сглаживания)
```
Модель: sd-v1-5-inpainting.ckpt
URL: https://huggingface.co/runwayml/stable-diffusion-inpainting/resolve/main/sd-v1-5-inpainting.ckpt
Папка: ComfyUI/models/checkpoints/
Размер: ~4 GB
Используется: Stage 5 - inpainting для сглаживания переходов
```

#### 3. SAM Model (для сегментации головы)
```
Модель: sam_vit_h_4b8939.pth (наиболее точный)
URL: https://dl.fbaipublicfiles.com/segment_anything/sam_vit_h_4b8939.pth
Папка: ComfyUI/models/sams/
Размер: ~2.4 GB
Альтернатива (быстрее): sam_vit_b_01ec64.pth (375 MB)
URL: https://dl.fbaipublicfiles.com/segment_anything/sam_vit_b_01ec64.pth
```

#### 4. Ultralytics Face Detection Model (для автоматической детекции лиц)
```
Модель: face_yolov8m.pt
Установка: Через ComfyUI Impact Pack Manager (встроенный загрузчик моделей)
Или вручную:
URL: https://huggingface.co/Bingsu/adetailer/resolve/main/face_yolov8m.pt
Папка: ComfyUI/models/ultralytics/bbox/
Размер: ~52 MB
Используется: BBoxDetectorCombined → SAMDetectorCombined для автоматической детекции лица
```

---

## ПОЛНЫЙ WORKFLOW - Пошаговая инструкция

### STAGE 1: Генерация базовой курицы

**Nodes и их connections:**

```
1. CheckpointLoaderSimple
   Параметры:
   - ckpt_name: "v1-5-pruned-emaonly.ckpt"
   Outputs: MODEL, CLIP, VAE

2. EmptyLatentImage
   Параметры:
   - width: 512
   - height: 768
   - batch_size: 1
   Output: LATENT

3. CLIPTextEncode (Positive)
   Connections:
   - clip: из CheckpointLoaderSimple → CLIP
   Параметры:
   - text: "full body anthropomorphic chicken standing upright, humanoid posture, wearing simple clothes, white background, centered, professional photo, full height"
   Output: CONDITIONING

4. CLIPTextEncode (Negative)
   Connections:
   - clip: из CheckpointLoaderSimple → CLIP
   Параметры:
   - text: "cropped, partial body, multiple chickens, text, watermark, blurry, deformed, closeup"
   Output: CONDITIONING

5. KSampler
   Connections:
   - model: из CheckpointLoaderSimple → MODEL
   - positive: из CLIPTextEncode (Positive) → CONDITIONING
   - negative: из CLIPTextEncode (Negative) → CONDITIONING
   - latent_image: из EmptyLatentImage → LATENT
   Параметры:
   - seed: (random, например 42)
   - steps: 20
   - cfg: 7.0
   - sampler_name: "euler"
   - scheduler: "normal"
   - denoise: 1.0
   Output: LATENT

6. VAEDecode
   Connections:
   - samples: из KSampler → LATENT
   - vae: из CheckpointLoaderSimple → VAE
   Output: IMAGE

7. SaveImage
   Connections:
   - images: из VAEDecode → IMAGE
   Параметры:
   - filename_prefix: "chicken_base"

8. LoadImage (для дальнейшего использования)
   Параметры:
   - image: выберите сохраненное изображение курицы из output
   Outputs: IMAGE, MASK
```

**ВАЖНО**: После генерации курицы сохраните изображение и используйте LoadImage для загрузки его в следующие stages.

---

### STAGE 2: Детекция и сегментация человеческой головы

**Вариант А: Автоматическая сегментация через Ultralytics + SAM (Impact Pack)**

```
9. LoadImage (человеческое фото)
   Параметры:
   - image: ваше фото с человеком
   Output: IMAGE, MASK

10. SAMLoader (Impact) - НЕ SAMModelLoader!
    Параметры:
    - model_name: "sam_vit_h (2.56GB)"
    - device_mode: "AUTO"
    Output: SAM_MODEL

11. UltralyticsDetectorProvider (from Impact SUBPACK - НЕ Impact Pack!)
    Параметры:
    - model_name: "face_yolov8m.pt"
    Output: BBOX_DETECTOR

    ВАЖНО: Этот node из ComfyUI-Impact-Subpack, который устанавливается ОТДЕЛЬНО!

12. BBoxDetectorCombined (from Impact Pack)
    Connections:
    - bbox_detector: из UltralyticsDetectorProvider → BBOX_DETECTOR
    - image: из LoadImage (шаг 9) → IMAGE
    Параметры:
    - threshold: 0.5
    - dilation: 10
    - crop_factor: 3.0
    - drop_size: 10
    Output: SEGS (bounding boxes сегменты)

13. SAMDetectorCombined (from Impact Pack)
    Connections:
    - sam_model: из SAMLoader (шаг 10) → SAM_MODEL
    - segs: из BBoxDetectorCombined (шаг 12) → SEGS
    - image: из LoadImage (шаг 9) → IMAGE
    Параметры:
    - detection_hint: "center-1"
    - dilation: 0
    - threshold: 0.93
    - bbox_expansion: 0
    - mask_hint_threshold: 0.7
    Output: SEGS (refined сегменты с точными масками от SAM)

14. SEGSToImageList (from Impact Pack)
    Connections:
    - segs: из SAMDetectorCombined → SEGS
    Output: IMAGE (список изображений сегментов)

15. Используйте первое изображение из списка (индекс 0)
    - Если несколько лиц детектировано, выберите нужное
```

**Вариант Б: Ручная маска (простой способ)**

```
9. LoadImage (человеческое фото)
   Параметры:
   - image: ваше фото
   Output: IMAGE, MASK

10. LoadImageMask
    Параметры:
    - image: файл маски (белая голова на черном фоне, созданная в GIMP/Photoshop)
    Output: MASK
```

---

### STAGE 3: Извлечение и масштабирование головы

**Для Варианта А (автоматическая сегментация):**

```
16. SEGSToImageList уже вернул IMAGE с извлеченной головой (шаг 14)
    - Голова уже обрезана и изолирована
    - Если детектировано несколько лиц, выберите нужное

17. ImageScale
    Connections:
    - image: из SEGSToImageList (шаг 14) → IMAGE
    Параметры:
    - width: 150 (подберите под размер головы курицы)
    - height: 180 (примерно, сохраняя пропорции)
    - upscale_method: "lanczos"
    - crop: "disabled"
    Output: IMAGE (масштабированная голова)
```

**Для Варианта Б (ручная маска):**

```
16. ImageScale
    Connections:
    - image: из LoadImage (шаг 9) → IMAGE (исходное фото)
    Параметры:
    - width: целевой размер для композитинга
    - height: пропорционально
    - upscale_method: "lanczos"
    - crop: "disabled"
    Output: IMAGE
```

---

### STAGE 4: Создание маски для головы курицы

**ВАЖНО**: Нам нужна маска ОБЛАСТИ КУРИЦЫ, куда вставить человеческую голову.

**Вариант 1: Ручная маска (рекомендуется для начала)**

```
18. Создайте маску вручную:
    - Откройте изображение курицы в GIMP/Photoshop/Krita
    - Создайте новый слой
    - Закрасьте БЕЛЫМ область головы курицы
    - Остальное оставьте ЧЕРНЫМ
    - Примените Gaussian Blur 8-16 pixels к маске
    - Сохраните как PNG: "chicken_head_mask.png"

19. LoadImageMask
    Параметры:
    - image: "chicken_head_mask.png"
    Output: MASK
```

**Вариант 2: Ручное выделение через ImageToMask (WAS Node Suite)**

```
18. ImageToMask (WAS Node Suite)
    Connections:
    - image: LoadImage курицы (шаг 8) → IMAGE
    Параметры:
    - channel: "red" (или выберите канал с головой курицы)
    - method: "threshold" или "intensity"
    Output: MASK

    Примечание: Этот метод работает если голова курицы имеет
    отличающийся цвет от тела. Может потребовать обработки.

19. MaskBlur (WAS Node Suite)
    Connections:
    - mask: из ImageToMask → MASK
    Параметры:
    - blur_radius: 8
    Output: MASK
```

**Вариант 3: Использование встроенного MaskEditor ComfyUI**

```
18. MaskEditor (встроенный node)
    Connections:
    - image: LoadImage курицы (шаг 8) → IMAGE
    Параметры:
    - Нарисуйте маску вручную в интерфейсе ComfyUI
    - Закрасьте область головы курицы
    Output: MASK

19. MaskBlur
    Connections:
    - mask: из MaskEditor → MASK
    Параметры:
    - blur_radius: 8
    Output: MASK
```

---

### STAGE 5: Композитинг - объединение головы с телом

```
21. ImageCompositeMasked
    Connections:
    - destination: LoadImage курицы (шаг 8) → IMAGE
    - source: ImageScale человеческой головы (шаг 17) → IMAGE
    - mask: Маска головы курицы (шаг 19 или 20) → MASK
    Параметры:
    - x: 180 (координата X, где разместить голову - ПОДБЕРИТЕ!)
    - y: 50 (координата Y - ПОДБЕРИТЕ!)
    - resize_source: false (уже масштабировали в шаге 17)
    Output: IMAGE (композит)

Как определить x, y:
1. Используйте Preview Image для курицы
2. Найдите центр головы курицы (например, x=250, y=120)
3. x_offset = center_x - (human_head_width / 2) = 250 - 75 = 175
4. y_offset = center_y - (human_head_height / 2) = 120 - 90 = 30
5. Подберите точные значения итеративно

22. SaveImage
    Connections:
    - images: из ImageCompositeMasked → IMAGE
    Параметры:
    - filename_prefix: "composite_before_inpaint"
```

---

### STAGE 6: Inpainting для сглаживания перехода

```
23. CheckpointLoaderSimple (ВТОРОЙ загрузчик!)
    Параметры:
    - ckpt_name: "sd-v1-5-inpainting.ckpt"
    Outputs: MODEL, CLIP, VAE

24. CLIPTextEncode (Positive для inpainting)
    Connections:
    - clip: из CheckpointLoaderSimple inpainting → CLIP
    Параметры:
    - text: "smooth natural transition between human skin and chicken feathers, seamless blend, natural neck connection, fantasy creature, professional render"
    Output: CONDITIONING

25. CLIPTextEncode (Negative для inpainting)
    Connections:
    - clip: из CheckpointLoaderSimple inpainting → CLIP
    Параметры:
    - text: "harsh edge, visible seam, artifacts, unnatural, poorly blended"
    Output: CONDITIONING

26. Создайте маску переходной зоны (шея/граница):
    Вариант А - Ручная маска:
    - Откройте композит
    - Закрасьте БЕЛЫМ область шеи (где нужно сгладить)
    - Blur 16 pixels
    - Сохраните как "transition_mask.png"

    LoadImageMask
    - image: "transition_mask.png"
    Output: MASK

27. InpaintModelConditioning
    Connections:
    - positive: из CLIPTextEncode positive inpainting (шаг 24) → CONDITIONING
    - negative: из CLIPTextEncode negative inpainting (шаг 25) → CONDITIONING
    - vae: из CheckpointLoaderSimple inpainting (шаг 23) → VAE
    - pixels: из ImageCompositeMasked (шаг 21) → IMAGE
    - mask: маска перехода (шаг 26) → MASK
    Outputs: CONDITIONING (positive), CONDITIONING (negative), LATENT

28. KSampler (для inpainting)
    Connections:
    - model: из CheckpointLoaderSimple inpainting (шаг 23) → MODEL
    - positive: из InpaintModelConditioning → CONDITIONING (positive)
    - negative: из InpaintModelConditioning → CONDITIONING (negative)
    - latent_image: из InpaintModelConditioning → LATENT
    Параметры:
    - seed: random
    - steps: 20
    - cfg: 7.0
    - sampler_name: "euler"
    - scheduler: "normal"
    - denoise: 0.5 (НАЧНИТЕ С ЭТОГО, потом экспериментируйте)
    Output: LATENT

29. VAEDecode
    Connections:
    - samples: из KSampler inpainting (шаг 28) → LATENT
    - vae: из CheckpointLoaderSimple inpainting (шаг 23) → VAE
    Output: IMAGE

30. SaveImage
    Connections:
    - images: из VAEDecode → IMAGE
    Параметры:
    - filename_prefix: "chicken_human_final"
```

---

## Опциональные улучшения

### Коррекция освещения и цвета (перед композитингом)

```
Вставить между шагом 17 (ImageScale) и шагом 21 (ImageCompositeMasked):

17a. ColorCorrect (WAS Node Suite)
     Connections:
     - image: из ImageScale (шаг 17) → IMAGE
     Параметры:
     - temperature: -5 to +5 (подберите)
     - brightness: -0.1 to +0.1
     - contrast: 0.9 to 1.1
     - saturation: 0.9 to 1.1
     Output: IMAGE

Или используйте ColorMatch для автоматического подбора:

17b. ColorMatch (WAS)
     Connections:
     - image: человеческая голова из ImageScale
     - reference: изображение курицы
     Параметры:
     - method: "match_histogram"
     Output: IMAGE
```

---

## Параметры для экспериментов

### Тест 1: Denoise strength (в KSampler inpainting, шаг 28)
- **0.3** - минимальные изменения, сохраняет оригинал
- **0.5** - сбалансированно (НАЧНИТЕ ОТСЮДА)
- **0.7** - сильное изменение, больше творчества
- **0.9** - почти полная перегенерация области

### Тест 2: Размер маски перехода (blur_radius в MaskBlur)
- **4 pixels** - резкий переход, видна граница
- **8 pixels** - мягкий переход (рекомендуется)
- **16 pixels** - очень плавный
- **32 pixels** - размытый, может затронуть много области

### Тест 3: CFG Scale для inpainting (в KSampler, шаг 28)
- **5** - больше свободы AI, креативнее
- **7** - баланс (стандарт)
- **10** - строгое следование промпту

---

## Проблемы и решения

### Проблема 1: Голова слишком большая/маленькая
**Решение:**
- Измените параметры `width` и `height` в ImageScale (шаг 17)
- Попробуйте разные значения: 120, 150, 180, 200 pixels
- Сохраняйте пропорции

### Проблема 2: Разное освещение головы и тела
**Решение:**
- Добавьте ColorCorrect ДО композитинга (шаг 17a)
- Или используйте ColorMatch для автоматического подбора
- Параметры:
  - brightness: подберите чтобы совпала яркость
  - temperature: теплее (+) или холоднее (-)

### Проблема 3: Резкая граница, видна склейка
**Решение:**
1. Увеличьте blur маски перехода (шаг 26) до 16-24 pixels
2. Увеличьте denoise в inpainting до 0.6-0.7
3. Расширьте область маски перехода (закрасьте больше зоны шеи)

### Проблема 4: Inpainting портит детали лица
**Решение:**
- Уменьшите denoise до 0.3-0.4
- Сузьте маску перехода - только область шеи, НЕ лицо
- Проверьте что маска не захватывает лицо

### Проблема 5: Голова не на месте
**Решение:**
- Подберите координаты x, y в ImageCompositeMasked (шаг 21)
- Используйте Preview Image чтобы видеть результат
- Increment по 10 pixels за раз: попробуйте x+10, x-10, y+10, y-10

---

## Домашнее задание

### Уровень 1: Базовый
1. Создайте 3 разных курицы-основы (разные seeds в шаге 5)
2. Попробуйте 2 разных фото людей
3. Найдите оптимальный denoise для вашего случая
4. Запишите параметры лучшего результата

### Уровень 2: Улучшения
1. Добавьте ColorCorrect для подбора освещения
2. Экспериментируйте с размерами масок перехода
3. Попробуйте разные prompts для inpainting
4. Создайте 3 варианта с разным качеством склейки

### Уровень 3: Вариации
1. Курица в костюме с галстуком (измените prompt шага 3)
2. Курица-воин в доспехах
3. Курица-маг в мантии
4. Для каждого варианта подберите свой inpainting prompt

---

## Метрики успеха

Для каждого созданного результата запишите:

- [ ] **Время**: сколько заняла полная генерация
- [ ] **Попытки**: сколько раз пришлось переделывать
- [ ] **Параметры**:
  - Размер головы (width, height в шаге 17)
  - Позиция (x, y в шаге 21)
  - Denoise (шаг 28)
  - Blur маски (шаг 26)
- [ ] **Проблемы**: что пошло не так
- [ ] **Улучшения**: что можно сделать лучше

---

## Вопросы для понимания

### 1. Почему мы используем два разных checkpoint'а?
- **Ответ**: Standard SD для генерации, Inpainting SD для сглаживания
- **Почему**: Inpainting модель специально обучена работать с масками и смешивать области

### 2. Что происходит при denoise = 1.0 vs 0.0?
- **1.0**: Полная регенерация области маски, игнорирует оригинал
- **0.0**: Никаких изменений, возвращает оригинал
- **0.5**: Баланс между сохранением и изменением

### 3. Как mask blur влияет на результат?
- **0 pixels**: Резкая граница, видна склейка
- **8 pixels**: Плавный переход, естественный
- **32 pixels**: Очень размытая граница, может затронуть много области

### 4. Почему InpaintModelConditioning принимает IMAGE, а не LATENT?
- **Ответ**: Узел сам кодирует изображение внутри с учетом маски
- **Это важно**: Он создает специальный latent для inpainting моделей

### 5. Зачем нужны CropMaskToBounds и ImageScale?
- **CropMaskToBounds**: Обрезает изображение до размера маски, убирает лишний фон
- **ImageScale**: Масштабирует голову под размер головы курицы
- **Без этого**: Композитили бы все фото целиком, не только голову

---

## Следующий шаг

После освоения этого метода:
1. Экспериментируйте с разными фотографиями
2. Попробуйте другие существа (кошки, собаки, драконы)
3. Изучите Method 2 с ControlNet для более точного контроля
4. Освойте продвинутые техники color grading и lighting

**Важно**: Сохраняйте успешные workflow в .json файлы через ComfyUI "Save" функцию!
