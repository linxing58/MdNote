# postgis几何函数汇总
POSTGIS 包含大量几何函数，方便我们处理空间数据，本文节选自官方文档，方便查找

#### 5.3 几何构造

`ST_Collect` —从一组几何创建一个 GeometryCollection 或 Multi \*几何。  
`ST_LineFromMultiPoint` —从 MultiPoint 几何图形创建 LineString。  
`ST_MakeEnvelope` —从最小和最大坐标创建一个矩形 Polygon。  
`ST_MakeLine` —从 Point，MultiPoint 或 LineString 几何形状创建线串。  
`ST_MakePoint` —创建 2D，3DZ 或 4D 点。  
`ST_MakePointM` —从 X，Y 和 M 值创建一个点。  
`ST_MakePolygon` —从壳和孔的可选列表创建多边形。  
`ST_Point` —使用给定的坐标值创建一个 Point。 ST_MakePoint 的别名。  
`ST_Polygon` —从具有指定 SRID 的 LineString 创建多边形。  
`ST_TileEnvelope` —使用 XYZ 拼贴系统在 Web Mercator（SRID：3857）中创建一个矩形多边形。  
`ST_HexagonGrid` —返回一组完全覆盖几何参数范围的六边形和单元格索引。  
`ST_SquareGrid` —返回一组完全覆盖几何参数范围的网格正方形和单元格索引。  
`ST_Hexagon` —使用六边形网格空间内提供的边缘大小和像元坐标，返回单个六边形。  
`ST_Square` —使用六角形网格空间内提供的边缘大小和像元坐标，返回单个正方形。

#### 5.4 几何访问

`GeometryType` —以文本形式返回几何的类型。  
`ST_Boundary` —返回几何的边界。  
`ST_CoordDim` —返回几何的坐标尺寸。  
`ST_Dimension` —返回几何的拓扑尺寸。  
`ST_Dump` —返回用于几何图形组件的一组 geometry_dump 行。  
`ST_DumpPoints` —返回几何中的点的一组 geometry_dump 行。  
`ST_DumpRings` —返回多边形的外环和内环的一组 geometry_dump 行。  
`ST_EndPoint` —返回 LineString 或 CircularLineString 的最后一点。  
`ST_Envelope` —返回表示几何图形边界框的几何图形。  
`ST_BoundingDiagonal` —返回几何边界框的对角线。  
`ST_ExteriorRing` —返回代表 Polygon 外环的 LineString。  
`ST_GeometryN` —返回几何集合的第 N 个几何元素。  
`ST_GeometryType` —以文本形式返回几何的 SQL-MM 类型。  
`ST_HasArc` —测试几何图形是否包含圆弧  
`ST_InteriorRingN` —返回多边形的第 N 个内环（孔）。  
`ST_IsPolygonCCW` —测试多边形是否具有沿逆时针方向定位的外环和沿顺时针方向定位的内环。  
`ST_IsPolygonCW` —测试多边形是否具有顺时针方向的外环和逆时针方向的内环。  
`ST_IsClosed` —测试 LineStrings 的起点和终点是否重合。对于 PolyhedralSurface 测试，是否封闭（体积）。  
`ST_IsCollection` —测试几何是否为几何集合类型。  
`ST_IsEmpty` —测试几何是否为空。  
`ST_IsRing` —测试 LineString 是否闭合且简单。  
`ST_IsSimple` —测试几何图形是否没有自相交或自相切的点。  
`ST_M` —返回点的 M 坐标。  
`ST_MemSize` —返回几何占用的内存空间量。  
`ST_NDims` —返回几何的坐标尺寸。  
`ST_NPoints` —返回几何中的点（顶点）数。  
`ST_NRings` —返回多边形几何中的环数。  
`ST_NumGeometries` —返回几何集合中的元素数。  
`ST_NumInteriorRings` —返回多边形的内部环数（孔）。  
`ST_NumInteriorRing` —返回多边形的内部环数（孔）。 ST_NumInteriorRings 的 Aias  
`ST_NumPatches` —返回多面曲面上的面数。对于非多面体几何形状，将返回 null。  
`ST_NumPoints` —返回 LineString 或 CircularString 中的点数。  
`ST_PatchN` —返回多面曲面的第 N 个几何（面）。  
`ST_PointN` —返回几何图形中第一个 LineString 或圆形 LineString 中的第 N 个点。  
`ST_Points` —返回包含几何图形所有坐标的 MultiPoint。  
`ST_StartPoint` —返回 LineString 的第一个点。  
`ST_Summary` —返回几何内容的文本摘要。  
`ST_X` —返回点的 X 坐标。  
`ST_Y` —返回点的 Y 坐标。  
`ST_Z` —返回点的 Z 坐标。  
`ST_Zmflag` —返回指示几何图形的 ZM 坐标尺寸的代码。

#### 5.5 几何编辑

这些函数通过改变类型、结构或顶点来创建修改过的几何图形。  
`ST_AddPoint`—将点添加到 LineString。  
`ST_CollectionExtract` —给定（多）几何，返回仅由指定类型的元素组成的（多）几何。  
`ST_CollectionHomogenize` —给定几何集合，返回内容的 “最简单” 表示形式。  
`ST_CurveToLine` —将包含曲线的几何转换为线性几何。  
`ST_FlipCoordinates` —返回 X 和 Y 轴已翻转的几何版本。  
`ST_Force2D` —将几何强制为 “二维模式”。  
`ST_Force3D` —将几何强制为 XYZ 模式。这是 ST_Force3DZ 的别名。  
`ST_Force3DZ` —将几何强制为 XYZ 模式。  
`ST_Force3DM` —将几何强制为 XYM 模式。  
`ST_Force4D` —将几何强制为 XYZM 模式。  
`ST_ForcePolygonCCW` —逆时针旋转所有外部环，顺时针旋转所有内部环。  
`ST_ForceCollection` —将几何转换为 GEOMETRYCOLLECTION。  
`ST_ForcePolygonCW` —顺时针定向所有外环，逆时针定向所有内环。  
`ST_ForceSFS` —强制几何仅使用 SFS 1.1 几何类型。  
`ST_ForceRHR` —强制多边形中顶点的方向遵循右手规则。  
`ST_ForceCurve` —将几何体转换为其弯曲类型（如果适用）。  
`ST_LineMerge` —返回通过将 MULTILINESTRING 缝合在一起而形成的（一组）LineString。  
`ST_LineToCurve` —将线性几何转换为弯曲几何。  
`ST_Multi` —将几何返回为 MULTI \*几何。  
`ST_Normalize` —以规范形式返回几何。  
`ST_QuantizeCoordinates` —将坐标的最低有效位设置为零  
`ST_RemovePoint` —从线串中删除点。  
`ST_RemoveRepeatedPoints` —返回给定几何的版本，其中删除了重复的点。  
`ST_Reverse` —返回顶点顺序相反的几何。  
`ST_Segmentize` —返回修改后的几何 / 地理，其线段的长度不超过给定距离。  
`ST_SetPoint` —用给定的点替换线串的点。  
`ST_ShiftLongitude` —在 - 180..180 和 0..360 之间移动地理坐标的几何。  
`ST_WrapX` —将几何图形环绕 X 值。  
`ST_SnapToGrid` —将输入几何图形的所有点捕捉到常规网格。  
`ST_Snap` —将输入几何的线段和顶点与参考几何的顶点对齐。  
`ST_SwapOrdinates` —返回交换给定纵坐标值的给定几何的版本。

#### 5.6 几何验证

这些函数根据 OGC SFS 标准测试几何图形是否有效。它们还提供有关残疾的性质和地点的信息。还有一个函数可以从一个无效的几何图形中创建一个有效的几何图形。  
`ST_IsValid` —测试几何是否在 2D 中格式正确。  
`ST_IsValidDetail`—返回 valid_detail 行，该行说明几何是否有效，如果不是，则说明原因和位置。  
`ST_IsValidReason` —返回说明几何是否有效或无效原因的文本。  
`ST_MakeValid` —尝试使无效的几何图形有效而不丢失顶点。

#### 5.7 空间参考系统功能

这些功能与几何的空间参考系统一起工作。  
`ST_SetSRID` —将几何图形上的 SRID 设置为特定的整数值。  
`ST_SRID` —返回空间索引表中定义的 ST_Geometry 的空间参考标识符。  
`ST_Transform` —返回一个新几何，其坐标已转换为其他空间参考系统。

#### 5.8 几何输入

这些函数从各种文本或二进制格式创建几何体对象。

##### 5.8.1. Well-Known Text (WKT)

`ST_BdPolyFromText` — 构造一个多边形给定一个封闭的字符串作为一个多行众所周知的文本表示的任意集合。  
`ST_BdMPolyFromText` — 构造一个多多边形，给定一个封闭的字符串的任意集合，作为一个多字符串文本表示法众所周知的文本表示法。  
`ST_GeogFromText` — 从已知的文本表示或扩展（WKT）返回指定的地理值。  
`ST_GeographyFromText` — 从已知的文本表示或扩展（WKT）返回指定的地理值。  
`ST_GeomCollFromText` — 使用给定的 SRID 从集合 WKT 生成集合几何体。如果未给定 SRID，则默认为 0。  
`ST_GeomFromEWKT` — 从扩展的已知文本表示法（EWKT）返回指定的`ST_Geometry`。  
`ST_GeometryFromText` — 从已知文本表示法（WKT）返回指定的`ST_Geometry`。这是`ST_GeomFromText`的别名  
`ST_GeomFromText` — 从已知文本表示法（WKT）返回指定的`ST_Geometry`。  
`ST_LineFromText` — 使用给定的 SRID 从 WKT 表示生成几何体。如果未给定 SRID，则默认为 0。  
`ST_MLineFromText` — 从 WKT 表示返回指定的 stu MultiLineString 值。  
`ST_MPointFromText` — 使用给定的 SRID 从 WKT 生成几何体。如果未给定 SRID，则默认为 0。  
`ST_MPolyFromText` — 使用给定的 SRID 从 WKT 生成多多边形几何体。如果未给定 SRID，则默认为 0。  
`ST_PointFromText` — 使用给定的 SRID 从 WKT 生成点几何体。如果未给定 SRID，则默认为未知。  
`ST_PolygonFromText` — 使用给定的 SRID 从 WKT 生成几何体。如果未给定 SRID，则默认为 0。  
`ST_WKTToSQL` — 从已知文本表示法（WKT）返回指定的`ST_Geometry`。这是`ST_GeomFromText`的别名

##### 5.8.2. Well-Known Binary (WKB)

`ST_GeogFromWKB` — 从已知的二进制几何表示（WKB）或扩展的已知二进制（EWKB）创建地理实例。  
`ST_GeomFromEWKB` — 从扩展的已知二进制表示法（EWKB）返回指定的 ST\\几何值。  
`ST_GeomFromWKB` — 从已知的二进制几何体表示（WKB）和可选 SRID 创建几何体实例。  
`ST_LineFromWKB` — 使用给定的 SRID 从 WKB 生成一个 LINESTRING  
`ST_LinestringFromWKB` — 使用给定的 SRID 从 WKB 生成几何体。  
`ST_PointFromWKB` — 使用给定的 SRID 从 WKB 生成几何体  
`ST_WKBToSQL` — 从已知的二进制表示法（WKB）返回指定的`ST_Geometry`值。这是不需要 srid 的`ST_GeomFromWKB`的别名

##### 5.8.3. Other Formats

`ST_Box2dFromGeoHash` — 从 GeoHash 字符串返回 BOX2D。  
`ST_GeomFromGeoHash` — 从 GeoHash 字符串返回几何体。  
`ST_GeomFromGML` — 以几何图形的 GML 表示形式作为输入，并输出 PostGIS 几何对象  
`ST_GeomFromGeoJSON` — 将几何体的 geojson 表示形式作为输入，并输出 PostGIS 几何体对象  
`ST_GeomFromKML` — 将几何体的 KML 表示形式作为输入，并输出 PostGIS 几何体对象  
`ST_GeomFromTWKB` — 从 TWKB（"Tiny Well-Known Binary"）几何体表示创建几何体实例。  
`ST_GMLToSQL` — 从 GML 表示返回指定的 ST\\U 几何体值。这是`ST_GeomFromGML`的别名  
`ST_LineFromEncodedPolyline` — 从编码的多段线创建线字符串。  
`ST_PointFromGeoHash` — 从 GeoHash 字符串返回一个点。

#### 5.9 几何输出

这些函数将几何体对象转换为各种文本或二进制格式。

##### 5.9.1. Well-Known Text (WKT)

`ST_AsEWKT` — 返回带有 SRID 元数据的几何体的已知文本（WKT）表示。  
`ST_AsText` — 返回不带 SRID 元数据的几何图形 / 地理图形的已知文本（WKT）表示形式。

##### 5.9.2. Well-Known Binary (WKB)

`ST_AsBinary` — 返回不带 SRID 元数据的几何 / 地理的已知二进制（WKB）表示。  
`ST_AsEWKB` — 返回带有 SRID 元数据的几何体的已知二进制（WKB）表示。  
`ST_AsHEXEWKB` — 使用小端（NDR）或大端（XDR）编码返回 HEXEWKB 格式的几何体（作为文本）。

##### 5.9.3. 其他格式

`ST_AsEncodedPolyline` — 从 LineString 几何图形返回编码的多段线。  
`ST_AsGeobuf` — 返回一组行的 Geobuf 表示形式。  
`ST_AsGeoJSON` — 将几何体作为 GeoJSON 元素返回。  
`ST_AsGML` — 将几何体作为 GML 版本 2 或 3 元素返回。  
`ST_AsKML` — 将几何体作为 KML 元素返回。几个变种。默认版本 = 2，默认 maxdecimaldigits=15  
`ST_AsLatLonText` — 返回给定点的度、分、秒表示。  
`ST_AsMVTGeom` — 将几何图形转换为 Mapbox 矢量图块的坐标空间。  
`ST_AsMVT` — 返回一组行的 Mapbox 向量平铺表示的聚合函数。  
`ST_AsSVG` — 返回几何体的 SVG 路径数据。  
`ST_AsTWKB` — 将几何体返回为 TWKB，也称为 "Tiny Well-Known Binary"  
`ST_AsX3D` — 返回 X3D xml 节点元素格式的几何体： ISO-IEC-19776-1.2-X3DEncodings-XML  
`ST_GeoHash` — 返回几何体的 GeoHash 表示形式。

#### 5.11 空间关系

这些函数决定几何图形之间的空间关系。

##### 5.11.1 拓扑关系

`ST_3DIntersects` —如果两个几何在 3D 中在空间上相交，则返回 true - 仅用于点，线串，多边形，多面曲面（区域）。  
`ST_Contains` —当且仅当 B 的点不位于 A 的外部且 B 的内部的至少一个点位于 A 的内部时，才返回 true。  
`ST_ContainsProperly` —如果 B 与 A 的内部而不是边界（或外部）相交，则返回 true。 A 本身不包含自身，但是包含自身。  
`ST_Covers` —如果 B 中没有点在 A 之外，则返回 true  
`ST_CoveredBy` —如果 Geometry / Geography A 中没有点在 Geometry / Geography B 之外，则返回 true  
`ST_Crosses` —如果两个几何具有一些但不是全部内部相同的点，则返回 true。  
`ST_LineCrossingDirection` —返回一个数字，指示两个 LineString 的交叉行为。  
`ST_Disjoint` —如果两个几何在空间上不相交（它们没有共同点），则返回 true。  
`ST_Equals` —如果两个几何在空间中包含相同的点集，则返回 true。  
`ST_Intersects` —如果两个 “几何” /“地理” 在 2D 空间上相交（至少有一个共同点），则返回 true。  
`ST_OrderingEquals` —如果两个几何表示相同的几何并且具有相同的方向顺序的点，则返回 true。  
`ST_Overlaps` —如果两个几何相交且具有相同的尺寸，但彼此之间不完全包含，则返回 true。  
`ST_Relate` —测试两个几何是否具有与给定 “交集矩阵” 模式匹配的拓扑关系，或计算它们的“交集矩阵”  
`ST_RelateMatch` —测试 DE-9IM 交集矩阵是否与交集矩阵模式匹配  
`ST_Touches` —如果两个几何至少有一个共同点，但它们的内部不相交，则返回 true。  
`ST_Within` —如果几何 A 完全在几何 B 内，则返回 true

##### 5.11.2 距离关系

`ST_3DDWithin` —如果两个 3D 几何形状在给定的 3D 距离内，则返回 true  
`ST_3DDFullyWithin` —如果两个 3D 几何完全在给定的 3D 距离内，则返回 true  
`ST_DFullyWithin` —如果两个几何完全在给定距离内，则返回 true  
`ST_DWithin` —如果两个几何在给定距离内，则返回 true  
`ST_PointInsideCircle` —测试点几何是否在由中心和半径定义的圆内。

#### 5.12 测量功能

`ST_Area` —返回多边形几何的面积。  
`ST_Azimuth` —返回以北为基准的方位角，以弧度为单位，从点 A 的垂直方向到点 B 的角度以弧度为单位。  
`ST_Angle` —返回 3 个点之间或 2 个向量之间的角度（4 个点或 2 条线）。  
`ST_ClosestPoint` —返回 g1 上最接近 g2 的 2D 点。这是最短线的第一点。  
`ST_3DClosestPoint` —返回 g1 上最接近 g2 的 3D 点。这是 3D 最短线的第一点。  
`ST_Distance` —返回两个几何或地理值之间的距离。  
`ST_3DDistance` —以投影单位返回两个几何之间的 3D 笛卡尔最小距离（基于空间参考）。  
`ST_DistanceSphere` —使用球形地球模型返回两个 lon / lat 几何之间的最小距离（以米为单位）。  
`ST_DistanceSpheroid` —使用球形地球模型返回两个 lon / lat 几何之间的最小距离。  
`ST_FrechetDistance` —返回两个几何之间的 Fréchet 距离。  
`ST_HausdorffDistance` —返回两个几何之间的 Hausdorff 距离。  
`ST_Length` —返回线性几何的 2D 长度。  
`ST_Length2D` —返回线性几何的 2D 长度。 ST_Length 的别名  
`ST_3DLength` —返回线性几何的 3D 长度。  
`ST_LengthSpheroid` —返回球体上 lon / lat 几何图形的 2D 或 3D 长度 / 周长。  
`ST_LongestLine` —返回两个几何之间的 2D 最长线。  
`ST_3DLongestLine` —返回两个几何之间的 3D 最长线  
`ST_MaxDistance` —以投影单位返回两个几何之间的最大二维距离。  
`ST_3DMaxDistance` —以投影单位返回两个几何之间的 3D 笛卡尔最大距离（基于空间参考）。  
`ST_MinimumClearance` —返回几何图形的最小间隙，度量几何图形的鲁棒性。  
`ST_MinimumClearanceLine` —返回跨越几何最小间隙的两点 LineString。  
`ST_Perimeter` —返回多边形几何或地理边界的长度。  
`ST_Perimeter2D` —返回多边形几何的 2D 周长。 ST_Perimeter 的别名。  
`ST_3DPerimeter` —返回多边形几何的 3D 周长。  
`ST_Project` —返回从起点投影一个距离和方位角（方位角）的点。  
`ST_ShortestLine` —返回两个几何之间的 2D 最短线  
`ST_3DShortestLine` —返回两个几何之间的 3D 最短线

#### 5.13 叠加功能

这些函数计算两个几何图形叠加产生的结果。这些也被称为点集理论布尔运算。还提供了一些相关的功能。  
`ST_ClipByBox2D` —返回属于矩形的几何图形的一部分。  
`ST_Difference` —返回表示几何 A 的不与几何 B 相交的部分的几何。  
`ST_Intersection` —返回代表几何 A 和 B 共享部分的几何。  
`ST_MemUnion` —聚合函数，以内存有效但较慢的方式合并几何  
`ST_Node` —结点线的集合。  
`ST_Split` —返回通过将一个几何图形拆分为另一个几何图形而创建的几何图形的集合。  
`ST_Subdivide` —计算几何的直线细分。  
`ST_SymDifference` —返回一个几何，表示不相交的几何 A 和 B 的部分。  
`ST_Union` —返回代表输入几何图形的点集并集的几何图形。  
`ST_UnaryUnion` —计算单个几何图形的组件的并集。

#### 5.14 几何处理

这些函数计算几何结构，或改变几何尺寸或形状  
`ST_Buffer —返回一个几何图形，该几何图形覆盖距几何图形给定距离内的所有点。`ST_BuildArea —创建由几何的线条构成的多边形几何。  
`ST_Centroid —返回几何的几何中心。`ST_ConcaveHull —计算可能包含所有输入几何顶点的凹形几何  
`ST_ConvexHull —计算几何的凸包。`ST_DelaunayTriangles —返回几何顶点的 Delaunay 三角剖分。  
`ST_FilterByM` —根据顶点的 M 值移除顶点  
`ST_GeneratePoints` —生成多边形或多多边形中包含的随机点。  
`ST_GeometricMedian` —返回 MultiPoint 的几何中间值。  
`ST_MaximumInscribedCircle` —计算完全包含在几何图形内的最大圆。  
`ST_MinimumBoundingCircle` —返回包含几何的最小圆形多边形。  
`ST_MinimumBoundingRadius` —返回包含几何的最小圆的中心点和半径。  
`ST_OrientedEnvelope` —返回包含几何的最小面积的矩形。  
`ST_OffsetCurve` —返回距输入线给定距离和边距的偏移线。  
`ST_PointOnSurface` —返回保证位于多边形或几何图形上的点。  
`ST_Polygonize` —计算由一组几何图形的线条形成的多边形的集合。  
`ST_ReducePrecision` —返回一个有效的几何图形，所有点均四舍五入到提供的网格公差。  
`ST_SharedPaths` —返回包含两个输入线串 / 多线串共享的路径的集合。  
`ST_Simplify` —使用 Douglas-Peucker 算法返回几何的简化版本。  
`ST_SimplifyPreserveTopology` —使用 Douglas-Peucker 算法返回几何的简化和有效版本。  
`ST_SimplifyVW` —使用 Visvalingam-Whyatt 算法返回几何的简化版本  
`ST_ChaikinSmoothing` —使用 Chaikin 算法返回几何的平滑版本  
`ST_SetEffectiveArea` —使用 Visvalingam-Whyatt 算法设置每个顶点的有效面积。  
`ST_VoronoiLines` —返回几何顶点的 Voronoi 图的边界。  
`ST_VoronoiPolygons` —返回几何顶点的 Voronoi 图的像元。

#### 5.15 仿射变换

这些函数使用仿射变换改变几何图形的位置和形状.  
`ST_Affine` —将 3D 仿射变换应用于几何图形。  
`ST_Rotate` —围绕原点旋转几何。  
`ST_RotateX` —绕 X 轴旋转几何。  
`ST_RotateY` —围绕 Y 轴旋转几何。  
`ST_RotateZ` —绕 Z 轴旋转几何。  
`ST_Scale` —按给定因子缩放几何。  
`ST_Translate` —按给定的偏移量转换几何图形。  
`ST_TransScale` —按给定的偏移量和系数平移和缩放几何。

#### 5.16 聚合功能

这些函数实现了几何集合的聚类算法。  
`ST_ClusterDBSCAN` —窗口函数，使用 DBSCAN 算法为每个输入几何返回聚类 ID。  
`ST_ClusterIntersecting` —聚合函数，用于将输入几何形状聚类为连接的集合。  
`ST_ClusterKMeans` —窗口函数，使用 K-means 算法为每个输入几何返回聚类 ID。  
`ST_ClusterWithin` —聚合函数，用于按分隔距离对输入几何图形进行聚类。

#### 5.17 边框功能

这些函数产生或操作边界框。通过使用自动或显式类型转换，它们还可以提供和接受几何值。  
`Box2D` —返回代表几何 2D 范围的 BOX2D。  
`Box3D` —返回代表几何图形 3D 范围的 BOX3D。  
`ST_EstimatedExtent` —返回空间表的 “估计” 范围。  
`ST_Expand` —返回从另一个边界框或几何图形扩展的边界框。  
`ST_Extent` —一个聚合函数，该函数返回包围几何行的边界框。  
`ST_3DExtent` —一个聚合函数，该函数返回 3D 边界框，该边界框限制几何图形的行。  
`ST_MakeBox2D` —创建由两个 2D 点几何定义的 BOX2D。  
`ST_3DMakeBox` —创建由两个 3D 点几何定义的 BOX3D。  
`ST_XMax` —返回 2D 或 3D 边界框或几何图形的 X 最大值。  
`ST_XMin` —返回 2D 或 3D 边界框或几何的 X 最小值。  
`ST_YMax` —返回 2D 或 3D 边界框或几何的 Y 最大值。  
`ST_YMin` —返回 2D 或 3D 边界框或几何的 Y 最小值。  
`ST_ZMax` —返回 2D 或 3D 边界框或几何的 Z 最大值。  
`ST_ZMin` —返回 2D 或 3D 边界框或几何的 Z 最小值。

#### 5.18 线性参考

`ST_LineInterpolatePoint` —返回沿直线插补的点。第二个参数是一个介于 0 和 1 之间的 float8，代表必须定位该点的线串总长度的一部分。  
`ST_3DLineInterpolatePoint` —返回沿 3D 线插入的点。第二个参数是一个介于 0 和 1 之间的 float8，代表必须定位该点的线串总长度的一部分。  
`ST_LineInterpolatePoints` —返回沿一条线插补的一个或多个点。  
`ST_LineLocatePoint` —返回 0 到 1 之间的浮点数，表示 LineString 上与给定 Point 点最近的点的位置，以总 2d 线长的一部分表示。  
`ST_LineSubstring` —返回一个线串，该线串是输入的子串，从总 2d 长度的给定分数开始和结束。第二个和第三个参数是介于 0 和 1 之间的 float8 值。  
`ST_LocateAlong` —返回具有与指定度量匹配的元素的派生几何集合值。不支持多边形元素。  
`ST_LocateBetween` —返回派生的几何集合值，其元素与指定的范围（包括范围在内）相匹配。  
`ST_LocateBetweenElevations` —返回派生的几何（集合）值，其元素与指定的高程范围相交。  
`ST_InterpolatePoint` —返回在与提供的点接近的点处的几何的度量尺寸的值。  
`ST_AddMeasure` —返回带有在起点和终点之间线性插值的测量元素的派生几何。

#### 5.19 轨迹函数

这些函数支持使用轨迹。轨迹是在每个坐标上都有一个度量 (M 值) 的线性几何图形。测量值必须沿直线增加。时空数据可以通过使用相对时间 (如 epoch) 作为度量值来建模。  
`ST_IsValidTrajectory` —如果几何是有效轨迹，则返回 true。  
`ST_ClosestPointOfApproach` —返回沿两个轨迹插补的点最接近的度量。  
`ST_DistanceCPA` —返回两个轨迹的最接近点之间的距离。  
`ST_CPAWithin` —如果两个轨迹的最接近点在指定距离内，则返回 true。

#### 5.20 SFCGAL 函数

SFCGAL 是一个围绕 CGAL 的 c++ 包装库，提供了高级的 2D 和 3D 空间函数。为了鲁棒性，几何坐标具有精确的有理数表示。  
这个库的安装说明可以在 SFCGAL 主页 ([http://www.sfcgal.org](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.sfcgal.org)) 上找到。要启用这些函数，请使用 create extension postgis_sfcgal。  
`postgis_sfcgal_version` —返回正在使用的 SFCGAL 的版本  
`ST_Extrude` —将曲面拉伸到相关体积  
`ST_StraightSkeleton` —从几何图形计算直线骨架  
`ST_CloseMedialAxis` —计算平面几何体的近似中间轴。  
`ST_IsPlanar` —检查曲面是否为平面  
`ST_Orientation` —确定表面方向  
`ST_ForceLHR` —强制 LHR 方向  
`ST_MinkowskiSum` —执行 Minkowski 和  
`ST_ConstrainedDelaunayTriangles` —返回围绕给定输入几何形状的约束 Delaunay 三角剖分。  
`ST_3DIntersection` —执行 3D 相交  
`ST_3DDifference` —执行 3D 差异  
`ST_3DUnion` —执行 3D 合并  
`ST_3DArea` —计算 3D 表面几何形状的面积。对于实体将返回 0。  
`ST_Tesselate` —对多边形或多面体表面执行表面镶嵌处理，并以 TIN 或 TINS 集合的形式返回  
`ST_Volume` —计算 3D 实体的体积。如果应用于表面（甚至闭合）几何，则将返回 0。  
`ST_MakeSolid` —将几何体转换为实体。不执行检查。为了获得有效的实体，输入几何必须是封闭的多面曲面或封闭的 TIN。  
`ST_IsSolid` —测试几何图形是否为实体。不执行有效性检查。

##### 格式转换

`ST_Force2D(geom)` postgres 里不能预览三维数据  
`ST_AsText(geom)` wkb 转 wkt  
`ST_AsGeoJSON(geom)` wkb 转 geojson

##### 几何处理

`ST_LineMerge(ST_Union(geoms))` 线段连接合并，效率较低，但适合大部分场景  
`ST_MakeLine(geoms)` 线段连接合并效率高，但严格要求方向一致  
`ST_SimplifyPreserveTopology(geometry ,< 阈值>)` 几何抽稀

##### 几何查询

`ST_Numgeometries(geom)` 计算 mulit 类型包含几何的数量  
`ST_GeometryN(geom,index)` 获取 mulit 类型中的第几个单体几何  
`ST_Lenght(geom)` 计算长度

###### 拓扑关系

`ST_Intersects` 相交关系，两个图形之间存在公共部分，比如公共点，公共线，公共面  
`ST_Disjoint` 相离关系，两个图形无丝毫公共部分，与 ST_Intersects 完全相反  
`ST_Contains` 包含关系，图形 A 包含图形 B：ST_Contains(A,B)，如点在面内，线在面内。  
`ST_Within` 被包含关系，图形 A 被 B 包含: ST_Within(A,B)，与 ST_Contains 完全相反。  
`ST_Covers` 覆盖关系，图形 A 完全覆盖住了图形 B：ST_Covers (A,B)，部分关系与`ST_Contains` 重叠，但不是完全一样。  
`ST_Crosses` 穿越关系，图形 A 与图形 B 有一部分公共内点，但不是全部。  
`ST_Equals` 相等关系，两个图形完全相等。  
`ST_Overlaps` 压盖关系  
`ST_Touches` 相连关系，两个图形只有边界存在公共连接关系。 
 [https://www.jianshu.com/p/4022de46d8df](https://www.jianshu.com/p/4022de46d8df)
