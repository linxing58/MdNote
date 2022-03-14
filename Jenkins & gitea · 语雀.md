# Jenkins & gitea · 语雀
    \# docker build -t ${PWD##\*/} --build-arg appName=${PWD##\*/} . 

    \# 以目录作为应用名

    ​

    FROM golang:alpine AS build

    ​

    \# 应用名

    ARG appName

    ​

    \# 为我们的镜像设置必要的环境变量

    ENV GO111MODULE=on \\

     CGO\_ENABLED=0 \\

     GOOS=linux \\

     GOARCH=amd64 \\

     GOPROXY="https://goproxy.io,direct"

    ​

    \# 工作目录

    WORKDIR /app

    ​

    \# 将代码复制到容器中(若包含go.mod go.sum 则注释下面的 go mod 一行)

    COPY . .

    ​

    \# 将我们的代码编译成二进制可执行文件

    RUN go mod init ${appName} && go mod tidy

    RUN go build -o ${appName}

    ​

    RUN echo "#!/usr/bin/env sh" > ./run.sh

    RUN echo "/app/${appName}" >> ./run.sh

    ​

    ​

    FROM alpine:3 as final

    \# 声明，否则会为空

    ARG appName

    ​

    WORKDIR /app

    COPY --from=build /app/${appName} .

    COPY --from=build /app/run.sh .

    ​

    \# 启动容器时运行的命令

    ENTRYPOINT \[ "sh", "./run.sh"\]

 [https://www.yuque.com/docs/share/7bc533e3-d803-41b1-b241-8c0233f92b23?#%20%E3%80%8AJenkins%20&%20gitea%E3%80%8B](https://www.yuque.com/docs/share/7bc533e3-d803-41b1-b241-8c0233f92b23?#%20%E3%80%8AJenkins%20&%20gitea%E3%80%8B)
