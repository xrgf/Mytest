数据共享问题

//通过容器创建多个线程

//这种多线程他会进行资源抢夺 谁快谁先输出

#include<iostream>
#include<thread>
#include<vector>
using namespace  std;
void printData(int i) {

	cout <<"线程序号::"<< i << endl;
}
void TestCreateThread() {

	vector<thread*>test;
	for (int i = 0; i < 10; i++) {
		test.push_back(new thread(printData, i));
	}
	for (auto v : test) {
		v->join();
	}
	cout << "主线程" << endl;
}
int main() {


	TestCreateThread();



	return 0;

}		

//数据共享问题分析
//只读操作：稳定安全，不需要特殊处理，直接读就可以
vector<int> g_data={ 1,2,3 };
void printVector(int i) {
	cout << "子线程id::" << i << endl;
	for (auto v : g_data) {
		cout << v << "  ";
	}
	cout << endl;
}
void TestThread() {

	vector<thread*>test;
	for (int i = 0; i < 10; i++){
	
		test.push_back(new thread(printVector, i));
	}
	for (auto v:test) {
		v->join();
	}
}

共享锁

class Seaking {
public:
	void makeFriend() {
		for (int i = 0; i < 10000; i++) {

			//加锁
			m_mutex.lock();
			cout << "增加一个女朋友" << endl;
			mm.push_back(i);
			m_mutex.unlock();
		}
	}
	void breakup() {
		for (int i = 0; i < 10000; i++) {
			if (!mm.empty()) {
				m_mutex.lock();
				cout << "减少一个女朋友" << endl;
				mm.pop_back();
				m_mutex.unlock();
			}
			else {
				cout << "单身狗一个" << endl;
			}
		}
	}

private:
	list<int>mm;
	mutex m_mutex;//构建互斥量对象


};
/*	mutex类(互斥量)创建互斥量类的对象
* 1.1通过调用lock函数进行加锁
* 1.2通过调用unlock函数进行解锁
* 注意点lock和unlock成对出现

*/



//独享锁

class DoSomething {
public:
	void wc() {

		for (int i=0; i < 100; i++) {
	
			//No.1 unique_lock加锁过程
			//unique_lock<mutex>m(mtx);
			//No.2 unique_lock其他参数
			//2.1 adopt_lock先进行lock操作，不适用，程序会调用abort函数终止程序
			//mtx.lock();
			//unique_lock<mutex>unique(mtx, adopt_lock);
			//2.2 defer_lock,需要自己手动创造lock，创键一个没有lock的锁
			//unique_lock<mutex>unique(mtx, defer_lock);//想到一个没有上锁的锁
			//unique.lock();unique.unlock()
			//2.3 try_to_lock
			unique_lock<mutex>unique(mtx, try_to_lock);
			cout << "上厕所" << endl;
			if (unique.owns_lock()) {
				num.push_back(i);
			}


​			
​		}
​	}
​	void batch() {
​		for (int i = 0; i < 100; i++) {
​			if (!num.empty()) {
​				unique_lock<mutex>m(mtx);
​				cout << "batch" << endl;
​				num.pop_back();
​			}
​			else {
​				cout << "正在做事" << endl;
​			}


		}
		
	}

private:
	list<int> num;
	mutex mtx;


};

条件变量

#include<condition_variable>
#include<iostream>
#include<thread>
#include<chrono>
#include<mutex>
using namespace std;
/*
	condition_variable类
	1.调用waid函数 阻塞线程
	2.unique_lock 加锁线程
	3.notify_one notify_all 唤醒线程
*/
condition_variable cv;
mutex cv_m;
int i = 0;// 描述唤醒条件
bool done = false;//充当开关
void wait_one() {
	unique_lock<mutex>lk(cv_m);
	cout << "等待中......" << endl;
	cv.wait(lk, [] {return i == 1; });
	//this_thread::sleep_for(1s);
	cout << "运行结束....." << endl;
	done = true;
}
void signal_one() {
	this_thread::sleep_for(1s);

	cout << "不做条件变量唤醒操作....." << endl;
	cv.notify_one();//不做唤醒
	
	unique_lock<mutex>lkc(cv_m);
	i = 1;
	while (!done) {
		
		cout << "条件变量的唤醒操作" << endl;
		lkc.unlock();
		cv.notify_one();//唤醒线程中的一个线程
		
		this_thread::sleep_for(1s);
		
		lkc.lock();
	}
}
void TestNotifyOne() {
	thread t1(wait_one), t2(signal_one);
	t1.join();
	t2.join();
}
int main() {


	TestNotifyOne();
	
	return 0;

}

//浅谈自己对条件变量的理解 为什么要condition_variable和mutex要一起用

//mutex相当于一个共同的资源 然后condition_variable有二个线程一个是等待一个是唤醒等待的线程

//然后防止二者资源竞争问题用一个mutex等待线程的上锁关锁是wait函数来决定的wait函数阻塞的时候关锁

//wait函数满足条件的时候上锁

//另外一个唤醒等待函数的线程就要防止和等待函数线程有资源竞争二者就要  等待函数开  （unique_mutex<>是默认有构造和析构函数上锁开关）唤醒等待函数就关 这样就防止二个人资源竞争的问题 保证正常输出







***原子类型

#include<iostream>
#include<thread>
#include<atomic>
using namespace std;


/*


	原子操作：对变量读写操作不可分割
	atomic:模版类
	atomic<int>f();
	//写操作
	inum.store(2, memory_order::memory_order_relaxed);
	//读操作
	inum.load(memory_order::memory_order_relaxed);
	//修改
	inum.exchange(2, memory_order::memory_order_relaxed);
	//memory_order 内存模型
	enum memory_order {
	memory_order_relaxed,//对内存顺序不做要求   有效的
	memory_order_consume,//本线程中，后续原子操作必须在本原子操作结束之后开始
	memory_order_acquire,//本线程中，后续原子 读 操作必须在本原子操作结束之后开始
	memory_order_release,//本线程中，后续原子 写 操作必须在本原子操作结束之后开始   有效的
	memory_order_acq_rel,//  memory_order_acquire 和memory_order_release 综合
	memory_order_seq_cst //全部的读写按照顺序来做    有效的
	
	强顺讯:代码顺序和执行顺序一样
	弱顺序：代码执行顺序会被处理器适当调整
	注意点：
	并不会使用
	memory_order_consume
	memory_order_acquire
	memory_order_acq_rel
	不能使用
};



*/
atomic<int>a;
atomic<int>b;
void threadFunc() {
	int t = 1;
	a = t;
	b = 1;//这行代码不依赖上面的任何，执行过程，b=1，可能比上面执行的快
}
void SetValue() {
	int t = 1;
	a.store(t);//a=t;
	b.store(2);
}
void PrintAtomic() {
	
	//cout << a << "\t" << b << endl;
	//直接这样输出的话 他可能这个线程先进行
	a.exchange(1111);
	cout << a.load() << "\t" << b.load();
	
	//写操作必须在本操作完成后再进行


​	
}
int main() {

	thread t1(SetValue), t2(PrintAtomic);
	
	t1.join();
	//t3.join();
	t2.join();
}

//***自旋锁

#include<iostream>
#include<atomic>
#include<thread>
using namespace std;
/*

	atomic_flag:无锁的
		<1>test_and_set():如果没有被设置就设置一下，如果设置了就返回true
		<2>clear():清除标记 让下一次调用test_and_set()返回false


	其他原子类型：is_lock_free()//判断是否无锁


	自旋锁：互斥锁的特点是自愿申请者只能进入休眠状态，自旋锁不会引起调用者的休眠
	        调用者一直循环在哪里，等待锁的释放。



*/
atomic_flag lock = {};
void func1(int n) {
	while (lock.test_and_set(memory_order::memory_order_relaxed)) {
		cout << "等待中......" <<n<< endl;
	}
	cout << "线程结束....."<<n << endl;
}
void func2(int n) {
	cout << "线程启动...." << n << endl;
	this_thread::sleep_for(10ms);
	lock.clear();//线程2操作这个操作线程1才不满足条件才可以退出
	//相当于1做1的事 2做2的事情  2还可以通知1停止做
	cout << "线程完成....." << n << endl;
}
int main() {


	lock.test_and_set();
	thread t1(func1, 1);
	thread t2(func2, 2);
	t1.join();
	t2.join();



	return 0;

}