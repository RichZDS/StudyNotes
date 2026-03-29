```sql
-- 增删改
INSERT INTO products (maker, model, type) 
VALUES ('I', '3008', 'printer');
INSERT INTO sales (customer_id, model, quantity, day, paid, type_of_payment)
VALUES ('1112223333', '3008', 2, '2024-01-01', 200, 'cash');
INSERT INTO customers (customer_id, firstname, lastname, city, address, email)
VALUES ('1112223333', 'Lisa', 'Smith', 'Dublin', '5 Main St.', 'lisa@example.com');

-- 更新
UPDATE pcs 
SET price = 2200 
WHERE model = '1001';


UPDATE sales 
SET type_of_payment = 'visa credit' 
WHERE customer_id = '1122334455' AND model = '2010';

UPDATE customers 
SET email = 'norah.jones@yahoo.com' 
WHERE customer_id = '9999999999';

-- 删除
DELETE FROM printers 
WHERE model = '3006';


DELETE FROM sales 
WHERE day = '2013-12-17';

DELETE FROM customers 
WHERE customer_id = '1231231231';




-- 完成数据库新用户创建、
CREATE USER `zds`
-- 授权访问数据库、
GRANT  SELECT, UPDATE, INSERT ON TABLE sales TO zds;

-- 授权用户传递权限（with grant option）
GRANT SELECT ON sdut.sales TO zds WITH GRANT OPTION
-- 收回用户的sales表更新权限
REVOKE UPDATE ON sdut.sales FROM zds;

-- 收回所有权限（彻底回收）
REVOKE ALL PRIVILEGES ON sdut.sales FROM zds
--  删除创建的用户
DROP USER zds;





-- 触发器
-- 1
CREATE TABLE pcprice_log (
    log_id INT AUTO_INCREMENT PRIMARY KEY,
    model VARCHAR(10) NOT NULL,
    old_price DECIMAL(10,2) NOT NULL,
    new_price DECIMAL(10,2) NOT NULL
);


CREATE TRIGGER insert_pcpricelog_values_when_pc_price_changed
AFTER UPDATE ON pcs
FOR EACH ROW
BEGIN
  IF OLD.price != NEW.price THEN
    INSERT INTO pcprice_log (model, old_price, new_price) 
    VALUES (OLD.model, OLD.price, NEW.price);
  END IF;
END;

-- 测试
-- INSERT INTO pcs (model, price, speed, ram, hd ) 
-- VALUES ('1005', 200, 3.0, 8, 512);
-- UPDATE pcs set price=22 WHERE model=1005


-- 2定义AFTER行级触发器，当删除顾客表Customer中的一个顾客信息后，同时删除购买信息表Sales中该顾客的所有购买记录

CREATE TRIGGER deleteCustomer
after DELETE on customers 
for each row 
BEGIN
    DELETE FROM sales 
    WHERE customer_id = OLD.customer_id;
END

-- -- 测试
-- DELETE FROM customers WHERE customer_id=9999999999

-- 3需要对在表上进行DML操作的用户进行安全检查，看是否具有合适的特权
-- 权限表：存储用户可执行的DML权限（JSON数组格式，如["insert","update"]）
CREATE TABLE IF NOT EXISTS userPrivilege (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_name VARCHAR(100) NOT NULL UNIQUE COMMENT '用户名（格式：user@host，如root@localhost，匹配CURRENT_USER()）',
    privilege JSON NOT NULL COMMENT '权限列表，JSON数组，值为"insert"/"update"/"delete"（统一小写）',
    create_time DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '权限创建时间'
);

-- 插入测试数据：给用户zds@localhost分配INSERT/UPDATE权限，不给DELETE权限
INSERT INTO userPrivilege (user_name, privilege)
VALUES ('zds@localhost', '["insert", "update"]');
-- 权限表：存储用户可执行的DML权限（JSON数组格式，如["insert","update"]）
CREATE TABLE IF NOT EXISTS userPrivilege (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_name VARCHAR(100) NOT NULL UNIQUE COMMENT '用户名（格式：user@host，如root@localhost，匹配CURRENT_USER()）',
    privilege JSON NOT NULL COMMENT '权限列表，JSON数组，值为"insert"/"update"/"delete"（统一小写）',
    create_time DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '权限创建时间'
);

-- 插入测试数据：给用户zds@localhost分配INSERT/UPDATE权限，不给DELETE权限
INSERT INTO userPrivilege (user_name, privilege)
VALUES ('zds@localhost', '["insert", "update"]');

DELIMITER //
CREATE TRIGGER trg_check_update_priv
BEFORE UPDATE ON customers -- 同样可替换为目标表
FOR EACH ROW
BEGIN
    DECLARE user_priv JSON;
    DECLARE has_update INT DEFAULT 0;
    DECLARE curr_user VARCHAR(100) DEFAULT CURRENT_USER();

    -- 查询当前用户权限
    SELECT privilege INTO user_priv
    FROM userPrivilege
    WHERE user_name = curr_user;

    -- 判断是否有"update"权限
    IF user_priv IS NOT NULL THEN
        SET has_update = JSON_CONTAINS(user_priv, '"update"');
    END IF;

    -- 无权限抛错
    IF has_update = 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = CONCAT('安全检查失败：用户 [', curr_user, '] 无 ', TABLE_NAME, ' 表的 UPDATE 权限');
    END IF;
END //
DELIMITER ;
DELIMITER //
CREATE TRIGGER trg_check_delete_priv
BEFORE DELETE ON customers -- 同样可替换为目标表
FOR EACH ROW
BEGIN
    DECLARE user_priv JSON;
    DECLARE has_delete INT DEFAULT 0;
    DECLARE curr_user VARCHAR(100) DEFAULT CURRENT_USER();

    -- 查询当前用户权限
    SELECT privilege INTO user_priv
    FROM userPrivilege
    WHERE user_name = curr_user;

    -- 判断是否有"delete"权限
    IF user_priv IS NOT NULL THEN
        SET has_delete = JSON_CONTAINS(user_priv, '"delete"');
    END IF;

    -- 无权限抛错
    IF has_delete = 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = CONCAT('安全检查失败：用户 [', curr_user, '] 无 ', TABLE_NAME, ' 表的 DELETE 权限');
    END IF;
END //
DELIMITER ;

-- 4如果某个特定客户ID的购买记录从销售表中删除后，则从客户表中删除该客户。
CREATE TRIGGER deleteCustomer_by4
AFTER DELETE on sales
for each row 
BEGIN 
        DELETE FROM customers 
        WHERE customer_id = OLD.customer_id;
END

CREATE TRIGGER discount AFTER INSERT ON sales FOR EACH ROW
BEGIN
    DECLARE
      sum_paid DECIMAL ( 12, 2 );
    DECLARE
      discount DECIMAL ( 5, 2 );
    SELECT
      sum( paid ) INTO sum_paid 
    FROM
      sales 
    WHERE
      customer_id = NEW.customer_id;
    SELECT
      discount INTO discount 
    FROM
      customers 
    WHERE
      customer_id = NEW.customer_id;
    IF
      SUM_paid > 10000 
      AND discount < 10.00 THEN
        UPDATE customers 
        SET discount = 10.00 
        WHERE
          customer_id = NEW.customer_id;
        
      END IF;
    
  END
  
  
--   6在客户中添加一个新的"AllPaid"栏，如果插入或更新或删除一个销售元组，那么修改该客户的"AllPaid"的值

ALTER TABLE customers
ADD COLUMN AllPaid DECIMAL(12,2) NOT NULL DEFAULT 0.00 

CREATE TRIGGER  insert_value_customer
AFTER INSERT on customers
for each ROW
BEGIN
 UPDATE customers 
 set AllPaid =(
  SELECT SUM(paid) FROM sales
  WHERE customer_id = NEW.customer_id
 )
 WHERE customer_id = NEW.customer_id;
 END
 
 
 CREATE TRIGGER  UPDATE_value_customer
AFTER UPDATE on customers
for each ROW
BEGIN
 UPDATE customers 
 set AllPaid =(
  SELECT SUM(paid) FROM sales
  WHERE customer_id = NEW.customer_id
 )
 WHERE customer_id = NEW.customer_id;
 END
 
 
 CREATE TRIGGER  DELETE_value_customer
AFTER DELETE on customers
for each ROW
BEGIN
 UPDATE customers 
 set AllPaid =(
  SELECT SUM(paid) FROM sales
  WHERE customer_id = old.customer_id
 )
 WHERE customer_id = old.customer_id;
 END
-- 测试


```

