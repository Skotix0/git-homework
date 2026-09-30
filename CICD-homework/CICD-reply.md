# Домашнее задание к занятию "Что такое DevOps. CI/CD" - Борисенко Андрей


### Задание 1

**Что нужно сделать:**

1. Установите себе jenkins по инструкции из лекции или любым другим способом из официальной документации. Использовать Docker в этом задании нежелательно.
2. Установите на машину с jenkins [golang](https://golang.org/doc/install).
3. Используя свой аккаунт на GitHub, сделайте себе форк [репозитория](https://github.com/netology-code/sdvps-materials.git). В этом же репозитории находится [дополнительный материал для выполнения ДЗ](https://github.com/netology-code/sdvps-materials/blob/main/CICD/8.2-hw.md).
4. Создайте в jenkins Freestyle Project, подключите получившийся репозиторий к нему и произведите запуск тестов и сборку проекта ```go test .``` и  ```docker build .```.

*В качестве ответа пришлите скриншоты с настройками проекта и результатами выполнения сборки.*

#### Ответ на задание 1:

Настройки:

![1](https://github.com/Skotix0/homeworks-netology/blob/main/CICD-homework/screenshots/%D0%9D%D0%B0%D1%81%D1%82%D1%80%D0%BE%D0%B9%D0%BA%D0%B8-1%20%D0%B7%D0%B0%D0%B4%D0%B0%D0%BD%D0%B8%D0%B5%201.png)
![2](https://github.com/Skotix0/homeworks-netology/blob/main/CICD-homework/screenshots/%D0%9D%D0%B0%D1%81%D1%82%D1%80%D0%BE%D0%B9%D0%BA%D0%B8-2%20%D0%B7%D0%B0%D0%B4%D0%B0%D0%BD%D0%B8%D0%B5%201.png)

Результат сборки:

![3](https://github.com/Skotix0/homeworks-netology/blob/main/CICD-homework/screenshots/%D0%A0%D0%B5%D0%B7%D1%83%D0%BB%D1%8C%D1%82%D0%B0%D1%82%20%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D1%8B%20%D0%B7%D0%B0%D0%B4%D0%B0%D0%BD%D0%B8%D0%B5%201.png)
---

### Задание 2

**Что нужно сделать:**

1. Создайте новый проект pipeline.
2. Перепишите сборку из задания 1 на declarative в виде кода.

*В качестве ответа пришлите скриншоты с настройками проекта и результатами выполнения сборки.*

#### Ответ на задание 2:

Тело скрипта:

```groovy
pipeline {
    agent any

    stages {
        stage('Git') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Skotix0/fork-for-cicd-homework.git'
            }
        }

        stage('Test') {
            steps {
                sh '''
                    set -eu
                    go version
                    go test .
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                    set -eu
                    docker build -t "hello-world-pipeline:v${BUILD_NUMBER}" .
                '''
            }
        }
    }

    post {
        success {
            echo 'Тесты и сборка Docker-образа завершены успешно.'
        }
        failure {
            echo 'Сборка завершилась ошибкой. Проверь Console Output.'
        }
    }
}
```
Результат сборки:

![Результат](https://github.com/Skotix0/homeworks-netology/blob/main/CICD-homework/screenshots/%D0%A0%D0%B5%D0%B7%D1%83%D0%BB%D1%8C%D1%82%D0%B0%D1%82%20%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D1%8B%20%D0%B7%D0%B0%D0%B4%D0%B0%D0%BD%D0%B8%D0%B5%202.png)
---

### Задание 3

**Что нужно сделать:**

1. Установите на машину Nexus.
2. Создайте raw-hosted репозиторий.
3. Измените pipeline так, чтобы вместо Docker-образа собирался бинарный go-файл. Команду можно скопировать из Dockerfile.
4. Загрузите файл в репозиторий с помощью jenkins.

*В качестве ответа пришлите скриншоты с настройками проекта и результатами выполнения сборки.*

#### Ответ на задание 3:

Настройки репозитория в нексусе:

![1](https://github.com/Skotix0/homeworks-netology/blob/main/CICD-homework/screenshots/%D0%9D%D0%B0%D1%81%D1%82%D1%80%D0%BE%D0%B9%D0%BA%D0%B8%20%D1%80%D0%B5%D0%BF%D0%BE%D0%B7%D0%B8%D1%82%D0%BE%D1%80%D0%B8%D1%8F%20%D0%B2%20%D0%BD%D0%B5%D0%BA%D1%81%D1%83%D1%81%D0%B5%20%D0%B7%D0%B0%D0%B4%D0%B0%D0%BD%D0%B8%D0%B5%203.png)

Тело скрипта в дженкинсе:

```groovy
pipeline {
    agent any

    environment {
        NEXUS_URL = 'http://localhost:8081'
        NEXUS_REPOSITORY = 'go-binaries'
    }

    stages {
        stage('Git') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Skotix0/fork-for-cicd-homework.git'
            }
        }

        stage('Test') {
            steps {
                sh '''
                    set -eu

                    echo "=== Версия Go ==="
                    go version

                    echo "=== Запуск тестов ==="
                    go test .
                '''
            }
        }

        stage('Build binary') {
            steps {
                sh '''
                    set -eu

                    mkdir -p dist

                    echo "=== Сборка бинарника по команде из Dockerfile ==="
                    CGO_ENABLED=0 GOOS=linux go build \
                        -a \
                        -installsuffix nocgo \
                        -o dist/app \
                        .

                    echo "=== Проверка результата ==="
                    test -s dist/app
                    ls -lh dist/app
                    sha256sum dist/app
                '''
            }
        }

        stage('Upload to Nexus') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'nexus-credentials',
                        usernameVariable: 'NEXUS_USER',
                        passwordVariable: 'NEXUS_PASSWORD'
                    )
                ]) {
                    sh '''
                        set +x
                        set -eu

                        ARTIFACT_URL="${NEXUS_URL}/repository/${NEXUS_REPOSITORY}/hello-world/v${BUILD_NUMBER}/app"

                        echo "=== Загрузка бинарника в Nexus ==="

                        curl --fail --show-error --silent \
                            --user "$NEXUS_USER:$NEXUS_PASSWORD" \
                            --upload-file dist/app \
                            "$ARTIFACT_URL"

                        echo "Файл успешно загружен: $ARTIFACT_URL"
                    '''
                }
            }
        }
    }

    post {
        success {
            archiveArtifacts artifacts: 'dist/app',
                             fingerprint: true

            echo 'Бинарник собран и загружен в Nexus.'
        }

        failure {
            echo 'Сборка завершилась ошибкой. Проверь Console Output.'
        }
    }
}
```

Результат работы пайплайна:

![1](https://github.com/Skotix0/homeworks-netology/blob/main/CICD-homework/screenshots/%D0%A0%D0%B5%D0%B7%D1%83%D0%BB%D1%8C%D1%82%D0%B0%D1%82%20%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D1%8B%20%D0%B7%D0%B0%D0%B4%D0%B0%D0%BD%D0%B8%D0%B5%203.png)

Полученный файл в нексусе:

![1](https://github.com/Skotix0/homeworks-netology/blob/main/CICD-homework/screenshots/%D0%9F%D0%BE%D0%BB%D1%83%D1%87%D0%B5%D0%BD%D0%BD%D0%B0%D1%8F%20%D1%81%D0%B1%D0%BE%D1%80%D0%BA%D0%B0%20%D0%B2%20%D0%BD%D0%B5%D0%BA%D1%81%D1%83%D1%81%D0%B5%20%D0%B7%D0%B0%D0%B4%D0%B0%D0%BD%D0%B8%D0%B5%203.png)
---
### Задание 4*

Придумайте способ версионировать приложение, чтобы каждый следующий запуск сборки присваивал имени файла новую версию. Таким образом, в репозитории Nexus будет храниться история релизов.

Подсказка: используйте переменную BUILD_NUMBER.

*В качестве ответа пришлите скриншоты с настройками проекта и результатами выполнения сборки.*

#### Ответ на задание 4*:

