### 部分机型Text文本显示不全

>问题：一加和oppo部分机型的Text标签下的文本存在显示不全的情况  

>原因：React Native的bug或者部分机型系统自带兼容性bug ???

>解决方案：（通过在React Native官方ISSUE下面搜索发现有遇到同样的问题，解决方法是通过在App.js文件即app入口处统一对Text标签的fontFamily属性做处理，确保之后的字体显示正常）  

    //封装方法如下：  
    fixSomeModelTextLabelCompatibilityIssue = () => {
      let oldRender = Text.render;
      Text.render = function (...args) {
        let origin = oldRender.call(this, ...args);
        return React.cloneElement(origin, {
            style: [{color: "#333", ...Platform.select({
              android: {
                fontFamily: 'icomoon'
              }
            })}, origin.props.style]
        });
      };
    }

参考链接：https://github.com/facebook/react-native/issues/15114