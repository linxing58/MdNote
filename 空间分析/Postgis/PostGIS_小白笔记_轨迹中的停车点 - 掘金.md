# PostGIS_小白笔记_轨迹中的停车点 - 掘金
如果原始 GPS 点没有包括行驶速度等信息，那就需要使用时间和位置这些已知信息，通过额外计算车辆的行驶状态，来找出车辆的停车点。

## 使用 PostGIS 函数

使用到的 PostGIS 函数如下

-   [ST_RemovePoint](https://link.juejin.cn/?target=https%3A%2F%2Fpostgis.net%2Fdocs%2FST_RemovePoint.html "https&#x3A;//postgis.net/docs/ST_RemovePoint.html")
-   [ST_PointN](https://link.juejin.cn/?target=https%3A%2F%2Fpostgis.net%2Fdocs%2FST_PointN.html "https&#x3A;//postgis.net/docs/ST_PointN.html")

业务需求是，如果一系列在 N 分钟内的 GPS 点距离小于等于 M 米，则认为车辆在当前位置停车 N 分钟。

**注意：**  从严格定义来说，我们求的是 “停留点” 而不是“停车点”，因为车辆非常慢的行驶也可能被算法当做是停车了。

通过相邻两个 GPS 点的距离和时间，可以判断是否停车。思路如下

-   遍历轨迹中的每一个点，并把点依次放入停车点集合；
-   如果集合起点 P1 和当前点 PT 的距离小于 M，时间大于 N，则标记 PT 为停车点，并放入当前停车点聚集集合中；
-   检查下一个点，直到 PT 被标记为行驶点，则已经找到当前位置的所有停车点，返回当前集合；
-   重复上诉过程直到处理完所有点；

## 停车点算法

    drop function if exists f_get_staypoints;

    /*
    查找给定行驶轨迹的停车点聚集。
    停车点的判定依据是，如果一系列在N分钟内的GPS点距离小于等于M米，则这些点构成一个停车点聚集，每一个聚集作为一个Linestring返回。
    @traj 轨迹
    @length_meters 用米作为单位的距离阈值
    @duration_mins 用分钟作为单位的停车时长阈值
    */
    create or replace function f_get_staypoints(traj geometry, length_meters int=50, duration_mins int=30)
      returns table(sp_cluster geometry)
    as ?
    declare
      pt geometry;
      p1 geometry;
      segment geometry;
      is_stop boolean;
      prev_stop boolean;
    begin
      p1 := null;
      segment := null;
      is_stop := false;
      prev_stop := false;
      
      -- 遍历轨迹中的每一个点
      for pt in select (st_dumppoints(traj)).geom loop
        -- 1st point 
        if segment is null and p1 is null then
          p1 := pt;
        -- 2nd point  
        elsif segment is null then
          segment := st_makeline(p1, pt);
          p1 := null;
          if st_length(segment::geography) <= length_meters then is_stop := true; end if;
        -- others points
        else
          segment := st_addpoint(segment, pt);
          -- 如果是停车状态就扩展聚集；否则就把聚集中点的时长控制在阈值内
          if not is_stop then
            while st_npoints(segment) > 2 and ( st_m(st_endpoint(segment)) - st_m(st_startpoint(segment)) ) >= duration_mins*60 loop
              segment := st_removepoint(segment, 0);
            end loop;
          end if;
          
          -- 根据距离阈值判定当前状态是否为停车
          if st_length(segment::geography) <= length_meters then is_stop := true; 
          elsif st_distance( st_startpoint(segment)::geography, pt::geography) > length_meters then is_stop := false;
          else is_stop := false;
          end if;
          
          -- 已经找到停车聚集的最后一个点，还需要确保满足必须的停车时长。如果都满足，就返回当前停车点聚集
          if not is_stop and prev_stop then
            -- 终点和起点的距离 是否大于阈值
            if ( st_m(st_pointn(segment, st_npoints(segment)-1)) - st_m(st_startpoint(segment)) ) >= duration_mins*60 then
              sp_cluster := st_removepoint(segment, st_npoints(segment)-1);
              return next;
              segment := null;
              p1 := pt;
            end if;
          end if;
        end if;
        prev_stop := is_stop;
      end loop;
      
      -- 最后一个轨迹点是否是停车点聚集的最后一个点
      if prev_stop and ( st_m(st_endpoint(segment)) - st_m(st_startpoint(segment)) ) >= duration_mins*60 then
        sp_cluster := segment;
        return next;
      end if;
      
    end;
    ? language plpgsql strict;
    复制代码

## 调用算法测试

    -- 停车点聚集
    with sp_cluster as 
    (                                    
      select tr.tid, tr.dt, 
        demo.f_get_staypoints(traj, 1000, 30) as pts 
      from demo.t_taxi_trajectory tr 
      where tr.tid = 1353 and tr.dt = '2008-02-03' 
    ),
    -- 求每一个停车点聚集的信息
    sp_single as
    (
      select 
        spc.tid, spc.dt,
        st_npoints(spc.pts) sp_count,
        st_length( spc.pts::geography )::int len_m,
        to_timestamp( st_m( ST_startpoint(spc.pts ) ))::timestamp without time zone as ts_start,
        to_timestamp( st_m( ST_endpoint(spc.pts ) ))::timestamp without time zone as ts_end,
        st_startpoint(spc.pts ) sp,
        spc.pts
      from sp_cluster spc
    )
    -- 把聚集的第一个点标记为本次停车点
    select sps.tid, sps.dt,
      sps.ts_start, 
      sps.ts_end, 
      (sps.ts_end-sps.ts_start) as duration,
      sps.sp_count,
      sps.len_m,
      st_x(sps.sp) as lng,
      st_y(sps.sp) as lat,
      -- 逆地址查找停车点
      (select f.ext_path from demo.t_area_fence f where st_within(sps.sp, f.fence_polygon) order by f.id desc limit 1) as stay_area,
      sps.sp as staypoint
    from sp_single sps
    order by tid, dt, ts_start
    ;
    复制代码

部分查询结果如下

     tid  |     dt     |      ts_start       |       ts_end        | duration | sp_count | len_m |    lng    |   lat    |      stay_area
    ------+------------+---------------------+---------------------+----------+----------+-------+-----------+----------+----------------------
     1353 | 2008-02-03 | 2008-02-03 06:07:36 | 2008-02-03 08:48:26 | 02:40:50 |       17 |   674 | 116.36887 | 39.85015 | 北京市-北京市-丰台区
     1353 | 2008-02-03 | 2008-02-03 10:58:37 | 2008-02-03 11:38:49 | 00:40:12 |       11 |   805 | 116.37633 |  39.8974 | 北京市-北京市-西城区
     1353 | 2008-02-03 | 2008-02-03 13:22:57 | 2008-02-03 14:32:51 | 01:09:54 |       12 |   797 | 116.36912 | 39.85082 | 北京市-北京市-丰台区
     1353 | 2008-02-03 | 2008-02-03 18:51:22 | 2008-02-03 19:26:44 | 00:35:22 |        5 |   931 | 116.42133 | 39.89582 | 北京市-北京市-东城区
    (4 rows)
    复制代码

 [https://juejin.cn/post/6844904020931264519](https://juejin.cn/post/6844904020931264519)
