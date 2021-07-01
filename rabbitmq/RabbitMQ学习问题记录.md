# RabbitMQ学习问题记录

1. 修改vm_memory_high_watermark_paging_ratio后，是否重启rabbitmq才会生效？

   ```shell
   # 此值大于1时，相当于禁用了换页功能。
   vm_memory_high_watermark_paging_ratio = 0.75
   ```

   > 是的

2. 调整磁盘阈值使用 `mem_relative` 是否指相对于磁盘的比例大小？

   ```shell
   # fraction 为相对比值，建议的取值为1.0~2.0之间
   rabbitmqctl set_disk_free_limit mem_relative <fraction>
   ```

   

3. 如何描述RabbitMQ的消息分发机制？

   

4. header exhange使用？

5. fanout exhange使用？

6. 集群模式下，如果用代码使用？应该连接哪一个节点？

7. 什么是有状态？什么是无状态？

   * 应用可以启动多个
   * 访问每个应用都能得到相同的结果

   > 有状态的应用：Eureka，zookeeper，mysql
   >
   > 有状态的应用不建议水平扩展

8. rabbitmq重连代码研究

9. configure regexp , write regexp, read regexp

10. 

