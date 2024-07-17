# C++基础语法
## 1、C++多态的几种形式
	主要通过子类和父类继承机制来实现一个基类的不同形态来达到调用特定函数的目的实现多态，或者通过
1. 临时多态——在编译时确定对象的形态和调用的函数，主要是函数的重载实现。
2. 动态多态（运行时多态）——主要通过父类的虚函数和子类的重写实现。
3. 参数化多态（编译时多态）——模板实现，通过将类型作为参数来实现多态。对不同的类型变量执行相同代码操作。
4. 强制转换——通过类型转换实现多态，基础类型转化为目标类型实现多态，如果构造函数没有定义explicit关键词，也会发生强制转换。在自定义类型中可以重定义类型转换符operator int()
## 2、C++的智能指针
1. auto_ptr-C++98提出，可以托管指针，防止内存泄漏，该类重载了*和->操作符，可以像普通指针一样使用，使用方式为auto_ptr<类型> aptr(new 类型)，不支持auto_ptr<int[]> aptr(new int[]),不能对数组对象的内存进行托管。release函数取消内存托管但需要自己释放内存，get函数获得对象指针，reset函数重置托管的内存地址，若参数为空指针，释放托管对象。但是不支持可复制构造copy constructable和可赋值，会导致对象的所有权的转移，不能对数组对象的内存进行管理。
2. unique_ptr-C++11提出，排他所有权模式，两个指针不能指向同一个资源；无法进行左值构造，可以用临时右值构造或者赋值；在在指向某个对象的指针离开作用域时会自动释放它指向的对象；内存中保存指针是安全的。
在stl容器中和auto_ptr类似，不允许直接用左值赋值，需要用移动语义转换为临时右值，都具有排他性可以使用unique_ptr<int[]> uptr(new int[])托管数组内存对象。
```C++
unique_ptr<string> p1(new string("test1"));
unique_ptr<string> p2(new string("test2"));
		
p1 = p2; //错误，不允许左值赋值或者构造；
unique_ptr<string> p3(p2); // 错误，不允许左值构造；
p1 = std::move(p2); //正确，临时右值赋值；
unique_ptr<string> p3(std::move(p2)); // 正确，移动语义临时左值构造；
```
3. shared_ptr—为了解决auto_ptr和unique_ptr的排他所有权问题——两个指针不能同时指向同一个内存对象，也就是不支持可复制可赋值，提出了shared_ptr，该指针在对象创建时会记录一个引用计数，表示有多少指针指向该对象内存，当有新的指针复制或拷贝时，引用计数+1，当指针取消指向该对象也就是智能指针析构时，引用计数-1，当引用计数为0时表示没有智能指针指向该对象内存，那我们就释放该对象内存。
```C++
// 仿函数
class Destruction{
	public:
	void operator()(string* str){
		delete str;
		std::cout<<str+"is destructed"<<std::endl;
	}
}
shared_ptr<string> p1;
string *str = new string("test1");
p1.reset(str);
shared_ptr<string> p2(p1); // 指向同一对象
shared_ptr<string[]> p3(new string[3]{"test1","test2","test3"}); // C++17以后支持，指向类型为T[]的数组对象
shared_ptr<string> p4(new string("test"),Destruction()); // C++17以后支持，指向一个T类型对象，并将Destruction作为删除器
shared_ptr<string> p5 = make_shared<string>("test"); // make_shared效率更高，在动态内存中初始化对象并且返回一个指向该对象的shared_ptr对象，圆括号内的参数列表最多支持十个。
std::cout<<"p1 use_count"<<p1.use_count()<<endl;
std::cout<<"p2 use_count"<<p2.use_count()<<endl;

```
4. 仿函数-实际上是一个类，重载了对应操作符operator()，
		
		