
**请给出下面问题的一个实现，并输出结果**

目标是解析log并计算**每一支股票的**预测值的`r_squared`（定义：[Wiki: Coefficient of determination](https://en.wikipedia.org/wiki/Coefficient_of_determination)）。

有一组数据，记录了深圳A股的行情切片，以及某种模型对其未来变化做出的预测，样例如下：

    DEBUG 20230614 093013.845941 50994 | #2208046107 MD|I=300410.SZ,ETS=09:30:07.940,L=8.19,V=95900,B1=8.15:24700,A1=8.19:300,B2=8.14:10500,A2=8.2:1500,B3=8.13:42400,A3=8.21:200,B4=8.12:3600,A4=8.22:5100,B5=8.11:2900,A5=8.23:9500,T=782513.000000 - app_aid_report.c:383
    DEBUG 20230614 093013.845966 50994 | ##2208046107 300410.SZ prediction=0.21052289, raw=0.21052289, theo0=8.17172, bp=8.0927331,sp=8.2561675,q_bp=7.3572783,q_sp=8.9916223,bv1=0,bv2=0,av1=0,av2=0,order_cnt=152,trade_cnt=102,cur_factor_idx=12 - app_aid_report.c:407
    DEBUG 20230614 093013.846028 50994 | #2208046108 MD|I=301287.SZ,ETS=09:30:07.940,L=55.5,V=1031534,B1=55.3:400,A1=55.66:300,B2=55.15:100,A2=56:25770,B3=55.14:100,A3=56.01:100,B4=55.12:493,A4=56.02:500,B5=55.11:100,A5=56.06:100,T=56842643.760000 - app_aid_report.c:383
    DEBUG 20230614 093013.846055 50994 | ##2208046108 301287.SZ prediction=0.70498127, raw=0.70498127, theo0=55.5191, bp=54.963922,sp=56.074305,q_bp=49.967202,q_sp=61.071025,bv1=0,bv2=0,av1=0,av2=0,order_cnt=3,trade_cnt=0,cur_factor_idx=93 - app_aid_report.c:407

记其中`prediction`为预测值，并定义真值如下：

```
    mt = (A1.AtTime(t) + B1.AtTime(t)) / 2;
    mtn = (A1.AtTime(t + n) + B1.AtTime(t + n)) / 2;
    y = (mtn - mt) / mt * 1000
```

其中`A1,B1`为卖一、买一价，`ETS`为当前时间，`n`为参数，这里设定为`30s`。真实值`y`也可表述为“未来`n`秒后的中间价变化，单位为0.1%”。

数据存储在Azure storage blob，SAS共享访问签名URL：

```
https://ivodatascience.blob.core.windows.net/interview-log-parsing?sv=2021-10-04&st=2023-09-19T02%3A06%3A59Z&se=2024-09-20T02%3A06%3A00Z&sr=c&sp=rl&sig=KQxDOq1jwpVSS5LBfXeGeeI%2FfmuVMr0pBYjYP3QIAUU%3D
```

在Azure Storage Explorer中访问的方法：连接->选择资源：Blob容器或目录->共享访问签名URL(SAS)->填入上面的地址。

注意事项：

1. “#”和“##”后的数字可以确保行情与预测值时间点的对应，log本身不确保两种记录的先后顺序（每种记录各自有序）
2. `A1 - B1 <= 0`的情况可以作为异常值丢弃
3. `n`秒后的行情，取距离预定时间最近的点即可
4. 运行时间加分（语言，并发，复杂度，tricks）
5. C# 实现加分
6. 低内存需求加分
7. Azure storage 相关可以查阅官方文档，数据样本可以用 [Azure Storage Explorer](https://azure.microsoft.com/en-us/features/storage-explorer/)来查看
