# -
create database taobao;
use taobao;

create table user_behavior (user_id int(9), item_id int(9), category_id int(9), behavior_type varchar(5), timestamp int(14) );

use taobao;
desc user_behavior;
select * from user_behavior limit 5;

-- 改变字段名
alter table user_behavior change timestamp timestamps int(14);
desc user_behavior;

-- 检查空值 
select * from user_behavior where user_id is null;
select * from user_behavior where item_id is null;
select * from user_behavior where category_id is null;
select * from user_behavior where behavior_type is null;
select * from user_behavior where timestamps is null;

-- 检查重复值 
select user_id,item_id,timestamps from user_behavior
group by user_id,item_id,timestamps
having count(*)>1;

-- 去重 
alter table user_behavior add id int first;
select * from user_behavior limit 5;
alter table user_behavior modify id int primary key auto_increment;

delete user_behavior from
user_behavior,
(
select user_id,item_id,timestamps,min(id) id from user_behavior
group by user_id,item_id,timestamps
having count(*)>1
) t2
where user_behavior.user_id=t2.user_id
and user_behavior.item_id=t2.item_id
and user_behavior.timestamps=t2.timestamps
and user_behavior.id>t2.id;

-- 新增日期：date time hour
-- 更改buffer值
show VARIABLES like '%_buffer%';
set GLOBAL innodb_buffer_pool_size=1070000000;
-- datetime
alter table user_behavior add datetimes TIMESTAMP(0);
update user_behavior set datetimes=FROM_UNIXTIME(timestamps);
select * from user_behavior limit 5;
-- date
alter table user_behavior add dates char(10);
alter table user_behavior add times char(8);
alter table user_behavior add hours char(2);
-- update user_behavior set dates=substring(datetimes,1,10),times=substring(datetimes,12,8),hours=substring(datetimes,12,2);
update user_behavior set dates=substring(datetimes,1,10);
update user_behavior set times=substring(datetimes,12,8);
update user_behavior set hours=substring(datetimes,12,2);
select * from user_behavior limit 5;

-- 去异常 
select max(datetimes),min(datetimes) from user_behavior;
delete from user_behavior
where datetimes < '2017-11-25 00:00:00'
or datetimes > '2017-12-03 23:59:59';

-- 数据概览 
desc user_behavior;
select * from user_behavior limit 5;
SELECT count(1) from user_behavior; -- 100095496条记录

-- 创建临时表 
create table temp_behavior like user_behavior;

-- 截取 
insert into temp_behavior
select * from user_behavior limit 100000;

select * from temp_behavior;

-- pv 
select dates
,count(*) 'pv'
from temp_behavior
where behavior_type='pv'
GROUP BY dates;
-- uv 
select dates
,count(distinct user_id) 'pv'
from temp_behavior
where behavior_type='pv'
GROUP BY dates;

-- 一条语句 
select dates
,count(*) 'pv'
,count(distinct user_id) 'uv'
,round(count(*)/count(distinct user_id),1) 'pv/uv'
from temp_behavior
where behavior_type='pv'
GROUP BY dates;

-- 处理真实数据 
create table pv_uv_puv (
dates char(10),
pv int(9),
uv int(9),
puv decimal(10,1)
);

insert into pv_uv_puv
select dates
,count(*) 'pv'
,count(distinct user_id) 'uv'
,round(count(*)/count(distinct user_id),1) 'pv/uv'
from user_behavior
where behavior_type='pv'
GROUP BY dates;

select * from pv_uv_puv

delete from pv_uv_puv where dates is null;

-- 留存情况
select * from user_behavior where dates is null;
delete from user_behavior where dates is null;


select user_id,dates 
from temp_behavior
group by user_id,dates;

-- 自关联 
select * from 
(select user_id,dates 
from temp_behavior
group by user_id,dates
) a
,(select user_id,dates 
from temp_behavior
group by user_id,dates
) b
where a.user_id=b.user_id;

-- 筛选 
select * from 
(select user_id,dates 
from temp_behavior
group by user_id,dates
) a
,(select user_id,dates 
from temp_behavior
group by user_id,dates
) b
where a.user_id=b.user_id and a.dates<b.dates;

-- 留存数 
select a.dates
,count(if(datediff(b.dates,a.dates)=0,b.user_id,null)) retention_0
,count(if(datediff(b.dates,a.dates)=1,b.user_id,null)) retention_1
,count(if(datediff(b.dates,a.dates)=3,b.user_id,null)) retention_3 from

(select user_id,dates 
from temp_behavior
group by user_id,dates
) a
,(select user_id,dates 
from temp_behavior
group by user_id,dates
) b
where a.user_id=b.user_id and a.dates<=b.dates
group by a.dates

-- 留存率 
select a.dates
,count(if(datediff(b.dates,a.dates)=1,b.user_id,null))/count(if(datediff(b.dates,a.dates)=0,b.user_id,null)) retention_1
from
(select user_id,dates 
from temp_behavior
group by user_id,dates
) a
,(select user_id,dates 
from temp_behavior
group by user_id,dates
) b
where a.user_id=b.user_id and a.dates<=b.dates
group by a.dates

-- 保存结果 
create table retention_rate (
dates char(10),
retention_1 float 
);

insert into retention_rate 
select a.dates
,count(if(datediff(b.dates,a.dates)=1,b.user_id,null))/count(if(datediff(b.dates,a.dates)=0,b.user_id,null)) retention_1
from
(select user_id,dates 
-- from temp_behavior
-- 纠错，这里漏改为原表了
from user_behavior
group by user_id,dates
) a
,(select user_id,dates 
from user_behavior
group by user_id,dates
) b
where a.user_id=b.user_id and a.dates<=b.dates
group by a.dates

select * from retention_rate

-- 跳失率 
-- 跳失用户  -- 88
select count(*) 
from 
(
select user_id from user_behavior
group by user_id
having count(behavior_type)=1
) a


select sum(pv) from pv_uv_puv; -- 89660670

-- 88/89660670


-- 时间序列分析
-- 统计日期-小时的行为
select dates,hours
,count(if(behavior_type='pv',behavior_type,null)) 'pv'
,count(if(behavior_type='cart',behavior_type,null)) 'cart'
,count(if(behavior_type='fav',behavior_type,null)) 'fav'
,count(if(behavior_type='buy',behavior_type,null)) 'buy'

from temp_behavior
group by dates,hours
order by dates,hours

-- 存储 
create table date_hour_behavior(
dates char(10),
hours char(2),
pv int,
cart int,
fav int,
buy int);

-- 结果插入 
insert into date_hour_behavior
select dates,hours
,count(if(behavior_type='pv',behavior_type,null)) 'pv'
,count(if(behavior_type='cart',behavior_type,null)) 'cart'
,count(if(behavior_type='fav',behavior_type,null)) 'fav'
,count(if(behavior_type='buy',behavior_type,null)) 'buy'

from user_behavior
group by dates,hours
order by dates,hours

select * from date_hour_behavior;


-- 用户转化率分析
-- 统计各类行为用户数 
select behavior_type
,count(DISTINCT user_id) user_num
from temp_behavior
group by behavior_type
order by behavior_type desc;

-- 存储 
create table behavior_user_num(
behavior_type varchar(5),
user_num int);

insert into behavior_user_num
select behavior_type
,count(DISTINCT user_id) user_num
from user_behavior
group by behavior_type
order by behavior_type desc;

select * from behavior_user_num

select 672404/984105 -- 这段时间购买过商品的用户的比例

-- 统计各类行为的数量 
select behavior_type
,count(*) user_num
from temp_behavior
group by behavior_type
order by behavior_type desc;

-- 存储 
create table behavior_num(
behavior_type varchar(5),
behavior_count_num int);

insert into behavior_num
select behavior_type
,count(*) behavior_count_num
from user_behavior
group by behavior_type
order by behavior_type desc;

select * from behavior_num

select 2015807/89660670 -- 购买率 

select (2888255+5530446)/89660670 -- 收藏加购率

-- 行为路径分析
select '难度增加'
drop view user_behavior_view
drop view user_behavior_standard
drop view user_behavior_path
drop view path_count 

create view user_behavior_view as
select user_id,item_id
,count(if(behavior_type='pv',behavior_type,null)) 'pv'
,count(if(behavior_type='fav',behavior_type,null)) 'fav'
,count(if(behavior_type='cart',behavior_type,null)) 'cart'
,count(if(behavior_type='buy',behavior_type,null)) 'buy'
from temp_behavior
group by user_id,item_id

-- 用户行为标准化  
create view user_behavior_standard as
select user_id
,item_id
,(case when pv>0 then 1 else 0 end) 浏览了
,(case when fav>0 then 1 else 0 end) 收藏了
,(case when cart>0 then 1 else 0 end) 加购了
,(case when buy>0 then 1 else 0 end) 购买了
from user_behavior_view

-- 路径类型 

create view user_behavior_path as
select *,
concat(浏览了,收藏了,加购了,购买了) 购买路径类型
from user_behavior_standard as a
where a.购买了>0


-- 统计各类购买行为数量 
create view path_count as
select 购买路径类型
,count(*) 数量
from user_behavior_path
group by 购买路径类型
order by 数量 desc

-- 人话表
create table renhua(
path_type char(4),
description varchar(40));

insert into renhua 
values('0001','直接购买了'),
('1001','浏览后购买了'),
('0011','加购后购买了'),
('1011','浏览加购后购买了'),
('0101','收藏后购买了'),
('1101','浏览收藏后购买了'),
('0111','收藏加购后购买了'),
('1111','浏览收藏加购后购买了')

select * from renhua

select * from path_count p 
join renhua r 
on p.购买路径类型=r.path_type 
order by 数量 desc

-- 存储 
create table path_result(
path_type char(4),
description varchar(40),
num int);


create view user_behavior_view as
select user_id,item_id
,count(if(behavior_type='pv',behavior_type,null)) 'pv'
,count(if(behavior_type='fav',behavior_type,null)) 'fav'
,count(if(behavior_type='cart',behavior_type,null)) 'cart'
,count(if(behavior_type='buy',behavior_type,null)) 'buy'
from user_behavior
group by user_id,item_id

-- 用户行为标准化  
create view user_behavior_standard as
select user_id
,item_id
,(case when pv>0 then 1 else 0 end) 浏览了
,(case when fav>0 then 1 else 0 end) 收藏了
,(case when cart>0 then 1 else 0 end) 加购了
,(case when buy>0 then 1 else 0 end) 购买了
from user_behavior_view

-- 路径类型 

create view user_behavior_path as
select *,
concat(浏览了,收藏了,加购了,购买了) 购买路径类型
from user_behavior_standard as a
where a.购买了>0


-- 统计各类购买行为数量 
create view path_count as
select 购买路径类型
,count(*) 数量
from user_behavior_path
group by 购买路径类型
order by 数量 desc


insert into path_result
select path_type,description,数量 from
path_count p 
join renhua r 
on p.购买路径类型=r.path_type 
order by 数量 desc

select * from path_result

select sum(buy)
from user_behavior_view
where buy>0 and fav=0 and cart=0

 -- 1528016
 
 select 2015807-1528016 -- 487791
 
 select 487791/(2888255+5530446)


-- rfm模型
-- 最近购买时间 
select user_id
,max(dates) '最近购买时间'
from temp_behavior
where behavior_type='buy'
group by user_id
order by 2 desc;

-- 购买次数 
select user_id
,count(user_id) '购买次数'
from temp_behavior
where behavior_type='buy'
group by user_id
order by 2 desc;

-- 统一 
select user_id
,count(user_id) '购买次数'
,max(dates) '最近购买时间'
from user_behavior
where behavior_type='buy'
group by user_id
order by 2 desc,3 desc;

-- 存储
drop table if exists rfm_model;
create table rfm_model(
user_id int,
frequency int,
recent char(10)
);

insert into rfm_model
select user_id
,count(user_id) '购买次数'
,max(dates) '最近购买时间'
from user_behavior
where behavior_type='buy'
group by user_id
order by 2 desc,3 desc;

-- 根据购买次数对用户进行分层 
alter table rfm_model add column fscore int;

update rfm_model
set fscore = case
when frequency between 100 and 262 then 5
when frequency between 50 and 99 then 4
when frequency between 20 and 49 then 3
when frequency between 5 and 20 then 2
else 1
end

-- 根据最近购买时间对用户进行分层 
alter table rfm_model add column rscore int;

update rfm_model
set rscore = case
when recent = '2017-12-03' then 5
when recent in ('2017-12-01','2017-12-02') then 4
when recent in ('2017-11-29','2017-11-30') then 3
when recent in ('2017-11-27','2017-11-28') then 2
else 1
end

select * from rfm_model

-- 分层
set @f_avg=null;
set @r_avg=null;
select avg(fscore) into @f_avg from rfm_model;
select avg(rscore) into @r_avg from rfm_model;

select *
,(case
when fscore>@f_avg and rscore>@r_avg then '价值用户'
when fscore>@f_avg and rscore<@r_avg then '保持用户'
when fscore<@f_avg and rscore>@r_avg then '发展用户'
when fscore<@f_avg and rscore<@r_avg then '挽留用户'
end) class
from rfm_model

-- 插入
alter table rfm_model add column class varchar(40);
update rfm_model
set class = case
when fscore>@f_avg and rscore>@r_avg then '价值用户'
when fscore>@f_avg and rscore<@r_avg then '保持用户'
when fscore<@f_avg and rscore>@r_avg then '发展用户'
when fscore<@f_avg and rscore<@r_avg then '挽留用户'
end


-- 统计各分区用户数
select class,count(user_id) from rfm_model
group by class

-- 商品热度分析
-- 统计商品的热门品类、热门商品、热门品类热门商品
select category_id
,count(if(behavior_type='pv',behavior_type,null)) '品类浏览量'
from temp_behavior
GROUP BY category_id
order by 2 desc
limit 10

select item_id
,count(if(behavior_type='pv',behavior_type,null)) '商品浏览量'
from temp_behavior
GROUP BY item_id
order by 2 desc
limit 10

select category_id,item_id,
品类商品浏览量 from
(
select category_id,item_id
,count(if(behavior_type='pv',behavior_type,null)) '品类商品浏览量'
-- ,rank()over(partition by category_id order by '品类商品浏览量' desc) r
-- 纠错，'品类商品浏览量'这里不能指代count(if(behavior_type='pv',behavior_type,null))因为还没返回
,rank()over(partition by category_id order by count(if(behavior_type='pv',behavior_type,null)) desc) r
from temp_behavior
GROUP BY category_id,item_id
order by 3 desc
) a
where a.r = 1
order by a.品类商品浏览量 desc
limit 10

-- 存储 

create table popular_categories(
category_id int,
pv int);
create table popular_items(
item_id int,
pv int);
create table popular_cateitems(
category_id int,
item_id int,
pv int);


insert into popular_categories
select category_id
,count(if(behavior_type='pv',behavior_type,null)) '品类浏览量'
from user_behavior
GROUP BY category_id
order by 2 desc
limit 10;


insert into popular_items
select item_id
,count(if(behavior_type='pv',behavior_type,null)) '商品浏览量'
from user_behavior
GROUP BY item_id
order by 2 desc
limit 10;

insert into popular_cateitems
select category_id,item_id,
品类商品浏览量 from
(
select category_id,item_id
,count(if(behavior_type='pv',behavior_type,null)) '品类商品浏览量'
,rank()over(partition by category_id order by '品类商品浏览量' desc) r
from user_behavior
GROUP BY category_id,item_id
order by 3 desc
) a
where a.r = 1
order by a.品类商品浏览量 desc
limit 10


select * from popular_categories;
select * from popular_items;
select * from popular_cateitems;

-- 商品转换率分析
-- 特定商品转化率
select item_id
,count(if(behavior_type='pv',behavior_type,null)) 'pv'
,count(if(behavior_type='fav',behavior_type,null)) 'fav'
,count(if(behavior_type='cart',behavior_type,null)) 'cart'
,count(if(behavior_type='buy',behavior_type,null)) 'buy'
,count(distinct if(behavior_type='buy',user_id,null))/count(distinct user_id) 商品转化率
from temp_behavior
group by item_id
order by 商品转化率 desc


-- 保存 
create table item_detail(
item_id int,
pv int,
fav int,
cart int,
buy int,
user_buy_rate float);

insert into item_detail
select item_id
,count(if(behavior_type='pv',behavior_type,null)) 'pv'
,count(if(behavior_type='fav',behavior_type,null)) 'fav'
,count(if(behavior_type='cart',behavior_type,null)) 'cart'
,count(if(behavior_type='buy',behavior_type,null)) 'buy'
,count(distinct if(behavior_type='buy',user_id,null))/count(distinct user_id) 商品转化率
from user_behavior
group by item_id
order by 商品转化率 desc;

select * from item_detail;

-- 品类转化率 

create table category_detail(
category_id int,
pv int,
fav int,
cart int,
buy int,
user_buy_rate float);

insert into category_detail
select category_id
,count(if(behavior_type='pv',behavior_type,null)) 'pv'
,count(if(behavior_type='fav',behavior_type,null)) 'fav'
,count(if(behavior_type='cart',behavior_type,null)) 'cart'
,count(if(behavior_type='buy',behavior_type,null)) 'buy'
,count(distinct if(behavior_type='buy',user_id,null))/count(distinct user_id) 品类转化率
from user_behavior
group by category_id
order by 品类转化率 desc;

select * from category_detail;
