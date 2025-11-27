CREATE PROCEDURE RateModel ( IN cid VARCHAR ( 20 ), IN m VARCHAR ( 20 ), IN r INT ) BEGIN-- 1. 插入新评分记录
  INSERT INTO ratings ( customer_id, model, rating_time, rating )
  VALUES
    ( cid, m, NOW( ), r ) -- 2. 更新 Products 表的总评分和评分次数
    INSERT IGNORE INTO Products ( total_rating, number_of_ratings )
  VALUES
    ( 0, 0 );-- 3. 计算该型号的最新总评分和评分次数
  UPDATE products 
  SET total_rating = ( SELECT COALESCE( SUM( rating ), 0 ) FROM Ratings WHERE model = m ),
  number_of_ratings = ( SELECT COUNT( * ) FROM Ratings WHERE model = m ) 
  WHERE
    model = m;

  END -- 
-- 2
  CREATE PROCEDURE DeleteRating ( IN cid ( 10 ), IN m ( 10 ), ) BEGIN
    CALL RateModel ( p_customer_id, p_model, NULL );
    
    END -- 3
    CREATE PROCEDURE CleanRatings ( INT cid ( 10 ) INT m ( 10 ) ) BEGIN
      DELETE 
      FROM
        Rating 
      WHERE
        customer_id = cid 
        AND model = m 
        AND rating_time < (
          SELECT
            max_time 
          FROM
            ( SELECT MAX( r2.rating_time ) AS max_time FROM Ratings AS r2 WHERE r2.customer_id = p_customer_id AND r2.model = p_model ) AS t 
        );
      
    END;
    CALL RateModel ( '1122334455', '1001', 3 );
    CALL RateModel ( '1122334455', '1001', 2 );
    CALL RateModel ( '9999999999', '1001', 5 );
    CALL RateModel ( '1122334455', '1001', NULL );
    CALL RateModel ( '9999999999', '1001', 4 );
    CALL RateModel ( '9999999999', '1001', NULL );
    CALL RateModel ( '1122334455', '1001', 2 );
  CALL CleanRatings ( '9999999999', '1001' );
    CALL CleanRatings ( '1122334455', '1001' );
    
    
    
    
--     3
CREATE VIEW AverageRatings(model, average_rating)
AS
SELECT 
    model,
    COALESCE(SUM(rating) / COUNT(*), 0) AS average_rating
FROM Ratings
GROUP BY model
HAVING COUNT(*) > 0; -- 仅包含至少有1次评分的型号


-- 3.b
DELIMITER //
CREATE FUNCTION BestModel()
RETURNS VARCHAR(20)
DETERMINISTIC
BEGIN
    DECLARE max_avg FLOAT;
    DECLARE best_model VARCHAR(20);
    
    -- 步骤1：获取最高平均评分
    SELECT MAX(average_rating) INTO max_avg FROM AverageRatings;
    
    -- 步骤2：若无评分，返回NULL
    IF max_avg IS NULL THEN
        RETURN NULL;
    END IF;
    
    -- 步骤3：获取平均评分最高的所有型号，选择最新非空评分的型号
    SELECT model INTO best_model
    FROM (
        -- 子查询：获取平均评分最高的型号，及其最新非空评分时间
        SELECT 
            ar.model,
            MAX(CASE WHEN r.rating IS NOT NULL THEN r.rating_time ELSE NULL END) AS latest_non_null_time
        FROM AverageRatings ar
        JOIN Ratings r ON ar.model = r.model
        WHERE ar.average_rating = max_avg
        GROUP BY ar.model
        ORDER BY latest_non_null_time DESC -- 按最新非空评分时间降序
        LIMIT 1 -- 取最新的一个
    ) AS temp;
    
    RETURN best_model;
END //
DELIMITER ;


-- 3.c