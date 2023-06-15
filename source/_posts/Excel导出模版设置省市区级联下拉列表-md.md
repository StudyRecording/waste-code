---
title: Excel导出模版设置省市区级联下拉列表
tags:
  - excel
  - 级联下拉列表
categories:
  - Excel
excerpt: 以省市区为例子，记录设置Excel的下拉级联的操作
thumbnail: https://t.mwm.moe/ycy
cover: https://t.mwm.moe/pc
sticky: 1
date: 2023-06-15 20:53:00
---

# Excel导出模版设置省市区级联下拉列表

>  业务中Excel导入操作，为了与数据库中的数据匹配，需要限制Excel中导入数据的内容，特别是多个内容中存在级联关系，以省市区为例子.



- 创建4个sheet页，分别是信息导入模版，省数据，市数据，区数据)
  
  ![](https://raw.githubusercontent.com/StudyRecording/waste-code-image/main/img/20230615210156.png)

- 在信息导入模版sheet页中添加3列：居住地址-省，居住地址-市，居住地址-区

- 补充省数据sheet页中的数据)
  
  ![](https://raw.githubusercontent.com/StudyRecording/waste-code-image/main/img/20230615210818.png)

- 补充市数据，其中最左列为省份数据，同一行右侧的数据为该省份下的城市
  
  ![](https://raw.githubusercontent.com/StudyRecording/waste-code-image/main/img/20230615211133.png)

- 补充区数据，其中最左列为城市数据，同一行右侧的数据为该城市下的区
  
  ![](https://raw.githubusercontent.com/StudyRecording/waste-code-image/main/img/20230615211451.png)

- 将省数据sheet页中的数据作为信息导入模版sheet页中”居住地址-省“的下拉选项
  
  - 在信息导入模版sheet页中，全选”居住地址-省“一列
  
  - 然后按照command键(Mac电脑)，选择”居住地址-省“的标题单元格，这样便全选了整列数据但唯独去除了标题栏
  
  - 选择wps中”数据“工具栏，并选择”有效性按钮“
    
    ![](https://raw.githubusercontent.com/StudyRecording/waste-code-image/main/img/20230615212128.png)
  
  - 选择有效性条件”序列“，然后点击来源右边的按钮
    
    ![](https://raw.githubusercontent.com/StudyRecording/waste-code-image/main/img/20230615212401.png)
  
  - 跳转到省数据sheet页并全选省数据，此时数据有效性的输入标签自动输入了全选的省数据单元格范围
    
    ![](https://raw.githubusercontent.com/StudyRecording/waste-code-image/main/img/20230615212719.png)
  
  - 此时将鼠标定位的有效性的输入框并回车，然后点击确定，之后切换到”信息导入模版“sheet页中，点击”居住地址-省“列下的单元格，就可以选择省级数据了
    
    ![](https://raw.githubusercontent.com/StudyRecording/waste-code-image/main/img/20230615213127.png)

- 将市数据sheet页中的数据作为”居住地址-市“列的下拉选项，并与”居住地址-省“列产生级联效果
  
  - 切换到市数据sheet页，并全选数据
  
  - 点击wps的”开始“工具栏，并选择”查找“，点击定位
    
    ![](https://raw.githubusercontent.com/StudyRecording/waste-code-image/main/img/20230615213602.png)    
  
  - 选着下图中已选的选项，然后点击定位按钮
    
    ![](https://raw.githubusercontent.com/StudyRecording/waste-code-image/main/img/20230615213839.png)
  
  - 选择wps的公式标签，然后选着”指定“按钮，在打开的”指定名称“的标签页中，选择”最左列“选项（因为市数据中每一行最左单元格为当前行的省份），然后点击标签页的确定按钮
    
    ![](https://raw.githubusercontent.com/StudyRecording/waste-code-image/main/img/20230615214351.png)
  
  - 此时，点击wps的公式工具栏，然后点击名称管理器，可以看到省份所对应的城市。这一步只是确认上一步是否成功，之后关闭标签页就行。
  
  - 切换到信息上传模版的sheet页，然后全选”居住地址-市“一列，接着按住command键单击”居住地址-市“标题单元格，这样变选择了除标题栏外的整列
  
  - 然后点击wps的数据工具栏，点击有效性按钮，在弹出的”数据有效性“标签中选着有效性条件允许为”序列“，然后在来源中输入”=INDIRECT(U2)“, 其中INDIRECT是Excel的函数，U2是”居住地址-省“可以选着数据的第一个单元格(作用是为了省市级联选择),然后点击标签页的确定按钮
    
    ![](https://raw.githubusercontent.com/StudyRecording/waste-code-image/main/img/20230615215010.png)
  
  - 此时在信息上传模版的sheet页中，省市变可以级联选择了
    
    ![](https://raw.githubusercontent.com/StudyRecording/waste-code-image/main/img/20230615215518.png)

- 关于市区级联与省市级联的操作步骤完全一样，只不过区数据sheet页中最左列是市名，同一行右侧的数据是该市下的其他区





> 在其中遇到了一些问题，比如直辖市的存在，北京市的三级级联是：北京市 -> 北京市 -> 各个区, 因此在名称管理器中省名对应的市，与市名对应的区的名称(开发角度可看成key)值都是”北京市“, 容易被覆盖，因此将省级的北京市特殊处理修改名为”北京“， 市级还是原来的”北京市“, 以此来解决名称冲突
