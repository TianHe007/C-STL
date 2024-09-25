- **进程**：操作系统进行资源分配的最基本单位
    - 操作系统在分配内存的时候，操作系统是吧内存空间分配给某一个进程
    - 比如申请一些读写资源的时候，这些资源也就是被分配给了某个进程
- 线程是进程中的概念，因此线程也是一类资源，也是操作系统分配给进程的一批资源
    - 创建和销毁线程都是相对昂贵的操作，特别是在高并发场景下，频繁地创建和销毁线程会极大地降低程序的性能。通过线程池预先创建一定数量的线程并保存在内存中，可以避免频繁地创建和销毁线程，从而提高程序的性能
    - 也就是说申请一个线程，操作系统就要拿出一份资源，申请两个，操作系统就要拿出来两个资源
    - 比如递归深度过深跑着跑着就爆栈了，那么他爆的是什么栈呢，爆的就是我们线程所占的栈空间，而我们一个线程所占用的栈空间是多大呢：8Mb，所以操作系统每申请一个线程，就要多占用8Mb的空间
    - 但是我们在设计程序的时候都希望我们的程序时可控的，比如我们进程在申请空间的时候最大不能超过4GB，那么我们就可能在程序中使用一个**内存池**一样的概念，把我们程序中所有申请内存的动作统一的管控起来，这样就可以保证是说我的进程中包含的所申请的内存空间大小不会超过一个范围，也就是我申请内存的动作在我的进程中是可控的
    - 那么既然申请内存这个动作他是可控的，那么既然线程也代表了一类资源的话，是不是按理来说我们希望申请线程这个资源也是可控的
- 根据我们之前使用多线程的一个方式的话显然申请多线程这个动作在我们程序中他就不是可控的
    - 有一个线程对象就有一个线程，有两个线程对象就有两个线程，有100个对象就有100个线程
- **那么我们要想让我们线程资源可控，那么就提出线程池这个概念**
    
   
    
    - 在我的进程中有这么一个地方，这个地方叫做线程池，线程池内部可能装着**若干个线程**
    - 而线程池中究竟装着多少线程这玩意是可控的，假设他最多就装10个，这就是在线程池中线程的数量是可控的
    - 接下来在程序中有一些计算任务，而这些计算任务，我希望把他们在多线程的场景中执行的时候，那么我们就会吧这个计算任务打包成为一个独立的个体然后塞到一个所谓的**任务队列**中
    - 任务队列是干什么的呢，任务队列就是给每一个线程去取任务的，而这些线程就是到这些任务队列中去取任务，线程的工作方式就是类似于流水线上那个工人的工作方式，线程看到任务队列来任务啦，然后取走然后干完然后干完之后把结果放回去，然后再去取任务然后再干完再放回去，周而复始….这就是线程池中线程的工作逻辑
    - 那什么是**计算任务**呢？
        - 完整的计算任务应该包含计算过程和计算数据
        - 而在我们程序中计算过程就是我们的**函数方法**，计算数据就是我们给**函数传递的参数**
        - 那么我们**如何将函数方法和参数进行打包呢**？—**bind方法**
- 至此线程池概念已经理解，接下来进行程序设计
    - 我们首先封装一个Task类，表示计算任务，他的作用就是将函数参数和函数方法进行绑定，然后返回一个可调用对象，便于后续线程的直接调用
    
    ```cpp
    class Task {
    public :
        template<typename FUNC_T, typename ...ARGS>
        Task(FUNC_T func, ARGS ... args) {
            this->func = bind(func, forward<ARGS>(args)...);
        }
        void run() {
            this->func();
            return;
        }
    private :
        function<void()> func;
    };
    
    void test() {
        cout << "hello world" << endl;
        return;
    }
    
    void func(int a, int b) {
        cout << "function " << a << " " << b << endl;
        return ;
    }
    
    void add_one(int &n) {
        n += 1;
        return;
    }
    int main() {  //测试代码
        Task t(func, 3, 4);
        t.run();
        t.run();
        Task t2(test);
        t2.run();
        int n = 1;
        Task t3(add_one, ref(n));
        t3.run();
        t3.run();
        t3.run();
        t3.run();
        cout << n << endl;
        return 0;
    }
    ```
    
    - 接下来进行线程池的设计，假设有一个构造函数传入值，表示线程池中有几个线程
    
    ```cpp
    class Threadpool {
    public :
        Threadpool(int n = 1) : threads(n) {
        }
    };
    
    ```
    
    - 这些线程我是不是要存起来，因此使用vector<thread*>进行线程的存储
    - 那么存储的线程执行什么任务呢？
        - 取任务，执行任务，是不是类似于一个死循环的过程
    - 设计函数入口worker，创建的线程对象执行worker任务，那么就有
    
    ```cpp
    class Threadpool {
    public :
        Threadpool(int n = 1) : threads(n) {
            for (int i = 0; i < n; i++) {
                threads[i] = new thread(worker);
            }
        }
        void worker() {
            while (1) {
                //取任务
                //执行任务
            }
            return;
        }
    private :
        vector<thread *> threads;
    };
    
    ```
    
    - 具体来说，我们应该使用Threadpool类中的worker函数，因此变成了
        - `threads[i] = new thread(&Threadpool::worker);`
        - 加上我们之前对虚函数表和this指针的深入理解，成员函数有一个隐藏参数，this指针，因此再进行线程创建的时候，要将这个this指针传进去
    
    ```cpp
    class Threadpool {
    public :
        Threadpool(int n = 1) : threads(n) {
            for (int i = 0; i < n; i++) {
                threads[i] = new thread(&Threadpool::worker, this);
            }
        }
        void worker() {
            while (1) {
                //取任务
                //执行任务
            }
            return;
        }
    private :
        vector<thread *> threads;
    };
    ```
    
    - 那么后续如何设计呢？再设计之前需要想清楚线程池我们想怎么用：
    
    ```cpp
    bool isprime(int x) {
        for (int i = 2; i * i <= x; i++) {
            if (x % i == 0) return false;
        }
        return true;
    }
    
    int prime_count(int l, int r) {
        int ans = 0;
        for (int i = l; i <= r; i++) {
            ans += isprime(i);
        }
        return ans;
    }
    
    void worker(int l, int r, int &ret) {
        ret = prime_count(l, r);
        return ;
    }
    
    int main() {
        Threadpool tp(5);
        const int batch = 5e6;
        int ret[10];
        for (int i = 0, j = 1; i < 10; i++, j += batch) {
            tp.add_task(worker, j, j + batch - 1, ref(ret[i]));
        }
        tp.stop();
        int ans = 0;
        for (auto x : ret) ans += x;
        cout << ans << endl;
        return 0;
    }
    ```
    
    - 我们将前的判定素数的代码拷贝进来，然后准备10个任务，add_task表示向线程池中丢进去一个任务，我们线程池中有5个线程，但是一共有10个任务！
    - `stop`表示等待所有线程执行完毕然后将线程停掉，然后就可以计算获取答案了
    - `worker`方法是判断当前线程是否再执行，如果再执行那么我while循环就成立，就可以进行任务操作
    - 设计一个`unordered_map`类，里面放入`thread::id`
        - 如果忘记类型，那么就可以反推：`decltype(this_thread::get_id())`
        - 值类型为`bool`类型，表示当前id的线程是否正在执行
    
    ```cpp
    void worker() {
            auto id = this_thread::get_id();
            running[id] = true;   //表示当前id正在执行
            while (running[id]) {
                Task *t = get_task();
                t->run();
                delete t;
            }
            return;
        }
        
    unordered_map<decltype(this_thread::get_id()), bool> running;
    ```
    
    - 使用`get_task`函数获取一个任务，然后调用`Task`类中的`run`方法，进行任务执行，最后销毁
    - 再设计一个stop方法，将若干线程给他停掉，如何设计？
        - 使用**毒药方法**：就是说一旦线程取到这个任务，那么就下岗，自己杀自己
        
        ```cpp
        void stop_running() {
                auto id = this_thread::get_id();
                running[id] = false;
                return;
            }
        ```
        
        - 获取当前线程id，然后将运行状态表示为false
    - 有了毒药之后，给所有线程派出这个任务，使得他们全部取到，也就说将他们的状态全部标记为false
    
    ```cpp
    void stop() {
            if (starting == false) return;
            for (int i = 0; i < threads.size(); i++) {
                this->add_task(&Threadpool::stop_running, this);
            }
            for (int i = 0; i < threads.size(); i++) {
                threads[i]->join();
            }
            for (int i = 0; i < threads.size(); i++) {
                delete threads[i];
                threads[i] = nullptr;
            }
            starting = false;
            return;
        }
    ```
    
    - 将线程池运行状态`starting`标记为`false` ，然后给每一个线程状态标记为false，然后依次等待每一个线程执行结束，最后将这些线程全部释放掉即可
        - 注意，也可以直接`threads[i] = nullptr`，因为将指针赋值为nullptr会自动调用对象的析构函数
    - 随后实现`get_task`和`add_task`，首先分清楚这`get_task`是给内部使用的，`add_task`是给外部使用的，因此`get_task`实现为私有成员方法
    - `add_task`
        - 需要将相关参数打包进我们的任务队列中，因此需要一个任务队列
        - 注意该任务队列是临界资源，因此访问临界资源需要加锁`unique_lock<mutex> lock(m_mutex);`
        - 下面表示已经抢占到锁，然后就可以push进去一个新的任务对象
        
        ```cpp
        void add_task(FUNC_T func, ARGS... args) {
                unique_lock<mutex> lock(m_mutex);
                tasks.push(new Task(func, forward<ARGS>(args)...));
                m_cond.notify_one();  //见下面get_task代码解释
                return;
            }
        ```
        
    - `get_task`
        - 获取一个任务队列中的任务，肯定要访问临界资源。因此需要加锁，和上面一样
        - 判断当前队列是否为空，只要队列为空，那么就要一直等到队列非空为止
            - **多线程编程：条件信号量(condition_variable)**
            - 再多线程编程中就类似于条件通知，相当于一个条件信号，我每一个条件信号量都代表一个条件，当条件满足的时候就会被触发，条件不满足的时候就一直等着
            - 该地方的条件信号量表示该队列中是否被放入了任务，如果放入了那么就会被触发
            - **条件信号量需要配合一个锁使用**
            - 当我们往队列放入任务的时候，我们就需要通知所有等待着添加任务的线程说我已经添加了一个任务因此 `m_cond.notify_one();` 表示所有等在这个状态`m_cond`下的线程，通知他说我已经添加了一个任务
            - 当我队列为空的时候，我什么东西都取不到啊，那么我就用条件信号在这里等着并且释放这个锁，等待信号再等待的时候他会把这个lock锁释放掉，这样的话其他添加任务的线程才有可能继续添加任务
            - 为什么使用while循环呢？因为他有可能采用**虚假唤醒**
                - 添加任务的时候我通知了一下，但是呢这个时候突然被另外一个线程取走了然后我紧接下来是我等待着这个条件信号量中的线程被唤醒，唤醒以后呢他要去取得任务其实已经被其他线程取走了，这个时候队列依然为空，所以要用while循环防止虚假唤醒
        
        ```cpp
        Task *get_task() {
                unique_lock<mutex> lock(m_mutex);
                while (tasks.empty()) m_cond.wait(lock);
                Task *t = tasks.front();
                tasks.pop();
                return t;
            }
        private: 
            condition_variable m_cond;
        ```
        
    - 接下来优化程序设计，增加start函数，构造函数调用start函数，再start函数内部进行线程的创建，使用成员变量starting表示线程池运作，否则是停止状态
    - 析构函数直接调用stop函数，然后将队列清空即可
    - 线程池类代码
        
        ```cpp
        class Task {
        public :
            template<typename FUNC_T, typename ...ARGS>
            Task(FUNC_T func, ARGS ... args) {
                this->func = bind(func, forward<ARGS>(args)...);
            }
            void run() {
                this->func();
                return;
            }
        private :
            function<void()> func;
        };
        
        class Threadpool {
        public :
            Threadpool(int n = 1) : thread_size(n), threads(n), starting(false) {
                this->start();
                return;
            }
            void worker() {
                auto id = this_thread::get_id();
                running[id] = true;
                while (running[id]) {
                    Task *t = get_task();
                    t->run();
                    delete t;
                }
                return;
            }
            void start() {
                if (starting == true) return;
                for (int i = 0; i < thread_size; i++) {
                    threads[i] = new thread(&Threadpool::worker, this);
                }
                starting = true;
                return;
            }
            void stop() {
                if (starting == false) return;
                for (int i = 0; i < threads.size(); i++) {
                    this->add_task(&Threadpool::stop_running, this);
                }
                for (int i = 0; i < threads.size(); i++) {
                    threads[i]->join();
                }
                for (int i = 0; i < threads.size(); i++) {
                    delete threads[i];
                    threads[i] = nullptr;
                }
                starting = false;
                return;
            }
            template<typename FUNC_T, typename ...ARGS>
            void add_task(FUNC_T func, ARGS... args) {
                unique_lock<mutex> lock(m_mutex);
                tasks.push(new Task(func, forward<ARGS>(args)...));
                m_cond.notify_one();
                return;
            }
            ~Threadpool() {
                this->stop();
                while (!tasks.empty()) {
                    delete tasks.front();
                    tasks.pop();
                }
                return;
            }
        private :
            Task *get_task() {
                unique_lock<mutex> lock(m_mutex);
                while (tasks.empty()) m_cond.wait(lock);
                Task *t = tasks.front();
                tasks.pop();
                return t;
            }
            void stop_running() {
                auto id = this_thread::get_id();
                running[id] = false;
                return;
            }
            bool starting;
            int thread_size;
            std::mutex m_mutex;
            condition_variable m_cond;
            vector<thread *> threads;
            unordered_map<thread::id, bool> running;
            queue<Task *> tasks;
        };
        
        ```
