
[TOC]
# Android技术前沿：DBFlow的实践

> DBFlow是一个基于AnnotationProcessing(注解处理器)的强大、健壮同时又简单的ORM框架。相比于使用模板代码生成的GreenDao，DBFlow使用更加方便；相比使用反射的ActiveAndroid，在性能有着绝对的优势。
> 此框架设计为了速度、性能和可用性。消除了大量死板的数据库代码，取而代之的是强大而又简洁的API接口。

相关链接：[https://github.com/Raizlabs/DBFlow](https://github.com/Raizlabs/DBFlow)
官方文档：[https://agrosner.gitbooks.io/dbflow/content/](https://agrosner.gitbooks.io/dbflow/content/)

数据持久化主要解决三个问题：

1.  数据库、表的创建
2.  表数据的增删改查操作
3.  数据库的版本管理：版本升级、数据迁移等

本文档将按照这三个问题论述DBFlow的使用

### 数据库、表的创建

创建Java类指定数据库的名称和版本号。
在Application的OnCreate方法中执行DBFlow SDK方法FlowManager.init(context); SDK会自动创建数据库。

我这里写了一个示例，仅供参考：

~~~java
@Database(name = AppDatabase.NAME, version = AppDatabase.VERSION)
public final class AppDatabase {
    private AppDatabase() {
    }

    public static final String NAME = "studio"; // 数据库名称

    public static final int VERSION = 2; // 数据库版本号

    public static void setup(Context context) {
        FlowManager.init(context);
    }

    /**
     * 数据库的修改：
     * 1、PatientSession 表结构的变化
     * 2、增加表字段，考虑到版本兼容性，老版本不建议删除字段
     *
     */
    @Migration(version = 2, database = AppDatabase.class)
    public static class Migration2PS extends AlterTableMigration<PatientSession> {

        public Migration2PS(Class<PatientSession> table) {
            super(table);
        }

        @Override
        public void onPreMigrate() {
            addColumn(SQLiteType.TEXT, "patientId");
            addColumn(SQLiteType.TEXT, "pinyin");
            addColumn(SQLiteType.TEXT, "sortLetters");
            addColumn(SQLiteType.INTEGER, "attention");
        }
    }
}

public class MyApplication extends Application {

    @Override
    public void onCreate() {
        super.onCreate();
        AppDatabase.setup(this);
    }
}
~~~

表的创建也是通过Java类和Annotation来实现，示例如下：

~~~
@Table(database = AppDatabase.class)
public class PatientSession extends BaseModel implements Parcelable {
    @PrimaryKey
    @Column
    public String patientDocId; // 患者档案ID
    @PrimaryKey
    @Column
    public String docId;
    @Column
    public String patientDocName; // 患者档案姓名
    @Column
    public String patientName; // 患者账号微信昵称
    @Column
    public String patientId; // 患者账号微信ID
    @Column
    @SerializedName("note")
    public String noteName; // 备注姓名
    @Column
    public String pinyin;
    @Column
    public String sortLetters;
    ...
}
~~~

所有表对象集成BaseModel，该方法实现了该表对应的增删改查操作，数据库表的操作就直接转换成对Java表对象的操作了，简介明了。关于使用到的基本Annotation，@Table、@PrimaryKey、@Column、@Unique、@ForeignKey等，从名字就可以知晓这是表结构的基本术语。表会在使用的时候进行创建，而不是数据库创建的时候全部创建好。

更多Annotation查阅：
[https://github.com/agrosner/DBFlowDocs/blob/master/Models.md](https://github.com/agrosner/DBFlowDocs/blob/master/Models.md)

### 表的增删改查、外键关联

基本的增删改查操作通过BaseModel中的save()、delete()、insert()、update()方法来实现。

手动查询操作：

~~~
SQLite.select(EmployeeModel_Table.department,
Method.avg(EmployeeModel_Table.salary.as("average_salary")),
                EmployeeModel_Table.title)
      .from(EmployeeModel.class)
      .groupBy(EmployeeModel_Table.department, EmployeeModel_Table.title);

SQLite.select().from(UserProfile.class).where(UserProfile_Table.id.eq(doctorId)).querySingle();

核心类即 SQLite
~~~

事务操作可以通过TransactionManager 提供的相关方法来实现。

~~~
TransactionManager.getInstance().addTransaction(new BaseTransaction() {
                @Override
                public Object onExecute() {
                    Logger.d("Save Sessions in a Transaction");
                    for (PatientSession session : sessions) {
                        session.save();
                    }
                    return null;
                }

                @Override
                public int compareTo(Object another) {
                    return 0;
                }
            });
~~~

外键关联

官方文档说明：
[https://github.com/agrosner/DBFlowDocs/blob/master/Relationships.md](https://github.com/agrosner/DBFlowDocs/blob/master/Relationships.md)

DBFlow提供了外键关联的操作，同大部分ORM框架提供的功能类似，外键关联可实现查询、插入、删除等关联操作。DbFlow当前支持one-one, @OneToMany 和 @ManyToMany三种，one-one好说，后面两个使用时有点坑，简要说明一下。

假设Solution是方案，SolutionItem是方案下面的子项，Solution-SolutionItem构成one-to-many关系，它们的外键关系建立如下：

1.  Solution声明one-to-many 关系

~~~
@ModelContainer
@Table(database = AppDatabase.class)
public class Solution extends BaseModel implements Parcelable {
    @PrimaryKey
    public String solutionCode; // 方案编号
    public List<SolutionItem> solutionItems;

    @OneToMany(methods = {OneToMany.Method.SAVE, OneToMany.Method.DELETE},
            variableName = "solutionItems")
    public List<SolutionItem> getSolutionItems() {
        if (solutionItems == null) {
            solutionItems = new Select()
                    .from(SolutionItem.class)
                    .where(SolutionItem_Table.solution_id.eq(solutionCode))
                    .queryList();
        }
        return solutionItems;
    }
}

要点：@ModelContainer @OneToMany 并且指定methods = {OneToMany.Method.SAVE, OneToMany.Method.DELETE}， 或者指定OneToMany.Method.ALL， 该指定可以实现修改的同步。
~~~

1.  SolutionItem指定外键

~~~
@Table(database = AppDatabase.class)
public class SolutionItem extends BaseModel implements Parcelable, Cloneable {
    @PrimaryKey
    public Integer itemId;
    @Column
    public String itemName;

    @ForeignKey(references = {@ForeignKeyReference(columnName = SOLUTION_ID,
            columnType = String.class, foreignKeyColumnName = "solutionCode")},
            saveForeignKeyModel = false)
    ForeignKeyContainer<Solution> solutionModelContainer;

    public void associateSolution(Solution solution) {
        solutionModelContainer = new ForeignKeyContainer<>(Solution.class);
        solutionModelContainer.setModel(solution);
        // put foreignKey
        solutionModelContainer.put("solutionCode", solution.solutionCode);
    }
}

要点：
外键声明：@ForeignKey
外键关联： void associateSolution(Solution solution)；
~~~

1.  使用方式，通过调用associateSolution方法建立关联关系，然后再做DB操作，我写了一个测试类，代码如下：

~~~
public class SolutionTest {

    @org.junit.Before
    public void setUp() throws Exception {
        Solution solution = new Solution();
        solution.solutionCode = "a";
        List<SolutionItem> solutionItems = new ArrayList<>();
        SolutionItem item1 = new SolutionItem(1, "麻黄");
        SolutionItem item2 = new SolutionItem(2, "三七");
        solutionItems.add(item1);
        solutionItems.add(item2);
        item1.associateSolution(solution);
        item2.associateSolution(solution);
        solution.solutionItems = solutionItems;
        solution.save();
    }

    @org.junit.After
    public void tearDown() throws Exception {

    }

    @Test
    public void testOneToMany() throws Exception {
        List<SolutionItem> items = SQLite.select().from(SolutionItem.class).queryList();
        Assert.assertEquals("one-to-many save success", 2, items.size());

        Solution solution = SQLite.select().from(Solution.class).where(Solution_Table.solutionCode.eq("a")).querySingle();
        Assert.assertEquals("one-to-many get success", 2, solution.getSolutionItems().size());

        solution.delete();

        List<SolutionItem> items2 = SQLite.select().from(SolutionItem.class).queryList();
        Assert.assertEquals("one-to-many delete success", 0, items2.size());
    }
}
~~~

### 数据库的版本管理：版本升级、数据迁移

官方文档说明：
[https://github.com/agrosner/DBFlowDocs/blob/master/Migrations.md](https://github.com/agrosner/DBFlowDocs/blob/master/Migrations.md)

[https://github.com/agrosner/DBFlowDocs/blob/master/Migration3Guide.md](https://github.com/agrosner/DBFlowDocs/blob/master/Migration3Guide.md)

DBFlow提供了三个Migration方法：

1.  AlterTableMigration
2.  IndexMigration/IndexPropertyMigration
3.  UpdateTableMigration

在前面所述的数据库类定义中有一个TableMigration的示例，实现了表字段的增加，只要在在数据库类中声明Migration类，并override 上述三个Migration基类的方法，在重载方法中做数据库升级等操作。

### 结语

以上描述大都从实践的角度来说明，有兴趣的读者可以研读一下DBFlow的代码，特别的，现在Annotation的使用是一个非常好的编码方式，可以学习之。另外需要说明的是，以上的实践也是我基于DBFlow官方文档进行的，读者在实践的时候也要结合官方文档，以免信息传递中出现信息差。

