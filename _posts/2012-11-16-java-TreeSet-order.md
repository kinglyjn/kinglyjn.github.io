---
layout: post
title:  "从Set里面取出有序的记录"
desc: "从Set里面取出有序的记录"
keywords: "从Set里面取出有序的记录,TreeSet,java,kinglyjn"
date: 2012-11-16
categories: [Java]
tags: [java]
icon: fa-coffee
---

> <i style="font-szie:10px;">Set里面的记录是无序的，如果想使用Set，然后又想里面的记录是有序的，就可以使用TreeSet，而不是HashSet，在使用TreeSet的时候，里面的元素必须是实现了Comparable接口的，TreeSet在进行排序的时候就是通过比较它们的Comparable接口的实现！</i>

```java
import java.util.Set;
import java.util.TreeSet;

public class Test12 {
	public static void main(String[] args) {
		Set<Student> set = new TreeSet<Student>();
		set.add(new Student("张三", 23));
		set.add(new Student("张三", 23));
		set.add(new Student("小娟", 23));
		set.add(new Student("赵六", 28));
		set.add(new Student("王五", 21));
		set.add(new Student("田七", 90));
		
		System.out.println(set.size()); //5
		for (Student student : set) {
			System.out.println(student);
		}
	}
}

class Student implements Comparable<Student> {
	private String name;
	private int score;

	public Student(String name, int score) {
		super();
		this.name = name;
		this.score = score;
	}

	public Student() {
		super();
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public int getScore() {
		return score;
	}

	public void setScore(int score) {
		this.score = score;
	}

	@Override
	public String toString() {
		return "Student [name=" + name + ", score=" + score + "]";
	}

	//
	@Override
	public int compareTo(Student stu) {
		if (stu==null) {
			return 1;
		}
		if (this.score == stu.getScore()) {
			return this.getName().compareTo(stu.getName());
		} else if (this.score > stu.getScore()) {
			return 1;
		} else {
			return -1;
		}
	}

	/*
	  HashSet 只有在 hashCode 返回值冲突的时 候才会调用 equals 方法进行判断，
	  也就是说，两个对象，如果 hashCode 没有冲突，HashSet 就不会调用 equals 
	  方法判断而直接认为这两个对象是不同的对象。
	  综上：
	  如果要正常使用 HashSet 存放对象，为了保证对象的内容不重复，则要求这 个对象满足:
	  1. 覆盖 equals 方法。要求相同的对象，调用 equals 方法返回 true。
	  2. 覆盖 hashCode 方法。要求，相同对象的 hashCode 相同，不同对象的 hashCode 尽量不同。
	*/
	@Override
	public int hashCode() {
		final int prime = 31;
		int result = 1;
		result = prime * result + ((name == null) ? 0 : name.hashCode());
		result = prime * result + score;
		return result;
	}
	//
	@Override
	public boolean equals(Object obj) {
		if (this == obj)
			return true;
		if (obj == null)
			return false;
		if (getClass() != obj.getClass())
			return false;
		Student other = (Student) obj;
		if (name == null) {
			if (other.name != null)
				return false;
		} else if (!name.equals(other.name))
			return false;
		if (score != other.score)
			return false;
		return true;
	}
}
```

运行结果：

```
5
Student [name=王五, score=21]
Student [name=小娟, score=23]
Student [name=张三, score=23]
Student [name=赵六, score=28]
Student [name=田七, score=90]
```

<br>

> [注]<br>
> ArrayList......数组实现，增删慢，查询快<br>
> LinkedList....链表实现，增删快，查询慢<br>
>
> HashMap.......轻量级，速度快，线程不安全，允许键和值为null<br>
> Hashtable......重量级，速度慢，线程安全，不允许键和值为null<br>