## hang's_deleted_flag

### 解题思路

正常来说考察的是简单的git命令。经不住有些同学直接去git文件里面翻找（我寻思这效果不是相同的吗，为什么整这么麻烦？还是学一下简单方便的git命令把。）

首先：`git log` 查看提交历史<img src="hang's_deleted_flag/img/image-20251210011212892.png" alt="image-20251210011212892" style="zoom: 25%;" />，认为第二版本存在flag。

然后：`git checkout 6f8c`切换到指定地址：<img src="wp/img/image-20251210011613946.png" alt="image-20251210011613946" style="zoom:25%;" />，直接获得flag图片：<img src="hang's_deleted_flag/img/image-20251210011530278.png" alt="image-20251210011530278" style="zoom:50%;" />

人眼识别一下：`r00t2025{W3lc0me_1+1_T2_G1t_wo2D}`

轻松搞定。