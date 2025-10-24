#### 一、链表
数据结构：
function ListNode(val,next){
  this.val = val === undefined?0:val
  this.next = next === undefined?null:next
}

链表遍历：
let cur = head
while(cur){
  cur = curnext
}

链表反转：
var reverseList = function(head) {
    let prev = null;
    let curr = head;
    while (curr) {
        const next = curr.next;
        curr.next = prev;
        prev = curr;
        curr = next;
    }
    return prev;
};

链表判断有环：
快慢指针
var hasCycle = function(head) {
    let slow = head, fast = head; // 乌龟和兔子同时从起点出发
    while (fast && fast.next) {
        slow = slow.next; // 乌龟走一步
        fast = fast.next.next; // 兔子走两步
        if (fast === slow) { // 兔子追上乌龟（套圈），说明有环
            return true;
        }
    }
    return false; // 访问到了链表末尾，无环
};


合并两个有序链表：
var mergeTwoLists = function(list1, list2) {
    if(list1 === null){
        return list2
    }
    if(list2 === null){
        return list1
    }
    let cur1 = list1
    let cur2 = list2
    let res = new ListNode()
    let curRes = res
    while(cur1||cur2){
        if(cur1 === null){
            curRes.next = cur2
            curRes = curRes.next
            cur2 = cur2.next
        }else  if(cur2 === null){
            curRes.next = cur1
            curRes = curRes.next
            cur1 = cur1.next
        }else  if(cur1.val<=cur2.val){
            curRes.next = cur1
            curRes = curRes.next
            cur1 = cur1.next
        }else if(cur1.val>cur2.val){
            curRes.next = cur2
            curRes = curRes.next
            cur2 = cur2.next
        }
    }
    return res.next
};

交换两个链表的值：
1、递归算法
var swapPairs = function(head) {
    if (head === null|| head.next === null) {
        return head;
    }
    const newHead = head.next;
    head.next = swapPairs(newHead.next);
    newHead.next = head;
    return newHead;
};

2、迭代
var swapPairs = function(head) {
    let cur = head
    let res = new ListNode(0,head)
    let resC = res
    while(cur){
        let p = cur
        cur = cur.next
        if(cur){
             let aa = cur.next
        cur.next = p
        p.next = aa
        resC.next = cur
        resC = resC.next.next
        cur = cur.next.next
        }else{
            return res.next
        }
       
    }
    return res.next
};
