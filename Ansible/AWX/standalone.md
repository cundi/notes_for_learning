# 离线安装AWX

应用场景：生成环境无法用公网，需要在主机本地加载导出的docker镜像。

1. 安装官方说明，构建5个awx的镜像
2. 导出镜像，并上传到目标主机
3. 登录目标主机，
4. 安装Python的docker客户端，

    ```shell
    pip install docker
    ```

5. 修改docker.sock的权限

    ```shell
    chmod o=rwx /var/run/docker.sock
    ```

6. 修改`installer/roles/local_docker/tasks/standalone.yml`中的`{{ awx_web_docker_actual_image }}`和`{{ awx_task_docker_actual_image }}`为`ansible/axw_web:2.1.2`、`ansible/awx_task:2.1.2`。

7. postgres数据持久化，修改installer文件夹下的iventory的`postgres_data_dir`。

8. 返回awx的安装目录，即`awx-2.1.2/installer`，然后用下面的命令来执行指定的task。

    ```shell
    ansible-playbook -vvv --step --start-at-task="Activate AWX web container" -i ./inventory install.yml
    ```

    注意，task交互提示时请选择`Set properties with postgres for awx_web`,`Set properties with postgres for awx_task`，