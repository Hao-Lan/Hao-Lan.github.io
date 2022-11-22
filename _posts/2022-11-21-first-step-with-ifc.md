---
layout: single
title:  "迈向 IFC 的第一步"
date:   2022-11-21 10:50:00 +0800
author: lanhao
header:
  overlay_image: /assets/img/ifc/ifcopenshellacademy.png

tags:

- IFC
- ifcopenshell

---

解读IFCOpenshellAcademy 中[示例][IFC First Example].

## 准备阶段

### 载入相关工具库

```python
import uuid
import time
import tempfile
import ifcopenshell
```

### 基础定义

```python
O = 0., 0., 0.
X = 1., 0., 0.
Y = 0., 1., 0.
Z = 0., 0., 1.
```

不难看出,这里是希望定义原点坐标值,XYZ 三个坐标轴的方向向量的值.

**注意**:

此时定义的`O`,`X`,`Y`,`Z`还仅仅是python的基础数据类型:元组.

还未创建成为IFC 里面的点和向量这个概念.

**问题**:

- 里面的值,为何都接一个小数点`.`?

  这个和IFC的 API 强相关.在IFC 创建点和向量的函数中,xyz 三个值,数据类型必须为小数.

### 可复用模块/函数

#### 创建放置坐标系

```python
# Creates an IfcAxis2Placement3D from Location, Axis and RefDirection specified as Python tuples
def create_ifcaxis2placement(ifcfile, point=O, dir1=Z, dir2=X):
    point = ifcfile.createIfcCartesianPoint(point)
    dir1 = ifcfile.createIfcDirection(dir1)
    dir2 = ifcfile.createIfcDirection(dir2)
    axis2placement = ifcfile.createIfcAxis2Placement3D(point, dir1, dir2)
    return axis2placement
```

**问题**：

- 什么是放置坐标?

  参见[IfcAxis2Placement3D][IfcAxis2Placement3D]详细介绍

- `ifcfile` 是什么?

  一个打开的IFC 文件,数据类型由`ifcopenshell`提供,后文会出现.

- `ifcfile.createIfcXXX` 调用说明来自于哪里？

  ifcopenshell提供两种创建 IFC 对象的写法,分别为`createXXX`调用和`create(xxxx,*args,**kwargs)`.

  例如,原代码中创建放置坐标,写法可以改为:
  ```python
  axis2placement = ifcfile.create("IfcAxis2Placement3D", point, dir1, dir2)
  ```

  其他的对象,`IfcCartesianPoint`和`IfcDirection`分别对应点和向量.

  而调用说明,需要参考创建的实体对象规范,如[IfcAxis2Placement3D][IfcAxis2Placement3D].
  ![参数说明](/assets/img/ifc/ifcaxis2placement3d.png)
  **注意**:
    - `?`表示可为空,`[:]`表示传入列表对象,列表元素为表内表述.
    - 调用创建函数,需要按它列出来的序传入,空,请传`None`对象.

#### 创建局部坐标

```python
# Creates an IfcLocalPlacement from Location, Axis and RefDirection, specified as Python tuples, and relative placement
def create_ifclocalplacement(ifcfile, point=O, dir1=Z, dir2=X, relative_to=None):
    axis2placement = create_ifcaxis2placement(ifcfile, point, dir1, dir2)
    ifclocalplacement2 = ifcfile.createIfcLocalPlacement(relative_to, axis2placement)
    return ifclocalplacement2
```

调用说明如前所述,可查看[IfcLocalPlacement][IfcLocalPlacement].
**问题**:

- 放置坐标和局部坐标如何正确理解?

  需要主观去理解,难以文字化陈述.

#### 多段线

```python
# Creates an IfcPolyLine from a list of points, specified as Python tuples
def create_ifcpolyline(ifcfile, point_list):
    ifcpts = []
    for point in point_list:
        point = ifcfile.createIfcCartesianPoint(point)
        ifcpts.append(point)
    polyline = ifcfile.createIfcPolyLine(ifcpts)
    return polyline
```

#### 拉伸创建几何体

```python
def create_ifcextrudedareasolid(ifcfile, point_list, ifcaxis2placement, extrude_dir, extrusion):
    polyline = create_ifcpolyline(ifcfile, point_list)
    ifcclosedprofile = ifcfile.createIfcArbitraryClosedProfileDef("AREA", None, polyline)
    ifcdir = ifcfile.createIfcDirection(extrude_dir)
    ifcextrudedareasolid = ifcfile.createIfcExtrudedAreaSolid(ifcclosedprofile, ifcaxis2placement, ifcdir, extrusion)
    return ifcextrudedareasolid
```

#### 创建唯一ID

```python
create_guid = lambda: ifcopenshell.guid.compress(uuid.uuid1().hex)
```

## 示例:创建墙

### 生成/打开IFC 文件/模板

```python
# IFC template creation
filename = "hello_wall.ifc"
timestamp = time.time()
timestring = time.strftime("%Y-%m-%dT%H:%M:%S", time.gmtime(timestamp))
creator = "Kianwee Chen"
organization = "NUS"
application, application_version = "IfcOpenShell", "0.5"
project_globalid, project_name = create_guid(), "Hello Wall"

# A template IFC file to quickly populate entity instances for an IfcProject with its dependencies
template = """ISO-10303-21;
HEADER;
FILE_DESCRIPTION(('ViewDefinition [CoordinationView]'),'2;1');
FILE_NAME('%(filename)s','%(timestring)s',('%(creator)s'),('%(organization)s'),'%(application)s','%(application)s','');
FILE_SCHEMA(('IFC2X3'));
ENDSEC;
DATA;
#1=IFCPERSON($,$,'%(creator)s',$,$,$,$,$);
#2=IFCORGANIZATION($,'%(organization)s',$,$,$);
#3=IFCPERSONANDORGANIZATION(#1,#2,$);
#4=IFCAPPLICATION(#2,'%(application_version)s','%(application)s','');
#5=IFCOWNERHISTORY(#3,#4,$,.ADDED.,$,#3,#4,%(timestamp)s);
#6=IFCDIRECTION((1.,0.,0.));
#7=IFCDIRECTION((0.,0.,1.));
#8=IFCCARTESIANPOINT((0.,0.,0.));
#9=IFCAXIS2PLACEMENT3D(#8,#7,#6);
#10=IFCDIRECTION((0.,1.,0.));
#11=IFCGEOMETRICREPRESENTATIONCONTEXT($,'Model',3,1.E-05,#9,#10);
#12=IFCDIMENSIONALEXPONENTS(0,0,0,0,0,0,0);
#13=IFCSIUNIT(\*,.LENGTHUNIT.,$,.METRE.);
#14=IFCSIUNIT(\*,.AREAUNIT.,$,.SQUARE_METRE.);
#15=IFCSIUNIT(\*,.VOLUMEUNIT.,$,.CUBIC_METRE.);
#16=IFCSIUNIT(\*,.PLANEANGLEUNIT.,$,.RADIAN.);
#17=IFCMEASUREWITHUNIT(IFCPLANEANGLEMEASURE(0.017453292519943295),#16);
#18=IFCCONVERSIONBASEDUNIT(#12,.PLANEANGLEUNIT.,'DEGREE',#17);
#19=IFCUNITASSIGNMENT((#13,#14,#15,#18));
#20=IFCPROJECT('%(project_globalid)s',#5,'%(project_name)s',$,$,$,$,(#11),#19);
ENDSEC;
END-ISO-10303-21;
""" % locals()

# Write the template to a temporary file 
temp_handle, temp_filename = tempfile.mkstemp(suffix=".ifc")
with open(temp_filename, "wb") as f:
    f.write(template)

# Obtain references to instances defined in template
ifcfile = ifcopenshell.open(temp_filename)
owner_history = ifcfile.by_type("IfcOwnerHistory")[0]
project = ifcfile.by_type("IfcProject")[0]
context = ifcfile.by_type("IfcGeometricRepresentationContext")[0]
```

**问题**:

- `template` 内部是一个IFC 模板文件,如何从零生成IFC 文件?

  创建IFC 文件相对简单:
  ```python
  from ifcopenshell.file import file as IfcFile
  ifcfile = IfcFile()
  ```
  但复杂的,为,需要创建配套的单位,材质,等基础信息.这会占用较大代码篇幅,所以用模板也是一种方法,按需选择即可.
- `owner_history` 什么作用?

几乎在每一个创建实体的调用中都需要传入.用于记录这个操作是由谁发起的.详见`IfcOwnerHistory`说明.

- `project` 什么作用?

一个IFC文件,有且仅有一个`IfcProject`,项目.

- `context` 什么作用?

直译,几何上下文.见`IfcGeometricRepresentationContext`说明.

### 设置层级关系

```python
# IFC hierarchy creation
site_placement = create_ifclocalplacement(ifcfile)
site = ifcfile.createIfcSite(create_guid(), owner_history, "Site", None, None, site_placement, None, None, "ELEMENT",
                             None, None, None, None, None)

building_placement = create_ifclocalplacement(ifcfile, relative_to=site_placement)
building = ifcfile.createIfcBuilding(create_guid(), owner_history, 'Building', None, None, building_placement, None,
                                     None, "ELEMENT", None, None, None)

storey_placement = create_ifclocalplacement(ifcfile, relative_to=building_placement)
elevation = 0.0
building_storey = ifcfile.createIfcBuildingStorey(create_guid(), owner_history, 'Storey', None, None, storey_placement,
                                                  None, None, "ELEMENT", elevation)

container_storey = ifcfile.createIfcRelAggregates(create_guid(), owner_history, "Building Container", None, building,
                                                  [building_storey])
container_site = ifcfile.createIfcRelAggregates(create_guid(), owner_history, "Site Container", None, site, [building])
container_project = ifcfile.createIfcRelAggregates(create_guid(), owner_history, "Project Container", None, project,
                                                   [site])
```

在IFC 定义中,`IfcBuilding`,`IfcBuildingStorey`,`IfcSite`,需要自行去理解.

他们被用来放置某个实体.

`IfcRelAggregates`用来在他们之间建立链接(类似于归属)

### wall

```python
# Wall creation: Define the wall shape as a polyline axis and an extruded area solid
wall_placement = create_ifclocalplacement(ifcfile, relative_to=storey_placement)
polyline = create_ifcpolyline(ifcfile, [(0.0, 0.0, 0.0), (5.0, 0.0, 0.0)])
axis_representation = ifcfile.createIfcShapeRepresentation(context, "Axis", "Curve2D", [polyline])

extrusion_placement = create_ifcaxis2placement(ifcfile, (0.0, 0.0, 0.0), (0.0, 0.0, 1.0), (1.0, 0.0, 0.0))
point_list_extrusion_area = [(0.0, -0.1, 0.0), (5.0, -0.1, 0.0), (5.0, 0.1, 0.0), (0.0, 0.1, 0.0), (0.0, -0.1, 0.0)]
solid = create_ifcextrudedareasolid(ifcfile, point_list_extrusion_area, extrusion_placement, (0.0, 0.0, 1.0), 3.0)
body_representation = ifcfile.createIfcShapeRepresentation(context, "Body", "SweptSolid", [solid])

product_shape = ifcfile.createIfcProductDefinitionShape(None, None, [axis_representation, body_representation])

wall = ifcfile.createIfcWallStandardCase(create_guid(), owner_history, "Wall", "An awesome wall", None, wall_placement,
                                         product_shape, None)
```

通过:封闭多段线 -> 拉伸几何体 -> 墙 的逻辑,创建了`wall`.各个实体在创建时需要的参数都可在文档种查阅.

详见[IfcWallStandardCase][IfcWallStandardCase]

**问题**

- 为何`product_shape`中,包含了一个几何体,又包含了一个`axis_representation`?

  更多是使用方法,IFC中无定义.对比功能可以参见Revit 结构梁柱中的那条细线.

### 材质

```python
material = ifcfile.createIfcMaterial("wall material")
material_layer = ifcfile.createIfcMaterialLayer(material, 0.2, None)
material_layer_set = ifcfile.createIfcMaterialLayerSet([material_layer], None)
material_layer_set_usage = ifcfile.createIfcMaterialLayerSetUsage(material_layer_set, "AXIS2", "POSITIVE", -0.1)
ifcfile.createIfcRelAssociatesMaterial(create_guid(), owner_history, RelatedObjects=[wall],
                                       RelatingMaterial=material_layer_set_usage)
```

创建材质,建立材质和`wall`的联系.

### 属性

```python
property_values = [
    ifcfile.createIfcPropertySingleValue("Reference", "Reference",
                                         ifcfile.create_entity("IfcText", "Describe the Reference"), None),
    ifcfile.createIfcPropertySingleValue("IsExternal", "IsExternal", ifcfile.create_entity("IfcBoolean", True), None),
    ifcfile.createIfcPropertySingleValue("ThermalTransmittance", "ThermalTransmittance",
                                         ifcfile.create_entity("IfcReal", 2.569), None),
    ifcfile.createIfcPropertySingleValue("IntValue", "IntValue", ifcfile.create_entity("IfcInteger", 2), None)
]
property_set = ifcfile.createIfcPropertySet(create_guid(), owner_history, "Pset_WallCommon", None, property_values)
ifcfile.createIfcRelDefinesByProperties(create_guid(), owner_history, None, None, [wall], property_set)

# Add quantity information
quantity_values = [
    ifcfile.createIfcQuantityLength("Length", "Length of the wall", None, 5.0),
    ifcfile.createIfcQuantityArea("Area", "Area of the front face", None, 5.0 \*solid.Depth),
ifcfile.createIfcQuantityVolume("Volume", "Volume of the wall", None, 5.0 \*solid.Depth \*material_layer.LayerThickness)
]
element_quantity = ifcfile.createIfcElementQuantity(create_guid(), owner_history, "BaseQuantities", None, None,
                                                    quantity_values)
ifcfile.createIfcRelDefinesByProperties(create_guid(), owner_history, None, None, [wall], element_quantity)
```

创建属性集合,再建立属性集合同`wall`的联系.

**问题**:

- `IfcElementQuantity`和`IfcPropertySet`区别?

前者定义物理上,长度,体积,面积这类的属性.后者相对普适,具体定义参见文档.

### 开窗

```python
# Create and associate an opening for the window in the wall
opening_placement = create_ifclocalplacement(ifcfile, (0.5, 0.0, 1.0), (0.0, 0.0, 1.0), (1.0, 0.0, 0.0), wall_placement)
opening_extrusion_placement = create_ifcaxis2placement(ifcfile, (0.0, 0.0, 0.0), (0.0, 0.0, 1.0), (1.0, 0.0, 0.0))
point_list_opening_extrusion_area = [(0.0, -0.1, 0.0), (3.0, -0.1, 0.0), (3.0, 0.1, 0.0), (0.0, 0.1, 0.0),
                                     (0.0, -0.1, 0.0)]
opening_solid = create_ifcextrudedareasolid(ifcfile, point_list_opening_extrusion_area, opening_extrusion_placement,
                                            (0.0, 0.0, 1.0), 1.0)
opening_representation = ifcfile.createIfcShapeRepresentation(context, "Body", "SweptSolid", [opening_solid])
opening_shape = ifcfile.createIfcProductDefinitionShape(None, None, [opening_representation])
opening_element = ifcfile.createIfcOpeningElement(create_guid(), owner_history, "Opening", "An awesome opening", None,
                                                  opening_placement, opening_shape, None)

ifcfile.createIfcRelVoidsElement(create_guid(), owner_history, None, None, wall, opening_element)
```

首先在`wall`之上开洞.

**问题**:

- 从本处可以延伸出的开洞两种形式?

    - 第一种,在创建墙的Solid时,就通过几何体布尔运算开洞.
    - 第二种,如上述,完整创建墙(主体)后,再在墙上创建`IfcOpeningElement`开洞.

- 两种开洞区别?

    - `IfcOpeningElement`可以更容易保留开洞信息,易于提取开洞数据.前者更适合用于不规则复杂表示.
    - 如果多个主体(墙)想复用一个局部shape,那么开洞方式需要酌情考虑.

```python
# Create a simplified representation for the Window
window_placement = create_ifclocalplacement(ifcfile, (0.0, 0.0, 0.0), (0.0, 0.0, 1.0), (1.0, 0.0, 0.0),
                                            opening_placement)
window_extrusion_placement = create_ifcaxis2placement(ifcfile, (0.0, 0.0, 0.0), (0.0, 0.0, 1.0), (1.0, 0.0, 0.0))
point_list_window_extrusion_area = [(0.0, -0.01, 0.0), (3.0, -0.01, 0.0), (3.0, 0.01, 0.0), (0.0, 0.01, 0.0),
                                    (0.0, -0.01, 0.0)]
window_solid = create_ifcextrudedareasolid(ifcfile, point_list_window_extrusion_area, window_extrusion_placement,
                                           (0.0, 0.0, 1.0), 1.0)
window_representation = ifcfile.createIfcShapeRepresentation(context, "Body", "SweptSolid", [window_solid])
window_shape = ifcfile.createIfcProductDefinitionShape(None, None, [window_representation])
window = ifcfile.createIfcWindow(create_guid(), owner_history, "Window", "An awesome window", None, window_placement,
                                 window_shape, None, None)

# Relate the window to the opening element
ifcfile.createIfcRelFillsElement(create_guid(), owner_history, None, None, opening_element, window)
```

其次,在开洞的位置放置窗户,即创建窗户.并建立窗和此前洞的联系(填充)

### 放置'容器'内

```python
# Relate the window and wall to the building storey
ifcfile.createIfcRelContainedInSpatialStructure(create_guid(), owner_history, "Building Storey Container", None,
                                                [wall, window], building_storey)

# Write the contents of the file to disk
ifcfile.write(filename)  # 输出至磁盘文件
```

将窗户,墙,都放置在某个选择的容器内.

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
