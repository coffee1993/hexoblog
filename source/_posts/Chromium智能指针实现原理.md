title: Android智能指针实现原理
date: 2016-07-05 20:31:09
tags:
- Android
categories:
- Android
---

## 几个重要的概念

C++中对象的构造和稀构是自动的，对象的创建和释放需要new 和 delete来完成，因此用对象的构造和稀构来完成对象的引用和释放，将可以实现智能指针技术。

智能指针是一个对象，而不是一个指针。

对象的生命周期需要引用计数器来维护，每当有一个新的指针指向这个对象，这个对象的引用计数就+1,当一个指针不再指向这个对象，这个对象的引用计数就减一，当对象的引用计数为0时，所占的内存就可以安全释放。


### 简单的引用计数器不能解决循环引用的问题。

A 引用 B ，B 引用 A，的时候，A不再使用，但是B引用着A，A不能释放。同理 B被A引用 ，B也不能释放。如果把引用分为强引用和弱引用，将引用关系方向化，父-子，子-父，则规定：父引用子为强引用，子引用父为弱引用，当子引用父的时候及弱引用，父对象不再使用时，父对象生命周期的销毁不受子对象的约束。当释放了父对象，父对子的计数器也会被释放；当子不再使用的时候，子也就安全释放了，因为没有父对象对他进行强引用。总而言之就是：不能释放被强引用的对象，可以释放被弱引用的对象。

### 被弱引用的对象不能被直接使用，需要升级为强引用

如果一个对象A被B弱引用，因为A对象的销毁不受B控制，所以B不知道A是否销毁，就不能对A直接使用，需要将对A的弱引用升级为强引用之后才能使用。如果不能升级，说明A对象已经释放。




## C++提供的三种智能指针 Light Pointer ,Strong Pointer, Weak Pointer

Light Pointer 轻量级指针使用的就是简单的引用技术，不能解决循环引用的问题。

所以支持智能指针的对象都需要对引用的对象进行计数。所以所有类都需要继承一个公共的引用技术类。 不同智能指针继承的公共引用计数类不同。

同时 Webkit 中的智能指针 和Chromium 中的智能指针和 Android系统源代码中的智能指针都有区别。


### Light Pointer 轻量级指针

在源代码目录结构：
coffee@ubuntu:~/Android/frameworks/rs/cpp/util$ ls
RefBase.h  StrongPointer.h  TypeHelpers.h

RefBase.h 中一个内部类： LightRefBase 轻量级指针的引用计数类就是LightRefBase 强指针的引用计数类为RefBase  

class LightRefBase
```c++
//.... RefBase ...
// -----------------------------------------------------------------------   
template <class T>                                           
class LightRefBase {
public:
    inline LightRefBase() : mCount(0) { }
    inline void incStrong(__attribute__((unused)) const void* id) const {      
        __sync_fetch_and_add(&mCount, 1);              
    }          

    inline void decStrong(__attribute__((unused)) const void* id) const {     
        if (__sync_fetch_and_sub(&mCount, 1) == 1) {            
            delete static_cast<const T*>(this);                                                                                                                               
        }                                                                                                                                                                     
    }         
    //! DEBUGGING ONLY: Get current strong ref count.                                                    
    inline int32_t getStrongCount() const {                                                                                                                                   
        return mCount;                                                                                                                                                        
    }                                                                                                                                                                         
    typedef LightRefBase<T> basetype;     
protected:                                                                   
    inline ~LightRefBase() { }
    friend class ReferenceMover;            

    inline static void moveReferences(void*, void const*, size_t,                                                                                                             
            const ReferenceConverterBase&) { }                     
private:                                                                                                                                                                      
    mutable volatile int32_t mCount;                                                                                                                                          
};                                                                                                                                                                                                                 // ---------------------------------------------------------------------------   
// ... RefBase ...
```

LightRefBase 只有一个成员变量 int32_t 类型的 mCount 用来描述一个对象的引用计数。
incStrong 内联函数用来增加引用计数
desStrong 内联函数用来减少引用计数
如果引用计数减少后为0,表示释放这个对象所占用的内存。


轻量级指针的实现是sp，同时也是 **强指针** 的实现类。

sp 其本身也是一个模板类
通过RefBase::weakref_type来实现引用计数
m_ptr 为成员变量，指向实际引用的对象。
包含 构造函数和拷贝构造函数，两个都对指向的对象执行从父类继承的incStrong操作。从而增加引用计数。

### 轻量级指针的使用使用举列：

```c++
#include <stdio.h>
#include <utils/RefBase.h>

using namespace android;

class LightClass : public LightRefBase <LightClass>
{
public:
  LightClass(){

      printf("Construct LightClass object");

  }

  virtual ~LightClass(){
      printf("Destroy LightClass Object");
  }

}

int int main(int argc, char ** argv ) {

  LightClass * pLightClass = new LightClass();
  sp<LightClass> lpOut = pLightClass; //引用计数器+1
  return 0;
}
```
### 强指针 弱指针

轻量级指针不能解决循环引用的问题，需要强指针和弱指针的概念，强指针和弱指针的关系比较密切，需要配合使用，同时 RefBase 相对于 LightRefBase 复杂，解决了更多问题。
和轻量级指针的相同点：都用incStrong ,desStrong 来操作引用计数器。但是引用计数器又有强指针和弱指针的概念，轻量级指针仅仅通过整形变量mutable volatile int32_t mCount 来做计数。RefBase中，强指针弱指针通过类型为 weakref_impl 类型的 mRefs 变量来提供计数功能，同时 weakref_impl中 也有一个RefBase * 类型的指针 mBase来指向宿主类RefBase。
其引用计数器不仅仅是简单的 接口类在RefBase中定义weakref_type:

RefBase代码片段 weakref_type 定义
```c++
//.......
//.......

// 1 RefBase 源代码
class RefBase  
{  
public:  
    void            incStrong(const void* id) const;  
    void            decStrong(const void* id) const;  

    void            forceIncStrong(const void* id) const;  

    //! DEBUGGING ONLY: Get current strong ref count.  
    int32_t         getStrongCount() const;  

    class weakref_type   //接口定义
    {  
    public:  
        RefBase*            refBase() const;  

        void                incWeak(const void* id);  
        void                decWeak(const void* id);  

        bool                attemptIncStrong(const void* id);  

        //! This is only safe if you have set OBJECT_LIFETIME_FOREVER.  
        bool                attemptIncWeak(const void* id);  

        //! DEBUGGING ONLY: Get current weak ref count.  
        int32_t             getWeakCount() const;  

        //! DEBUGGING ONLY: Print references held on object.  
        void                printRefs() const;  

        //! DEBUGGING ONLY: Enable tracking for this object.  
        // enable -- enable/disable tracking  
        // retain -- when tracking is enable, if true, then we save a stack trace  
        //           for each reference and dereference; when retain == false, we  
        //           match up references and dereferences and keep only the   
        //           outstanding ones.  

        void                trackMe(bool enable, bool retain);  
    };  

    weakref_type*   createWeak(const void* id) const;  

    weakref_type*   getWeakRefs() const;  

    //! DEBUGGING ONLY: Print references held on object.  
    inline  void            printRefs() const { getWeakRefs()->printRefs(); }
    //! DEBUGGING ONLY: Enable tracking of object.  
    inline  void            trackMe(bool enable, bool retain)  
    {  
        getWeakRefs()->trackMe(enable, retain);  
    }  

protected:  
    RefBase();  
    virtual                 ~RefBase();  

    //! Flags for extendObjectLifetime()  
    enum {  
      OBJECT_LIFETIME_STRONG  = 0x0000,
      OBJECT_LIFETIME_WEAK    = 0x0001,
      OBJECT_LIFETIME_MASK    = 0x0001
    };  

    void            extendObjectLifetime(int32_t mode);  

    //! Flags for onIncStrongAttempted()  
    enum {  
        FIRST_INC_STRONG = 0x0001  
    };  

    virtual void            onFirstRef();  
    virtual void            onLastStrongRef(const void* id);  
    virtual bool            onIncStrongAttempted(uint32_t flags, const void* id);  
    virtual void            onLastWeakRef(const void* id);  

private:  
    friend class weakref_type;  
    class weakref_impl;  

    RefBase(const RefBase& o);  
    RefBase&        operator=(const RefBase& o);  

    weakref_impl* const mRefs;  
};  



强指针的的定义为sp ，通过RefBase::weakref_type来实现引用计数:
RefBase.h中包含SP的头文件：

```c++
#include <stdint.h>
#include <sys/types.h>
#include <stdlib.h>
#include <string.h>

#include "StrongPointer.h" //SP
#include "TypeHelpers.h"
```

sp的定义
StrongPointer.h
```c++
// ... StrongPointer ...
//...
template <typename T>  
class sp  
{  
public:  
    typedef typename RefBase::weakref_type weakref_type;  

    inline sp() : m_ptr(0) { }

    sp(T* other);  
    sp(const sp<T>& other);  
    template<typename U> sp(U* other);  
    template<typename U> sp(const sp<U>& other);  

    ~sp();  

    // Assignment  

    sp& operator = (T* other);  
    sp& operator = (const sp<T>& other);  

    template<typename U> sp& operator = (const sp<U>& other);  
    template<typename U> sp& operator = (U* other);  

    //! Special optimization for use by ProcessState (and nobody else).  
    void force_set(T* other);  

    // Reset  

    void clear();  

    // Accessors  

    inline  T&      operator * () const  { return * m_ptr; }  
    inline  T*      operator-> () const { return m_ptr;  }  
    inline  T*      get() const         { return m_ptr; }  

    // Operators  

    COMPARE(==)  
        COMPARE(!=)  
        COMPARE(>)  
        COMPARE(<)  
        COMPARE(<=)  
        COMPARE(>=)  

private:  
    template<typename Y> friend class sp;  
    template<typename Y> friend class wp;  

    // Optimization for wp::promote().  
    sp(T* p, weakref_type* refs);  

    T*              m_ptr;  
};  

//...
```
sp的实现也在头文件中
weakref_impl 的接口在RefBase::weakref_type中定义，如上源代码。weakref_impl 继承 RefBase::weakref_type实现在
coffee@ubuntu:~/Android/system/core/libutils$ RefBase.cpp中

```c++
class RefBase::weakref_impl : public RefBase::weakref_type  
{  
public:  
    volatile int32_t    mStrong;  
    volatile int32_t    mWeak;  
    RefBase* const      mBase;  
    volatile int32_t    mFlags;  
#if !DEBUG_REFS  

    weakref_impl(RefBase* base)  
        : mStrong(INITIAL_STRONG_VALUE)  
        , mWeak(0)  
        , mBase(base)  
        , mFlags(0)  
    {  
    }

    void addStrongRef(const void* /*id*/) { }  
    void removeStrongRef(const void* /*id*/) { }  
    void addWeakRef(const void* /*id*/) { }  
    void removeWeakRef(const void* /*id*/) { }  
    void printRefs() const { }  
    void trackMe(bool, bool) { }  

#else  
    weakref_impl(RefBase* base)  
        : mStrong(INITIAL_STRONG_VALUE)  
        , mWeak(0)  
        , mBase(base)  
        , mFlags(0)  
        , mStrongRefs(NULL)  
        , mWeakRefs(NULL)  
        , mTrackEnabled(!!DEBUG_REFS_ENABLED_BY_DEFAULT)  
        , mRetain(false)  
    {  
        //LOGI("NEW weakref_impl %p for RefBase %p", this, base);  
    }  

    ~weakref_impl()  
    {  
        LOG_ALWAYS_FATAL_IF(!mRetain && mStrongRefs != NULL, "Strong references remain!");  
        LOG_ALWAYS_FATAL_IF(!mRetain && mWeakRefs != NULL, "Weak references remain!");  
    }  

    void addStrongRef(const void* id)  
    {  
        addRef(&mStrongRefs, id, mStrong);  
    }  

    void removeStrongRef(const void* id)  
    {  
        if (!mRetain)  
            removeRef(&mStrongRefs, id);  
        else  
            addRef(&mStrongRefs, id, -mStrong);  
    }  

    void addWeakRef(const void* id)  
    {  
        addRef(&mWeakRefs, id, mWeak);  
    }  
    void removeWeakRef(const void* id)  
    {  
        if (!mRetain)  
            removeRef(&mWeakRefs, id);  
        else  
            addRef(&mWeakRefs, id, -mWeak);  
    }  

    void trackMe(bool track, bool retain)  
    {  
        mTrackEnabled = track;  
        mRetain = retain;  
    }  

    ......  

private:  
    struct ref_entry  
    {  
        ref_entry* next;  
        const void* id;  


#if DEBUG_REFS_CALLSTACK_ENABLED  
        CallStack stack;  
#endif  

        int32_t ref;  
    };  

    void addRef(ref_entry** refs, const void* id, int32_t mRef)  
    {  
        if (mTrackEnabled) {  
            AutoMutex _l(mMutex);  
            ref_entry* ref = new ref_entry;  
            // Reference count at the time of the snapshot, but before the  
            // update.  Positive value means we increment, negative--we  
            // decrement the reference count.  
            ref->ref = mRef;  
            ref->id = id;  

#if DEBUG_REFS_CALLSTACK_ENABLED  
            ref->stack.update(2);  
#endif  

            ref->next = *refs;  
            *refs = ref;  
        }  
    }  

    void removeRef(ref_entry** refs, const void* id)  
    {  
        if (mTrackEnabled) {  
            AutoMutex _l(mMutex);  

            ref_entry* ref = *refs;  
            while (ref != NULL) {  
                if (ref->id == id) {  
                    *refs = ref->next;  
                    delete ref;  
                    return;  
                }  

                refs = &ref->next;  
                ref = *refs;  
            }  

            LOG_ALWAYS_FATAL("RefBase: removing id %p on RefBase %p (weakref_type %p) that doesn't exist!",  
                id, mBase, this);  
        }  
    }  

    ......  

    Mutex mMutex;  
    ref_entry* mStrongRefs;  
    ref_entry* mWeakRefs;  

    bool mTrackEnabled;  
    // Collect stack traces on addref and removeref, instead of deleting the stack references  
    // on removeref that match the address ones.  
    bool mRetain;  

    ......  
#endif  
};  
```

RefBase.cpp 代码结构：

```
#if !DEBUG_REFS  

......  release 部分源代码 空实现

#else  

#else   

......  Debug部分源代码 成员函数有实现 还有一个结构体ref_entity

#endif  
```
成员函数分析：
```c++
volatile int32_t    mStrong;  
volatile int32_t    mWeak;  
RefBase* const      mBase;  
volatile int32_t    mFlags;  
```
mStrong 和 mWeak 分别表示强引用计数 和弱引用计数 , mBase山文已经提到过，RefBase中，强指针弱指针通过类型为 weakref_impl 类型的 mRefs 变量来提供计数功能，同时 weakref_impl中 也有一个RefBase * 类型的指针 mBase来指向宿主类RefBase。 mFlag 表示维护 对象引用计数的 策略 取值如下：

RefBase.h
```c++
//! Flags for extendObjectLifetime()
 enum {
        OBJECT_LIFETIME_STRONG  = 0x0000,
        OBJECT_LIFETIME_WEAK    = 0x0001,
        OBJECT_LIFETIME_MASK    = 0x0001
};
```

分析sp 的实现构造函数如下：
```c++
template<typename T>
sp<T>::sp(T* other)
: m_ptr(other)
  {
    if (other) other->incStrong(this);
  }

template<typename T>
sp<T>::sp(const sp<T>& other)
: m_ptr(other.m_ptr)
  {
    if (m_ptr) m_ptr->incStrong(this);
  }

template<typename T> template<typename U>
sp<T>::sp(U* other) : m_ptr(other)
{
    if (other) ((T*)other)->incStrong(this);
}
```

构造函数传入的参数T other 肯定需要继承至RefBase类


coffee@ubuntu:~/Android/system/core/libutils$
RefBase.cpp 实现：
```c++
// ---------------------------------------------------------------------------

void RefBase::incStrong(const void* id) const
{
    weakref_impl* const refs = mRefs;
    //增加弱引用计数
    refs->incWeak(id);
    //增加强引用技术
    refs->addStrongRef(id);
    // c 为 mStrong +1 之前的值  c == INITIAL_STRONG_VALUE 说明第一次调用
    const int32_t c = android_atomic_inc(&refs->mStrong);
    ALOG_ASSERT(c > 0, "incStrong() called on %p after last strong ref", refs);
#if PRINT_REFS
    ALOGD("incStrong of %p from %p: cnt=%d\n", this, id, c);
#endif
    // INITIAL_STRONG_VALUE : 1<<28
    if (c != INITIAL_STRONG_VALUE)  {
        return;
    }   
    //首次调用incStrong 函数，会调用onFirstRef函数 对象首次引用的时候需要一些逻辑处理
    android_atomic_add(-INITIAL_STRONG_VALUE, &refs->mStrong);
    refs->mBase->onFirstRef();
}



void RefBase::decStrong(const void* id) const
{
    weakref_impl* const refs = mRefs;
    refs->removeStrongRef(id);
    const int32_t c = android_atomic_dec(&refs->mStrong);
#if PRINT_REFS
    ALOGD("decStrong of %p from %p: cnt=%d\n", this, id, c);
#endif
    ALOG_ASSERT(c >= 1, "decStrong() called on %p too many times", refs);
    if (c == 1) {
        refs->mBase->onLastStrongRef(id);
        if ((refs->mFlags&OBJECT_LIFETIME_MASK) == OBJECT_LIFETIME_STRONG) {
            delete this;
        }   
    }   
    refs->decWeak(id);
}


```
