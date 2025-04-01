# oil-spill-devops

## 📌 Git Workflow
Каждый сервис имеет свои ветки разработки:
- main → стабильная версия, ничего не пушим напрямую
- develop → текущая разработка
- feature/* → для новых фич

### Как вносить изменения?
1. Перейти в подмодуль:
```cd frontend```
2. Переключиться на develop и подтянуть изменения:
```
git checkout develop
git pull origin develop
```
3. Создать новую ветку:
```git checkout -b feature/upload-files```
4. Сделать коммиты и пушнуть:
```
# Работаешь, коммитишь
git add .
git commit -m "Добавлена форма загрузки файлов"
git push origin feature/upload-files
```
4. Сделать Pull Request (PR) в ```develop``` на GitHub.


## Как запустить весь проект?
В папке oil-spill-devops запустить:

```docker-compose up --build```

## Как работать с кодом локально?
Каждый разработчик клонирует репозиторий и обновляет подмодули:

```git clone --recurse-submodules https://github.com/your-org/oil-spill-devops.git```

```cd oil-spill-devops```

```git submodule update --init --recursive```

Как обновлять подмодули после изменений?

```git submodule update --remote --merge```

## Повседневная работа
### Изменения в основном репозитории
Например, добавили новый файл в oil-spill-devops/README.md:

```git add README.md```

```git commit -m "Обновил документацию"```

```git push origin main```

### Изменения в подмодуле (например, frontend)

```cd frontend```

Сделайте изменения, закоммитьте и запушите в репозиторий подмодуля:

```git add .```

```git commit -m "Новый фич: кнопка"```

```git push origin main``` (любая другая ветка, подмодули работают независимо)

Вернитесь в основной репозиторий:

```cd ..```

Обновите ссылку на подмодуль:

```git add frontend```

```git commit -m "Обновил подмодуль frontend до последней версии"```

```git push origin main```

### Как обновлять подмодули
Если кто-то обновил подмодуль:

```
# В основном репозитории
git pull  # получить изменения основного репозитория
git submodule update --remote  # обновить подмодули до версий, указанных в основном репозитории
```
Еще есть вариант ```git submodule update --remote --merge```. В каких сценариях что использовать так и не понял..
