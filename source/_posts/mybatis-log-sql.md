---
title: Mybatis日志还原SQL
date: 2018-07-20 17:27:51
tags: Mybatis
---
## 介绍网站 
[sql.xia-fei.com](http://sql.xia-fei.com)
把mybatis 输出的sql日志还原成完整的sql语句
# 背景
 平时开发过成过程中吧。需要调式mybatis运行的SQL
通长我们会这么做吧
>1.首先我们会查看mybatis的log日志
>2.将日志里面的? 手动替换成参数
# LOG文本还原SQL
<!-- more -->
于是我开发了一个网站将Mybatis  log内容还原成sql
例如日志内容
```sql
2018-07-20 17:32:00.544 superapiplus:WYKlypR6JN1tZndzTMI0yAde416673 DEBUG batchInsert -
				==>  Preparing: insert into sh_push_code (id, user_id, open_id, form_id, form_id_status, generate_time, create_time, update_time, create_person, update_person) values (?, ?, ?, ?, ?, ?, now(), now(), 'system', 'system') 
2018-07-20 17:32:00.544 superapiplus:WYKlypR6JN1tZndzTMI0yAde416673 DEBUG batchInsert -
				==> Parameters: null, 122527041(Long), ouMUI0fPepwDr7YFVvtbONMi_nRk(String), 1532079119360(String), 1(Integer), 2018-07-20 17:32:00.543(Timestamp)
2018-07-20 17:32:00.545 superapiplus:WYKlypR6JN1tZndzTMI0yAde416673 DEBUG batchInsert -
				<==    Updates: 1
2018-07-20 17:32:00.723 sh:6IQwHsHvIHcrOvL3inZFNQde72201 DEBUG selectRolesByStoreId -
				==>  Preparing: select id, store_id, role_code, app_template_id, create_person, create_time, update_person, update_time from sh_role_store where store_id = ? 
2018-07-20 17:32:00.724 sh:6IQwHsHvIHcrOvL3inZFNQde72201 DEBUG selectRolesByStoreId -
				==> Parameters: 1617818(Integer)
2018-07-20 17:32:00.724 sh:6IQwHsHvIHcrOvL3inZFNQde72201 DEBUG selectRolesByStoreId -
				<==      Total: 17
2018-07-20 17:32:00.725 sh:6IQwHsHvIHcrOvL3inZFNQde72201 DEBUG queryByUserIds -
				==>  Preparing: select id, member_id, role_id, star_level, store_id, is_enable, begin_time, end_time, create_person, (select min(d.create_time) from sh_employee_info_detail d WHERE del = 0 and d.member_id = k.member_id) as create_time, update_person, update_time from sh_employee_info_detail k where del = 0 and ? >=begin_time and end_time > ? and member_id in ( ? , ? , ? , ? , ? , ? , ? , ? , ? ) and store_id = ? 
2018-07-20 17:32:00.725 sh:6IQwHsHvIHcrOvL3inZFNQde72201 DEBUG queryByUserIds -
				==> Parameters: 2018-07-20 17:32:00.725(Timestamp), 2018-07-20 17:32:00.725(Timestamp), 122207147(Integer), 122060800(Integer), 122053199(Integer), 122038827(Integer), 120143921(Integer), 121993631(Integer), 120139730(Integer), 120141267(Integer), 120141259(Integer), 1617818(Integer)
2018-07-20 17:32:00.727 sh:6IQwHsHvIHcrOvL3inZFNQde72201 DEBUG queryByUserIds -
				<==      Total: 9
2018-07-20 17:32:00.727 sh:6IQwHsHvIHcrOvL3inZFNQde72201 DEBUG selectHaveEmployeeRole -
				==>  Preparing: select l.*, (select count(*) from sh_employee_info_detail k where k.role_id = s.id and ? >= k.begin_time and k.end_time > ?) as count, s.id as role_id from sh_role_list l join sh_role_store s on l.role_code = s.role_code join sh_employee_info_detail d on d.role_id = s.id where s.store_id = ? and d.del = 0 and ((? >= d.begin_time and d.end_time > ?) or ((? >= d.begin_time and d.end_time > ?)) ) GROUP BY l.role_code order by sort_by 
2018-07-20 17:32:00.728 sh:6IQwHsHvIHcrOvL3inZFNQde72201 DEBUG selectHaveEmployeeRole -
				==> Parameters: 2018-07-01 00:00:00.0(Timestamp), 2018-07-01 00:00:00.0(Timestamp), 1617818(Integer), 2018-07-01 00:00:00.0(Timestamp), 2018-07-01 00:00:00.0(Timestamp), 2018-08-01 00:00:00.0(Timestamp), 2018-08-01 00:00:00.0(Timestamp)
```
我们可以`随意`复制一部分日志内容，粘贴到[sql.xia-fei.com](https://sql.xia-fei.com)
它就会将日志内容转换成可运行的`SQL`

页面结果如图
![](https://xiafei-storage.oss-cn-beijing.aliyuncs.com/18-7-20/27203484.jpg)

各位开发大神,有啥优化改善建议,欢迎在下方留言-,-