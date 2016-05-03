---
title: 简单测试 static 和 const
---
#  const

 三种使用 const 的方式

	const NSMutableString * constString = @"constString".mutableCopy;
    NSMutableString const * stringConst = @"stringConst".mutableCopy;
    NSMutableString * const string_const = @"string_const".mutableCopy;
    
 给它们赋值，也就是改变指针 地址

    constString = @"newConstString".mutableCopy; //const 不管在 NSMutableString 前后是一样的东西
    stringConst = @"newStringConst".mutableCopy; 
    //string_const = @"newString_const".mutableCopy; //报错 const 和 指针在一起时指针不可以指向新的地址,也就是常量指针
    
这里没有预料中的 直接报红 也没报黄

    [constString stringByAppendingString:@"_new"];
    [stringConst stringByAppendingString:@"_new"];
    //但是打印出来的值 是:  ---constString = newConstString -- stringConst = newStringCon 值没有被改变,也就是 值是常量
    NSLog(@" ---constString = %@ -- stringConst = %@",constString,stringConst);

    //总结：const 在 NSMutableString 旁边，那么就是指针指向的值是常量，改变值不会报错，但是不会有效果。
    //     const 在 指针            旁边，那么改指针就是常量指针，不可以重新赋值。

    //如果 要 常量指针  指向 常量 那么 ：
    const NSMutableString * const const__constString = @"const__constString".mutableCopy;
    //或者
    NSMutableString const  * const const_constString = @"const_constString".mutableCopy;



# static



    //static 在 创建的时候 不能赋值给 一个 不是编译的常数
    //只可以 赋值 这样的
    static int a = 1;
    static char b = '6';
    static char c[3] = "122";
    static char * d = "233";
    static NSString * e = @"233";

    static const NSString * static_constString_new = @"23333";//这样 也可以 我已经晕了

	//    static ViewController * mySelf = [[ViewController alloc] init]; // 这样就不行了
	//    static const ViewController * mySelf = [[ViewController alloc] init]; //nonono

	//    static const NSMutableString * static_constString = @"2333".mutableCopy; //这样是不行的
	//    static const NSString * static_constString_new_new = [NSString new]; //这样也不行

    //总之 好像 就能赋值给 一个 常量
关于  static 的一点 ，它在 app 的生命周期 只执行一次，下一次如果跑到了 这里，那么这句 初始化的 代码就被直接跳过了


    //先 测试 const 和 NSMutableString 在一起 指针指向 常量
    static const NSMutableString * static_constString = nil;
    const static NSMutableString * const_staticString = nil;
    const NSMutableString static * const__staticString = nil;  //这三种方式 猜测是一样的
	//    const NSMutableString * static const_string_static = nil; 这样干爆红了

    static_constString = @"static_constString".mutableCopy;
    const_staticString = @"const_staticString".mutableCopy;
    const__staticString = @"const__staticString".mutableCopy;

    // 打印 结果
	//    static_constString = static_constString
	//    const_staticString = const_staticString
	//    const__staticString = const__staticString

    NSLog(@"static_constString = %@ \n const_staticString = %@ \n const__staticString = %@",static_constString,const_staticString,const__staticString);

    static_constString = @"static_constString_new".mutableCopy;
    const_staticString = @"const_staticString_new".mutableCopy;
    const__staticString = @"const__staticString_new".mutableCopy;

    // 打印 结果
	//    static_constString = static_constString_new
	//    const_staticString = const_staticString_new
	//    const__staticString = const__staticString_new
    NSLog(@"static_constString = %@ \n const_staticString = %@ \n const__staticString = %@",static_constString,const_staticString,const__staticString);

    [static_constString stringByAppendingString:@"++ new"];
    [const_staticString stringByAppendingString:@"++ new"];
    [const__staticString stringByAppendingString:@"++ new"];

    // 打印 结果
	//    static_constString = static_constString_new
	//    const_staticString = const_staticString_new
	//    const__staticString = const__staticString_new
    NSLog(@"static_constString = %@ \n const_staticString = %@ \n const__staticString = %@",static_constString,const_staticString,const__staticString);

结论： 加了 static 对 const 和 NSMutableString 在一起的，指针指向常量这种东西 没影响，依然可以正常赋值，并且不能改变值，但是 用了 static 后，这个指针会被 存入 静态区


    NSMutableString static * const static_xing_constString = nil;
    static NSMutableString * const static__constString = nil;

    //预料之中的 警告了
	//    static_xing_constString = @"static_xing_constString".mutableCopy;
	//    static__constString = @"static__constString".mutableCopy;

    [static_xing_constString stringByAppendingString:@"static_xing_constString"];
    [static__constString stringByAppendingString:@"static_xing_constString"];

    //因为是 nil 所以。。。
    NSLog(@"static_xing_constString = %@ \n static__constString = %@",static_xing_constString,static__constString);
    //这样 写 好像 没用了， 初始化的时候 不能赋值，完了 还不能改,一辈子 都是 nil