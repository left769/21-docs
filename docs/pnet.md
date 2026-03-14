🚧 РОЗДІЛ В РОЗРОБЦІ, ІНФОРМАЦІЯ НЕ АКТУАЛЬНА

<!DOCTYPE html>
<html lang="uk">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Інструкція роботи з PnetLAB</title>
    <style>
        @page {
            size: A4;
            margin: 20mm;
        }
        
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Arial, sans-serif;
            font-weight: 600;
            line-height: 1.6;
            color: #1a1a1a;
            max-width: 210mm;
            margin: 0 auto;
            padding: 20px;
            background: white;
        }
        
        .header {
            text-align: center;
            margin-bottom: 30px;
            padding-bottom: 20px;
            border-bottom: 3px solid #f6821f;
        }
        
        h1 {
            color: #f6821f;
            font-size: 28px;
            margin-bottom: 10px;
            font-weight: 700;
        }
        
        .subtitle {
            color: #666;
            font-size: 14px;
            font-weight: 600;
        }
        
        h2 {
            color: #f6821f;
            font-size: 20px;
            margin-top: 30px;
            margin-bottom: 15px;
            padding-left: 10px;
            border-left: 4px solid #f6821f;
            font-weight: 700;
        }
        
        h3 {
            color: #333;
            font-size: 16px;
            margin-top: 20px;
            margin-bottom: 10px;
            font-weight: 700;
        }
        
        .step {
            background: #f8f9fa;
            padding: 15px;
            margin: 15px 0;
            border-radius: 8px;
            border-left: 4px solid #f6821f;
        }
        
        .step-number {
            display: inline-block;
            background: #f6821f;
            color: white;
            width: 30px;
            height: 30px;
            border-radius: 50%;
            text-align: center;
            line-height: 30px;
            margin-right: 10px;
            font-weight: 700;
        }
        
        .note {
            background: #fff3cd;
            border-left: 4px solid #ffc107;
            padding: 12px;
            margin: 15px 0;
            border-radius: 4px;
        }
        
        .important {
            background: #f8d7da;
            border-left: 4px solid #dc3545;
            padding: 12px;
            margin: 15px 0;
            border-radius: 4px;
        }
        
        .success {
            background: #d4edda;
            border-left: 4px solid #28a745;
            padding: 12px;
            margin: 15px 0;
            border-radius: 4px;
        }
        
        .info {
            background: #d1ecf1;
            border-left: 4px solid #0dcaf0;
            padding: 12px;
            margin: 15px 0;
            border-radius: 4px;
        }
        
        code {
            background: #e9ecef;
            padding: 2px 6px;
            border-radius: 3px;
            font-family: 'Courier New', monospace;
            font-size: 13px;
            font-weight: 600;
        }
        
        .link {
            color: #0066cc;
            text-decoration: none;
            font-weight: 600;
        }
        
        .link:hover {
            text-decoration: underline;
        }
        
        ul {
            margin: 10px 0;
            padding-left: 25px;
        }
        
        li {
            margin: 8px 0;
        }
        
        .footer {
            margin-top: 40px;
            padding-top: 20px;
            border-top: 2px solid #e9ecef;
            text-align: center;
            color: #666;
            font-size: 12px;
        }
        
        .comparison-table {
            width: 100%;
            border-collapse: collapse;
            margin: 20px 0;
            background: white;
        }
        
        .comparison-table th {
            background: #f6821f;
            color: white;
            padding: 12px;
            text-align: left;
            font-weight: 700;
        }
        
        .comparison-table td {
            padding: 12px;
            border: 1px solid #dee2e6;
        }
        
        .comparison-table tr:nth-child(even) {
            background: #f8f9fa;
        }
        
        .device-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
            gap: 15px;
            margin: 20px 0;
        }
        
        .device-card {
            background: #f8f9fa;
            padding: 15px;
            border-radius: 8px;
            border-left: 4px solid #f6821f;
            text-align: center;
        }
        
        .device-icon {
            font-size: 32px;
            margin-bottom: 10px;
        }
        
        .software-list {
            background: #e7f3ff;
            padding: 15px;
            border-radius: 8px;
            margin: 20px 0;
        }
        
        .software-item {
            margin: 10px 0;
            padding: 10px;
            background: white;
            border-radius: 5px;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        
        @media print {
            body {
                print-color-adjust: exact;
                -webkit-print-color-adjust: exact;
            }
            
            .step, .note, .important, .success, .info, .software-list {
                page-break-inside: avoid;
            }
            
            h2 {
                page-break-after: avoid;
            }
            
            .button-container {
                display: none;
            }
        }
        
        .button {
            display: inline-block;
            background: #f6821f;
            color: white;
            padding: 10px 20px;
            border-radius: 5px;
            text-decoration: none;
            font-weight: 700;
            margin: 10px 5px;
            cursor: pointer;
            border: none;
        }
        
        .button:hover {
            background: #e67313;
        }
    </style>
</head>
<body>
    <div class="header">
        <h1>Інструкція з роботи в лабораторному середовищі PnetLAB</h1>
        <div class="subtitle">Віртуальна лабораторія мережевих технологій | viti.edu.ua</div>
    </div>

    <div class="important">
        <strong>⚠️ Обов'язкова умова:</strong> Для доступу до PnetLAB необхідно мати активне підключення через Cloudflare WARP. Переконайтеся, що WARP увімкнено перед початком роботи.
    </div>

    <h2>1. Перший вхід до системи</h2>
    
    <div class="step">
        <span class="step-number">1</span>
        <strong>Переконайтеся, що WARP активний</strong><br>
        Перевірте, що іконка Cloudflare WARP у системному треї показує активне з'єднання (зелений або синій колір).
    </div>
    
    <div class="step">
        <span class="step-number">2</span>
        <strong>Відкрийте PnetLAB у браузері</strong><br>
        Перейдіть за адресою: <a href="https://pnet.viti.edu.ua/" class="link">https://pnet.viti.edu.ua/</a><br>
        Рекомендовані браузери: Google Chrome, Mozilla Firefox, Microsoft Edge.
    </div>
    
    <div class="step">
        <span class="step-number">3</span>
        <strong>Введіть облікові дані</strong><br>
        Використовуйте логін та пароль, які надав викладач:<br>
        • <strong>Username:</strong> ваш логін<br>
        • <strong>Password:</strong> ваш пароль<br>
        Натисніть кнопку "Login".
    </div>
    
    <div class="step">
        <span class="step-number">4</span>
        <strong>Налаштуйте профіль (при першому вході)</strong><br>
        Після першого входу рекомендується змінити пароль:<br>
        • Клацніть на ваше ім'я користувача у правому верхньому куті<br>
        • Перейдіть в "Profile" або "Профіль"<br>
        • Введіть новий пароль двічі та збережіть
    </div>

    <div class="note">
        <strong>📝 Примітка:</strong> Зберігайте свої облікові дані в надійному місці. Не передавайте їх іншим людям.
    </div>

    <h2>2. Інтерфейс PnetLAB</h2>
    
    <div class="info">
        Після входу ви побачите головну сторінку з вашими лабораторними роботами. Основні елементи інтерфейсу:
        <ul>
            <li><strong>Folders</strong> — папки з лабораторними роботами</li>
            <li><strong>Labs</strong> — список доступних лабораторних робіт</li>
            <li><strong>Topology</strong> — топологія мережі (схема з'єднань)</li>
            <li><strong>Devices</strong> — список пристроїв у лабораторній роботі</li>
        </ul>
    </div>

    <h2>3. Робота з лабораторними роботами</h2>
    
    <h3>3.1 Відкриття лабораторної роботи</h3>
    
    <div class="step">
        <span class="step-number">1</span>
        <strong>Знайдіть потрібну лабораторну роботу</strong><br>
        У лівому меню оберіть папку з вашим курсом або групою. Клацніть на назву лабораторної роботи, яку потрібно виконати.
    </div>
    
    <div class="step">
        <span class="step-number">2</span>
        <strong>Відкрийте топологію</strong><br>
        Після відкриття лабораторної роботи ви побачите схему мережі з усіма пристроями та з'єднаннями між ними.
    </div>
    
    <div class="step">
        <span class="step-number">3</span>
        <strong>Запустіть необхідні пристрої</strong><br>
        Щоб запустити пристрій:<br>
        • Клацніть правою кнопкою миші на пристрої<br>
        • Оберіть "Start" або "Запустити"<br>
        • Дочекайтеся, поки статус пристрою зміниться на "Running" (зелений колір)
    </div>

    <div class="note">
        <strong>💡 Порада:</strong> Щоб запустити всі пристрої одразу, використовуйте кнопку "Start all" у верхній панелі інструментів.
    </div>

    <h3>3.2 Типи пристроїв у лабораторних роботах</h3>
    
    <div class="device-grid">
        <div class="device-card">
            <div class="device-icon">🔀</div>
            <strong>Cisco Router</strong><br>
            Маршрутизатори
        </div>
        <div class="device-card">
            <div class="device-icon">🔁</div>
            <strong>Cisco Switch</strong><br>
            Комутатори
        </div>
        <div class="device-card">
            <div class="device-icon">🛡️</div>
            <strong>Cisco ASA/FTD</strong><br>
            Міжмережеві екрани
        </div>
        <div class="device-card">
            <div class="device-icon">📡</div>
            <strong>MikroTik</strong><br>
            RouterOS пристрої
        </div>
        <div class="device-card">
            <div class="device-icon">🔐</div>
            <strong>FortiGate</strong><br>
            Fortinet Firewall
        </div>
        <div class="device-card">
            <div class="device-icon">🌐</div>
            <strong>pfSense</strong><br>
            Firewall/Router
        </div>
        <div class="device-card">
            <div class="device-icon">🖥️</div>
            <strong>Windows Server</strong><br>
            Windows Server 2022
        </div>
        <div class="device-card">
            <div class="device-icon">🐧</div>
            <strong>Linux</strong><br>
            Різні дистрибутиви
        </div>
        <div class="device-card">
            <div class="device-icon">💻</div>
            <strong>VPC</strong><br>
            Віртуальні ПК
        </div>
    </div>

    <h2>4. Підключення до пристроїв</h2>
    
    <h3>4.1 HTML Console vs Стандартна консоль</h3>
    
    <table class="comparison-table">
        <thead>
            <tr>
                <th>Характеристика</th>
                <th>HTML Console</th>
                <th>Стандартна консоль</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td><strong>Де працює</strong></td>
                <td>Безпосередньо в браузері</td>
                <td>Відкриває зовнішню програму на вашому комп'ютері</td>
            </tr>
            <tr>
                <td><strong>Додаткове ПЗ</strong></td>
                <td>Не потрібно</td>
                <td>Потрібен Telnet/SSH клієнт (PuTTY) або VNC клієнт</td>
            </tr>
            <tr>
                <td><strong>Зручність</strong></td>
                <td>Швидкий доступ, все в одному вікні</td>
                <td>Більше функцій, можливість копіювання/вставки</td>
            </tr>
            <tr>
                <td><strong>Коли використовувати</strong></td>
                <td>Для швидких налаштувань та перевірок</td>
                <td>Для складних конфігурацій та довготривалої роботи</td>
            </tr>
        </tbody>
    </table>

    <h3>4.2 Підключення через HTML Console</h3>
    
    <div class="step">
        <span class="step-number">1</span>
        <strong>Клацніть правою кнопкою на пристрої</strong><br>
        На топології клацніть правою кнопкою миші на потрібному пристрої.
    </div>
    
    <div class="step">
        <span class="step-number">2</span>
        <strong>Оберіть "HTML Console"</strong><br>
        У меню, що з'явилося, виберіть опцію "HTML Console" або "HTML5 Console".
    </div>
    
    <div class="step">
        <span class="step-number">3</span>
        <strong>Працюйте в консолі</strong><br>
        Відкриється нове вікно з консоллю пристрою безпосередньо в браузері. Можете вводити команди та налаштовувати пристрій.
    </div>

    <div class="note">
        <strong>💡 Порада:</strong> HTML Console зручна для швидкого доступу, але може мати обмеження при копіюванні/вставці великих конфігурацій.
    </div>

    <h3>4.3 Підключення через стандартну консоль</h3>
    
    <div class="step">
        <span class="step-number">1</span>
        <strong>Клацніть правою кнопкою на пристрої</strong><br>
        На топології клацніть правою кнопкою миші на потрібному пристрої.
    </div>
    
    <div class="step">
        <span class="step-number">2</span>
        <strong>Оберіть тип підключення</strong><br>
        Залежно від типу пристрою, оберіть:<br>
        • <strong>Telnet</strong> — для Cisco, MikroTik та інших CLI-пристроїв<br>
        • <strong>VNC</strong> — для пристроїв з графічним інтерфейсом (Windows, Linux GUI)<br>
        • <strong>SSH</strong> — для захищеного підключення (якщо налаштовано)
    </div>
    
    <div class="step">
        <span class="step-number">3</span>
        <strong>Використовуйте відповідний клієнт</strong><br>
        Система автоматично спробує відкрити підключення у встановленій програмі на вашому комп'ютері.
    </div>

    <h3>4.4 Необхідне програмне забезпечення</h3>
    
    <div class="software-list">
        <h3>Обов'язкові програми для роботи зі стандартною консоллю:</h3>
        
        <div class="software-item">
            <div>
                <strong>🔧 PuTTY</strong><br>
                <span style="font-size: 13px;">Telnet/SSH клієнт для підключення до мережевих пристроїв</span>
            </div>
            <a href="https://www.putty.org/" class="link" target="_blank">Завантажити</a>
        </div>
        
        <div class="software-item">
            <div>
                <strong>🖥️ VNC Viewer (RealVNC або TightVNC)</strong><br>
                <span style="font-size: 13px;">Клієнт для віддаленого доступу до графічного інтерфейсу</span>
            </div>
            <a href="https://www.realvnc.com/en/connect/download/viewer/" class="link" target="_blank">RealVNC</a> | 
            <a href="https://www.tightvnc.com/download.php" class="link" target="_blank">TightVNC</a>
        </div>
        
        <div class="software-item">
            <div>
                <strong>🦈 Wireshark (опційно)</strong><br>
                <span style="font-size: 13px;">Аналізатор мережевого трафіку для захоплення та аналізу пакетів</span>
            </div>
            <a href="https://www.wireshark.org/download.html" class="link" target="_blank">Завантажити</a>
        </div>
    </div>

    <div class="important">
        <strong>⚠️ Важливо:</strong> Встановіть ці програми завчасно, перед початком виконання лабораторних робіт. Переконайтеся, що вони правильно налаштовані та працюють.
    </div>

    <h2>5. Робота з конфігурацією</h2>
    
    <h3>5.1 Налаштування пристроїв</h3>
    
    <div class="step">
        <span class="step-number">1</span>
        <strong>Підключіться до пристрою</strong><br>
        Використовуйте HTML Console або стандартну консоль для доступу до CLI пристрою.
    </div>
    
    <div class="step">
        <span class="step-number">2</span>
        <strong>Виконайте налаштування</strong><br>
        Введіть необхідні команди згідно з завданням лабораторної роботи. Виконуйте налаштування покроково, перевіряючи результат після кожного етапу.
    </div>
    
    <div class="step">
        <span class="step-number">3</span>
        <strong>Перевірте конфігурацію</strong><br>
        Використовуйте команди перевірки (наприклад, <code>show running-config</code>, <code>show ip interface brief</code>) для контролю налаштувань.
    </div>

    <h3>5.2 Збереження конфігурації</h3>
    
    <div class="important">
        <strong>🚨 КРИТИЧНО ВАЖЛИВО!</strong><br>
        Не забувайте зберігати конфігурацію пристроїв після налаштування! Це найпоширеніша помилка студентів.
    </div>
    
    <div class="step">
        <span class="step-number">1</span>
        <strong>Збережіть конфігурацію на пристрої</strong><br>
        Після завершення налаштувань обов'язково збережіть конфігурацію у постійну пам'ять пристрою, використовуючи відповідні команди для вашого типу обладнання.
    </div>
    
    <div class="step">
        <span class="step-number">2</span>
        <strong>Збережіть стан лабораторної роботи</strong><br>
        У PnetLAB натисніть кнопку "Save" або "Commit" на панелі інструментів, щоб зберегти поточний стан всієї лабораторної роботи.
    </div>

    <div class="note">
        <strong>💡 Порада:</strong> Зберігайте конфігурацію регулярно, особливо після важливих змін. Це убезпечить вас від втрати роботи при несподіваних збоях.
    </div>

    <h2>6. Здача виконаної роботи</h2>
    
    <div class="step">
        <span class="step-number">1</span>
        <strong>Переконайтеся, що все працює</strong><br>
        Перевірте роботу всієї топології, протестуйте зв'язок між пристроями, переконайтеся, що всі завдання виконані.
    </div>
    
    <div class="step">
        <span class="step-number">2</span>
        <strong>Збережіть конфігурації та зробіть скріншоти</strong><br>
        Залежно від вимог завдання, підготуйте необхідні матеріали для здачі.
    </div>
    
    <div class="step">
        <span class="step-number">3</span>
        <strong>Здайте роботу</strong><br>
        Надайте викладачу результати виконання роботи так, як він визначив (Google Classroom, електронна пошта, інша платформа).
    </div>

    <div class="info">
        <strong>📋 Що зазвичай потрібно здати:</strong>
        <ul>
            <li>Скріншоти налаштованої топології</li>
            <li>Скріншоти результатів команд перевірки</li>
            <li>Файли конфігурацій пристроїв (якщо потрібно)</li>
            <li>Скріншоти успішних тестів зв'язку (ping, traceroute)</li>
        </ul>
    </div>

    <h2>7. Завершення роботи</h2>
    
    <div class="step">
        <span class="step-number">1</span>
        <strong>Збережіть всі зміни</strong><br>
        Переконайтеся, що всі конфігурації збережені як на пристроях, так і в самій лабораторній роботі.
    </div>
    
    <div class="step">
        <span class="step-number">2</span>
        <strong>Зупиніть пристрої (опційно)</strong><br>
        Якщо ви закінчили роботу, можете зупинити пристрої: клацніть правою кнопкою → "Stop" або використайте "Stop all" на панелі інструментів.
    </div>
    
    <div class="step">
        <span class="step-number">3</span>
        <strong>Вийдіть з системи</strong><br>
        Клацніть на ваше ім'я у правому верхньому куті та оберіть "Logout" або "Вийти".
    </div>

    <h2>8. Вирішення типових проблем</h2>
    
    <h3>Проблема: Не можу підключитися до PnetLAB</h3>
    <ul>
        <li>Перевірте, чи активний Cloudflare WARP</li>
        <li>Переконайтеся, що ви використовуєте правильну адресу: <code>https://pnet.viti.edu.ua/</code></li>
        <li>Очистіть кеш браузера та спробуйте знову</li>
        <li>Спробуйте інший браузер</li>
    </ul>
    
    <h3>Проблема: Пристрій не запускається</h3>
    <ul>
        <li>Дочекайтеся, поки завершиться запуск попередніх пристроїв</li>
        <li>Перевірте, чи не досягнуто ліміт запущених пристроїв</li>
        <li>Спробуйте зупинити інші непотрібні пристрої</li>
        <li>Якщо проблема залишається, зверніться до викладача</li>
    </ul>
    
    <h3>Проблема: Не працює HTML Console</h3>
    <ul>
        <li>Оновіть сторінку браузера (F5)</li>
        <li>Перевірте, чи не блокує браузер спливаючі вікна</li>
        <li>Використайте стандартну консоль (Telnet/VNC) як альтернативу</li>
    </ul>
    
    <h3>Проблема: Втратив налаштування після перезапуску</h3>
    <ul>
        <li><strong>Основна причина:</strong> Не збережена конфігурація на пристрої!</li>
        <li>Завжди зберігайте конфігурацію у постійну пам'ять пристрою</li>
        <li>Використовуйте функцію "Save" в PnetLAB після завершення роботи</li>
    </ul>
    
    <h3>Проблема: Не відкривається Telnet/VNC</h3>
    <ul>
        <li>Переконайтеся, що встановлено PuTTY або VNC Viewer</li>
        <li>Перевірте, чи правильно налаштовані асоціації файлів у Windows</li>
        <li>Використовуйте HTML Console як альтернативу</li>
    </ul>

    <div class="note">
        <strong>📞 Потрібна допомога?</strong><br>
        Якщо у вас виникли проблеми, які не описані в цій інструкції, зверніться до викладача або IT-підтримки. Опишіть проблему детально та вкажіть, які кроки ви вже спробували.
    </div>

    <h2>9. Корисні поради</h2>
    
    <div class="success">
        <strong>✅ Рекомендації для ефективної роботи:</strong>
        <ul>
            <li>Завжди починайте з ознайомлення з топологією перед запуском пристроїв</li>
            <li>Запускайте лише ті пристрої, з якими працюєте в даний момент</li>
            <li>Зберігайте конфігурацію після кожного важливого етапу налаштування</li>
            <li>Використовуйте блокнот для записування виконаних команд</li>
            <li>Робіть скріншоти проміжних результатів для звіту</li>
            <li>Перевіряйте кожне налаштування перед переходом до наступного</li>
            <li>Не соромтеся ставити питання викладачу, якщо щось незрозуміло</li>
            <li>Плануйте час: складні лабораторні можуть зайняти кілька годин</li>
        </ul>
    </div>

    <div class="footer">
        <p><strong>PnetLAB — Інструкція для студентів</strong></p>
        <p>Віртуальна лабораторія мережевих технологій | viti.edu.ua</p>
        <p>© 2026 Всі права захищені</p>
    </div>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
    
    <script>
        async function generatePDF() {
            const { jsPDF } = window.jspdf;
            const pdf = new jsPDF('p', 'mm', 'a4');
            
            const content = document.body;
            const buttonContainer = document.querySelector('.button-container');
            if (buttonContainer) buttonContainer.style.display = 'none';
            
            const canvas = await html2canvas(content, {
                scale: 2,
                useCORS: true,
                logging: false,
                width: content.scrollWidth,
                height: content.scrollHeight
            });
            
            const imgData = canvas.toDataURL('image/png');
            const imgWidth = 210;
            const pageHeight = 297;
            const imgHeight = (canvas.height * imgWidth) / canvas.width;
            let heightLeft = imgHeight;
            let position = 0;
            
            pdf.addImage(imgData, 'PNG', 0, position, imgWidth, imgHeight);
            heightLeft -= pageHeight;
            
            while (heightLeft > 0) {
                position = heightLeft - imgHeight;
                pdf.addPage();
                pdf.addImage(imgData, 'PNG', 0, position, imgWidth, imgHeight);
                heightLeft -= pageHeight;
            }
            
            const links = document.querySelectorAll('a');
            links.forEach(link => {
                const rect = link.getBoundingClientRect();
                const x = (rect.left / canvas.width) * imgWidth;
                const y = ((rect.top % pageHeight) / canvas.height) * imgHeight;
                const w = (rect.width / canvas.width) * imgWidth;
                const h = (rect.height / canvas.height) * imgHeight;
                
                if (link.href) {
                    pdf.link(x, y, w, h, { url: link.href });
                }
            });
            
            if (buttonContainer) buttonContainer.style.display = 'flex';
            
            pdf.save('Інструкція_PnetLAB.pdf');
        }
        
        function printPDF() {
            window.print();
        }
        
        window.addEventListener('load', function() {
            const container = document.createElement('div');
            container.className = 'button-container';
            container.style.position = 'fixed';
            container.style.top = '20px';
            container.style.right = '20px';
            container.style.zIndex = '1000';
            container.style.display = 'flex';
            container.style.gap = '10px';
            
            const pdfButton = document.createElement('button');
            pdfButton.textContent = '📄 PDF з посиланнями';
            pdfButton.className = 'button';
            pdfButton.onclick = generatePDF;
            
            const printButton = document.createElement('button');
            printButton.textContent = '🖨️ Друк';
            printButton.className = 'button';
            printButton.onclick = printPDF;
            
            container.appendChild(pdfButton);
            container.appendChild(printButton);
            document.body.appendChild(container);
        });
    </script>
</body>
</html>