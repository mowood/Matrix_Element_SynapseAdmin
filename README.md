🚀 Полная локальная установка Matrix + Element + Synapse Admin c помощью Docker + Portainer

# Шаг 1: Подготовка на сервере

## Залогиньтесь на сервер и создайте рабочую директорию
```
mkdir ~/matrix-server
cd ~/matrix-server
```

## Создайте директорию для данных Synapse
```
mkdir synapse-data
chmod 755 synapse-data
```
# Шаг 2: Генерация конфигурации Synapse

## Сгенерируйте базовую конфигурацию
```
docker run --rm \
  -v $(pwd)/synapse-data:/data \
  -e SYNAPSE_SERVER_NAME=YOUR_IP \
  -e SYNAPSE_REPORT_STATS=no \
  matrixdotorg/synapse:latest generate
```
## Проверьте что файлы создались
`ls -la synapse-data/`
# Шаг 3: Исправление конфига для PostgreSQL
 
## Создайте правильный конфиг homeserver.yaml
```
docker run --rm -v $(pwd)/synapse-data:/data -it alpine sh -c 'cat > /data/homeserver.yaml' << 'EOF'
server_name: "YOUR_IP"
report_stats: false
pid_file: /data/homeserver.pid

listeners:
  - port: 8008
    tls: false
    type: http
    x_forwarded: true
    resources:
      - names: [client, federation]
        compress: false

database:
  name: psycopg2
  args:
    user: synapse
    password: matrix_password_123
    database: synapse
    host: postgres
    cp_min: 5
    cp_max: 10

enable_registration: true
registration_shared_secret: "change_this_to_random_string_123"

max_upload_size: "100M"
allow_public_rooms: true
admin_contact: 'admin@YOUR_IP'

media_store_path: /data/media_store
uploads_path: /data/uploads

trusted_key_servers:
  - server_name: "matrix.org"

suppress_key_server_warning: false
EOF
```

# Шаг 4: Настройка в Portainer
Откройте Portainer: http://YOUR_IP:9000

Перейдите в Stacks → Add stack

Название стека: matrix-server

Выберите "Web editor"

Вставьте этот скаченный docker-compose.yml или создайте новый

Нажмите "Deploy the stack"

# Шаг 5: Проверка запуска
В Portainer перейдите в Containers и проверьте статусы:

✅ matrix-postgres - должен быть зеленым

✅ matrix-synapse - должен быть зеленым после 1-2 минут

✅ matrix-element - должен быть зеленым

✅ matrix-synapse-admin - должен быть зеленым

# Шаг 6: Создание администратора
 
## Дождитесь полного запуска Synapse (2-3 минуты)
`sleep 180`

## Создайте администратора
```
docker exec -it matrix-synapse register_new_matrix_user \
  http://localhost:8008 \
  -c /data/homeserver.yaml \
  -u admin \
  -p admin_password_123 \
  -a
```
# Шаг 7: Проверка доступа
После успешного создания администратора откройте в браузере:

🌐 Element Web: http://YOUR_IP:8080

⚙️ Synapse Admin: http://YOUR_IP:8765

🔧 Synapse API: http://YOUR_IP:8008

# 🔧 Если возникли проблемы
Проверка логов:

## В Portainer или через терминал
`docker logs matrix-synapse --tail 50`
Проверка конфига:

## Убедитесь что конфиг правильный
`docker exec matrix-synapse cat /data/homeserver.yaml | grep -A 10 "database:"`

# 🌐 Способы входа и создания пользователей
## Способ 1: Через Element Web (рекомендуется)
Откройте Element Web: http://YOUR_IP:8080

На странице входа:

Нажмите "Edit" рядом с "Sign in to matrix.org"

Введите ваш сервер: YOUR_IP

Нажмите "Continue"

Создание аккаунта:

Нажмите "Create account"

Username: например admin (или любое другое имя)

Password: надежный пароль

Confirm password: повторите пароль

Нажмите "Register"

# Способ 2: Через команды (если регистрация отключена)
 
## Создание пользователя через консоль
```
docker exec -it matrix-synapse register_new_matrix_user \
  http://localhost:8008 \
  -c /data/homeserver.yaml \
  -u username \
  -p password \
  --no-admin
```
## Пример создания тестового пользователя
```
docker exec -it matrix-synapse register_new_matrix_user \
  http://localhost:8008 \
  -c /data/homeserver.yaml \
  -u testuser \
  -p testpassword123 \
  --no-admin
```
# Способ 3: Создание администратора

## Создание пользователя с правами администратора
```
docker exec -it matrix-synapse register_new_matrix_user \
  http://localhost:8008 \
  -c /data/homeserver.yaml \
  -u admin \
  -p admin_password_123 \
  -a
```
# 🛠 Администрирование через Synapse Admin
Откройте Synapse Admin: http://YOUR_IP:8765

Войдите с учетной записью администратора

Возможности админки:

📊 Просмотр статистики сервера

👥 Управление пользователями

🏠 Управление комнатами

⚙️ Настройки сервера

🔧 Если команда создания администратора не работает
Попробуйте альтернативный синтаксис:

 
## Альтернативный способ
`docker exec -it matrix-synapse  `

## Внутри контейнера:
`register_new_matrix_user http://localhost:8008 -c /data/homeserver.yaml -u admin -p admin123 -a`

## Или если это не работает:
`python -m synapse.app.homeserver --config-path /data/homeserver.yaml &`
`sleep 10`
`register_new_matrix_user http://localhost:8008 -c /data/homeserver.yaml -u admin -p admin123 -a`
📋 Быстрая проверка доступности сервера
 
## Проверьте что Synapse отвечает
`curl http://localhost:8008/_matrix/client/versions`

## Проверьте что Element доступен
`curl -I http://localhost:8080`

## Проверьте что Synapse Admin доступен  
`curl -I http://localhost:8765`
