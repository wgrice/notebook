### Docker镜像加速

1. 针对Docker客户端版本大于 1.10.0 的用户

   您可以通过修改daemon配置文件/etc/docker/daemon.json来使用加速器

   ```shell
   sudo mkdir -p /etc/docker
   sudo tee /etc/docker/daemon.json <<-'EOF'
   {
     "registry-mirrors": ["https://bu1x10jp.mirror.aliyuncs.com"]
   }
   EOF
   sudo systemctl daemon-reload
   sudo systemctl restart docker
   ```