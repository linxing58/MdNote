# PostgreSQL实现数据表分区 - 简书
[![](https://upload.jianshu.io/users/upload_avatars/23850670/e550eb62-c240-4ca6-85e0-f655b088210d.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/96/h/96/format/webp)
](https://www.jianshu.com/u/b7ffb23ea5fc)

2020.06.22 15:57:51 字数 120 阅读 2,012

    CREATE TABLE "user" (
      "id" int8 NOT NULL,
      "nickname" varchar(255),
      "grade" int8 DEFAULT NULL,
      "create_date" timestamp DEFAULT NULL,
      "parent_id" int8 DEFAULT NULL,
      "invitation_code" varchar(255) DEFAULT NULL,
      "frozen" int2 DEFAULT 0,
      "update_date" timestamp DEFAULT NULL,
      "phone" varchar(255) DEFAULT NULL,
      "nick_pic" varchar(255) DEFAULT NULL,
      "is_auth" int2 DEFAULT NULL,
      "achievement" int8 DEFAULT NULL,
      PRIMARY KEY ("id")
    ) ;
    --给字段添加注释
    COMMENT ON COLUMN "public"."user"."id" IS 'ID';
    COMMENT ON COLUMN "public"."user"."nickname" IS '用户昵称';
    COMMENT ON COLUMN "public"."user"."grade" IS '会员等级';
    COMMENT ON COLUMN "public"."user"."create_date" IS '用户注册时间';
    COMMENT ON COLUMN "public"."user"."parent_id" IS '用户父ID';
    COMMENT ON COLUMN "public"."user"."invitation_code" IS '用户的邀请码';
    COMMENT ON COLUMN "public"."user"."frozen" IS '用户冻结状态 0代表正常 1代表冻结';
    COMMENT ON COLUMN "public"."user"."update_date" IS '最后一次活跃时间';
    COMMENT ON COLUMN "public"."user"."phone" IS '手机号';
    COMMENT ON COLUMN "public"."user"."nick_pic" IS '用户头像';
    COMMENT ON COLUMN "public"."user"."is_auth" IS '是否实名认证 0-否 1-是';
    COMMENT ON COLUMN "public"."user"."achievement" IS '会员成就'; 

> 根据主表的创建日期月份来建立分区

     create table user01
      (CHECK (extract(month from  create_date) = 1))
      INHERITS (user); 

> 继承主表的分区表无法继承主表的主键字段，需手动创建

    ALTER TABLE user01 ADD PRIMARY KEY (id); 

    create index user01_create_date ON user01(create_date); 

> 通过创建日期月份划分到不同的分区表

    CREATE OR REPLACE FUNCTION  exchange_detail_trigger() RETURNS  trigger AS $$
    BEGIN
    IF (
    extract(month from  NEW .create_time) = 1 
    ) THEN
    INSERT INTO exchange_detail01
    VALUES
    (NEW .*) ;
    ELSEIF (
    extract(month from  NEW .create_time) = 2
    ) THEN
    INSERT INTO exchange_detail02
    VALUES
    (NEW .*) ;
    ELSEIF (
    extract(month from  NEW .create_time) = 3
    ) THEN
    INSERT INTO exchange_detail03
    VALUES
    (NEW .*) ;
    ELSEIF (
    extract(month from  NEW .create_time) = 4
    ) THEN
    INSERT INTO exchange_detail04
    VALUES
    (NEW .*) ;
    ELSEIF(
    extract(month from  NEW .create_time) = 5
    )THEN
    INSERT INTO exchange_detail05
    VALUES
    (NEW .*) ;
    ELSEIF(
    extract(month from  NEW .create_time) = 6
    )THEN
    INSERT INTO exchange_detail06
    VALUES
    (NEW .*) ;
    ELSEIF(
    extract(month from  NEW .create_time) = 7
    )THEN
    INSERT INTO exchange_detail07
    VALUES
    (NEW .*) ;
    ELSEIF(
    extract(month from  NEW .create_time) = 8
    )THEN
    INSERT INTO exchange_detail08
    VALUES
    (NEW .*) ;
    ELSEIF(
    extract(month from  NEW .create_time) = 9
    )THEN
    INSERT INTO exchange_detail09
    VALUES
    (NEW .*) ;
    ELSEIF(
    extract(month from  NEW .create_time) = 10
    )THEN
    INSERT INTO exchange_detail10
    VALUES
    (NEW .*) ;
    ELSEIF(
    extract(month from  NEW .create_time) = 11
    )THEN
    INSERT INTO exchange_detail11
    VALUES
    (NEW .*) ;
    ELSEIF(
    extract(month from  NEW .create_time) = 12
    )THEN
    INSERT INTO exchange_detail12
    VALUES
    (NEW .*) ;
    ELSE
    RAISE EXCEPTION 'Date out of range!' ;
    END
    IF ; RETURN NULL ;
    END ; $$ LANGUAGE plpgsql; 

    CREATE TRIGGER user01_trigger BEFORE INSERT ON user
    FOR EACH ROW
    EXECUTE PROCEDURE user01_trigger_trigger(); 

 [https://www.jianshu.com/p/5a54bba9284f](https://www.jianshu.com/p/5a54bba9284f)
