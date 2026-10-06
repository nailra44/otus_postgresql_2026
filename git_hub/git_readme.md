```sh
# инициализируем репозиторий
git init
#
git config --global user.name
#nailra
git config --global user.email
#nailra44@yandex.ru
# Добавление файлов в репозиторий

git add -A        # ВСЁ: новые + изменённые + удалённые, по всему репо
git add .         # то же, но ограничено текущей папкой (в старых версиях Git)
git add -u        # только изменённые + удалённые (без новых файлов)
git add *         # оболочка сама раскрывает *, не видит удаления и часть файлов

git status
# commit
git commit -m 'commit_1'

# переименовать branch в main
git branch -M main
#проверка
git branch
# привязка удалённого репозотория
git remote add origin https://github.com/nailra44/otus_postgresql_2026.git
# проверка
git push -u origin main