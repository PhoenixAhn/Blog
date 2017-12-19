---
layout: post  
title: "如何生成 Agent 之间的 Orca Line"  
date:   2017-12-18  
categories: algorithm RVO

# author
author: Ahn
---

这篇文章主要解释的是 **RVO** 中运动的物体（**Agent**）如何基于相对位置和相对速度 生成对应的 **Orca Line**，其是一条用于约束 **Agent** 在当前帧的速度 的射线。


### 准备工作
 - 用于每个运动物体检测的时间片段为 **timeHorizon**，倒数为 **invTimeHorizon**  
 - 控制在相互碰撞后分开的平滑度也是由时间来控制 (**timeStep**), 倒数为 **invTimeStep**
 - 向量 **u** 用于辅助确定 **Line** 的 **Point** 值

###  Agent 的成员变量
 - **Position** 
 - **Velocity** 
 - **Radius**


### OrcaLine 的成员变量
 - **Direction**
 - **Point**

### 以两个运动的物体为例，如果 A 其在自己对运动障碍的检测范围中 检测到 B 的存在，则有一下关系式
相对位置: **Relative Position** `relativePosition = AgentB.Postion - AgentA.Position;`  
相对速度: **Relative Velocity** `relativeVelocity = AgentA.Velocity - AgentB.Velocity;`  
A 与 B 的半径和: **Combine Radius** `combineRadius = AgentA.Radius + AgentB.Radius;`
<!-- more -->

## No Collision
检测片段内的两个运动物体之间距离和检测时间片段的时长作为一个速度的参考量 **temp**：`Vector2 temp = invTimeHorizon * relativePosition`  
1. 做 **relativeVelocity** 与 **temp** 的向量差 **w**：`Vector2 w = relativeVelocity - invTimeHorizon * relativePosition;`  
2. 取 **w** 和 **relativePosition** 的点乘 `float dotProduct1 = Vector2.Dot(w, relativePosition);`  
3. **dotProduct1** 是用于比较 **w** 和 两个运动物体的半径和 **combineRadius**，若满足 `dotProduct1.sqrMagnitude > combineRadiusSq * wLengthSqr` （实际就是 **relativePosition** 在 **w** 方向上的投影长度 **大于** 两个运动物体的半径和），并且`dotProduct1 < .0f` ，此时投影在 Cut-off Circle 上，产生的对应的 **Line** 的 **Direction** 就是 **w** 的单位向量的向右的垂直向量，**u** 的值则用以下表达式：`u = (combineRadius * invTimeHorizon - w.magnitude) * unitW`  
4. 否则，即 `dotProduct >= .0f` 或 `dotProduct1.sqrMagnitude <= combineRadiusSq * wLengthSqr`，将投影在两条侧边上。侧边的长度计算为：以 **relativePosition** 为弦，**combineRadius** 为一条侧边。 `float leg = Math.Sqrt(distSq - combineRadiusSq);`  
5. 通过行列式判断 **w** 在 **relativePosition** 的左边或右边。 如果在投影在左侧边
`line.Direction = new Vector2(relativePosition.x * leg * ratio - relativePosition.y * combineRadius, relativePosition.x * combineRadius + relativePosition.y * leg * ratio) / distSq;` 如果投影在右侧边 `line.Direction = -new Vector2(relativePosition.x * leg * ratio + relativePosition.y * combineRadius, -relativePosition.x * combineRadius + relativePosition.y * leg * ratio) / distSq;`
6. 在5.的情况下，用 **dotProduct2** 表示 **relativeVelocity** 和 **line.Direction** 的点乘, **u** 的表达式为： `u = dotProduct2 * line.direction - relativeVelocity;`

### 用于表示在没有碰撞的情况下如何更改生成 Orca Line 的原理图
<pre class="highlight"><iframe scrolling="no" title="agents no collision" src="https://www.geogebra.org/material/iframe/id/JMvt64k8/width/1920/height/988/border/888888/smb/false/stb/false/stbh/false/ai/false/asb/false/sri/false/rc/false/ld/false/sdz/false/ctl/false" width="1200px" height="620px" style="border:0px;"> </iframe>
</pre>

## Collision
两个运动物体如果发生了碰撞，那么影响到它们相互如何推开的两个影响因素分别是：  
1. **relativeVelocity**  
2. **relativePosition**  

从 **Cut-off Center** 指向 **relative Velocity** 的向量 **w**为: `w = relativeVelocity - invTimeStep * relativePosition`，产生的对应的 **Line** 的 **Direction** 同样为 **w** 的单位向量的向右的垂直向量，**u** 的值则用以下表达式：`u = (combineRadius * invTimeStep - w.magnitude) * unitW`
 

