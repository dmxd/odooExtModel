ODOO Approval Flow
====
实现条件审批、会签、代签、加签、沟通、抄送等基本功能。

依赖
-------
模块依赖odoo的mail、hr等模块。

代码使用了networkx的python包，安装方法：pip3 install networkx。

主要功能
-------
* 条件审批
* 会签
* 代签
* 加签
* 沟通
* 审批完成抄送
* 驳回到任意节点
* 送审、取消审批、完成审批自动运行自定义代码

简单使用
-------
1. 流程配置
    * 配置审批节点
        * 审批类型，包括：组、用户、岗位、直属领导、部门领导等5种类型
            * 组。组下的所有用户有权参与审批
            * 用户。指定用户有权参与审批
            * 岗位。指定岗位下的员工关联的用户有权参与审批
            * 直属领导。单据的员工的经理(parent_id)关联的用户有权参与审批
            * 部门领导。单据的员工的部门(department_id)的经理(manager_id)关联的用户在权参与审批
        * 允许前加签，如果勾选，用户在审批单据时，可以指定任意一用户在审批当前节点前参与审批
        * 允许后加签，如果勾选，用户在审批单据时，可以指定任意一用户在审批当后节点前参与审批
        * 允许代签，如果勾选，用户在审批单据时，可以指定任意一用户审批当前节点
        * 需全组审批，如果勾选，则指定组下所有用户都必须参与审批
        * 需所有用户审批，如果勾选，则指定的所有用户都必须参与审批
        * 需岗位所有成员审批，如果勾选，则指定岗位下的员工关联的所有用户必须参与审批
        * 需直属领导的上级审批，如果勾选，单据员工的经理(parent_id)的经理(parent_id)关联的用户会在当前节点审批完成后，参与审批
        * 需部门领导的上级审批，如果勾选，单据员工的部门经理的经理关联的用户会在当前节点审批完成后参与审批
        * 仅单据公司，如果勾选，如果审批类型为组，则指定组下用户的当前公司字段值等于单据公司字段值才有权参与审批；如果审批类型为用户，则指定用户的当前公司字段值等于单据公司字段值才有权参与审批；如果审批类型为岗位，则指定岗位下员工的公司字段值等于单据公司字段值才有权参与审批
    * 配置审批流程
        * 有效
        * 适用公司
        * 条件，请遵循odoo的domain的写法，让单据适配审批流程
        * 完成后挱送类型，可选值有：用户、岗位，指定用户或岗位后，当单据审批完成，则会通过odoo自带的mail.message向相关用户发送系统通知。
        * 仅抄送单据公司
        * 送审自动运行，用户送审批自动运行自定义的函数，可用于一些送审条件判断，比如，订单如果没有明细，不能让送审
        * 取消审批自动运行，用户取消审批时自动运行自定义的函数，可用于一些清理工作
        * 完成后自动运行，单据审批完成自动运行自定义函数
2. 送审

    当当前用户等于单据的创建者(create_uid)时，当前用户可以送审，送审后，单据的审批步骤会保存在record_approval_state中。当再次送审时，不会再去计算步骤，而是直接读取保存的信息。代签和加签的信息不会被保存到单据的审批步骤中
3. 暂停审批

    当当前用户等于单据的创建者(create_uid)时，当前用户可以暂停单据的审批流程
4. 取消审批

    当当前用户等于单据的创建者(create_uid)时，当前用户可以取消单据的审批流程。当单据取消审批后，单据关联的所有的审批信息将被删除，保存在record_approval_state中的审批步骤也会被清理。再次送审时，将重新计算单据的审批步骤  
5. 驳回

    用户可驳回到当前节点前的任意节点，驳回到非开始节点时，程序会自动提交审批，开始一个新的审批实例。驳回到开始节点，则应由单据创建者修改相关信息后再次送审，开始一个新的审批实例。驳回时，未完成的步骤会保存在相应的表中    
      


注意
----
* 参与审批的model必须有company_id字段
* 如果节点审批类型是"岗位"、"直属领导"或"部门领导"，model必须有employee_id字段
* 配置好一个model的审批流后，需重启odoo，以便给model自动添加doc_approval_state(审批状态)字段和修改model的tree与form视图
* 一个单据可能会适配多条审批流程，程序不判断唯一性，而是任取一个审批流程
* 一张单据可能有多条审批路径，程序不判断审批路径的唯一性
* 单据的一个审批实例中，一个用户只会审批一次
* 单据在审批过程中或审批完成，不能修改或删除单据，如果传递approval_callback上下文，则可以修改单据。
* 通过上下文approval_supper解决多公司权限问题






