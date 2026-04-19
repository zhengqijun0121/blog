# C++11 新特性详细总结


# C++11 新特性全面梳理与总结

用户需详细了解C++11新特性，我会先梳理C++11核心模块，避免内容零散，按类别清晰总结，讲清各特性核心作用、用法及关键点。

我会先回忆C++11全称及重要地位，接着分模块梳理。语言核心层面，从类型推导、右值引用等14个方面展开，如auto、decltype用于类型推导，右值引用实现移动语义，lambda表达式有特定语法和捕获列表等。

我会继续梳理C++11新特性，语言核心方面，有noexcept异常说明符、追踪返回类型等，还需注意auto和decltype的规则。标准库新增特性分类梳理，包括智能指针、容器相关、并发编程相关、函数对象与回调、元编程相关及其他工具。

我会补充std::begin/std::end、std::ref/std::cref等特性。计划分两大块梳理，先搭建整体结构，再纠正常见误区，如明确auto、lambda等特性的适用标准。随后，按小节逐个梳理C++11核心语言特性，如类型推导机制中的auto关键字，阐述其作用、语法等。

我会阐述auto与decltype关键字的规则及示例。auto必须初始化，推导时会忽略顶层const、volatile等，不能用于函数形参等推导。示例有auto i = 42等。注意auto不会推导为右值引用，除非用decltype(auto)或显式&&配合std::move。decltype作用是推导表达式精确类型，不依赖初始化，常用于泛型编程。其规则为，若表达式是标识符等，推导为该实体声明类型；若是左值表达式，推导为T&，纯右值推导为T，将亡值推导为T&&。示例有decltype(x) a = 10等。

我会通过示例阐述decltype，如decltype(x + 1) c = 20推导为int。其典型场景是配合auto实现尾置返回类型，解决模板函数返回值依赖参数问题。接着介绍尾置返回类型，它能解决函数返回值类型依赖参数无法提前声明的问题，语法为auto 函数名(参数列表) -> decltype(表达式) { 函数体 } ，示例展示了在泛型编程中无需提前知类型即可推导返回值类型的优势。最后说明右值引用与移动语义，C++98临时对象拷贝有开销，移动语义转移资源所有权提升性能，右值引用用&&标记，绑定到右值延长其生命周期，还介绍了左值与右值的核心区别。

通过示例展示右值引用，如int&& rref = 10，10是纯右值，rref绑定后生命周期延长，而具名右值引用本身是左值。介绍移动构造函数与移动赋值运算符，作用是接管源对象资源，避免资源重复申请释放，语法如示例代码。核心要点为移动函数加noexcept保证异常安全，编译器会自动生成默认移动构造/赋值函数。

我会阐述移动对不同对象的性能影响，移动仅对持有堆资源对象有收益。介绍std::move函数，它将左值转为右值引用触发移动语义，原对象状态未定义。还会说明完美转发std::forward，在模板函数中保持参数值类别不变，解决右值引用引用折叠问题，通过示例展示其在工厂函数等场景的应用。

Lambda表达式作用为创建匿名函数对象，简化回调等参数传递，捕获上下文变量。完整语法含捕获列表、参数列表等部分。捕获列表有多种方式，如值捕获、引用捕获等。参数列表与普通函数一致，C++11不支持auto参数，C++14泛型lambda支持。mutable可修改值捕获变量副本，异常说明用于指定函数是否抛出异常，返回类型可省略。示例展示简单加法lambda。

通过add函数调用示例，展示Lambda表达式创建匿名函数对象及捕获列表、参数列表等语法。如值捕获加mutable修改副本示例，配合STL算法对容器元素操作示例。核心要点为每个Lambda生成唯一匿名类实例，引用捕获需保证外部变量生命周期长于闭包，无捕获lambda可隐式转函数指针。还介绍统一初始化，用{}统一初始化方式，防止窄化转换，示例有基本类型、数组、容器及关联容器初始化。

我会介绍C++11的统一初始化、std::initializer_list及范围for循环。统一初始化用{}初始化，禁止窄化转换，解决“最令人头疼的语法解析”问题。std::initializer_list让自定义类型支持列表初始化，是轻量级代理对象，指向数组首地址和长度，示例展示其用法及注意事项。范围for循环简化容器/数组遍历语法，无需手动管理迭代器，代码更简洁安全，语法为for (元素声明 : 可迭代对象) { 循环体 } 。

通过vector和数组遍历示例，阐述范围for循环规则，即可迭代对象包括数组、STL容器等，循环中修改容器可能致迭代器失效，auto推导默认值拷贝，改元素需加&。还介绍nullptr关键字，为解决C++98中NULL宏二义性，它是std::nullptr_t类型空指针常量，可隐式转指针类型，不可转整数类型（除bool） 。

我会通过示例阐述nullptr，它可隐式转指针类型，不能转整数类型（除bool），建议用于所有空指针场景替代NULL。还会介绍强类型枚举，解决C++98枚举缺陷，语法为enum class 枚举名 : 底层类型 { 枚举项 }，有强作用域、禁止隐式转换、可指定底层类型等特性，示例展示其用法及与C++98枚举对比。

我会阐述C++11类功能增强特性，其中 =default 用于显式要求编译器生成默认特殊成员函数，即便用户自定义构造函数，也能生成默认版本，示例展示了其用法。 =delete 则用于显式禁用函数，禁止编译器生成或用户调用，典型场景包括禁用拷贝构造/赋值实现不可拷贝类，以及禁用特定重载函数防止隐式转换，示例也进行了说明。

我会阐述C++11的override、final关键字及委托构造函数。override显式声明重写基类虚函数，编译器检查基类对应虚函数，避免笔误。final有修饰类和虚函数两种作用，修饰类则该类不可继承，修饰虚函数则该虚函数不可被子类重写。委托构造函数允许一个构造函数调用同一类其他构造函数，减少代码冗余，简化初始化逻辑，语法是在构造函数初始化列表调用其他构造函数。

我会阐述C++11的委托构造和继承构造函数。委托构造允许一个构造函数调用同一类其他构造函数，需注意不能同时初始化成员变量且不能形成委托循环。继承构造函数让子类继承基类构造函数，简化代码，但要留意子类新增成员变量及参数相同构造函数的覆盖问题。此外，还会介绍变长模板参数包，它实现泛型编程高度灵活性，是元编程核心特性。

我会阐述变参模板与constexpr特性。变参模板用...表示参数包，分模板和函数参数包，核心操作是展开。通过递归展开示例，实现任意参数打印，还展示模板类变长参数包用法，其典型场景在标准库相关函数实现中。constexpr作用是将计算移至编译期，提升性能，替代旧方案。其变量需用编译期常量初始化，函数体只能有return语句（C++11限制，C++14放宽），传入编译期常量时在编译期计算结果。

通过示例阐述constexpr特性，其变量需编译期常量初始化，函数体只能有return语句（C++11限制，C++14放宽），传入运行期变量会退化为普通函数，类构造函数也可声明为constexpr。还介绍static_assert，它是编译期断言，语法为static_assert(常量表达式, "错误提示字符串")，用于检查条件，若不满足在编译阶段输出错误信息，提升调试效率，断言条件必须是编译期可计算的常量表达式。

我会阐述C++11的noexcept、thread_local、alignas/alignof等特性。noexcept声明函数不抛异常，配合移动语义优化并保证异常安全，可带常量表达式，如移动构造函数常加noexcept。thread_local声明变量线程级生命周期，每个线程有独立副本，跨平台替代C++98平台相关方案。alignof获取类型对齐字节数，alignas指定变量/类型对齐字节数，覆盖默认规则，如示例中按16字节对齐结构体。

我会阐述C++11的alignof、explicit等特性。alignof用于获取类型对齐字节数，如示例中获取自定义结构体对齐字节数为16。explicit禁止隐式类型转换，示例中通过将类型转换操作符声明为explicit，实现对隐式转换的限制。还会介绍C++11标准库新增的智能指针，基于RAII机制，自动管理堆内存。其中std::unique_ptr独占对象所有权，不支持拷贝构造和赋值，仅支持移动语义，替代auto_ptr。

我会介绍std::unique_ptr和std::shared_ptr。std::unique_ptr开销小，通过std::move转移所有权，调用reset手动释放对象，get获取原始指针。std::shared_ptr基于引用计数，多个可指向同一对象，引用计数增减原子操作，线程安全，但对象读写需手动加锁。示例展示了两者创建、引用计数变化及释放等操作。

注意到循环引用会致内存泄漏，如两对象互持shared_ptr，引用计数永不为0。我会介绍std::weak_ptr，它辅助shared_ptr，不增引用计数，不控对象生命周期，用于观察shared_ptr状态。其常用方法有expired()检查对象是否释放、lock()获取shared_ptr、use_count()获取引用计数。通过示例，展示如何用weak_ptr解决循环引用，使对象离开作用域时能正确析构。

我会梳理C++11容器新增与增强特性。新增容器方面，std::array是固定大小数组，封装原生数组并提供STL接口，比原生数组安全且无堆内存开销；std::forward_list是单向链表，节省内存，插入删除效率高但不支持反向迭代器和size()方法；无序关联容器基于哈希表实现，查找、插入、删除平均时间复杂度为O(1)，元素无序。容器通用增强方面，emplace系列函数直接在容器内存中构造对象，减少一次拷贝/移动开销，提升性能。

我会梳理C++11容器及并发特性。容器方面，cbegin()/cend()提供const迭代器，shrink_to_fit()可释放多余容量。并发编程支持上，C++11引入std::thread创建管理线程，可绑定多种函数，需在对象销毁前调用join()或detach()；还提供std::mutex互斥量和std::lock_guard锁守卫，后者RAII风格，构造加锁、析构解锁，简单安全。

我会梳理C++11并发相关特性。std::unique_lock是更灵活的锁守卫，支持手动加锁等操作，常配合条件变量使用。条件变量std::condition_variable实现线程同步，避免轮询，需配合std::unique_lock，核心方法有wait()等。异步编程支持方面，std::future/promise实现线程间结果传递，std::async简化异步任务创建，返回future对象，可获取任务返回值。

我会阐述C++11函数对象与回调增强特性。先介绍std::function，它能统一包装各类可调用对象，实现类型擦除，简化回调实现，语法为std::function<返回值类型(参数类型列表)> ，并通过包装普通函数、lambda展示用法，典型场景有回调函数等。还会介绍std::bind，它可绑定可调用对象与参数，生成新可调用对象，支持参数重排等，用占位符表示调用时传入参数，通过示例展示绑定固定参数的用法。

我会通过示例阐述std::bind的用法，如创建add10函数对象，实现参数重排。还会介绍std::tuple元组，它是变长模板参数应用，可存储任意数量、类型元素，常用操作有创建、获取、解包，典型场景是函数返回多值。此外，介绍std::chrono时间库，它跨平台，有时长、时间点、时钟三部分，用于高精度时间计量。

我会介绍C++11的计时示例，通过获取起始和结束时间，计算耗时并输出。还会阐述正则表达式库，它实现字符串匹配等操作，核心类有std::regex等，示例展示了邮箱格式匹配。另外，介绍类型萃取库，它是元编程核心工具，可在编译期获取或修改类型属性，常用类型有类型判断和修改相关。

我会补充C++11的static_assert、random、ref等特性示例。static_assert用于编译期断言，如示例中确保模板参数为算术类型。random库替代rand()，示例展示用mt19937引擎和uniform_int_distribution分布器生成随机数。ref包装引用，解决模板传参引用退化问题，示例用于std::thread传递引用。最后总结C++11是里程碑，修复缺陷并引入现代特性，提升开发效率，奠定后续发展基础。

我需要将已总结的C++11特性内容进行组织，确保结构清晰、准确详细。同时，要严格审核，避免混入后续标准内容，保证每个特性都属于C++11。

我计划补充C++11遗漏特性，将内容分为核心语言特性与标准库新增及增强两部分。核心语言特性需补充类型别名、原始字符串字面量等8项；标准库部分补充std::atomic等4项。完善结构，修正遗漏，补充准确规则与示例。

我计划将异常处理及新增算法等内容，整合进已梳理的C++11新特性结构中。需确保结构清晰，每个特性讲清作用、语法等，且准确无误，不混入后续标准内容。

我会梳理auto与decltype关键字规则及示例。auto必须在声明时初始化，推导依赖初始值表达式，推导默认忽略顶层const/volatile和引用，不可用于函数形参等推导，示例展示了其用法。decltype用于精确推导表达式类型，不依赖变量初始化，核心用于泛型编程，推导规则为若表达式是标识符等，推导为该实体声明类型，若是左值表达式，推导为T&，纯右值推导为T，将亡值推导为T&& 。

通过示例阐述decltype，如decltype(x) a = 10推导为int，decltype((x)) b = x推导为int& ，decltype(x + 1) c = 20推导为int 。还介绍尾置返回类型，配合auto和decltype解决模板函数返回值依赖参数问题，语法为auto 函数名(参数列表) -> decltype(表达式) { 函数体 } 。另外，介绍using关键字，它扩展了类型别名能力，支持模板别名，比typedef更直观强大。

我会介绍using关键字，它扩展了类型别名能力，支持模板别名，如`template<typename T> using Vec = std::vector<T, std::allocator<T>>;`。还会阐述右值引用与移动语义，右值引用用`&&`标记，仅能绑定到右值，可延长其生命周期，允许修改右值，具名右值引用本身是左值。通过示例展示其用法，如`int&& rref = 10;`绑定纯右值10并可修改。

我会阐述类的移动构造与移动赋值函数，它们用于接管源对象资源，避免资源重复申请释放。核心规则为移动函数常声明`noexcept`，编译器仅在特定条件下自动生成默认移动函数。还给出`MyString`类的移动构造与赋值运算符示例，展示资源转移及源对象置空操作。

我会介绍std::move，它将左值转为右值引用以触发移动语义，自身不做移动操作，示例展示了其用法。还会阐述完美转发与std::forward，在模板函数中保持参数原始值类别，解决右值引用引用折叠问题，通过引用折叠规则、语法及示例说明其应用。

C++11引入匿名函数，支持函数式编程。其完整语法为`[捕获列表](参数列表) mutable 异常说明 -> 返回类型 { 函数体 }`。核心组成包括：捕获列表，定义闭包可访问变量及捕获方式，如`[]`空捕获、`[=]`值捕获等；参数列表与普通函数一致，C++11不支持auto参数（C++14泛型Lambda支持）；mutable，值捕获变量默认const，加此关键字可修改副本；返回类型，尾置返回类型，函数体仅含return语句或无返回值时，编译器可自动推导，可省略。

通过简单加法、值捕获加mutable及配合STL算法等Lambda示例，展示其用法。介绍Lambda核心特性，如生成唯一匿名闭包类型，无捕获Lambda可隐式转函数指针。还阐述统一初始化，用`{}`解决C++98初始化混乱问题，禁止窄化转换，有解决语法解析及禁止窄化转换等核心优势。

给出基本类型、数组、容器等多种初始化示例。介绍std::initializer_list，它让自定义类型支持列表初始化，是轻量级代理对象，示例展示其用法。还阐述类功能增强，=default显式要求编译器生成默认特殊成员函数，即便用户自定义构造函数，也能生成默认版本。

- **=delete**：显式禁用函数，替代C++98“私有声明不实现”方案，如示例中禁用拷贝构造和赋值，允许移动。

- **override**：显式声明重写基类虚函数，编译器检查基类匹配虚函数，避免笔误。

- **final**：修饰类不可继承，修饰虚函数不可被子类重写，示例展示了正确和错误使用override的情况。

我会阐述C++11的委托构造和继承构造函数。委托构造允许一个构造函数调用同一类其他构造函数，需注意不能同时初始化成员变量且不能形成委托循环，示例展示了其用法。继承构造函数让子类继承基类构造函数，简化代码，语法为`using 基类名::基类名;` ，示例也进行了说明。

我会阐述C++11的编译期增强特性，包括constexpr和static_assert。constexpr将变量、函数计算移至编译期，提升性能，替代旧方案，其变量需编译期常量初始化，函数体仅含return语句。static_assert是编译期断言，语法为`static_assert(常量表达式, "错误提示字符串")`，不满足条件编译即终止并输出错误信息，无运行时开销。

通过两个示例展示编译期检查，一是检查平台int大小，二是模板类型检查，static_assert用于确保条件为真，否则编译失败并输出提示。

强枚举有三核心特性：强作用域，枚举项需加作用域访问；禁止隐式类型转换，需显式`static_cast`；可指定底层类型，默认int，可自定义。示例展示了其用法。范围for循环简化容器/数组遍历，语法为`for (元素声明 : 可迭代对象) { 循环体 }`，示例演示了只读和修改元素的方式，核心规则是可迭代对象包括数组、STL容器及实现`begin()/end()`的自定义类型。

我会介绍C++11的原始字符串字面量、noexcept异常说明符等特性。原始字符串字面量解决字符串中转义符冗余问题，提升代码可读性，语法为`R"xxx(原始字符串内容)xxx"`，xxx为可选自定义分隔符，示例展示了普通字符串与原始字符串及多行字符串的不同。noexcept声明函数不抛异常，编译器可优化，配合移动语义保证异常安全，有两种语法，典型场景是移动构造/赋值函数常声明为noexcept。

我会介绍thread_local变量，其生命周期为线程级，各线程有独立副本，替代平台相关方案，示例展示了其用法。还会阐述其他核心语言特性，包括变长模板参数包，允许模板接收任意数量、类型参数，支撑std::tuple等实现；alignas/alignof用于获取和指定内存对齐规则；explicit转换操作符禁止意外隐式类型转换；重新定义POD类型为平凡和标准布局类型，保证与C语言内存兼容性；首次在语言层面定义多线程内存模型，为无锁编程等提供标准语义基础。

我会介绍用户自定义字面量，允许自定义字面量后缀，提升代码可读性。还会阐述智能指针，基于RAII机制自动管理堆内存，解决内存相关问题，替代auto_ptr。其中std::unique_ptr独占对象所有权，不支持拷贝仅支持移动，开销小；std::shared_ptr基于原子引用计数，多个可指向同一对象，引用计数为0时释放对象。

我会推荐使用`std::make_shared`创建智能指针，因其内存分配更高效安全。同时提醒，循环引用会致内存泄漏，需配合`weak_ptr`解决。还给出`std::shared_ptr`使用示例，展示引用计数变化。对于`std::weak_ptr`，阐述其辅助`shared_ptr`、不增计数、用于观察对象状态及解决循环引用的特性，给出解决循环引用的类定义示例。

给出解决循环引用的示例代码，通过智能指针实现，离开作用域时引用计数归零，对象正确析构。还介绍容器新增与增强特性，新增std::array等容器，emplace系列函数可直接在容器内存构造对象，提升性能。

我会梳理C++11容器与并发编程特性。容器方面，`emplace_back`直接在内存构造对象，无临时对象；新增`cbegin()`等获取const迭代器，`shrink_to_fit()`释放多余容量，所有容器支持`{}`列表初始化。并发编程上，C++11首次在标准层面实现跨平台多线程支持，`std::thread`用于创建管理线程，可绑定多种函数，核心规则是对象销毁前需调用`join()`或`detach()`，示例展示了创建线程执行函数的用法。

我会先介绍线程等待执行完成的方法`t.join()`。接着阐述互斥量相关类，如`std::mutex`等，其中`std::lock_guard`是RAII风格锁守卫，`std::unique_lock`更灵活。还会介绍条件变量`std::condition_variable`，它实现线程同步，需配合`std::unique_lock`使用，有`wait()`等核心方法。

我会梳理C++11异步编程支持，<future>头文件中的std::promise/std::future实现线程间结果传递，std::async简化异步任务创建。还会介绍<atomic>原子操作库，提供线程安全的无锁原子操作。此外，阐述<functional>头文件中std::function函数包装器，统一包装可调用对象，实现类型擦除。

std::function是函数包装器，语法为`std::function<返回值类型(参数类型列表)>`，可包装普通函数、lambda等，示例展示了其用法。std::bind是函数绑定器，可绑定可调用对象与参数，生成新可调用对象，支持参数重排、固定参数等，占位符`std::placeholders::_1/_2/...`表示调用时传入参数，示例也进行了说明。

介绍swap_add函数，其效果等价于add(20,10)。接着介绍std::ref/std::cref，将引用包装为可拷贝对象，解决模板传参引用退化问题，常用于std::bind等。还介绍std::tuple，固定大小异构集合，可存储任意数量任意类型元素，支持创建、获取、解包，常用于函数返回多值。最后介绍std::chrono，跨平台高精度时间处理库，分时长、时间点、时钟三部分，替代平台相关时间API。

我会介绍C++11新增的正则表达式库、类型萃取库、随机数库，以及全局迭代器获取函数和新增STL算法，如`std::all_of`等。同时，阐述C++11让C++从“带类的C”转变为现代多范式语言，其移动语义等特性大幅提升性能。

# C++11 新特性详细总结

C++11（ISO/IEC 14882:2011）是C++发展史上里程碑式的标准，被称为**现代C++的开端**。它修复了C++98的诸多缺陷，引入了超过140项新特性，让代码更简洁、更安全、性能更高，同时支持多范式编程，大幅提升了开发效率，也为后续C++14/17/20标准奠定了基础。

整体分为**核心语言特性**和**标准库新增与增强**两大模块，以下是详细梳理：

---

## 一、核心语言特性

### 1. 类型推导体系

#### 1.1 auto 关键字

C++11彻底重构了auto的语义（C++98中auto为存储类型说明符，几乎无实际用途），实现**编译期自动类型推导**，大幅简化复杂类型的声明。

- **核心规则**：

    1. 必须在声明时初始化，推导依赖初始值表达式；

    2. 推导默认忽略顶层`const/volatile`和引用，如需保留需显式声明；

    3. 不可用于函数形参类型推导（C++14才支持）、非静态成员变量的类内初始化、数组类型推导（除非声明为引用）。

- **示例**：

```C++

auto i = 42;                  // int
auto p = &i;                  // int*
const auto& ri = i;           // const int&，保留const和引用
std::vector<int> vec;
auto it = vec.begin();        // std::vector<int>::iterator，简化冗长迭代器声明
```

#### 1.2 decltype 关键字

用于**精确推导表达式的类型**（包括值类别、const/volatile、引用属性），不依赖变量初始化，核心用于泛型编程。

- **核心推导规则**：

    1. 若表达式是标识符/类成员访问，直接推导为该实体的声明类型；

    2. 若表达式是左值表达式，推导为`T&`；若为纯右值，推导为`T`；若为将亡值，推导为`T&&`。

- **示例**：

```C++

int x = 0;
decltype(x) a = 10;          // int，标识符直接匹配类型
decltype((x)) b = x;          // int&，(x)是左值表达式，推导为左值引用
decltype(x+1) c = 20;         // int，x+1是纯右值，推导为值类型
```

#### 1.3 尾置返回类型（追踪返回类型）

配合`auto`和`decltype`解决模板函数返回值类型依赖参数的问题，让返回值类型声明在参数列表之后，可直接使用函数参数。

- **语法**：`auto 函数名(参数列表) -> decltype(表达式) { 函数体 }`

- **示例**：

```C++

// 模板函数返回值类型依赖参数t和u的运算结果
template<typename T, typename U>
auto add(T t, U u) -> decltype(t + u) {
    return t + u;
}
```

#### 1.4 类型别名与模板别名（using 关键字）

C++11扩展了`using`的用法，提供比`typedef`更直观、更强大的类型别名能力，**支持模板别名**（typedef无法实现）。

- **示例**：

```C++

// 普通类型别名，等价于typedef unsigned int uint;
using uint = unsigned int;

// 模板别名，typedef无法实现
template<typename T>
using Vec = std::vector<T, std::allocator<T>>;
Vec<int> vec; // 等价于std::vector<int, std::allocator<int>> vec;
```

### 2. 右值引用与移动语义

这是C++11最核心的性能优化特性，通过**转移资源所有权**替代深拷贝，彻底解决临时对象带来的不必要性能开销。

#### 2.1 左值与右值

- **左值**：可以取地址、拥有持久生命周期的对象，如变量、解引用的指针；

- **右值**：临时的、无法取地址的对象，分为纯右值（字面量、表达式运算结果）和将亡值（生命周期即将结束的对象，如函数返回的临时对象）。

#### 2.2 右值引用

用`&&`标记，仅能绑定到右值，可延长右值的生命周期，允许修改右值，为移动语义提供语法基础。

- **核心特性**：具名的右值引用本身是左值（可被取地址）。

- **示例**：

```C++

int&& rref = 10; // 绑定到纯右值10，延长其生命周期
rref = 20;        // 可修改右值
int a = 20;
// int&& rref2 = a; // 编译错误，无法将左值绑定到右值引用
```

#### 2.3 移动构造与移动赋值

类的特殊成员函数，用于接管源对象的资源，而非拷贝，源对象最终处于**有效但未定义**的状态，避免资源重复申请与释放。

- **核心规则**：

    1. 移动函数通常声明为`noexcept`，保证异常安全，让容器优先选择移动而非拷贝；

    2. 编译器仅在用户未自定义拷贝构造/赋值、析构函数时，自动生成默认移动函数。

- **示例**：

```C++

class MyString {
public:
    char* data;
    size_t len;

    // 移动构造函数
    MyString(MyString&& other) noexcept : data(other.data), len(other.len) {
        other.data = nullptr; // 源对象置空，避免析构时释放资源
        other.len = 0;
    }

    // 移动赋值运算符
    MyString& operator=(MyString&& other) noexcept {
        if (this != &other) {
            delete[] data; // 释放当前对象资源
            // 转移资源所有权
            data = other.data;
            len = other.len;
            other.data = nullptr;
            other.len = 0;
        }
        return *this;
    }

    ~MyString() { delete[] data; }
};
```

#### 2.4 std::move 类型转换

无条件将左值转换为右值引用，使其可触发移动语义。**注意：std::move本身不做任何移动操作，仅做类型转换**。

- **示例**：

```C++

MyString a("hello");
MyString b = std::move(a); // 触发移动构造，a的资源被转移到b
// 移动后，a除了析构和赋值，不建议进行其他操作
```

#### 2.5 完美转发与std::forward

在模板函数中保持参数的原始值类别（左值/右值、const/volatile）不变，转发给其他函数，解决右值引用在模板中的引用折叠问题。

- **引用折叠规则**：模板中`T&&`作为万能引用时，传入左值则`T`推导为`T&`，最终类型为`T& && → T&`；传入右值则`T`推导为`T`，最终类型为`T&&`。

- **语法**：`std::forward<T>(参数)`

- **示例**：

```C++

// 工厂函数，完美转发参数到构造函数
template<typename T, typename... Args>
T create(Args&&... args) {
    return T(std::forward<Args>(args)...); // 保持参数原始值类别
}
```

### 3. Lambda 表达式

C++11首次引入匿名函数（闭包），支持函数式编程，简化回调函数、STL算法的参数传递，可捕获上下文变量。

#### 3.1 完整语法

```C++

[捕获列表](sslocal://flow/file_open?url=%E5%8F%82%E6%95%B0%E5%88%97%E8%A1%A8&flow_extra=eyJsaW5rX3R5cGUiOiJjb2RlX2ludGVycHJldGVyIn0=) mutable 异常说明 -> 返回类型 { 函数体 }
```

#### 3.2 核心组成

1. **捕获列表**：定义闭包可访问的外部变量及捕获方式

    - `[]`：空捕获，不访问任何外部变量；

    - `[=]`：值捕获，拷贝所有用到的外部变量，默认不可修改；

    - `[&]`：引用捕获，按引用访问所有用到的外部变量，需注意生命周期；

    - `[a, &b]`：混合捕获，a值捕获，b引用捕获；

    - `[this]`：捕获this指针，访问类的成员。

2. **参数列表**：与普通函数一致，C++11不支持auto作为参数类型（C++14泛型Lambda支持）。

3. **mutable**：值捕获的变量默认是const的，加mutable可修改拷贝的副本。

4. **返回类型**：尾置返回类型，若函数体仅含return语句或无返回值，编译器可自动推导，可省略。

#### 3.3 示例

```C++

// 简单加法Lambda
auto add = [](sslocal://flow/file_open?url=int+a%2C+int+b&flow_extra=eyJsaW5rX3R5cGUiOiJjb2RlX2ludGVycHJldGVyIn0=) { return a + b; };
int res = add(1, 2); // 3

// 值捕获+mutable
int x = 10;
auto modify =  mutable { x++; return x; };
modify(); // 11，修改的是副本，原x仍为10

// 配合STL算法
std::vector<int> vec = {1,2,3,4,5};
std::for_each(vec.begin(), vec.end(), [](sslocal://flow/file_open?url=int%26+n&flow_extra=eyJsaW5rX3R5cGUiOiJjb2RlX2ludGVycHJldGVyIn0=){ n *= 2; });
```

#### 3.4 核心特性

- 每个Lambda表达式会生成唯一的匿名闭包类型，运行时创建该类型的实例；

- 无捕获的Lambda可隐式转换为普通函数指针。

### 4. 统一初始化与初始化列表

#### 4.1 统一初始化（列表初始化）

用`{}`统一所有类型的初始化方式，解决C++98初始化方式混乱的问题，同时禁止窄化转换。

- **核心优势**：

    1. 解决C++"最令人头疼的语法解析"：`MyClass obj();`会被解析为函数声明，`MyClass obj{};`可正确调用默认构造；

    2. 禁止窄化转换：`int a{3.14};`会编译报错，而`int a(3.14)`只会静默截断。

- **示例**：

```C++

int a{10};                         // 基本类型
int arr[]{1,2,3,4};                // 数组
std::vector<int> vec{1,2,3,4};     // 容器
std::map<int, std::string> map{{1, "one"}, {2, "two"}}; // 关联容器
struct Point { int x; int y; };
Point p{10, 20};                    // 自定义结构体
```

#### 4.2 std::initializer_list

让自定义类型支持列表初始化，可接收任意长度的同类型初始化列表。本质是轻量级代理对象，仅引用数组元素，不持有数据。

- **示例**：

```C++

class MyVector {
public:
    std::vector<int> data;
    MyVector(std::initializer_list<int> list) {
        for (auto it = list.begin(); it != list.end(); ++it) {
            data.push_back(*it);
        }
    }
};
MyVector vec = {1,2,3,4,5}; // 支持列表初始化
```

### 5. 类功能全面增强

#### 5.1 =default 与 =delete

- **=default**：显式要求编译器生成默认的特殊成员函数（默认构造、拷贝/移动构造、析构、拷贝/移动赋值），即使用户自定义了其他构造函数，也能生成默认版本。

- **=delete**：显式禁用函数，禁止编译器生成或用户调用，替代C++98"私有声明不实现"的方案。

- **示例**：

```C++

class NonCopyable {
public:
    NonCopyable() = default;
    // 禁用拷贝构造和赋值
    NonCopyable(const NonCopyable&) = delete;
    NonCopyable& operator=(const NonCopyable&) = delete;
    // 允许移动
    NonCopyable(NonCopyable&&) = default;
    NonCopyable& operator=(NonCopyable&&) = default;
};

// 禁用特定重载，防止隐式转换
void func(int a) {}
void func(double) = delete;
func(3.14); // 编译报错
```

#### 5.2 override 与 final

- **override**：显式声明该函数重写基类虚函数，编译器会强制检查基类是否存在签名完全匹配的虚函数，避免笔误导致的函数隐藏。

- **final**：两个核心作用

    1. 修饰类：该类不可被继承；

    2. 修饰虚函数：该虚函数不可被子类重写。

- **示例**：

```C++

class Base {
public:
    virtual void show(int a) const {}
    virtual void func() final {}
};

class Derived : public Base {
public:
    void show(int a) const override {} // 正确，匹配基类虚函数
    // void show(double a) const override {} // 编译报错，签名不匹配
    // void func() override {} // 编译报错，基类func被final修饰
};

class FinalClass final {};
// class Derived2 : public FinalClass {}; // 编译报错，类不可被继承
```

#### 5.3 委托构造函数

允许一个构造函数调用同一个类中的其他构造函数，减少代码冗余，简化初始化逻辑。

- **示例**：

```C++

class Person {
public:
    std::string name;
    int age;
    // 全参数构造
    Person(std::string n, int a) : name(n), age(a) {}
    // 委托构造
    Person() : Person("", 0) {}
    Person(std::string n) : Person(n, 0) {}
};
```

- **注意**：委托构造不能在初始化列表中同时初始化成员变量，且不能形成委托循环。

#### 5.4 继承构造函数

允许子类继承基类的构造函数，无需手动转发参数，简化子类构造函数编写。

- **语法**：`using 基类名::基类名;`

- **示例**：

```C++

class Base {
public:
    Base(int a) : num(a) {}
    Base(std::string s, int a) : str(s), num(a) {}
private:
    std::string str;
    int num;
};

class Derived : public Base {
public:
    using Base::Base; // 继承基类所有构造函数
};

Derived d1(10); // 调用Base(int)
Derived d2("hello", 20); // 调用Base(std::string, int)
```

### 6. 编译期增强特性

#### 6.1 constexpr 编译期常量与计算

将变量、函数的计算放到编译期，提升运行时性能，替代C++98中宏和enum的编译期常量方案，类型更安全。

- **核心规则**：

    1. constexpr变量必须用编译期常量初始化，本身也是编译期常量；

    2. C++11中constexpr函数只能包含一条return语句，传入编译期常量时，在编译期计算结果。

- **示例**：

```C++

// 编译期常量
constexpr int arr_size = 10;
int arr[arr_size]; // 合法，编译期确定数组大小

// 编译期函数
constexpr int factorial(int n) {
    return n == 0 ? 1 : n * factorial(n-1);
}
constexpr int res = factorial(5); // 编译期计算出120
```

#### 6.2 static_assert 静态断言

编译期断言，在编译阶段检查条件，不满足则直接终止编译并输出错误信息，无任何运行时开销。

- **语法**：`static_assert(常量表达式, "错误提示字符串");`

- **示例**：

```C++

// 检查平台int大小
static_assert(sizeof(int) == 4, "int must be 4 bytes");

// 模板类型检查
template<typename T>
void func(T t) {
    static_assert(std::is_integral<T>::value, "T must be integer type");
}
```

### 7. 核心语法与关键字增强

#### 7.1 nullptr 空指针关键字

替代C++98的NULL宏，彻底解决NULL的二义性问题（NULL被定义为0，函数重载时会匹配int而非指针）。

- **核心特性**：

    1. nullptr是关键字，类型为`std::nullptr_t`，是空指针常量；

    2. 可隐式转换为任意指针/成员指针类型，无法隐式转换为整数类型（除了bool）。

- **示例**：

```C++

void func(int) {}
void func(int*) {}

func(NULL);    // 调用func(int)，不符合预期
func(nullptr); // 正确调用func(int*)
int* p = nullptr;
bool b = nullptr; // 转换为false，合法
// int a = nullptr; // 编译错误
```

#### 7.2 强类型枚举 enum class / enum struct

解决C++98枚举的三大缺陷：作用域污染、隐式类型转换、底层类型不固定。

- **核心特性**：

    1. 强作用域：枚举项必须通过`枚举名::枚举项`访问，不污染外层作用域；

    2. 禁止隐式类型转换：无法隐式转换为整数，必须显式`static_cast`；

    3. 可指定底层类型：默认是int，可自定义为char、long等整数类型。

- **示例**：

```C++

enum class Color : char {
    Red,
    Green,
    Blue
};

Color c = Color::Red; // 必须加作用域
// int a = Color::Green; // 编译错误，禁止隐式转换
int b = static_cast<int>(Color::Green); // 正确，显式转换
```

#### 7.3 范围for循环（Range-based for loop）

简化容器/数组的遍历语法，无需手动管理迭代器，代码更简洁安全。

- **语法**：`for (元素声明 : 可迭代对象) { 循环体 }`

- **示例**：

```C++

std::vector<int> vec = {1,2,3,4,5};
// 只读遍历
for (int n : vec) {
    std::cout << n << " ";
}
// 修改元素，必须用引用
for (int& n : vec) {
    n *= 2;
}
```

- **核心规则**：可迭代对象包括数组、STL容器、实现了`begin()/end()`的自定义类型。

#### 7.4 原始字符串字面量

解决字符串中转义符冗余的问题，无需对反斜杠、引号进行转义，提升代码可读性。

- **语法**：`R"xxx(原始字符串内容)xxx"`，其中xxx是可选的自定义分隔符。

- **示例**：

```C++

// 普通字符串需要双重转义
std::string regex_str = "\\d+\\.\\d*";
// 原始字符串无需转义
std::string raw_regex = R"(\d+\.\d*)";

// 多行字符串
std::string multi_line = R"(
    <html>
        <body>
            <p>hello world</p>
        </body>
    </html>
)";
```

#### 7.5 noexcept 异常说明符

声明函数不会抛出异常，编译器可进行针对性优化，同时配合移动语义保证异常安全。

- **语法**：

    1. `void func() noexcept;` 声明函数不会抛出异常；

    2. `void func() noexcept(常量表达式);` 表达式为true时声明不抛异常。

- **典型场景**：移动构造/移动赋值函数通常声明为noexcept，容器扩容时会优先选择noexcept的移动构造，否则会使用拷贝构造保证异常安全。

#### 7.6 thread_local 线程本地存储

声明变量的生命周期是线程级的，每个线程拥有独立的变量副本，线程启动时创建，结束时销毁，替代平台相关的线程本地存储方案。

- **示例**：

```C++

thread_local int count = 0; // 每个线程有独立的count副本
void func() {
    count++;
    std::cout << std::this_thread::get_id() << ": " << count << std::endl;
}
```

### 8. 其他核心语言特性

1. **变长模板参数包**：允许模板接收任意数量、任意类型的参数，是C++元编程的核心，支撑了std::tuple、emplace系列函数的实现；

2. **对齐支持alignas/alignof**：alignof获取类型的内存对齐字节数，alignas指定变量/类型的对齐规则；

3. **显式转换操作符**：允许将类型转换操作符声明为explicit，禁止意外的隐式类型转换；

4. **标准布局与平凡类型**：重新定义了POD类型，分为平凡类型和标准布局类型，保证与C语言的内存兼容性；

5. **多线程内存模型**：首次在语言层面定义了多线程内存模型，为无锁编程、原子操作提供了标准语义基础；

6. **用户自定义字面量**：允许自定义字面量后缀，扩展字面量的能力，提升代码可读性。

---

## 二、标准库新增与增强

### 1. 智能指针（<memory>头文件）

基于RAII机制自动管理堆内存，彻底解决内存泄漏、野指针问题，替代C++98有缺陷的auto_ptr。

#### 1.1 std::unique_ptr 独占式智能指针

- **核心特性**：独占对象所有权，同一时间仅能有一个unique_ptr指向同一个对象，不支持拷贝，仅支持移动语义，离开作用域时自动释放对象。

- **性能**：开销极小，与原始指针几乎无差别，是C++11最常用的智能指针。

- **示例**：

```C++

std::unique_ptr<int> p1(new int(10));
// std::unique_ptr<int> p2 = p1; // 编译错误，禁止拷贝
std::unique_ptr<int> p2 = std::move(p1); // 转移所有权，p1变为空
p2.reset(); // 手动释放对象
```

#### 1.2 std::shared_ptr 共享式智能指针

- **核心特性**：基于线程安全的原子引用计数，多个shared_ptr可指向同一个对象，引用计数为0时自动释放对象。

- **推荐用法**：使用`std::make_shared`创建，内存分配更高效、更安全。

- **注意**：循环引用会导致内存泄漏，需配合weak_ptr解决。

- **示例**：

```C++

auto p1 = std::make_shared<int>(10); // 推荐用法
std::cout << p1.use_count() << std::endl; // 1
std::shared_ptr<int> p2 = p1; // 引用计数+1
std::cout << p1.use_count() << std::endl; // 2
p2.reset(); // 引用计数-1，变为1
```

#### 1.3 std::weak_ptr 弱引用智能指针

- **核心特性**：辅助shared_ptr，不增加引用计数，不控制对象生命周期，用于观察对象状态、解决循环引用问题。

- **核心方法**：`expired()`检查对象是否已释放，`lock()`获取对应的shared_ptr。

- **示例**：

```C++

// 解决循环引用
class B;
class A {
public:
    std::weak_ptr<B> b_ptr; // 弱引用，不增加计数
    ~A() { std::cout << "A destructed" << std::endl; }
};
class B {
public:
    std::shared_ptr<A> a_ptr;
    ~B() { std::cout << "B destructed" << std::endl; }
};

auto a = std::make_shared<A>();
auto b = std::make_shared<B>();
a->b_ptr = b;
b->a_ptr = a;
// 离开作用域时，引用计数正常归零，对象正确析构
```

### 2. 容器新增与增强

#### 2.1 新增容器

|容器|核心特性|
|---|---|
|std::array<T, N>|固定大小数组，封装原生数组，提供STL容器接口，无堆开销，性能与原生数组一致，更安全|
|std::forward_list<T>|单向链表，比std::list更省内存，仅支持单向遍历，插入删除效率高|
|std::unordered_map/unordered_multimap|基于哈希表的关联容器，平均查找/插入/删除复杂度O(1)，元素无序|
|std::unordered_set/unordered_multiset|基于哈希表的集合，平均查找/插入/删除复杂度O(1)，元素无序|
#### 2.2 容器通用增强

1. **emplace系列函数**：`emplace_back`、`emplace_front`、`emplace`等，直接在容器内存中构造对象，避免临时对象的拷贝/移动，大幅提升性能。

    ```C++

    std::vector<std::string> vec;
    vec.emplace_back("hello world"); // 直接在容器内存中构造string，无临时对象
    ```

2. **const迭代器**：新增`cbegin()/cend()/crbegin()/crend()`，直接获取const迭代器，类型更安全；

3. **shrink_to_fit()**：释放容器多余的容量，减少内存占用，适用于vector、string、deque；

4. **列表初始化支持**：所有容器均支持`{}`列表初始化。

### 3. 并发编程标准库（首次加入标准）

C++11首次在标准层面实现了跨平台的多线程支持，替代平台相关的线程API。

#### 3.1 std::thread 线程类

用于创建和管理线程，可绑定普通函数、lambda、类成员函数、函数对象。

- **核心规则**：线程对象销毁前，必须调用`join()`（等待线程结束）或`detach()`（分离线程，后台执行），否则会触发程序终止。

- **示例**：

```C++

void func(int a) {
    std::cout << "thread: " << a << std::endl;
}

int main() {
    std::thread t(func, 10); // 创建线程，执行func
    t.join(); // 等待线程执行完成
    return 0;
}
```

#### 3.2 互斥量与锁（<mutex>头文件）

|类|核心特性|
|---|---|
|std::mutex|基础互斥锁，保护共享资源，避免多线程数据竞争|
|std::recursive_mutex|递归互斥锁，允许同一线程多次加锁|
|std::timed_mutex|带超时的互斥锁，支持超时等待加锁|
|std::lock_guard|RAII风格的锁守卫，构造时加锁，析构时自动解锁，无法手动控制，最简单安全|
|std::unique_lock|灵活的锁守卫，支持手动加锁/解锁、延迟加锁、超时锁、所有权转移，配合条件变量使用|
- **示例**：

```C++

std::mutex mtx;
int count = 0;

void add() {
    std::lock_guard<std::mutex> lock(mtx); // 构造加锁，离开作用域自动解锁
    count++;
}
```

#### 3.3 条件变量 std::condition_variable

实现线程间的同步，一个线程等待条件满足，另一个线程通知条件，避免轮询等待，是生产者消费者模型的核心同步机制。

- **必须配合std::unique_lock使用**，核心方法：`wait()`、`notify_one()`、`notify_all()`。

#### 3.4 异步编程支持（<future>头文件）

- **std::promise/std::future**：实现线程间的结果传递，promise用于设置结果，future用于获取结果；

- **std::async**：简化异步任务创建，自动启动线程执行任务，返回future对象，可获取任务返回值。

- **示例**：

```C++

// 异步执行任务
std::future<int> fut = std::async(std::launch::async, [](sslocal://flow/file_open?url=int+a%2C+int+b&flow_extra=eyJsaW5rX3R5cGUiOiJjb2RlX2ludGVycHJldGVyIn0=) {
    return a + b;
}, 10, 20);

int res = fut.get(); // 阻塞等待结果，返回30
```

#### 3.5 原子操作库 <atomic>

提供线程安全的无锁原子操作，支持整数、指针等类型的原子读写、加减、交换、比较交换等操作，是无锁编程的基础。

- **示例**：

```C++

std::atomic<int> count(0); // 原子计数器
void add() {
    count++; // 原子自增，无需加锁，线程安全
}
```

### 4. 函数式编程增强

#### 4.1 std::function 函数包装器（<functional>头文件）

统一包装任意可调用对象（普通函数、lambda、函数对象、类成员函数），实现类型擦除，定义统一的函数类型，是回调函数、策略模式的核心工具。

- **语法**：`std::function<返回值类型(参数类型列表)>`

- **示例**：

```C++

int add(int a, int b) { return a + b; }
// 包装普通函数
std::function<int(int, int)> func = add;
func(1, 2); // 3

// 包装lambda
func = [](sslocal://flow/file_open?url=int+a%2C+int+b&flow_extra=eyJsaW5rX3R5cGUiOiJjb2RlX2ludGVycHJldGVyIn0=) { return a * b; };
func(3, 4); // 12
```

#### 4.2 std::bind 函数绑定器（<functional>头文件）

绑定可调用对象与参数，生成新的可调用对象，支持参数重排、固定参数、占位符，替代C++98的bind1st/bind2nd。

- **占位符**：`std::placeholders::_1/_2/...` 表示调用时传入的第1/2...个参数。

- **示例**：

```C++

int add(int a, int b) { return a + b; }
// 固定第一个参数为10，生成单参数函数
auto add10 = std::bind(add, 10, std::placeholders::_1);
add10(20); // 30，等价于add(10,20)

// 参数重排
auto swap_add = std::bind(add, std::placeholders::_2, std::placeholders::_1);
swap_add(10, 20); // 30，等价于add(20,10)
```

#### 4.3 std::ref/std::cref 引用包装器

将引用包装为可拷贝的对象，解决模板传参时引用被退化的问题，常用于std::bind、std::thread、std::function中传递引用。

- **示例**：

```C++

int x = 10;
auto func = [](sslocal://flow/file_open?url=int%26+a&flow_extra=eyJsaW5rX3R5cGUiOiJjb2RlX2ludGVycHJldGVyIn0=) { a++; };
std::thread t(func, std::ref(x)); // 传递引用，否则x会被拷贝
t.join();
```

### 5. 其他重要标准库特性

1. **std::tuple 元组**：固定大小的异构集合，可存储任意数量、任意类型的元素，支持`std::make_tuple`创建、`std::get`获取元素、`std::tie`解包，常用于函数返回多个值。

    ```C++

    std::tuple<int, std::string, double> t = std::make_tuple(10, "hello", 3.14);
    int a = std::get<0>(t); // 10
    // 解包
    int x; std::string y; double z;
    std::tie(x, y, z) = t;
    ```

2. **std::chrono 时间库**：跨平台的高精度时间处理库，分为时长duration、时间点time_point、时钟clock三部分，用于计时、时间戳处理，替代平台相关的时间API。

3. **std::regex 正则表达式库**：C++标准首次加入正则支持，实现字符串的匹配、搜索、替换、分割等操作。

4. **<type_traits> 类型萃取库**：编译期获取/修改类型属性，是C++元编程的核心工具，配合static_assert、SFINAE使用。

5. **<random> 随机数库**：替代C++98的rand()函数，提供高质量的随机数引擎和多种分布器（均匀分布、正态分布等）。

6. **全局std::begin/std::end**：通用的迭代器获取函数，支持数组和所有STL容器，提升代码通用性。

7. **新增STL算法**：`std::all_of`、`std::any_of`、`std::none_of`、`std::copy_if`、`std::find_if_not`等。

---

## 三、C++11的核心意义

C++11彻底重构了C++的编程范式，让C++从"带类的C"转变为真正的现代多范式语言。它的核心价值体现在三个方面：

1. **性能飞跃**：移动语义、完美转发、emplace系列函数等特性，大幅减少了不必要的拷贝，让C++的性能优势进一步放大；

2. **开发效率提升**：auto、Lambda、范围for、智能指针等特性，简化了代码编写，降低了开发门槛，减少了样板代码；

3. **安全性与可维护性增强**：nullptr、强类型枚举、override/final、static_assert等特性，在编译期捕获更多错误，减少了运行时bug，让代码更健壮。

C++11是现代C++的起点，后续的C++14/17/20标准均是在C++11的基础上进行的扩展和优化，掌握C++11是掌握现代C++的核心基础。
> （注：文档部分内容可能由 AI 生成）
