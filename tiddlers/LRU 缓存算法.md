LRU，是一种缓存算法，比较简单，方便实现，之前有一次面试的时候，面试官出了这道题。

Least Recently Used，最近最少使用。一般用来做缓存，或者页面置换。结题思路就是维护一个哈希表和双向链表，用于满足快速查找和随机删除的需求。



可以做下 [146. LRU 缓存 - 力扣（Leetcode）](https://leetcode.cn/problems/lru-cache/description/) 练习下，参考 [算法就像搭乐高：带你手撸 LRU 算法 :: labuladong的算法小抄 (gitee.io)](https://labuladong.gitee.io/algo/di-yi-zhan-da78c/shou-ba-sh-daeca/suan-fa-ji-03840/)

我这里为了练习，就自己实现了一个双向链表，c# 是有现成的 LinkedList，可以直接使用。

```c#
public class LRUCache {

    private int capacity;
    private LinkedList cacheList;
    private Dictionary<int, LinkedNode> dict;

    public LRUCache(int capacity) {
        this.capacity = capacity;

        cacheList = new LinkedList();
        dict = new Dictionary<int, LinkedNode>();
    }
    
    public int Get(int key) 
    {
        if(dict.ContainsKey(key))
        {
            var node = dict[key];

            cacheList.RemoveNode(node);
            cacheList.AddFirst(node);

            return node.value;
        }
        else
        {
            return -1;
        }
    }
    
    public void Put(int key, int value) {

        if(dict.ContainsKey(key))
        {
            var node = dict[key];

            cacheList.RemoveNode(node);
            node.value = value;
            cacheList.AddFirst(node);
        }
        else
        {
            var node = new LinkedNode(key, value);

            dict[key] = node;
            cacheList.AddFirst(node);
        }

        while(cacheList.Size > capacity)
        {
            var last = cacheList.Last();
            dict.Remove(last.key);
            cacheList.RemoveLast();
        }
    }
}

public class LinkedNode
{
    public LinkedNode Pre;
    public LinkedNode Next;
    public int key;
    public int value;

    public LinkedNode(int key, int value)
    {
        this.key = key;
        this.value = value;
    }
}

public class LinkedList
{
    private LinkedNode Head;
    private LinkedNode Tail;
    private int nodeCount;
    public int Size => nodeCount;

    public LinkedList()
    {
        Head = new LinkedNode(0, 0);
        Tail = new LinkedNode(0, 0);

        Head.Pre = null;
        Head.Next = Tail;

        Tail.Pre = Head;
        Tail.Next = null;

        nodeCount = 0;
    }

    public LinkedNode Last()
    {
        return Tail.Pre;
    }

    public LinkedNode First()
    {
        return Head.Next;
    }

    public void AddFirst(LinkedNode node)
    {
        AddNode(Head, Head.Next, node);
    }

    public void AddLast(LinkedNode node)
    {
        AddNode(Tail, Tail.Pre, node);
    }

    private void AddNode(LinkedNode pre, LinkedNode next, LinkedNode node)
    {
        node.Next = next;
        node.Pre = pre;

        pre.Next = next.Pre = node;

        nodeCount++;
    }

    public void RemoveFirst()
    {
        if(nodeCount > 0)
        {
            RemoveNode(Head.Next);
        }
    }

    public void RemoveLast()
    {
        if(nodeCount > 0)
        {
            RemoveNode(Tail.Pre);
        }
    }

    public void RemoveNode(LinkedNode node)
    {
        node.Pre.Next = node.Next;
        node.Next.Pre = node.Pre;

        nodeCount--;
    }
}

/**

 * Your LRUCache object will be instantiated and called as such:
 * LRUCache obj = new LRUCache(capacity);
 * int param_1 = obj.Get(key);
 * obj.Put(key,value);
 */
```

