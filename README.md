# Spring Cloud Config для микросервисов E-Commerce

В этом репозитории сосредоточены внешние настройки (конфигурация) для микросервисов E-Commerce с использованием Spring Cloud Config. Конфигурация хранится в виде YAML-файлов для каждого сервиса и позволяет изменять параметры без пересборки и деплоя приложений.

## Структура репозитория

```
├── account.yml        # Настройки сервиса Account
├── eurekaserver.yml   # Настройки сервера Eureka Service Discovery
├── gateway.yml        # Настройки API Gateway
└── .idea/             # Настройки IDE (можно игнорировать)
```

- **account.yml**  
  Параметры для микросервиса `account`: URL базы данных, учётные данные, флаги фич и другие свойства, специфичные для этого сервиса.

- **eurekaserver.yml**  
  Конфигурация Eureka-сервера:
  ```yaml
  server:
    port: 8070
  eureka:
    instance:
      hostname: localhost
    client:
      registerWithEureka: false
      fetchRegistry: false
      serviceUrl:
        defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
  ```

- **gateway.yml**  
  Настройки API Gateway и подключение к Eureka:
  ```yaml
  server:
    port: 8072
  eureka:
    instance:
      preferIpAddress: true
    client:
      registerWithEureka: true
      fetchRegistry: true
      serviceUrl:
        defaultZone: "http://localhost:8070/eureka/"
  ```

## Быстрый старт

1. **Клонируйте репозиторий**
   ```bash
   git clone https://github.com/Yersdev/e-commerce-config.git
   cd e-commerce-config
   ```

2. **Настройте Spring Cloud Config Server**
   В приложении `configserver` (см. [e-commerce/configserver](https://github.com/Yersdev/e-commerce/tree/main/configserver)) укажите путь к этому репозиторию:
   ```yaml
   spring:
     application:
       name: configserver
     cloud:
       config:
         server:
           git:
             uri: file://${PWD}
             search-paths: '.'
   ```

3. **Включите Config Server**
   ```java
   @EnableConfigServer
   @SpringBootApplication
   public class ConfigServerApplication { ... }
   ```

4. **Запустите Config Server**
   ```bash
   mvn spring-boot:run
   ```
   По умолчанию сервер слушает порт `8888`.

5. **Получение конфигурации сервисов**

    - **Account Service**:
      ```http
      GET http://localhost:8888/account/default
      ```

    - **Eureka Server**:
      ```http
      GET http://localhost:8888/eurekaserver/default
      ```

    - **API Gateway**:
      ```http
      GET http://localhost:8888/gateway/default
      ```

## Профили и окружения

- Чтобы добавить настройки для разных окружений, создайте файлы вида `{application}-{profile}.yml`, например:
  ```
  account-dev.yml
  account-prod.yml
  ```
- В таких файлах переопределяйте только отличающиеся свойства (порт, URL БД, ключи и т. п.).

## Участие в проекте

1. Форкните репозиторий.
2. Создайте новую ветку (`git checkout -b feature/xyz`).
3. Внесите изменения (`git commit -am 'Добавил фичу xyz'`).
4. Запушьте на GitHub (`git push origin feature/xyz`).
5. Откройте Pull Request.

## Лицензия

Проект распространяется под лицензией MIT. Подробнее — в файле [LICENSE](LICENSE).

