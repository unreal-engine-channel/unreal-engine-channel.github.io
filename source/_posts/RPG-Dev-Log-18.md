---
title: RPG Game开发日志(十八)
date: 2020-11-16 15:24:17
categories: Logs
tags: UE_4.22
---

涉及内容：脚部IK制作。

<!--more-->

# 涉及内容

## 脚部IK制作

### 事件图表

#### 相关宏

1. 创建宏

   ```
   [Start(Vector),End(Vector)]TraceStart/End(X(Float),Y(Float),Z(Float))
   ```

   用于计算射线检测的起点和终点位置;<img src='https://img-blog.csdnimg.cn/20201116155309202.png'>
   
2. 创建宏

   ```
   [ClampedOffset(Vector)]ClampOffset(Roll(Float),Pitch(Float),Z(Float))
   ```

   用于限制X,Y,Z轴的偏移距离；<img src='https://img-blog.csdnimg.cn/20201116161451526.png'>

3. 创建宏

```
[Return(Vector)]InterpOffsets(Current(Vector),Target(Vector))
```
用于从Current到Target插值计算X,Y,Z轴偏移量;<img src='https://img-blog.csdnimg.cn/20201116164750110.png'>
> **[Vector2DInterpTo](https://docs.unrealengine.com/en-US/API/Runtime/Core/Math/FMath/Vector2DInterpTo/index.html)**
>
> Interpolate vector2D from Current to Target.
>
> **[FInterpTo](https://docs.unrealengine.com/en-US/API/Runtime/Core/Math/FMath/FInterpTo/index.html)**
>
> Interpolate float from Current to Target.
>
> ```c++
> CORE_API float FMath::FInterpTo( float Current, float Target, float DeltaTime, float InterpSpeed )
> {
>     // If no interp speed, jump to target value
>     if( InterpSpeed <= 0.f )
>     {
>         return Target;
>     }
> 
>     // Distance to reach
>     const float Dist = Target - Current;
> 
>     // If distance is too small, just set the desired location
>     if( FMath::Square(Dist) < SMALL_NUMBER )
>     {
>         return Target;
>     }
> 
>     // Delta Move, Clamp so we do not over shoot.
>     const float DeltaMove = Dist * FMath::Clamp<float>(DeltaTime * InterpSpeed, 0.f, 1.f);
> 
>     return Current + DeltaMove;
> }
> ```

#### 相关函数

1. 折叠函数

   ```
   [OutHit_bBlockingHit(Boolean),ClampedOffset(Vector)]TraceFootIK(A_Z(Float),B_X(Float),B_Y(Float))
   ```
   > **[LineTraceByChannel](https://docs.unrealengine.com/en-US/BlueprintAPI/Collision/LineTraceByChannel/index.html)**
   >
   > Does a collision trace along the given line and returns the first blocking hit encountered. This trace finds the objects that RESPONDS to the given TraceChannel

   调用宏TraceStart/End和函数LineTraceByChannel，计算脚部Roll,Pitch轴的Rotation偏移量和Z轴偏移量，并调用宏ClampOffset，限制偏移范围。具体计算方式为：
   
   Roll = arctan(OutHitNormal.y/OutHitNormal.z)
   
   Pitch = arctan(OutHitNormal.x/OutHitNormal.z)
   
   z = WorldLoaction.z - OutHitLocation.z；<img src='https://img-blog.csdnimg.cn/20201116163019606.png'>
   
2. 函数

   ```
   [void]FootIK()
   ```

   脚部IK数值计算汇总（此函数在EventBlueprintUpdateAnimation中调用）：当角色不在跳跃状态时，获取foot_l和foot_r的骨骼位置，调用函数TraceFootIK，根据Hit布尔值设置LeftFootOffsetsTarget的值(ClampedOffset or Zero)。循环调用宏InterpOffsets实现脚部偏移过渡效果，并设置Pelvis.z=Min(LeftFootOffset.z,RightFootOffset.z);

   <img src='https://img-blog.csdnimg.cn/2020111617001856.png'>

   <img src='https://img-blog.csdnimg.cn/2020111616594744.png'>

   <img src='https://img-blog.csdnimg.cn/20201116165832569.png'>

### 动画图表

1. 现有Pose：Base—>UpBody—>LocalPose;<img src='https://img-blog.csdnimg.cn/20201116170745543.png'>
2. 打开骨骼树，添加虚拟骨骼VB thigh_l_calf_l和VB thigh_r_calf_r；<img src='https://img-blog.csdnimg.cn/20201116171006740.png'>
3. 添加Pose：LocalPose—>FootIKPose

- pelvis

  <img src='https://img-blog.csdnimg.cn/20201116172111319.png'>

- ik_foot_l & VB thigh_l_calf_l

  <img src='https://img-blog.csdnimg.cn/20201116172150724.png'><img src='https://img-blog.csdnimg.cn/20201116172216387.png'>

- ik_foot_r & VB thigh_r_calf_r

  <img src='https://img-blog.csdnimg.cn/20201116172256840.png'><img src='https://img-blog.csdnimg.cn/2020111617232282.png'>

<img src='https://img-blog.csdnimg.cn/20201116172648890.png'>

4. 最后按布尔输出混合姿势
<img src='https://img-blog.csdnimg.cn/20201116172617594.png'>  

# 最终效果展示

## 添加前Shinbi右脚悬空

<img src='https://img-blog.csdnimg.cn/20201116152721533.png'>

## 添加后Shinbi右脚落地

<img src='https://img-blog.csdnimg.cn/20201116160346496.png'>