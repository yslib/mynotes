
#Git

### 添加submodule

git submodule add <url> <localpath>

此时只能clone下来这个submodule,如果添加的这个submodule还依赖其他submodule，应该进入这个submodule 运行git submodule [--recursive] update --init
或者 git submodule foreach git submodule update --init --recursive
