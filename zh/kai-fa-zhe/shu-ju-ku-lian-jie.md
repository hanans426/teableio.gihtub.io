---
description: 在 Teable 上开放原生数据库连接
---

# 通过 SQL 访问表格数据

本文档将指导您如何在 Teable 中设置和管理数据库连接，包括创建只读连接、获取 BaseId、关闭连接以及了解数据库表中的内容和如何修改数据库列名。

### 创建数据库连接

1. **进入表格**：首先，打开您想要连接的表格。
2. **选择 API 按钮**：在右侧找到并点击 API 按钮。
3. **选择 Postgresql Connection**：在弹出的选项中，选择 Postgresql Connection。
4. **进入设置界面**：点击创建，系统将自动创建一个只读的 Postgres 用户，包括用户名、密码和数据库链接地址。

<figure><img src="../.gitbook/assets/image (62).png" alt="" width="327"><figcaption><p>pg connection</p></figcaption></figure>

#### 连接方式

* **标准 PostgreSQL 客户端**：如 DataGrip, Navcat, TablePlus 等。
* **BI 产品**：如 PowerBI, Metabase, Superset 等。
* **低代码软件**：如 Appsmith, Budibase 等。
* **应用程序代码**：通过选择对应语言的 Postgres Driver。

### 获取 BaseId

1. 点击任意表格进入界面，URL 上方以 `bse` 开头的字符串就是 baseId。

<figure><img src="../.gitbook/assets/image (59).png" alt=""><figcaption><p>baseId</p></figcaption></figure>

2. 进入数据库设计界面（表格右上角的 ⚙ 图标），里面的 schema 名就是 baseId。

<figure><img src="../.gitbook/assets/image (60).png" alt="" width="362"><figcaption><p>Design page</p></figcaption></figure>

### 连接数据库

在不同的产品上，连接数据库的方式略有不同，但大同小异，我们这里进行一个简单的示例

<figure><img src="../.gitbook/assets/image (68).png" alt=""><figcaption><p>Connection example</p></figcaption></figure>

### 关闭数据库连接

1. **进入 Database Connection 界面**：在 Database Connection 界面点击删除，即可关闭连接。
2. **密码清除**：删除后旧的密码将被清除。重新新建将会是新的连接。

### 数据库表中的内容

点击表格右上角的 ⚙️ 图标进入**数据库设计界面**，里面包含了关于当前表格在数据库中的名称和结构详情，对于如何在外部进行数据查询非常有帮助

#### 数据库表名称

修改Teable 中的表名称不会同步修改数据库中的 table name，您必须显式的修改物理数据库中的表名。

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1).png" alt="" width="375"><figcaption><p>table info</p></figcaption></figure>

当你完成表名修改之后，就可以直接使用 SQL 进行查询，下面是一个例子查询当前表格的前 100 行数据

```sql
SELECT * from "bseamGnQT65TVSCzIaC"."clients" limit 100
```

查询的时候需要带上 schema 名称也就是 BaseId, 并使用物理数据库表名进行查询，**请注意在 Postgres 中必须使用双引号`"` 将 schema 和 table 名称包住才能够正确的区分大小写。**

#### **用户创建的字段**

您在表格中创建的所有字段。

注意，数据库表中的字段 (列) 名与用户在 Teable 界面上的命名不会完全一致，请在进行查询前先在数据库设计界面页面检查字段在数据库中的名称，并进行必要的修改。

**修改 teable 中的字段名并不会同步修改 dbFieldName, 您必须显式的修改 dbFieldName 以修改 Postgres 中实际的 column name**

<figure><img src="../.gitbook/assets/image (61).png" alt=""><figcaption><p>db field name</p></figcaption></figure>

#### **Teable 系统字段**

系统字段无法被重命名

1. `__id` 唯一 ID
2. `__version` 版本号
3. `__auto_number` 自增数字，主键
4. `__created_time` 创建时间
5. `__last_modified_time` 最近修改时间
6. `__created_by` 创建人 ID
7. `__last_modified_by` 最近修改人 ID
8. 视图索引字段：以 `__row` 开头的字段，用于维护当前视图中记录的顺序。
9. 外键：如果存在关联字段，会创建逻辑外键，命名以 `__fk` 开头。

#### 中间表

以 `junction_` 开头的表，用于处理 ManyToMany 以及单向关联关系。

### 连接数限制

数据库连接就像是应用和数据库之间的桥梁，用于交换信息。我们的系统允许最多同时建立3条这样的桥梁（也就是连接）。这意味着，同时只能有3个应用直接与数据库交流（一个应用可能占用多条连接，这取决于你的设置）。超过这个数量，其他用户就需要等待直到有连接变得可用。这个限制帮助我们保持系统的稳定和高效，确保每位用户都能获得良好的体验。你可以在连接设置界面查看到当前有几个连接。

### 权限说明

创建的连接用户权限访问范围为当前所在的 Base (Teable界面上称数据库），它对应 Postgres 中的 schema, schema 是 Postgres 中命名空间概念，提供了非常好的权限隔离特性，保证当前连接仅可访问该 schema 中的表，但是 postgres 中创建的角色会拥有所有 schema 的名称查看能力，这也就是为什么当你使用外部应用连接上数据库是时可能会看到下图一样非常多的 baseId (实际上是schema 名称）。但是可以放心的是，除了这个 Id 之外任何其他的用户内容是无法被访问的。

<figure><img src="../.gitbook/assets/image (66).png" alt="" width="253"><figcaption><p>base id in TablePlus</p></figcaption></figure>

### 直接写数据库

Teable 不允许通过外部连接直接操作数据库，直接修改数据库内容是危险的，可能会破坏应用程序的逻辑，导致崩溃或数据丢失。Teable 不提供可写的数据库连接。如果需要修改数据，请通过 Teable API 进行，API 提供了更精细的权限控制和良好的操作封装。

###
