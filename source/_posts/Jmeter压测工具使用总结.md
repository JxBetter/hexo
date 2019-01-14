---
title: Jmeter压测工具使用总结
date: 2019-01-14 13:47:27
top: 1
tags: 
	- 测试
categories: 
	- 软件测试
---
<blockquote>
<ul>
            <li>
                <h1>1、常用测试工具对比</h1>
                                 <blockquote>
                1、loadrunner
                    性能稳定，压测结果及细粒度大，可以自定义脚本进行压测，但是太过于重大，功能比较繁多<br>

                2、apache ab(单接口压测最方便)
                    模拟多线程并发请求,ab命令对发出负载的计算机要求很低，既不会占用很多CPU，也不会占用太多的内存，但却会给目标服务器造成巨大的负载, 简单DDOS攻击等<br>

                3、webbench
                    webbench首先fork出多个子进程，每个子进程都循环做web访问测试。子进程把访问的结果通过pipe告诉父进程，父进程做最终的统计结果。<br>
                <br>
                                </blockquote>
            </li>

<li>
                <h1>2、Jmeter目录文件讲解</h1>
                <blockquote>
                    bin:核心可执行文件，包含配置<br>
                        jmeter.bat: windows启动文件：<br>
                        jmeter: mac或者linux启动文件：<br>
                        jmeter-server：mac或者Liunx分布式压测使用的启动文件<br>
                        jmeter-server.bat：mac或者Liunx分布式压测使用的启动文件<br>
                        jmeter.properties: 核心配置文件<br>
                        <br>


                    extras：插件拓展的包<br>
                    lib:核心的依赖包<br>
                        ext:核心包<br>
                        junit:单元测试包<br>
                </blockquote>
            </li>

<li>
                <h1>3、Jmeter基础功能组件介绍线程组和Sampler</h1>
                <blockquote>
                    1、添加-&gt;threads-&gt;线程组（控制总体并发）<br>
                        线程数：虚拟用户数。一个虚拟用户占用一个进程或线程<br>
                        <br>
                        准备时长（Ramp-Up Period(in seconds)）：全部线程启动的时长，比如100个线程，20秒，则表示20秒内100个线程都要启动完成，每秒启动5个线程<br>

                        循环次数：每个线程发送的次数，假如值为5，100个线程，则会发送500次请求，可以勾选永远循环<br>
                        <br>


                    2、线程组-&gt;添加-&gt; Sampler(采样器) -&gt; Http <br>（一个线程组下面可以增加几个Sampler）<br>
                        名称：采样器名称<br>
                        注释：对这个采样器的描述<br>
                        <br>
                        web服务器：<br>
                            默认协议是http<br>
                            默认端口是80<br>
                            服务器名称或IP ：请求的目标服务器名称或IP地址<br>
                        <br>
                        路径：服务器URL<br>
                        <br>
                        Use multipart/from-data for HTTP POST ：当发送POST请求时，使用Use multipart/from-data方法发送，默认不选中。<br>

                        <br>
                    3、查看测试结果<br>
                        线程组-&gt;添加-&gt;监听器-&gt;察看结果树<br>
                        <br>
                </blockquote>
            </li>

            <li>
                <h1>4、Jmeter的断言基本使用</h1>
                <blockquote>
                    增加断言: 线程组 -&gt; 添加 -&gt; 断言 -&gt; 响应断言  <br>
                        apply to(应用范围):<br>
                        <blockquote>
                            Main sample only: 仅当前父取样器 <br>进行断言，一般一个请求，如果发一个请求会触发多个，则就有sub <br>sample（比较少用）<br>
                        </blockquote>
                        要测试的响应字段：<br>
                        <blockquote>
                            响应文本：即响应的数据，比如json等文本<br>
                            响应代码：http的响应状态码，比如200，302，404这些<br>
                            响应信息：http响应代码对应的响应信息，例如：OK, Found<br>
                            Response Header: 响应头<br>
                        </blockquote>
                        模式匹配规则：<br>
                        <blockquote>
                            包括：是响应文本的一个子集，是包含关系，可以用正则表达式<br>
                            匹配：使用正则表达式匹配<br>
                            equals：完全与响应文本相同，不能使用正则表达式<br>
                            substring：也是包含关系，但是不能使用正则表达式
                        </blockquote>       
                </blockquote>
                2、断言结果监听器: 线程组-&gt; 添加 -&gt; 监听器 -&gt; 断言结果<br>
                <blockquote>
                    里面的内容是sampler采样器的名称<br>
                    断言失败，查看结果树任务结果颜色标红<br>通过结果数里面双击不通过的记录，可以看到错误信息)<br>
                </blockquote>
                3、每个sample下面可以加单独的结果树，然后同时加多个断言，最外层可以加个结果树进行汇总<br>
            </li>

<li>
                <h1>5、结果聚合报告分析</h1>
                新增聚合报告：线程组-&gt;添加-&gt;监听器-&gt;聚合报告（Aggregate Report）<br>
                    <blockquote>
                        lable: sampler的名称<br>
                        Samples: 一共发出去多少请求,例如10个用户，循环10次，则是 100<br>
                        Average: 平均响应时间<br>
                        Median: 中位数，也就是 50％ 用户的响应时间<br>

                        90% Line : 90％ 用户的响应不会超过该时间 （90% of the samples took no more than this time. The remaining samples at least as long as this）<br>
                        95% Line : 95％ 用户的响应不会超过该时间<br>
                        99% Line : 99％ 用户的响应不会超过该时间<br>
                        min : 最小响应时间<br>
                        max : 最大响应时间<br>
                        <br>
                        Error%：错误的请求的数量/请求的总数<br>
                        Throughput： 吞吐量——默认情况下表示每秒完成的请求数（Request per Second) 可类比为qps<br>
                        KB/Sec: 每秒接收数据量<br>
                    </blockquote>
            </li>

<li>
                <h1>6、Jmeter用户自定义变量</h1>
                <blockquote>
                    1、线程组-&gt;add -&gt; Config Element(配置原件)-&gt; User Definde Variable（用户定义的变量）<br>

                    2、引用方式${XXX}，在接口中变量中使用<br>
                </blockquote>
            </li>

<li>
                <h1>7、CSV数据文件使用</h1>
                <blockquote>
                    1、线程组-&gt;add -&gt; Config Element(配置原件)-&gt; CSV data set config (CSV数据文件设置)<br>
                    2、如果是多个参数需要同时引用，则在CSV数据文件里面设置加多个字段 Variabled names(comma-delitited):  csv_name,csv_pwd<br>
                </blockquote>
            </li>

<li>
                <h1>8、数据库压测操作</h1>
                <blockquote>
                    配置讲解：<br>
                    <blockquote>
                        1、Thread Group -&gt; add -&gt; sampler -&gt; jdbc request 添加数据库请求<br>
                        2、jar包添加  mysql-connector-java-5.1.30.jar 
                        添加连接数据库的jar包<br>
                        3、JDBC connection Configuration 配置<br>
                        <blockquote>
                            JDBC request-&gt;add -&gt; config element -&gt; JDBC connection configuration<br>

                                <blockquote>
                                    核心配置<br>
                                    <ul>
                                        <li>
                                            Max Number of connections : 最大连接数<br>
                                        </li>
                                        <li>
                                            MAX wait :最大等待时间<br>
                                            Auto Commit: 是否自动提交事务<br>
                                        </li>
                                        <li>
                                            DataBase URL : 数据库连接地址 jdbc:mysql://127.0.0.1:3306/blog
                                        </li>
                                        <li>
                                            JDBC Driver Class : 数据库驱动，选择对应的mysql<br>
                                        </li>
                                        <li>
                                            username:数据库用户名<br>
                                        </li>
                                        <li>
                                            password:数据库密码<br>
                                        </li>
                                    </ul>
                                </blockquote>
                        </blockquote>
                    </blockquote>
                    数据库语句：<br>
                    <blockquote>
                            1、Debug Sampler使用（结果树中查看）
                                Thread Group -&gt; add -&gt; sampler -&gt; debug sampler<br>

                            2、参数讲解：(sql结尾不要加";")<br>
                                1、variable name of pool declared in JDBC connection configuration（和配置文件同名）<br>
                                2、Query Type 查询类型<br>
                                3、parameter values 参数值<br>
                                4、parameter types  参数类型<br>
                                5、variable names  sql执行结果变量名<br>
                                6、result variable names 所有结果当做一个对象存储
                                7、query timeouts  查询超时时间 <br>
                                8、 handle results  处理结果集<br>
                    </blockquote>
                </blockquote>
            </li>

<li>
                <h1>9、Jmeter非GUI界面参数</h1>
                <blockquote>
                     -h 帮助<br>
                    -n 非GUI模式<br>
                    -t 指定要运行的 JMeter 测试脚本文件<br>
                    -l 记录结果的文件 每次运行之前，(要确保之前没有运行过,即xxx.jtl不存在，不然报错)<br>
                    -r Jmter.properties文件中指定的所有远程服务器<br>
                    -e 在脚本运行结束后生成html报告<br>
                    -o 用于存放html报告的目录（目录要为空，不然报错）<br>
                </blockquote>
                 官方配置文件地址 <a href="http://jmeter.apache.org/usermanual/get-started.html" rel="nofollow">http://jmeter.apache.org/usermanual/get-started.html</a><br>
                 <br>
            </li>

<li>
                <h1>10、Jmeter压测接口的性能优化</h1>
                <blockquote>
                    1、使用非GUI模式：jmeter -n -t test.jmx -l result.jtl<br>
                    2、少使用Listener， 如果使用-l参数，它们都可以被删除或禁用。<br>
                    3、在加载测试期间不要使用“查看结果树”或“查看结果”表监听器，只能在脚本阶段使用它们来调试脚本。<br>
                    4、包含控制器在这里没有帮助，因为它将文件中的所有测试元素添加到测试计划中。<br>
                    5、不要使用功能模式,使用CSV输出而不是XML<br>
                    6、只保存你需要的数据,尽可能少地使用断言<br>
                    7、如果测试需要大量数据，可以提前准备好测试数据放到数据文件中，以CSV Read方式读取。<br>
                    8、用内网压测，减少其他带宽影响压测结果<br>
                    9、如果压测大流量，尽量用多几个节点以非GUI模式向服务器施压<br>
                </blockquote>
            </li>
<li>
                <h1>11、Jmeter图形化HTML压测报告dashboard</h1>
                <blockquote>
                    1、Test and Report informations<br>
                    <blockquote>
                        Source file：jtl文件名<br>
                        Start Time ：压测开始时间<br>
                        End Time ：压测结束时间<br>
                        Filter for display：过滤器<br>
                        Lable:sampler采样器名称  <br>
                    </blockquote>
                    2、APDEX(Application performance Index)<br>
                    <blockquote>
                        apdex:应用程序性能指标,范围在0~1之间，1表示达到所有用户均满意<br>
                        T(Toleration threshold)：可接受阀值<br>
                        F(Frustration threshold)：失败阀值<br>
                    </blockquote>
                    3、Requests Summary 请求总结<br>
                    <blockquote>
                        OK:成功率<br>
                        KO:失败率<br>
                    </blockquote>
                    4、Statistics 统计数据<br>
                    <blockquote>
                        lable:sampler采样器名称<br>

                        samples:请求总数，并发数*循环次数<br>
                        KO:失败次数<br>
                        Error%:失败率<br>

                        Average:平均响应时间<br>
                        Min:最小响应时间<br>
                        Max:最大响应时间<br>
                        90th pct: 90%的用户响应时间不会超过这个值（关注这个就可以了）
                        2ms,3ms,4,5,2,6,8,3,9<br>

                        95th pct: 95%的用户响应时间不会超过这个值<br>
                        99th pct: 99%的用户响应时间不会超过这个值 (存在极端值)<br>
                        throughtput:Request per Second吞吐量 qps<br>

                        received:每秒从服务器接收的数据量<br>
                        send：每秒发送的数据量<br>
                    </blockquote>   
                </blockquote>
            </li>

<li>
                <h1>12、Jmeter图形化HTML压测报告Charts报表讲解</h1>
                <blockquote>
                        1、Over Time（随着时间的变化）<br>
                        <blockquote>
                            Response Times Over Time：响应时间变化趋势<br>
                            Response Time Percentiles Over Time (successful responses)：最大，最小，平均，用户响应时间分布<br>
                            Active Threads Over Time：并发用户数趋势<br>
                            Bytes Throughput Over Time：每秒接收和请求字节数变化，蓝色表示发送，黄色表示接受<br>
                            Latencies Over Time：平均响应延时趋势<br>
                            Connect Time Over Time  ：连接耗时趋势<br>
                        </blockquote>               
                            2、Throughput<br>
                            <blockquote>
                                Hits Per Second (excluding embedded resources):每秒点击次数<br>
                                Codes Per Second (excluding embedded resources)：每秒状态码数量<br>
                                Transactions Per Second：即TPS，每秒事务数<br>
                                Response Time Vs Request：响应时间和请求数对比<br>
                                Latency Vs Request：延迟时间和请求数对比<br>
                            </blockquote>
                            3、Response Times<br>
                            <blockquote>
                                Response Time Percentiles：响应时间百分比<br>
                                Response Time Overview：响应时间概述<br>
                                Time Vs Threads：活跃线程数和响应时间<br>
                                Response Time Distribution：响应时间分布图<br>
                            </blockquote>
                </blockquote>
            </li>

<li>
                <h1>13、Jmeter压测注意事项</h1>
                <blockquote>
                    the firewalls on the systems are turned off or correct ports are opened.<br>
                    系统上的防火墙被关闭或正确的端口被打开。<br>
                    <br>
                    all the clients are on the same subnet.<br>
                    所有的客户端都在同一个子网上。<br>
                    <br>
                    the server is in the same subnet, if 192.x.x.x or 10.x.x.x IP addresses are used. If the server doesn't use 192.xx or 10.xx IP address, there shouldn't be any problems.<br>
                    如果使用192.x.x.x或10.x.x.x IP地址，则服务器位于同一子网中。 如果服务器不使用192.xx或10.xx IP地址，则不应该有任何问题。<br>
                    <br>
                    Make sure JMeter can access the server.<br>
                    确保JMeter可以访问服务器。<br>
                    <br>
                    Make sure you use the same version of JMeter and Java on all the systems. Mixing versions will not work correctly.<br>
                    确保在所有系统上使用相同版本的JMeter和Java。 <br>混合版本将无法正常工作。<br>
                    <br>
                    You have setup SSL for RMI or disabled it.<br>
                    您已为RMI设置SSL或将其禁用。<br>
                    <br>
                    <br>
                    官网地址 <a href="http://jmeter.apache.org/usermanual/jmeter_distributed_testing_step_by_step.html" rel="nofollow">http://jmeter.apache.org/usermanual/jmeter_distributed_testing_step_by_step.html</a><br>

                    压测注意事项：一定要用内网IP，不用用公网IP<br>
                </blockquote>
            </li>
</ul>
</blockquote>