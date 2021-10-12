# 微服务SpringCloud项目（四）：整合MinIo实现文件上传 - 掘金
**小知识，大挑战！本文正在参与「[程序员必备小知识](https://juejin.cn/post/7008476801634680869 "https&#x3A;//juejin.cn/post/7008476801634680869")」创作活动**

**本文已参与 [「掘力星计划」](https://juejin.cn/post/7012210233804079141 "https&#x3A;//juejin.cn/post/7012210233804079141") ，赢取创作大礼包，挑战创作激励金。** 

## 📖前言

    心态好了，就没那么累了。心情好了，所见皆是明媚风景。
    复制代码

> **`“一时解决不了的问题，那就利用这个契机，看清自己的局限性，对自己进行一场拨乱反正。” 正如老话所说，一念放下，万般自在。如果你正被烦心事扰乱心神，不妨学会断舍离。断掉胡思乱想，社区垃圾情绪，离开负面能量。心态好了，就没那么累了。心情好了，所见皆是明媚风景。`**

### 🚓进入正题

> **`不多说了：用的 mybatis-plus 具体的实现类什么的就不写了，毕竟复制粘贴谁都会希望你不是复制粘贴一把梭呵呵，进入正题吧`**
>
> **`结构如下`**

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e28e872b2ae14877bbdc1a5e03101563~tplv-k3u1fbpfcp-watermark.awebp?)

#### 1. 创建子模块单独存放配置读取和工具类 (为了演示方便我就放在一个项目里了别介意)

> `引入如下依赖：`

```xml

        <dependency>
            <groupId>io.minio</groupId>
            <artifactId>minio</artifactId>
            <version>${minio.version}</version>
        </dependency>
```

> **`yml 增加配置如下，其他配置自行搞定`**

```yaml

minio:
  server:
    url: mini访问地址
    accessKey: 密钥
    secretKey: 密钥
    originFileBucKetValue: dream-cloud-auth 
    allowOriginFileBucKetValue: dream-cloud-allow 
```

#### 2. `MinIoProperties.java` -- 用于 `Minio` 配置信息获取

```java
package com.cyj.dream.file.config;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.context.annotation.Configuration;


@Data
@RefreshScope
@Configuration
@ConfigurationProperties("minio.server")
public class MinIoProperties {

    
    private String url;

    
    private String accessKey;

    
    private String secretKey;

    
    private String chunkBucKetValue;

    
    private String originFileBucKetValue;

    
    private String allowOriginFileBucKetValue;

}
```

#### 3. `MinIoUtils.java` -- 用于操作 `MinIo` 工具类

```java
package com.cyj.dream.minio.util;

import cn.hutool.core.date.DateUtil;
import cn.hutool.core.lang.UUID;
import cn.hutool.core.util.StrUtil;
import com.cyj.dream.minio.config.MinIoProperties;
import com.google.common.io.ByteStreams;
import io.minio.*;
import io.minio.errors.MinioException;
import io.minio.http.Method;
import io.minio.messages.Item;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;
import javax.annotation.Resource;
import java.io.*;
import java.util.*;
import java.util.stream.Collectors;


@Slf4j
@Component
public class MinIoUtils {

    @Autowired
    private MinIoProperties minIoProperties;

    private static String url;

    private static String accessKey;

    private static String secretKey;

    public static String chunkBucKet;

    public static String originFileBucKet;

    public static String allowOriginFileBucKet;

    private static MinioClient minioClient;

    
    public final static boolean SORT = true;

    
    public final static boolean NOT_SORT = false;

    
    private final static Integer DEFAULT_EXPIRY = 60;

    
    @PostConstruct
    public void init() {
        url = minIoProperties.getUrl();
        accessKey = minIoProperties.getAccessKey();
        secretKey = minIoProperties.getSecretKey();
        chunkBucKet = minIoProperties.getChunkBucKetValue();
        originFileBucKet = minIoProperties.getOriginFileBucKetValue();
        allowOriginFileBucKet = minIoProperties.getAllowOriginFileBucKetValue();
    }

    public static void afterPropertiesSet() throws Exception {
        log.info("url ====>{}", url);
        minioClient = MinioClient.builder().endpoint(url).credentials(accessKey, secretKey).build();
        
        if (!StrUtil.isEmpty(chunkBucKet) && !isBucketExist(chunkBucKet)) {
            createBucket(chunkBucKet);
        }
        if (!StrUtil.isEmpty(originFileBucKet) && !isBucketExist(originFileBucKet)) {
            createBucket(originFileBucKet);
        }
        if (!StrUtil.isEmpty(allowOriginFileBucKet) && !isBucketExist(allowOriginFileBucKet)) {
            createBucket(allowOriginFileBucKet);
        }
    }

    
    public static MinioClient getInstance() throws Exception {

        if (minioClient == null) {
            synchronized (MinIoUtils.class) {
                if (minioClient == null) {
                    afterPropertiesSet();
                }
            }
        }
        return minioClient;
    }

    
    public static boolean isBucketExist(String bucketName) throws Exception {
        return getInstance().bucketExists(BucketExistsArgs.builder().bucket(bucketName).build());
    }

    
    public static void putObject(String bucketName, String filePath, File file) throws Exception {
        getInstance().putObject(
                PutObjectArgs.builder()
                        .bucket(bucketName)
                        .object(removeSlash(filePath))
                        .stream(new FileInputStream(file), file.length(), -1)
                        .build()
        );
    }

    
    public static boolean copyObject(String bucketName, String sourceFilePath, String targetFilePath) throws Exception {

        try {
            getInstance().copyObject(CopyObjectArgs.builder()
                    .bucket(bucketName)
                    .object(targetFilePath)
                    .source(CopySource
                            .builder()
                            .bucket(bucketName)
                            .object(sourceFilePath)
                            .build()
                    )
                    .build());
            return true;
        } catch (MinioException e) {
            System.out.println("Error occurred: " + e);
            return false;
        }

    }

    
    public static void putObject(String bucketName, String filePath, InputStream file) throws Exception {
        getInstance().putObject(
                PutObjectArgs.builder()
                        .bucket(bucketName)
                        .object(removeSlash(filePath))
                        .stream(file, file.available(), -1)
                        .build()
        );
    }

    
    private static String removeSlash(String str) {
        if (str.substring(0, 1).equals("/")) {
            return str.substring(1);
        }
        return str;
    }

    
    public static byte[] getObject(String bucketName, String filePath) throws Exception {

        InputStream inputStream = null;
        try {
            inputStream = getInstance().getObject(
                    GetObjectArgs.builder()
                            .bucket(bucketName)
                            .object(removeSlash(filePath))
                            .build()
            );
        } catch (MinioException e) {
            System.out.println("Error occurred: " + e);
            return null;
        }
        ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
        {
            ByteStreams.copy(inputStream, outputStream);
            byte[] buffer = outputStream.toByteArray();
            return buffer;
        }
    }

    
    public static StatObjectResponse statObject(String bucketName, String filePath) throws Exception {

        try {
            return getInstance().statObject(
                    StatObjectArgs.builder()
                            .bucket(bucketName)
                            .object(removeSlash(filePath))
                            .build()
            );
        } catch (MinioException e) {
            System.out.println("Error occurred: " + e);
            return null;
        }
    }

    
    public static void removeObject(String bucketName, String filePath) throws Exception {
        getInstance().removeObject(
                RemoveObjectArgs.builder()
                        .bucket(bucketName)
                        .object(removeSlash(filePath))
                        .build()
        );
    }

    
    public static boolean createBucket(String bucketName) throws Exception {
        getInstance().makeBucket(MakeBucketArgs.builder().bucket(bucketName).build());
        return true;
    }

    
    public static String getOriginalObjectUrl(String objectName, Integer expiry) throws Exception {
        return getObjectUrl(originFileBucKet, objectName, expiry);
    }

    
    public static String getObjectUrl(String bucketName, String objectName, Integer expiry) throws Exception {
        expiry = expiryHandle(expiry);
        return getInstance().getPresignedObjectUrl(
                GetPresignedObjectUrlArgs.builder()
                        .method(Method.GET)
                        .bucket(bucketName)
                        .object(removeSlash(objectName))
                        .expiry(expiry)
                        .build()
        );
    }

    
    public static String createUploadUrl(String bucketName, String objectName, Integer expiry) throws Exception {
        expiry = expiryHandle(expiry);
        return getInstance().getPresignedObjectUrl(
                GetPresignedObjectUrlArgs.builder()
                        .method(Method.PUT)
                        .bucket(bucketName)
                        .object(removeSlash(objectName))
                        .expiry(expiry)
                        .build()
        );
    }

    
    public static String createUploadUrl(String bucketName, String objectName) throws Exception {
        return createUploadUrl(bucketName, objectName, DEFAULT_EXPIRY);
    }

    
    public static List<String> createUploadChunkUrlList(String bucketName, String objectMD5, Integer chunkCount) throws Exception {
        if (null == bucketName) {
            bucketName = chunkBucKet;
        }
        if (null == objectMD5) {
            return null;
        }
        objectMD5 += "/";
        if (null == chunkCount || 0 == chunkCount) {
            return null;
        }
        List<String> urlList = new ArrayList<>(chunkCount);
        for (int i = 1; i <= chunkCount; i++) {
            String objectName = objectMD5 + i + ".chunk";
            urlList.add(createUploadUrl(bucketName, objectName, DEFAULT_EXPIRY));
        }
        return urlList;
    }

    
    public static String createUploadChunkUrl(String bucketName, String objectMD5, Integer partNumber) throws Exception {
        if (null == bucketName) {
            bucketName = chunkBucKet;
        }
        if (null == objectMD5) {
            return null;
        }
        objectMD5 += "/" + partNumber + ".chunk";
        return createUploadUrl(bucketName, objectMD5, DEFAULT_EXPIRY);
    }

    
    public static List<String> listObjectNames(String bucketName, String prefix, Boolean sort) throws Exception {
        ListObjectsArgs listObjectsArgs;
        if (null == prefix) {
            listObjectsArgs = ListObjectsArgs.builder()
                    .bucket(bucketName)
                    .recursive(true)
                    .build();
        } else {
            listObjectsArgs = ListObjectsArgs.builder()
                    .bucket(bucketName)
                    .prefix(prefix)
                    .recursive(true)
                    .build();
        }
        Iterable<Result<Item>> chunks = getInstance().listObjects(listObjectsArgs);
        List<String> chunkPaths = new ArrayList<>();
        for (Result<Item> item : chunks) {
            chunkPaths.add(item.get().objectName());
        }
        if (sort) {
            return chunkPaths.stream().distinct().collect(Collectors.toList());
        }
        return chunkPaths;
    }

    
    public static List<String> listObjectNames(String bucketName, String prefix) throws Exception {
        return listObjectNames(bucketName, prefix, NOT_SORT);
    }

    
    public static List<String> listChunkObjectNames(String bucketName, String ObjectMd5) throws Exception {
        if (null == bucketName) {
            bucketName = chunkBucKet;
        }
        if (null == ObjectMd5) {
            return null;
        }
        return listObjectNames(bucketName, ObjectMd5, SORT);
    }

    
    public static Map<Integer, String> mapChunkObjectNames(String bucketName, String ObjectMd5) throws Exception {
        if (null == bucketName) {
            bucketName = chunkBucKet;
        }
        if (null == ObjectMd5) {
            return null;
        }
        List<String> chunkPaths = listObjectNames(bucketName, ObjectMd5);
        if (null == chunkPaths || chunkPaths.size() == 0) {
            return null;
        }
        Map<Integer, String> chunkMap = new HashMap<>(chunkPaths.size());
        for (String chunkName : chunkPaths) {
            Integer partNumber = Integer.parseInt(chunkName.substring(chunkName.indexOf("/") + 1, chunkName.lastIndexOf(".")));
            chunkMap.put(partNumber, chunkName);
        }
        return chunkMap;
    }

    
    public static boolean composeObject(String chunkBucKetName, String composeBucketName, List<String> chunkNames, String objectName) throws Exception {
        if (null == chunkBucKetName) {
            chunkBucKetName = chunkBucKet;
        }
        List<ComposeSource> sourceObjectList = new ArrayList<>(chunkNames.size());
        for (String chunk : chunkNames) {
            sourceObjectList.add(
                    ComposeSource.builder()
                            .bucket(chunkBucKetName)
                            .object(chunk)
                            .build()
            );
        }
        getInstance().composeObject(
                ComposeObjectArgs.builder()
                        .bucket(composeBucketName)
                        .object(removeSlash(objectName))
                        .sources(sourceObjectList)
                        .build()
        );
        return true;
    }

    
    public static boolean composeObject(List<String> chunkNames, String objectName) throws Exception {
        return composeObject(chunkBucKet, originFileBucKet, chunkNames, objectName);
    }

    
    public static String upload(String fileName, InputStream inputStream, String fileBucketName) throws Exception {
        String suffix = fileName.substring(fileName.lastIndexOf(".") + 1);
        return upload(inputStream, suffix, fileBucketName);
    }

    
    public static String upload(InputStream inputStream, String suffix, String fileBucketName) throws Exception {
        String uuid = UUID.randomUUID().toString();
        String savePath = getSavePath(uuid + "." + suffix);
        putObject(fileBucketName, savePath, inputStream);
        return savePath;
    }

    private static String getSavePath(String fileName) {

        String dayStr = DateUtil.now();
        String days = dayStr.substring(0, dayStr.lastIndexOf(" "));
        String[] dayArr = days.split("-");

        String path = dayArr[0] + "/" + dayArr[1] + "/" + dayArr[2] + "/" + fileName;

        return path;
    }

    
    private static int expiryHandle(Integer expiry) {
        expiry = expiry * 60;
        if (expiry > 604800) {
            return 604800;
        }
        return expiry;
    }

}
```

#### 4. 文件中心常量

```java
package com.cyj.dream.file.contacts;


public class FileConstant {

    
    public static final String SCAN_BASE_PACKAGES_URL = "com.cyj.dream.file";

    
    public static final String MAPPER_URL = "com.cyj.dream.file.mapper";

    
    public final static String UPLOAD_TYPE_AUTH="auth";

    
    public final static String UPLOAD_TYPE_ALLOW="allow";

}
```

#### 5. 创建操作数据库实体

`FileUploadPieceRecord` -- 文件分片断点续传记录表

```java
package com.cyj.dream.file.model;

import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.annotation.TableId;
import com.baomidou.mybatisplus.annotation.TableName;
import com.cyj.dream.core.constant.TreeEntity;
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.ToString;

import javax.persistence.*;


@Data
@Entity
@ToString
@Table(name = "file_upload_piece_record")
@TableName("file_upload_piece_record")
@ApiModel(value = "FileUploadPieceRecord", description = "file_upload_piece_record 分片上传记录表")
public class FileUploadPieceRecord extends TreeEntity {

    @Id
    @TableId(value = "piece_id", type = IdType.AUTO)
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(nullable = false, columnDefinition = "bigint(20) unsigned COMMENT '分片上传记录表主键--自增'")
    @ApiModelProperty(value = "文件路径id", example = "1")
    private Long pieceId;

    
    @ApiModelProperty(value = "分片存储桶名称")
    @Column(columnDefinition = "varchar(128) COMMENT '文件路径'")
    private String chunkBucketName;

    
    @ApiModelProperty(value = "源文件存储桶名称")
    @Column(columnDefinition = "varchar(100) COMMENT '源文件存储桶名称'")
    private String fileBucketName;

    
    @ApiModelProperty(value = "分片数量")
    @Column(columnDefinition = "bigint(20) COMMENT '分片数量'")
    private Long chunkCount;

    
    @ApiModelProperty(value = "上传文件的md5")
    @Column(columnDefinition = "varchar(255) COMMENT '上传文件的md5'")
    private String fileMd5;

    
    @ApiModelProperty(value = "上传文件/合并文件的格式")
    @Column(columnDefinition = "varchar(64) COMMENT '上传文件/合并文件的格式'")
    private String fileSuffix;

    
    @ApiModelProperty(value = "文件名称")
    @Column(columnDefinition = "varchar(128) COMMENT '文件名称'")
    private String fileName;

    
    @ApiModelProperty(value = "文件大小（b）")
    @Column(columnDefinition = "varchar(255) COMMENT '文件大小（b）'")
    private String fileSize;

    
    @ApiModelProperty(value = "文件地址")
    @Column(columnDefinition = "varchar(500) COMMENT '文件地址'")
    private String filePath;

    
    @ApiModelProperty(value = "上传状态")
    @Column(columnDefinition = "int(2) COMMENT '上传状态 0.上传完成 1.已上传部分 int(2)'")
    private Integer uploadStatus;

    
    @Transient
    private Integer partNumber;

    
    @Transient
    private String uploadUrl;

}
```

`FileUploadRecord` -- 文件上传记录

```java
package com.cyj.dream.file.model;

import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.annotation.TableId;
import com.baomidou.mybatisplus.annotation.TableName;
import com.cyj.dream.core.constant.BaseEntity;
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.ToString;

import javax.persistence.*;


@Data
@Entity
@ToString
@Table(name = "file_upload_record")
@TableName("file_upload_record")
@ApiModel(value = "FileUploadRecord", description = "file_upload_record 文件上传记录")
public class FileUploadRecord extends BaseEntity {

    @ApiModelProperty(value = "文件路径id", example = "1")
    @Id
    @TableId(value = "file_id", type = IdType.AUTO)
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(nullable = false, columnDefinition = "bigint(20) unsigned COMMENT '文件上传记录主键--自增'")
    private Long fileId;

    @ApiModelProperty(value = "文件路径 varchar(500)")
    @Column(columnDefinition = "varchar(500) COMMENT '文件路径'")
    private String filePath;

    @ApiModelProperty(value = "文件名称 varchar(255)")
    @Column(columnDefinition = "varchar(255) COMMENT '文件名称'")
    private String fileName;

    @ApiModelProperty(value = "上传文件/合并文件的格式 varchar(64)")
    @Column(columnDefinition = "varchar(64) COMMENT '上传文件/合并文件的格式'")
    private String fileSuffix;

    @ApiModelProperty(value = "源文件存储桶名称 varchar(128)")
    @Column(columnDefinition = "varchar(128) COMMENT '源文件存储桶名称'")
    private String fileBucketName;

    @ApiModelProperty(value = "类型（allow 是放行,auth 是需要鉴权的 ） varchar(128)")
    @Column(columnDefinition = "varchar(128) COMMENT '类型（allow 是放行,auth 是需要鉴权的 ）'")
    private String fileType;

}
```

#### 6. `FileManagementServiceImpl` -- 列一下文件管理实现类

```java
package com.cyj.dream.file.service.impl;

import cn.hutool.core.util.ObjectUtil;
import cn.hutool.core.util.StrUtil;
import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
import com.cyj.dream.core.constant.pagemodel.ResponseUtil;
import com.cyj.dream.core.util.date.DateUtils;
import com.cyj.dream.file.contacts.FileConstant;
import com.cyj.dream.file.mapper.FileUploadPieceRecordMapper;
import com.cyj.dream.file.mapper.FileUploadRecordMapper;
import com.cyj.dream.file.model.FileUploadPieceRecord;
import com.cyj.dream.file.model.FileUploadRecord;
import com.cyj.dream.file.service.FileManagementService;
import com.cyj.dream.file.util.MinIoUtils;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

import javax.annotation.Resource;
import java.time.LocalDateTime;
import java.util.*;


@Slf4j
@Service
public class FileManagementServiceImpl implements FileManagementService {

    
    private final Integer UPLOAD_SUCCESS = 1;

    
    private final Integer UPLOAD_PART = 0;

    @Resource
    private FileUploadRecordMapper fileUploadRecordMapper;

    @Resource
    private FileUploadPieceRecordMapper fileUploadPieceRecordMapper;

    @Override
    public Object initChunkUpload(String md5, String chunkCount, String fileSize) {
        try {
            FileUploadPieceRecord uploadDto = new FileUploadPieceRecord();
            uploadDto.setFileSize(fileSize);
            uploadDto.setFileMd5(md5);
            uploadDto.setChunkCount(Long.valueOf(chunkCount));
            QueryWrapper<FileUploadPieceRecord> queryWrapper = new QueryWrapper<>();
            queryWrapper.lambda().eq(FileUploadPieceRecord::getFileMd5, md5);
            FileUploadPieceRecord mysqlFileData = fileUploadPieceRecordMapper.selectOne(queryWrapper);
            if (ObjectUtil.isNotEmpty(mysqlFileData)) {
                
                if (UPLOAD_SUCCESS.equals(mysqlFileData.getUploadStatus())) {
                    return mysqlFileData;
                }
                
                
                Map<Integer, String> okChunkMap = MinIoUtils.mapChunkObjectNames(MinIoUtils.chunkBucKet, uploadDto.getFileMd5());
                List<FileUploadPieceRecord> chunkUploadUrls = new ArrayList<>();
                if (ObjectUtil.isNotEmpty(okChunkMap) && okChunkMap.size() > 0) {
                    for (int i = 1; i <= uploadDto.getChunkCount(); i++) {
                        
                        if (!okChunkMap.containsKey(i)) {
                            
                            FileUploadPieceRecord url = new FileUploadPieceRecord();
                            url.setPartNumber(i);
                            url.setUploadUrl(MinIoUtils.createUploadChunkUrl(MinIoUtils.chunkBucKet, uploadDto.getFileMd5(), i));
                            chunkUploadUrls.add(url);
                        }
                    }
                    if (chunkUploadUrls.size() == 0) {
                        return "所有分片已经上传完成，仅需要合并文件";
                    }
                    return chunkUploadUrls;
                }
            }
            
            List<String> uploadUrls = MinIoUtils.createUploadChunkUrlList(MinIoUtils.chunkBucKet, uploadDto.getFileMd5(),
                    Integer.valueOf(uploadDto.getChunkCount().toString()));
            List<FileUploadPieceRecord> chunkUploadUrls = new ArrayList<>();
            for (int i = 1; i <= uploadUrls.size(); i++) {
                FileUploadPieceRecord url = new FileUploadPieceRecord();
                url.setPartNumber(i);
                url.setUploadUrl(uploadUrls.get(i - 1));
                chunkUploadUrls.add(url);
            }
            
            uploadDto.setUploadStatus(UPLOAD_PART);
            if (ObjectUtil.isEmpty(mysqlFileData)) {
                uploadDto.setChunkBucketName(MinIoUtils.chunkBucKet);
                uploadDto.setFileBucketName(MinIoUtils.originFileBucKet);
                uploadDto.setCreateTime(DateUtils.toLocalDateTime(new Date()));
                uploadDto.setUpdateTime(DateUtils.toLocalDateTime(new Date()));
                fileUploadPieceRecordMapper.insert(uploadDto);
            }
            return chunkUploadUrls;
        } catch (Exception ex) {
            log.error("发生异常，异常信息为：{}", ex);
            return ResponseUtil.error("初始化文件失败");
        }
    }

    @Override
    public Object composeFile(String md5, String fileName) {
        try {
            
            List<String> chunks = MinIoUtils.listObjectNames(MinIoUtils.chunkBucKet, md5);
            
            List<String> newChunks = new ArrayList<>(chunks.size());
            for (int i = 1; i <= chunks.size(); i++) {
                String newChunkName = md5 + "/" + i + ".chunk";
                newChunks.add(newChunkName);
            }
            
            String suffix = fileName.substring(fileName.lastIndexOf("."));
            String filePath = md5 + "/" + fileName;
            
            if (!MinIoUtils.composeObject(newChunks, filePath)) {
                return ResponseUtil.error("合并文件失败");
            }
            String url = null;
            try {
                
                url = MinIoUtils.getOriginalObjectUrl(filePath, 60);
            } catch (Exception e) {
                log.error("发生异常，异常信息为：{}", e);
                return ResponseUtil.error("获取文件下载连接失败");
            }
            
            QueryWrapper<FileUploadPieceRecord> queryWrapper = new QueryWrapper<>();
            queryWrapper.lambda().eq(FileUploadPieceRecord::getFileMd5, md5);
            FileUploadPieceRecord dbData = fileUploadPieceRecordMapper.selectOne(queryWrapper);
            dbData.setUploadStatus(UPLOAD_SUCCESS);
            dbData.setFileName(fileName);
            dbData.setFilePath(url);
            dbData.setFileSuffix(suffix);

            dbData.setCreateTime(DateUtils.toLocalDateTime(new Date()));
            dbData.setUpdateTime(DateUtils.toLocalDateTime(new Date()));
            
            fileUploadPieceRecordMapper.updateById(dbData);
            return dbData;
        } catch (Exception ex) {
            log.error("发生异常，异常信息为：{}", ex);
            return ResponseUtil.error("合并失败");
        }
    }

    @Override
    public Object authUpload(MultipartFile file, String fileName) {
        return this.upload(file, fileName, FileConstant.UPLOAD_TYPE_AUTH, MinIoUtils.originFileBucKet);
    }

    @Override
    public Object allowUpload(MultipartFile file, String fileName) {
        return this.upload(file, fileName, FileConstant.UPLOAD_TYPE_ALLOW, MinIoUtils.allowOriginFileBucKet);
    }

    @Override
    public Object getPicBase64(FileUploadRecord fileUploadRecord) {
        Map<String, String> result = new HashMap<>(16);
        try {
            byte[] file = new byte[0];
            if (!StrUtil.isEmpty(fileUploadRecord.getFilePath())) {
                file = MinIoUtils.getObject(fileUploadRecord.getFileBucketName(), fileUploadRecord.getFilePath());
            }
            String encoded = Base64.getEncoder().encodeToString(file);
            String type = "";
            if (fileUploadRecord.getFilePath().toLowerCase().contains(".jpg")) {
                type = "data:image/jpeg;base64,";
            } else if (fileUploadRecord.getFilePath().toLowerCase().contains(".png")) {
                type = "data:image/png;base64,";
            } else if (fileUploadRecord.getFilePath().toLowerCase().contains(".gif")) {
                type = "data:image/gif;base64,";
            }
            result.put("base64", type + encoded);
        } catch (Exception ex) {
            log.error("发生异常，异常信息为：{}", ex);
            return ResponseUtil.error("获取异常");
        }
        return result;
    }

    
    private Object upload(MultipartFile file, String fileName, String type, String fileBucketName) {
        try {
            String suffix = fileName.substring(fileName.lastIndexOf(".") + 1);
            String saveUrl = MinIoUtils.upload(file.getInputStream(), suffix, fileBucketName);
            FileUploadRecord fileUploadRecord = new FileUploadRecord();
            fileUploadRecord.setFileBucketName(fileBucketName);
            fileUploadRecord.setFileName(fileName);
            fileUploadRecord.setFileSuffix(suffix);
            fileUploadRecord.setFilePath(saveUrl);
            fileUploadRecord.setFileType(type);

            fileUploadRecord.setCreateTime(DateUtils.toLocalDateTime(new Date()));
            fileUploadRecord.setUpdateTime(DateUtils.toLocalDateTime(new Date()));
            fileUploadRecordMapper.insert(fileUploadRecord);
            return fileUploadRecord;
        } catch (Exception ex) {
            log.error("发生异常，异常信息为：{}", ex);
            return ResponseUtil.error("上传失败");
        }
    }

}
```

#### 7. `FilesManagementController` 控制器

```java
package com.cyj.dream.file.controller.minio;

import cn.hutool.core.util.StrUtil;
import com.alibaba.fastjson.JSONObject;
import com.cyj.dream.core.aspect.annotation.ResponseResult;
import com.cyj.dream.core.constant.pagemodel.ResponseUtil;
import com.cyj.dream.file.model.FileUploadRecord;
import com.cyj.dream.file.service.FileManagementService;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiImplicitParam;
import io.swagger.annotations.ApiImplicitParams;
import io.swagger.annotations.ApiOperation;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;


@Slf4j
@ResponseResult
@RestController
@RequestMapping(value = "/file/manager", name = "文件管理")
@Api(value = "/file/manager", tags = "文件管理")
public class FilesManagementController {

    @Autowired
    private FileManagementService fileManagementService;

    
    @ApiOperation("初始化大文件上传")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "md5", value = "文件信息的md5值", dataType = "String", required = true),
            @ApiImplicitParam(name = "chunkCount", value = "分片计数", dataType = "String", required = true),
            @ApiImplicitParam(name = "fileSize", value = "文件大小", dataType = "String", required = true)
    })
    @RequestMapping(value = "/initChunkUpload", name = "初始化大文件上传", method = RequestMethod.POST)
    public Object initChunkUpload(String md5, String chunkCount, String fileSize) {
        log.info("进入 初始化大文件上传 控制器方法，md5：{}，chunkCount：{}，fileSize：{}", md5, chunkCount, fileSize);
        if(StrUtil.isEmpty(md5) || StrUtil.isEmpty(chunkCount) || StrUtil.isEmpty(fileSize)){
            log.error("参数缺失请检查参数后重新提交~");
            return ResponseUtil.error("参数缺失请检查参数后重新提交");
        }
        return fileManagementService.initChunkUpload(md5, chunkCount, fileSize);
    }

    
    @ApiOperation("合并文件并返回文件信息")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "md5", value = "文件信息的md5值", dataType = "String", required = true),
            @ApiImplicitParam(name = "fileName", value = "文件名称", dataType = "String", required = true)
    })
    @RequestMapping(value = "/composeFile", name = "合并文件并返回文件信息", method = RequestMethod.POST)
    public Object composeFile(String md5, String fileName) {
        log.info("进入 合并文件并返回文件信息 控制器方法，md5：{}，fileName：{}", md5, fileName);
        if(StrUtil.isEmpty(md5) || StrUtil.isEmpty(fileName)){
            log.error("参数缺失请检查参数后重新提交~");
            return ResponseUtil.error("参数缺失请检查参数后重新提交");
        }
        return fileManagementService.composeFile(md5, fileName);
    }

    
    @ApiOperation("限制上传直接上传")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "file", value = "文件信息", dataType = "MultipartFile", required = true),
            @ApiImplicitParam(name = "fileName", value = "文件名称", dataType = "String", required = true)
    })
    @RequestMapping(value = "/authUpload", name = "限制上传直接上传", method = RequestMethod.POST)
    public Object authUpload(MultipartFile file, String fileName) {
        log.info("进入 限制上传直接上传 控制器方法，fileSize：{}，fileName：{}", file.getSize(), fileName);
        if(file == null || file.getSize() == 0 || StrUtil.isEmpty(fileName)){
            log.error("参数缺失请检查参数后重新提交~");
            return ResponseUtil.error("参数缺失请检查参数后重新提交");
        }
        return fileManagementService.authUpload(file, fileName);
    }

    
    @ApiOperation("放行上传直接上传")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "file", value = "文件信息", dataType = "MultipartFile", required = true),
            @ApiImplicitParam(name = "fileName", value = "文件名称", dataType = "String", required = true)
    })
    @RequestMapping(value = "/allowUpload", name = "放行上传直接上传", method = RequestMethod.POST)
    public Object allowUpload(MultipartFile file, String fileName) {
        log.info("进入 放行上传直接上传--如何测试-- 控制器方法，fileSize：{}，fileName：{}", file.getSize(), fileName);
        if(file == null || file.getSize() == 0 || StrUtil.isEmpty(fileName)){
            log.error("参数缺失请检查参数后重新提交~");
            return ResponseUtil.error("参数缺失请检查参数后重新提交");
        }
        return fileManagementService.allowUpload(file, fileName);
    }

    
    @ApiOperation("获取到图片文件的base64")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "fileUploadRecord", value = "文件上传记录对象", dataType = "FileUploadRecord", required = true)
    })
    @RequestMapping(value = "/getPicBase64", name = "获取到图片文件的base64", method = RequestMethod.POST)
    public Object getPicBase64(@RequestBody FileUploadRecord fileUploadRecord) {
        log.info("进入 获取到图片文件的base64 控制器方法，fileUploadRecord：{}，", JSONObject.toJSONString(fileUploadRecord));
        if(fileUploadRecord == null){
            log.error("参数缺失请检查参数后重新提交~");
            return ResponseUtil.error("参数缺失请检查参数后重新提交");
        }
        return fileManagementService.getPicBase64(fileUploadRecord);
    }

}
```

#### 8. 结果如下

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f10e218524bd424baadde2f343e5d3cb~tplv-k3u1fbpfcp-watermark.awebp?)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/db3d618aa5f04be18a12f05f5742c71c~tplv-k3u1fbpfcp-watermark.awebp?)

**最后感谢大家耐心观看完毕，原创不易，留个点赞收藏是您对我最大的鼓励！**

* * *

### 🎉总结：

-   **更多参考精彩博文请看这里：[《陈永佳的博客》](https://juejin.cn/user/862483929905639/posts "https&#x3A;//juejin.cn/user/862483929905639/posts")**
-   **喜欢博主的小伙伴可以加个关注、点个赞哦，持续更新嘿嘿！** 
    [https://juejin.cn/post/7016528308451934221](https://juejin.cn/post/7016528308451934221)
