Dubbo注意事项

-  # 主要typefeat:     增加新功能fix:      修复bug​# 特殊typedocs:     只改动了文档相关的内容style:    不影响代码含义的改动，例如去掉空格、改变缩进、增删分号build:    构造工具的或者外部依赖的改动，例如webpack，npmrefactor: 代码重构时使用revert:   执行git revert打印的message​# 暂不使用typetest:     添加测试或者修改现有测试perf:     提高性能的改动ci:       与CI（持续集成服务）有关的改动chore:    不修改src或者test的其余修改，例如构建过程或辅助工具的变动bash

- Dubbo 接口实现需要使用 `@org.apache.dubbo.config.annotation.Service` 不要用 Spring 的 Service