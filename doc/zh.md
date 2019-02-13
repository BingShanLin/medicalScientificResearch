# Reference API 

## HTML

### Root Element

Poplar-annotator可以绑定到任何一个空HTML元素上。

我们推荐绑定到`div`元素上。

#### Example

```html
<div id="example"></div>
```

## CSS

目前支持设置内容文字大小等样式，请使用下面的CSS代码：

```css
svg > text tspan {
	font-size: 20px;
}
```

你可以在svg前添加你自己的选择器。

## JS

### 创建

为了使用Poplar-annotation，我们需要在JS中创建Annotator对象：

```typescript
import {Annotator} from 'poplar-annotation'
/**
  * 构造Annotator对象
  * @param data          数据，可以为JSON格式或纯文本
  * @param htmlElement   要放置内容的html元素
  * @param config        配置对象
  */
new Annotator(data: string, htmlElement: HTMLElement, config?: Object)
```

### 移除

要移除Annotator实例，只需调用Annotator对象的`remove()`方法即可。

#### Example

```typescript
annotator.remove();
```

#### data

在data为JSON时，格式如下：

![JSON格式](http://www.pic68.com/uploads/2018/08/1(7).png)

在data为纯文本时，相当于`content`为文本内容，其他为`[]`的JSON。

注意`content`中的全角空格`\u3000`和多个连续空格都会被替换为单个空格。

`Label`等的坐标系统按照替换后的计算。

构造后，对应元素内应该就会显示出对应的SVG图片。

#### config

`config`字典是一个`object`，其中可配置的值如下：

| 配置项             | 说明                                             | 默认值 |
| ------------------ | ------------------------------------------------ | ------ |
| maxLineWidth       | 最大行宽，在一行中的字符数超过此数值后会进行折行 | 80     |
| allowMultipleLabel | 是否允许同一块文本被不同的Label标记              | true   |

### Events

#### textSelected

在用户在页面上选取了一段文本后，会触发`textSelected`事件。

这个event会带两个参数，我们将其分别称为`startIndex`和`endIndex`：

| 参数       | 意义               |
| ---------- | ------------------ |
| startIndex | 选取部分的开始坐标 |
| endIndex   | 选取部分的结束坐标 |

它们代表用户选取了在原文本中的`[startIndex, endIndex)`部分。

可以拦截此事件，并向用户询问要添加的标注所属的类型，然后添加标注（如何添加见后）。

##### Example

```typescript
let originString = 'hello world';
let annotator = new Annotator(originString, document.getElementById('test'));
annotator.on('textSelected', (startIndex: number, endIndex: number) => {
    // 输出用户选取的那些字
    console.log(originString.slice(startIndex, endIndex));
});
```

#### labelClicked

在用户左键点击了一个Label后会触发这个事件。

这个event会带一个参数，为被点击的标注的ID：

| 参数 | 意义             |
| ---- | ---------------- |
| id   | 被点击的标注的id |

##### Example

```typescript
let originString = 'hello world';
let annotator = new Annotator(originString, document.getElementById('test'));
annotator.on('labelClicked', (id: number) => {
    // 输出用户点击的label的ID
    console.log(id);
});
```

在下面的`twoLabelsClicked`事件不能满足需求时，可使用这一事件定制添加连接的逻辑。

#### labelRightClicked

在用户右键点击了一个Label后会触发这个事件。

这个event会带三个参数，为被点击的标注的ID和被点击时鼠标的坐标：

| 参数 | 意义              |
| ---- | ----------------- |
| id   | 被点击的标注的id  |
| x    | 被点击时鼠标的X值 |
| y    | 被点击时鼠标的Y值 |

##### Example

```typescript
let originString = 'hello world';
let annotator = new Annotator(originString, document.getElementById('test'));
annotator.on('labelRightClicked', (id: number,x: number,y: number) => {
    // 输出用户点击的label的ID, 被点击时鼠标的 X,Y 值
    console.log(id,x,y);
});
```

可以使用这一事件来让用户对`Label`进行修改。

#### twoLabelsClicked

在用户先后左键点击了两个Label后会触发这个事件。

这个event会带两个参数，我们将其分别称为`first`和`second`：

| 参数   | 意义                 |
| ------ | -------------------- |
| first  | 第一个点击的标注的id |
| second | 第二个点击的标注的id |

他们代表了用户先后点击了id为`first`和`second`的两个标注。

可以拦截此事件，并向用户询问要添加的连接所属的类型，然后添加连接（如何添加见后）。

##### Example

```typescript
let originString = 'hello world';
let annotator = new Annotator(originString, document.getElementById('test'));
annotator.on('twoLabelsClicked', (first: number, second: number) => {
    // 输出用户选取的两个label的ID
    console.log(first,second);
});
```

#### connectionRightClicked

在用户右键点击了一个连接的文字部分后会触发这个事件。

这个event会带三个参数，为被点击的连接的ID和被点击时鼠标的坐标：

| 参数 | 意义              |
| ---- | ----------------- |
| id   | 被点击的连接的id  |
| x    | 被点击时鼠标的X值 |
| y    | 被点击时鼠标的Y值 |

##### Example

```typescript
let originString = 'hello world';
let annotator = new Annotator(originString, document.getElementById('test'));
annotator.on('connectionRightClicked', (id: number,x: number,y: number) => {
    // 输出用户点击的Connection的ID, 被点击时鼠标的 X,Y 值
    console.log(id,x,y);
});
```

可以使用这一事件来让用户对`connection`进行修改。

### Actions

可以通过`applyAction`方法向`Annotator`对象发送`Action`来改变其中的内容。

`Action`主要指对标注（Label）和连接（Connection）的C~~R~~UD操作（R的操作看下面），如下：

| Action                     | 说明                     | 参数                               |
| -------------------------- | ------------------------ | ---------------------------------- |
| `Action.Label.Create`      | 创建Label                | (categoryId, startIndex, endIndex) |
| `Action.Label.Update`      | 修改Label的category      | (labelId,categoryId)               |
| `Action.Label.Delete`      | 删除Label                | (labelId)                          |
| `Action.Connection.Create` | 创建Connection           | (categoryId, startIndex, endIndex) |
| `Action.Connection.Update` | 修改Connection的category | (connectionId,categoryId)          |
| `Action.Connection.Delete` | 删除Connection           | (connectionId)                     |

##### Example

```typescript
let originString = 'hello world';
let annotator = new Annotator(originString, document.getElementById('test'));
annotator.on('textSelected', (startIndex: number, endIndex: number) => {
    // 先通过某种方式获取用户想要添加的categoryId
    let userChoosedCategoryId = getUserChoosedCategoryId();
    annotator.applyAction(Action.Label.Create(userChoosedCategoryId, startIndex, endIndex));
});
```

### 查询内部状态

`annotator.store`里有各种对象的Repository，可以用来查询各种对象的内容：

| `annotator.store`的成员变量 |
| --------------------------- |
| `content`                   |
| `labelCategoryRepo`         |
| `labelRepo`                 |
| `connectionCategoryRepo`    |
| `connectionRepo`            |

其中`…Repo`都是`Repository`类型，这一类型可以通过与ES6中（以ID为键，对象内容`Entity`为值的）`Map`对象相似的方式使用（即使用`get(id)`方式取对象的信息，也可使用`for-of`遍历）。

⚠️：虽然`Repository`上确实有`add`和`set`方法，而且调用它们确实会有相应的效果，但这些方法仅供内部使用，不建议绕开`Action`向`Repository`直接添加内容！

这一功能的常见作用是为Vue、Angular、React等MVVM框架提供让用户选择`labelCategory`、`connnectionCategory`的控件的数据源。

#### Example

```typescript
let data = [];
for(let [id, entity] of annotator.store.labelCategoryRepo) {
    data.push({id: id, value: entity.text});
}
```

### 序列化

所有`Repository`和`annotator.store`都有`json`属性，直接取值即可得到json对象。

而所有`Entity`都可以使用`JSON.stringify()`序列化。

`annotator.store`对象序列化得到的json可以用作`new Annotator`的第一个参数，来重建Annotator对象。

### 图片导出

目前支持导出到svg文件，只需调用`annotator.export()`来获取到svg文件的内容，然后使用`FileReader`等方式写入文件即可。
