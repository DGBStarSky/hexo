---
title: GO对象排序
date: 2020-04-14 21:30:06
tags: go排序
---

排序需要三个条件

1. **待排序元素个数 n ；** 
2. **第 i 和第 j 个元素的比较函数；**
3. **i 和 第 j 个元素的交换函数。**  

GO的排序在sort包中

sort包中定义了一个Interface接口，如下

{% asset_img 1.png %}

<!--more-->

只要slice（切片）实现了上述三个方法，就可以使用sort()方法对想要排序的类型进行排序。

例如当前想要对结构体切片按照时间排序。

结构体定义如下：

```go
type Topic struct {
	ID              string       `json:"id"`
	Name            string       `json:"name"` //topic名称
	Namespace       string       `json:"namespace"`
	Tenant          string       `json:"tenant"`          //topic的所属租户名称
	TopicGroup      string       `json:"topicGroup"`      //topic所属分组ID
	Partition       int          `json:"partition"`       //topic的分区数量，不指定时默认为1，指定partition大于1，则该topic的消息会被多个broker处理
	IsNonPersistent bool         `json:"isNonPersistent"` //非持久化，默认为false，非必填topic
	URL             string       `json:"url"`             //URL
	CreatedAt       int64        `json:"createdAt"`       //创建Topic的时间戳
	Status          v1.Status    `json:"status"`
	Message         string       `json:"message"`
	Permissions     []Permission `json:"permissions"`
	Users           user.Users   `json:"users"`
	MessageSize     float64      `json:"messageSize"` //消息总量
}
```

想要按照结构体的CreateAt字段进行排序。

定义一个TopicSlice类型，实现Interface的方法。

```go
type TopicSlice TopicList
//重写Interface的len方法
func (t TopicSlice) Len() int{
	return len(t)
}
//重写Interface的Swap方法
func (t TopicSlice) Swap(i,j int) {
	t[i],t[j] = t[j],t[i]
}
//重写Interface的Less方法
func (t TopicSlice) Less(i,j int) bool{
	return t[i].CreatedAt > t[j].CreatedAt
}
```

tps变量为一个Topic切片类型，排序如下

```go
sort.Sort(TopicSlice(tps))
```
