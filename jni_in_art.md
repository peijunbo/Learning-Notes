JavaVM相关接口的相关实现在java_vm_ext.cc

JNIEnv相关接口的实现在jni_internal.cc

对于art而言，JavaVM对象的实际类型为JavaVMExt类，JavaVMExt类继承了JavaVM类，JavaENV与JavaENVExt同理。



ArtField类的declaring_class成员变量指向声明该成员的类是谁



jfieldID 类在art中实际上对应ArtField*

jmethod对应ArtMethod*

jobject对应mirror::Object*

jobject实际类型为`IndirectRef`，表示一个引用。如果kind为kJniTransition，表示这个对象来自函数参数，此时IndirectRef存储的实际类型为`StackReference<mirror::Object*>`。如果为其他三类，则从对应的`IndirectRefTable`中找到对应的`mirror::Object`对象。
```c++
namespace mirror {
class Object;
}  // namespace mirror
// Indirect reference definition.  This must be interchangeable with JNI's jobject, and it's
// convenient to let null be null, so we use void*.
//
// We need a 2-bit reference kind (global, local, weak global) and the rest of the `IndirectRef`
// is used to locate the actual reference storage.
//
// For global and weak global references, we need a (potentially) large table index and we also
// reserve some bits to be used to detect stale indirect references: we put a serial number in
// the extra bits, and keep a copy of the serial number in the table. This requires more memory
// and additional memory accesses on add/get, but is moving-GC safe. It will catch additional
// problems, e.g.: create iref1 for obj, delete iref1, create iref2 for same obj, lookup iref1.
// A pattern based on object bits will miss this.
//
// Local references use the same bits for the reference kind but the rest of their `IndirectRef`
// encoding is different, see `LocalReferenceTable` for details.
using IndirectRef = void*;
// Indirect reference kind, used as the two low bits of IndirectRef.
//
// For convenience these match up with enum jobjectRefType from jni.h, except that
// we use value 0 for JNI transitions instead of marking invalid reference type.
enum IndirectRefKind {
  kJniTransition = 0,  // <<JNI transition frame reference>>
  kLocal         = 1,  // <<local reference>>
  kGlobal        = 2,  // <<global reference>>
  kWeakGlobal    = 3,  // <<weak global reference>>
  kLastKind      = kWeakGlobal
};
```
