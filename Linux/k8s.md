```
kubectl get pods -n prod |grep level  # 查pod运行状态

kubectl get pod -n prod k8s-gpu-hobby-video-level-timesformer-prod-5cd9bc4f77-76rh8 -o yaml | grep -C 5 limit  # 查cpu和gpu的limit

kubectl logs -n prod k8s-gpu-hobby-video-level-timesformer-prod-5cd9bc4f77-ftzs2 k8s-gpu-hobby-video-level-timesformer-prod | tail -10 # 查看pod日志

kubectl exec -it -n prod k8s-gpu-hobby-video-level-timesformer-prod-5cd9bc4f77-lw4xm /bin/bash # 进入pod

kubectl describe pod -n prod k8s-gpu-hobby-video-level-timesformer-prod-5b898d7489-8d7fc

kubectl logs -n prod k8s-gpu-hobby-video-level-timesformer-prod-5b898d7489-8d7fc k8s-gpu-hobby-video-level-timesformer-prod | tail -1000 | grep '发往层级 队列 当前队列大小 frame_data_queue'


```



> 13    | k8s-node-dev-01-社区                                          | 172.21.0.98     
> 页码：1，每页行数：39，总页数：1，总数量：13
>
> [Host]> 13
>   ID    | 名称                                                             | 用户名
> +-------+------------------------------------------------------------------+-----------------------------------------------------------------+
>   1     | developer                                                        | developer
>   2     | hobby-root                                                       | root
> 提示：输入系统用户ID直接登录
> 返回：B/b

> ID> 1
> [developer@k8s-node-dev-01 hobby_level_classifier]$ pwd
> **/home/developer/prod/hobby_level_classifier**
> [developer@k8s-node-dev-01 hobby_level_classifier]$ ls
> hobby_level_classifier.log
>
> ID> 2
> [root@k8s-node-dev-01 ~]#cd 
> **/store/k8s/prod/nas-4819d838-bf70-4dbc-baca-f2f7b2e6f17a/hobby_level_classifier**
> [root@k8s-node-dev-01 hobby_level_classifier]# ls
> hobby_level_classifier.log             hobby_level_classifier.log.2022-07-06  
>
> 服务器模型地址：/nas/dk/code/timesformer_merge/checkpoints/level_merge_0712/M2512_bce_text/model_best.pt
> **k8s模型存放地址：/store/k8s/prod/nas-4819d838-bf70-4dbc-baca-f2f7b2e6f17a/model_data/level_classifier**

