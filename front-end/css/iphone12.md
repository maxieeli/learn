# 模仿iphone12页面卷动逐行滑入效果

```html
<html>
  <h2>
    Superfast wireless
    <p>Hello 5G</p>
  </h2>
  <div id="iphone">
    <div id="hardware"></div>
    <div id="ui">
      <img src="https://assets.codepen.io/2002878/iphone12-5g_top_ui.jpg" class="top-ui" alt="" />
      <ul>
        <li>
          <img src="https://assets.codepen.io/2002878/iphone12-5g_show_01.jpg"/>
        </li>
        <li>
          <img src="https://assets.codepen.io/2002878/iphone12-5g_show_02.jpg"/>
        </li>
        <li>
          <img src="https://assets.codepen.io/2002878/iphone12-5g_show_03.jpg"/>
        </li>
        <li>
          <img src="https://assets.codepen.io/2002878/iphone12-5g_show_04.jpg"/>
        </li>
        <li>
          <img src="https://assets.codepen.io/2002878/iphone12-5g_show_05.jpg"/>
        </li>
        <li>
          <img src="https://assets.codepen.io/2002878/iphone12-5g_show_06.jpg"/>
        </li>
      </ul>
    </div>
  </div>
</html>
```

```css
:root {
  --device-width: 770px;
  --device-height: 1336px;
  --ui-width: 640px;
  font-size: 15px;
}

body {
  background-color: #000;
  margin: 0;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  font-family: Helvetica;
  padding: 4rem 0;
}

h2 {
  color: #6E6E73;
  text-align: center;
  font-size: 4.5rem;
  font-weight: 600;
  margin: 6rem 0;
}

h2 p {
  margin: 0;
  color: #FFF;
}

#iphone {
  position: relative;
  width: var(--device-width);
  height: var(--device-height);
}

#hardware {
  width: 100%;
  height: 100%;
  background-image: url(https://assets.codepen.io/2002878/iphone12-5g_on_phone.jpg);
  background-size: var(--device-width) var(--device-height);
  mask-image: url(https://assets.codepen.io/2002878/iphone12-5g_on_phone_mask.png);
  -webkit-mask-image: url(https://assets.codepen.io/2002878/iphone12-5g_on_phone_mask.png);
  mask-size: var(--device-width) var(--device-height);
  -webkit-mask-size: var(--device-width) var(--device-height);
}

#ui {
  position: absolute;
  top: 0;
  left: 50%;
  transform: translateX(-50%);
}

#ui .top-ui {
  display: block;
  width: var(--ui-width);
  height: auto;
  margin: 70px auto 0;
  padding-bottom: 30px;
  border-bottom: 1px solid #222;
}

#ui ul {
  list-style: 0;
  margin: 0;
  padding: 0;
  --progress: 0;
}

#ui ul li img {
  display: block;
  width: var(--ui-width);
  height: auto;
  margin: 10px auto;
  padding-bottom: 10px;
  border-bottom: 1px solid #222;
  transform: scale(calc(1.8 - (0.8 * var(--progress)))) translateY(calc(-60px * (1 - var(--progress))));
  opacity: var(--progress);
}
```

```javascript
const rows = document.querySelectorAll('#ui ul li'); // 将每一个li获取回来
const html = document.documentElement;

document.addEventListener('scroll', (e) => { // 监听页面滚动
  let scrolled = html.scrollTop / (html.scrollHeight - html.clientHeight); // 得到一个0-1的值。页面最顶为0，最低为1

  let total = 1 / rows.length;
  
  for(let [index, row] of rows.entries()) {
  // 将每个li的--progress值设为1
  // 由于动画是连续的，所以页面卷动多少可以由scrolled获取到
  // 所以要计算每个li在0-1的所属区间
    let start = total * index;
    let end = total * (index + 1);
    let progress = (scrolled - start) / (end - start);
    if(progress >= 1) progress = 1;
    if(progress < 0) progress = 0;
    
    row.style.setProperty('--progress', progress); // 设置在每个li上
  }
})
```