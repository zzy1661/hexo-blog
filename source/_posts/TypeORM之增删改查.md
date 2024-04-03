---
 title: "TypeORM之增删改查"
 date: 2022-02-15
 tags: [Node.js]
 categories: [前端笔记]
---

「这是我参与2022首次更文挑战的第20天，活动详情查看：[2022首次更文挑战](https://juejin.cn/post/7052884569032392740 "https://juejin.cn/post/7052884569032392740")」

Active Record 与 Data Mapper
---------------------------

TypeORM中有两个模式： Active Record 和 Data Mapper，这两个模式的区别在于查询的方式。

### Active Record

Active Record可以通过模型访问数据库：

```ts
const user = new User();
user.firstName = "Timber";
// 保存
await user.save();

// 删除
await user.remove();

// 查询
const users = await User.find({ skip: 2, take: 5 });
const timber = await User.findOne({ firstName: "Timber", lastName: "Saw" });
```

要使用Active Record，entity 必须要继承BaseEntity

```ts
import { BaseEntity, Entity, PrimaryGeneratedColumn, Column } from "typeorm";

@Entity()
export class User extends BaseEntity {
    ...
  
}
```

BaseEntity具有Repository中大部分的方法，无论是实例还是实体类，都可以方便地进行增删改查。如果需要增加自定义的增删改查方法，可以在实体类中增加一个静态方法，使用QueryBuilder：

```ts
@Entity()
export class User extends BaseEntity {
  ...
  static findByName(firstName: string, lastName: string) {
    return this.createQueryBuilder("user")
      .where("user.firstName = :firstName", { firstName })
      .andWhere("user.lastName = :lastName", { lastName })
      .getMany();
  }
}

const timber = await User.findByName("Timber", "Saw");
```

### Data Mapper

Data Mapper模式下需要通过Repository进行增删改查，entity类不需要继承BaseEntity

```ts
// 获取Repository
const userRepository = connection.getRepository(User);

// 保存
const user = new User();
user.firstName = "Timber";
await userRepository.save(user);

// 删除
await userRepository.remove(user);

// 查询
const users = await userRepository.find({ skip: 2, take: 5 });
const timber = await userRepository.findOne({ firstName: "Timber", lastName: "Saw" });
```

如果需要扩展Repository,可以自定义一个类继承Repository：

```ts
import { EntityRepository, Repository } from "typeorm";
import { User } from "../entity/User";

@EntityRepository()
export class UserRepository extends Repository<User> {
  findByName(firstName: string, lastName: string) {
    return this.createQueryBuilder("user")
      .where("user.firstName = :firstName", { firstName })
      .andWhere("user.lastName = :lastName", { lastName })
      .getMany();
  }
}

const userRepository = connection.getCustomRepository(UserRepository);
const timber = await userRepository.findByName("Timber", "Saw");
```

使用QueryBuilder增删改查
------------------

QueryBuilder可以帮助我们以更便捷的方式构建SQL语句：

```ts
const firstUser = await connection
  .getRepository(User)
  .createQueryBuilder("user")
  .where("user.id = :id", { id: 1 })
  .getOne();
```

上面的代码可以返回一条user记录，相当于

```vbnet
SELECT
    user.id as userId,
    user.firstName as userFirstName,
    user.lastName as userLastName
FROM users user
WHERE user.id = 1
```

### 创建QueryBuilder

可以使用connection、entity manager和repository创建QueryBuilder：

```ts
import { getConnection,getManager,getRepository } from "typeorm";
getConnection().createQueryBuilder()...
getManager().createQueryBuilder(User, "user")...
getRepository(User).createQueryBuilder("user")...
```

### 使用QueryBuilder

QueryBuilder实例具有增删改查方法，分别对应insert/delete/update/select

```ts
createQueryBuilder().insert().into(User).values([...])
createQueryBuilder().delete().from(User).where(...)
createQueryBuilder().update(User).set(...)
createQueryBuilder().select("user").from(User, "user")
```

#### QueryBuilder的常用api

*   别名 创建QueryBuilder可以传入一个参数：`createQueryBuilder("user")`，这个参数表示对表设置别名user，之后where语句中可以使用该别名：`.where("user.name = :name", { name: "Timber" });`
*   表达式和参数 QueryBuilder支持多种表达式，如where，having，group by，order by，limit，offset:

```ts
createQueryBuilder("user").where("user.name = :name", { name: "Timber" });
createQueryBuilder("user").offset(10);
...
```

上面`.where("user.name = :name", { name: "Timber" })`等价于

```bash
.where("user.name = :name")
.setParameter("name", "Timber")
```

使用这种方式传参可以避免SQL注入。

*   联合查询 QueryBuilder支持联合查询，比如`createQueryBuilder("user").leftJoinAndSelect()`