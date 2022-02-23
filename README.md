# uView-cascaderList
# **依赖uViewUI的移动端级联字段组件**

在开发小程序的过程中的，由于业务需求，项目中使用的[uViewUI](https://v1.uviewui.com/) 组件库不能满足需求，因此在其基础上开发了此级联(多列)字段展示方案。

## 解决的问题：

uViewUI中的多列选择在某些情况下(选项过多、选项内容过长时)的移动端(小程序)展示效果不够理想：

内容较多，层级过长时选项内容展示被隐藏：

![image](https://user-images.githubusercontent.com/65997647/155294613-ce45f6fd-5560-483b-a0c5-4a1abad98816.png)

针对上述出现的情况，在uView多列选择基础上开发了此组件，此组件非常适合使用uni-app + uViewUI 的小程序项目（其他平台未进行验证）。能够更好的适应移动端的小屏展示场景：

![image](https://user-images.githubusercontent.com/65997647/155297312-86ced388-610f-4ce1-bb8c-58268a58e883.png)
![image](https://user-images.githubusercontent.com/65997647/155297359-e06bf306-ae26-417b-888c-b06424e15ff1.png)

同时该组件对uView多列选择进行了优化：
- 可支持不对称多列数据（例如：1::1-1, 2::2-1::2-1-1 形式）

## API

所有uViewUI[列选择器](https://v1.uviewui.com/components/select.html)API，除`mode`模式选择外。

扩展API：

| 参数      | 类型 | 默认值 | 描述 |
| ----------- | ----------- | ----------- | ----------- |
|listLength|Number| 1 |最大级长度，用于直接设定列数，解决非对称数据列选择按照第一项初始化错误的问题|
|defaultSelectValue|Array<{value:any,label:String}>| [{value:'',label:'请选择'}] | 选择器上方展示的当前选择的值 |



