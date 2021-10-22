## Syntax
`docker <component> <commmand>`

**Components:**

- image
- container
- network
- volume
- ...

**Commands:**

- ls
- run
- exec
- stop
- pull
- prune
- ...

## image

- `docker image pull <image>` : tải image từ registry (default tag: latest)
- `docker image pull <image>:<tag>`
- `docker image push <image>:<tag>`
- `docker image ls` | `docker images`
- `docker image prune`: dọn dẹp image

**Short-hand:**

- `docker pull`
- `docker push`

**Đánh tag: username/image:tag**
## container

- `docker container run <image>`
- `docker container ls ` | `docker container ls -a` | `docker ps` | `docker ps -a`
- `docker container stop <container_id>`
- `docker container prune` | `docker rm`
- `docker container exec <container_id> <command>`: chạy 1 câu lệnh bên trong 1 container bất kỳ
- `docker logs -f <container_id>`

### flag

- -i: interactive (Ctrl + D để thoát)
- -t: tty
- -d: detach
**Short-hand:**
- `docker run`
- `docker stop`
- `docker exec -it`

**COMMAND**: lệnh đầu tiên docker sẽ chạy lần đầu tiên khi khởi động

## Container Life Cycle

- docker run => up => docker stop/kill => exited => docker rm => dead
    - exited => docker run => up

**alpine**: hệ điều hành linux đã đc rút gọn đến mức tối giản

- **pid(1)**: tiến trình chủ đạo, khi nó kết thúc thì container sẽ kết thúc

## Port Mapping

- VD:
docker (nginx) port 80 (internal) -> port 80 (local machine)

- Syntax: `docker run -p <target_port>:<container_port>...`

## Volume - bind mount

*Image is immutable. Container is stateless*

=> tắt container đi thì mọi dữ liệu mất hết
=> How to make container stateful ? -> use volume: bộ nhớ mà docker dùng để lưu trữ dữ liệu cho container (1 thư mục ảo do docker tạo ra và quản lý)
- bind mount: thao tác gắn volume vào container

- Syntax
    - `docker volume create [volume_name]`
    - `docker run -v [local_dir/volume]:[container_dir]`

- `docker run -v pgdata:/var/lib/postgresql/data -p 5432:5432 postgres`

## Dockerfile

```
FROM alpine:latest
RUN apk add bash // lệnh cài bash vào trong alpine
CMD ["bash"] // hook command: lệnh đầu tiên chạy khi khởi tạo container từ image
```

- `docker build -t <image_name>:<tag> -n <docker_name> .
    - dấu **.**: build context refer đến thư mục chứa Dockerfile. Toàn bộ nội dung bên trong folder chứ Dockerfile đc gọi là build context bao gồm cả nó.
- Docker client gửi toàn bộ build context đến docker deamon để build image
- *Cẩn thận*: Để những file không liên quan trong build context thì dữ liệu gửi đến docker deamon quá nặng khiến build image lâu hơn
    - using .dockerignore ~ .gitignore

### Keyworks

  - FROM <image> : chỉ định base image
  - RUN <command>:
  - WORKDIR <directory>: tạo 1 thư mục và đặt thư mục đó làm thư mục mặc định của container khi khởi tạo
  - COPY <src> <dest>: copy file từ máy local đi vào bên trong image
  - ADD <src/URL> <dest>: xịn hơn copy khi có thể tải từ internet về
  - EXPOSE <port>: thông báo port của ứng dụng đang chạy bên trong container, chỉ là 1 dòng comment
  - CMD ["command", "argument1", ...] : chỉ thực thi khi khởi chạy container

### Phân tích 1 số mẫu Dockerfile

#### samples - backend (Java)

```
FROM openjdk:8-jre
EXPOSE 8080
WORKDIR /app
COPY ./target/app-1.0-SNAPSHOT.jar .
CMD ["java", "jar", "app-1.0-SNAPSHOT.jar"]
```

*Build process*:
- Image là tập hợp nhiều layer. mỗi layer tương ứng với 1 câu lệnh trong Dockerfile.
- Khi build image, docker sẽ duyệt qua từng câu lệnh. qua mỗi câu lệnh, docker sẽ khởi tạo 1 container tạm thời từ image của layer trc đó, tạo ra thay đổi bên trong container đúng theo mô tả của câu lệnh và sau đó nó sẽ sao lưu container này lại thành 1 container mới và xong xuôi docker sẽ loại bỏ container tạm thời này và tiếp tục lặp lại việc này cho đến khi duyệt qua hết các câu lệnh


```
FROM node:alpine
WORKDIR /server
COPY ./*.json ./
COPY ./*.node_modules ./node_modules
COPY ./*.js ./
COPY ./public ./public
CMD ["node, "app.js"]
```