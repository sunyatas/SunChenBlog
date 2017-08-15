---
title: LiteOrm数据库
date: 2016-08-17 20:31
---

最近需要用到数据库保存保存一些数据，由于太久没用数据库，SQLite的语句都忘得七七八八了，所以决定自(xuan)力(ze)更(kai)生(yuan)，google了一下现在比较流行的android数据库框架，有很多，感谢开源的伟大。
<!-- more -->
一开始是想用GreenDao的，这是一个人气很高的库，github主页上随意看了下，发现很强大但规则太多官网文档略长，学习成本有点高而我并不需要那么多的功能。于是又看了看国内的一个叫LiteOrm的库（其实还有一个叫做OrmLite的框架，似乎和GreenDao是相爱相杀的一对，没有去细看），相较起来似乎容易上手的多，小巧精悍。当然很多东西都是有所取舍的，没有绝对的完美，容易上手的不一定其他方面表现的就更好，我只是根据自己的实际需求(懒)进行了选择。

作者在github上对它的介绍如下：

> LiteOrm是一个小巧、强大、比系统自带数据库操作性能快1倍的 android ORM 框架类库，开发者一行代码实现数据库的增删改查操作，以及实体关系的持久化和自动映射。

有点吸引人

开始使用：

作者不知道是什么原因似乎还没有切换使用AS（项目太大不不宜转移？），无法使用gradle进行构建，只好手动导入lib包，只有112k，大小控制的很好。

```
static LiteOrm liteOrm;
  if (liteOrm == null) {
            liteOrm = LiteOrm.newSingleInstance(this, "testorm.db");
        }
        liteOrm.setDebugged(true);
        liteOrm.single().save(testModel);
        testModelArrayList = liteOrm.query(TestModel.class);
```

这是一个非常非常简单的实体类，用注解的方式可以建一个表，非常简单。
```
@Table("test_model")//表名
public class TestModel {

    //主键为每一个表必须有的，自增，这个不说了有点数据库基础都知道...
    @PrimaryKey(AssignType.AUTO_INCREMENT)
    private int id;

    @NotNull
    private String name;


    public int getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public void setId(int id) {
        this.id = id;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "TestModel{" +
                "id=" + id +
                ", name='" + name + '\'' +
                '}';
    }
}
```

添加数据，有很多种方式添加
```
TestModel model = new TestModel();
model.setName(edStr);
liteOrm.insert(model);
```

清空数据
```
 liteOrm.deleteAll(testModel.getClass());
```
查询数据，也有很多种类型可以直接查询
```
testModelArrayList = liteOrm.query(TestModel.class);
```


总结：感觉LiteOrm这个数据库框架还是很好用的，虽然取的名字略有点山寨OrmLite，不过api命名合理，使用很简单，性能方面更是没有问题，好评。

