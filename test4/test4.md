# 实验4：对象管理
## 实验目的：
了解Oracle表和视图的概念，学习使用SQL语句Create Table创建表，学习Select语句插入，修改，删除以及查询数据，学习使用SQL语句创建视图，学习部分存储过程和触发器的使用。
## 实验场景：
假设有一个生产某个产品的单位，单位接受网上订单进行产品的销售。通过实验模拟这个单位的部分信息：员工表，部门表，订单表，订单详单表。
## 用户名：du_li
## 实验内容：
### 录入数据：
要求至少有1万个订单，每个订单至少有4个详单。至少有两个部门，每个部门至少有1个员工，其中只有一个人没有领导，一个领导至少有一个下属，并且它的下属是另一个人的领导（比如A领导B，B领导C）。
### 序列的应用:
插入ORDERS和ORDER_DETAILS 两个表的数据时，主键ORDERS.ORDER_ID, ORDER_DETAILS.ID的值必须通过序列SEQ_ORDER_ID和SEQ_ORDER_ID取得，不能手工输入一个数字。
### 触发器的应用：
维护ORDER_DETAILS的数据时（insert,delete,update）要同步更新ORDERS表订单应收货款ORDERS.Trade_Receivable的值。
### 运行脚本建表过程
![image](https://github.com/03DuLi/Oracle/blob/master/test4/a.png)
### 查询某order_id为3040:
```sql
select * from orders where order_id=3040;
select * from order_details where order_id=3040;
```
![image](https://github.com/03DuLi/Oracle/blob/master/test4/b.png)
### 在order_details表中查询到该order_id:
![image](https://github.com/03DuLi/Oracle/blob/master/test4/c.png)
### 进行验证：
![image](https://github.com/03DuLi/Oracle/blob/master/test4/d.png)
因为折扣是固定不变的，所以该验证成功。
### 更新order_id的product_num的数据：
```sql
update order_details set product_num=2 where id=9119;
```
![image](https://github.com/03DuLi/Oracle/blob/master/test4/e.png)
### 更新成功
![image](https://github.com/03DuLi/Oracle/blob/master/test4/f.png)
### order_details金额已改变
![image](https://github.com/03DuLi/Oracle/blob/master/test4/g.png)
### 再次进行验证
![image](https://github.com/03DuLi/Oracle/blob/master/test4/h.png)
验证成功。
### 删除数据
```sql
delete from order_details where order_id=3040;
```
![image](https://github.com/03DuLi/Oracle/blob/master/test4/i.png)
### 插入数据
```sql
insert into order_details values (300001,8022,'paper1',1,3829.72);
```
![image](https://github.com/03DuLi/Oracle/blob/master/test4/k.png)
### 验证插入
![image](https://github.com/03DuLi/Oracle/blob/master/test4/l.png)
