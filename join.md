
## 查询出所有广东省的市、县做思路总结 ##


>1、查询广东省

	select * from s_provinces where cityName = '广东省';

结果如下：

![](https://i.imgur.com/pAsFbT4.png)
>2、查找所有广东省的市（从数据中看出，当满足s1.parenId=s2.id(父亲的id，也就是上一级，和某条数据的id相同就是当前的下一级)于是采用自己链接自己筛选数据 ）
	
	select s1.id,s2.cityName,s1.cityName from s_provinces s1 join s_provinces s2 on s1.parentId=s2.id where s2.cityName='广东省';

结果如下：

![](https://i.imgur.com/yFSOK3d.png)

>3、再往下查找所有的市下的区

	select s1.id,s3.cityName,s2.cityName,s1.cityName from s_provinces s1 join  s_provinces s2 on s1.parentId=s2.id join s_provinces s3 on s2.parentId=s3.id where s3.cityName='广东省'；

这里与上一个相同思路

结果如下：

![](https://i.imgur.com/LmUIldq.png)

>4、从老师给的提示( union )补上有效数据，将查找市的语句和查找区的语句想结合（使用 union ，不要使用 union all ,虽然在这里结果一样，但是在实际中应使用 union 较好。区别 union 是去出过重复的数据，union 则相反）

	#查找所有广东省的区
	select s1.id,s3.cityName,s2.cityName,s1.cityName from s_provinces s1 join  s_provinces s2 on s1.parentId=s2.id join s_provinces s3 on s2.parentId=s3.id where s3.cityName='广东省'
	union
	# 查找所有广东省底下的市
	select s1.id,s2.cityName,s1.cityName,null from s_provinces s1 join s_provinces s2 on s1.parentId=s2.id where s2.cityName='广东省';

这里在查找广东省下的市加上null这一字段，是因为 union 需要列数相同

结果如下：

![](https://i.imgur.com/WXtOrtl.png)

	
## 结束查询出所有广东省的市、县做思路总结 ##