# Hello Go — CI/CD и GitHub Releases

[![Go CI/CD](https://github.com/Focsi4ek/hello-go-bin/actions/workflows/ci.yml/badge.svg)](https://github.com/Focsi4ek/hello-go-bin/actions/workflows/ci.yml)
[Релиз v1.0.0](https://github.com/Focsi4ek/hello-go-bin/releases/tag/v1.0.0)

Учебный проект на Go с автоматическими проверками, матричной сборкой
и публикацией бинарников для Linux, macOS и Windows.

Локальные проверки выполняются через Docker. Готовые программы
распространяются через GitHub Releases.

## Что делает приложение

- Выводит версию, операционную систему и архитектуру.
- Показывает приветствие.
- Вычисляет сумму чисел от 1 до 10.
- Выводит переданные аргументы командной строки.

## Файлы проекта

| Файл | Назначение |
| --- | --- |
| `main.go` | Точка входа и вывод информации |
| `greeting/greeting.go` | Функции приветствия и вычисления суммы |
| `greeting/greeting_test.go` | Тесты функций |
| `go.mod` | Описание модуля и минимальная версия Go |
| `.github/workflows/ci.yml` | Автоматические проверки, сборки и публикация |
| `.gitignore` | Исключение бинарников и локальных файлов из Git |

## Локальная проверка через Docker

Нужен установленный и запущенный Docker Desktop.
Выполните из корневой папки проекта:

```bash
docker run --rm \
  -u "$(id -u):$(id -g)" \
  -e HOME=/tmp \
  -e GOPATH=/tmp/go \
  -e GOCACHE=/tmp/go-cache \
  -e CGO_ENABLED=0 \
  -v "$PWD":/app \
  -w /app \
  golang:1.23-alpine \
  sh -c 'gofmt -w . && go vet ./... && go test ./... -v && go run . Привет'
```

Команда форматирует исходники, выполняет статические проверки,
запускает тесты и приложение. При успешном выполнении тестов
появляется `PASS`.

При запуске исходников версия приложения — `dev`.
Операционная система в этом случае — `linux`, поскольку программа
выполняется внутри контейнера.

## Как работает CI/CD

| Событие | Действия |
| --- | --- |
| Push в `main` или Pull Request | Проверка форматирования, `go vet`, тесты и пробная сборка |
| Push тега `v*` | Проверки, пять параллельных сборок и публикация GitHub Release |

Workflow состоит из трёх заданий:

1. **test** — проверяет код.
2. **build** — собирает бинарники под пять платформ и сохраняет artifacts.
3. **release** — получает artifacts и публикует все бинарники в одном релизе.

Версия внедряется во время сборки через:

```text
-ldflags="-s -w -X main.version=$VERSION"
```

В опубликованном бинарнике версия соответствует тегу Git,
например `v1.0.0`.

## Доступные бинарники

| Система | Архитектура | Файл |
| --- | --- | --- |
| Linux | AMD64 / x86-64 | `hello-go-linux-amd64` |
| Linux | ARM64 | `hello-go-linux-arm64` |
| macOS, Intel | AMD64 / x86-64 | `hello-go-darwin-amd64` |
| macOS, Apple Silicon | ARM64 | `hello-go-darwin-arm64` |
| Windows | AMD64 / x86-64 | `hello-go-windows-amd64.exe` |

Скачать файлы можно в разделе [Releases](https://github.com/Focsi4ek/hello-go-bin/releases).

## Запуск релиза на macOS с Apple Silicon

```bash
curl -fL \
  -o hello-go-darwin-arm64 \
  https://github.com/Focsi4ek/hello-go-bin/releases/download/v1.0.0/hello-go-darwin-arm64

chmod +x hello-go-darwin-arm64
./hello-go-darwin-arm64
```

Пример вывода:

```text
hello-go version v1.0.0
Hello from Go! 🐹
OS: darwin
Arch: arm64
Hello, GitHub!
Sum 1..10 = 55
```

Для запуска готового бинарника установка Go и Docker не требуется.

## Создание следующего релиза

После коммита и отправки изменений в `main` создайте новый тег:

```bash
git tag v1.1.0
git push origin v1.1.0
```

GitHub Actions выполнит проверки, соберёт пять бинарников
и опубликует новый релиз.

## Используемые инструменты

Go 1.23 · Docker · Git · GitHub Actions · GitHub Releases
