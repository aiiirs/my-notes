== = 或 ！= 只能判断基本数据类型 is 关键字只能判断null <=> 既能判断null 又能判断 基本数据类型  ==

外连接时要注意where和on的区别，on是在连接构造临时表时执行的，不管on中条件是否成立都会返回主表（也就是left join左边的表）的内容，where是在临时表形成后执行筛选作用的，不满足条件的整行都会被过滤掉。



#### [608. 树节点](https://leetcode.cn/problems/tree-node/)

![[Pasted image 20221101200326.png]]

```sql
select
    id,
    case when p_id is null then "Root"
         when id not in (select p_id from tree) then "Leaf"
         else "Inner"
    end as Type
from
    tree

#select p_id from tree 子查询可能返回null,not in null返回都为false,因此不会返回"Leaf"
```


```sql
select 
	id, 
	case when p_id is null then "Root" 
		when id not in (select p_id from tree where p_id is not null) then "Leaf" 
		else "Inner" 
	end as "Type" 
from 
	tree

```

Not in null返回均为false

= 或 != 只能判断基本数据类型 ,is 关键字只能判断null ,<=> 既能判断null,又能判断基本数据类型




#### [1158. 市场分析 I](https://leetcode.cn/problems/market-analysis-i/)

![[Pasted image 20221101200637.png]]
![[Pasted image 20221101200735.png]]
![[Pasted image 20221101200759.png]]

```sql
select user_id as buyer_id, join_date, count(order_id) as orders_in_2019
from Users as u left join Orders as o on u.user_id = o.buyer_id and 
//表连接使用where时会将不满足2019的记录排除在外
year(order_date)='2019'
group by user_id

```


外连接时要注意where和on的区别，on是在连接构造临时表时执行的，不管on中条件是否成立都会返回主表（也就是left join左边的表）的内容，where是在临时表形成后执行筛选作用的，不满足条件的整行都会被过滤掉。

