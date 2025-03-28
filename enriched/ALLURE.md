#go #testing #allure #reports #integration #xunit #customization #testframeworks #community

# Интеграция Allure Test Report с Go

```table-of-contents
```

Задача интеграции Allure Test Report с проектами на Go, на первый взгляд, может показаться нетривиальной, поскольку Allure изначально ориентирован на Java и JVM-совместимые языки. Тем не менее, существует несколько подходов, позволяющих добиться желаемого результата, используя гибкость Allure и возможности Go. Рассмотрим каждый из них подробно.

## Использование Allure Command Line Tool

Этот подход является наиболее прямолинейным и не требует написания дополнительного кода на Go. Его суть заключается в следующем:

1.  **Формат вывода результатов тестирования:** Go предоставляет встроенный пакет `testing`, который позволяет запускать тесты и получать результаты. Однако стандартный вывод `testing` несовместим с Allure. Для решения этой проблемы необходимо преобразовать результаты тестов в формат, понятный Allure. Одним из наиболее распространенных форматов, поддерживаемых Allure, является xUnit.

2.  **Преобразование в xUnit:** Существует несколько сторонних библиотек и утилит, позволяющих конвертировать вывод `go test` в xUnit. Например, можно использовать `go-junit-report` ([https://github.com/jstemmer/go-junit-report](https://github.com/jstemmer/go-junit-report)).  Эта утилита читает стандартный вывод `go test` и генерирует XML-файл в формате JUnit, который совместим с xUnit и, следовательно, с Allure.

    Пример использования `go-junit-report`:

    ```bash
    go test -v ./... | go-junit-report > report.xml
    ```
    Эта команда запустит тесты во всех подпакетах текущего проекта (`./...`), направит вывод в `go-junit-report`, который, в свою очередь, создаст файл `report.xml` в формате JUnit.

3.  **Генерация отчета Allure:** После получения XML-файла с результатами тестов в формате xUnit, можно использовать Allure Command Line Tool для генерации HTML-отчета.

    ```bash
    allure generate report.xml -o allure-report
    ```
    Эта команда обработает `report.xml` и создаст директорию `allure-report`, содержащую HTML-отчет Allure.

    Для открытия отчёта:
    ```bash
    allure open allure-report
    ```

**Преимущества:**

*   Простота реализации: не требует изменения кода тестов.
*   Независимость от тестового фреймворка: работает с любым фреймворком, который может выводить результаты в формате, совместимом с `go test`.

**Недостатки:**

*   Ограниченная информация в отчете: в отчете будет представлена только базовая информация о тестах (имя, статус, время выполнения). Дополнительные метаданные (например, шаги теста, вложения) не будут отображаться.
*   Необходимость использования сторонних утилит.

## Написание обертки или плагина

Этот подход предоставляет максимальную гибкость и позволяет интегрировать Allure с любым тестовым фреймворком на Go, а также добавлять в отчет любую необходимую информацию. Однако он требует значительных усилий по разработке.

1.  **Понимание внутреннего устройства Allure:**  Для создания обертки необходимо понимать, как Allure собирает данные о тестах. Allure использует концепцию [[Test Result Container]] и [[Test Case Result]].
    *   **Test Result Container:**  Представляет собой контейнер для группировки результатов тестов.  Обычно соответствует тестовому классу или модулю.
    *   **Test Case Result:**  Представляет собой результат выполнения одного конкретного теста.  Содержит информацию о статусе теста, шагах, вложениях и т.д.

2.  **Создание обертки:** Обертка должна перехватывать события, происходящие во время выполнения тестов (начало теста, завершение теста, выполнение шага, добавление вложения и т.д.), и формировать соответствующие [[Test Result Container]] и [[Test Case Result]] в формате JSON. Эти JSON-файлы затем будут использоваться Allure Command Line Tool для генерации отчета.

3. **Взаимодействие с тестовым фреймворком:** Обертка должна интегрироваться с используемым тестовым фреймворком. Это может быть стандартный пакет `testing` или любой другой сторонний фреймворк (например, `testify`, `ginkgo`, `godog`). Способ интеграции будет зависеть от конкретного фреймворка.  Например, для `testing` можно использовать функции `T.Run` для создания вложенных тестов, соответствующих шагам Allure, и `T.Log` для добавления информации, которая будет отображаться в отчете.

4. **Генерация JSON-файлов:** Обертка должна генерировать JSON-файлы в формате, ожидаемом Allure. Структура этих файлов описана в документации Allure.

**Пример структуры JSON для Test Case Result (упрощенный):**

```json
{
  "uuid": "уникальный идентификатор теста",
  "name": "имя теста",
  "status": "passed", // или "failed", "broken", "skipped"
  "start": 1678886400000, // timestamp начала теста
  "stop": 1678886401000, // timestamp завершения теста
  "steps": [
    {
      "name": "шаг 1",
      "status": "passed",
      "start": 1678886400100,
      "stop": 1678886400500
    },
    {
        "name": "шаг 2",
        "status": "failed",
        "start": 1678886400600,
        "stop": 1678886400900
    }

  ],
  "attachments": [
    {
      "name": "лог",
      "source": "лог.txt",
      "type": "text/plain"
    }
  ]
}
```

5. **Использование Allure Command Line Tool:** После того как обертка сгенерирует JSON-файлы, можно использовать Allure Command Line Tool для генерации HTML-отчета, как и в предыдущем подходе.

**Преимущества:**

*   Полный контроль над информацией в отчете: можно добавлять любые метаданные (шаги, вложения, параметры, ссылки и т.д.).
*   Поддержка любого тестового фреймворка.

**Недостатки:**

*   Сложность реализации: требует глубокого понимания Allure и выбранного тестового фреймворка.
*   Необходимость поддержки обертки.

## Использование существующих инструментов (с осторожностью)

Этот подход предполагает поиск готовых решений, которые могут упростить интеграцию Allure с Go. На момент написания этого текста, *официальной* поддержки Allure для Go нет. Однако, сообщество Go активно развивается, и сторонние библиотеки и инструменты могут появляться.

**Где искать:**

*   **GitHub:** Поиск по ключевым словам "allure", "go", "testing".
*   **Awesome Go:**  ([https://awesome-go.com/](https://awesome-go.com/)) - curated list of awesome Go frameworks, libraries and software. Просмотрите раздел, посвященный тестированию.
*   **Форумы и сообщества:**  Задайте вопрос на Stack Overflow, Reddit (r/golang) или в других сообществах, посвященных Go и тестированию.

**Важно:** При использовании сторонних инструментов, особенно неофициальных, необходимо тщательно оценивать их надежность, безопасность и поддерживаемость.

**Преимущества:**

*   Потенциально более простое решение, чем написание собственной обертки.

**Недостатки:**

*   Зависимость от сторонних инструментов, которые могут быть некачественными или неподдерживаемыми.
*   Ограниченная гибкость: возможности инструмента могут не соответствовать вашим требованиям.

## Пример интеграции с `testing` и `go-junit-report` (подробно)

Рассмотрим более подробно первый подход, как наиболее доступный.

**1. Установка `go-junit-report`:**

```bash
go install github.com/jstemmer/go-junit-report@latest
```

**2. Написание тестов (пример):**

```go
package mypackage

import (
	"testing"
)

func TestAddition(t *testing.T) {
	result := 1 + 1
	if result != 2 {
		t.Errorf("Expected 2, got %d", result)
	}
}

func TestSubtraction(t *testing.T) {
    t.Run("Positive numbers", func(t *testing.T) {
        result := 5 - 3
        if result != 2 {
            t.Errorf("Expected 2, got %d", result)
        }
    })

    t.Run("Negative numbers", func(t *testing.T) {
        result := 3 - 5
        if result != -2 {
            t.Errorf("Expected -2, got %d", result)
        }
    })
}
```

**3. Запуск тестов и генерация xUnit отчета:**

```bash
go test -v ./... | go-junit-report > report.xml
```

**4. Установка Allure Command Line Tool:**

Инструкции по установке Allure Command Line Tool можно найти на официальном сайте Allure: [https://docs.qameta.io/allure/](https://docs.qameta.io/allure/).  Способ установки зависит от вашей операционной системы.

**5. Генерация Allure отчета:**

```bash
allure generate report.xml -o allure-report
```

**6. Открытие Allure отчета:**

```bash
allure open allure-report
```

Эта команда откроет сгенерированный отчет в вашем браузере.

## Заключение

Выбор подхода для интеграции Allure с Go зависит от ваших потребностей и ресурсов.  Если вам нужна простая интеграция и достаточно базовой информации о тестах, используйте `go-junit-report` и Allure Command Line Tool.  Если вам нужен полный контроль над информацией в отчете и вы готовы к разработке, создайте собственную обертку.  И, наконец, следите за развитием сообщества Go – возможно, появятся готовые решения, упрощающие интеграцию.

```old
На момент последнего обновления моих данных, Allure Test Report, широко используемый инструмент для создания наглядных отчетов о тестировании, не имеет официальной интеграции с языком программирования Go (Golang) напрямую. Allure первоначально разрабатывался для работы с Java и другими JVM-совместимыми языками, но теперь поддерживает и другие языки и тестовые фреймворки через различные плагины и обертки.

Для Go, поскольку официальной поддержки нет, есть несколько подходов, которые можно использовать для интеграции Allure с вашими тестами на Go:

1. **Использование Allure Command Line Tool**: Вы можете генерировать отчеты Allure, используя командную строку Allure, если ваши тесты на Go могут генерировать результаты в формате, который Allure может прочитать (например, в формате xUnit). Выполните тесты, экспортируйте результаты в поддерживаемый Allure формат, а затем используйте командную строку Allure для генерации отчета.

2. **Написание обертки или плагина**: Если у вас есть время и ресурсы, вы можете создать обертку вокруг вашего тестового фреймворка на Go, которая будет собирать данные о прохождении тестов и формировать из них файлы в формате, который Allure сможет прочитать. Это может быть достаточно сложная задача, требующая глубокого понимания как Allure, так и фреймворка для тестирования на Go, который вы используете.

3. **Использование существующих инструментов с поддержкой Allure**: Некоторые инструменты и библиотеки, возможно, уже предоставляют возможность экспорта данных в формат, совместимый с Allure, или могут быть настроены для этого. Примером такого инструмента для других языков может служить TestNG для Java или pytest для Python. Для Go такие инструменты могут появиться в будущем, поэтому стоит следить за обновлениями в сообществе.

Поскольку ситуация с инструментами и библиотеками меняется, рекомендуется проверять наличие новых инструментов и поддержки со стороны Allure и Go-сообщества. На форумах и в репозиториях, связанных с Go и тестированием, также можно искать новые разработки и решения от сообщества.
```