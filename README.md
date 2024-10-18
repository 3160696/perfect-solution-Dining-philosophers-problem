# -
问题描述：n个哲学家，m个碗，每两人中间一个筷子，同时有碗筷才能开吃，吃完还回  
  
While（1）{  
P（mutex） //检查拿起一气呵成  
If（有左筷子&&有右筷子&&有碗）{  
P（左筷子）；  
P（右筷子）；  
P（碗）；  
V（mutex）//如果能拿到的情况下，解锁mutex,允许其他进程“检查拿起一气呵成”  
Break  //跳出while循环，开始干饭  
}  
V（mutex）； //拿不到就解锁，允许其他进程“检查拿起一气呵成”  
P（左筷子）； //拿不到就自锁，做到让权等待。（而不是一直while等待）  
V（左筷子）； //实现了“仅自锁切换进程让权等待，而不改变任何数据”的效果  
P（右筷子）；  
V（右筷子）；  
P（碗）；  
V（碗）； //走完一趟While（1）说明大概率此时都有碗筷了，再次走就触发if-3P-break了  
}  
吃  
P（mutex_2）//还碗一气呵成  
V（左筷）  
V（右筷）  
V（碗）  
V（mutex_2）//还碗一气呵成  
  
//同时做到了以下几点：  
1、最多人能同时吃饭  
2、永不会发生死锁  
3、不会忙等，满足让权等待  
  
//解释：  
1、最多人能同时吃饭  
当有人无法同时拿到碗筷，立刻自锁，允许其他进程检查拿起  
  
2、永不会死锁  
解释方法1：纵观代码：两对mutex是为了一气呵成，不会死锁。while中的三对PV，没有连续P的情况，根本不可能死锁（只有连续的P才会死锁）。  
解释方法2：对于每个进程，只有一种阻塞情况（无法同时拿到碗筷），其所缺的碗筷，必然会有拿到的时候！（它所缺的碗筷有且仅有下列两种情况：1、在某个干饭人手里，干饭结束一定会还回去。 2、在while的三对PV中刚P成功被切换进程，切回去之后还是会V还回。）  
解释方法3：破坏了死锁的条件之一：请求保持 中的保持  
在while结尾的三对PV中，进程要么自锁（P）等待资源，要么被唤醒且持有资源一瞬间后立刻还回去（V），即：进程根本不会保持任何资源，即：破坏了请求保持条件，即：不会发生死锁。  
  
3、不会忙等，满足让权等待  
当无法同时拿到碗筷，会自锁（P），CPU将切换进程  
当能够拿到某碗筷而被唤醒之后，会立刻还回去该碗筷（V），进程只有同时能拿到碗筷的时候才是真正的P，真正的能拿到手里！  

//总结  
本伪代码完美解决哲学家问题
