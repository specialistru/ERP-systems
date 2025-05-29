# 🧩 Подборка примеров паттернов проектирования на C++

---

## 1. Singleton

**Цель:** Обеспечить существование только одного экземпляра класса (например, менеджер конфигурации или подключения к БД).

```cpp
class Configuration {
private:
    Configuration() { /* Загрузка настроек */ }
    Configuration(const Configuration&) = delete;
    Configuration& operator=(const Configuration&) = delete;

public:
    static Configuration& getInstance() {
        static Configuration instance;
        return instance;
    }

    void showConfig() {
        std::cout << "Параметры конфигурации..." << std::endl;
    }
};
```

---

## 2. Factory Method

**Цель:** Создавать объекты без указания точного класса.

```cpp
class User {
public:
    virtual void role() = 0;
    virtual ~User() = default;
};

class Admin : public User {
public:
    void role() override { std::cout << "Я админ\n"; }
};

class Manager : public User {
public:
    void role() override { std::cout << "Я менеджер\n"; }
};

class UserFactory {
public:
    static std::unique_ptr<User> createUser(const std::string& type) {
        if (type == "admin") return std::make_unique<Admin>();
        if (type == "manager") return std::make_unique<Manager>();
        return nullptr;
    }
};
```

---

## 3. Observer

**Цель:** Позволяет объектам подписываться на события другого объекта (например, обновление UI при изменении данных).

```cpp
#include <vector>
#include <algorithm>

class Observer {
public:
    virtual void update() = 0;
    virtual ~Observer() = default;
};

class Subject {
    std::vector<Observer*> observers;
public:
    void attach(Observer* obs) { observers.push_back(obs); }
    void detach(Observer* obs) {
        observers.erase(std::remove(observers.begin(), observers.end(), obs), observers.end());
    }
    void notify() {
        for (auto obs : observers) obs->update();
    }
};

class DataModel : public Subject {
    int data = 0;
public:
    void setData(int val) {
        data = val;
        notify();
    }
    int getData() const { return data; }
};

class UI : public Observer {
    DataModel& model;
public:
    UI(DataModel& m) : model(m) {}
    void update() override {
        std::cout << "UI обновлен, новое значение: " << model.getData() << std::endl;
    }
};
```

---

## 4. Strategy

**Цель:** Выбор алгоритма во время выполнения (например, разные способы расчёта скидок).

```cpp
class DiscountStrategy {
public:
    virtual double applyDiscount(double price) = 0;
    virtual ~DiscountStrategy() = default;
};

class NoDiscount : public DiscountStrategy {
public:
    double applyDiscount(double price) override { return price; }
};

class SeasonalDiscount : public DiscountStrategy {
public:
    double applyDiscount(double price) override { return price * 0.9; }
};

class Context {
    DiscountStrategy* strategy;
public:
    Context(DiscountStrategy* strat) : strategy(strat) {}
    void setStrategy(DiscountStrategy* strat) { strategy = strat; }
    double execute(double price) { return strategy->applyDiscount(price); }
};
```

---

## 5. Decorator

**Цель:** Динамическое расширение функционала (например, логирование вызовов).

```cpp
class IDataSource {
public:
    virtual void writeData(const std::string& data) = 0;
    virtual ~IDataSource() = default;
};

class FileDataSource : public IDataSource {
public:
    void writeData(const std::string& data) override {
        std::cout << "Запись в файл: " << data << std::endl;
    }
};

class DataSourceDecorator : public IDataSource {
protected:
    IDataSource* wrappee;
public:
    DataSourceDecorator(IDataSource* source) : wrappee(source) {}
    void writeData(const std::string& data) override {
        wrappee->writeData(data);
    }
};

class LoggingDecorator : public DataSourceDecorator {
public:
    LoggingDecorator(IDataSource* source) : DataSourceDecorator(source) {}
    void writeData(const std::string& data) override {
        std::cout << "[Лог] Данные: " << data << std::endl;
        DataSourceDecorator::writeData(data);
    }
};
```

---

## 6. State

**Цель:** Изменять поведение объекта в зависимости от его состояния (например, состояние заказа).

```cpp
class Order;

class State {
public:
    virtual void handle(Order* order) = 0;
    virtual ~State() = default;
};

class NewOrderState : public State {
public:
    void handle(Order* order) override;
};

class ProcessingState : public State {
public:
    void handle(Order* order) override;
};

class Order {
    State* state;
public:
    Order(State* initState) : state(initState) {}
    void setState(State* newState) { state = newState; }
    void process() { state->handle(this); }
};

void NewOrderState::handle(Order* order) {
    std::cout << "Заказ новый — начинаем обработку.\n";
    order->setState(new ProcessingState());
}

void ProcessingState::handle(Order* order) {
    std::cout << "Заказ в обработке.\n";
}
```
