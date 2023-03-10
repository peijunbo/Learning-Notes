```c
jclass pcls = env->GetObjectClass(parent);
jclass ccls = env->GetObjectClass(child);
jclass fcls = env->FindClass("com/peijunbo/jnidemo/Parent");
_jmethodID
jmethodID fHello = env->GetMethodID(fcls, "hello", "()V");
jmethodID pHello = env->GetMethodID(pcls, "hello", "()V");
jmethodID cHello = env->GetMethodID(ccls, "hello", "()V");
```



# case 1

```java
Parent parent = new Parent();
Child child = new Child();
```



# field id

1. 根据实际类型

2. 父子定义同名变量，id不同

   ```java
   class Parent {
       int s = 1;
       ...
   }
   class Child extends Parent{
       int s = 2;// id不同
   }
   ```

   

3. 孩子修改继承自父亲变量，id相同

   ```java
   class Parent {
       int s = 1;
       ...
   }
   class Child extends Parent{
       Child() {
           s = 2;// id相同
       }
   }
   ```

4. 函数中的field id 是编译时跟class绑定的

   ```java
   class Parent {
       int s = 1;// 假设fieldID=0
       public void hello() {
           sout(s);// jMethodId中会直接访问fieldId=0的内存
       }
       ...
   }
   class Child extends Parent{
       int s = 2;// 假设fieldID=2
       public void hello() {
          sout(s);// jMethodID中会直接访问fieldId=2的内存
       }
       ...
   }
   ```

   
