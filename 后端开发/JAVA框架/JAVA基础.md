# JAVA入门

## JAVA特点

- 面向对象:所有代码都在类当中
- 静态语言:变量类型必须提前给定,且不能更改

## Java VS python

## == vs isEqual()









## HashSet

```java
//创建HashSet
Set<String> set = new HashSet<>();
//添加元素
set.add("apple");
//移除元素
set.remove("apple");
//判断是否包含某个元素
boolean contains = set.contains("apple");
```

```java
//set包含元素个数
int size = set.size();
//遍历集合
for(String item:set){
    ......
}
```

## HashMap

基本操作:

```java
//创建HashMap
HashMap<String,Integer> hashmap = new HashMap<>();
//添加元素
hashmap.put("apple",8);
//移除元素
hashmap.remove("apple");

//获取值
if(hashmap.get("apple")!=NULL)
	Integer price = hashmap.get("apple");
//或者:
Integer price = hashmap.getOrDefault("apple",0);
```

遍历键值对(Entry意思是"实体"):

```java
Set<Map.Entry<String,Integer>> entryset = hashmap.entrySet();
for(Map.Entry<String,Integer> entry:entryset){
    String key = entry.getKey();
    String value = entry.getValue();
    ......
}
```

其他操作

```java
//查看是否包含键
hashmap.containsKey("apple");
//查看是否包含值
hashmap.containsValue(8);
//获取键集合
Set<String> keys = hashmap.keyset();
```



平时成绩:60%

期末大作业:40%