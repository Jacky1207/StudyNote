<!--
 * @Author: Jacky
 * @Date: 2023-09-13 14:39:44
 * @LastEditors: Jacky
 * @LastEditTime: 2023-09-13 14:53:46
 * @FilePath: \学习笔记\编程语言\C++\C++11\shared_ptr.md
-->
shared_ptr 有两种构造方式

##### 1.shared_ptr

~~~c++
//	推荐 
shared_ptr<int> sp = shared_ptr<int>(new int(2)); 

// 不推荐
int *p = new int(2);
shared_ptr<int> sp = shared_ptr<int>(p); 
~~~
后一种方式shared_ptr管理的对象的裸指针p暴露在代码中，coder有机会直接delete对象，这样是不安全的。
对于coder在代码中通过裸指针直接delete对象的行为，shared_ptr是不可知的，因此shared_ptr会自以为正常的再次delete对象，导致对象的重复删除。
##### 2.make_shared
~~~c++
shared_ptr<int> sp = make_shared<int>(2);

class Student
{
public:
    Student();
    Student(std::string name);
};

shared_ptr<Student> sp = make_shared<Student>();
shared_ptr<Student> sp1 = make_shared<Student>("John");

~~~
**std::shared_ptr构造函数会执行两次内存申请，而std::make_shared则执行一次。**

#### 使用场景
- 场景一: 有多个使用者共同使用同一个对象，而这个对象没有一个明确的拥有者.
    这就好比教室里的电灯，大家都在使用这个电灯，但没有那个人来专门负责这个电灯的开关。往往是教室里一个人都没有，电灯却明晃晃地亮着。在这种场景下，我们就可以使用shared_ptr来管理这个电灯，通过shared_ptr，谁都可以使用这个电灯，但谁最后使用电灯谁就负责最后关灯。
- 场景二: 某一个对象的复制操作很费时.
    如果一个对象的复制操作很费时，同时我们又需要在函数间传递这个对象，我们往往会选择传递指向这个对象的指针来代替传递对象本身，以此来避免对象的复制操作。
    既然选择使用指针，那么使用shared_ptr是一个更好的选择，即起到了向函数传递对象的作用，又不用为释放对象操心。

- 场景三: 要把指针存入标准库容器.
    不管容器中保存的是普通指针还是智能指针，在使用上，两者并无太大区别，使用智能指针的优越性主要体现在容器使用完毕后清空容器的操作上。如果容器中保存的是普通指针，当我们在清空某个容器时，先要释放容器中指针所指向的资源，然后才能清空这些指针本身。例如，在SalarySys类的析构函数中，我们需要清空vecEmp容器中保存的Employee*指针：

- 场景四: 当管理需要特殊清除方式的资源时，可以通过定制shared_ptr的删除器来实现.

#### 使用示例
~~~c++
class Student {
public:
    Student(const std::string& name, int grade) : name(name), grade(grade) {}
    ~Student() {
        std::cout << "Student " << name << " is destroyed" << std::endl;
    }

    std::string _name(){return name;}

    std::string name;
    int grade;
};

int main() {
    // 创建一个 std::shared_ptr，管理一个新的 Student 对象。
    std::shared_ptr<Student> s1 = std::make_shared<Student>("John", 90);


    // 输出引用计数 (应为 1)
    std::cout << "s1 use count: " << s1.use_count() << s1->_name()  << std::endl;

    // 创建一个新的 std::shared_ptr，管理同一个 Student 对象。
    std::shared_ptr<Student> s2 = s1;

    // 输出引用计数 (现在应为 2，因为 s1 和 s2 都在引用同一个对象)
    std::cout << "s1 use count: " << s1.use_count() << std::endl;
    std::cout << "s2 use count: " << s2.use_count() << std::endl;

    // 将 s1 添加到一个 vector 中
    std::vector<std::shared_ptr<Student>> vec;
    vec.push_back(s1);

    // 输出引用计数 (现在应为 3，因为 s1、s2 和 vector 中的元素都在引用同一个对象)
    std::cout << "s1 use count: " << s1.use_count() << std::endl;
    std::cout << "s2 use count: " << s2.use_count() << std::endl;

    // 当我们清空 vector 时，vector 中的元素将被销毁，引用计数减 1。
    vec.clear();

    // 输出引用计数 (现在应为 2)
    std::cout << "s1 use count: " << s1.use_count() << std::endl;
    std::cout << "s2 use count: " << s2.use_count() << std::endl;

    return 0;
}

输出:
s1 use count: 1John
s1 use count: 2
s2 use count: 2
s1 use count: 3
s2 use count: 3
s1 use count: 2
s2 use count: 2
Student John is destroyed

~~~

~~~~c++
class Temp
{
public:
    Temp(){std::cout << "Temp" << "\n";}
    ~Temp(){std::cout << "~Temp" << "\n";}
};

using SharedPtr = std::shared_ptr<Temp>;
using TmpQueuePtr = std::queue<Temp>;
using SharedQueuePtr = std::queue<SharedPtr>;
int   SLEEP =  100;
static void thread_pro(SharedQueuePtr *q1, SharedQueuePtr *q2)
{
    std::cout << "thread_pro start " << q1 << ":" << q2 <<"\n";
    while(1){
        int size = q1->size();
        std::cout << "q2 queue size "<< q2->size() <<"\n";
        if(size > 0){
            SharedPtr p = q1->front();
            std::cout << "used 1 shared " << "  p.get() = " << p.get() << ", p.use_count() = " << p.use_count() << '\n';
            q2->push(p);
            std::cout << "used 2 shared " << "  p.get() = " << p.get() << ", p.use_count() = " << p.use_count() << '\n';
            q1->pop();
            std::cout << "used 3 shared " << "  p.get() = " << p.get() << ", p.use_count() = " << p.use_count() << '\n';
        }
        std::this_thread::sleep_for(std::chrono::milliseconds(SLEEP));
    }
}

static void out_pro(SharedQueuePtr *q1, TmpQueuePtr *q2)
{
    std::cout << "out_pro start "<< q1 <<"\n";
    while(1){
        int size = q1->size();
        if(size > 0){
            SharedPtr p = q1->front();
            Temp *tmp = p.get();
            std::cout << "Queue 1 shared " << "  p.get() = "<< tmp << ":" << p.get() << ", p.use_count() = " << p.use_count() << '\n';
            q1->pop();
            std::cout << "Queue 2 shared " << "  p.get() = "<< tmp << ":" << p.get() << ", p.use_count() = " << p.use_count() << '\n';
        }
        std::this_thread::sleep_for(std::chrono::milliseconds(SLEEP));
    }
}

int main(int argc, char *argv[])
{
    TmpQueuePtr q1;
    std::queue<SharedPtr> q2, q3;
    
    std::thread t1(thread_pro, &q2, &q3), t2(out_pro, &q3, &q1);
    std::cout << "queue info "<< &q1 << ":" << &q2 << ":" << &q3 <<"\n";

    while(1){
        {
            SharedPtr p = std::make_shared<Temp>();
            std::cout << "Created a shared " << "  p.get() = " << p.get() << ", p.use_count() = " << p.use_count() << '\n';
            q2.push(p);
            std::cout << "q2 queue size "<< q2.size() <<"\n";
            //q1.pop();
        }
        std::this_thread::sleep_for(std::chrono::milliseconds(SLEEP));
    }

    t1.join();
    t2.join();
    return 0;
}

~~~~