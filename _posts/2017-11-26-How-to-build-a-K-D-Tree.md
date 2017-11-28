---
layout: post
title: "How to build a K-D Tree"
date:   2017-11-26
categories: algorithm RVO
permalink: how to build a k-d tree

# author
author: Ahn
---

# K = 2
# 定义 2-D Tree 的结构 
## 定义用于比较的关系式结构
	
	private struct FloatPair
    {
        private float a_;
        private float b_;

		/**
         * <summary>Constructs and initializes a pair of scalar
         * values.</summary>
         *
         * <param name="a">The first scalar value.</returns>
         * <param name="b">The second scalar value.</returns>
         */

        internal FloatPair(float a, float b)
        {
            a_ = a;
            b_ = b;
        }
	}
相关运算符操作

	/**
         * <summary>Returns true if the first pair of scalar values is less
         * than the second pair of scalar values.</summary>
         *
         * <returns>True if the first pair of scalar values is less than the
         * second pair of scalar values.</returns>
         *
         * <param name="pair1">The first pair of scalar values.</param>
         * <param name="pair2">The second pair of scalar values.</param>
         */

        public static bool operator <(ValuePair pair1, ValuePair pair2)
        {
            return pair1.a_ < pair2.a_ || !(pair2.a_ < pair1.a_) && pair1.b_ < pair2.b_;
        }

        /**
         * <summary>Returns true if the first pair of scalar values is less
         * than or equal to the second pair of scalar values.</summary>
         *
         * <returns>True if the first pair of scalar values is less than or
         * equal to the second pair of scalar values.</returns>
         *
         * <param name="pair1">The first pair of scalar values.</param>
         * <param name="pair2">The second pair of scalar values.</param>
         */

        public static bool operator <=(ValuePair pair1, ValuePair pair2)
        {
            return (pair1.a_ == pair2.a_ && pair1.b_ == pair2.b_) || pair1 < pair2;
        }

        /**
         * <summary>Returns true if the first pair of scalar values is
         * greater than the second pair of scalar values.</summary>
         *
         * <returns>True if the first pair of scalar values is greater than
         * the second pair of scalar values.</returns>
         *
         * <param name="pair1">The first pair of scalar values.</param>
         * <param name="pair2">The second pair of scalar values.</param>
         */

        public static bool operator >(ValuePair pair1, ValuePair pair2)
        {
            return !(pair1 <= pair2);
        }

        /**
         * <summary>Returns true if the first pair of scalar values is
         * greater than or equal to the second pair of scalar values.
         * </summary>
         *
         * <returns>True if the first pair of scalar values is greater than
         * or equal to the second pair of scalar values.</returns>
         *
         * <param name="pair1">The first pair of scalar values.</param>
         * <param name="pair2">The second pair of scalar values.</param>
         */

        public static bool operator >=(ValuePair pair1, ValuePair pair2)
        {
            return !(pair1 < pair2);
        }

# K-D树中节点有两种（关于障碍物的节点和关于Agent的节点）
## 关于障碍物的节点的定义
	private class ObstacleTreeNode
    {
        internal OrcaObstacle Obstacle;
        internal ObstacleTreeNode Left;
        internal ObstacleTreeNode Right;
    }
## 递归构建障碍物的 K-D Tree

循环遍历 **Obstacles** 中的所有 **Obstacle**，取其 `ObstacleI1` 和 `ObstacleI2 = ObObstacleI1.Next`，再循环遍历**Obstacles** 中的所有 **Obstacle**，取`ObstacleJ1` 和 `ObstacleJ2 = ObstacleJ1.Next`，判断 `ObstacleJ1` 和 `ObstacleJ2` 分别在由 **ObstacleI1** 和 **ObstacleI2** 组成的 segment 的左边还是右边，进而确定每一颗子树节点的个数，最后保证这是一颗平衡树
 	
	for (int i = 0; i < obstacles.Count; i++)
    {
        int leftSize = 0;
        int rightSize = 0;

        OrcaObstacle obstacleI1 = obstacles[i];
        OrcaObstacle obstacleI2 = obstacleI1.Next;

        for (int j = 0; j < obstacles.Count; j++)
        {
            if (i == j)
            {
                continue;
            }

            OrcaObstacle obstacleJ1 = obstacles[j];
            OrcaObstacle obstacleJ2 = obstacleJ1.Next;

            float j1LeftOfI = MathUtil.LeftOf(obstacleI1.Point, obstacleI2.Point, obstacleJ1.Point);
            float j2LeftOfI = MathUtil.LeftOf(obstacleI1.Point, obstacleI2.Point, obstacleJ2.Point);

            if (j1LeftOfI >= -MathUtil.RVO_EPSILON && j2LeftOfI >= -MathUtil.RVO_EPSILON)
            {
                leftSize++;
            }
            else if (j1LeftOfI <= MathUtil.RVO_EPSILON && j2LeftOfI <= MathUtil.RVO_EPSILON)
            {
                rightSize++;
            }
            else
            {
                leftSize++;
                rightSize++;
            }
			
			/* 只要左子树节点的个数或者右子树节点的个数 大于 左边或者右边可以拥有的个数时 跳出循环 */
            if (new FloatPair(Math.Max(leftSize, rightSize), Math.Min(leftSize, rightSize)) >= new FloatPair(Math.Max(minLeft, minRight), Math.Min(minLeft, minRight)))
            {
                break;
            }
        }

		/*判断如果是跳出循环的，去更新 minLeft, minRight, optimalSplit*/
        if (new FloatPair(Math.Max(leftSize, rightSize), Math.Min(leftSize, rightSize)) < new FloatPair(Math.Max(minLeft, minRight), Math.Min(minLeft, minRight)))
        {
            minLeft = leftSize;
            minRight = rightSize;
            optimalSplit = i;
        }
    }

再找到最合适的上面三个值之后，需要去构建 **Split Node**

下标为 i 的Obstacle 正好是接下来所去构建的 K-D Tree 的顶点 `OrcaObstacle obstacleI1 = obstacles[i];`，接下来需要去以构建 **ObstacleI1** 为顶点的左右子树

**ObstacleI1** 的 **Next** 节点 `OrcaObstacle obstacleI2 = obstacleI1.Next;`，依次遍历 **Obstacles** 中的除去 **ObstacleI1** 的其他节点 `OrcaObstacle obstacleJ1 = obstacles[j];` 及其 **Next** 节点，`OrcaObstacle obstacleJ2 = obstacleJ1.Next;`

判断 **obstacleJ1** 和 **obstacleJ2** 在由 **obstacleI1** 和 **obstacleI2** 组成的 Segment 的左边还是右边（判断的方法还是 行列式），得到的值分别为 **j1LeftOfI**、**j2LeftOfI**

如果 **j1LeftOfI**、**j2LeftOfI** 都大于 **-RVOMath.RVO_EPSILON** (RVOMath.RVO_EPSILON = 0.00001f)，则将 **obstacles[j]** 加到左子树上
	
	if (j1LeftOfI >= -RVOMath.RVO_EPSILON && j2LeftOfI >= -RVOMath.RVO_EPSILON)
    {
        leftObstacles[leftCounter++] = obstacles[j];
    }

如果 **j1LeftOfI**、**j2LeftOfI** 都小于 **RVOMath.RVO_EPSILON**，则将 **obstacles[j]** 加到右子树上
	
	else if (j1LeftOfI <= RVOMath.RVO_EPSILON && j2LeftOfI <= RVOMath.RVO_EPSILON)
    {
        rightObstacles[rightCounter++] = obstacles[j];
    }

若上面两种情况都不满足，则会去构建 **Split Node**，如何去确定 **Split Node** 的位置(对比行列式)
	
	float t = RVOMath.det(obstacleI2.Point - obstacleI1.Point, obstacleJ1.Point - obstacleI1.Point) / RVOMath.det(obstacleI2.Point - obstacleI1.Point, obstacleJ1.Point - obstacleJ2.Point);
    Vector2 splitPoint = obstacleJ1.Point + t * (obstacleJ2.Point - obstacleJ1.Point);

然后构造新的 Obstacle

	Obstacle newObstacle = new Obstacle();
    newObstacle.Point = splitPoint;
    newObstacle.Previous = obstacleJ1;
    newObstacle.Next = obstacleJ2;
    newObstacle.Convex = true;
    newObstacle.Direction = obstacleJ1.Direction;

    newObstacle.Id = Simulator.Instance.obstacles_.Count;

    Simulator.Instance.obstacles_.Add(newObstacle);

    obstacleJ1.Next = newObstacle;
    obstacleJ2.Previous = newObstacle;

再根据 **j1LeftOfI** 与 0 的关系，确定将 **obstacleJ1** 和 **newObstacle** 加到左子树或者右子树上

	if (j1LeftOfI > 0.0f)
    {
        leftObstacles[leftCounter++] = obstacleJ1;
        rightObstacles[rightCounter++] = newObstacle;
    }
    else
    {
        rightObstacles[rightCounter++] = obstacleJ1;
        leftObstacles[leftCounter++] = newObstacle;
    }

最后将 **leftObstacles** 和 **rightObstacles** 递归调用去构建完整的 **2-D Tree**

## 通过递归去查找 K-D Tree
获取当前 **ObstacleTreeNode** 对应的 **Obstacle** 及其 **Next**

	Obstacle obstacle1 = node.obstacle_;
    Obstacle obstacle2 = obstacle1.Next;

判断当前 **agent** 是在由 **obstacle1** 和 **obstacle2** 组成的 segment 的左边还是右边，由此来决定是继续从左子树递归还是右子树递归

	float agentLeftOfLine = RVOMath.leftOf(obstacle1.Point, obstacle2.Point, agent.position_);
    queryObstacleTreeRecursive(agent, rangeSq, agentLeftOfLine >= 0.0f ? node.left_ : node.right_);

判断 **agent** 到由 **obstacle1** 和 **obstacle2** 组成的 segment 的距离 **distSqLine** 与检测范围 **rangeSq** 的大小，只有当满足: 

- 距离小于检测范围 （distSqLine < rangeSq）
- 在 **Obstacle** 的右边 （agentLeftOfLine < 0.0f）
 
才会将这个 **Obstacle** 视为该 **agent** 的 **Obstacle Neighbor**，否则继续递归查找

	float distSqLine = RVOMath.sqr(agentLeftOfLine) / RVOMath.absSq(obstacle2.Point - obstacle1.Point);

    if (distSqLine < rangeSq)
    {
        if (agentLeftOfLine < 0.0f)
        {
            /*
             * Try obstacle at this node only if agent is on right side of
             * obstacle (and can see obstacle).
             */
            agent.insertObstacleNeighbor(node.obstacle_, rangeSq);
        }

        /* Try other side of line. */
        queryObstacleTreeRecursive(agent, rangeSq, agentLeftOfLine >= 0.0f ? node.right_ : node.left_);
    }

## 整理 ObstacleNeighbors
在插入到 **obstacle neighbors** 时，需要保证到是按照 **agent** 到各个 **Obstacle** 及其 **Next** 组成的 segment 的距离由小到大排序 
	
	internal void insertObstacleNeighbor(Obstacle obstacle, float rangeSq)
    {
        Obstacle nextObstacle = obstacle.Next;

        float distSq = RVOMath.distSqPointLineSegment(obstacle.Point, nextObstacle.Point, position_);

        if (distSq < rangeSq)
        {
            obstacleNeighbors_.Add(new KeyValuePair<float, Obstacle>(distSq, obstacle));

            int i = obstacleNeighbors_.Count - 1;

            while (i != 0 && distSq < obstacleNeighbors_[i - 1].Key)
            {
                obstacleNeighbors_[i] = obstacleNeighbors_[i - 1];
                --i;
            }
            obstacleNeighbors_[i] = new KeyValuePair<float, Obstacle>(distSq, obstacle);
        }
    }