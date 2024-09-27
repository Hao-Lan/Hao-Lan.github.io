---
layout: post
title: "IFC First Example"
date: 2022-11-21 10:50:00 +0800
categories: [ IFCOpenshell,python,memo]
---

解读 IFCOpenshellAcademy 中[IFC First Example][IFC First Example].

## `ifcfile.createIfcXXX` 调用说明来自于哪里？

ifcopenshell提供两种创建 IFC 对象的写法,分别为`createXXX`调用和`create(xxxx,*args,**kwargs)`.

例如,原代码中创建放置坐标,写法可以改为:

```python
axis2placement = ifcfile.create("IfcAxis2Placement3D", point, dir1, dir2)
```

而调用的参数说明,需要参考创建的实体对象规范,如[IfcAxis2Placement3D][IfcAxis2Placement3D].
![参数说明](/assets/img/ifc/ifcaxis2placement3d.png)
**注意**:

  - `?`表示可为空,`[:]`表示传入列表对象,列表元素为表内表述.
  - 调用创建函数,需要按它列出来的序传入,空,请传`None`对象.

## `template` 内部是一个IFC 模板文件,如何从零生成IFC 文件?

创建IFC 文件相对简单:
```python
from ifcopenshell.file import file as IfcFile
ifcfile = IfcFile()
```
但复杂的,为,需要创建配套的单位,材质,等基础信息.这会占用较大代码篇幅.

所以用模板也是一种方法,按需选择即可.

## 为何`product_shape`中,包含了一个几何体,又包含了一个`axis_representation`?

更多是使用方法,IFC中无定义.对比功能可以参见Revit 结构梁柱中的那条细线.



## 从本处可以延伸出的开洞两种形式?

- 第一种,在创建墙的Solid时,就通过几何体布尔运算开洞.
- 第二种,如上述,完整创建墙(主体)后,再在墙上创建`IfcOpeningElement`开洞.

## 两种开洞区别?

- `IfcOpeningElement`可以更容易保留开洞信息,易于提取开洞数据.前者更适合用于不规则复杂表示.
- 如果多个主体(墙)想复用一个局部shape,那么开洞方式需要酌情考虑.

## 参考资料

- [IFC 参考手册][IFC4 Document]
- [IFC First Example][IFC First Example]
- [IfcAxis2Placement3D][IfcAxis2Placement3D]
- [IfcLocalPlacement][IfcLocalPlacement]

[//]: # (## 参考链接)

[IFC First Example]: https://academy.ifcopenshell.org/posts/creating-a-simple-wall-with-property-set-and-quantity-information/

[IFC4 Document]: https://standards.buildingsmart.org/IFC/DEV/IFC4_3/RC2/HTML/

[IfcAxis2Placement3D]: https://standards.buildingsmart.org/IFC/DEV/IFC4_3/RC2/HTML/schema/ifcgeometryresource/lexical/ifcaxis2placement3d.htm

[IfcLocalPlacement]:https://standards.buildingsmart.org/IFC/DEV/IFC4_3/RC2/HTML/schema/ifcgeometricconstraintresource/lexical/ifclocalplacement.htm

[IfcWallStandardCase]:https://standards.buildingsmart.org/IFC/DEV/IFC4_3/RC2/HTML/schema/ifcsharedbldgelements/lexical/ifcwallstandardcase.htm
