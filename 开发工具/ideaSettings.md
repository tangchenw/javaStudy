## IDEA核心配置文件

找到idea安装目录，进入其bin目录下可以看见两个核心配置文件idea.properties和idea64.exe.vmoptions。

![settings](C:\Users\汤琛\Desktop\学习资料\开发工具\images\settings.png)

> 建议使用idea自带菜单中的`Help -> Edit Custom VM Options` 来配置idea64.exe.vmoptions文件（JVM配置文件），使用`Help -> Edit Custom Properties`来配置idea.properties文件（属性配置文件）。

**常用修改参数**

- `idea.max.intellisense.filesize=2500`，该属性主要用于提高在编辑大文件时候的代码帮助。IntelliJ IDEA 在编辑大文件的时候还是很容易卡顿的。
- `idea.cycle.buffer.size=1024`，该属性主要用于控制控制台输出缓存。有遇到一些项目开启很多输出，控制台很快就被刷满了没办法再自动输出后面内容，这种项目建议增大该值或是直接禁用掉，禁用语句 `idea.cycle.buffer.size=disabled`。

### IDEA编码设置

![fontSettings](C:\Users\汤琛\Desktop\学习资料\开发工具\images\fontSettings.png)

