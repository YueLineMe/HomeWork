## 总结分割方法 ##

>1、由之前查看省市县的做法，通过s_provinces表创建lagou_city，并添加数据 (根据某个表的数据创建，其使用语法为 create table tablename2 as select * from tablename2)
	
	create table lagou_city  as select s1.id,s3.cityName province,s2.cityName city,s1.cityName district from s_provinces s1 join  s_provinces s2 on s1.parentId=s2.id join s_provinces s3 on s2.parentId=s3.id where s3.depth=1
	union
	select s1.id,s2.cityName,s1.cityName,null from s_provinces s1 join s_provinces s2 on s1.parentId=s2.id where s2.depth=1;
    select * from region ;

>2、根据数据创建表 lagou_company，语法与之前的一致

	create table lagou_company as select distinct t.company_id cid,t.company_short_name shor_name,t.company_full_name full_name,t.company_size size,t.financestage from temp t;

> 3、获得数据

	#首先获得position中district为空的数据
	
	select p.*, c.cid from (select * from lagou_position_bk where district is null) p
    join lagou_city c on c.city like concat(p.city, '%') and c.district is null

或者

	select p.*, c.cid from (select * from lagou_position_bk where district is null) p
    join lagou_city c on LOCATE(p.city,c.city)  and c.district is null

函数 concat（str1,str2,...）详解

	函数concat 返回结果为连接参数产生的字符串。
		如:	SELECT CONCAT('My', 'S', 'QL'); -> 'MySQL'
	如有任何一个参数为NULL ，则返回值为 NULL。
		如： SELECT CONCAT('My', NULL, 'QL');-> NULL
	

函数 LOCATE(str1,str2) 使用详解

	函数 LOCATE 返回子串 substr 在字符串 str 中第一次出现的位置，返回一个数值，该数值为其所在位置

	如:
	
	LOCATE('3','3,2,1') → 1
	LOCATE('3,4','3,2,1') → 0
	LOCATE('bar','foobarbar') → 4

继续数据分离

	#其次查询position 表中 district 不为空的数据

	select p.*, c.cid from (select * from lagou_position_bk where district is not null) p
    join lagou_city c on c.city like concat(p.city, '%') and c.district like concat(p.district, '%')

与之前一样通过同样的语法,将两个数据合并并生成一个表及数据

	create table lagou_position
	as
	select pid, cid as city, company_id as company, position, field, salary_min, salary_max, workyear, education, ptype, pnature, advantage, published_at, updated_at from
	(
	# position 表中 district 为空的数据
	select p.*, c.cid from (select * from lagou_position_bk where district is null) p
    join lagou_city c on c.city like concat(p.city, '%') and c.district is null
  	union all
	# position 表中 district 不为空的数据
	select p.*, c.cid from (select * from lagou_position_bk where district is not null) p
    join lagou_city c on c.city like concat(p.city, '%') and c.district like concat(p.district, '%')
	) as ppp;

至此三个表的分离就此结束！

附加：

	#为公司表添加主键
	alter table lagou_company add primary key (cid) ;

	#为company添加外键约束
	alter table lagou_position add constraint FK_company_cid foreign key(company) REFERENCES company(cid);

	#为城市信息表添加主键
	alter table lagou_city add primary key (id) ;

	#为city添加外键约束
	alter table lagou_position add constraint FK_lagou_city_id foreign key(city) REFERENCES company(id);