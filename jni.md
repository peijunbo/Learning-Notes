# 编译运行

##### 生成头文件

使用下面的命令生成头文件(记得代码里要有native方法，否则是无法生成的)

```shell
javac -h <directory> <source_files>
# example
javac -h . Student.java
```

##### C/C++实现函数并编译出动态链接库

略

##### 执行.class文件

在`-Djava.library.path`参数中指明lib*.so文件的路径

```shell
java -Djava.library.path=./lib -classpath ./src/classes com.peijunbo.jnidemo.Main
```

# 类型转换

##### jstring -- const char *

```c
// jstring to const char *
// JNIEnv *env, jstring jstr
jboolean b = JNI_TRUE;
const char *str = env->GetStringUTFChars(/*str:*/jstr, /*isCopy:*/&b);

// const char * to jstring
jstring jstr = env->NewStringUTF(str);
```

##### j\<Type\>Array -- \<Type\> *

```c
```



