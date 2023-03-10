
**请给出下面问题的一个实现，并输出结果**

目标是解析log并计算**每一支股票的**预测值的`r_squared`（定义：[Wiki: Coefficient of determination](https://en.wikipedia.org/wiki/Coefficient_of_determination)）。

有一组数据，记录了深圳A股的行情切片，以及某种模型对其未来变化做出的预测，样例如下：

    DEBUG 20200812 093021.573147 177626 | #2135219323 MD|I=300290.SZ,ETS=09:30:22.210,L=7.62,V=126100,B1=7.62:1000,A1=7.63:300,B2=7.6:700,A2=7.64:3000,B3=7.59:3900,A3=7.65:2000,B4=7.58:2000,A4=7.66:2500,B5=7.57:600,A5=7.67:20500,T=956446.000000 - app_aid_report.c:252
    DEBUG 20200812 093021.573171 177626 | ##2135219323 300290.SZ prediction=-0.91910517, raw=-0.91910517, theo0=0, bp=0,sp=0,q_bp=0,q_sp=0,bv1=0,bv2=0,av1=0,av2=0 - app_aid_report.c:267
    DEBUG 20200812 093021.573267 177626 | #2135219324 MD|I=002708.SZ,ETS=09:30:22.210,L=6.58,V=67400,B1=6.58:21300,A1=6.6:1300,B2=6.57:2500,A2=6.61:1000,B3=6.56:30200,A3=6.62:9700,B4=6.55:13800,A4=6.63:14600,B5=6.54:3500,A5=6.64:9100,T=445688.000000 - app_aid_report.c:252
    DEBUG 20200812 093021.573283 177626 | ##2135219324 002708.SZ prediction=0.74412155, raw=0.74412155, theo0=6.5949, bp=6.545442,sp=6.5784165,q_bp=6.5371984,q_sp=6.5866601,bv1=0,bv2=0,av1=21300,av2=0 - app_aid_report.c:267

记其中`prediction`为预测值，并定义真实值如下：

```
    mt = (A1.AtTime(t) + B1.AtTime(t)) / 2;
    mtn = (A1.AtTime(t + n) + B1.AtTime(t + n)) / 2;
    y = (mtn - mt) / mt * 1000
```

其中`A1,B1`为卖一、买一价，`ETS`为当前时间，`n`为参数，这里设定为`30s`。真实值`y`也可表述为“未来`n`秒后的中间价变化，单位为0.1%”。

数据按照~25M切分为若干个文件，文件编号也对应时间顺序，存储在Azure storage blob，连接字符串

```
DefaultEndpointsProtocol=https;AccountName=tmptask;AccountKey=bsNjOiTW35QqxGVm4qlBbQ+6E3zGA/IK1qV4VzOYUDJ3R3xqEyCFIvCTK0VvLR0nBNiv5kkPOKHkioqBVgk14g==;EndpointSuffix=core.chinacloudapi.cn
```

注意事项：

1. “#”和“##”后的数字可以确保行情与预测值时间点的对应，log本身不确保两种记录的先后顺序（每种记录各自有序）
2. `A1 - B1 <= 0`的情况可以作为异常值丢弃
3. `n`秒后的行情，取距离预定时间最近的点即可
4. 程序应包含IO的处理（使用API从Azure Storage Blob下载数据）
5. 运行时间加分（语言，并发，复杂度，tricks）
6. C# 实现加分
7. 低内存需求加分
8. Azure storage 相关可以查阅官方文档，数据样本可以用 [Azure Storage Explorer](https://azure.microsoft.com/en-us/features/storage-explorer/)来查看
