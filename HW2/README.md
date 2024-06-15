# ЛР 1. Dockerfile

1) bad_docker
2) good_docker

## Плохие практики
### Использование latest тега.
```go
FROM python:latest
```

Почему это плохо: Тег latest может указывать на разные версии образа в разное время, что делает сборку непредсказуемой и может привести к неожиданным ошибкам.   
Исправление: Использование конкретного тега версии образа.   
```go
FROM python:3.9-slim
```

### Выполнение нескольких команд RUN в отдельных слоях
Плохая практика: Разделение команд RUN, что приводит к большому количеству слоев.
```go
RUN apt-get update
RUN apt-get install -y python3-pip
RUN pip install jupyter
```
Почему это плохо: Каждый RUN создает новый слой, увеличивая размер образа и количество слоев, что усложняет кэширование.   
Исправление: Объединение команд RUN в один слой.
```go
RUN apt-get update && \
    apt-get install -y python3-pip vim && \
    pip install jupyter && \
    apt-get clean && rm -rf /var/lib/apt/lists/*
```


