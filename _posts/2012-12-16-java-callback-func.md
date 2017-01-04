---
layout: post
title:  "java回调函数"
desc: "java回调函数"
keywords: "java回调函数,回调函数,java,kinglyjn"
date: 2012-12-16
categories: [Java]
tags: [java]
icon: fa-coffee
---

### 概述

`回调`通常的实现方式：匿名类。<br>

回调模式的优点：

1. 封装隐藏细节
2. 开发者可以自定义回调函数（new interface(){...}实现）
3. 可以同步，也可以异步回调

<br>

### 同步回调

示例一：

```java
public static List<Person> queryPerson() {
    QueryRunner queryRunner = new QueryRunner(DataSourceSupport.getDataSource());
    return queryRunner.query("select t.name, t.age from person t ", new ResultSetHandler<List<Person>>(){
        List list = new ArrayList<Person>();
        @Override
        public List<Person> handle(ResultSet rs) throws SQLException {
            while(rs.next()) {
                Person person = new Person();
                person.setName(rs.getString(0));
                person.setAge(rs.getInt(1));
                list.add(person);
            }
            return list;
        }
    });
}
```

示例二：

```java
public class RoomMate {
    public void getAnswer(String homework, DoHomeWork someone) {
        if("1+1=?".equals(homework)) {
            someone.doHomeWork(homework, "2");
        } else {
            someone.doHomeWork(homework, "(空白)");
        }
    }

    public static void main(String[] args) {
        RoomMate roomMate = new RoomMate();
        roomMate.getAnswer("1+1=?", new DoHomeWork() {
            @Override
            public void doHomeWork(String question, String answer) {
                System.out.println("问题："+question+" 答案："+answer);
            }
        });
    }
}
public interface DoHomeWork {
    void doHomeWork(String question, String answer);
}
```
<br>


### 异步回调

示例：

```java
public class Student implements DoHomeWork {
	@Override
	public void doHomeWork(String question, String answer) {
		System.out.println("作业本");
		if (answer != null) {
			System.out.println("作业：" + question + " 答案：" + answer);
		} else {
			System.out.println("作业：" + question + " 答案：" + "(空白)");
		}
	}

	public void ask(final String homework, final RoomMate roomMate) {
		new Thread(new Runnable() {

			@Override
			public void run() {
				roomMate.getAnswer(homework, this);
			}
		}).start();
		goHome();
	}

	public void goHome() {
		System.out.println("我回家了……好室友，帮我写下作业。");
	}

	public static void main(String[] args) {
		Student student = new Student();
		String homework = "当x趋向于0，sin(x)/x =?";
		student.ask(homework, new RoomMate());
	}
}

public class RoomMate {
	public void getAnswer(String homework, DoHomeWork someone) {
		if ("1+1=?".equals(homework)) {
			someone.doHomeWork(homework, "2");
		} else if ("当x趋向于0，sin(x)/x =?".equals(homework)) {
			System.out.print("思考：");
			for (int i = 1; i <= 3; i++) {
				System.out.print(i + "秒 ");
				try {
					Thread.sleep(1000);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
			System.out.println();
			someone.doHomeWork(homework, "1");
		} else {
			someone.doHomeWork(homework, "(空白)");
		}
	}
}
```

为了体现异步的意思，我们给好室友设置了个较难的问题，希望好室友能多有点时间思考。新开的线程纯粹用来等待好室友来写完作业。由于在好室友类中设置了3秒的等待时间，所以可以看到goHome方法将先执行。这意味着该学生在告知好室友做作业后，就可以做自己的事情去了，不需要同步阻塞去等待结果。一旦好室友完成作业，写入作业本，该线程也就结束运行了。<br>


