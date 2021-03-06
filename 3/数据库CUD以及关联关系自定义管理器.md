### 一.Django对数据库数据的增删改

增:

```
方式一:
先创建一个对象
model = Model()
再给对象属性设置对应值
model.name = '小白'
model.age = 15
调用sava方法更新到到数据库
model.save()
方式二:
在model类里面重载父类构造函数(注意是重载不是重写),
model = Model('小白',15)
再调用sava()方法更新到数据库
model.save()
这种方式不是很好,所以需要另外定义一个类方法来创建
def createStudent(name, age,....)
	stu = cls(name=name...)
return stu
最后在views使用
stu = createStudent(...)
stu.save()
方法三:
直接使用objects的create方法
model.objects.create(name='小明', age=15)
```

删:

```
将查询到的数据调用delete()方法从数据库中删除
 stu = model.objects.filter(id__gt=5)
 stu.delete()
```

改:

```
方式一(model里面没有重载或者重写父类的构造函数才行):
stu = model.objects.filter(id=2).first()
stu.name = '小小红'
stu.save()
方式二:
model.objects.filter(id=3).update(name='小兵兵')
```

**总结:尽量不使用重载Init方法的方式去处理**

### 二.数据库之间的一对一,一对多,多对多关系

主要有OneToOneField, ForeignKey,ManyToManyField三种建立连接的方式:
具体看代码:

```
# 一对一和一对多可以直接通过这是属性值的方式去改变 实体间的对应关系(当一对多设置外键唯一的时候,就相当于一对一了)
# 多对多则需要add()和remove()的方式去改变  详细见下:
# 班级表 
class Grade(models.Model):
    g_name = models.CharField(max_length=10)
# 学生表 
class Student(models.Model):
    s_name = models.CharField(max_length=10, unique=True)
    s_age = models.IntegerField(default=16)
    s_sex = models.BooleanField(default=1)
    operator_time = models.DateTimeField(auto_now=True)
    create_time = models.DateTimeField(auto_now_add=True, null=True)
    yuwen = models.DecimalField(max_digits=3, decimal_places=1, default=60)
    shuxue = models.DecimalField(max_digits=3, decimal_places=1, default=60)
    # 学生与班级 多对一
    g = models.ForeignKey(Grade, null=True, related_name="test")
# 学生联系方式表
class StudentInfo(models.Model):
    address = models.CharField(max_length=30, null=True)
    phone = models.IntegerField()
    # 学生的联系方式  一对一
    stu = models.OneToOneField(Student)
# 课程
class Course(models.Model):
    c_name = models.CharField(max_length=10)
    # 学生与课程多对多
    stu = models.ManyToManyField(Student)
    
将某个学生与其他实体添加绑定以及解除绑定
def one_to_one_select(request):
    if request.method == 'GET':
        sql="select phone, address, s_name from tb_student stu,app_studentinfo info where info.stu_id=stu.id"
        """
        方式一:
        stu = Student.objects.filter(id=2).first()
        info = StudentInfo.objects.filter(stu_id=2).first()
        方式二:
        stu = Student.objects.get(id=3)
        info = stu.studentinfo
        方式三:
        info = StudentInfo.objects.get(stu_id=4)
        stu = info.stu
        """
       

# 一对多
def one_to_many_select(request):
    if request.method == 'GET':
        # 通过一来查询多
        # 如果定义了related_name,那么就不能用类名_set去获取了,必须要用related_name的属性值去获取
        # 如果related_name的属性值以'+'结尾的话,那么一对多的查询就无效了(不能执行一对多的查询)
        grade = Grade.objects.get(pk=1)
        students = grade.student_set.all()
        # students = grade.test.all()

        # 通过多来查询一
        # stu = Student.objects.get(pk=5)
        # grade= stu.g
       


# 多对多 给学生添加课程(多对多直接的查询,绑定和解除绑定)
def many_to_many_select(request):
    if request.method == 'GET':
        """
        将学生和课程绑定
        方式一:
        stu = Student.objects.get(pk=4)
        cou = Course.objects.get(pk=1)
        cou1 = Course.objects.get(pk=2)
        stu.course_set.add(cou)
        stu.course_set.add(cou1)
        方式二:
        cou = Course.objects.get(pk=3)
        stu1 = Student.objects.get(pk=19)
        stu2 = Student.objects.get(pk=20)
        cou.stu.add(stu1)
        cou.stu.add(stu2)
        """
        """
        解除绑定  
        方式一      
        stu = Student.objects.get(pk=4)
        cou = Course.objects.get(pk=3)
        stu.course_set.remove(cou)
        方式二:
        cou = Course.objects.get(pk=4)
        stu = Student.objects.get(pk=4)
        cou.stu.remove(stu)
        """
```

### 三.自定义管理器:一般用于查询,过滤掉软删除的数据

我们前面都是直接使用的默认的管理器objects,但是在实际开发中,我们可能会将获取到的数据提前进行过滤,所以就需要我们自定义管理器来实现.

首先定义个一个管理器的子类

```
class StudentsManager(models.manager):
	def get_queryset(self):
	# 过滤掉软删除的数据
	return super(StudentsManager,self).get_queryset().filter(isDelete=False)
```

然后在我们自定义的模型中使用管理器

```
class Student(models.Model):
	stuObject = StudentsManager()
    s_name = models.CharField(max_length=10, unique=True)
    s_age = models.IntegerField(default=16)
    s_sex = models.BooleanField(default=1)
    operator_time = models.DateTimeField(auto_now=True)
    create_time = models.DateTimeField(auto_now_add=True, null=True)
    yuwen = models.DecimalField(max_digits=3, decimal_places=1, default=60)
    shuxue = models.DecimalField(max_digits=3, decimal_places=1, default=60)
    # 学生与班级 多对一
    g = models.ForeignKey(Grade, null=True, related_name="test")
```

在实际使用

```
Student.stuOject.all()  # 这样获取到的数据就是提前过滤后的
```

