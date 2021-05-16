
## 子节点 diff

~~~js

  function updateChildren (parentElm, oldCh, newCh, insertedVnodeQueue, removeOnly) {
    let oldStartIdx = 0
    let newStartIdx = 0
    let oldEndIdx = oldCh.length - 1
    let oldStartVnode = oldCh[0]
    let oldEndVnode = oldCh[oldEndIdx]
    let newEndIdx = newCh.length - 1
    let newStartVnode = newCh[0]
    let newEndVnode = newCh[newEndIdx]
    let oldKeyToIdx, idxInOld, vnodeToMove, refElm

    // removeOnly is a special flag used only by <transition-group>
    // to ensure removed elements stay in correct relative positions
    // during leaving transitions
    const canMove = !removeOnly

    if (process.env.NODE_ENV !== 'production') {
      checkDuplicateKeys(newCh)
    }

    while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
      if (isUndef(oldStartVnode)) {
        oldStartVnode = oldCh[++oldStartIdx] // Vnode has been moved left
      } else if (isUndef(oldEndVnode)) {
        oldEndVnode = oldCh[--oldEndIdx]
      } else if (sameVnode(oldStartVnode, newStartVnode)) {
        patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
        oldStartVnode = oldCh[++oldStartIdx]
        newStartVnode = newCh[++newStartIdx]
      } else if (sameVnode(oldEndVnode, newEndVnode)) {
        patchVnode(oldEndVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx)
        oldEndVnode = oldCh[--oldEndIdx]
        newEndVnode = newCh[--newEndIdx]
      } else if (sameVnode(oldStartVnode, newEndVnode)) { // Vnode moved right
        patchVnode(oldStartVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx)
        canMove && nodeOps.insertBefore(parentElm, oldStartVnode.elm, nodeOps.nextSibling(oldEndVnode.elm))
        oldStartVnode = oldCh[++oldStartIdx]
        newEndVnode = newCh[--newEndIdx]
      } else if (sameVnode(oldEndVnode, newStartVnode)) { // Vnode moved left
        patchVnode(oldEndVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
        canMove && nodeOps.insertBefore(parentElm, oldEndVnode.elm, oldStartVnode.elm)
        oldEndVnode = oldCh[--oldEndIdx]
        newStartVnode = newCh[++newStartIdx]
      } else {
        if (isUndef(oldKeyToIdx)) oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx)
        idxInOld = isDef(newStartVnode.key)
          ? oldKeyToIdx[newStartVnode.key]
          : findIdxInOld(newStartVnode, oldCh, oldStartIdx, oldEndIdx)
        if (isUndef(idxInOld)) { // New element
          createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx)
        } else {
          vnodeToMove = oldCh[idxInOld]
          if (sameVnode(vnodeToMove, newStartVnode)) {
            patchVnode(vnodeToMove, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
            oldCh[idxInOld] = undefined
            canMove && nodeOps.insertBefore(parentElm, vnodeToMove.elm, oldStartVnode.elm)
          } else {
            // same key but different element. treat as new element
            createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx)
          }
        }
        newStartVnode = newCh[++newStartIdx]
      }
    }
    if (oldStartIdx > oldEndIdx) {
      refElm = isUndef(newCh[newEndIdx + 1]) ? null : newCh[newEndIdx + 1].elm
      addVnodes(parentElm, refElm, newCh, newStartIdx, newEndIdx, insertedVnodeQueue)
    } else if (newStartIdx > newEndIdx) {
      removeVnodes(oldCh, oldStartIdx, oldEndIdx)
    }
  }
~~~

### 四个指针

* `oldCh` 保存旧的虚拟子节点
* `newCh` 保存新的虚拟子节点

* `oldStartIdx = 0` 指向旧子节点 从前往后遍历
* `oldEndIdx = oldCh.length - 1` 指向旧的子节点 从后往前遍历
* `newStartIdx = 0` 指向新的子节点 从前往后遍历
* `newEndIdx = newCh.length - 1` 指向新的子节点 从后往前遍历

* `oldStartVnode = oldCh[0]`
  `oldEndVnode = oldCh[oldEndIdx]`
  `newEndIdx = newCh.length - 1`
  `newStartVnode = newCh[0]`
  `newEndVnode = newCh[newEndIdx]`

### 五种场景

* `patchVnode(oldStartVnode, newStartVnode)` `oldStartVnode` 和`newStartVnode` 是可以复用的虚拟节点 `path` 后`oldStartIdx++`, `newStartIdx++` 。
* `patchVnode(oldEndVnode, newEndVnode)` `oldEndVnode 和 newEndVnode` 是可以复用的虚拟的节点 `path` 后 `oldEndIdx--` , `newEndIdx--`。
* `patchVnode(oldStartVnode, newEndVnode)` 这种场景说明`oldStartVnode`对应的真实`dom`节点已经移动到了当前的末尾即`oldEndIdx` 之后 所以需要执行`nodeOps.insertBefore(parentElm, oldStartVnode.elm, nodeOps.nextSibling(oldEndVnode.elm))` 将节点插入响应的位置  `insertBefore` 其实就是调用的`node.insertBefore`。
* `patchVnode(oldEndVnode, newStartVnode)` 这种情况说明已`oldEndVnode`对应的真实`dom`节点已经移动到了当前节点的前面同样需要执行`nodeOps.insertBefore(parentElm, oldEndVnode.elm, oldStartVnode.elm)`将节点插入到对应的位置。
* 最后一种情况 使用`key` `path`

循环执行经过以上`步骤`
如果当前新的子树节点还有没被遍历到的节点即(oldStartIdx > oldEndIdx)则这些节点都是新增的节点执行`addVnodes(parentElm, refElm, newCh, newStartIdx, newEndIdx, insertedVnodeQueue)`
如果当前旧的子树节点中还有没被遍历到节点即(newStartIdx > newEndIdx)则这些节点都是需要移除的节点

### `key`的作用

如果前四种场景都没有匹配则会使用`key`寻找可复用的节点 步骤大致如下

1. 创建一个 `oldKey` 到`index`的映射`oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx)`大致类似这样的对象{ key1: 1, key2: 2 }。
2. 如果存在`newStartVnode.key`寻找可复用的节点即`oldIndex = oldKeyToIdx[newStartVnode.key]`
3. 如果不存在`newStartVnode.key` 则需要遍历子树剩下的所有节点寻找可复用节点，所以这里就体现出`key`的重要性了。
4. 如果找到可复用节点则执行`path` 执行完`path` 后同样需要整理`nodeOps.insertBefore(parentElm, vnodeToMove.elm, oldStartVnode.elm)`, 
5. 否则当做新节点处理。