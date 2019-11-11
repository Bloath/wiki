对于ubuntu来讲，登陆的用户并非root，所以说在获取权限时，需要在命令前加sudo，但是sudo会将环境变量进行重置，明明通过修改`~/.bashrc`文件中添加了环境变量，但是sudo时扔然找不到路径。



解决方法：加入`sudo env PATH=$PATH`，但是每次加入都很麻烦，所以说直接使用别名即可，在`.bashrc中加入alias sudo="sudo PATH=$PATH"`即可

