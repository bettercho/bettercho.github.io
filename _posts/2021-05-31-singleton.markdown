---
layout: post
title:  "싱글턴 패턴"
date:   2021-05-31 23:50:00
categories: C++ DesignPattern
---
<br/><br/>
오늘 특정 모듈 설계를 하면서 싱글턴으로 구현해야 하는지 인스턴스를 다수개 가질수 있도록 해야하는지에 대한 논의가 있었다.   
싱글턴 패턴이란 무엇이고, 어떤 경우에 싱글턴으로 구현해야 할까?   
<br/><br/>
# Singleton Pattern
싱글턴 패턴(Singleton pattern)으로 구현된 클래스는   
생성자가 여러 차례 호출되더라도 실제로 생성되는 객체는 하나이고   
최초 생성 이후에 호출된 생성자는 최초의 생성자가 생성한 객체를 리턴한다. <br/><br/>

혹은 아예 생성자를 protected 로 선언하여   
외부에서 해당 객체를 만들지 못하게 하고, 특정 static 함수를 통해 생성되도록 구현되기도 한다.<br/><br/>

구현 방법은 다양하지만, 결과적으로 다수의 Thread 가 존재하는 시스템에서   
하나의 인스턴스만 생성되어 필요한 Thread 들에 공유되어 사용할 수 있게 하는 것이 싱글턴 패턴이다.   
<br/>
여러 곳에서 동시에 데이터가 변경되면 안되는 경우에 사용할 수 있고,   
다른곳에서 변경된 데이터가 모든 시스템에 바로 적용이 되어 공유되어야 할 경우에 유용하게 사용할 수 있다.
<br/>
<br/>
Sigleton 을 구현하는 방법은 다양한데, 한가지 예를 가져와봤다. 
<br/>

```c++
    class Singleton
    {
    private:
        static Singleton * pinstance_;
        static std::mutex mutex_;

    protected:
        // Singleton의 생성자와 소멸자는 다른곳에서 new 로 생성되는 것을 막기 위해 public 으로 선언되어서는 안된다.
        Singleton(const std::string value): value_(value)
        {
        }
        ~Singleton() {}
        std::string value_;

    public:
        /**
        * Singletons should not be cloneable.
        */
        Singleton(Singleton &other) = delete;
        /**
        * Singletons should not be assignable.
        */
        void operator=(const Singleton &) = delete;
        // 다른 각체에서는 GetInstance 함수를 통해 Singleton 객체를 가져갈 수 있다.
        // 처음 호출 시 new 되고, 그 뒤부터는 이미 생성된 객체를 reuse 한다.
        static Singleton *GetInstance(const std::string& value);
        /**
        * Finally, any singleton should define some business logic, which can be
        * executed on its instance.
        */
        void SomeBusinessLogic()
        {
            // ...
        }
        std::string value() const{
            return value_;
        } 
    };

    Singleton* Singleton::pinstance_{nullptr};
    std::mutex Singleton::mutex_;

    // 멀티쓰레드 환경에서 안전하기 위해 lock 을 사용해 Singleton 객체가 생성될 때 까지 다른 쓰레드에서 접근하지 못하도록 한다.
    // Static 함수는 class 밖에 구현되어야 한다.
    Singleton *Singleton::GetInstance(const std::string& value)
    {
        std::lock_guard<std::mutex> lock(mutex_);
        if (pinstance_ == nullptr)
        {
            pinstance_ = new Singleton(value);
        }
        return pinstance_;
    }

    void ThreadFoo(){
        // Following code emulates slow initialization.
        std::this_thread::sleep_for(std::chrono::milliseconds(1000));
        Singleton* singleton = Singleton::GetInstance("FOO");
        std::cout << singleton->value() << "\n";
    }

    void ThreadBar(){
        // Following code emulates slow initialization.
        std::this_thread::sleep_for(std::chrono::milliseconds(1000));
        Singleton* singleton = Singleton::GetInstance("BAR");
        std::cout << singleton->value() << "\n";
    }

    int main()
    {   
        std::cout <<"If you see the same value, then singleton was reused (yay!\n" <<
                    "If you see different values, then 2 singletons were created (booo!!)\n\n" <<
                    "RESULT:\n";   
        std::thread t1(ThreadFoo);
        std::thread t2(ThreadBar);
        t1.join();
        t2.join();
        
        return 0;
    }
```


실행결과)   
If you see the same value, then singleton was reused (yay!   
If you see different values, then 2 singletons were created (booo!!)   

RESULT:   
BAR   
BAR   


mutex lock 을 사용하지 않으면 멀티쓰레드일 때 객체가 두개 생성되는 문제가 있다.    
그래서, 객체를 생성하는 부분에는 꼭 **lock** 을 사용해 멀티쓰레드에서도 안전하게 동작하게 해야 한다.



