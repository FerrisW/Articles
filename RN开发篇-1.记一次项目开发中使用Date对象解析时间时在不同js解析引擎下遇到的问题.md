#### 特定格式的日期在用Date对象进行处理时在不同js引擎下的解析结果会有差异

>问题：  
&emsp;&emsp;智能家居APP中后端接口返回的单个策略日志对象中包含策略的执行时间，格式为：`2020-05-16 17:07:30`，在对返回的策略日志数据进行处理最终SectionList列表的形式进行展示。在google浏览器下开启debug模式时，能成功正常展示。但在关闭debug模式或release打包模式下策略日志无法显示（操作前后代码及接口无任何改变）
    
    单个策略日志对象如下：
    LogItem:{    
      id: 1,    
      st_id: 3,
      st_name: '策略名称',
      date: '2020-05-16 17:07:30',    
      result: '执行成功'    
    }

>原因：  
&emsp;&emsp;开启debug模式时，React Native中解析js代码的工作由google的js引擎V8来完成,但在非debug模式下或release模式下，React Native的js代码运行在android或ios下，此时解析js代码的工作由ios内置的JavaScriptCore或android的jsc.so来完成。V8下能正常执行 `new Date(LogItem.date)`，而在JavaScriptCore或jsc.so下不能正常执行 `new Date(LogItem.date)`

>解决方案：  
&emsp;&emsp;1.对 `LogItem.date` 进行处理。如 `LogItem.Date.replace(' ', 'T')`  
&emsp;&emsp;2.同后端人员约定date的返回值为时间戳，便于转换

>番外篇：  
&emsp;&emsp;android和ios中的js代码解析工作均由JavaScriptCore来完成  
&emsp;&emsp;不同之处在于：ios直接内置了JavaScriptCore,而android则使用的是webkit.org开源的jsc.so

参考资料链接：  
&emsp;&emsp;1. https://blog.csdn.net/BonJean/article/details/78453547  
&emsp;&emsp;2. https://www.jianshu.com/p/048229e8d59b  
&emsp;&emsp;3. https://segmentfault.com/q/1010000015064544  
&emsp;&emsp;4. https://juejin.im/post/5b395eb96fb9a00e556123ef