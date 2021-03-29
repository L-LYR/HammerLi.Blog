---
Author: HammerLi
Date: 2021.3.25
Tag: [Multithread][Concurrency Model]
---

# 线程与锁模型

该模型主要是在多线程模型中使用锁来作为同步机制，有效解决竞态条件、保证内存可见性、预防死锁。

## Tips

- 共享变量的访问同步化
- 读写线程的同步化
- 顺序获取多把锁
- 持有锁时，避免访问“外星方法”，或调用前进行保护性拷贝
- 尽可能降低持有锁的时长
- 可重入锁是一种糟糕的设计
- 线程池比无脑开线程好得多

## 竞态条件

考虑一个`Counter`类，单纯使用后缀自增运算符对某一共享变量进行修改，因为后缀自增运算是经典的**读-改-写**操作，所以可在多线程环境中，这里必然出现竞态条件。

### 使用互斥锁来消除竞态

在该`Counter`类中加入互斥锁来实现多线程间的同步。

```c++
class Counter {
    volatile int m_cnt;
    std::mutex m_mut;
  public:
    Counter() : m_cnt(0) {}
    auto inc() -> void {
        m_mut.lock();
        m_cnt++;
        m_mut.unlock();
    }
    auto get_cnt() -> int { return m_cnt; }
};

auto main(void) -> int {
    Counter counter;
    
    auto run = [&]() {
        for (int i = 0; i < 100000; ++i) {
            counter.inc();
        }
    };
    
    auto t1 = std::thread(run);
    auto t2 = std::thread(run);
    
    t1.join();
    t2.join();
    
    std::cout << counter.get_cnt() << "\n";
    return 0;
}
```

### 使用原子变量来消除竞态

如果将上述的后缀自增运算操作转变为原子操作，自然就不需要再使用锁来进行线程间同步。因此，可以将`m_cnt`声明为原子变量，这样某个线程在对其进行修改时，不能被另外的线程打断。

```c++
class Counter {
    std::atomic_int m_cnt;

  public:
    Counter() : m_cnt(0) {}
    auto inc() -> void { m_cnt++; }
    auto get_cnt() -> int { return m_cnt; }
};
```

## 死锁

### 经典哲学家进餐死锁场景复现

```c++
class Chopstick {
    int m_i;
    std::mutex m_mut;

  public:
    explicit Chopstick(int i = 0) : m_i(i) {}
    auto setIdx(int i) -> void { m_i = i; }

    friend auto operator<<(std::ostream& out, const Chopstick& c) -> std::ostream& {
        out << "chopstick " << c.m_i;
        return out;
    }

    auto take_up() -> void { m_mut.lock(); }
    auto put_down() -> void { m_mut.unlock(); }
};

static std::mutex output_mutex; // global output mutex

class Idiot {
    int m_i;

    using guard = std::lock_guard<std::mutex>;

    auto m_take_up_chopstick(Chopstick& c) -> void {
        c.take_up();
        {
            guard output_lock(output_mutex);
            std::cout << "Idiot " << m_i << " took up " << c << std::endl;
        }
    }

    auto m_think(int t) -> void {
        std::this_thread::sleep_for(std::chrono::milliseconds(t));
    }

    auto m_seize(Chopstick& lc, Chopstick& rc) -> void {
        while (true) {
            {
                guard output_lock(output_mutex);
                std::cout << "Idiot " << m_i << " began to seize chopsticks" << std::endl;
            }
            m_think(rand() % 1000); // 1
            m_take_up_chopstick(lc);
            m_think(rand() % 1000); // 2
            m_take_up_chopstick(rc);
            {
                guard output_lock(output_mutex);
                std::cout << "Idiot " << m_i << " began to have meal......" << std::endl;
            }
            m_think(rand() % 1000); // 3
            // These three sleep()s make the program meet deadlock much more easily. 
            {
                guard output_lock(output_mutex);
                std::cout << "Idiot " << m_i << " put down " << rc << std::endl;
                rc.put_down();

                std::cout << "Idiot " << m_i << " put down " << lc << std::endl;
                lc.put_down();
            }
        }
    }

  public:
    explicit Idiot(int i = 0) : m_i(i) {}
    auto setIdx(int i) -> void { m_i = i; }
    auto operator()(Chopstick& lc, Chopstick& rc) -> void { m_seize(lc, rc); }
};

auto main() -> int {
    srand(time(0));
    int num = 5;
    std::vector<Idiot> idiots(num);
    std::vector<Chopstick> chopsticks(num);
    std::vector<std::thread> threads;

    for (int i = 0; i < num; ++i) {
        idiots[i].setIdx(i);
        chopsticks[i].setIdx(i);
    }
    for (int i = 0; i < num; ++i) {
        auto tt = std::thread(idiots.at(i), std::ref(chopsticks.at(i)),
                              std::ref(chopsticks.at((i + 1) % num)));
        threads.push_back(std::move(tt));
    }

    std::for_each(threads.begin(), threads.end(), [](std::thread& t) { t.join(); });
    return 0;
}
```

### 使用含超时功能的互斥锁解决死锁

```c++
class Chopstick {
    int m_i;
    std::timed_mutex m_mut;
    std::chrono::seconds m_timeout;

  public:
    explicit Chopstick(int i = 0) : m_i(i), m_timeout(1) {}
    auto setIdx(int i) -> void { m_i = i; }

    friend auto operator<<(std::ostream& out, const Chopstick& c) -> std::ostream& {
        out << "chopstick " << c.m_i;
        return out;
    }

    auto take_up() -> bool { return m_mut.try_lock_for(m_timeout); }
    auto put_down() -> void { m_mut.unlock(); }
};

static std::mutex output_mutex;

class Idiot {
    int m_i;

    using guard = std::lock_guard<std::mutex>;

    auto m_take_up_chopstick(Chopstick& c) -> bool {
        if (c.take_up()) {
            guard output_lock(output_mutex);
            std::cout << "Idiot " << m_i << " took up " << c << std::endl;
            return true;
        }
        return false;
    }

    auto m_think(int t) -> void {
        std::this_thread::sleep_for(std::chrono::milliseconds(t));
    }

    auto m_seize(Chopstick& lc, Chopstick& rc) -> void {
        while (true) {
            {
                guard output_lock(output_mutex);
                std::cout << "Idiot " << m_i << " began to seize chopsticks" << std::endl;
            }
            m_think(rand() % 1000);
            if (m_take_up_chopstick(lc)) {
                m_think(rand() % 1000);
                if (m_take_up_chopstick(rc)) {
                    guard output_lock(output_mutex);
                    std::cout << "Idiot " << m_i << " began to have meal......" << std::endl;
                    m_think(rand() % 1000);
                    std::cout << "Idiot " << m_i << " put down " << rc << std::endl;
                    rc.put_down();
                }
                guard output_lock(output_mutex);
                std::cout << "Idiot " << m_i << " put down " << lc << std::endl;
                lc.put_down();
            }
        }
    }

  public:
    explicit Idiot(int i = 0) : m_i(i) {}
    auto setIdx(int i) -> void { m_i = i; }
    auto operator()(Chopstick& lc, Chopstick& rc) -> void { m_seize(lc, rc); }
};

auto main() -> int {
    srand(time(0));
    int num = 5;
    std::vector<Idiot> idiots(num);
    std::vector<Chopstick> chopsticks(num);
    std::vector<std::thread> threads;

    for (int i = 0; i < num; ++i) {
        idiots[i].setIdx(i);
        chopsticks[i].setIdx(i);
    }
    for (int i = 0; i < num; ++i) {
        auto tt = std::thread(idiots.at(i), std::ref(chopsticks.at(i)),
                              std::ref(chopsticks.at((i + 1) % num)));
        threads.push_back(std::move(tt));
    }

    std::for_each(threads.begin(), threads.end(), [](std::thread& t) { t.join(); });
    return 0;
}
```

### 使用条件变量解决死锁

```c++
static std::mutex table;

class Idiot {
    int m_i;
    bool m_eating;
    Idiot* m_left;
    Idiot* m_right;
    std::condition_variable m_condition;

    auto think() -> void {
        table.lock();
        m_eating = false;
        m_left->m_condition.notify_all();
        m_right->m_condition.notify_all();
        std::cout << m_i << " Idiot thinking..." << std::endl;
        table.unlock();
        rest();
    }

    auto eat() -> void {
        std::unique_lock<std::mutex> lk(table);
        m_condition.wait(lk, [this]() -> bool {
            return !this->m_left->m_eating && !this->m_right->m_eating;
        });
        std::cout << m_i << " Idiot eating..." << std::endl;
        m_eating = true;
        rest();
        lk.unlock();
    }

    auto rest() -> void {
        std::this_thread::sleep_for(std::chrono::milliseconds(rand() % 1000));
    }

  public:
    explicit Idiot() : m_i(0),
                       m_eating(false),
                       m_left(nullptr),
                       m_right(nullptr) {}

    auto setIdx(int i) -> void { m_i = i; }

    auto setNeightbours(Idiot* left, Idiot* right) -> void {
        m_left = left;
        m_right = right;
    }

    auto operator()() -> void {
        while (true) {
            think();
            eat();
        }
    }
};

auto main() -> int {
    srand(time(0));

    int num = 5;
    std::vector<Idiot> idiots(num);
    std::vector<std::thread> threads;

    for (int i = 0; i < num; ++i) {
        idiots[i].setIdx(i);
        idiots[i].setNeightbours(&idiots[(i - 1 + num) % num], &idiots[(i + 1) % num]);
    }

    for (int i = 0; i < num; ++i) {
        threads.push_back(std::thread([&idiots, i]() { idiots[i](); }));
    }

    std::for_each(threads.begin(), threads.end(), [](std::thread& t) { t.join(); });
    return 0;
}
```

## 使用交替锁实现并发支持链表

对有序链表进行并发插入与删除的同时，监控整个链表的内容。

```cpp
class ConcurrentSotedList {
    struct Node {
        int m_value;
        Node* m_prev;
        Node* m_next;
        std::mutex m_mut;

        Node() = default;
        Node(int val, Node* prev, Node* next) : m_value(val), m_prev(prev), m_next(next) {}
        ~Node() = default;

        auto lock() -> void { m_mut.lock(); }
        auto unlock() -> void { m_mut.unlock(); }
    };

  private:
    Node m_head_noop;
    Node m_tail_noop;
    Node* m_head;
    Node* m_tail;

    auto cleanup() -> void {
        Node* curp = m_head->m_next;
        Node* nxtp;
        while (curp != m_tail) {
            nxtp = curp->m_next;
            delete curp;
            curp = nxtp;
        }
        m_head->m_prev = m_head->m_next = m_tail;
        m_tail->m_prev = m_tail->m_next = m_head;
    }

  public:
    ConcurrentSotedList() {
        m_head = &m_head_noop;
        m_tail = &m_tail_noop;

        m_head_noop.m_value = INT_MAX;
        m_tail_noop.m_value = INT_MIN;

        m_head->m_prev = m_head->m_next = m_tail;
        m_tail->m_prev = m_tail->m_next = m_head;
    }
    ~ConcurrentSotedList() { cleanup(); }

    auto insert(int value) -> void {
        Node* curp = m_head;
        curp->lock();
        Node* nxtp = curp->m_next;
        while (true) {
            nxtp->lock();
            if (nxtp == m_tail || nxtp->m_value < value) {
                Node* new_node = new Node(value, curp, nxtp);
                nxtp->m_prev = new_node;
                curp->m_next = new_node;

                nxtp->unlock();
                curp->unlock();

                return;
            }
            curp->unlock();
            curp = nxtp;
            nxtp = curp->m_next;
        }
    }

    auto remove(int value) -> bool {
        Node* curp = m_head;
        curp->lock();
        Node* nxtp = curp->m_next;
        while (nxtp != m_tail) {
            nxtp->lock();
            if (nxtp->m_value == value) {
                curp->m_next = nxtp->m_next;

                nxtp->m_next->lock();
                nxtp->m_next->m_prev = curp;
                nxtp->m_next->unlock();

                delete nxtp;
                curp->unlock();
                return true;
            }
            curp->unlock();
            curp = nxtp;
            nxtp = curp->m_next;
        }
        curp->unlock();
        return false;
    }

    auto display() -> std::string {
        Node* curp = m_tail;
        Node* tmpp;
        std::string res;
        while (curp->m_prev != m_head) {
            tmpp = curp;
            tmpp->lock();
            curp = curp->m_prev;
            res.append(std::to_string(curp->m_value));
            res.append(1, ' ');
            tmpp->unlock();
        }
        return res;
    }
};

auto sleep_for_random_time(int lim) -> void {
    std::this_thread::sleep_for(std::chrono::milliseconds(rand() % lim));
}

auto sleep_for_fixed_time(int t) -> void {
    std::this_thread::sleep_for(std::chrono::milliseconds(t));
}

auto main() -> int {
    srand(time(0));

    ConcurrentSotedList lst;
    const int lim = 50;
    const int gap = 3;
    
    std::bitset<gap> insert_finish, remove_finish;
    
    std::vector<std::thread> threads;
    
    std::mutex cnt_mut;
    int remove_failure = 0;
    std::vector<int> failures;

    auto insert = [&](int begin) {
        for (int val = begin; val < lim; val += gap) {
            lst.insert(val);
            sleep_for_random_time(1000);
        }
        insert_finish[begin] = true;
    };
    
    auto remove = [&](int begin) {
        for (int val = begin; val < lim; val += gap) {
            if (!lst.remove(val)) {
                cnt_mut.lock();
                remove_failure++;
                failures.push_back(val);
                cnt_mut.unlock();
            }
            sleep_for_random_time(1000);
        }
        remove_finish[begin] = true;
    };
    
    std::thread monitor([&]() {
        while (true) {
            if (insert_finish.count() == gap && remove_finish.count() == gap) break;
            sleep_for_fixed_time(100);
            std::cout << lst.display() << std::endl;
        }
    });
    
    for (int i = 0; i < gap; ++i) {
        threads.push_back(std::thread(insert, i));
    }
    sleep_for_fixed_time(2000);
    for (int i = 0; i < gap; ++i) {
        threads.push_back(std::thread(remove, i));
    }
    
    for (auto& t : threads) t.join();
    monitor.join();
    
    std::cout << "current number of nodes: " << remove_failure << std::endl;
    for (auto& i : failures) {
        std::cout << i << " ";
    }
    std::cout << std::endl
              << "current content of list: " << std::endl;
    std::cout << lst.display() << std::endl;
    
    return 0;
}
```

