Если не работает appveyor

Если по каким-то причинам у вас не работает appveyor. Например, ошибка на самом сервисе: 

`There was an error while trying to complete the current operation. Please contact AppVeyor support`

Проблему к сожалению не получится решить оперативно. 
Можно создать Issues  с проблемой на по ссылке выше. 

Вы можете настроить все на githubActions. 
Инструкция по настройке:
1. Открываете ваш проект инициализируете новый репозиторий локально, делаете все настройки, создаете .gitignore  коммитите и отправляете на  github
2. Создаете Actions на github с именем gradle.yml
3. Помещаете туда код (Это пример кода он может меняться в зависимости от проекта)

name: Java CI with Gradle

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

permissions:
  contents: read

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3
    - name: Set up JDK 11
      uses: actions/setup-java@v3
      with:
        java-version: '11'
        distribution: 'temurin'
    - name: Build with Gradle
      uses: gradle/gradle-build-action@67421db6bd0bf253fb4bd25b31ebb98943c375e1
      with:
        java-version: '11'
        distribution: 'adopt'
    - name: Run SUT
      run:  java -jar ./artifacts/app-mbank.jar & sleep 10
    - name: Grant execute permission for gradlew
      run: chmod +x gradlew
    - name: Build witch Gradle
      run: ./gradlew test --info -Dselenide.headless=true
3. Сохраняем изменения
4. Делаем git pull (поскольку вы внесли изменения в удаленный репозиторий вам нужно забрать изменения в локальной репозиторий во избежание конфликтов)
5. Создаем бэдж

   5.1 Создаем файл README.md в корне проекта 
   
   5.2 Помещаем туда этот код
  
[![Java CI with Gradle](https://github.com/<вашае имя на гитхаб>/<название репозитория>/actions/workflows/gradle.yml/badge.svg)](https://github.com/<вашае имя на гитхаб>/<название репозитория>/actions/workflows/gradle.yml)
Где вместо <....> нужно вписать ваши данные 
если у адрес на ваш репозиторий 
https://github.com/IvanIvanov123/myRepo 
 вместо <вашае имя на гитхаб>/<название репозитория>
нужно вставить IvanIvanov123/myRepo
