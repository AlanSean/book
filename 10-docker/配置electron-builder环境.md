# 配置electron-builder环境

## 拉取electronuserland/builder环境

docker pull electronuserland/builder


##  创建映射 

把本地项目映射到，容器/project下
` docker run --rm -ti -v E:\github\image_compress:/project -w /project electronuserland/builder`