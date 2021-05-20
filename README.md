# IOS-Concurrency

# GCD
## Intro

```swift

// Class level variable

let queue = DispatchQueue(label: "ru.egorskikh.worker")

// Somewhere in your function

queue.async {

  // Call slow non-UI methods here

  DispatchQueue.main.async {

    // Update the UI here
    
  }  

}

```
## GCD предоставляет три основных типа очередей:

* Основная очередь: выполняется в основном потоке и представляет собой последовательную очередь.

* Глобальные очереди: параллельные очереди, общие для всей системы. Таких очередей четыре с разными приоритетами: высокий, по умолчанию, низкий и фоновый. Очередь с фоновым приоритетом имеет самый низкий приоритет и регулируется при любых операциях ввода-вывода, чтобы минимизировать негативное влияние на систему.

* Пользовательские очереди: создаваемые вами очереди, которые могут быть последовательными или параллельными. Запросы в этих очередях фактически попадают в одну из глобальных очередей.

### Краткое руководство о том, как и когда использовать различные очереди с async:

* Основная очередь : это распространенный выбор для обновления пользовательского интерфейса после завершения работы над задачей в параллельной очереди. Для этого вы кодируете одно закрытие внутри другого. Нацеливание на основную очередь и вызов asyncгарантирует, что эта новая задача будет выполнена через некоторое время после завершения текущего метода.

* Глобальная очередь: это распространенный выбор для выполнения работы, не связанной с пользовательским интерфейсом, в фоновом режиме.

* Пользовательская последовательная очередь: хороший выбор, если вы хотите выполнять фоновую работу последовательно и отслеживать ее. Это устраняет конкуренцию за ресурсы и состояния гонки, поскольку вы знаете, что одновременно выполняется только одна задача. Обратите внимание: если вам нужны данные из метода, вы должны объявить другое закрытие, чтобы получить его или рассмотреть возможность использования sync.

## Quality of service  (qos)

* .userInteractive

.UserInteractive QoS рекомендуется для задач, с которыми пользователь напрямую взаимодействует. Расчеты, анимация и все, что нужно для обновления пользовательского интерфейса, чтобы пользовательский интерфейс оставался быстрым и отзывчивым. Если работа не выполняется быстро, может показаться, что все замерло. Задачи, отправленные в эту очередь, должны выполняться практически мгновенно.

* .userInitiated

Очередь .userInitiated следует использовать, когда пользователь запускает задачу из пользовательского интерфейса, которая должна выполняться немедленно, но может выполняться асинхронно. Например, вам может потребоваться открыть документ или прочитать его из локальной базы данных. Если пользователь нажал кнопку, вероятно, это именно та очередь, которая вам нужна. Задачи, выполняемые в этой очереди, должны занять несколько секунд или меньше.

* .utility

Вы захотите использовать очередь отправки .utility для задач, которые обычно включают индикатор выполнения, таких как длительные вычисления, ввод-вывод, сеть или непрерывные потоки данных. Система пытается сбалансировать быстродействие и производительность с энергоэффективностью. Задачи в этой очереди могут занять от нескольких секунд до нескольких минут.

* .background

Для задач, о которых пользователь не знает напрямую, следует использовать очередь .background. Они не требуют взаимодействия с пользователем и не зависят от времени. Предварительная выборка, обслуживание базы данных, синхронизация удаленных серверов и выполнение резервного копирования - все это отличные примеры. ОС будет сосредоточена на энергоэффективности, а не на скорости. Вы захотите использовать эту очередь для работы, которая займет значительное время, порядка минут или больше.

* .default и .unspecified

Существуют еще два возможных варианта, но вы не должны использовать их явно. Существует параметр .default, который находится между 
.userInitiated и .utility и является значением по умолчанию для аргумента qos. Он не предназначен для непосредственного использования. Второй вариант - .unspecified и существует для поддержки устаревших API-интерфейсов, которые могут отключать поток от качества обслуживания. Хорошо знать, что они существуют, но если вы их используете, вы почти наверняка делаете что-то не так.

```swift

    DispatchQueue.global(qos: .utility).async { [weak self] in
      // Update the UI here
    }
    
```