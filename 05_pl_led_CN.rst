.. _Alinx官网: http://www.alinx.com

.. image:: images/images_0/88.png  

========================================
《第五章》PL的“Hello world”LED 实验
========================================
**实验Vivado工程为“led”**

- 对于ZYNQ来说PL(FPGA)开发是至关重要的,这也是ZYNQ比其他ARM的有优势的地方,可以定制化很多ARM端的外设,在定制ARM端的外设之前先让我们通过一个LED例程来熟悉PL(FPGA)的开发流程,熟悉Vivado软件的基本操作,这个开发流程和不带ARM的FPGA芯片完全一致。
- 在本例程中,我们要做的是LED灯控制实验,每秒钟控制开发板上的LED灯翻转一次,实现亮、灭、亮、灭的控制。会控制LED灯,其它外设也慢慢就会了。 

5.1LED硬件介绍
========================================
1)开发板的PL部分连接了4个红色的用户LED灯。这4个灯完全由PL控制。

.. image:: images/images_5/image130.png  
   :align: center

2)我们可以根据原理图的连线关系确定LED和PL管脚的绑定关系。

.. image:: images/images_5/image131.png  
   :align: center

对应的管脚信息

3)原理图中以PS_MIO开头的IO都是PS端IO,不需要绑定,也不能绑定

.. image:: images/images_5/image132.png  
   :align: center

5.2创建Vivado工程
========================================
1)启动Vivado,在Windows中可以通过双击Vivado快捷方式启动

.. image:: images/images_5/image133.png  
   :align: center

2)在Vivado开发环境里点击“Create New Project”,创建一个新的工程。

.. image:: images/images_5/image134.png  
   :align: center

3)弹出一个建立新工程的向导,点击“Next”

.. image:: images/images_5/image135.png  
   :align: center

4)在弹出的对话框中输入工程名和工程存放的目录,我们这里取一个led的工程名。需要注意工程路径“Project location”不能有中文空格,路径名称也不能太长。

.. image:: images/images_5/image136.png  
   :align: center

5)在工程类型中选择“RTL Project”

.. image:: images/images_5/image137.png  
   :align: center

6)目标语言“Target language”选择“Verilog”,虽然选择Verilog,但VHDL也可以使用,支持多语言混合编程。

.. image:: images/images_5/image138.png  
   :align: center

7)点击“Next”,不添加任何文件

.. image:: images/images_5/image139.png  
   :align: center

8)在“Part”选项中,器件家族“Family”选择“Zynq-7000”,AX7020开发板封装类型“Package”选择“clg400”,Speed选择”-2”减少选择范围。在下拉列表中选择“xc7z020clg400-2”,“-2”表示速率等级,数字越大,性能越好,速率高的芯片向下兼容速率低的芯片。

.. image:: images/images_5/image140.png  
   :align: center

AX7010开发板选择型号“xc7z010clg400-1”

.. image:: images/images_5/image141.png  
   :align: center

9)点击“Finish”就可以完成以后名为“led”工程的创建。

.. image:: images/images_5/image142.png  
   :align: center

10)Vivado软件界面

.. image:: images/images_5/image143.png  
   :align: center

5.3创建Verilog HDL文件点亮LED
========================================
1)点击Project Manager下的Add Sources图标(或者使用快捷键Alt+A)

.. image:: images/images_5/image144.png  
   :align: center

2)选择添加或创建设计源文件“Add or create design sources”,点击“Next”

.. image:: images/images_5/image145.png  
   :align: center

3)选择创建文件“Create File”

.. image:: images/images_5/image146.png  
   :align: center

4)文件名“File name”设置为“led”,点击“OK”

.. image:: images/images_5/image147.png  
   :align: center

5)点击“Finish”,完成“led.v”文件添加

.. image:: images/images_5/image148.png  
   :align: center

6)在弹出的模块定义“Define Module”,中可以指定“led.v”文件的模块名称“Module name”,这里默认不变为“led”,还可以指定一些端口,这里暂时不指定,点击“OK”。

.. image:: images/images_5/image149.png  
   :align: center

7)在弹出的对话框中选择“Yes”

.. image:: images/images_5/image150.png  
   :align: center

8)双击“led.v”可以打开文件,然后编辑

.. image:: images/images_5/image151.png  
   :align: center

9)编写“led.v”,这里定义了一个32位的寄存器timer, 用于循环计数0~49999999(1秒钟), 计数到49999999(1秒)的时候, 寄存器timer变为0,并翻转四个LED。这样原来LED是灭的话,就会点亮,如果原来LED为亮的话,就会熄灭。编写好后的代码如下:

::

 module led(
     input sys_clk,
     input rst_n,
     output reg [3:0] led
     );
 reg[31:0] timer_cnt;
 always@(posedge sys_clk or negedge rst_n)
 begin
     if (!rst_n)
     begin
         led <= 4'd0 ;
         timer_cnt <= 32'd0 ;
     end
     else if(timer_cnt >= 32'd49_999_999)
     begin
         led <= ~led;
         timer_cnt <= 32'd0;
     end
     else
     begin
         led <= led;
         timer_cnt <= timer_cnt + 32'd1;
     end
     
 end
 endmodule 

10)编写好代码后保存

5.4添加管脚约束
========================================
Vivado使用的约束文件格式为xdc文件。xdc文件里主要是完成管脚的约束,时钟的约束, 以及组的约束。这里我们需要对led.v程序中的输入输出端口分配到FPGA的真实管脚上。

1)点击“Open Elaborated Design”

.. image:: images/images_5/image152.png  
   :align: center

2)在弹出的窗口中点击“OK”按钮

.. image:: images/images_5/image153.png  
   :align: center

3)在菜单中选择“Window -> I/O Ports”

.. image:: images/images_5/image154.png  
   :align: center

4)在弹出的I/O Ports中可以看到管脚分配情况

.. image:: images/images_5/image155.png  
   :align: center

5)将复位信号rst_n绑定到PL端的按键,给LED和时钟分配管脚、电平标准,完成后点击保存图标

.. image:: images/images_5/image156.png  
   :align: center

6)弹出窗口,要求保存约束文件,文件名我们填写“led”,文件类型默认“XDC”,点击“OK”

.. image:: images/images_5/image157.png  
   :align: center

7)打开刚才生成的“led.xdc”文件,我们可以看到是一个TCL脚本,如果我们了解这些语法,完全可以通过自己编写led.xdc文件的方式来约束管脚

.. image:: images/images_5/image158.png  
   :align: center

- 下面来介绍一下最基本的XDC编写的语法,普通IO口只需约束引脚号和电压,管脚约束如下:
- set_property  PACKAGE_PIN "引脚编号"  [get_ports “端口名称”] 
- 电平信号的约束如下:
- set_property  IOSTANDARD  "电平标准"  [get_ports “端口名称”] 
- 这里需要注意文字的大小写,端口名称是数组的话用{ }刮起来,端口名称必须和源代码中的名字一致,且端口名字不能和关键字一样。
- 电平标准中“LVCMOS33”后面的数字指FPGA的BANK电压,LED所在BANK电压为3.3伏,所以电平标准为“LVCMOS33”。 **Vivado默认要求为所有IO分配正确的电平标准和管脚编号。**

5.5添加时序约束
========================================
一个FPGA设计除了管脚分配以外,还有一个重要的约束,那就是时序约束,这里通过向导方式演示如果进行一个时序约束。

1)点击“Run Synthesis”开始综合

.. image:: images/images_5/image159.png  
   :align: center

2)弹出对话框点击“OK”

.. image:: images/images_5/image160.png  
   :align: center

3)综合完成以后点击“Cancel”

.. image:: images/images_5/image161.png  
   :align: center

4)点击“Constraints Wizard”

.. image:: images/images_5/image162.png  
   :align: center

5)在弹出的窗口中点击“Next”

.. image:: images/images_5/image163.png  
   :align: center

6)时序约束向导分析出设计中的时钟,这里把“sys_clk”频率设置为50Mhz,然后点击“Skip to Finish”结束时序约束向导。

.. image:: images/images_5/image164.png  
   :align: center

7)弹出的窗口中点击“OK”

.. image:: images/images_5/image165.png  
   :align: center

8)点击“Finish”

.. image:: images/images_5/image166.png  
   :align: center

9)这个时候led.xdc文件已经更新,点击“Reload”重新加载文件,并保存文件

.. image:: images/images_5/image167.png  
   :align: center

5.6生成BIT文件
========================================
1)编译的过程可以细分为综合、布局布线、生成bit文件等,这里我们直接点击“Generate Bitstream”,直接生成bit文件。

.. image:: images/images_5/image168.png  
   :align: center

2)在弹出的对话框中可以选择任务数量,这里和CPU核心数有关,一般数字越大,编译越快,点击“OK”

.. image:: images/images_5/image169.png  
   :align: center

3)这个时候开始编译,可以看到右上角有个状态信息,在编译过程中可能会被杀毒软件、电脑管家拦截运行,导致无法编译或很长时间没有编译成功。

.. image:: images/images_5/image170.png  
   :align: center

4)编译中没有任何错误,编译完成,弹出一个对话框让我们选择后续操作,可以选择“Open Hardware Manger”,当然,也可以选择“Cancel”,我们这里选择 “Cancel”,先不下载。

.. image:: images/images_5/image171.png  
   :align: center

5.7Vivado仿真
========================================
接下来我们不妨小试牛刀,利用Vivado自带的仿真工具来输出波形验证流水灯程序设计结果和我们的预想是否一致(注意:在生成bit文件之前也可以仿真)。具体步骤如下:
1) 设置Vivado的仿真配置,右击SIMULATION中Simulation Settings。 

.. image:: images/images_5/image172.png  
   :align: center

2)在Simulation Settings窗口中进行如下图来配置,这里设置成50ms(根据需要自行设定),其它按默认设置,单击OK完成。

.. image:: images/images_5/image173.png  
   :align: center

3)添加激励测试文件,点击Project Manager下的Add Sources图标,按下图设置后单击Next。

.. image:: images/images_5/image174.png  
   :align: center

4)点击Create File生成仿真激励文件。 

.. image:: images/images_5/image175.png  
   :align: center

在弹出的对话框中输入激励文件的名字,这里我们输入名为vtf_led_test。

.. image:: images/images_5/image176.png  
   :align: center

5) 点击Finish按钮返回。

.. image:: images/images_5/image177.png  
   :align: center

这里我们先不添加IO Ports,点击OK。

.. image:: images/images_5/image178.png  
   :align: center

在Simulation Sources目录下多了一个刚才添加的vtf_led_test文件。双击打开这个文件,可以看到里面只有module名的定义,其它都没有。

.. image:: images/images_5/image179.png  
   :align: center

6)接下去我们需要编写这个vtf_led_test.v文件的内容。首先定义输入和输出信号,然后需要实例化led_test模块,让led_test程序作为本测试程序的一部分。再添加复位和时钟的激励。完成后的vtf_led_test.v文件如下:

::

 `timescale 1ns / 1ps
 //////////////////////////////////////////////////////////////////////////////////
 // Module Name: vtf_led_test
 //////////////////////////////////////////////////////////////////////////////////
 
 module vtf_led_test;
 // Inputs
 reg sys_clk;
 reg rst_n ;
 // Outputs
 wire [3:0] led;
 
 // Instantiate the Unit Under Test (UUT)
 led uut (
     .sys_clk(sys_clk),   
     .rst_n(rst_n),
     .led(led)
  );
 
 initial 
 begin
 // Initialize Inputs
     sys_clk = 0;
     rst_n = 0 ;
     #1000 ;
     rst_n = 1; 
 end
 //Create clock
 always #10 sys_clk = ~ sys_clk;  
 
 endmodule

7) 编写好后保存,vtf_led_test.v自动成了这个仿真Hierarchy的顶层了,它下面是设计文件led_test.v。

.. image:: images/images_5/image180.png  
   :align: center

8)点击Run Simulation按钮,再选择Run Behavioral Simulation。这里我们做一下行为级的仿真就可以了。

.. image:: images/images_5/image181.png  
   :align: center

如果没有错误,Vivado中的仿真软件开始工作了。

10. 在弹出仿真界面后如下图,界面是仿真软件自动运行到仿真设置的50ms的波形。

.. image:: images/images_5/image182.png  
   :align: center

由于LED[3:0]在程序中设计的状态变化时间长,而仿真又比较耗时,在这里观测timer[31:0]计数器变化。把它放到Wave中观察(点击Scope界面下的uut, 再右键选择Objects界面下的timer, 在弹出的下拉菜单里选择Add Wave Window)。

.. image:: images/images_5/image183.png  
   :align: center

添加后timer显示在Wave的波形界面上,如下图所示。

.. image:: images/images_5/image184.png  
   :align: center

11. 点击如下标注的Restart按钮复位一下,再点击Run All按钮。(需要耐心!!!),可以看到仿真波形与设计相符。(注意:仿真的时间越长,仿真的波形文件占用的磁盘空间越大,波形文件在工程目录的xx.sim文件夹)

.. image:: images/images_5/image185.png  
   :align: center

.. image:: images/images_5/image186.png  
   :align: center

我们可以看到led的信号会变成F,说明LED1~LED4灯同时变亮。

5.8下载
========================================
1)连接好开发板的JTAG接口,给开发板上电
2)在“HARDWARE MANAGER”界面点击“Auto Connect”,自动连接设备

.. image:: images/images_5/image187.png  
   :align: center

3)可以看到JTAG扫描到arm和FPGA内核

.. image:: images/images_5/image188.png  
   :align: center

4)选择xc7z020_1,右键“Program Device...”

.. image:: images/images_5/image189.png  
   :align: center

5)在弹出窗口中点击“Program”

.. image:: images/images_5/image190.png  
   :align: center

6)等待下载

.. image:: images/images_5/image191.png  
   :align: center

7)下载完成以后,我们可以看到4颗LED开始每秒变化一次。到此为止Vivado简单流程体验完成。后面的章节会介绍如果把程序烧录到Flash,需要PS系统的配合才能完成,只有PL的工程不能直接烧写Flash。在”体验ARM,裸机输出”Hello World”一章的常见问题中有介绍。

5.9在线调试
========================================
前面介绍了仿真和下载,但仿真并不需要程序烧写到板子,是比较理想化的结果,下面介绍Vivado在线调试方法,观察内部信号的变化。Vivado有内嵌的逻辑分析仪,叫做ILA,可以用于在线观察内部信号的变化,对于调试有很大帮助。在本实验中我们观察timer_cnt和led的信号变化。
添加ILA IP核
1)点击IP Catalog,在搜索框中搜索ila,双击ILA的IP

.. image:: images/images_5/image192.png  
   :align: center

2)修改名称为ila,由于要采样两个信号,Probes的数量设置为2,Sample Data Depth指的是采样深度,设置的越高,采集的信号越多,同样消耗的资源也会越多。

.. image:: images/images_5/image193.png  
   :align: center

3)在Probe_Ports页面,设置Probe的宽度,设置PROBE0位宽为32,用于采样timer_cnt,设置PROBE1位宽为4,用于采样led。点击OK

.. image:: images/images_5/image194.png  
   :align: center

弹出界面,选择OK

.. image:: images/images_5/image195.png  
   :align: center

再如下设置,点击Generate

.. image:: images/images_5/image196.png  
   :align: center

4)在led.v中例化ila,并保存

.. image:: images/images_5/image197.png  
   :align: center

5)重新生成Bitstream

.. image:: images/images_5/image198.png  
   :align: center

6)下载程序

.. image:: images/images_5/image1985.png  
   :align: center

这时候看到有bit和ltx文件,点击program

.. image:: images/images_5/image199.png  
   :align: center

7)此时弹出在线调试窗口,出现了我们添加的信号

.. image:: images/images_5/image200.png  
   :align: center

点击运行按钮,出现信号的数据

.. image:: images/images_5/image201.png  
   :align: center

也可以触发采集,在Trigger Setup窗口点击“+”,深度选择timer_cnt信号

.. image:: images/images_5/image202.png  
   :align: center

将Radix改为U,也就是十进制,在Value中设置为49999999,也就是timer_cnt计数的最大值

.. image:: images/images_5/image203.png  
   :align: center

再次点击运行,即可以看到触发成功,此时timer_cnt显示为十六进制,而led也在此时翻转。

.. image:: images/images_5/image204.png  
   :align: center

5.9.2 MARK DEBUG
---------------------
上面介绍了添加ILA IP的方式在线调试,下面介绍在代码中添加综合属性,实现在线调试。

1)首先打开led.v,将ila的例化部分注释掉

.. image:: images/images_5/image205.png  
   :align: center

2)在led和timer_cnt的定义前面添加( MARK_DEBUG=”true” ),保存文件。

.. image:: images/images_5/image206.png  
   :align: center

3)点击综合

.. image:: images/images_5/image207.png  
   :align: center

4)综合结束后,点击Set Up Debug

.. image:: images/images_5/image208.png  
   :align: center

5)弹出的窗口点击Next

.. image:: images/images_5/image209.png  
   :align: center

按照默认点击Next

.. image:: images/images_5/image210.png  
   :align: center

采样深度窗口,选择Next

.. image:: images/images_5/image211.png  
   :align: center

点击Finish

.. image:: images/images_5/image212.png  
   :align: center

点击保存

.. image:: images/images_5/image213.png  
   :align: center

在xdc文件中即可看到添加的ila核约束

.. image:: images/images_5/image214.png  
   :align: center

5)重新生成bitstream

.. image:: images/images_5/image215.png  
   :align: center

6)调试方法与前面一样,不再赘述。

5.10实验总结
========================================
本章节介绍了如何在PL端开发程序,包括工程建立,约束,仿真,在线调试等方法,在后续的代码开发方式中皆可参考此方法。

  
.. image:: images/images_0/888.png  

*ZYNQ-7000开发平台 FPGA教程*    - `Alinx官方网站 <http://www.alinx.com>`_
