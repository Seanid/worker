# Vertica 数据清理

#### 公司：HJSD
#### 职位：Java开发工程师
#### 项目代号：G

## 简介
G项目是一个比较综合的系统，这其中涉及到部分数据统计的功能，为此用到了Vertica。Vertica是一个列存储数据库，在数据查询方面有着显著的性能优势。不过有一点不怎么好，并不完全开源，社区版最多允许1TB的原始数据。因此，需要对数据库的数据进行监控，并定时清理无用数据，来保证不违反社区版的规定。

## 监控

    //Vertica查询原始数据使用空间
    SELECT GET_COMPLIANCE_STATUS();

G项目使用python定期通过GET_COMPLIANCE_STATUS查询Vertica的原始数据使用空间，如果发现超过指定限额，会进行报警通知。

TIP：需要特别注意的是，使用GET_COMPLIANCE_STATUS()获取到的数据并非实时变更的的，Vertica会固定时间产生数据，如果手动清除数据，希望GET_COMPLIANCE_STATUS()同步更新，需要人工触发重新统计原始数据使用空间。

## 数据清理



    //查询表压缩后的数据大小，单位GB，此处需要修改的是anchor_table_schema = 'public'
    //查询语句来自：https://woothon.iteye.com/blog/2211252
    SELECT /*+(compressed_table_size)*/
        anchor_table_schema, 
        anchor_table_name, 
        SUM(used_bytes) / ( 1024^3 ) AS used_compressed_gb 
    FROM   v_monitor.projection_storage 
    Where anchor_table_schema = 'public'
    GROUP  BY anchor_table_schema, 
            anchor_table_name 
    ORDER  BY SUM(used_bytes) DESC;

数据清理的时候，一般的目标是找占用空间大的无用数据清理，这样子才能有明显的效果。所以通常查询占用空间最大的几个表，排查无用数据进行清理即可。


## 重新统计

GET_COMPLIANCE_STATUS()所得到的数据，并非实时更新的，如果清理数据后，需要同步到GET_COMPLIANCE_STATUS()，执行下述语句实现。特别注意，该语句需要管理员账号执行。

    //使用管理员账号,重新统计原始数据使用空间
    select audit_license_size()



数据诚可贵，删除需谨慎。


---

#### 点滴知识，源于思考

