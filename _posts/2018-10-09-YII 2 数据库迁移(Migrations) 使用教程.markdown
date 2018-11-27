---
layout: post
title:  "2018-10-09-YII 2 数据库迁移(Migrations) 使用教程"
date:   2018-10-09 17:00:13 +0000
categories: fermin update
---


```
所有操作都需要在项目根目录下操作完成

windows系统在当前根目录打开DOS命令行完成以下操作
```



> ### 0.迁移数据库

一般情况下其他人对数据库有变动时只需要执行如下命令即可在自己的电脑上完成数据库变动的操作

这个指令会提交所有迁移

```
yii migrate
```

执行效果如下图所示：

![1539053658345](https://github.com/fyflzjz/fyflzjz.github.io/blob/master/image/2018-10-09-001/1539053658345.png?raw=true)



根据提示，填写yes

最后输出**Migrated up successfully**表示数据库迁移成功

这时会在数据库中出现一个叫做**migration**的表，用来记录迁移记录，如下图所示：

![1539050047223](https://github.com/fyflzjz/fyflzjz.github.io/blob/master/image/2018-10-09-001/1539050047223.png?raw=true)



至此数据库迁移完成



当前也可以只提交一个迁移

```
yii migrate <name>
```

例如：

  ```
  yii migrate/to m180927_064417_create_page_template
  ```



> ### 1.一些其他迁移命令

还原迁移

```
#还原最近一次迁移
yii migrate/down

#还原最近三次迁移
yii migrate/down 3  
```

重做迁移，意思是先还原指定迁移，然后再次提交

```
#重做最近一次提交的迁移
yii migrate/redo

#重做最近三次提交的迁移
yii migrate/redo 3
```

列出迁移

```
#显示最近10次提交的迁移
yii migrate/history

#显示最近6次提交的迁移
yii migrate/history 6

#显示所有提交的迁移
yii migrate/history all
```

```
#显示最近10次未提交的迁移
yii migrate/new

#显示最近6次未提交的迁移
yii migrate/new 6

#显示所有未提交的迁移
yii migrate/new  all
```



> ### 2.创建数据库迁移

创建一个数据库迁移：

```
yii migrate/create <name>  
```

  这是一个通用的创建数据库迁移格式，其中<name>是必填的参数，用来描述当前迁移

ps：<name>只能是字母、数字、下划线，因为这个指令会生辰一个迁移类，<name>会不是这个类的类名的一部分

例如创建一个page_template表的数据库迁移,命令如下：

```
yii migrate/create create_page_template
```

执行效果如下图所示：

![1539051600146](https://github.com/fyflzjz/fyflzjz.github.io/blob/master/image/2018-10-09-001/1539051600146.png?raw=true)

提示创建，填写yes

此时在项目路径/console/migrations内会按照一定规则生成一个文件名为[m181009_021814_create_page_template.php]的文件

![1539051712362](https://github.com/fyflzjz/fyflzjz.github.io/blob/master/image/2018-10-09-001/1539051712362.png?raw=true)

m181009_021814_create_page_template.php的内容如下：

```php
<?php

use yii\db\Migration;

/**
 * Class m181009_021814_create_page_template
 */
class m181009_021814_create_page_template extends Migration
{
    /**
     * {@inheritdoc}
     */
    public function safeUp()
    {

    }

    /**
     * {@inheritdoc}
     */
    public function safeDown()
    {
        echo "m181009_021814_create_page_template cannot be reverted.\n";

        return false;
    }

    /*
    // Use up()/down() to run migration code without a transaction.
    public function up()
    {

    }

    public function down()
    {
        echo "m181009_021814_create_page_template cannot be reverted.\n";

        return false;
    }
    */
}

```

`safeUp()`用来编写改变数据库结构的代码；`safeDown()`中编写代码来恢复由`safeUp()`所做的改变。当使用migration升级数据库时，`safeUp()`会被调用，反之，`safeDown()`会被调用。 

我们看到还有两个方法`up()`和`down()`。`safeUp()`和 `safeDown()`和`up()`、`down()`的不同点就在于它们是被隐式的封装到事务当中的。 如此一来，只要这些方法里面的任何一个操作失败了，那么所有之前的操作都会被自动的回滚。 



添加数据库迁移内容

```php
<?php

use yii\db\Migration;

/**
 * Class m181009_021814_create_page_template
 */
class m181009_021814_create_page_template extends Migration
{
	
    /**
     * 改变数据表结构的方法
     * {@inheritdoc}
     */
    public function safeUp()
    {
	//创建数据表page_template并且定义表中字段
	$tableOptions = null;
	if ($this->db->driverName === 'mysql') {
		$tableOptions = 'CHARACTER SET utf8mb4 ENGINE=InnoDB COMMENT="单页模板表"';
	}

	$this->createTable('{{%page_template}}', [
		'id' => $this->primaryKey()->comment('主键ID'),
		'member_id' => $this->integer(11)->notNull()->comment('用户ID'),
		'wechat_id' => $this->integer(11)->notNull()->comment('小程序ID'),
		'group_id' => $this->integer(11)->defaultValue('0')->comment('分组ID'),
		'template_json' => $this->text()->comment('模板json串'),
		'page_name' => $this->string(64)->defaultValue('')->comment('页面名称'),
		'background_color' => $this->string(16)->defaultValue('')->comment('背景颜色'),
		'background_image' => $this->string(255)->defaultValue('')->comment('背景图片地址'),
		'is_login' => $this->smallInteger(1)->defaultValue('0')->comment('是否需要登录 0:否 1:是'),
		'is_detail' => $this->smallInteger(1)->defaultValue('0')->comment('是否详情页 0:否 1:是'),
		'is_refresh' => $this->smallInteger(1)->defaultValue('0')->comment('是否可下拉刷新 0:否 1:是'),
		'create_at' => $this->integer(11)->defaultValue(null)->comment('创建时间'),
		'sort' => $this->integer(5)->defaultValue('0')->comment('排序'),
		'reg_ip' => $this->string(255)->defaultValue('')->comment('提交ip地址'),
		'update_at' => $this->integer(11)->defaultValue(null)->comment('修改时间'),
		'is_delete' => $this->smallInteger(1)->defaultValue('0')->comment('是否删除 0:否 1:是'),
	], $tableOptions);

	//插入一条数据
	$this->insert('page_template',[
		'id' => 1,
		'member_id' => '1',
		'wechat_id' => '1',
		'group_id' => '1',
		'template_json' => '{"test":"1234"}',
		'page_name' => '页面名称',
		'background_color' => '#000',
		'background_image' => 'upload/img/1.jpg',
		'is_login' => '1',
		'is_detail' => '1',
		'is_refresh' => '1',
		'create_at' => '1532326325',
		'sort' => '255',
		'reg_ip' => '127.0.0.1',
		'update_at' => '1532326325',
		'is_delete' => '1'
	]);

	//添加字段
	$this->addColumn('page_template', 'test', $this->string(32)->defaultValue('')->comment('test字段'));

    }

    /**
     * 还原数据表改动的方法
     * 执行顺序要与safeUp方法内部操作顺序相反
     * {@inheritdoc}
     */
    public function safeDown()
    {
        //删除字段
        $this->dropColumn('page_template', 'test');

        //删除一条数据
        $this->delete('page_template',[
            'id' => 1
        ]);

        //删除整张表
        $this->dropTable('{{%page_template}}');
    }

    /*
    // Use up()/down() to run migration code without a transaction.
    public function up()
    {

    }

    public function down()
    {
        echo "m181009_021814_create_page_template cannot be reverted.\n";

        return false;
    }
    */
}

```



以下是所有数据库访问方法的列表

```php
yii\db\Migration::execute(): 执行一条 SQL 语句
yii\db\Migration::insert(): 插入单行数据
yii\db\Migration::batchInsert(): 插入多行数据
yii\db\Migration::update(): 更新数据
yii\db\Migration::delete(): 删除数据
yii\db\Migration::createTable(): 创建表
yii\db\Migration::renameTable(): 重命名表名
yii\db\Migration::dropTable(): 删除一张表
yii\db\Migration::truncateTable(): 清空表中的所有数据
yii\db\Migration::addColumn(): 加一个字段
yii\db\Migration::renameColumn(): 重命名字段名称
yii\db\Migration::dropColumn(): 删除一个字段
yii\db\Migration::alterColumn(): 修改字段
yii\db\Migration::addPrimaryKey(): 添加一个主键
yii\db\Migration::dropPrimaryKey(): 删除一个主键
yii\db\Migration::addForeignKey(): 添加一个外键
yii\db\Migration::dropForeignKey(): 删除一个外键
yii\db\Migration::createIndex(): 创建一个索引
yii\db\Migration::dropIndex(): 删除一个索引
```



> ### 常用命令



创建数据库迁移
```
yii migrate/create <name> 
```

执行迁移
```
yii migrate
```

迁移某个
```
yii migrate/to m180927_064417_create_page_template
```

还原最近一次迁移
```
yii migrate/down
```

还原最近三次迁移
```
yii migrate/down 3  
```

重做最近一次提交的迁移
```
yii migrate/redo
```

重做最近三次提交的迁移
```
yii migrate/redo 3
```

显示最近几次提交的迁移
```
yii migrate/history 5
```

显示最近几次未提交的迁移
```
yii migrate/new 5
```

创建表字段模板
```
'group_id' => $this->integer(11)->defaultValue('0')->comment('分组ID'),

'template_json' => $this->text()->comment('模板json串'),

'page_name' => $this->string(64)->defaultValue('')->comment('页面名称'),

'is_login' => $this->tinyInteger(1)->defaultValue('0')->comment('是否需要登录 0:否 1:是'),

'price' => $this->decimal(10,2)->defaultValue('0.00')->comment('金额'),
```

添加字段
```
$this->addColumn('page_template', 'test', $this->string(32)->defaultValue('')->comment('test字段'));
```

删除字段
```
$this->dropColumn('page_template', 'test');
```

删除一条数据
```
$this->delete('page_template',['id' => 1]);
```

删除整张表
```
$this->dropTable('{{%page_template}}');
```

