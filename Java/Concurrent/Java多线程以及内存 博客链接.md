# 概述
并发三大定律：
1. Amdahl定律. 给定问题规模，可并行化部分占12%，那么即使把并行运用到极致，系统的性能最多也只能提高1/(1-0.12)=1.136倍。即：并行对提高系统性能有上限。
2. Gustafson定律. Gustafson定律说Amdahl定律没有考虑随着cpu的增多而有更多的计算能力可被使用。其本质在于更改问题规模从而可以把Amdahl定律中那剩下的88%的串行处理并行化，从而可以突破性能门槛。本质上是一种空间换时间。
3. Sun-Ni定律. 是前两个定律的进一步推广。其主要思想是计算的速度受限于存储而不是CPU的速度. 所以要充分利用存储空间等计算资源，尽量增大问题规模以产生更好/更精确的解.

# 文章
- http://www.infoq.com/cn/articles/memory_barriers_jvm_concurrency/（内存屏障与JVM并发）
- javap反编译后的指令详解：http://hubingforever.blog.163.com/blog/static/17104057920117822653653/
- JAVA多线程与并发学习总结:http://www.cnblogs.com/yshb/archive/2012/06/15/2550367.html
- java并发：http://dapple.iteye.com/blog/787563 【值得一看，通俗易懂】
  Java并发程序设计教程:http://files.cnblogs.com/files/jobs/Java%E5%B9%B6%E5%8F%91%E7%A8%8B%E5%BA%8F%E8%AE%BE%E8%AE%A1%E6%95%99%E7%A8%8B-2010-08-10.pdf
- java工程师成神之路：http://www.hollischuang.com/archives/489
- 网站架构演化：http://www.cnblogs.com/GmrBrian/p/3777076.html