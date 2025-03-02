
# Flask Jenkins CI/CD Pipeline

## Описание
Этот репозиторий содержит простое Flask-приложение с одним эндпоинтом и тестами.  
Проект интегрирован с Jenkins для автоматизации процесса доставки кода на сервер, запуска тестов и развертывания в виде systemd-сервиса.

## 📂 Структура проекта

```
.
├── app.py                  # Flask-приложение
├── requirements.txt        # Зависимости Python
├── tests.py                 # Юнит-тесты
├── Jenkinsfile              # Конфигурация Jenkins Pipeline
└── README.md
```

---

## 🚀 Описание CI/CD процесса

### Основной процесс включает:
1. Получение кода из репозитория.
2. Обновление рабочей директории на сервере.
3. Настройку виртуального окружения (venv).
4. Установку зависимостей.
5. Запуск тестов.
6. Перезапуск Flask-приложения через systemd.
7. Уведомление в Telegram о результате выполнения пайплайна.

---

## ⚙️ Окружения
Пайплайн поддерживает два окружения:

- **dev** — можно развернуть из любой ветки.
- **prod** — можно развернуть только из ветки `main`.

При запуске пайплайна в Jenkins можно выбрать параметр `ENV` (dev/prod).

---

## 📦 Настройка системы (GCP VM)

### 1. Создание директории
На сервере создаётся рабочая директория:

```
/opt/flask-jenkins
```

### 2. Права на директорию
Папка принадлежит пользователю Jenkins:

```bash
sudo chown -R jenkins:jenkins /opt/flask-jenkins
```

---

## 🧰 Запуск и управление приложением
Приложение разворачивается как systemd-сервис:

### Файл сервиса
`/etc/systemd/system/flask-jenkins.service`

### Пример содержимого
```ini
[Unit]
Description=Flask Jenkins Application
After=network.target

[Service]
User=jenkins
WorkingDirectory=/opt/flask-jenkins
ExecStart=/opt/flask-jenkins/venv/bin/python /opt/flask-jenkins/app.py --host=0.0.0.0 --port=5001
Restart=always

[Install]
WantedBy=multi-user.target
```

### Команды управления
```bash
sudo systemctl start flask-jenkins     # Запуск
sudo systemctl stop flask-jenkins      # Остановка
sudo systemctl restart flask-jenkins   # Перезапуск
sudo systemctl status flask-jenkins    # Проверить статус
```

---

## 🔗 Интеграция с Jenkins
### Пайплайн
Процесс автоматизирован с помощью Jenkins Pipeline, который хранится в файле:

```
Jenkinsfile
```

### Этапы
- Получение кода (git pull)
- Настройка окружения
- Запуск тестов
- Перезапуск systemd-сервиса
- Отправка уведомления в Telegram

### Параметры
| Параметр | Описание | Значение |
|---|---|---|
| ENV | Окружение (dev/prod) | dev, prod |

### Автозапуск
Каждый пуш в репозиторий вызывает запуск Jenkins Pipeline через GitHub Webhook.

---

## 📬 Уведомления
По завершению каждого билда отправляется уведомление в Telegram с информацией:
- Статус билда (успех/ошибка)
- Хеш последнего коммита
- Автор коммита
- Ветка
- Выбранное окружение (dev/prod)

---

## 📎 Полезные команды для сервера
```bash
# Проверить логи systemd-сервиса
sudo journalctl -u flask-jenkins -f
