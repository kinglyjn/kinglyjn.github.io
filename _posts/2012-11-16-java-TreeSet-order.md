---
layout: post
title:  "Collection接口及其子类（List & TreeSet）以及TreeMap进行排序"
desc: "Collection接口及其子类（List & TreeSet）进行排序"
keywords: "Collection接口及其子类（List & TreeSet）进行排序,java,kinglyjn"
date: 2012-11-16
categories: [Java]
tags: [java]
icon: fa-coffee
---

> <i style="font-szie:10px;">集合分为两大类，即Collection（List、ArrayList、LinkedList、Vector；HashSet、LinkedHashSet、TreeSet） 和 Map（HashMap、LinkedHashMap、TreeMap、Hashtable&Properties-古老的实现类线程安全并且键不能为null），Map的key使用set集合进行存储（组成keySet），所以map的key不能重复。使用Collections操作collection及map，使用Arrays操作数组。</i><br>
>
> <i style="font-szie:10px;">Set里面的记录是无序的，如果想使用Set，然后又想里面的记录是有序的，就可以使用TreeSet，而不是HashSet，在使用TreeSet的时候，里面的元素必须是实现了Comparable接口的，TreeSet在进行排序的时候就是通过比较它们的Comparable接口的实现！</i><br>
>
> <i style="font-szie:10px;">HashSet、LinkedHashSet、TreeSet的元素的存储位置都是无序的，由元素的hashCode决定，不同之处在于HashSet是按照hashCode决定的存储位置对Set遍历的；LinkedHashSet维护了一个双向链表，能够按照元素的添加顺序进行遍历；TreeSet是根据其Comparable接口的compareTo方法对其元素进行顺序遍历的。</i><br>
>
> <i style="font-szie:10px;">那么，Set中的元素是如何存储的呢？其实它是使用了hashCode方法，计算当前元素的hashCode值，此hashCode值决定了此元素在Set中的存储位置，若此位置之前没有存储过元素，则这个对象直接被存储到该位置；若此位置之前存储过对象，则再通过equals方法比较前后两个元素是否相同，如果相同则后一个对象就添加不进来，所以一般要求hashCode和equals方法需要一致。</i><br>



### 使用Comparable接口内部实现排序

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

<br>



### 使用Comparator实现外部排序

```java
public static void main(String[] args) {
    List<Integer> list = new ArrayList<Integer>();
    list.add(-3);
    list.add(1);
    list.add(2);

    for (Integer i : list) {
      System.out.println(i);
    } //-3 1 2

    Collections.sort(list, new Comparator<Integer>() {
      @Override
      public int compare(Integer o1, Integer o2) {
        return Math.abs(o1)-Math.abs(o2);
      }
    });

    for (Integer i : list) {
      System.out.println(i);
    } //1 2 -3
}



public static void main(String[] args) {
    Set<Integer> set = new TreeSet<Integer>(new Comparator<Integer>() {
      @Override
      public int compare(Integer o1, Integer o2) {
        return Math.abs(o1)-Math.abs(o2);
      }
    });
    set.add(-3);
    set.add(1);
    set.add(2);

    for (Integer i : set) {
      System.out.println(i);
    } //1 2 -3
}



public static void main(String[] args) {
		Map<Integer, String> map = new TreeMap<Integer,String>(new Comparator<Integer>() {
			@Override
			public int compare(Integer o1, Integer o2) {
				return Math.abs(o1)-Math.abs(o2);
			}
		});
		
		map.put(-3, "aaa");
		map.put(1, "bbb");
		map.put(2, "ccc");
		
		Set<Integer> keySet = map.keySet();
		for (Integer key : keySet) {
			System.out.println(key + "--->" + map.get(key));
		}
  
  		//1--->bbb
        //2--->ccc
        //-3--->aaa
	}
```



### Collections包装collection和map，实现集合类的线程安全

```java
Collection<Integer> sCollection = Collections.synchronizedCollection(list); //

ArrayList<Integer> list = new ArrayList<Integer>(); //list是线程不安全的
List<Integer> sList = Collections.synchronizedList(list); //sList是线程安全的

//类似的可以生成线程安全的set和map
```

