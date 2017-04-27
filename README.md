# AetherUpload-Laravel
提供超大文件上传的Laravel扩展包，基于Laravel 5开发，目前支持5.1、5.2、5.3、5.4版本。  
  
我们知道，在以前，文件上传采用的是直接传整个文件的方式，这种方式对付一些小文件是没有问题的。而当需要上传大文件时，此种方式不仅操作繁琐，需要修改web服务器和后端语言的配置，而且会大量占用服务器的内存，导致服务器内存吃紧，严重的甚至传输超时或文件过大无法上传。很显然，普通的文件上传方式已无法满足现在越来越高的要求。  
  
随着技术的发展，如今我们可以利用HTML5的分块上传技术来轻松解决这个困扰，通过将大文件分割成小块逐个上传再拼合，来降低服务器内存的占用，突破服务器及后端语言配置中的上传大小限制，可上传任意大小的文件，同时也简化了操作，提供了直观的进度显示。  

![示例页面](http://ww4.sinaimg.cn/mw690/69e23056gw1f9v66gs965j20gc0fngn5.jpg). 

# 功能特性
- [x] 百分比进度条  
- [x] 文件类型和大小限制  
- [x] 同步上传 *①*  
- [x] 自定义中间件  
- [x] 断线续传 *②*  
- [x] 自定义样式  
- [ ] 文件秒传（开发中）
- [ ] 组件同表单多次调用（暂未实现）
- [ ] 多文件队列上传（暂未实现）

*①：同步上传相比异步上传，在上传带宽足够大的情况下速度稍慢，但同步可在上传同时进行文件的拼合，而异步因文件块上传完成的先后顺序不确定，需要在所有文件块都完成时才能拼合，将会导致异步上传在接近完成时需等待较长时间。同步上传每次只有一个文件块在上传，在单位时间内占用服务器的内存较少，相比异步方式可支持更多人同时上传。*  
*②：断线续传和断点续传不同，断线续传是指遇到断网或无线网络不稳定时，在不关闭页面的情况下，上传组件会定时自动重试，一旦网络恢复，文件会从未上传成功的那个文件块开始继续上传。断线续传在刷新页面或关闭后重开是无法续传的，之前上传的部分已成为无效文件。*  

# 用法
0) 在终端内切换到你的laravel项目根目录，执行`composer require peinhu/aetherupload-laravel dev-master`  

1) 在`config/app.php`的`providers`数组中添加一行`Peinhu\AetherUpload\AetherUploadServiceProvider::class,`  
  
2) 执行`php artisan vendor:publish --tag=aetherupload --force`来发布一些文件和目录  
  
3) 赋予上传目录相应权限，在项目根目录下，执行`chmod 755 storage/app/uploads -R`    
  
4) 在浏览器访问`http://域名/aetherupload`可到达示例页面  

*提示：更改相关配置选项请编辑`config/aetherupload.php`。*  

# 优化建议
* （推荐）设置每天自动清除无效文件。  
由于上传流程存在意外终止的情况，如在传输过程中强行关闭页面或浏览器，将会导致已产生的文件部分成为无效文件，占据大量的存储空间，我们可以使用Laravel的任务调度功能来定期清除它们。  
在Linux中运行`crontab -e`命令，确保文件中包含这行代码：  
`* * * * * php /项目根目录的绝对路径/artisan schedule:run 1>> /dev/null 2>&1`  
在`app/Console/Kernel.php`中的`schedule`方法中添加以下代码：
```php
  $schedule->call(function () {
      (new \Peinhu\AetherUpload\Uploader())->cleanUpDir();
  })->daily();
```
* 提高临时文件读写速度。  
利用Linux的tmpfs文件系统，来达到将临时文件放到内存中快速读写的目的。执行以下命令：    
`mkdir /dev/shm/tmp`  
`chmod 1777 /dev/shm/tmp`  
`mount --bind /dev/shm/tmp /tmp`  

# 安全性
AetherUpload未使用MIME Type来检测上传文件类型，而是直接限制文件扩展名来防止上传可执行文件(默认屏蔽了常见的可执行文件扩展名)，因为MIME Type也可伪造并不安全，无法起到应有的作用。  
虽然做了诸多安全工作，但恶意文件上传是防不胜防的，建议将所有上传目录权限设置为755一劳永逸，这样即使恶意文件被上传了也无法执行。  

# 更新日志
**2017-04-27 v1.0.1**  

修复[局部刷新导致的上传无响应问题](https://github.com/peinhu/AetherUpload-Laravel/issues/6)  
后端代码结构大幅优化  
后端代码格式统一规范化  
前端代码改进  
一些其它改进  

**2016-07-13 v1.0.0正式版**  

添加完整说明，修正一些小问题。  

**2016-06-24 v1.0.0测试版**  

初次提交

# 许可证
使用GPLv2许可证, 查看LICENCE文件以获得更多信息。

