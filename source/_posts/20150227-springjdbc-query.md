title: java8的lambda表达式与SpringJdbc简单结合
date: 2015-02-27 22:47:00
tags: [Java]
---
虽然jdbc的世界早已被各种ORM框架占领，而且Mybatis等框架确实在大型项目中有非常大的优势，但是在一些小型项目中，大型框架总让人感觉杀鸡用牛刀。

虽然spring自带的JdbcTemplate看起来已经有些过时，但合适的就是最好的，很多项目并没有庞大到非用Mybatis不可。最近在一个项目中使用JdbcTemplate，非常轻便灵活，再配合java8新增的lambda表达式，既满足了需要，也保持了代码的简洁。

### query()方法

jdbcTemplate类中的query方法有很多，这里只使用其中一个

    public <T> T query(String sql, ResultSetExtractor<T> rse, Object... args) throws DataAccessException {
      return query(sql, newArgPreparedStatementSetter(args), rse);
    }

很明显，这里只需要传进sql语句，ResultSetExtractor接口实现，以及sql的参数值，就可以查出结果并且封装为自定义类型。
ResultSetExtractor接口代码：

    public interface ResultSetExtractor<T> {
      T extractData(ResultSet rs) throws SQLException, DataAccessException;
    }

这是一个专供query()查询的回调接口，实现它时必须实现其中的extractData(ResultSet rs)方法，将结果集抽取成自定义类型（可以使用BeanPropertyRowMapper的mapRow方法来处理结果集）。

如果是按照java8之前的语法，我们应该写一个实现类，实现ResultSetExtractor接口，然后new一个对象作为参数传进这个方法。
当然，可以简化一下，在上一篇[关于匿名内部类的文章](http://www.guang.gift/2015/02/25/anonymous-inner-class/)中说过

* 当想要定义一个回调函数且不想编写大量代码时，使用匿名内部类比较便捷。

所以我们可以使用匿名内部类调用这个方法：

jdbcTemplate.query("SELECT * FROM user WHERE id = ?", new ResultSetExtractor<Optional<? extends Object>>() {
            @Override
            public Optional<? extends Object> extractData(ResultSet rs) throws SQLException, DataAccessException {
                return rs.next() ? Optional.of(new BeanPropertyRowMapper<>(User.class).mapRow(rs, 0)) : Optional.empty();
            }
        }, id);

上面的代码实现了从user表中查出一条数据，并封装为自定义的User对象（如果对匿名内部类不太熟悉，请进[这篇文章](http://www.guang.gift/2015/02/25/anonymous-inner-class/)）

### lambda表达式

虽然用了匿名内部类简化了一些，但看上去还是很臃肿，我们不想写这么繁琐的代码怎么办？
这时候，java8的lambda表达式就派上用场了。
首先，我们要理解函数式接口(Functional Interface)的概念。贴一句官方文档对Functional Interface的描述：

    Conceptually, a functional interface has exactly one abstract method.

函数式接口只有一个抽象方法。这个概念虽然是java8中才引入的，但是在之前版本中也有一些符合FI条件的接口，比如Runable接口。
在java8中，函数式接口可以借助lambda表达式实现，比如我们刚才的ResultSetExtractor接口，也是只有一个抽象方法，可以通过lambda简化为：

    jdbcTemplate.query(sql, (ResultSet rs) -> {
            return rs.next() ? Optional.of(asSystem.mapRow(rs, 0)) : Optional.empty();
            }, systemKey);

上面把实现接口的代码去掉，只保留了方法的参数和方法体，用符号-> 相连。代码顿时清爽了许多。
这里的参数类型也可以省略，而且当方法体中只有return语句，可以把return和大括号省略：

    jdbcTemplate.query(sql, (rs) -> rs.next() ? Optional.of(asSystem.mapRow(rs, 0)) : Optional.empty(), id);

当只有一个参数，小括号也可以省略：

    jdbcTemplate.query(sql, rs -> rs.next() ? Optional.of(new BeanPropertyRowMapper<>(System.class).mapRow(rs, 0)) : Optional.empty(), systemKey);

在使用了lambda表达式之后，代码变得非常简洁。
