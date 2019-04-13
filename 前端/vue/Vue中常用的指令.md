# Vue中常用的指令

##### v-cloak

- 隐藏插值表达式

- 使用方法

  属性选择器设置[v-cloak]的display:none

  ```html
      <style>
      [v-cloak]{
        display: none
      }
      </style>
  ```

  要隐藏插值表达式的标签中引用v-cloak属性

  ```html
      </style>
  
      <script src="./vue.js"></script>
  
      <script>                
  
       onload=()=>{
  
         var vm= new Vue({
  
            el: "#app",
  
         data: { msg: `hello world!` }
  
        });
  
       }
  
     </script>
  
    </head>
  
    <body>
  
      <div id="app">
  
          <p v-cloak>{{msg}}---asd</p></div>
  
    </body>
  ```



##### v-text

- `v-text`也能隐藏指令他的实现方式是替换标签内的内容,因此使用`v-text`时不需要配置style

  ```html
      <div id="app">
        <p v-cloak>-----{{ msg }}---</p>
        <p v-text="msg">-----</p>
      </div>
  ```

  浏览器中打开

  ```html
  -----hello world!---
  
  hello world!
  ```


##### v-html

- `v-cloak`和`v-text`当vm中提供的数据为html代码时，html代码不会不渲染而是被转义输出。使用`v-html`则不会转义输出。



  ![1546865189070](C:\Users\陈勇\AppData\Roaming\Typora\typora-user-images\1546865189070.png)



  ![1546865166060](C:\Users\陈勇\AppData\Roaming\Typora\typora-user-images\1546865166060.png)


![1546865140310](C:\Users\陈勇\AppData\Roaming\Typora\typora-user-images\1546865140310.png)

