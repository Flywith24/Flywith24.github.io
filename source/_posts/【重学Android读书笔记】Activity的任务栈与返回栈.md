---
title: 【重学Android读书笔记】Activity的任务栈与返回栈
date: 2019.11.29 17:42
updated: 2019.11.29 17:42
categories: 
- app
tags: 
- app
- 重学Android

---

> 订阅 [重学安卓](https://xiaozhuanlan.com/topic/9074561823) 很久了，最近在整理读书笔记，在此记录之。
>
> 在此隆重推荐这位大佬 KunMinX
>
> 本文记录Activity任务栈与返回栈相关内容的疑问与探索

<!-- more -->

该专栏的代码地址： https://github.com/KunMinX/Relearn-Android 

#### 疑惑产生

根据大佬的代码以及文章描述，有个地方让我很疑惑。

![问题](【重学Android读书笔记】Activity的任务栈与返回栈\问题.png)

我这里没有过滤当前使用的app，所以app1和app2产生的日志都显示了出来。

大佬在评论区关于此的回答

>
> [KunMinX](https://xiaozhuanlan.com/u/kunminx)
>
> \#40
>
> [#2楼](https://xiaozhuanlan.com/topic/7812045693#reply2) [*@*Sky63](https://xiaozhuanlan.com/u/2711412979)
> 大概明白你提到的、造成困扰的地方了。
> D 和 C 是由 app1 启动，在 IDE 中可通过 app1 的视角观察到。同理，当 app2 的 B 唤起 D 时，你在回退 D 的时候能在 app2 和 app1 的视角中同时观察到 D 的销毁。而紧随其后再回退一次， 便能在 app1 中观察到 C 被销毁。再下一次才轮到 app2 中 B 被销毁。

#### 问题排查

通过 `adb shell` 中的 ` dumpsys activity activities` 命令可以查看 activity 栈信息，故我截取了相关的输出

log1

![log1](【重学Android读书笔记】Activity的任务栈与返回栈\log1.png)

```
➜  Relearn-Android (master) adb shell
sagit:/ $ dumpsys activity activities
ACTIVITY MANAGER ACTIVITIES (dumpsys activity activities)
Display #0 (activities from top to bottom):

  Stack #2: type=standard mode=fullscreen
    Task id #3

    Running activities (most recent first):
      TaskRecord{4920da #3 A=com.kunminx.task.c U=0 StackId=2 sz=2}
        Run #1: ActivityRecord{93c0c72 u0 com.kunminx.relearn_android/com.kunminx.basicfacttesting.test03_task_test.SingleTaskTwoActivity t3}
        Run #0: ActivityRecord{999111 u0 com.kunminx.relearn_android/com.kunminx.basicfacttesting.test03_task_test.SingleTaskOneActivity t3}

    mResumedActivity: ActivityRecord{93c0c72 u0 com.kunminx.relearn_android/com.kunminx.basicfacttesting.test03_task_test.SingleTaskTwoActivity t3}
    mLastPausedActivity: ActivityRecord{999111 u0 com.kunminx.relearn_android/com.kunminx.basicfacttesting.test03_task_test.SingleTaskOneActivity t3}

  Stack #1: type=standard mode=fullscreen
    Task id #2

    Running activities (most recent first):
      TaskRecord{d91b3e8 #2 A=com.kunminx.relearn_android U=0 StackId=1 sz=2}
        Run #1: ActivityRecord{8b59240 u0 com.kunminx.relearn_android/com.kunminx.basicfacttesting.test03_task_test.StandardOneActivity t2}
        Run #0: ActivityRecord{d76e9a5 u0 com.kunminx.relearn_android/com.kunminx.basicfacttesting.TestMainActivity t2}

    mLastPausedActivity: ActivityRecord{8b59240 u0 com.kunminx.relearn_android/com.kunminx.basicfacttesting.test03_task_test.StandardOneActivity t2}

  Stack #0: type=home mode=fullscreen
  isSleeping=false
  mBounds=Rect(0, 0 - 0, 0)

    Task id #1
    mBounds=Rect(0, 0 - 0, 0)
    mMinWidth=-1
    mMinHeight=-1
    mLastNonFullscreenBounds=null

    Running activities (most recent first):
      TaskRecord{94a7a01 #1 I=com.miui.home/.launcher.Launcher U=0 StackId=0 sz=1}
        Run #0: ActivityRecord{7d6409 u0 com.miui.home/.launcher.Launcher t1}

    mLastPausedActivity: ActivityRecord{7d6409 u0 com.miui.home/.launcher.Launcher t1}

  ResumedActivity: ActivityRecord{93c0c72 u0 com.kunminx.relearn_android/com.kunminx.basicfacttesting.test03_task_test.SingleTaskTwoActivity t3}
  mFocusedStack=ActivityStack{3a870a6 stackId=2 type=standard mode=fullscreen visible=true translucent=false, 1 tasks} mLastFocusedStack=ActivityStack{3a870a6 stackId=2 type=standard mode=fullscreen visib
le=true translucent=false, 1 tasks}
  mCurTaskIdForUser={0=3}
  mUserStackInFront={}
  displayId=0 stacks=3
   mHomeStack=ActivityStack{76ebae7 stackId=0 type=home mode=fullscreen visible=false translucent=true, 1 tasks}
  isHomeRecentsComponent=false  KeyguardController:
    mKeyguardShowing=false
    mAodShowing=false
    mKeyguardGoingAway=false
    mOccluded=false
    mDismissingKeyguardActivity=null
    mDismissalRequested=false
    mVisibilityTransactionDepth=0
  LockTaskController
    mLockTaskModeState=NONE
    mLockTaskModeTasks=
    mLockTaskPackages (userId:packages)=
      u0:[]

```

log2

![log2](【重学Android读书笔记】Activity的任务栈与返回栈\log2.png)

```
Display #0 (activities from top to bottom):

  Stack #3: type=standard mode=fullscreen
    Task id #4

    Running activities (most recent first):
      TaskRecord{718157f #4 A=com.kunminx.relearn_android_2 U=0 StackId=3 sz=4}
        Run #3: ActivityRecord{137a584 u0 com.kunminx.relearn_android_2/com.kunminx.basicfacttesting.test03_task_test.StandardTwoActivity t4}
        Run #2: ActivityRecord{648188e u0 com.kunminx.relearn_android_2/com.kunminx.basicfacttesting.test03_task_test.StandardOneActivity t4}
        Run #1: ActivityRecord{f39ebe9 u0 com.kunminx.relearn_android_2/com.kunminx.basicfacttesting.test03_task_test.StandardOneActivity t4}
        Run #0: ActivityRecord{f02fff8 u0 com.kunminx.relearn_android_2/com.kunminx.basicfacttesting.TestMainActivity t4}

    mResumedActivity: ActivityRecord{137a584 u0 com.kunminx.relearn_android_2/com.kunminx.basicfacttesting.test03_task_test.StandardTwoActivity t4}
    mLastPausedActivity: ActivityRecord{648188e u0 com.kunminx.relearn_android_2/com.kunminx.basicfacttesting.test03_task_test.StandardOneActivity t4}

  Stack #0: type=home mode=fullscreen
    Task id #1

    Running activities (most recent first):
      TaskRecord{2094d95 #1 I=com.miui.home/.launcher.Launcher U=0 StackId=0 sz=1}
        Run #0: ActivityRecord{a7fed1a u0 com.miui.home/.launcher.Launcher t1}

    mLastPausedActivity: ActivityRecord{a7fed1a u0 com.miui.home/.launcher.Launcher t1}

  Stack #2: type=standard mode=fullscreen
    Task id #3
    
    Running activities (most recent first):
      TaskRecord{9d79eaa #3 A=com.kunminx.task.c U=0 StackId=2 sz=2}
        Run #1: ActivityRecord{15d3e09 u0 com.kunminx.relearn_android/com.kunminx.basicfacttesting.test03_task_test.SingleTaskTwoActivity t3}
        Run #0: ActivityRecord{20d177c u0 com.kunminx.relearn_android/com.kunminx.basicfacttesting.test03_task_test.SingleTaskOneActivity t3}

    mLastPausedActivity: ActivityRecord{15d3e09 u0 com.kunminx.relearn_android/com.kunminx.basicfacttesting.test03_task_test.SingleTaskTwoActivity t3}

  Stack #1: type=standard mode=fullscreen
  isSleeping=false
  mBounds=Rect(0, 0 - 0, 0)

    Task id #2

    Running activities (most recent first):
      TaskRecord{e526e38 #2 A=com.kunminx.relearn_android U=0 StackId=1 sz=2}
        Run #1: ActivityRecord{19055b2 u0 com.kunminx.relearn_android/com.kunminx.basicfacttesting.test03_task_test.StandardOneActivity t2}
        Run #0: ActivityRecord{5a647a6 u0 com.kunminx.relearn_android/com.kunminx.basicfacttesting.TestMainActivity t2}

    mLastPausedActivity: ActivityRecord{19055b2 u0 com.kunminx.relearn_android/com.kunminx.basicfacttesting.test03_task_test.StandardOneActivity t2}

  ResumedActivity: ActivityRecord{137a584 u0 com.kunminx.relearn_android_2/com.kunminx.basicfacttesting.test03_task_test.StandardTwoActivity t4}
  mFocusedStack=ActivityStack{dfe2e11 stackId=3 type=standard mode=fullscreen visible=true translucent=false, 1 tasks} mLastFocusedStack=ActivityStack{dfe2e11 stackId=3 type=standard mode=fullscreen visib
le=true translucent=false, 1 tasks}
  mCurTaskIdForUser={0=4}
  mUserStackInFront={}
  displayId=0 stacks=4
   mHomeStack=ActivityStack{968fb76 stackId=0 type=home mode=fullscreen visible=false translucent=true, 1 tasks}
  isHomeRecentsComponent=false  KeyguardController:
    mKeyguardShowing=false
    mAodShowing=false
    mKeyguardGoingAway=false
    mOccluded=false
    mDismissingKeyguardActivity=null
    mDismissalRequested=false
    mVisibilityTransactionDepth=0
  LockTaskController
    mLockTaskModeState=NONE
    mLockTaskModeTasks=
    mLockTaskPackages (userId:packages)=
      u0:[]
      
```
log3

![log3](【重学Android读书笔记】Activity的任务栈与返回栈\log3.png)

```
sagit:/ $ dumpsys activity activities
ACTIVITY MANAGER ACTIVITIES (dumpsys activity activities)
Display #0 (activities from top to bottom):

  Stack #2: type=standard mode=fullscreen
    Task id #3

    Running activities (most recent first):
      TaskRecord{4920da #3 A=com.kunminx.task.c U=0 StackId=2 sz=3}
        Run #2: ActivityRecord{55f799d u0 com.kunminx.relearn_android_2/com.kunminx.basicfacttesting.test03_task_test.SingleTaskTwoActivity t3}
        Run #1: ActivityRecord{93c0c72 u0 com.kunminx.relearn_android/com.kunminx.basicfacttesting.test03_task_test.SingleTaskTwoActivity t3}
        Run #0: ActivityRecord{999111 u0 com.kunminx.relearn_android/com.kunminx.basicfacttesting.test03_task_test.SingleTaskOneActivity t3}

    mResumedActivity: ActivityRecord{55f799d u0 com.kunminx.relearn_android_2/com.kunminx.basicfacttesting.test03_task_test.SingleTaskTwoActivity t3}

  Stack #3: type=standard mode=fullscreen
    Task id #4

    Running activities (most recent first):
      TaskRecord{f14a51a #4 A=com.kunminx.relearn_android_2 U=0 StackId=3 sz=4}
        Run #3: ActivityRecord{1838862 u0 com.kunminx.relearn_android_2/com.kunminx.basicfacttesting.test03_task_test.StandardTwoActivity t4}
        Run #2: ActivityRecord{bdb3180 u0 com.kunminx.relearn_android_2/com.kunminx.basicfacttesting.test03_task_test.StandardOneActivity t4}
        Run #1: ActivityRecord{7b21450 u0 com.kunminx.relearn_android_2/com.kunminx.basicfacttesting.test03_task_test.StandardOneActivity t4}
        Run #0: ActivityRecord{ff57abb u0 com.kunminx.relearn_android_2/com.kunminx.basicfacttesting.TestMainActivity t4}

    mLastPausedActivity: ActivityRecord{1838862 u0 com.kunminx.relearn_android_2/com.kunminx.basicfacttesting.test03_task_test.StandardTwoActivity t4}

  Stack #0: type=home mode=fullscreen
    Task id #1

    Running activities (most recent first):
      TaskRecord{94a7a01 #1 I=com.miui.home/.launcher.Launcher U=0 StackId=0 sz=1}
        Run #0: ActivityRecord{7d6409 u0 com.miui.home/.launcher.Launcher t1}

    mLastPausedActivity: ActivityRecord{7d6409 u0 com.miui.home/.launcher.Launcher t1}

  Stack #1: type=standard mode=fullscreen

    Task id #2
        
    Running activities (most recent first):
      TaskRecord{d91b3e8 #2 A=com.kunminx.relearn_android U=0 StackId=1 sz=2}
        Run #1: ActivityRecord{8b59240 u0 com.kunminx.relearn_android/com.kunminx.basicfacttesting.test03_task_test.StandardOneActivity t2}
        Run #0: ActivityRecord{d76e9a5 u0 com.kunminx.relearn_android/com.kunminx.basicfacttesting.TestMainActivity t2}

    mLastPausedActivity: ActivityRecord{8b59240 u0 com.kunminx.relearn_android/com.kunminx.basicfacttesting.test03_task_test.StandardOneActivity t2}

  ResumedActivity: ActivityRecord{55f799d u0 com.kunminx.relearn_android_2/com.kunminx.basicfacttesting.test03_task_test.SingleTaskTwoActivity t3}
  mFocusedStack=ActivityStack{3a870a6 stackId=2 type=standard mode=fullscreen visible=true translucent=false, 1 tasks} mLastFocusedStack=ActivityStack{3a870a6 stackId=2 type=standard mode=fullscreen visib
le=true translucent=false, 1 tasks}
  mCurTaskIdForUser={0=4}
  mUserStackInFront={}
  displayId=0 stacks=4
   mHomeStack=ActivityStack{76ebae7 stackId=0 type=home mode=fullscreen visible=false translucent=true, 1 tasks}
  isHomeRecentsComponent=false  KeyguardController:
    mKeyguardShowing=false
    mAodShowing=false
    mKeyguardGoingAway=false
    mOccluded=false
    mDismissingKeyguardActivity=null
    mDismissalRequested=false
    mVisibilityTransactionDepth=0
  LockTaskController
    mLockTaskModeState=NONE
    mLockTaskModeTasks=
    mLockTaskPackages (userId:packages)=
      u0:[]

```

根据log3 的结果 

![结果](【重学Android读书笔记】Activity的任务栈与返回栈\结果.png)

的确有两个 `SingleTaskTwoActivity` 的 `ActivityRecord` 而且它所在的 `Stack` 为2 

