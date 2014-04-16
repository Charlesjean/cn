---
layout: post
title: "深入ObjectiveC Runtime之(四)类的加载与Category的实现"
date: 2014-04-16 21:41:24 +0800
comments: true
categories: ObjectiveC 
---
源代码经过编译、连接转化为机器码，在运行时又由runtime加载入内存，因此编译器、连接器和runtime在某种程度上说是协同工作的。在编译过程中，编译器会首先将源代码中的每一个类转换为汇编代码，每一个类的不同的变量，方法等都会被放在汇编代码的不同段当中，然后再将汇编代码转化为机器语言即二进制。为了确保runtime能够对程序进行解析加载，编译器和runtime之间必须有某种约定，以实现runtime对类信息的加载。
例如对于如下的代码：
{% codeblock lang:objc %}
#import <objc/Object.h>

@interface MySuperClass: Object {

}
-(void) myMessage1;
@end

@implementation MySuperClass
-(void) myMessage1 {

}
@end

@interface MyClass: MySuperClass {

}
-(void) myMessage2;
@end

@implementation MyClass
-(void) myMessage2 {

}
@end

int main() {

    return 0;
}
{% endcodeblock %}
编译成的汇编代码的一部分(无须细读汇编代码)：
{% codeblock lang:objc %}
  .file   "test.m"

    .type   L_OBJC_CLASS_NAME_,@object # @"\01L_OBJC_CLASS_NAME_"
    .section    "__TEXT,__objc_classname,cstring_literals","aw",@progbits
L_OBJC_CLASS_NAME_:
    .asciz   "MySuperClass"
    .size   L_OBJC_CLASS_NAME_, 13

    .type   l_OBJC_METACLASS_RO_$_MySuperClass,@object # @"\01l_OBJC_METACLASS_RO_$_MySuperClass"
    .section    "__DATA, __objc_const","aw",@progbits
    .align  8
l_OBJC_METACLASS_RO_$_MySuperClass:
    .long   1                       # 0x1
    .long   40                      # 0x28
    .long   40                      # 0x28
    .zero   4
    .quad   0
    .quad   L_OBJC_CLASS_NAME_
    .quad   0
    .quad   0
    .quad   0
    .quad   0
    .quad   0
    .size   l_OBJC_METACLASS_RO_$_MySuperClass, 72

    .type   OBJC_METACLASS_$_MySuperClass,@object # @"OBJC_METACLASS_$_MySuperClass"
    .section    "__DATA, __objc_data","aw",@progbits
    .globl  OBJC_METACLASS_$_MySuperClass
    .align  8
OBJC_METACLASS_$_MySuperClass:
    .quad   OBJC_METACLASS_$_Object
    .quad   OBJC_METACLASS_$_Object
    .quad   _objc_empty_cache
    .quad   _objc_empty_vtable
    .quad   l_OBJC_METACLASS_RO_$_MySuperClass
    .size   OBJC_METACLASS_$_MySuperClass, 40

    .type   L_OBJC_METH_VAR_NAME_,@object # @"\01L_OBJC_METH_VAR_NAME_"
    .section    "__TEXT,__objc_methname,cstring_literals","aw",@progbits
L_OBJC_METH_VAR_NAME_:
    .asciz   "myMessage1"
    .size   L_OBJC_METH_VAR_NAME_, 11

    .type   L_OBJC_METH_VAR_TYPE_,@object # @"\01L_OBJC_METH_VAR_TYPE_"
    .section    "__TEXT,__objc_methtype,cstring_literals","aw",@progbits
L_OBJC_METH_VAR_TYPE_:
    .asciz   "v16@0:8"
    .size   L_OBJC_METH_VAR_TYPE_, 8

    .type   l_OBJC_$_INSTANCE_METHODS_MySuperClass,@object # @"\01l_OBJC_$_INSTANCE_METHODS_MySuperClass"
    .section    "__DATA, __objc_const","aw",@progbits
    .align  8
l_OBJC_$_INSTANCE_METHODS_MySuperClass:
    .long   24                      # 0x18
    .long   1                       # 0x1
    .quad   L_OBJC_METH_VAR_NAME_
    .quad   L_OBJC_METH_VAR_TYPE_
    .quad   _2D__5B_MySuperClass_20_myMessage1_5D_
    .size   l_OBJC_$_INSTANCE_METHODS_MySuperClass, 32

    .type   l_OBJC_CLASS_RO_$_MySuperClass,@object # @"\01l_OBJC_CLASS_RO_$_MySuperClass"
    .align  8
l_OBJC_CLASS_RO_$_MySuperClass:
    .long   0                       # 0x0
    .long   8                       # 0x8
    .long   8                       # 0x8
    .zero   4
    .quad   0
    .quad   L_OBJC_CLASS_NAME_
    .quad   l_OBJC_$_INSTANCE_METHODS_MySuperClass
    .quad   0
    .quad   0
    .quad   0
    .quad   0
    .size   l_OBJC_CLASS_RO_$_MySuperClass, 72

    .type   OBJC_CLASS_$_MySuperClass,@object # @"OBJC_CLASS_$_MySuperClass"
    .section    "__DATA, __objc_data","aw",@progbits
    .globl  OBJC_CLASS_$_MySuperClass
    .align  8
OBJC_CLASS_$_MySuperClass:
    .quad   OBJC_METACLASS_$_MySuperClass
    .quad   OBJC_CLASS_$_Object
    .quad   _objc_empty_cache
    .quad   _objc_empty_vtable
    .quad   l_OBJC_CLASS_RO_$_MySuperClass
    .size   OBJC_CLASS_$_MySuperClass, 40
{% endcodeblock %}
在编译得到的汇编代码中，不同的代码段对应ObjectiveC runtime中不同类型的数据结构。

`main`函数通常是我们认为的程序的入口，但是实际上，在进程刚刚开始执行时，`_objc_init`函数会在`main`函数之前被调用，在`_objc_init`函数中我们看到如下语句
{% codeblock lang:objc %}
dyld_register_image_state_change_handler(dyld_image_state_bound,
                                             1/*batch*/, &map_images);
{% endcodeblock %}
即在`_objc_init`时，runtime调用了`map_images`方法，`map_images`中又调用了`map_images_nolock`方法，继续下去我们会发现，程序调用了名为`realizeAllClasses`的方法，并最终在`static Class realizeClass(Class cls)`中具体实现了对某一个类的加载。

在`static Class realizeClass(Class cls)`中，系统实现了对类`cls`的初始化，包括为类对象分配空间，生成类的函数列表等。在这里我们以Category的实现为例。

我们可以在`static void methodizeClass(Class cls)`方法中看到对`static void attachCategoryMethods`的调用，这个方法就是我们要找的category的实现原理所在：
{% codeblock lang:objc %}
static void 
attachCategoryMethods(Class cls, category_list *cats, bool flushCaches)
{
    if (!cats) return;
    if (PrintReplacedMethods) printReplacements(cls, cats);

    bool isMeta = cls->isMetaClass();
    method_list_t **mlists = (method_list_t **)
        _malloc_internal(cats->count * sizeof(*mlists));

    // Count backwards through cats to get newest categories first
    int mcount = 0;
    int i = cats->count;
    BOOL fromBundle = NO;
    while (i--) {
        method_list_t *mlist = cat_method_list(cats->list[i].cat, isMeta);
        if (mlist) {
            mlists[mcount++] = mlist;
            fromBundle |= cats->list[i].fromBundle;
        }
    }

    attachMethodLists(cls, mlists, mcount, NO, fromBundle, flushCaches);

    _free_internal(mlists);

}
{% endcodeblock %}
函数第一个参数为当前正加载的类，第二个参数为对应类的所有的category的实现，在方法中，程序遍历了所有的category，将所有的在category中定义的函数，放在`mlists`中，然后利用`attachMethodLists`方法将category中添加的方法整合到类的method list当中，从而实现了category这一语言特性。

代码并不复杂，关键是我们在月的的过程中要慢慢找到函数间的调用层次，类加载这一部分的函数调用层次如下所示：
![](https://raw.githubusercontent.com/Charlesjean/cn/master/source/_posts/Category.png)

