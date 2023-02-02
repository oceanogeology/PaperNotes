进入redis镜像：docker run --rm -it docker.io/library/redis:latest bash

链接redis： redis-cli -h r-bp19jg8evl44vctmh4.redis.rds.aliyuncs.com -p 6379

redis命令：

```
llen hobby_video_level_classification_debug

lrange hobby_video_level_classification_debug 6020 6026
lpush hobby_video_level_classification_debug 100002678490824704  # list格式
rpush hobby_video_level_classification_debug 100002231814225920 100002403612917760

set hobby_video_level_classification_debug 100002678490824704 # key-value格式

del hobby_video_level_classification_debug
```

```
import redis
import yaml

with open("config/plug_config.yaml", encoding="u8") as f:
    redis_config = yaml.load(f, yaml.CLoader)["redis"]["prod"]
conn_redis = redis.Redis(host=redis_config["host"], port=redis_config["port"],
                         password=redis_config["password"])
\# conn_redis.rpush("ai_tagging_multimodal_video_understanding_2.0_test", 8063)
\# conn_redis.rpush("ai_tagging_multimodal_video_understanding_2.0_test", 8064)
\# conn_redis.rpush("ai_tagging_multimodal_video_understanding_2.0_test", 8065)
\# conn_redis.rpush("ai_tagging_multimodal_video_understanding_2.0_test", 8066)
\# conn_redis.rpush("ai_tagging_multimodal_video_understanding_2.0_test", 8067)
\# conn_redis.rpush("ai_tagging_multimodal_video_understanding_2.0_test", 8068)
\# conn_redis.rpush("ai_tagging_multimodal_video_understanding_2.0_test", 511075)
\# conn_redis.rpush("ai_tagging_multimodal_video_understanding_2.0_test", 511076)
\# conn_redis.rpush("ai_tagging_multimodal_video_understanding_2.0_test", 511077)
\# conn_redis.rpush("ai_tagging_multimodal_video_understanding_2.0_test", 511078)
\# conn_redis.rpush("ai_tagging_multimodal_video_understanding_2.0_test", 8032)
\# conn_redis.rpush("ai_tagging_multimodal_video_understanding_2.0_test", 8183)
\# conn_redis.rpush("ai_tagging_multimodal_video_understanding_2.0_test", 8184)
\# conn_redis.rpush("ai_tagging_multimodal_video_understanding_2.0_test", 8185)
\# conn_redis.rpush("ai_tagging_multimodal_video_understanding_2.0_test", 8186)
\# conn_redis.rpush("ai_tagging_multimodal_video_understanding_2.0_test", 8055)
\# conn_redis.rpush("ai_tagging_multimodal_video_understanding_2.0_test", 8056)
\# conn_redis.rpush("ai_tagging_multimodal_video_understanding_2.0_test", 8057)
\# conn_redis.rpush("ai_tagging_multimodal_video_understanding_2.0_test", 8058)
\# conn_redis.rpush("ai_tagging_multimodal_video_understanding_2.0_test", 8059)

\# result = conn_redis.delete("ai_tagging_multimodal_video_understanding_2.0")
len_data = conn_redis.llen("ai_tagging_multimodal_video_understanding_2.0")
\# len_data = conn_redis.lpop("ai_tagging_multimodal_video_understanding_2.0")
print(int(len_data))


\# print(result)
\# len_data = len_dataconn_redis.llen("ai_tagging_multimodal_video_understanding_2.0")
\# len_data = conn_redis.lpop("ai_tagging_multimodal_video_understanding_2.0")
\# print(int(len_data))
```

