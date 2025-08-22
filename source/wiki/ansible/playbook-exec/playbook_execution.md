---
layout: wiki  # 使用wiki布局模板
wiki: ansible # 这是项目id，对应 /_data/wiki/ansible.yml
title: 6.1 运行 Playbook
order: 0
---

# 运行 Playbook

准备运行您的 Ansible Playbook 吗？

运行复杂的 Playbook 需要一些试错，因此请了解 Ansible 提供的一些功能，以确保成功执行。您可以使用“干运行” Playbook 验证任务，使用 start-at-task 和 step 模式选项来高效地排除 Playbook 故障。您还可以使用 Ansible 调试器来纠正执行过程中的任务。Ansible 还通过异步 Playbook 执行和标签提供了灵活性，让您可以运行 Playbook 的特定部分。

- 验证任务：检查模式和差异模式
  - 使用检查模式
  - 使用差异模式
- 理解权限提升：become
  - 使用 become
  - become 的风险和限制
  - become 和网络自动化
  - become 和 Windows
- 标签
  - 使用 tags 关键字添加标签
  - 特殊标签
  - 在运行 Playbook 时选择或跳过标签
- 执行 Playbook 进行故障排除
  - start-at-task
  - 步进模式
- 调试任务
  - 启用调试器
  - 解决调试器中的错误
  - 可用的调试命令
  - 调试器如何与 free 策略交互
- 异步操作和轮询
  - 异步即席任务
  - 异步 Playbook 任务
- 控制 Playbook 执行：策略等
  - 选择策略
  - 设置 fork 数量
  - 使用关键字控制执行
