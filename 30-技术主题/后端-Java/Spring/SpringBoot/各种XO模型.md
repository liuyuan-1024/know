# SpringBoot中的各种模型


<!-- 这是一张图片，ocr 内容为：展示层 页面 页面 页面 VO VO VO 业务解释 业务解释 业务解释 DTO 业务逻辑层 业务处理 BO 数据访问层 PO PO PO DAO 数据库 知乎@史墨轩 -->
![](https://cdn.nlark.com/yuque/0/2024/webp/43050341/1710504939918-556b0206-826c-4e0f-92ce-7ecd1743f7cb.webp)

## DTO


> Data Transfer Object 数据传输对象
>

## VO


> Value Object  值对象
>
>  
>
> VO就是展示用的数据，不管展示方式是网页，还是客户端，还是APP，只要是这个东西是让人看到的，这就叫VO；  
VO主要的存在形式就是js里面的对象（也可以简单理解成json）
>

## VO和DTO的区别


主要有两个区别  
一个是字段不一样，VO根据需要会删减一些字段  
另一个是值不一样，VO会根据需要对DTO中的值进行展示业务的解释

## PO


> Persistant Object  持久对象
>
>  
>
> 简单说PO就是数据库中的记录，一个PO的数据结构对应着库中表的结构，表中的一条记录就是一个PO对象  
通常PO里面除了get，set之外没有别的方法  
对于PO来说，数量是相对固定的，一定不会超过数据库表的数量  
等同于Entity，这俩概念是一致的
>

## BO


> Business Object  业务对象
>
>  
>
> BO就是PO的组合
>

## DO


现在主要有两个版本



> 阿里巴巴的开发手册中的定义  
DO（ Da ta Object）====   PO
>



> DDD（Domain-Driven Design）领域驱动设计中  
DO（Domain Object）===  BO
>

