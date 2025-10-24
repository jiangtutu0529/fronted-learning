### 二叉树
定义：最多有两个节点，左子节点和右子节点

```
class TreeNode {
    constructor(value) {
        this.value = value;
        this.left = null;
        this.right = null;
    }
}
```

二叉树的类型：
满二叉树：度只有0和度只有2的二叉树，深度为k,有2^k-1个结点，每一层的结点数都达到最大值
       1// 深度为 3 的满二叉树，共有 2^3 - 1 = 7 个结点
      / \
     2   3
    / \ / \
   4  5 6  7
完全二叉树：  深度为 k 的二叉树，其前 (k-1) 层是满的，且第 k 层的结点都​​从左到右连续排列​​
       1 // 这是一个完全二叉树
      / \
     2   3
    / \ /
   4  5 6

       1       // 这不是完全二叉树，因为第三层的结点6和7不连续
      / \
     2   3
    /     \
   4       7
平衡二叉树：一棵二叉树中，​​任意结点​​的​​左子树和右子树的高度差的绝对值不超过 1​​。空树的高度定义为 -1 或 0（定义不同，但思想一致）
// 平衡二叉树示例：每个结点左右子树高度差 <= 1
       1
      / \
     2   3
    / \
   4   5

// 不平衡二叉树示例：根结点1的左子树高度为2，右子树高度为0，高度差为2
       1
      /
     2
    / \
   4   5
二叉搜索树：
也称为二叉排序树。它或者是一棵空树，或者是具有下列性质的二叉树：
若它的左子树不空，则​​左子树上所有结点的值均小于​​它的根结点的值。
若它的右子树不空，则​​右子树上所有结点的值均大于​​它的根结点的值。
它的左、右子树也分别为二叉搜索树
// 一个有效的二叉搜索树
       5
      / \
     3   7
    / \ / \
   2  4 6  8
// 中序遍历结果: 2, 3, 4, 5, 6, 7, 8 (有序)

##### 创建二叉树：
const root = new TreeNode(1)
root.left = new TreeNode(2)
root.right = new TreeNode(3)

##### 二叉树遍历：
一、深度遍历：
前序遍历：
function preorderTraverse(root){
  const res = []
  function traverse(params){
    if(!params) return 
    res.push(params.value)
    traverse(params.left)
    traverse(params.right)
  }
  traverse(root)
  return res
}
中序遍历：
function preorderTraverse(root){
  const res = []
  function traverse(params){
    if(!params) return 
    traverse(params.left)
    res.push(params.value)
    traverse(params.right)
  }
  traverse(root)
  return res
}
后序遍历：
function preorderTraverse(root){
  const res = []
  function traverse(params){
    if(!params) return 
    traverse(params.left)
    traverse(params.right)
    res.push(params.value)
  }
  traverse(root)
  return res
}

二、广度遍历：
```
function levelTraverse(root){
  if(!root) return []
  const res = []
  const queue = [root]
  while(queue.length>0){
    const curLevel = []
    for(let i = 0;i<queue.length;i++){
      const curNode = queue.shift()
      curLevel.push(curNode.value)
      if(curNode.left){
        queue.push(curNode.left)
      }
      if(curNode.right){
        queue.push(curNode.right)
      }
    }
    res.push(curLevel)
  }
  return res
}
```
三、二叉树
```
Class BinaryTree {
  construtor(){
    this.root = null
  }
  insert(val){
    const curNode = new TreeNode(val)
    if(!this.root){
      this.root = curNode
      return
    }
    let current = this.root
    while(true){
      if(val<current.val){
        if(!current.left){
          currenr.left = current
          return 
        }else[
          current = current.next
        ]
      }else{
        if(!currrent.right){
          current.right = current
          return
        }else{
          current = current.right
        }
      }
    }
  }
  getHeight(node = this.root){
    if(!node) return 0
    return 1+Math.max(this.getHeight(node.left),this.getHeight(ndoe.right))
  }
  getSize(node = this.root){
    if(!node) return 0
    return 1+this.getSize(node.left)+this.getSize(node.right)
  }
  search(value){
    let current = this.root
    while(current){
      if(value<current.val){
        current = current.left
      }
      if(value > current.val){
        current = current.right
      }
      if(value === current.val){
        return current
      }
    }
    
    return null
  }
  swap(node = this.root){
    if(!node) return node
    let temp = node.left
    node.left = node.right
    node.right = temp
    swap(node.left)
    swap(node.right)
  }
}
```

