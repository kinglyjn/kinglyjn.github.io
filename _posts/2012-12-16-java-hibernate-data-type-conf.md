---
layout: post
title:  "Hibernate data type conf"
desc: "Hibernate data type conf"
keywords: "Hibernate data type conf,hibernate,java,kinglyjn"
date: 2012-12-16
categories: [Java]
tags: [java]
icon: fa-coffee
---

### 实体类

```java
public class MultiTypeDemo {
	private String id;
	private Integer ii;
	private Short sht;
	private Byte bt;
	private Long lg;
	private Float flt;
	private Double dbl;
	private BigDecimal bdml;
	private Boolean bl;
	private Character c;
	private String str;
	private Date d;
	private Date t;
	private Date ts;
	private Calendar cl;
	private Blob image;		//BLOB	blob mediumblob
	private String text;	        //TEXT	text mediumtext
	private Serializable dog;  //BLOB
	private Class<?> clazz;	//class
	private Locale locale;	//locale
	private TimeZone tz;	//timezone;
	private Currency cy;	//currency
	public String getId() {
		return id;
	}
	public void setId(String id) {
		this.id = id;
	}
	public Integer getIi() {
		return ii;
	}
	public void setIi(Integer ii) {
		this.ii = ii;
	}
	public Short getSht() {
		return sht;
	}
	public void setSht(Short sht) {
		this.sht = sht;
	}
	public Byte getBt() {
		return bt;
	}
	public void setBt(Byte bt) {
		this.bt = bt;
	}
	public Long getLg() {
		return lg;
	}
	public void setLg(Long lg) {
		this.lg = lg;
	}
	public Float getFlt() {
		return flt;
	}
	public void setFlt(Float flt) {
		this.flt = flt;
	}
	public Double getDbl() {
		return dbl;
	}
	public void setDbl(Double dbl) {
		this.dbl = dbl;
	}
	public BigDecimal getBdml() {
		return bdml;
	}
	public void setBdml(BigDecimal bdml) {
		this.bdml = bdml;
	}
	public Boolean getBl() {
		return bl;
	}
	public void setBl(Boolean bl) {
		this.bl = bl;
	}
	public Character getC() {
		return c;
	}
	public void setC(Character c) {
		this.c = c;
	}
	public String getStr() {
		return str;
	}
	public void setStr(String str) {
		this.str = str;
	}
	public Date getD() {
		return d;
	}
	public void setD(Date d) {
		this.d = d;
	}
	public Date getT() {
		return t;
	}
	public void setT(Date t) {
		this.t = t;
	}
	public Date getTs() {
		return ts;
	}
	public void setTs(Date ts) {
		this.ts = ts;
	}
	public Calendar getCl() {
		return cl;
	}
	public void setCl(Calendar cl) {
		this.cl = cl;
	}
	public Blob getImage() {
		return image;
	}
	public void setImage(Blob image) {
		this.image = image;
	}
	public String getText() {
		return text;
	}
	public void setText(String text) {
		this.text = text;
	}
	public Serializable getDog() {
		return dog;
	}
	public void setDog(Serializable dog) {
		this.dog = dog;
	}
	public Class<?> getClazz() {
		return clazz;
	}
	public void setClazz(Class<?> clazz) {
		this.clazz = clazz;
	}
	public Locale getLocale() {
		return locale;
	}
	public void setLocale(Locale locale) {
		this.locale = locale;
	}
	public TimeZone getTz() {
		return tz;
	}
	public void setTz(TimeZone tz) {
		this.tz = tz;
	}
	public Currency getCy() {
		return cy;
	}
	public void setCy(Currency cy) {
		this.cy = cy;
	}
	@Override
	public String toString() {
		return "MultiTypeDemo [id=" + id + ", ii=" + ii + ", sht=" + sht
				+ ", bt=" + bt + ", lg=" + lg + ", flt=" + flt + ", dbl=" + dbl
				+ ", bdml=" + bdml + ", bl=" + bl + ", c=" + c + ", str=" + str
				+ ", d=" + d + ", t=" + t + ", ts=" + ts + ", cl=" + cl
				+ ", image=" + image + ", text=" + text + ", dog=" + dog
				+ ", clazz=" + clazz + ", locale=" + locale + ", tz=" + tz
				+ ", cy=" + cy + "]";
	}
}
```
<br><br>


### 实体类的orm映射

```xml
<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
"http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd">
<!-- Generated 2016-5-13 0:53:12 by Hibernate Tools 3.4.0.CR1 -->
<hibernate-mapping>
    <class name="com.demo.coltype.MultiTypeDemo" table="T_MULTITYPE_DEMO">
        <id name="id" type="java.lang.String">
            <column name="ID" />
            <generator class="uuid" />
        </id>
        <property name="ii" type="integer">
            <column name="II" />
        </property>
        <property name="sht" type="short">
            <column name="SHT" />
        </property>
        <property name="bt" type="byte">
            <column name="BT" />
        </property>
        <property name="lg" type="long">
            <column name="LG" />
        </property>
        <property name="flt" type="float">
            <column name="FLT" />
        </property>
        <property name="dbl" type="double">
            <column name="DBL" />
        </property>
        <property name="bdml" type="big_decimal">
            <column name="BDML" />
        </property>
        <property name="bl" type="boolean">
            <column name="BL" />
        </property>
        <property name="c" type="character">
            <column name="C" />
        </property>
        <property name="str" type="string">
            <column name="STR" />
        </property>
        <property name="d" type="date">
            <column name="D" />
        </property>
        <property name="t" type="time">
            <column name="T" />
        </property>
        <property name="ts" type="timestamp">
            <column name="TS" />
        </property>
        <property name="cl" type="calendar">
            <column name="CL" />
        </property>
        <property name="image" type="blob">
            <column name="IMAGE" sql-type="mediumblob"/>
        </property>
        <property name="text" type="text">
            <column name="TEXT" sql-type="mediumtext"/>
        </property>
        <property name="dog" type="serializable">
            <column name="DOG" />
        </property>
        <property name="clazz" type="class">
            <column name="CLAZZ" />
        </property>
        <property name="locale" type="locale">
            <column name="LOCALE" />
        </property>
        <property name="tz" type="timezone">
            <column name="TZ" />
        </property>
        <property name="cy" type="currency">
            <column name="CY" />
        </property>
    </class>
</hibernate-mapping>
```
<br>