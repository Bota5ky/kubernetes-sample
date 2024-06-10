### Deployment		

set 命令：kubectl set image deployment/nginx busybox=busybox nginx=nginx:1.9.1

`.spec.revisionHistoryLimit`：限制旧 RS 版本保留的数目，在回滚版本时可用

回退版本指定具体版本：kubectl rollout undo deployment/abc --to-revision=3

查看具体版本的信息：kubectl rollout history daemonset/abc --revision=3