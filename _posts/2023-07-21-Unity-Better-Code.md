## Unity中一些代码相关的优化

- Send Message

  有的人喜欢用Send Message去快速执行某个方法，确实很便捷，单帧内调用少量次数看不太出来，如果大体量的调用就比较消耗了

  当数量级达到几十万以上后可以发现，单纯的GetComponent方法获取组件再去执行某个函数要比直接SendMessage调用执行函数快超出两倍的速度

  

  

- Collider

  Physics.OverlapSphere在运行中是要比Physics.OverlapSphereNonAlloc快约一倍，但是存在的问题是Physics.OverlapSphere会产生内存垃圾需回收，而Physics.OverlapSphereNonAlloc不会产生垃圾，具体在使用上看情况使用

  

- 循环

  如果整个循环中需要多次访问数组的索引,foreach会更快，如果只访问一次，那么for循环要快两倍或者更多，for循环时，其长度最好用一个int临时变量来代替，这样也更加的有效率

  

- string和stringbuilder

  当有需要累计处理大量文本时，建议用StringBuilder，而非用string+="xxxx"来做处理，因为string+=每次都要重新分配内存，StringBuilder相比string+=大约快有8-9倍的效率。

  

  

- Find

  FindObjectsOfType每次寻找会便利所有的物体身上所拥有的所有组件，非常浪费时间

  GameObject.FindGameObjectsWithTag则只需要找相关标签即可，不同情况下能通过Tag 找到的目标尽量别用FindObjectsOfType，实在没办法没有Tag那就另当别

  

- 乘法运算

  一般我们做乘法运算时，习惯性的会将Vector3值写在最前面

  比如Vector3.forward * SpeedTreeImporter * Time.deltaTime 特别是计算移动旋转什么的

  其实只需要调换一下位置 SpeedTreeImporter * Time.deltaTime * Vector3.forward 就可以使其更快的得到结果

  

  

  

  

  

  

  

  



